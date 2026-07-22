---
name: hubspot-email-design
description: >-
  Créer/livrer un EMAIL avec un vrai DESIGN dans HubSpot (marketing ou automation) sans se battre avec les éditeurs natifs. Leçon centrale (prouvée douloureusement 2026-07-06) — un export Claude Design `.dc.html` est un BUNDLE WEB (JS + `<script>` + logo SVG), PAS un HTML email → ne jamais l'importer (l'IA "Generate design" HubSpot gèle/échoue dessus). Méthode qui marche : réécrire un HTML EMAIL-SAFE à la main (tables + CSS inline, zéro JS/SVG, images hébergées File Manager) → livrer en "coded email". **Règle de fidélité (§0.1)** : email-safe est une TRADUCTION du design, jamais une refonte — interdit de substituer (logo texte au lieu du vrai logo image) ou de simplifier (retirer la nav, remodeler le footer). Porte l'extraction du bundle Claude Design (le vrai HTML vit dans `__bundler/template`), le pipeline PNG pour les polices script/médaillons, le pattern d'URL File Manager, le contournement redaction, et les anti-galères Chrome MCP. Triggers "email hubspot", "email codé hubspot", "design email hubspot", "refaire le design d'un email", "redesign email hubspot", "reproduire une maquette en email", "reproduire fidèlement un design en email", "template email hubspot", "newsletter hubspot avec design", "créer un email marketing/automation hubspot", "mail post-commande hubspot".
---

# HubSpot Email Design — livrer un email designé sans galère

> Cas inaugural : Pastry Chef's Boutique, mail post-commande « How was your recent order? », 2026-07-06. Détail complet : `memory/clients/pastry-chefs-boutique/email-post-commande-flow.md`.

## 🔎 §0.0 — RÈGLE N°1 : la VÉRIF passe par la PREVIEW NATIVE de HubSpot (gravée 2026-07-16)

> **Politique marketing** (Florent 2026-07-16) : *« C'est utiliser le MCP Chrome, tu verrais que tu peux faire des previews. Ce serait bien de créer une compétence où tu utilises bien la preview de l'outil. »*
> **Cas inaugural** : j'ai passé **une heure** à interroger l'API, grepper le DOM et rendre le HTML **en local** pour savoir si 2 icônes s'affichaient — alors que HubSpot **rend l'email dans 9 clients en 2 clics**. Florent a ouvert la preview et a eu la réponse tout de suite.

**Avant de déclarer un email bon, on ne devine pas : on ouvre la preview HubSpot via Chrome MCP.**

| Vue | Sert à | Vaut quoi |
|---|---|---|
| **Rendu web officiel** — onglet `email.<domaine>/-temporary-slug-…?hs_preview=<previewKey>-<emailId>` | **LA source autoritaire.** Même origine → `javascript_tool` lit le DOM : `img.complete && img.naturalWidth>0` prouve qu'une image **charge vraiment**. | ✅ **Fait foi** |
| **`Preview as a contact`** (même écran, 1er onglet) | Voir l'email avec les tokens résolus | ✅ **Fiable** (croisé OK le 16/07) |
| **`/client_test`** — « Preview in an email client », 9 clients (Litmus) | Voir les **écarts entre clients** (Outlook / Gmail / mobile) | ⚠️ **Vignettes CACHÉES** — cf piège |
| **Canvas de l'éditeur** | Éditer, rien d'autre | ❌ **Ne fait PAS foi** |

**🚨 LE PIÈGE — les vignettes Litmus sont CACHÉES par client, et republier ne les rafraîchit pas.** Prouvé le 16/07 : Outlook 2016 montrait les nouvelles icônes pendant que Gmail montrait **les émojis de l'ancienne version** — des émojis que l'email **ne contenait plus** (0 occurrence : API **et** rendu web). Florent : *« j'ai republié pour voir, ça ne change rien »*.
→ **Une vignette Litmus divergente n'est PAS une preuve de bug.** Ce piège m'a fait bâtir une **fausse théorie** (« le module `rich_text` assainit les `<img>` ») et j'ai failli refondre un email **qui marchait**. **Rendu web d'abord, vignettes ensuite.**

