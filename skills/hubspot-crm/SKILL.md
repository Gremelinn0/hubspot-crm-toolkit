---
name: hubspot-crm
description: >-
  Skill CENTRAL HubSpot — à charger en premier à toute session HubSpot (règle CLAUDE.md agence). Porte les règles socles — doc Notion client à lire AVANT toute action HubSpot (ciblage et cahier des charges = source de vérité métier, HubSpot = exécution) · Breeze (IA native HubSpot) obligatoire avant tout diagnostic incertain ou action irréversible · convention naming et folder = s'inspirer de l'existant, jamais improviser · catalogue complet des outils MCP HubSpot (search_crm_objects, manage_crm_objects, propriétés, owners, campagnes) · protocole d'investigation CRM en 3 étapes (source via MCP et Breeze, état intégration, source des nouveaux contacts) · table de décision Breeze vs MCP vs Computer-Use · URLs de navigation clés du portail · repos GitHub dev liés (hubspot-cli, sample-apps-nodejs, dbt_hubspot). Activer quand Florent glisse un cahier des charges ou brief client, pour toute investigation ou action dans HubSpot, toute question sur le fonctionnement du CRM, ou tout sujet enrichissement/import/automatisation/pipeline/contact/deal. Triggers "hubspot", "crm client", "brief client hubspot", "action hubspot", "portail hubspot", "pourquoi ce contact ne marche pas hubspot", "comment fonctionne x dans hubspot".
---

# Skill — HubSpot CRM Agent

> 📇 **Avant de créer un nouveau projet/chat Breeze** (ou tout autre projet AI plateforme, ex Claude Design) → checker le registre `memory/plateformes-ai-registry.md` : un projet existant peut déjà couvrir le besoin, pas de raison d'en recréer un.

## Déclencheur

Activer ce skill quand :
- Florent glisse un cahier des charges / brief client
- Toute investigation ou action dans HubSpot
- Toute question sur le fonctionnement du CRM
- Enrichissement, import, automatisation, pipeline, contact, deal

---

## §0 RÈGLE ABSOLUE — Doc Notion client d'abord, puis partir de l'existant HubSpot (gravée 2026-05-04, mise à jour 2026-05-04 v2)

**Avant TOUTE action HubSpot (création liste, segment, vue, workflow, propriété, ciblage, recommandation)** :

### Étape 0 — Lire la doc Notion client D'ABORD (obligatoire)

- Page parente client dans Notion = source officielle du **cahier des charges + ciblage + décisions stratégiques**
- Pour Pastry Chef : `https://www.notion.so/prospectpartner/CRM-Marketing-15001e69443c80338050f70d4f6c188c` (page CRM & Marketing) + page racine `27501e69-443c-8015-a4cc-f912ec6286e9`
- **Toujours commencer par la doc Notion**, pas par les MCP / l'UI HubSpot
- Si nouveau client → demander à Florent l'URL Notion parente client AVANT toute action

### Étape 1 — Audit existant HubSpot

Après doc Notion lue :

1. **Inventorier** ce qui existe déjà (listes, segments, vues, workflows pertinents)
2. **Filtrer par créateur Florent** = compte avec email `florent.maisoncelle@gmail.com` (souvent affiché "Rachel Ibgui" dans portails clients) — c'est ce que Florent maîtrise et dont il est responsable
3. **Écarter les anciennes** : créateurs externes / inconnus / collaborateurs anciens (Arthur Pomiès, Valentin Lerouge, Ludwig Meert, Maxime Blaszka, "Unknown user", "Pastry Chef's Boutique n/a" type comptes techniques)
4. **Vérifier paramétrage** : pour chaque liste candidate → ouvrir page `/objectLists/{ID}/filters` → lire critères réels → catégoriser :
   - Vraie dynamique (filtre métier `industry = X`, `lifecycle = Y`)
   - Statique-déguisée (filtre = `is in import X` → équivalent figé)
   - Vraie statique (Type "Static" explicite)
5. **Si une liste existante peut servir** → l'utiliser / cloner / adapter (ajouter filtre supplémentaire)
6. **Si aucune ne convient** → créer nouvelle (validation Florent obligatoire)

