---
name: hubspot-segments-audit
description: Cartographie + audit de TOUS les segments/listes d'un compte HubSpot via API — LA base pour « aspirer » et comprendre la data d'un compte client. Pull complet (filtres réels, volumes, usage), résumé lisible de chaque filtre, flags santé (vide / statique / non-utilisé / doublon / références inter-segments), et génère un doc d'audit (familles + cartographie + bilan + recommandations) réutilisable aussi pour l'IA HubSpot (Breeze). Triggers "cartographie segments", "audit segments", "aspirer la data du compte", "analyse tous les segments", "carte des listes HubSpot", "comprendre le CRM client", "audit complet du compte".
---

# Skill — HubSpot Segments Audit & Cartographie

## §0 But — la 1ʳᵉ chose à faire pour comprendre un compte

Quand on arrive sur un compte CRM (ou qu'on veut « voir clair »), on **cartographie TOUS ses segments** : c'est la base pour comprendre comment la data est organisée et pour piloter ensuite (campagnes, ménage, refonte). Ce skill **aspire l'inventaire complet via API**, résume chaque filtre en clair, flag les problèmes de santé, et produit un **doc d'audit** (pour nous + collable dans un projet HubSpot pour Breeze).

## §1 Méthode = API d'abord (les 2 options)

L'**API est LA base** : rapide, complète, déterministe (ex. 78 segments + filtres + santé en une passe). L'**IA HubSpot (Breeze)** = aller segment par segment dans l'UI lui demander l'analyse → **lent + manuel + non scalable** → on la garde en **complément** (générer des descriptions, du narratif), pas pour l'aspiration de masse.

**Endpoints** (Private App token, scope `crm.lists.read` [+ `crm.objects.*.read` pour les vérifs de volume]) :
- `POST /crm/v3/lists/search` `{count, offset}` → tous les segments (paginer via `hasMore`/`offset`). `additionalProperties` : `hs_folder_name`, `hs_list_size`, `hs_list_reference_count`.
- `GET /crm/v3/lists/{id}?includeFilters=true` → `filterBranch` + `processingType` (DYNAMIC/SNAPSHOT/MANUAL).

## §2 Script réutilisable (pull + résumé + flags → dump UTF-8)

> ⚠️ Windows : écrire la sortie dans un **fichier UTF-8** (pas `print` → crash cp1252 sur accents/emoji). Token depuis un fichier gitignored (cf `hubspot-marketing-segments`).

```python
import json,urllib.request,urllib.error
TOKEN=open("<path>/secrets.local").read().split("HUBSPOT_PRIVATE_APP_TOKEN=",1)[1].split("\n")[0].strip()
H={"Authorization":"Bearer "+TOKEN,"Content-Type":"application/json"}
def call(m,u,b=None):
    d=json.dumps(b).encode() if b is not None else None
    r=urllib.request.Request(u,data=d,method=m,headers=H)
    try:
        x=urllib.request.urlopen(r);bd=x.read().decode();return x.status,(json.loads(bd) if bd.strip() else {})
    except urllib.error.HTTPError as e: return e.code,{}
OBJ={"0-1":"Contact","0-2":"Company","0-3":"Deal","0-5":"Ticket"}
def opstr(f):
    p=f.get("property");o=f.get("operation",{});opr=o.get("operator","")
    if "values" in o: v=",".join(map(str,o["values"]))[:70]
    elif "value" in o: v=str(o.get("value"))[:40]
    elif "timePoint" in o:
        tp=o["timePoint"];v="%s-%02d-%02d"%(tp.get("year"),tp.get("month") or 0,tp.get("day") or 0)
    else: v=""
    return ("%s %s %s"%(p,opr,v)).strip()
def bstr(b,d=0):                       # résume un filterBranch en texte lisible
    t=b.get("filterBranchType")
    if t=="ASSOCIATION":
        inner=[opstr(f) for f in b.get("filters",[])]+[bstr(x,d+1) for x in b.get("filterBranches",[])]
        return "ASSOC<%s>(%s)"%(OBJ.get(b.get("objectTypeId"),b.get("objectTypeId"))," & ".join(p for p in inner if p))
    parts=[opstr(f) for f in b.get("filters",[])]+[bstr(x,d+1) for x in b.get("filterBranches",[])]
    s=(" OR " if t=="OR" else " AND ").join(p for p in parts if p)
    return "("+s+")" if d>0 and s else s
# pull all + GET filters
lists=[];off=0
while True:
    c,r=call("POST","https://api.hubapi.com/crm/v3/lists/search",{"count":100,"offset":off})
    lists+=r.get("lists",[])
    if not r.get("hasMore"): break
    off=r.get("offset",off+len(r.get("lists",[])))
from collections import defaultdict,Counter
names=Counter(l["name"] for l in lists); g=defaultdict(list)
for l in lists:
    ap=l.get("additionalProperties",{}); c,fr=call("GET","https://api.hubapi.com/crm/v3/lists/%s?includeFilters=true"%l["listId"])
    L=fr.get("list",{}); fb=L.get("filterBranch"); pt=L.get("processingType","?")
    flags=[f for f,c in [("VIDE",ap.get("hs_list_size") in("0",0)),("STATIQUE",pt in("SNAPSHOT","MANUAL")),("used=0",ap.get("hs_list_reference_count") in("0",0)),("DUP",names[l["name"]]>1)] if c]
    g[ap.get("hs_folder_name") or "(racine)"].append((l["listId"],OBJ.get(l["objectTypeId"]),l["name"],ap.get("hs_list_size"),ap.get("hs_list_reference_count"),pt,bstr(fb) if fb else "(membership/import — non rendu)",flags))
with open("<path>/_segments-dump.local","w",encoding="utf-8") as fo:
    for folder in sorted(g):
        fo.write("\n## %s (%d)\n"%(folder,len(g[folder])))
        for x in g[folder]: fo.write("- **%s** | %s | sz=%s | used=%s | %s | `%s`%s\n"%(x[2],x[1],x[3],x[4],x[5],x[6]," ⚠️ "+",".join(x[7]) if x[7] else ""))
```

## §3 Flags santé (le « bilan »)

- **VIDE** (size 0) · **STATIQUE** (`SNAPSHOT`/`MANUAL` = figé) · **used=0** (pas de ref workflow/email/ad/report) · **DUP-NOM**.
- 🔴 **Références inter-segments** : un filtre `filterType: IN_LIST` (« est membre de la liste X ») fait qu'un segment **dépend d'un autre**. → **`used=0` ne garantit PAS qu'un segment est libre** → on **ARCHIVE, jamais supprime** (CLAUDE.md agence). Détecter : chercher `IN_LIST` / `listId` dans les `filterBranch`.
- Filtre rendu `(membership/import — non rendu)` = filtre d'appartenance/source-import que l'export API ne détaille pas → vérifier dans l'UI si besoin, **pas cassé**.

## §4 Le doc d'audit (livrable type)

Structure (cf `segments-audit-cartographie.md` Pastry Chef) : **(1)** vue d'ensemble par familles métier · **(2)** 🩺 bilan santé (mal-paramétré / vide / statique / inutile / doublon) · **(3)** segments importants · **(4)** cartographie complète par dossier (avec filtre + flags) · **(5)** cible/recommandations · **(6)** méthode. → **collable dans un projet HubSpot pour Breeze** (contexte IA).

**Rigueur** : ne jamais conclure « mal paramétré » sur un simple indice (taille…) → **inspecter le `filterBranch` réel** avant d'affirmer (cas Pastry Chef : fausse alerte « Clients - X » corrigée après inspection).

## §5 Complément Breeze (IA HubSpot)
Utile pour : **générer des descriptions** de segments (bouton « Generate description with AI » — **UI only, pas d'API**), narratif. Pas pour l'aspiration de masse (l'API §1-2 fait ça mieux).

## §6 Prérequis
Private App token (`crm.lists.read` + `crm.objects.contacts/companies.read`), stocké gitignored (`*.local`). Cf `hubspot-marketing-segments` §2 pour la création du token.

## §7 Cas inaugural — Pastry Chef's Boutique (2026-06-13)
78 segments cartographiés + audités via ce flow → `memory/clients/pastry-chefs-boutique/segments-audit-cartographie.md` (dump : `_segments-dump.local`). Findings : familles métier, « Clients - X » bien construits (après inspection), 6 imports statiques à archiver, références inter-segments (→ ne pas supprimer).

## Skills liés
`hubspot-marketing-segments` (créer les listes contacts) · `crm-investigation-output` (format de rendu) · `revops-api-scripts` (scripts API) · `hubspot-crm` (règles CRM) · `claude-breeze` (le jumeau UI-first / Breeze — quand une fonctionnalité liée aux segments n'a pas d'API propre à aspirer, ou pour déléguer un diagnostic à Breeze).
