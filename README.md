# HubSpot CRM Toolkit — plugin Claude Code

Une boîte à outils de compétences Claude Code pour **auditer, diagnostiquer et piloter un portail HubSpot** — pensée pour les agences et les RevOps qui vivent dans le CRM d'un client.

Sa particularité : elle est bâtie sur un principe d'**arbitrage de tokens**. Au lieu que Claude fasse tout le travail lui-même (cher), il **délègue au cerveau le moins cher qui sait le faire** — à commencer par **Breeze, l'IA native de HubSpot** (gratuite dans le portail) — et garde son énergie pour ce que lui seul peut faire : le local et la synthèse à forte valeur.

## Le principe — déléguer au cerveau le moins cher

`claude-ia-delegation` est le **chapeau** : devant une recherche ou un diagnostic, il choisit le cerveau le moins coûteux, puis délègue.

- Contexte HubSpot → **Breeze** (l'IA du portail, crédits gratuits) ou un audit écran-par-écran.
- Recherche générale → GPT / Gemini (crédits déjà payés).
- Le local et la valeur ajoutée → Claude directement.

Le résultat : moins de tokens brûlés, on exploite les crédits qu'on paie déjà ailleurs.

## Les compétences

| Compétence | Ce qu'elle fait |
|---|---|
| **claude-ia-delegation** | Le chapeau — route chaque tâche vers le cerveau le moins cher. |
| **hubspot-breeze-audit** | Investiguer une question HubSpot au moindre coût : mode Breeze (demander à l'IA native) OU audit écran-par-écran des fonctionnalités sans API (Buyer Intent, Lead Scoring, Permissions…). |
| **hubspot-crm** | Le socle : contexte portail, catalogue des outils MCP HubSpot, protocole d'investigation, table de décision Breeze / MCP / navigation. |
| **hubspot-segments-audit** | Cartographie + audit de tous les segments/listes via API (filtres réels, volumes, santé). |
| **hubspot-workflows-audit** | Cartographie + audit de tous les workflows/automatisations via API. |
| **hubspot-marketing-segments** | Créer et gérer les listes/segments marketing. |
| **hubspot-create-list** | Créer proprement une liste HubSpot. |
| **hubspot-email-design** | Concevoir des emails HubSpot (contraintes de rendu, images, footer conforme). |
| **crm-investigation-output** | Formater le rendu d'une investigation : verdict, email client, infographie. |

## Installation

Dans Claude Code :

```
/plugin marketplace add Gremelinn0/hubspot-crm-toolkit
/plugin install hubspot-crm-toolkit
```

Puis redémarrer Claude Code. Les compétences deviennent invocables (ex : « audit Buyer Intent », « demande à Breeze pourquoi ce workflow n'enrôle plus »).

## Setup (optionnel)

**La méthode est portée par l'agent** (le skill `hubspot-breeze-audit`), pas par une config HubSpot — rien n'est obligatoire côté portail pour que le toolkit marche quand c'est Claude qui pilote. L'agent travaille dans **tes projets Breeze existants**, change les fenêtres via Chrome MCP pour nourrir Breeze écran par écran, et laisse Breeze produire l'analyse.

Si tu veux que **Breeze tienne la discipline même sans agent** (un humain qui pilote à la main, ou une autre IA sans le skill), garde le prompt `skills/hubspot-breeze-audit/references/navigation-assistee-prompt.md` dans ta **bibliothèque de prompts partagés HubSpot** (ou dans les instructions d'un projet Breeze). HubSpot ne permet pas d'instructions au niveau du compte, seulement au niveau projet — d'où le prompt partagé plutôt qu'une config globale.

## Philosophie

Un CRM cache sa vérité sur plusieurs écrans, et beaucoup de plateformes exposent une IA gratuite qu'on n'utilise pas. Ce toolkit part de là : **regarder au bon endroit, et faire réfléchir la bonne IA** — pas toujours la plus chère.

---

*Auteur : Florent Maisoncelle. Compétences extraites d'un usage réel en agence sur des portails HubSpot clients.*
