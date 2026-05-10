# Système Agent IA — V2 Système Complet

> **Statut** : Version cible complète — avec GTM, MVP, itératif/incrémental et boucle de feedback.  
> **Philosophie** : Solo builder qui produit comme une agence — plus tôt on livre, plus tôt on a du vrai feedback, plus tôt on réadapte.  
> **Stack** : CrewAI (rôles) + LangGraph (orchestration + human-in-loop) + Playwright (E2E) + CLI sécu

---

## Pourquoi cette version ?

La V1 validait l'idée et livrait un produit. Elle avait une limite fondamentale : **le pipeline s'arrêtait à la livraison**. On ne savait pas si le produit était adopté. On ne pouvait pas réadapter.

La V2 introduit trois notions clés :

- **GTM (Go-to-Market)** : avant de coder, on définit la stratégie de lancement, le périmètre MVP, et les métriques de succès
- **Itératif/incrémental** : on ne livre pas "le produit", on livre des incréments utiles en cycle court. Le premier incrément c'est le MVP
- **Boucle de feedback** : chaque livraison génère du feedback (synthétique avant, réel après) qui réalimente le cycle suivant

**Le principe lean appliqué** : build → measure → learn → repeat. Plus tôt on livre, plus tôt on sait si on construit la bonne chose.

---

## Architecture globale

```
┌─────────────────────────────────────────────────────────────┐
│                    PHASE DISCOVERY                          │
│                                                             │
│  TOI — Brief produit (idée, besoin, contexte)              │
│         │              │              │                     │
│    Data.gouv      EMBER+RAZOR     Web Research              │
│    INSEE·FR       Builder+Destroyer Reddit·HN·G2           │
│         │              │              │                     │
│         └──────────────┼──────────────┘                    │
│                        ▼                                    │
│              CONTRARIAN (assumptions cachées)               │
│                        │                                    │
│              JUDGE — Gates binaires (Mycelium)              │
│                        │                                    │
│         ✋ TOI — Go/No-Go + Canvas YAML généré              │
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   PHASE GTM                                 │
│                                                             │
│  Agent GTM (3 modes séquentiels) :                         │
│  1. Stratégie → cible, canal, prix, timing                 │
│  2. MVP Scoper → Must / Should / Later                     │
│  3. Metric Setter → KPIs + critères pivot/persévère        │
│                                                             │
│         ✋ TOI — Validation stratégie + scope MVP           │
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│         PHASE DELIVERY itérative — CrewAI + LangGraph       │
│                                                             │
│              Agent PM (stories + BDD + tickets)             │
│         ✋ TOI — Validation specs + priorisation            │
│                    │              │                         │
│               Agent Dev    Audit Tech [100% auto]           │
│               Code·Archi   Sécu·Scalabilité·Perf           │
│                    │              │                         │
│         ✋ TOI — Rapport audit (feux R/O/V)                 │
│                         │                                   │
│              Agent QA E2E (3-5 parcours critiques)          │
│         ✋ TOI — Recette PO visuelle (screenshots/vidéos)   │
│                         │                                   │
│                  Livraison incrément ✓                      │
└─────────────────────────────────────────────────────────────┘
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
   Feedback synth   Feedback réel   Insight Synthesizer
   4 personas       Métriques·NPS   → Canvas YAML mis à jour
   avant livraison  après livraison
          │              │
          └──────────────┘
                  ▼
         ✋ TOI — Pivot / Persévère / Backlog +1
                  │
         ┌────────┴────────────────┐
         ▼                         ▼
  Backlog sprint +1         GTM ou Discovery
  (ajustement mineur)       (hypothèse fausse)
```

---

## Phase Discovery — 3 agents (EMBER+RAZOR fusionnés)

### Pourquoi fusionner EMBER et RAZOR ?
> Dans la V1, EMBER et RAZOR étaient deux agents séparés. En V2, ils sont fusionnés en un seul agent en "mode débat interne" — il pitche ET détruit l'idée dans la même session. Cela économise des tokens et accélère le cycle sans perdre la tension builder/destroyer.

