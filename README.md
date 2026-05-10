# Solo Builder AI

Documentation vivante du système d'agents IA que je construis pour produire seul des projets de qualité agence.

## Contexte

Je suis Scrum Master / Coach Agile / Chef de projet / Delivery Manager freelance, marchés FR + Suisse, FR + EN. Mon profil est PM/PO, pas développeur. Je veux pouvoir prendre en charge des projets de bout en bout — du challenge de l'idée jusqu'à la livraison et au feedback — sans avoir à recruter ni sous-traiter à chaque étape.

Le système d'agents IA est l'outil qui rend ça possible. Il me permet :
- de **challenger** une idée avec des preuves réelles avant de construire (pas avec de l'enthousiasme)
- de **scoper** un MVP plutôt que livrer un produit fini d'un coup
- de **produire** code, tests, audit technique, livrables clients en cycle court
- d'**apprendre** de chaque livraison via une boucle de feedback structurée

Mon rôle reste celui où j'apporte le plus de valeur : checkpoints de validation, arbitrages, relation client. Le reste est délégué à des agents avec des rôles précis et des points d'arrêt explicites.

## Architecture

Stack cible : **CrewAI** (rôles agents) + **LangGraph** (orchestration + human-in-loop) + **Playwright** (tests E2E) + **CLI sécu** (Bandit, npm audit, Lighthouse).

5 phases en boucle : **Discovery** → **GTM** → **Delivery itératif** → **Feedback 2 couches** → pivot / persévère / backlog +1.

Une mémoire centrale (**Canvas YAML**) versionnée dans Git, écrite uniquement par l'orchestrateur LangGraph — zéro conflit d'écriture entre agents.

## Documents

| Doc | Contenu |
|---|---|
| [`v1_systeme_initial.md`](./v1_systeme_initial.md) | Version initiale — pipeline linéaire Discovery → Delivery. Sert de référence pour comprendre l'évolution. |
| [`v2_systeme_complet.md`](./v2_systeme_complet.md) | Version cible — avec GTM, MVP itératif, boucle de feedback, 8-10 agents. Document de référence actif. |

## Statut

Conception terminée. En cours : construction concrète des agents (prompts détaillés, schéma Canvas YAML, code Python).

## Évolutions

Ce repo est mis à jour à mesure que le système évolue. Chaque version majeure du système donne lieu à un nouveau document (`v3_...md`, etc.) plutôt qu'à une réécriture en place — pour garder la trace des décisions et de leur motivation.
