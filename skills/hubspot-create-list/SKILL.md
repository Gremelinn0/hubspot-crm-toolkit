---
name: hubspot-create-list
description: Crée listes/segments HubSpot via Chrome MCP UI — flow standardisé Add filter → property → operator → value → AND filters → folder → save → rename. Auto-retry Chrome extension bugs. Post-create update inventory + actions-log. Trigger quand Florent dit "crée liste HubSpot", "nouvelle segment", "create list X", "ajoute liste Y au portail", ou pour création multi-listes (4 listes 2026+ par exemple).
---

# Skill — HubSpot Create List (UI flow standardisé)

## §0 Pré-requis OBLIGATOIRES

Avant invocation skill :
1. **Skill `hubspot-crm` §0 appliqué** : doc Notion lue + audit existant fait + naming/folder validé
2. **Validation explicite Florent dans chat** (règle CLAUDE.md projet §1 — action mutante client)
3. **Chrome MCP perms accordées** dans `.claude/settings.local.json`
4. **Tab Chrome actif** (vérifier via `tabs_context_mcp`)

Si pré-requis manquant → STOP, propose à Florent de combler.

---

## §1 Input spec

Pour chaque liste à créer, le skill attend :

```yaml
listes:
  - nom: "Main list Hospitality 2026+"          # naming aligné existant
    object_type: "Companies"                     # Companies / Contacts / Deals / Tickets
    type: "Active"                               # Active (dynamique) ou Static
    folder: "Clients"                            # nom folder ; sera matché dans tree
    filters:
      - property: "Industry"
        operator: "is any of"
        value: "Hospitality"
      - property: "Create date"
        operator: "is after"
        value: "12/31/2025"                      # MM/DD/YYYY format
    portal: "21450353"                           # portail HubSpot
    client_slug: "pastry-chefs-boutique"         # pour update inventory
```

Multi-listes : array de specs.

---

## §2 Workflow UI — par liste

### Étape 1 : Navigate create wizard

```
URL: https://app.hubspot.com/contacts/{portal}/objectLists/create?objectTypeId={object_type_id}
```

`object_type_id` :
- Companies = `0-2`
- Contacts = `0-1`
- Deals = `0-3`
- Tickets = `0-5`

### Étape 2 : Add filters (boucle pour chaque filter)

Pour chaque filter dans `filters[]` :

1. `find` "Add filter button center" (1ère) ou "Add filter button inside Group 1" (suivantes)
2. `left_click` → ouvre dropdown
3. `find` "Search Company properties searchbox" → `left_click` → `type` property name
4. `find` "{property} property option button" → `left_click`
5. **Si property = enum/multi-select** (Industry, Lifecycle, etc.) :
   - `find` "Enter value dropdown" → `left_click`
   - `type` value
   - `find` "{value} checkbox option" → `left_click`
   - `key` "Escape"
6. **Si property = text contains** :
   - `find` "operator dropdown" → `left_click`
   - `find` "contains any of operator option" → `left_click`
   - `find` "value input combobox" → `left_click` → `type` value → `key` "Enter"
7. **Si property = date** :
   - `find` "operator dropdown" → `left_click`
   - `find` "is after operator option" → `left_click` (note : "is on or after" pas dispo, utilise "is after [jour-1]")
   - `find` "date input MM/DD/YYYY" → `left_click` → `type` "MM/DD/YYYY" → `key` "Escape"

### Étape 3 : Next + Review

1. `find` "Next button top right" → `left_click`
2. Wait modal "Review and save"

### Étape 4 : Change folder

1. `find` "Change folder link" → `left_click`
2. `find` "{folder} folder option" → `left_click`
3. `find` "Move button" → `left_click`

### Étape 5 : Save

1. `find` "Save and process segment button" → `left_click`
2. Wait 6s pour processing
3. Capture URL → extract list ID `objectLists/{ID}/filters`

### Étape 6 : Rename