### Agent EMBER+RAZOR — Builder ET Destroyer
**Rôle** : Pitcher l'idée sous son meilleur jour, puis immédiatement la challenger avec des preuves contraires. Mode débat interne.  
**Sources** :
- Web search (Reddit, HN, G2, Product Hunt, Crunchbase, Failory)
- **Data.gouv** — connecteur dédié pour données officielles françaises (INSEE, marchés sectoriels, données entreprises). Source fiable et non-biaisée, essentielle pour ancrer les analyses dans le contexte FR.

**Output** : Pitch + preuves terrain + red flags + concurrents morts identifiés.

> **Commentaire** : La clé de cet agent c'est "accumule des preuves, pas de l'enthousiasme". Data.gouv est particulièrement précieux pour les marchés français — taille réelle d'un secteur, nombre d'entreprises cibles, données financières sectorielles. Ça évite les projections optimistes non fondées.

### Agent CONTRARIAN
**Rôle** : Attaquer spécifiquement les **assumptions cachées** — ce que Vincent ne questionne pas car il le croit évident.  
**Différence avec RAZOR** : RAZOR cherche des preuves externes (concurrents, marché). CONTRARIAN cherche les angles morts internes (logique du modèle, dépendances implicites, risques légaux/opérationnels).  
**Output** : Fatal flaws · Hidden assumptions · Stress-test questions.

> **Commentaire** : Exemple type — RAZOR cherche des concurrents et des échecs marché. CONTRARIAN aurait posé : "Ton modèle économique est-il légalement compatible avec ton statut juridique ?" C'est une question évidente pour un juriste mais invisible quand tu es focalisé sur le produit. CONTRARIAN est l'avocat du diable systématique.

### Agent JUDGE — Gates binaires
**Rôle** : Verdict Go/No-Go strict basé sur des critères binaires. Ne peut pas être charmé.  
**Inspiration** : Pattern Mycelium — theory gates.  
**Règle** : Chaque gate est binaire — la preuve existe ou elle n'existe pas. Pas de "on espère que". Bloque le pipeline si les preuves sont absentes.  
**Output** : Go/No-Go + Canvas YAML v1 généré (mémoire partagée du projet).

### Checkpoint TOI — Go/No-Go
Tu lis le rapport complet, tu valides le Go ou tu demandes un pivot d'orientation avant de passer au GTM.

---

## Phase GTM — 1 agent fusionné (3 modes)

### Pourquoi une phase GTM entre Discovery et Delivery ?
> La V1 passait directement de l'idée validée au dev. Problème : on pouvait construire le bon produit pour la mauvaise audience, via le mauvais canal, au mauvais prix. Le GTM définit **comment** lancer avant de coder. C'est aussi là que se décide le périmètre MVP — ce qui évite de construire trop avant d'avoir du vrai feedback.

### Agent GTM — Stratégie + Scope + Métriques
**Rôle** : Trois modes séquentiels dans un seul agent pour économiser les tokens et garder la cohérence.

**Mode 1 — GTM Stratégie**
- Segment cible prioritaire
- Canal d'acquisition principal (et risque de concentration si un seul canal)
- Positionnement et message
- Prix et modèle de revenus
- Timing de lancement

