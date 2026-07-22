---
name: hubspot-breeze-audit
description: >-
  Investiguer / comprendre / auditer une question HubSpot AU MOINDRE COÛT de tokens Claude, en déléguant au cerveau le moins cher avant de faire le travail soi-même. Compétence ADAPTATIVE à 2 modes qu'elle choisit selon la situation : (1) MODE BREEZE — poser la question à l'IA native de HubSpot (tokens gratuits, 3000 crédits) pour tout diagnostic / analyse / raisonnement sur la data du portail ; (2) MODE NAVIGATION ASSISTÉE — auditer écran par écran une fonctionnalité dont la vérité est éparpillée sans API de dump (Buyer Intent, Lead Scoring, Permissions, Pipelines, Settings complexes), en classant CONFIRMÉ / À VÉRIFIER / INCOHÉRENT / À IMPLÉMENTER. Règle d'or de l'audit : jamais conclure depuis un seul écran (la config globale ment souvent sur l'usage réel). En clôture, elle loge ce qu'elle a produit/utilisé dans le registre des plateformes AI. À invoquer sur "breeze", "demande à breeze", "économise les tokens hubspot", "fais bosser breeze", "pourquoi ce workflow / contact / donnée HubSpot", "audit guidé", "audit multi-écrans", "fais le tour de X dans HubSpot", "audit Buyer Intent / Lead Scoring / permissions / pipelines", ou toute investigation HubSpot où lire un seul écran ou tout faire soi-même serait une fausse conclusion ou du gaspillage de tokens.
---

# Skill — HubSpot Breeze & Audit (investiguer au moindre coût)

## §0 Le principe — au moindre coût de tokens Claude

Répondre à une question HubSpot en **déléguant au cerveau le moins cher qui sait le faire**, avant de le faire soi-même. Deux moyens pour une même fin (« comment marche vraiment cette fonctionnalité / cette donnée ? ») :

- **Breeze** = l'IA native de HubSpot, **tokens gratuits** → tout ce qu'elle peut analyser/répondre, on le lui demande.
- **Navigation assistée** = traverser les écrans soi-même, **seulement quand aucun oracle ne peut répondre** (vérité éparpillée, pas d'API).

> 🌍 **Cette compétence = 1ʳᵉ instance d'un principe plus large (cap, à généraliser quand une 2e plateforme le demande — pas avant).** Le principe : router chaque recherche vers le cerveau le moins cher — Breeze ici, **GPT / Gemini / une autre session** ailleurs — pour que Claude reste concentré sur la valeur ajoutée et le local qu'il est seul à pouvoir exécuter. Aujourd'hui = HubSpot. Demain, si un 2e contexte l'exige, la **méthode de navigation** (§3) est agnostique et s'extrait en skill cross-plateforme ; le **mode Breeze** (§2), lui, reste HubSpot-only. Aligne avec la doctrine « cerveau le moins cher » (mémoire `token-economy-recommend-cheapest`, règle « Breeze à fond »). → **Le chapeau `claude-ia-delegation`** (global global) orchestre ce choix et délègue ICI quand le contexte est HubSpot ; cette compétence = sa branche HubSpot.

## §1 Choisir le mode au démarrage — le décideur

Avant d'agir, une question : **la réponse est-elle atteignable en la DEMANDANT, ou faut-il ALLER LA VOIR écran par écran ?**

| Situation | Mode |
|---|---|
| Diagnostic / analyse / « pourquoi X » sur la data du portail (workflow, contact, volume, campagne) | **Breeze** (§2) — gratuit |
| Fonctionnalité répartie sur 3+ écrans, sans API de dump (Buyer Intent, Lead Scoring, Permissions, Pipelines, Record customization, Settings complexes) | **Navigation assistée** (§3) |
| Lecture/écriture d'1 donnée structurée précise (contact, deal, propriété) | ni l'un ni l'autre → **MCP HubSpot** direct (cf `hubspot-crm` §7) |

**Au lancement, si le bon mode n'est pas clair → le trancher selon la situation, ou demander à Florent lequel.** On ne se lance pas dans un audit écran-par-écran coûteux si Breeze répond en 30 s ; on ne demande pas à Breeze ce qui vit dans des réglages qu'il ne voit pas. Les deux modes **synergisent** : pendant une navigation (§3), poser une question à Breeze à une étape si ça évite d'ouvrir un écran de plus.

## §2 Mode Breeze — déléguer le raisonnement à l'IA native (gratuit)

But : décharger le raisonnement HubSpot sur **Breeze** pour économiser les tokens Claude. Règle Florent : *« Breeze à fond — normalement c'est lui qui fait toute la réflexion »*.

**Comment on opère — gardien de la méthode (Florent 2026-07-22)** : HubSpot ne permet **aucune instruction custom au niveau du COMPTE** (seulement dans un projet Breeze — mais en mettre dans chaque projet le surcharge). Donc **la méthode est portée par TOI, l'agent, via ce skill — jamais par une config HubSpot**. Concrètement :
- Tu travailles dans les **projets Breeze EXISTANTS** de l'utilisateur (tu n'en crées **pas** un nouveau).
- Tu **changes les fenêtres via Chrome MCP** pour amener la data à Breeze **écran par écran**, et **Breeze fait le raisonnement**.
- Tu **boucles jusqu'à ce que Breeze ait absorbé TOUTE la data ET produit l'analyse**.

Toi = les mains + le gardien de la discipline · Breeze = le cerveau (gratuit). C'est l'arbitrage de tokens en pratique. Le prompt `references/navigation-assistee-prompt.md` (rangé dans les **prompts partagés** HubSpot de Florent) = un **renfort optionnel**, pas le mécanisme.

**Où** (portail 21450353) : ✦ « Open Assistant » (haut-droite) → 📁 Projects → projet **« Marketing automation »**. Crédits IA 3000. Détail : `pastry-chefs-boutique/CLAUDE.md` §4bis.

**La mécanique — tout en auto via JS (méthode prouvée 2026-06-16)** :
1. **Ouvrir** : `find` « Open Assistant » → clic.
2. **Écrire** (`javascript_tool`) : descendre récursivement les iframes jusqu'au doc `[contenteditable="true"]`, `el.focus()` puis **`doc.execCommand('insertText', false, txt)`** (le champ est dans un iframe imbriqué same-origin → `computer type` / CDP n'atterrit pas ; execCommand oui).
3. **Envoyer** : dans CE doc, `doc.querySelector('[data-test-id="chat-send-button"]').click()`.
4. **Lire** : poller `body.innerText` récursif jusqu'à ce que la longueur arrête de grandir (~30-50 s).