1. `find` "edit pencil button next to segment name title" → `left_click`
2. Wait
3. `find` "Segment name textbox"
4. `left_click` + `key` "ctrl+a" + `key` "Backspace"
5. Wait 3s (Chrome ext stabilise)
6. `type` nom liste
7. `find` "Save changes button" → `left_click`
8. Wait 4s + screenshot pour vérifier titre OK

---

## §3 Auto-retry bugs Chrome MCP

**Bug fréquent** : `Failed to type: Cannot access a chrome-extension:// URL of different extension`

**Cause** : Chrome extension conflict après UI element re-render

**Workflow retry** (cap 3 tries) :
1. `wait 3-4s` après erreur
2. Re-`find` même element (ref change après re-render)
3. Re-tente `type` ou `left_click`
4. Si fail 3x → `screenshot` + report Florent

**Bug rename concat** : si triple_click pas tout sélectionné, type concatène à existant.

**Workflow rename robust** :
1. `left_click` textbox
2. `key` "ctrl+a"
3. `key` "Backspace"
4. `wait 3s` (laisse Chrome stabiliser)
5. `type` nouveau nom
6. Vérifier screenshot : si concat → re-clear (ctrl+a + Backspace) + re-type

---

## §4 Validation pré-création

### Naming convention check

- Aligné format existant ? (ex : "Main list X N" si existing utilise ce pattern)
- Langue cohérente (fr OU anglais, pas mix)
- Pas de doublon dans `listes-inventory.md` client

### Folder check

- Folder existe dans HubSpot tree ?
- Cohérent avec logique métier (cf hubspot-crm §0 v3)

### Filters check

- Property name exact (case-sensitive HubSpot)
- Value matches enum HubSpot (Hospitality, Restaurants, Gambling & Casinos, etc.)
- Date format MM/DD/YYYY US

Si validation fail → STOP + propose ajustement Florent.

---

## §5 Post-create updates

Après création successful, update :

### Update `listes-inventory.md` client

Path : `memory/clients/{client_slug}/listes-inventory.md`

Ajouter row dans section appropriée (§1.2 nouvelles dynamiques, ou autre selon contexte) :

```markdown
| **{ID}** | {nom} | Active dynamique | **{vol}** | `{filtre résumé}` | https://app.hubspot.com/contacts/{portal}/objectLists/{ID}/filters |
```

Update §15 changelog :

```markdown
| 2026-XX-XX | Création liste | {nom} (ID {ID}) | Session Claude via Chrome MCP — folder {folder}, filtres : {filtres}, vol initial : {vol} |
```

### Update `actions-log.md` client

Append entry sous date courante.

### Update CLAUDE.md client si snapshot listes change

Si listes critiques (top 8) changées → maj snapshot dans `CLAUDE.md` client §2.

---

## §6 Multi-listes en batch

Pour créer N listes d'un coup (ex : 4 listes 2026+ par secteur) :

1. **Boucle** sur chaque spec
2. Entre 2 listes : `navigate URL create` (reset wizard)
3. **Pas de batch parallèle** (Chrome MCP single tab)
4. Update inventory progressif après chaque liste (atomic, pas en bulk fin)

---

## §7 Exemple invocation

**Florent dit** : "crée 4 listes 2026+ secteurs Pastry Chef"

**Skill exécute** :

```yaml
listes:
  - nom: "Main list Hospitality 2026+"
    object_type: "Companies"
    type: "Active"
    folder: "Clients"
    filters:
      - {property: "Industry", operator: "is any of", value: "Hospitality"}
      - {property: "Create date", operator: "is after", value: "12/31/2025"}
    portal: "21450353"
    client_slug: "pastry-chefs-boutique"
  - nom: "Main list Hotels 2026+"
    # ...similar
```

**Output** :

```
✅ 4 listes créées :
- 627 Main list Hospitality 2026+ (15 companies)
- 628 Main list Hotels 2026+ (1 company)
- 629 Main list - Restaurants 2026+ (0 companies)
- 630 Main list Casinos 2026+ (0 companies)

Inventory + actions-log mis à jour.
```