**🔬 Verdict complet du 16/07 (3 vues croisées, même email)** : rendu web **✅ les 4 images chargent** (`complete && naturalWidth>0`) · `Preview as a contact` **✅ icônes visibles** · Litmus Outlook 2016 **✅** · Litmus Gmail **❌ montre les émojis d'une version antérieure**. → **C'est la vignette Litmus qui ment, pas l'email.** Republier ne la rafraîchit pas, et l'incohérence est **par client** (Outlook bon, Gmail périmé) — donc même « le reste bugge partout sauf le web » ne prouve rien. **2 vues concordantes suffisent** (web + contact) : ne jamais refondre un email sur la foi d'une seule vignette.

**La procédure (3 étapes, dans l'ordre)** :
1. Ouvrir l'email → **`Preview & test`** → récupérer l'onglet du **rendu web** (`previewKey` dispo aussi via `GET /marketing/v3/emails/{id}`).
2. `javascript_tool` sur CET onglet → lister les `<img>` (`naturalWidth`, `complete`) + grepper le texte (émojis résiduels, tokens `{{ … }}` non résolus).
3. **Puis seulement** les 9 vignettes, pour juger les **écarts entre clients** — jamais pour dire si l'email est correct.

❌ **Anti-patterns** : juger sur le **canvas de l'éditeur** · conclure « cassé » depuis **une seule vignette** · rendre le HTML **en local** et croire que ça prouve le rendu HubSpot · déclarer un email bon **sans avoir ouvert la preview**.

---

## 🎯 §0.1 — RÈGLE DE FIDÉLITÉ : email-safe est une TRADUCTION, pas une refonte (gravée 2026-07-21)

> **Cas inaugural** : redesign de l'email Trustpilot Pastry Chef. J'ai réécrit le design en email-safe *correctement sur la technique*, mais en me donnant des libertés que personne n'avait demandées : **logo remplacé par du texte** (Georgia au lieu du vrai logo image), **nav du header supprimée**, **footer remodelé**. Florent : *« je ne comprends pas ce que tu as fait… est-ce que tu sais ce qu'il fallait faire ? »*. Le doc client disait pourtant noir sur blanc « header = vrai logo image ». **Le design de Florent EST le cahier des charges — pas une source d'inspiration.**

Le piège est subtil parce qu'il se déguise en bon sens : *« ces liens de nav sont morts en email, autant les enlever »*, *« la police script ne s'embarque pas, autant mettre du texte »*, *« ce footer est chargé, autant le simplifier »*. **Chacune de ces pensées est une trahison du design.** La contrainte email-safe autorise à changer le **MOYEN technique**, jamais à **retirer ou remplacer** un élément :

| Élément qu'on ne peut pas rendre à l'identique | ❌ Ce qu'on est tenté de faire | ✅ Le bon réflexe email-safe |
|---|---|---|
| Logo (image/SVG dans la maquette) | le retaper en **texte** | **héberger le vrai logo image** (File Manager) et le pointer |
| Barre de nav du header | la **supprimer** (« liens morts ») | la **garder** en liens `<a>` dans une table (pointer vers le site) |
| Footer riche (colonnes, newsletter) | le **remodeler** en version courte | le **reproduire colonne par colonne** en tables |
| Filigrane / calque de fond | le **droper** | le **rendre en image de fond** (PNG) positionnée pareil |
| Police script (Allura, Balmonte…) | fallback **Georgia/Arial** | **rendre le mot en PNG** (cf §2bis) |

**La question à se poser devant chaque écart : « est-ce que je change le MOYEN, ou est-ce que je change le RÉSULTAT VISUEL ? »** Changer le moyen (flex→table, SVG→PNG, webfont→image) = OK et obligatoire. Changer le résultat (un élément en moins, un logo différent, une mise en page « nettoyée ») = interdit sans que Florent l'ait demandé. Au moindre doute « je simplifie ou je reproduis ? » → **on reproduit**, et si un élément résiste vraiment, on le **signale** au lieu de trancher tout seul (règle client §1 : *« fais ce qu'on te demande… au pire tu poses une question »*).

**Avant de livrer, comparer côte à côte** la maquette et le rendu email : chaque bloc de la maquette doit avoir son équivalent. Un bloc manquant = un écart à corriger, pas un choix.

---

## §0 Quand m'invoquer
Dès qu'on doit mettre **un email avec un design soigné** dans HubSpot (marketing email OU automation email), à partir d'un design existant (Claude Design, Figma, HTML reçu) ou à créer.