```js
function findDoc(d){if(d.querySelector('[contenteditable="true"]'))return d;
for(const f of d.querySelectorAll('iframe')){try{const x=f.contentDocument;if(x){const r=findDoc(x);if(r)return r;}}catch(e){}}return null;}
```
⚠️ REPL `javascript_tool` persiste les `var` → wrapper en IIFE `(()=>{...})()`. **Submit fiable = 1ʳᵉ question d'un thread** ; un follow-up dans le même thread peut ne pas déclencher la génération → **1 question = 1 thread neuf** (ou `computer key Return` trusted). Ne PAS `selectAll` avant l'insertText.

**Banque de prompts** : « Pourquoi le workflow X n'enrôle plus ? Volume/mois sur 6 mois, date de chute. » · « Combien de contacts/visiteurs trackés/mois sur 6 mois ? » · « Dernier deal e-commerce, quelle date ? » · « Pourquoi ce contact n'a pas de téléphone ? » (historique propriété).

**Breeze ne sait PAS** : le statut d'une app tierce (→ Chrome / URL intégrations) · créer un objet (il analyse, il ne crée rien).

**⚠️ Breeze rate les CHIFFRES EXACTS (limite prouvée 2026-07-22)** : il lit l'écran courant mais **manque des métriques** — ex les compteurs d'en-tête d'une vue, qu'il n'a pas « vus ». → **Bon pour RAISONNER / interpréter, PAS pour relever un chiffre précis.** Chiffre exact requis → le **lire soi-même** via Chrome (navigation §3) ou via l'**API/MCP** (`hubspot-crm` §7), jamais le demander à Breeze. Corollaire de l'opérateur ci-dessus : c'est TOI qui apportes la data (les chiffres exacts inclus), Breeze raisonne dessus.

## §3 Mode Navigation assistée — auditer écran par écran (pas d'API)

Certaines fonctionnalités n'ont **aucune API de dump** — leur vérité est répartie sur plusieurs écrans, et **la config globale peut être large et brute pendant que l'usage réel, plus fin, vit ailleurs** (une vue sauvegardée, un filtre, un panneau replié).

**Les 2 rôles** — **Navigateur** = qui a la main sur l'écran (Claude par défaut, via Chrome MCP : `navigate`/`find`/`read_page`/`computer` ; Florent seulement si un écran n'est pas atteignable par Claude). **Pilote = toujours Claude** : lit, classe, décide de l'écran suivant, documente. Le réflexe du pilote n'est jamais « qu'est-ce que je clique ? » mais **« quel écran confirme ou infirme l'hypothèse actuelle ? »**.