**Mode 2 — MVP Scoper**
- Must (sans ça le produit n'a aucune valeur) : à livrer en sprint 1
- Should (important mais pas bloquant) : sprint 2-3
- Later (nice to have) : après validation du MVP

> **Commentaire** : Le MVP Scoper est le garde-fou du scope. Sans lui, l'Agent Dev construit le produit entier. Avec lui, le premier incrément est minimal et livrable rapidement pour avoir du vrai feedback utilisateur. "Si on ne peut pas être embarrassé par la V1, on a attendu trop longtemps."

**Mode 3 — Metric Setter**
- KPIs de succès par incrément (activation, rétention, time-to-value)
- Critères de pivot (si ces métriques ne sont pas atteintes → on change)
- Critères de persévère (si ces métriques sont atteintes → on continue et on approfondit)

> **Commentaire** : Les critères pivot/persévère sont définis AVANT de livrer — pas après. Ça évite le biais de confirmation ("on garde parce qu'on a construit"). Les métriques tranchent objectivement.

### Checkpoint TOI — Validation stratégie + scope MVP
Tu valides la stratégie de lancement et le périmètre du premier incrément. C'est le dernier checkpoint avant que l'Agent PM commence à générer des stories.

---

## Phase Delivery — Itérative et incrémentale

### Le principe fondamental
> On ne livre pas "le produit". On livre des incréments utiles en cycle court. Le premier incrément c'est le MVP défini par le GTM. Chaque sprint suivant est piloté par ce qu'on a appris du précédent.

### Agent PM
**Rôle** : Transformer le scope validé en specs structurées, stories et tickets priorisés.  
**Inputs** : Canvas YAML + scope MVP Scoper.  
**Output** :
- Backlog priorisé (Must en sprint 1, Should en sprint 2+)
- User stories au format BDD/Gherkin (lisibles sans formation technique)
- Critères d'acceptance par story

> **Commentaire** : Le BDD est le pont entre le PM/PO et le développement. Les scénarios Gherkin (Étant donné / Quand / Alors) sont en langage naturel — Vincent les valide sans lire du code. Ces mêmes scénarios deviennent les tests d'intégration de l'Agent Dev. Un seul document source, deux usages.

### Checkpoint TOI — Validation specs + priorisation
Ton checkpoint le plus important en Delivery. Tu valides les stories, tu t'assures que le Must est bien le MVP défini, tu ajustes les priorités si besoin.

### Agent Dev
**Rôle** : Produire le code, l'architecture, le schéma de base de données.  
**Règle clé** : Il génère SES PROPRES tests unitaires et d'intégration en même temps qu'il code. Pas d'agent QA séparé pour ça.  
**Output** : Code · Tests unit/intégration · Décisions d'architecture internes.

> **Commentaire** : L'Agent Dev absorbe les tests pour éviter la prolifération d'agents (on avait identifié le risque d'avoir trop d'agents). Un bon dev génère ses tests en même temps qu'il code — l'agent fait pareil. Ça maintient la cohérence entre le code et ses tests.

### Agent Audit Tech — 100% automatique
**Rôle** : Vérifier la sécurité, scalabilité, performances. Bloquant si critique.  
**Outils CLI** (quasi zéro tokens LLM) :
- **Bandit** — audit sécurité Python
- **npm audit** — dépendances JS vulnérables
- **Lighthouse** — performance et accessibilité web

**Règle de blocage** :
- 🔴 Rouge → livraison bloquée automatiquement par LangGraph
- 🟠 Orange → remonte à Vincent pour décision
- 🟢 Vert → pipeline continue

> **Commentaire** : Les outils CLI tournent sans LLM — ce qui signifie quasi zéro coût tokens pour cette couche. C'est intentionnel : les outils de sécu CLI sont plus fiables que le LLM pour détecter les vulnérabilités connues. Le LLM intervient uniquement pour synthétiser le rapport en langage clair pour Vincent.

### Checkpoint TOI — Rapport audit
Tu lis le rapport (feux R/O/V). Les rouges sont déjà bloqués. Tu décides sur les oranges.

### Agent QA E2E — Playwright
**Rôle** : Tester les parcours utilisateur réels dans un vrai browser.  
**Règle absolue** : 3-5 parcours critiques maximum. Pas exhaustif.  
**Parcours à couvrir** : Login/auth · Parcours principal (la feature core du MVP) · 2-3 cas limites critiques.  
**Output** : Screenshots + vidéos de chaque parcours.

> **Commentaire** : La règle des 3-5 est non-négociable pour deux raisons. (1) Coût : un E2E complet consomme 20-50k tokens par run et prend 5-15 minutes. (2) Fragilité : les tests E2E cassent dès qu'un élément UI change de place, même si la fonctionnalité marche. On couvre ce qui ne change pas souvent et qui représente la valeur core.

### Checkpoint TOI — Recette finale PO
Tu regardes les screenshots et vidéos. Tu valides que les parcours fonctionnent comme prévu. Zéro code à lire — c'est le principe.

### Livraison incrément ✓
Le produit est livré à de vrais utilisateurs en conditions réelles. Ce n'est pas "le produit fini" — c'est le prochain incrément utile.

---

## Boucle de feedback — 2 couches

### Pourquoi deux couches ?
> Une seule couche de feedback crée un angle mort. Le feedback synthétique est rapide mais artificiel. Le feedback réel est la vérité mais il arrive après. Les deux ensemble donnent un cycle court ET fiable.

### Couche 1 — Feedback synthétique (avant livraison)
**Quand** : Sur prototype, avant livraison réelle.  
**Comment** : 4 agents personas qui interagissent avec le prototype.  
**Limites** : Ne ressent pas la vraie frustration. Signal de détection précoce, pas de vérité.

**Les 4 personas — ancrés domaine IT/PME français** :

| Persona | Profil | Ce qu'il challenge | Signal d'alerte |
|---|---|---|---|
| DSI PME prudent | Budget serré, allergique à la complexité | Maintenabilité, coût TCO, dépendance fournisseur | Solution trop complexe à maintenir sans Vincent |
| Dirigeant non-tech | Voit la valeur métier, pas les choix techniques | Clarté valeur, ROI visible, timeline | Livrable illisible sans explication |
| Dev senior sceptique | Challenge tout, déteste le sur-engineering | Qualité code, dette technique, scalabilité | Choix techniques non justifiés |
| Utilisateur final pressé | Adopte si intuitif en 2 min, abandonne sinon | UX, onboarding, friction parcours | Parcours principal > 3 étapes |

### Couche 2 — Feedback réel (après livraison)
**Quand** : Post-livraison, métriques usage réel.  
**Comment** : KPIs définis par le Metric Setter + verbatims + tickets support.  
**Limites** : Plus lent, mais source de vérité.

**Métriques types** :
- Activation (% qui complètent le parcours principal)
- Rétention (% qui reviennent après J+7)
- Time-to-value (temps avant le premier "aha moment")
- NPS / verbatims qualitatifs

### Agent Insight Synthesizer
**Rôle** : Croiser feedback synthétique + réel. Prioriser ce qui est confirmé par les deux sources. Produire des insights actionnables.  
**Output** : Insights priorisés + Canvas YAML mis à jour + recommandation de niveau de remontée.

> **Commentaire** : L'Insight Synthesizer est la pièce centrale de la boucle. Il ne se contente pas de lister le feedback — il trie ce qui est confirmé par les deux couches (priorité haute) de ce qui n'est que dans une couche (à surveiller). Il propose aussi le niveau de remontée pour chaque insight.

### Checkpoint TOI — Décision pivot/persévère
Tu tranches le niveau de remontée de chaque insight :
- **Backlog sprint +1** : ajustement mineur, on continue le cycle
- **GTM revisité** : la cible ou le canal est à revoir
- **Discovery relancée** : une hypothèse de fond était fausse, on repart de zéro sur cet axe

> **Règle d'or** : Synthétique = vitesse + détection précoce. Réel = vérité + décision. Les deux ensemble = cycle court fiable.

---

## Agent Livrable — 6 livrables event-driven

### Pourquoi un agent dédié aux livrables ?
> L'Agent PM gère les specs internes (stories, tickets). L'Agent Livrable gère les documents clients — c'est une audience différente, un langage différent, un format différent. Un agent dédié peut être optimisé uniquement pour la qualité de présentation client sans compromettre la rigueur technique.

### Principe de double version systématique
Chaque livrable existe en deux formats depuis la même source (Canvas YAML) :
- **Version Exécutif** : 1 page, langage métier, lisible sans contexte technique
- **Version Technique** : détaillée, jargon IT, pour DSI et équipes dev

### Déclenchement event-driven
L'Agent Livrable ne tourne pas à chaque étape — il écoute les événements système et se déclenche uniquement quand un livrable est impacté.

| Livrable | Événement déclencheur | Fréquence MAJ |
|---|---|---|
| Impact Map | Fin Discovery validée | Si scope change |
| Story Map | Après Agent PM | Chaque sprint |
| Roadmap Now/Next/Later | Validation GTM | Chaque sprint |
| Sprint Review Deck | Fin chaque incrément | Chaque sprint |
| Rapport Audit Tech (2 versions) | Après Agent Audit Tech | Chaque incrément |
| Decision Log | Toute décision système | Continu |

### Format de sortie
- **Notion** (principal) : partageable par lien, templates dupliquables, connecteur MCP déjà actif
- **HTML statique** : pour les visuels complexes (Impact Map, Story Map) — rendu dans browser, convertissable PDF

> **Commentaire** : Le Decision Log est souvent sous-estimé mais c'est un différenciateur commercial fort. C'est une trace horodatée de toutes les décisions prises (pourquoi ce scope, pourquoi ce choix technique, pourquoi ce pivot). En contexte réglementaire ou projet complexe, personne ne le produit spontanément — et les clients adorent recevoir ça.

---

## QA — Architecture 3 couches

### Couche 1 — Tests unitaires (dans Agent Dev)
Absorbés par l'Agent Dev — générés en même temps que le code. Pytest (Python) ou Jest (JS). Vise 80%+ de coverage. Bloque si sous le seuil défini.

### Couche 2 — Tests d'intégration BDD (dans Agent Dev)
Scénarios Gherkin générés depuis les critères d'acceptance de l'Agent PM. Valide les contrats entre composants (front/back/DB) et les endpoints API. Résultats présentés en vert/orange/rouge par feature — lisible non-tech.

### Couche 3 — Tests E2E (Agent QA dédié)
Playwright · 3-5 parcours critiques · Screenshots + vidéos. Voir section Agent QA E2E ci-dessus.

> **Commentaire** : L'architecture 3 couches couvre les 3 niveaux de risque à des coûts très différents. Unitaires = presque gratuit en tokens, rapides. Intégration = moyen. E2E = plus cher et plus lent, donc limité aux parcours critiques. C'est le rapport 80/20 appliqué à la QA.

---

## Infrastructure

### Phase 1 — Construction et test (maintenant)
**Où** : Poste local perso.  
**Coût** : 0€ de plus que l'abonnement Claude Max existant.  
**Pour quoi** : Construire les agents, tester les prompts, valider que le pipeline fonctionne.

### Phase 2 — Clients (quand système validé)
**Où** : Railway ou Render (cloud léger).  
**Coût** : ~10€/mois.  
**Pour quoi** : Pipelines sans être devant l'ordi · Livrables accessibles par URL client.

### Répartition Claude Max vs Ollama local

| Tâche | Moteur | Coût tokens |
|---|---|---|
| Discovery, challenge, specs, livrables | Claude Max | Inclus abonnement |
| Reformatage, extraction, variations simples | Ollama (Mac Mini M2) | 0€ après achat hardware |
| Orchestration pipelines | Railway/Render | ~10€/mois |
| Automatisations métier | Make | ~10€/mois |

**Budget cible** : 90€ Max + 20-30€ cloud/Make = **110-120€/mois**. Marge 80-180€ pour outils spécifiques projets clients.

**Mac Mini M2 d'occasion** : ~500-600€ achat unique. Fait tourner Llama 3 ou Mistral localement pour toutes les tâches répétitives. ROI en 4-5 mois vs coût API équivalent.

---

## Sécurité des données

> Règle pragmatique adaptée au niveau de démarrage — à faire évoluer avec les premiers clients sensibles.

- **Principe** : Anonymiser les briefs avant injection dans les agents (CLIENT_A, données financières arrondies)
- **Suffisant pour** : 95% des cas de départ, PME, projets internes
- **Pour clients grand compte / RGPD strict** : Claude for Enterprise ou anonymisation systématique renforcée
- **Ne jamais injecter** : données personnelles identifiables, secrets industriels non anonymisés, mots de passe ou clés API

---

## Canvas YAML — Mémoire centrale du système

**Rôle** : Fichier unique versionné dans GitHub. Contient tout — contexte projet, décisions, outputs de chaque phase, profil destinataires livrables, métriques de succès.

**Règle de gouvernance** :
- **Un seul écrivain** : LangGraph Orchestrateur
- Les agents **soumettent** leurs outputs à l'Orchestrateur
- L'Orchestrateur **valide et écrit** dans le Canvas
- **Zéro conflit** d'écriture possible

**Tous les agents lisent** le Canvas en début de run pour avoir le contexte complet.  
**L'Agent Livrable** s'en sert comme source unique pour générer tous les documents clients.

---

## Règles d'or du système

| Règle | Pourquoi |
|---|---|
| Un agent = un rôle, une responsabilité | Un agent surchargé fait deux choses à moitié |
| Canvas YAML = seule source de vérité | Tous lisent/écrivent via LangGraph Orchestrateur uniquement |
| Checkpoint TOI avant chaque phase critique | Discovery → GTM → Specs → Livraison → Feedback |
| E2E Playwright = 3-5 parcours critiques max | E2E exhaustif = fragile et coûteux. 20% scénarios = 80% bugs réels |
| Claude Max pour le complexe, Ollama pour le répétitif | Gestion raisonnée coût/gain tokens |
| Livrables double version systématique | Exécutif (métier) + Technique (IT) depuis le même Canvas |
| Anonymiser avant d'injecter | Données clients sensibles ne transitent pas telles quelles |
| MVP du système avant système complet | Valider 3-4 agents core avant d'en ajouter |
| Feedback synthétique + réel ensemble | Synthétique = vitesse. Réel = vérité. Les deux = fiabilité |

---

## Récapitulatif des 8 agents

| # | Agent | Phase | Rôle en une ligne |
|---|---|---|---|
| 1 | EMBER+RAZOR | Discovery | Builder ET Destroyer — débat interne avec preuves réelles |
| 2 | CONTRARIAN | Discovery | Attaque les assumptions cachées |
| 3 | JUDGE | Discovery | Go/No-Go binaire — bloque sans preuves |
| 4 | Agent GTM | GTM | Stratégie + MVP Scoper + Metric Setter (3 modes) |
| 5 | Agent PM | Delivery | Stories BDD + tickets priorisés |
| 6 | Agent Dev | Delivery | Code + archi + ses propres tests unit/intégration |
| 7 | Agent Audit Tech | Delivery | Sécu + perf CLI — 100% auto — bloquant si rouge |
| 8 | Agent QA E2E | Delivery | Playwright 3-5 parcours — screenshots + vidéos |
| 9 | Insight Synthesizer | Feedback | Croise synth + réel → insights actionnables |
| 10 | Agent Livrable | Transverse | 6 livrables clients event-driven — Notion + HTML |

> **Note** : On vise 8 agents actifs en régime de croisière. Agent Dev absorbe QA unit/intégration. Agent GTM fusionne 3 rôles. L'objectif est d'éviter la prolifération d'agents qui complexifie la maintenance.

---

## Prochaine étape — MVP du système

Construire les 4 agents qui donnent 80% de la valeur en premier :

| Priorité | Agent | Valeur immédiate |
|---|---|---|
| 1 | EMBER+RAZOR+CONTRARIAN | Challenge l'idée sur les angles morts |
| 2 | Agent PM | Génère les specs structurées depuis le brief validé |
| 3 | Agent Audit Tech | Filet de sécurité technique automatique (CLI) |
| 4 | Insight Synthesizer | Digère le feedback et pilote le cycle suivant |

Une fois ces 4 agents validés, enrichissement dans cet ordre :  
**Agent GTM → Agent QA E2E → Agent Livrable → Boucle feedback complète**

---

*Document généré dans le cadre de la conception du système agent IA de Vincent — Scrum Master freelance, solo builder.*  
*Référence V1 : `v1_systeme_initial.md`*
