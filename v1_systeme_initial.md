# Système Agent IA — V1 Système Initial

> **Statut** : Version de référence avant introduction des notions d'itératif/incrémental et MVP.  
> **Philosophie** : Solo builder qui produit comme une agence — challenger avant de construire.  
> **Stack** : CrewAI (rôles agents) + LangGraph (orchestration + human-in-loop)

---

## Pourquoi ce document ?

Ce document capture la **première version du système** conçue avant d'introduire les notions de MVP, d'itératif et de boucle de feedback. Elle couvre deux phases principales : Discovery et Delivery. Elle sert de référence pour comprendre l'évolution vers la V2.

**Ce que cette version fait bien** :
- Challenge solide de l'idée avant de construire (EMBER, RAZOR, CONTRARIAN, JUDGE)
- Pipeline Delivery structuré avec points d'intervention humaine clairs
- Séparation nette des responsabilités entre agents

**Ce qu'il manque** (résolu en V2) :
- Pas de phase GTM — on passe de l'idée validée directement au dev sans stratégie de lancement
- Pas de notion de MVP — on livre "le produit" entier, pas un incrément utile
- Pas de boucle de feedback — une fois livré, le système s'arrête
- Pas de livrables clients automatisés

---

## Architecture globale

```
PHASE DISCOVERY
      │
      ▼
TOI — Brief produit
      │
      ├──────────────────────────────────┐
      ▼                                  ▼
Data.gouv              EMBER — Builder              Web Research
INSEE · marchés FR     Pitch · Recherche web        Reddit · HN · G2
données officielles    Évolue l'idée                Concurrents · Funding
      │                      │                           │
      └──────────────────────┼───────────────────────────┘
                             ▼
               ┌─────────────────────────┐
               │  RAZOR — Destroyer      │   CONTRARIAN
               │  Concurrents morts      │   Fatal flaws
               │  Preuves terrain        │   Hidden assumptions
               └─────────────────────────┘
                             │
                             ▼
                  JUDGE — Gates binaires
                  Theory gates · Mycelium
                  Bloque si preuves absentes
                             │
                             ▼
                  ✋ TOI — Go / No Go + pivot
                             │
                             ▼
                    Canvas YAML → specs
                             │
                             ▼
PHASE DELIVERY — CrewAI + LangGraph
                             │
                             ▼
                          Agent PM
                  User stories · Tickets
                  Critères d'acceptance
                             │
                             ▼
                  ✋ TOI — Validation specs
                             │
                    ┌────────┴────────────┐
                    ▼                     ▼
               Agent Dev          Agent Audit Tech [100% auto]
               Front · Back · DB  Sécu · Scalabilité
               Code · Architecture Archi · Perf
                    │                     │
                    └────────┬────────────┘
                             ▼
                  ✋ TOI — Rapport audit technique
                             │
                             ▼
                          Agent QA
                  Tests · Bugs · UAT
                  Critères acceptance
                             │
                             ▼
                  ✋ TOI — Recette finale PO
                             │
                             ▼
                      Produit livré ✓
```

---

## Phase Discovery — 4 agents

### Pourquoi une phase Discovery ?
> Avant de coder quoi que ce soit, le système challenge l'idée avec des preuves réelles. L'objectif est d'éviter de construire quelque chose que personne ne veut ou qui est déjà mort sur le marché. EMBER pitche, RAZOR tue, CONTRARIAN cherche les angles morts, JUDGE tranche.