**Le protocole — 5 étapes en boucle** :
1. **Objectif** : quelle fonctionnalité, quel doute précis à lever.
2. **Point de départ** : l'écran actuel (souvent l'Overview de la fonctionnalité).
3. **Prochain écran + pourquoi** : le pilote annonce quel écran ouvrir et quelle hypothèse ça teste — jamais de navigation sans hypothèse déclarée.
4. **Ouverture + constat** : le navigateur ouvre, le pilote relève, met à jour le journal (ci-dessous), reformule l'écran suivant.
5. **Boucle jusqu'à couverture complète** : tant qu'il reste une zone floue, une contradiction, ou un réglage non confirmé.

**⭐ Règle d'or — config globale ≠ usage réel.** Ne jamais conclure depuis un seul écran. Chaque constat se classe dans une des 4 cases : **CONFIRMÉ** (vu directement) · **À VÉRIFIER** (indice, pas encore confirmé ailleurs) · **INCOHÉRENT** (deux écrans se contredisent) · **À IMPLÉMENTER** (gap → action à proposer, jamais exécutée dans l'audit).

**Anti-patterns** : naviguer sans hypothèse · ouvrir 10 écrans d'un coup · conclure avant couverture complète · supposer que la config globale = l'usage réel sans vérifier vues/filtres/segments dérivés.

**Le journal (doc d'audit)** → `memory/clients/<slug>/audit-<feature>.md` :
```markdown
# Audit <Fonctionnalité> — navigation assistée — <Client>
**Portail :** <id> · **Date :** <YYYY-MM-DD> · **Objectif :** <doute à lever>
## Journal
| # | Écran | Hypothèse testée | Constat | Statut |
## Bilan  (CONFIRMÉ / À VÉRIFIER / INCOHÉRENT / À IMPLÉMENTER)
## Conclusion  (la phrase qui résiste à la conclusion prématurée)
```
⚠️ Un « No visitors found / -- » juste après avoir changé d'écran = souvent un **état de chargement**, pas un vrai zéro → re-vérifier après un wait (CLAUDE.md global §9, « le zéro se prouve »).

**📎 Sans Chrome MCP (fallback humain / autre IA)** : `references/navigation-assistee-prompt.md` porte le prompt d'origine (Florent, brouillon) qui formalise cette même discipline pour un contexte sans navigateur — copiable tel quel dans ChatGPT/Gemini/le chat Breeze, ou à tenir soi-même si Chrome MCP est mort en session (jamais conclure avant confirmation de la bonne page ; toujours nom de la page + pourquoi + URL + action attendue).

## §4 Clôture — loger dans le registre des plateformes AI

En finissant, la compétence **absorbe son travail** : elle loge dans `memory/plateformes-ai-registry.md` le projet/chat Breeze utilisé ou créé, et tout asset AI croisé pendant l'audit qui n'y est pas encore (source Claude Design, vue automatisée…). Colonnes en tête du fichier. Même passe, jamais différé — c'est ce qui évite de recréer la fois d'après. **Avant** d'ouvrir un nouveau projet/chat → checker le registre d'abord (il existe peut-être déjà).

## §5 Garde-fous

- **Read-only par défaut.** Cette compétence investigue et documente ; elle ne crée/modifie/supprime rien sur le portail sans le gate consentement (`Clients/CLAUDE.md` §1).
- **Un gap « À IMPLÉMENTER » se présente à Florent** avec le plan précis — jamais exécuté avant son « oui ».

## §6 Cas inaugural — Pastry Chef's Boutique, Buyer Intent (2026-07-22)

Premier run du mode navigation (Chrome MCP, lecture seule) : 9 saved views + l'écran Configuration, portail 21450353. La règle d'or (§3) s'est vérifiée en direct — la vue par défaut (975 companies, tout à « Any ») laissait croire à un setup non discriminant ; l'écran Configuration a révélé la vraie racine : un *Intent Criteria* compte-large (5 chemins) et 6 *Target Markets* sectoriels précis existent mais ne sont câblés sur quasiment aucune vue. Sans traverser les écrans → conclusion fausse « pas de segmentation » au lieu de « segmentation prête mais pas branchée ». Doc : `memory/clients/pastry-chefs-boutique/audit-buyer-intent.md`.

## Skills liés

`hubspot-crm` (skill central, table de décision Breeze/MCP/Computer-Use §7) · `hubspot-segments-audit` + `hubspot-workflows-audit` (audit API-first, quand une API de dump existe) · `crm-investigation-output` (format verdict/email/infographie si l'audit débouche sur une action client) · registre `memory/plateformes-ai-registry.md` (la clôture §4).
