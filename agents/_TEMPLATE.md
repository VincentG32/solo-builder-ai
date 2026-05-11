# Agent — [NOM]

> **Rôle en une ligne** : [phrase courte qui résume ce que fait l'agent]
> **Phase** : Discovery / GTM / Delivery / Feedback / Transverse
> **Modèle LLM** : Claude Opus 4.7 / Sonnet 4.6 / Haiku 4.5 / Ollama local / CLI (sans LLM)
> **Statut spec** : 🟡 Cadrage léger · 🟢 Spec exécutable

---

## 1. Pourquoi cet agent ?

> Le problème concret qu'il résout dans le pipeline. Ce qui se passerait si on l'enlevait.
> 2-3 phrases max.

---

## 2. Inputs

D'où viennent les données que l'agent consomme.

| Source | Type | Description | Obligatoire ? |
|---|---|---|---|
| `canvas.section.X` | yaml | … | ✅ |
| Brief humain initial | texte libre | … | ❌ |
| Output Agent Y | structuré | … | ✅ |

---

## 3. Outputs

Ce que l'agent produit. **Contrat strict** = schéma Pydantic + exemple JSON.

```python
from pydantic import BaseModel, Field
from typing import Literal

class AgentOutput(BaseModel):
    """Schéma de sortie de l'agent."""
    champ_a: str = Field(..., description="…")
    champ_b: list[str] = Field(default_factory=list)
    verdict: Literal["go", "no_go", "pivot"] | None = None
```

**Exemple JSON valide** :
```json
{
  "champ_a": "valeur",
  "champ_b": ["item1", "item2"],
  "verdict": "go"
}
```

**Section du Canvas YAML où l'output est écrit** : `canvas.section.Y`
(L'agent ne l'écrit pas directement — il soumet à l'Orchestrateur LangGraph qui valide et écrit.)

---

## 4. Outils

Ce dont l'agent a besoin pour faire son travail.

- [ ] **Web search** — sources : Reddit · HN · G2 · Product Hunt · Crunchbase · Failory · …
- [ ] **MCP Data.gouv** — données officielles FR (INSEE, marchés sectoriels)
- [ ] **MCP Notion** — pour les livrables
- [ ] **CLI** — Bandit / npm audit / Lighthouse / Playwright
- [ ] **Autres** — …

---

## 5. Prompt système

> 🟡 **Mode cadrage léger** : laisser cette section en TODO avec 3-5 bullets sur le contenu attendu.
> 🟢 **Mode spec exécutable** : prompt système complet rédigé, prêt à coller dans CrewAI.

```text
[Identité]
Tu es … . Tu travailles dans le système Solo Builder AI de Vincent.

[Mission]
…

[Sources autorisées]
…

[Règles non négociables]
- …
- …

[Format de sortie]
Tu retournes UNIQUEMENT du JSON valide respectant ce schéma : …
Pas de prose, pas de markdown, pas de commentaires hors JSON.

[Garde-fous]
- Si tu n'as pas assez d'éléments pour répondre → retourne {"verdict": "insuffisant", "raison": "…"}
- Tu ne dois jamais inventer de chiffres ou de sources.
```

---

## 6. Comportements attendus

- …
- …

## 7. Comportements à éviter

- …
- …

---

## 8. Critères de "fait" (Definition of Done)

- [ ] L'output respecte le schéma Pydantic (validation automatique)
- [ ] L'output est écrit dans la bonne section du Canvas YAML
- [ ] Toutes les sources externes sont citées avec URL + date
- [ ] …

---

## 9. Golden cases

Cas canoniques pour valider que l'agent fait son travail correctement.

### Cas 1 — [titre court]

**Input simulé** :
```yaml
…
```

**Output attendu (forme, pas mot pour mot)** :
```json
…
```

**3 critères d'acceptance** :
1. …
2. …
3. …

### Cas 2 — [titre court — choisir un cas différent du 1]

…

---

## 10. Gestion d'erreur & coût

| Item | Valeur |
|---|---|
| **Stratégie retry** | 1 tentative avec prompt légèrement modifié, puis escalade vers checkpoint humain |
| **Cas d'escalade immédiate** | Output non parsable / refus du LLM / sources injectables détectées |
| **Coût estimé** | ~X tokens entrée / ~Y tokens sortie par run |
| **Durée estimée** | ~Z secondes |
| **Fréquence d'exécution** | Une fois par phase / par sprint / event-driven |

---

## 11. Dépendances & ordre d'exécution

- **S'exécute après** : [agents qui doivent finir avant]
- **S'exécute avant** : [agents qui consomment son output]
- **Checkpoint humain en aval ?** : ✅ / ❌ — si oui, lequel

---

## 12. Notes & questions ouvertes

> Espace libre pour les points à clarifier, les TODO, les hypothèses non validées.

- [ ] …