### Agent EMBER — Builder
**Rôle** : Pitcher l'idée sous son meilleur jour, rechercher des preuves de marché positives.  
**Sources** : Web search (Reddit, HN, G2, Product Hunt, Crunchbase) + Data.gouv (INSEE, données sectorielles françaises officielles).  
**Output** : Pitch structuré + preuves terrain (taille marché, concurrents, funding, signaux d'adoption).

> **Commentaire** : EMBER n'est pas naïf — il cherche des preuves réelles, pas de l'enthousiasme. La source Data.gouv est cruciale pour ancrer les données dans le contexte français (statistiques INSEE, données entreprises, marchés sectoriels).

### Agent RAZOR — Destroyer
**Rôle** : Attaquer l'idée avec des preuves contraires. Chercher ce qui a échoué sur le même sujet.  
**Sources** : Startups mortes (Failory, CB Insights), articles critiques, signaux négatifs marché.  
**Output** : Liste des red flags avec preuves — concurrents morts, raisons d'échec, obstacles identifiés.

> **Commentaire** : RAZOR et EMBER sont complémentaires. Ensemble ils forment un débat interne équilibré. Dans la V2, ils seront fusionnés en un seul agent pour économiser les tokens tout en gardant la tension builder/destroyer.

### Agent CONTRARIAN
**Rôle** : Attaquer les assumptions cachées — celles que Vincent ne questionne pas car il les croit évidentes.  
**Sources** : Brief Vincent + outputs EMBER+RAZOR.  
**Output** : Fatal flaws · Hidden assumptions · Stress-test questions.

> **Commentaire** : C'est l'agent le plus précieux car il challenge ce que vous ne voyez pas vous-même. Exemple type : "Ton modèle économique est-il légalement compatible avec ton statut juridique ?" — une question évidente pour un juriste, invisible pour quelqu'un focalisé sur le produit.

### Agent JUDGE — Gates binaires
**Rôle** : Émettre un verdict Go/No-Go basé sur des critères binaires stricts. Ne peut pas être charmé par un bon pitch.  
**Inspiration** : Pattern Mycelium (theory gates).  
**Règle** : Si les preuves sont absentes → blocage automatique. Pas de "on verra bien".  
**Output** : Go/No-Go + raisons documentées + Canvas YAML v1 généré.

> **Commentaire** : Le JUDGE est le garde-fou du système. Sans lui, EMBER peut convaincre de construire n'importe quoi avec suffisamment d'enthousiasme. Les gates binaires sont non-négociables — soit la preuve existe, soit elle n'existe pas.

### Checkpoint TOI — Go/No-Go
**Quand** : Après le verdict JUDGE.  
**Tu fais quoi** : Tu lis le rapport Discovery complet, tu valides ou tu demandes un pivot d'orientation.  
**Output** : Décision actée dans le Canvas YAML.

### Canvas YAML — Mémoire du projet
**Rôle** : Fichier central versionné dans GitHub. Contient toutes les décisions, outputs de chaque phase, contexte projet.  
**Règle critique** : Un seul agent a le droit d'écrire = LangGraph Orchestrateur. Les autres soumettent leurs outputs, il arbitre et écrit. Zéro conflit d'écriture.

---

## Phase Delivery — 3 agents + QA

### Pourquoi CrewAI + LangGraph ?
> CrewAI gère les rôles (qui fait quoi), LangGraph gère l'orchestration et les points d'arrêt humains (human-in-loop). Ensemble ils permettent d'avoir un pipeline automatique avec des checkpoints précis où Vincent intervient — sans que tout dépende de lui.

### Agent PM
**Rôle** : Transformer le brief validé en specs structurées, stories, tickets.  
**Inputs** : Canvas YAML + décisions Discovery.  
**Output** : Backlog priorisé · User stories au format BDD (Gherkin) · Critères d'acceptance.

> **Commentaire** : Le BDD (Behaviour Driven Development) est clé pour Vincent car les scénarios Gherkin (Étant donné / Quand / Alors) sont en langage naturel — un PO les lit sans formation technique. Ces scénarios deviennent directement les tests d'intégration de l'Agent Dev.

### Checkpoint TOI — Validation specs
**Quand** : Après l'Agent PM, avant que l'Agent Dev commence à coder.  
**Tu fais quoi** : Tu valides les user stories, tu ajustes les priorités, tu confirmes les critères d'acceptance.  
**Pourquoi c'est ton moment fort** : C'est le checkpoint où ton profil PM/PO a le plus de valeur. Le reste (code, archi) sera validé par les agents techniques.

### Agent Dev
**Rôle** : Produire le code, l'architecture, le schéma de base de données.  
**Règle** : Il génère ses propres tests unitaires et d'intégration — pas un agent séparé pour ça.  
**Output** : Code · Tests unit/intégration · Décisions d'architecture (ADR internes).

> **Commentaire** : L'Agent Dev absorbe les tests unitaires pour éviter la prolifération d'agents. Un dev senior génère ses tests en même temps qu'il code — l'agent fait pareil. Ça économise des agents ET améliore la qualité car les tests sont écrits par celui qui connaît le code.

### Agent Audit Tech — 100% automatique
**Rôle** : Vérifier la sécurité, la scalabilité, les performances du code produit.  
**Outils CLI** (quasi zéro tokens LLM) : Bandit (sécu Python) · npm audit (dépendances JS) · Lighthouse (perf/accessibilité).  
**Règle bloquante** : Si critique rouge → livraison bloquée automatiquement. Orange → remonte à Vincent pour décision. Vert → pipeline continue.  
**Output** : Rapport feux R/O/V.

> **Commentaire** : L'Agent Audit Tech tourne en CLI sans LLM pour les vérifications — ce qui signifie quasi zéro coût tokens. Le LLM intervient uniquement pour synthétiser le rapport en langage clair. C'est le filet de sécurité technique pour Vincent qui ne peut pas challenger le code lui-même.

### Checkpoint TOI — Rapport audit
**Quand** : Après l'Agent Audit Tech.  
**Tu fais quoi** : Tu lis le rapport (feux R/O/V), tu décides sur les oranges, les rouges sont déjà bloquants.

### Agent QA — Tests E2E Playwright
**Rôle** : Tester les parcours utilisateur réels dans un vrai browser.  
**Règle** : 3-5 parcours critiques maximum — pas exhaustif. 20% des scénarios couvrent 80% des bugs réels.  
**Output** : Screenshots + vidéos de chaque parcours = recette PO sans lire de code.

> **Commentaire** : La règle des 3-5 parcours est non-négociable. Un E2E exhaustif est fragile (casse dès qu'un bouton change de place) et coûteux en temps d'exécution. On couvre le login, le parcours principal, et 2-3 cas limites critiques — c'est tout.

### Checkpoint TOI — Recette finale PO
**Quand** : Après l'Agent QA.  
**Tu fais quoi** : Tu regardes les screenshots et vidéos, tu valides que les parcours fonctionnent comme prévu. Pas de code à lire.

---

## Stack technique

| Composant | Usage | Coût |
|---|---|---|
| Claude Max | Moteur principal — Discovery, challenge, specs, livrables | ~90€/mois |
| CrewAI | Framework rôles agents | Gratuit open source |
| LangGraph | Orchestration + checkpoints human-in-loop | Gratuit open source |
| Playwright | Tests E2E browser automation | Gratuit open source |
| GitHub | Versionning prompts + Canvas YAML | Gratuit |
| Data.gouv | Données officielles françaises en Discovery | Gratuit (API) |

---

## Règles d'or de cette version

| Règle | Pourquoi |
|---|---|
| Un agent = un rôle, une responsabilité | Un agent surchargé fait deux choses à moitié |
| Canvas YAML = seule source de vérité | LangGraph Orchestrateur est le seul à écrire |
| Checkpoint TOI à chaque phase critique | Discovery → Specs → Livraison |
| E2E = 3-5 parcours critiques max | E2E exhaustif = fragile et coûteux |
| Agent Audit Tech = 100% auto | Filet de sécurité technique sans expertise code |

---

## Limites identifiées → résolues en V2

> Ces limites ont motivé l'évolution vers la V2. Elles ne sont pas des bugs — elles sont des choix délibérés de simplification pour une première version.

1. **Pas de GTM** : On valide l'idée mais on ne définit pas comment la lancer (cible, canal, prix, timing). Risque de construire le bon produit pour la mauvaise audience.
2. **Pas de MVP** : On livre "le produit" en une fois. Si l'idée est mauvaise, on le découvre trop tard et on a tout construit.
3. **Pipeline linéaire** : Discovery → Delivery → Fini. Pas de boucle. Pas d'apprentissage post-livraison.
4. **Pas de livrables clients** : Le système produit du code et des specs internes mais rien de présentable à un client non-tech.
5. **Pas de feedback loop** : Une fois livré, le système ne sait pas si le produit est adopté ou abandonné.

---

*Document généré dans le cadre de la conception du système agent IA de Vincent — Scrum Master freelance, solo builder.*
