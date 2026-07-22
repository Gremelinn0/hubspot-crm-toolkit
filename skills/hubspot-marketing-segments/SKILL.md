---
name: hubspot-marketing-segments
description: Construit des listes/segments de CONTACTS HubSpot pour newsletters / emails marketing à partir d'un ciblage entreprise (secteur, liste company). Applique la recette « email marketing = contacts only » → miroir des filtres sur la SOCIÉTÉ ASSOCIÉE + Email known + Marketing contact status. Création via API v3 Lists + Private App token (le MCP ne sait pas créer de listes, l'UI en auto fige HubSpot). Triggers "liste contacts pour newsletter", "segment marketing", "newsletter au secteur X", "convertir liste entreprise en liste contact", "miroir contact d'une liste company", "audience marketing".
---

# Skill — HubSpot Marketing Segments (listes contacts pour newsletters)

## §0 Fondamental + quand utiliser

**Règle HubSpot non négociable** : un **email marketing / newsletter s'envoie à des CONTACTS, jamais à des entreprises.** Donc pour cibler « les hôtels / restaurants / casinos », on ne peut PAS envoyer sur une liste d'entreprises → il faut une **liste de contacts** dont la **société associée** matche le secteur.

**Quand l'utiliser** : Florent (ou la cliente) veut envoyer une newsletter / campagne email à un secteur, et la cible existe en **liste d'entreprises** (companies). Ce skill crée la **liste contacts miroir** prête pour Marketing Emails → Send to.

---

## §1 La recette de conversion (cœur)

Pour chaque liste **entreprise** source → 1 segment **contact** miroir :

| Filtre de la liste entreprise | Devient (dans la liste contact) |
|---|---|
| `Industry = X` | `Contact > société associée > Industry is any of X` |
| `Company name contains "hotel"` | `Contact > société associée > Company name contains any of hotel` |
| `Create date is after JJ/MM/AAAA` | `Contact > société associée > Create date is after JJ/MM/AAAA` |
| *(ajout systématique)* | `Contact > Email is known` |
| *(ajout systématique)* | `Contact > Marketing contact status is any of Marketing contact` |

Type = **Active (dynamique)** par défaut (cf CLAUDE.md projet — statique seulement si Florent précise). Les 2 garde-fous contact garantissent l'éligibilité à l'envoi marketing.

---

## §2 Méthode de création = API v3 Lists + Private App token

**Le MCP HubSpot NE crée PAS de listes** (`OBJECT_LIST write = NOT_AVAILABLE`, aucun outil de création) — reconnecter le MCP n'y change rien. **L'automation de l'UI** (Chrome MCP) **fige HubSpot ~45s par interaction** (builder de segments très lourd) → inexploitable.

→ **Voie unique fiable : API v3 Lists avec un Private App token.**

- Endpoint : `POST https://api.hubapi.com/crm/v3/lists` (header `Authorization: Bearer <token>`).
- Token = Private App (Réglages → Intégrations → Private Apps), scopes `crm.lists.read` + `crm.lists.write` (+ `crm.objects.contacts.read` / `crm.objects.companies.read` pour vérifs). Stocké en **fichier gitignored** (`*.local` ou `.env`), JAMAIS commité, JAMAIS réaffiché.
- Lire une liste existante : `GET /crm/v3/lists/{id}?includeFilters=true` (donne la syntaxe exacte des filtres + permet de copier un modèle).
- Trouver le bon `listFolderId` : `GET /crm/v3/lists/folders` (arbre des dossiers) ou `POST /crm/v3/lists/search` (renvoie `hs_folder_name`).

---

## §3 filterBranch exact (à copier)

Structure : `OR > AND { filtres contact + branche ASSOCIATION vers la société }`. La branche **ASSOCIATION** porte les filtres entreprise **directement dans son `filters`**.

```json
{
  "name": "Main list Casinos 2026+",
  "objectTypeId": "0-1",
  "processingType": "DYNAMIC",
  "listFolderId": 241094031,
  "filterBranch": {
    "filterBranchType": "OR", "filterBranchOperator": "OR", "filters": [],
    "filterBranches": [{
      "filterBranchType": "AND", "filterBranchOperator": "AND",
      "filters": [
        {"property":"email","operation":{"operator":"IS_KNOWN","includeObjectsWithNoValueSet":false,"operationType":"ALL_PROPERTY"},"filterType":"PROPERTY"},
        {"property":"hs_marketable_status","operation":{"operator":"IS_ANY_OF","includeObjectsWithNoValueSet":false,"values":["true"],"operationType":"ENUMERATION"},"filterType":"PROPERTY"}
      ],
      "filterBranches": [{
        "filterBranchType": "ASSOCIATION", "filterBranchOperator": "AND",
        "objectTypeId": "0-2", "operator": "IN_LIST",
        "associationTypeId": 279, "associationCategory": "HUBSPOT_DEFINED",
        "filterBranches": [],
        "filters": [
          {"property":"industry","operation":{"operator":"IS_ANY_OF","includeObjectsWithNoValueSet":false,"values":["GAMBLING_CASINOS"],"operationType":"ENUMERATION"},"filterType":"PROPERTY"},
          {"property":"createdate","operation":{"operator":"IS_AFTER","includeObjectsWithNoValueSet":false,"timePoint":{"timezoneSource":"CUSTOM","zoneId":"US/Eastern","year":2025,"month":12,"day":31,"hour":23,"minute":59,"second":59,"millisecond":999,"timeType":"DATE"},"endpointBehavior":"EXCLUSIVE","propertyParser":"VALUE","type":"TIME_POINT","operationType":"TIME_POINT"},"filterType":"PROPERTY"}
        ]
      }]
    }]
  }
}
```

