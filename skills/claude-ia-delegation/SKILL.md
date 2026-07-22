---
name: claude-ia-delegation
description: >-
  L'ORCHESTRATEUR d'arbitrage de tokens : au lieu que Claude fasse tout le travail lui-même, router chaque recherche / diagnostic / investigation vers le CERVEAU LE MOINS CHER qui sait la faire, puis garder Claude pour la valeur ajoutée et le local qu'il est seul à pouvoir exécuter. Compétence CHAPEAU (mince) qui DÉCIDE selon le contexte et DÉLÈGUE à une compétence plateforme — elle ne ré-implémente jamais la méthode d'une plateforme. Routage : contexte HubSpot → `hubspot-breeze-audit` (Breeze gratuit + audit écran-par-écran) ; recherche générale offloadable → GPT / Gemini (crédits déjà payés) ; une autre session a déjà la connexion/le contexte → on lui envoie le travail ; seul Claude peut le faire (fichiers locaux, connexions, synthèse fine) → il le fait. À invoquer dès qu'une tâche de recherche/analyse pourrait être déléguée pour économiser les tokens, sur "délègue ça", "au lieu de tout faire toi-même", "économise mes tokens", "utilise Breeze / GPT / Gemini", "arbitrage de tokens", "fais bosser une autre plateforme / une autre session", "cerveau le moins cher", ou tout diagnostic/recherche où faire tout soi-même serait du gaspillage alors qu'une IA moins chère peut répondre.
---

# Skill — claude-ia-delegation (déléguer au cerveau le moins cher)

## §0 Le principe — arbitrage de tokens

Au lieu que Claude fasse **tout** le travail lui-même (cher en tokens), router chaque tâche vers **le cerveau le moins cher qui sait la faire** — les IA natives des plateformes (crédits gratuits/déjà payés), GPT/Gemini, ou une autre session déjà connectée — et garder Claude pour ce qu'il est **seul** à pouvoir faire : le local (fichiers, connexions) et la synthèse à forte valeur ajoutée.

But = **efficience token + coût** : exploiter les crédits qu'on paie déjà ailleurs, ne pas cramer les tokens Claude sur ce qu'une IA moins chère fait aussi bien.

## §1 Le décideur — choisir le cerveau (le moins cher d'abord)

> 🎯 **Argument optionnel — forcer la plateforme.** On peut préciser la plateforme en suffixe, comme le mode de caveman (`/caveman ultra`) : ex **`/claude-ia-delegation hubspot`** force la branche HubSpot. **Totalement optionnel.** L'utilisateur le précise, OU le contexte de la conversation le rend clair → on prend cette plateforme. Sinon → n'importe laquelle, on s'en fout, **on ne demande RIEN** (cf la règle plus bas).

Avant de se lancer dans une recherche/analyse, une question : **quel cerveau, le moins cher, peut la faire ?** Descendre l'échelle, s'arrêter au premier qui suffit :

| La tâche… | Cerveau | Comment |
|---|---|---|
| porte sur une plateforme avec une IA native + crédits (HubSpot…) | **l'IA de la plateforme** | via sa compétence plateforme (§2) |
| = recherche / raisonnement général offloadable | **GPT / Gemini** | crédits déjà payés (§2) |
| a besoin d'une connexion/contexte qu'une **autre session** a déjà (ex Chrome MCP mort ici mais vivant ailleurs) | **une autre session** | `send_message` / `spawn_task` (cf `/sessions`, CLAUDE-GLOBAL §8/§27) |
| = local, connexions, synthèse fine, jugement | **Claude directement** | c'est la valeur ajoutée, on la garde |

**Si le bon cerveau n'est pas clair → trancher SEUL selon la situation. JAMAIS demander à l'utilisateur « sur quelle plateforme veux-tu que je travaille ? »** — on ouvre la session, on pose la question à l'agent de la plateforme, on récupère sa réponse, et on la ferme. Pas de temps perdu en questions. On ne délègue pas ce qui perdrait à l'être (données sensibles, jugement fin, décision produit) ; on ne fait pas soi-même ce qu'un cerveau gratuit fait aussi bien.

## §2 Les branches (routage — le chapeau POINTE, il ne fait pas)

- **HubSpot** → **`hubspot-breeze-audit`** (projet Clients) : mode Breeze (IA native, gratuit) pour un diagnostic, mode navigation écran-par-écran pour une fonctionnalité sans API. La méthode HubSpot vit là-bas, pas ici.
- **Recherche générale** → **GPT / Gemini** : principe posé, mécanique à câbler quand on l'utilise vraiment (leur UI via Chrome MCP, ou un MCP dédié si connecté). Ne pas gonfler ce skill avec cette mécanique avant de l'avoir.
- **Autre session** → router le travail à la session qui a la connexion/le contexte (`send_message`), plutôt que de tout refaire ici.
- **Local / valeur ajoutée** → Claude exécute directement (le reste de ses compétences).

## §3 Garde-fous — rester MINCE

- **Ce skill ROUTE, il ne ré-implémente pas.** La méthode de chaque plateforme vit dans SA compétence (ex `hubspot-breeze-audit`). Si ce fichier se met à décrire *comment* parler à une plateforme, c'est qu'une branche déborde → la sortir dans une compétence plateforme.
- **Une branche non câblée reste un principe d'une ligne** (GPT/Gemini aujourd'hui) — pas d'abstraction sur du vide, elle grossit le jour où on la branche pour de vrai.
- **La clôture appartient à la compétence plateforme** : c'est elle qui loge son travail dans le registre des projets IA (`plateformes-ai-registry.md`), pas l'orchestrateur.

## §4 Étendre — ajouter une plateforme

Nouvelle plateforme avec une IA / des crédits → (1) créer ou pointer une **compétence plateforme dédiée** (via `anthropic-skills:skill-creator` puis `/skill-factory`, CLAUDE-GLOBAL §1), (2) ajouter **une branche** dans le tableau §1 + §2. Jamais la mécanique ici.

## Skills liés

`hubspot-breeze-audit` (branche HubSpot — Breeze + audit écran-par-écran) · `/sessions` (router vers une autre session, CLAUDE-GLOBAL §27) · registre `plateformes-ai-registry.md` (la clôture, tenue par les compétences plateforme). Doctrine « cerveau le moins cher » : mémoire `token-economy-recommend-cheapest`.