### Pourquoi doc Notion d'abord

- Florent maintient le ciblage / cahier des charges client dans Notion (PAS dans HubSpot)
- HubSpot = exécution. Notion = source vérité métier.
- Sans lire la doc Notion → risque de proposer des actions hors cahier des charges, créer du bruit, ou rater des décisions déjà prises
- Verbatim Florent 2026-05-04 : *"tu es obligé de te baser sur la documentation quoi. Obligé, obligé."*

### Étape 2 — Naming + folder = INSPIRATION existant (gravée 2026-05-04 v3)

**Avant de créer une nouvelle liste/segment** :

1. **Naming convention** : s'inspirer des noms existants similaires. Si "Main list Hospitality 1" existe → nouvelle = "Main list Hospitality 2026+" (PAS "Nouveaux comptes Hospitality 2026+" en variation gratuite). Garde le préfixe / format métier établi.
2. **Langue** : si listes existantes sont en français ("Hôtellerie", "Casinos") → naming français. Si anglais ("Main list Hospitality") → anglais. PAS mélanger.
3. **Folder destination** : suivre la **logique métier**, pas la facilité technique. Vérifier où sont placées les listes du même type métier (ex : Main lists prospection → folder "Clients" même si sources d'inspiration sont dans "Cibles"). La décision métier prime sur le folder technique de la source clonée.
4. **Vérifier folder réel des listes existantes via UI** (breadcrumb), pas supposer

**Anti-pattern** :
- ❌ Inventer naming différent de l'existant ("Nouveaux comptes X" au lieu de "Main list X")
- ❌ Mettre dans folder source d'inspiration au lieu du folder métier
- ❌ Mélanger français/anglais
- ❌ Ne pas vérifier breadcrumb folder réel

**Verbatim Florent 2026-05-04 v3** : *"il faut toujours s'inspirer des noms, ça c'est la base. Et le fichier il faut le mettre dans les mêmes dossiers."*

### Pourquoi

- Les portails clients accumulent des dizaines/centaines de segments hérités
- Recréer = duplication, perte de temps, confusion client
- Partir de l'existant = gain temps + cohérence avec décisions passées
- Filtre créateur Florent = sait qu'il en est responsable, n'écrase pas le travail d'autres collaborateurs/équipes

### Format présentation à Florent

Avant toute proposition de création :

```
Listes existantes pertinentes (créées par toi) :
- [Nom liste] (ID, vol, type) — filtres : [...] — verdict : utilisable telle quelle / à cloner / pas adapté

Si à cloner : clone "<nom>" + ajouter filtre <X> → résultat liste "<nom>_<contexte>"
Si rien adapté : créer nouvelle "<nom proposé>" avec filtres <Y>

→ Confirme approche ?
```

### Anti-pattern

- ❌ Créer une nouvelle liste sans avoir inventorié l'existant
- ❌ Inventorier mais ignorer le filtre créateur
- ❌ Présenter solution "créer X" sans option "cloner Y existant"

---

## §1 RÈGLE ABSOLUE — Breeze en premier

**Breeze = IA native HubSpot. TOUJOURS consulter AVANT d'agir sur comportement CRM inconnu.**

Workflow :
1. Formuler la question / hypothèse
2. Poser à Breeze (bouton ✦ Assistant en haut à droite HubSpot)
3. Valider réponse → PUIS agir

**Breeze obligatoire pour :**
- Comportement inconnu d'une propriété, d'une app, d'une automatisation
- **Quand Claude doute de son analyse** → valider avec Breeze AVANT de déployer la solution
- "Pourquoi ce contact n'a pas de téléphone ?" → Breeze d'abord
- "Pourquoi ce workflow n'a pas déclenché ?" → Breeze d'abord
- "Quelle app gère les imports ?" → Breeze d'abord
- "Comment fonctionne X dans ce portail ?" → Breeze d'abord
- Tout diagnostic avant d'exécuter une action irréversible (suppression, modification masse)

**Breeze facultatif (MCP direct OK) pour :**
- Lecture de données : contacts, deals, propriétés, campagnes
- Recherche / filtres CRM
- Diagnostics depuis données structurées (pas d'interprétation HubSpot needed)

Ne jamais supposer le comportement CRM sans validation Breeze quand comportement incertain.

### Capacités Breeze confirmées (portail 21450353)

Portail : **Sales Hub Pro + Marketing Hub Pro + Service Hub Starter** / Super Admin

| Domaine | Ce que Breeze peut faire |
|---------|-------------------------|
| **Contacts / Entreprises** | Chercher, retrouver, analyser, associations entre objets |
| **Enrichissement** | Investiguer champs manquants, proposer stratégie, distinguer sources — *ne peut pas dire quelle source exacte est active sans vérifier apps connectées* |
| **Deals** | Analyser ouverts/gagnés/perdus/stagnants, filtrer, identifier deals à risque |
| **Workflows** | Concevoir, expliquer triggers/branches/délais, auditer logique, créer dans certains cas |
| **Investigation** | ⭐ Meilleur usage — diagnostic pourquoi donnée manque, record pas mis à jour, workflow pas déclenché, résultat inattendu |

### Breeze seul vs Breeze + toi

**Breeze peut faire seul :**
- Chercher / retrouver des records
- Analyser données CRM, résumer situations
- Proposer logique d'automatisation
- Créer certains éléments (workflows, rapports simples)
- Rédiger emails / follow-ups / contenus CRM

**Breeze fait AVEC toi (pas seul) :**
- Arbitrer la bonne structure CRM
- Prioriser champs et automatisations
- Investiguer cas métier ambigus
- Expliquer impacts d'un changement avant exécution

**Limites Breeze (nécessite vérif manuelle) :**
- Enrichissement natif disponible → vérifier apps connectées
- Statut sync apps tierces → ouvrir URL intégrations
- Fonctionnalités très spécifiques liées au plan/seat
- Limites exactes du portail

### Exemples prompts Breeze efficaces

```
# Contacts
"Trouve les contacts sans owner"
"Analyse les doublons potentiels"
"Montre-moi les entreprises associées à ce contact"
"Quelles propriétés utilise-t-on pour qualifier les leads ?"

# Deals
"Quels deals sont inactifs depuis 21 jours ?"
"Résume-moi ce pipeline"
"Quels deals ont un montant mais pas de date de closing ?"
"Quels commerciaux n'ont aucune activité sur leurs deals ouverts ?"

# Investigation / diagnostic
"Pourquoi ce workflow n'inscrit personne ?"
"Diagnostique pourquoi ce workflow ne marche pas"
"Pourquoi ce contact n'a pas été mis à jour ?"
"Quelle source a enrichi ce contact ?"

# Enrichissement
"Quels champs devrions-nous enrichir en priorité ?"
"Aide-moi à structurer un process d'enrichissement"
"Quels contacts ont un email mais pas de téléphone ?"

# Automatisations
"Crée un workflow pour assigner les leads entrants par pays"
"Conçois un workflow de réassignation"
"Je veux une alerte si un deal reste 14 jours sans activité"
"Automatise la mise à jour du lifecycle stage"

# Audit CRM
"Fais-moi un audit rapide de la qualité des données"
"Aide-moi à comprendre pourquoi mes équipes ne suivent pas le process"
```

---

## §2 MCP HubSpot (accès programmatique)

Config dans `navigateur/.claude/settings.json` :
```json
{
  "mcpServers": {
    "hubspot": {
      "command": "npx",
      "args": ["-y", "mcp-hubspot"],
      "env": {
        "HUBSPOT_ACCESS_TOKEN": "<TOKEN_PRIVATE_APP>"
      }
    }
  }
}
```

Portail actif : **21450353** (Pastry Chef's Boutique)

Scopes Private App requis :
- `crm.objects.contacts.read` / `.write`
- `crm.objects.companies.read` / `.write`
- `crm.objects.deals.read` / `.write`
- `crm.import.read`
- `content` (campagnes)

### Catalog outils MCP disponibles

| Outil | Usage |
|-------|-------|
| `search_crm_objects` | Chercher contacts/deals/companies avec filtres |
| `get_crm_objects` | Récupérer objets CRM par ID |
| `manage_crm_objects` | Créer / modifier / supprimer objets CRM |
| `get_properties` | Lister toutes les propriétés d'un type d'objet |
| `search_properties` | Chercher une propriété par nom ou label |
| `get_organization_details` | Infos portail (nom, timezone, currency) |
| `get_user_details` | Infos user connecté |
| `search_owners` | Trouver un owner par nom/email |
| `get_campaign_analytics` | Métriques campagne (opens, clicks, etc.) |
| `get_campaign_asset_metrics` | Métriques par asset de campagne |
| `get_campaign_asset_types` | Types d'assets disponibles |
| `get_campaign_contacts_by_type` | Contacts liés à une campagne |
| `tool_guidance` | Auto-documentation — demander à MCP comment utiliser un outil |

**Tip** : appeler `tool_guidance` en premier si incertain sur paramètres d'un outil.

---

## §3 Workflow Brief Client → Exécution

### Intake (brief entrant)

Quand Florent glisse un fichier brief/cahier des charges :

1. **Lire le brief** → identifier : client, objectif, actions HubSpot demandées
2. **Créer/mettre à jour** `memory/clients/<nom-client>/brief.md`
3. **Consulter Breeze** pour valider l'approche si comportement CRM incertain
4. **Exécuter** via MCP HubSpot ou computer-use selon l'action
5. **Logger le résultat** dans `memory/clients/<nom-client>/actions-log.md`
6. **Mettre à jour** `crm-state.md` si état CRM change

### Structure mémoire client

```
memory/clients/<nom-client>/
├── brief.md          ← Cahier des charges / objectifs
├── crm-state.md      ← État actuel CRM (contacts, deals, apps connectées)
└── actions-log.md    ← Historique actions exécutées + résultats + dates
```

### Règle : 1 info = 1 endroit

- Spec client → `brief.md`
- État CRM → `crm-state.md`
- Ce qui a été fait → `actions-log.md`
- Jamais dupliquer entre les 3

---

## §4 Protocol Investigation CRM

Quand un problème est signalé (ex: enrichissement cassé, contact sans données, import raté) :

### Étape 1 — Identifier la source (MCP + Breeze)

**Via MCP :**
```
search_crm_objects(objectType="contacts", query="<nom contact>")
→ récupérer ID contact

get_crm_objects(objectType="contacts", objectId="<id>", properties=["phone","email","hs_analytics_source","notes_last_updated"])
→ voir valeurs actuelles + quelle source a rempli
```

**Via Breeze :**
- Ouvrir fiche contact → cliquer "voir historique" sur propriété `phone` ou `email`
- Historique liste : qui a modifié, quand, valeur avant/après
- Source identifiable : "Dropcontact", "Kaspr", "Apollo", "Make", "Import", "Manuel"

**Sources d'enrichissement courantes :**
| Source | Signe dans historique |
|--------|----------------------|
| Dropcontact | `Dropcontact` |
| Kaspr | `Kaspr` |
| Apollo | `Apollo.io` |
| Make / Zapier | `Integration` ou nom du scénario |
| Import manuel | `Import` |
| HubSpot Insights | `HubSpot` |

### Étape 2 — État intégration

**Via URL directe :**
- `https://app.hubspot.com/integrations-settings/21450353/installed`
- Badge vert = connecté / rouge = erreur / absent = désinstallé

**Via MCP :** pas d'outil direct pour statut intégration → computer-use ou URL.

**Si Make/Zapier :** ouvrir plateforme concernée et vérifier scénario/zap associé (actif ? erreur récente ?).

### Étape 3 — Source des nouveaux contacts

**Via URL :**
- `https://app.hubspot.com/contacts/21450353/import` → historique imports
- Cliquer import concerné → voir mapping colonnes

**Via MCP :**
```
search_crm_objects(objectType="contacts", filters=[{"propertyName":"hs_analytics_source","operator":"EQ","value":"IMPORT"}])
→ contacts importés récemment
```

**Vérifier :**
- Email mappé correctement ? (colonne "Email" → propriété `email`)
- Contacts importés ont-ils un email ?
- Si pas d'email → aucun outil d'enrichissement ne peut travailler

### Livrable obligatoire

```
## Rapport Investigation — [Date]
- Contact / scope analysé : [nom ou segment]
- Outil enrichissement identifié : [nom ou "inconnu"]
- État actuel : [Connecté / Déconnecté / Erreur / Jamais installé]
- Cause blocage : [email manquant / mauvais mapping / outil cassé / quota dépassé]
- Recommandation : [action concrète + qui doit la faire]
```

---

## §5 Navigation HubSpot — URLs clés

| Page | URL |
|------|-----|
| Contacts | `https://app.hubspot.com/contacts/21450353/contacts` |
| Deals | `https://app.hubspot.com/contacts/21450353/deals` |
| Apps connectées | `https://app.hubspot.com/connected-apps/21450353/installed` |
| Paramètres intégrations | `https://app.hubspot.com/integrations-settings/21450353/installed` |
| Historique imports | `https://app.hubspot.com/contacts/21450353/import` |
| Automatisations | `https://app.hubspot.com/workflows/21450353` |
| Private Apps | `https://app.hubspot.com/private-apps/21450353` |
| Propriétés contacts | `https://app.hubspot.com/property-settings/21450353/contact` |

---

## §6 Clients actifs

| Client | Dossier mémoire | Portail HubSpot |
|--------|----------------|-----------------|
| Pastry Chef's Boutique | `memory/clients/pastry-chefs-boutique/` | 21450353 |

→ Ajouter nouveaux clients ici au fur et à mesure.

---

## §7 Règle de décision — Breeze vs MCP vs Computer-Use

| Besoin | Outil |
|--------|-------|
| Comprendre comportement HubSpot (pourquoi X?) | **Breeze** |
| Lire des données CRM (contacts, deals, props) | **MCP** |
| Modifier des données CRM | **MCP** (`manage_crm_objects`) |
| Voir historique propriété d'un contact | **Computer-Use** (UI HubSpot) |
| Vérifier statut intégration/app | **Computer-Use** (URL directe) |
| Diagnostiquer scénario Make/Zapier | **Computer-Use** (ouvrir Make/Zapier) |
| Chercher contacts par critère | **MCP** (`search_crm_objects`) |
| Analyser campagne email | **MCP** (`get_campaign_analytics`) |
| Action inconnue → valider approche | **Breeze** d'abord, puis MCP/CU |
| Diagnostic / analyse / « pourquoi X » sur la data du portail | **`claude-breeze`** (mode Breeze — gratuit) |
| Fonctionnalité éparpillée sur plusieurs écrans, pas d'API propre (Buyer Intent, Lead Scoring, Permissions, Record customization…) | **`claude-breeze`** (mode navigation — audit écran par écran) |
| Coder module/theme/serverless local | **hubspot-cli** |
| Exemple OAuth/webhook Node.js | **sample-apps-nodejs** |
| Transformer données HubSpot → BI | **dbt_hubspot** (Fivetran) |

---

## §8 Outils Dev — Repos GitHub clés

| Repo | Usage | Quand |
|------|-------|-------|
| [`hubspot/hubspot-cli`](https://github.com/HubSpot/hubspot-cli) | Déployer themes/modules/serverless depuis local via `hs watch` | Dev de modules custom, CI/CD, serverless functions |
| [`hubspot/sample-apps-nodejs`](https://github.com/HubSpot/sample-apps-nodejs) | Exemples complets OAuth2, webhooks, rate limits avec `@hubspot/api-client` | Base de code pour toute intégration Node.js → voir skill `revops-api-scripts` |
| [`fivetran/dbt_hubspot`](https://github.com/fivetran/dbt_hubspot) | Modèles SQL pré-écrits pour transformer données brutes HubSpot → tables BI propres | Quand HubSpot data part dans BigQuery/Snowflake |

**CLI install :** `npm install -g @hubspot/cli` → `hs auth` → `hs watch <local> <hubspot>`