- **`associationTypeId: 279`** (HUBSPOT_DEFINED) = « toute société associée » au contact (contact 0-1 → company 0-2). `1` = société **primaire** seulement. Vérifier les ids du portail via `GET /crm/v4/associations/0-1/0-2/labels`.
- `industry` est un **enum** : utiliser la **valeur interne** (`GAMBLING_CASINOS`, `RESTAURANTS`, `HOSPITALITY`…), pas le label. Récupérer via `get_properties(companies,["industry"])` (MCP) ou la doc.
- Filtre `name contains` (ex Hotels) : `{"property":"name","operation":{"operator":"CONTAINS","values":["hotel"],"operationType":"MULTISTRING","includeObjectsWithNoValueSet":false},"filterType":"PROPERTY"}` à ajouter dans le `filters` de la branche ASSOCIATION.

---

## §4 Convention nommage / dossier (CLAUDE.md projet)

- **Même NOM** que la liste entreprise source (le type d'objet Contact vs Company suffit à distinguer). Ex : la liste contact s'appelle aussi `Main list Casinos 2026+`.
- **Même DOSSIER** que la liste entreprise (`listFolderId` identique). Un **dossier mixte** companies + contacts est OK.
- **Dynamique** par défaut (`processingType: DYNAMIC`). Statique (`SNAPSHOT`/`MANUAL`) seulement si Florent précise.

---

## §5 Script réutilisable + workflow

Modèle : `memory/clients/pastry-chefs-boutique/create_lists.local` (Python, lit le token depuis `secrets.local`, builders de filtres, configs par secteur, POST + GET de vérif). Réutilisable pour de nouveaux secteurs : ajouter une entrée dans `SECTORS` (nom, `industry` interne, `name_contains` optionnel) puis `python create_lists.local <clé>`.

**Workflow** :
1. Identifier la/les liste(s) entreprise source + lire ses filtres (`GET ...?includeFilters=true`).
2. Récupérer le `listFolderId` de la source (même dossier) + l'`associationTypeId` contact→company.
3. Construire le payload (recette §3), `POST /crm/v3/lists`.
4. **Vérifier** : `GET /crm/v3/lists/{id}?includeFilters=true` → objet `0-1`, `DYNAMIC`, `processingStatus COMPLETE`, `size` cohérent vs la liste company, filtres bien stockés.
5. **Post-create** §6.

---

## §6 Post-create (obligatoire)

- `listes-inventory.md` du client : ajouter les listes (ID/nom/vol/miroir/URL) + ligne(s) §15 changelog. *(Règle absolue listes — CLAUDE.md projet)*.
- `actions-log.md` : entrée datée.
- Si la création change le snapshot du `CLAUDE.md` client → maj le pointeur.

---

## §7 Cas inaugural — Pastry Chef's Boutique (2026-06-12)

4 listes contacts créées en miroir des Main lists 2026+ companies (627-630), même nom + même dossier (241094031), via API + token `Claude Lists API` :

| Contact | ID | ← Company | Vol |
|---|---|---|---|
| Main list Casinos 2026+ | 632 | 630 | 1 |
| Main list - Restaurants 2026+ | 634 | 629 | 0 |
| Main list Hotels 2026+ | 636 | 628 | 1 |
| Main list Hospitality 2026+ | 638 | 627 | 33 |

Pattern validé. Pour le déroulé en volume (autres secteurs, listes « tout secteur » sans filtre date) : même recette, nouveaux dossiers propres.

---

## §8 Anti-patterns

- ❌ Envoyer une newsletter sur une liste d'**entreprises** (impossible — contacts only).
- ❌ Tenter de créer la liste via le **MCP** (`OBJECT_LIST write` indispo) ou via l'**automation UI** (fige 45s/clic).
- ❌ Mettre le **label** d'industrie au lieu de la valeur enum interne.
- ❌ Committer / réafficher le **token** (toujours gitignored).
- ❌ Oublier le post-create (`listes-inventory.md` + `actions-log.md`).
- ❌ Renommer / changer de dossier le miroir : il garde **même nom + même dossier** que la source.

---

## Doc client de référence (source canonique)
**Pastry Chef's Boutique** → `memory/clients/pastry-chefs-boutique/segments-et-campagnes.md` = modèle entreprise→contact, référentiel géographique (`east___west_coast` + options + remplissage mesuré), cartographie complète des 74 segments par dossier, gouvernance. **Lire ce doc avant tout travail segments/campagnes sur ce client.** (Charger aussi `listes-inventory.md` + `crm-state.md`.)

## Skills liés
`hubspot-create-list` (flow UI générique, à éviter sur ce portail car fige), `hubspot-crm` (règles CRM globales), `revops-api-scripts` (scripts API), `revops-data-model` (propriétés).