## §1 ⛔ LE PIÈGE — un design ≠ un HTML email
Un fichier de **Claude Design (`.dc.html`)** — ou toute page web — est un **bundle qui rend via JavaScript** (logo en SVG, `<script>`, `#__bundler_loading`). **Un email ne peut PAS exécuter de JS ni afficher de SVG** (Gmail/Outlook les suppriment). Donc :
- **NE PAS** faire « Upload design → Generate design » avec ce fichier : l'IA HubSpot **gèle ou échoue** dessus.
- **Diagnostic 10 s** : `grep`/python sur le fichier → **0 `<img>`**, présence de `<script>` / `<svg>` / `#__bundler_loading` = c'est un bundle web, pas un email. Stop, on rebâtit.

### §1bis — EXTRAIRE le vrai HTML du bundle (travailler sur le CODE, jamais un screenshot)

Le bundle est illisible d'un bloc (2 Mo, il **gèle souvent le renderer** si on l'ouvre) — mais il **contient le design complet en clair**. On ne screenshote pas, on **déplie** : la fidélité (§0.1) exige de lire les vraies couleurs, textes, tailles, marges **dans le code**, pas de les deviner à l'œil.

**Le bundle a deux `<script>` de données** :
- `<script type="__bundler/template">` = une **string JSON qui EST le HTML du design** (~1400 lignes : header, hero, footer, styles inline, `@font-face`, refs d'images en UUID).
- `<script type="__bundler/manifest">` = les **assets embarqués** (images, woff2) en base64, indexés par les mêmes UUID.

```python
import json, re, base64
raw = open('design.dc.html', encoding='utf-8').read()
tpl = re.search(r'<script type="__bundler/template">(.*?)</script>', raw, re.S).group(1)
html = json.loads(tpl)                       # <- le vrai HTML du design
open('_design.html','w',encoding='utf-8').write(html)
# une image embarquée : retrouver son UUID dans html (src="UUID"), puis dans le manifest
man = json.loads(re.search(r'<script type="__bundler/manifest">(.*?)</script>', raw, re.S).group(1))
# man[uuid] ressemble à "data:image/png;base64,...." -> décoder et sauver si besoin
```

Puis lire `_design.html` : en extraire **la palette exacte** (`re.findall(r'#[0-9a-fA-F]{6}', html)`), le **texte visible**, et le **squelette** (tags + `style=` filtré sur color/background/font/padding/border) pour reproduire chaque bloc. Les images propres au client (logo…) sont souvent **déjà hébergées au File Manager** (§3) — les réutiliser plutôt que de ré-extraire.

## §2 ✅ MÉTHODE — réécrire un HTML email-safe → "coded email"
Écrire à la main un HTML email robuste, puis le livrer en **coded email** HubSpot (Design Manager), **jamais** en drag-drop.
Checklist email-safe :
- **Tables** (`role="presentation"`) pour la mise en page, **CSS inline** (pas de classes lourdes ; `<style>` seulement pour `@media` + resets).
- **Zéro JS, zéro SVG.** Polices **email-safe** (Georgia/Arial ; les polices web ne s'embarquent pas).
- **Images HÉBERGÉES** (voir §3), jamais base64 lourd (Gmail clippe > ~102 KB).
- **Boutons bulletproof** : `<table><td bgcolor><a>` (pas de `display:flex`, ignoré en email).
- **Préheader** caché, **responsive** `@media max-width:620px` (colonnes `.stack` en `display:block`).
- **Obligatoire HubSpot/CAN-SPAM** : `{{ unsubscribe_link_all }}` + **adresse physique** de l'entreprise dans le footer, sinon publication refusée.
- Type **"Automated"** si l'email est envoyé par un workflow.
- Squelette de référence : `Downloads/PCB-mail-assets/email-hubspot-coded.html` (Pastry Chef).

## §2bis Polices script + médaillons → PNG transparents (au lieu d'un fallback qui trahit le design)

Une police décorative (Allura, Balmonte, un logo cursif) ne s'embarque pas en email, et un fallback Georgia/Arial **change le résultat visuel** (interdit §0.1). Le bon moyen email-safe : **rendre le mot/badge en PNG transparent** et le poser en `<img>`. Pareil pour un médaillon, un badge à dégradé, une icône fine (un `<td bgcolor>` rond est carré sous Outlook).

**Recette (Chrome headless + Google Fonts + trim PIL)** :
```bash
CHROME="/c/Program Files/Google/Chrome/Application/chrome.exe"
# 1 petit HTML par élément, fond transparent, la webfont via Google Fonts
"$CHROME" --headless=new --disable-gpu --hide-scrollbars \
  --force-device-scale-factor=2 --default-background-color=00000000 \
  --virtual-time-budget=4000 --window-size=400,160 \
  --screenshot="$(cygpath -w out_raw.png)" "$(cygpath -w mot.html)"
```
- `--force-device-scale-factor=2` = rendu **retina** ; dans l'email, afficher à la **moitié** de la largeur du PNG (`width` CSS = px/2) pour la netteté.
- `--default-background-color=00000000` = **transparent** (sinon fond blanc collé) ; **chemins Windows via `cygpath -w`** sinon Chrome ne produit rien.
- `--virtual-time-budget` laisse le temps à la webfont de charger — **vérifier ensuite** que ce n'est pas le fallback (ouvrir le PNG, l'œil confirme le script).
- **Trim** les bords transparents avec PIL (`im.getchannel('A').getbbox()` → `crop`) + garder ~8 px de marge, sinon le centrage est faux.

Puis **héberger au File Manager** (§3) comme toute image, et référencer en `<img>` avec `width`/`height` explicites. Garde-fou : le SVG ne s'affiche pas en email (cf skill `hubspot-crm` / §12 du doc client) → tout vectoriel décoratif passe par ce pipeline.

## §3 Images — File Manager + URL
1. Uploader dans un **dossier dédié client** du File Manager (cf règle agence « 1 dossier File Manager par client »).
2. URL publique — pattern : `https://{portalId}.fs1.hubspotusercontent-na1.net/hubfs/{portalId}/{Dossier}/{fichier}` (ex Pastry : `.../hubfs/21450353/Florent/TrustPilot/1-logo.png` — **attention au sous-dossier parent** `Florent/`).
3. ⚠️ **Redaction** : les URLs File Manager sont **masquées dans les retours d'outils Claude** (filtre sécu) — lecture JS, presse-papier, screenshot du panneau : tout bloqué. → **demander à Florent** (⋮ → *Copy file URL* → colle) OU **construire** avec le pattern ci-dessus. Ne pas s'acharner à les lire en auto.

## §4 Anti-galères Chrome MCP sur HubSpot
- **Éditeurs email drag-drop + form builder** = **iframes cross-origin** (`hubspotpreview`) → screenshots **gèlent**, JS du canvas inaccessible → **impilotables en auto**. D'où le coded email + les forms via clone/legacy.
- **Boutons d'action React** : les clics MCP (ref/coord) **passent souvent pas** → utiliser **`javascript_tool` + `.click()`** (fiable).
- **"Upload files"** ouvre un **native picker** qui **gèle le renderer** ; `file_upload` refuse les chemins disque → faire **glisser par Florent** sur la zone drop, ou **ouvrir la bonne page** et le laisser déposer (règle « ouvre la page cible »).
- **Preview** : rendre l'email en local (`python -m http.server --bind 127.0.0.1` + Chrome, ou `explorer.exe fichier.html`) et **l'ouvrir soi-même** dans le navigateur (règle « montrer = ouvrir »).

## §4bis Import réussi + réglages obligatoires (send-options)
**Ce qui MARCHE (prouvé 2026-07-06)** : *Create email → Automated → template gallery → "Upload design" → **glisser le fichier HTML clean** (email-safe, PAS le `.dc.html`) → "Generate design"* → l'IA convertit en email drag-drop **fidèle** (logo + images OK). Le collage "Paste the HTML" est dans un iframe → **impilotable en auto** ; faire **glisser le fichier** par Florent (flow qu'il maîtrise) est le plus fiable.
**Après import, HubSpot bloque sur des champs obligatoires (page send-options)** — les régler sinon l'email n'est pas valide :
- **Subscription type** (requis, anti-spam) → choisir **"Marketing Information"** pour un email marketing/automation (satisfaction post-achat, newsletter). = la catégorie de désabonnement.
- **Token `contact.firstname` sans défaut** (warning) → si le sujet/corps personnalise avec le prénom et qu'un contact n'en a pas, HubSpot veut une valeur par défaut. **Le plus simple = sujet SANS prénom** (ex « 😊 How was your recent order? »). Sinon définir un défaut.
- Vérifier **From name / From address** (ex Rachel Ibgui / `...@hubspotemail.net`) + **Subject line**.

## §5 Chaînage
- Email codé créé → **repointer/brancher** dans le workflow d'automation (ne pas recréer le workflow si un existant fait le job — cf `revops-workflows`).
- Boutons de l'email → cibles (landing page, form → ticket) : voir `hubspot-crm` / forms.
- Toujours §1 consentement client (Clients/CLAUDE.md) avant toute mutation prod.