---

## §8 Limite actuelle + future API

### Limite UI

- Chrome MCP fragile (extension URL bugs)
- ~30s par liste UI vs ~2s API
- Pas de batch parallèle

### Future migration API

HubSpot Lists API v3 endpoint : `POST /crm/v3/lists/`

Body :
```json
{
  "name": "Main list Hospitality 2026+",
  "objectTypeId": "0-2",
  "processingType": "DYNAMIC",
  "filterBranch": {
    "filterBranchType": "AND",
    "filterBranches": [],
    "filters": [
      {"property": "industry", "operator": "IN", "values": ["HOSPITALITY"]},
      {"property": "createdate", "operator": "GTE", "value": "2026-01-01T00:00:00Z"}
    ]
  }
}
```

Auth : Private App token dans `.claude/settings.json` (déjà présent pour HubSpot MCP).

**Migration future** : remplacer §2 workflow UI par appel API direct via fetch dans script Node/Python invoqué par skill. ~10x plus rapide + déterministe.

---

## §8bis — Méthode API PROUVÉE + patterns appris (session 2026-06-15)

L'**API v3 Lists est LA méthode** (l'UI auto fige le builder ~45s/interaction). Endpoints : créer `POST /crm/v3/lists` · **modifier le filtre** `PUT /crm/v3/lists/{id}/update-list-filters {filterBranch}` · renommer `PUT …/update-list-name?listName=` · déplacer `PUT …/folders/move-list {listId,newFolderId}` · créer dossier `POST …/folders {name,parentFolderId}` (⚠️ le `folderId` revient au **top-level** de la réponse). Token Private App dans `secrets.local`.

**Patterns de filtre (l'API refuse certaines formes au CREATE) :**
- **Dates = points ABSOLUS** (`{year,month,day,…,timeType:"DATE"}` + `IS_AFTER`/`IS_BEFORE`). Une **plage relative** (`indexReference:YEAR`) est **refusée au CREATE** (seuls TODAY/NOW), et `upperBoundEndpointBehavior:EXCLUSIVE` refusé → INCLUSIVE. Le plus simple = `closedate IS_AFTER 2025-12-31` (pas de borne haute).
- **« même deal » (strict)** = les 2 conditions (won ET closedate) dans **UNE seule** branche ASSOCIATION Deal (2 branches = 2 deals possibles).
- **« 1er achat / jamais avant »** = **PAS exprimable** sur une association (assoc = `IN_LIST` only, **aucune négation**). → créer une **liste-pivot** (« déjà client avant ») + filtrer `NOT_IN_LIST` sur l'**appartenance de liste** (la négation marche sur les listes, pas les assoc).
- **Miroir Contact** = `Email known + Marketing contact + ASSOC<Company>(membre de la liste Entreprise correspondante)`. ⚠️ **double imbrication interdite** (Contact→Company→Deal) → passer par l'appartenance à la liste Entreprise.

**Descriptions = UI-only** (aucun champ `description` dans l'API). Convention : **clair + concis + RAPPELER LE FILTRE**, en **anglais** (client US). Textes pré-écrits prêts à coller → `descriptions-EN.md`.

**Rangement** : dossier client = 2 sous-dossiers par OBJET (📇 Contacts newsletters / 🏢 Entreprises pilotage) ; briques techniques (pivot, mesures) dans « Z - Technique » hors du dossier client. **Le nom dit le critère.**

Build complet de référence (filtres exacts + map des IDs) = `memory/clients/pastry-chefs-boutique/segments-audit-cartographie.md` §5ter.

---

## §9 Anti-patterns

- ❌ Créer sans validation explicite Florent (règle §1 projet)
- ❌ Skip étape §0 (audit existant) → risque doublon
- ❌ Naming inventé ("Nouveaux comptes X" au lieu de "Main list X")
- ❌ Folder par défaut au lieu de logique métier
- ❌ Oublier post-create update inventory
