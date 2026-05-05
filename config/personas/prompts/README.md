# Compact Persona Prompts for LLM

**Purpose:** Token-efficient persona prompts for product selection agents.

---

## Why These Exist

The full persona files (SAVER_PERSONA.md, etc.) are **500-650 lines** and contain:
- Demographics and statistics
- Detailed mission breakdowns
- Background research
- All 8 mission scenarios

**Problem:** Too large for effective LLM prompting (15,000+ tokens per persona)

**Solution:** These compact prompts extract only the decision-making essentials:
- Decision framework (7-8 rules)
- NEVER/ALWAYS guardrails
- Key differentiators
- Sample decision scenarios
- **~60-80 lines / ~600-800 tokens** instead of 500+ lines

---

## Files

| File | Persona | Key Trait | Lines | Tokens (approx) |
|------|---------|-----------|-------|-----------------|
| **SAVER_PROMPT.md** | Budget-conscious families | Price first, unit pricing | ~60 | ~600 |
| **TRADITIONAL_PROMPT.md** | Brand-loyal, tried & tested | Familiar brands, consistency | ~75 | ~700 |
| **CONSCIOUS_PROMPT.md** | Health & values-driven | Ingredients, sustainability | ~80 | ~800 |
| **REFINED_PROMPT.md** | Quality over price | Premium, best quality | ~80 | ~800 |
| **ESSENTIAL_PROMPT.md** | Budget, own brand, basics | Cheapest, small baskets | ~70 | ~700 |

---

## Key Differentiators Preserved

Each compact prompt includes the **defining characteristics** that distinguish personas:

### SAVER
- Highest price sensitivity (86.3%)
- Highest promo engagement (53%)
- Yellow/red ticket focus
- Own brand default

### TRADITIONAL
- Largest segment (28%)
- Brand loyalty over price
- In-store preference
- Tried & tested brands

### CONSCIOUS
- Highest ingredient checking (60.8%)
- Highest dietary requirements (43.1%)
- Will pay more for health
- Sustainability matters

### REFINED
- Most affluent (Premium 2.2x)
- Lowest price sensitivity (45.9%)
- Premium quality always
- Own brand avoided

### ESSENTIAL
- Smallest baskets (20% smaller)
- Highest own brand adoption
- Lowest health concern (47.4%)
- Simple, price-driven

---

## Usage in Product Selection Agent

```python
# Load compact prompt
persona_prompt = load_file(f"config/personas/prompts/{persona_name}_PROMPT.md")

# Use in LLM call
prompt = f"""
{persona_prompt}

VENDOR: {vendor_name}
INGREDIENT NEEDED: {ingredient.description}

AVAILABLE PRODUCTS:
{format_product_catalog(products)}

Select the best product that matches this persona's priorities.
Explain your reasoning in 2-3 sentences.

Output format:
{{
    "product_id": "...",
    "reasoning": "..."
}}
"""
```

---

## Comparison: Full vs Compact

| Aspect | Full Persona Files | Compact Prompts |
|--------|-------------------|-----------------|
| **Lines** | 500-650 | 60-80 |
| **Tokens** | 15,000-20,000 | 600-800 |
| **Content** | Demographics, stats, all missions | Decision rules + differentiators only |
| **Use Case** | Reference, presentations, understanding | LLM prompting |
| **Context Window** | High impact (20% of context) | Minimal impact (1% of context) |
| **Decision Quality** | More context, may dilute focus | Focused on decision rules |

---

## Keep Both!

- **Full Persona Files** (`/config/personas/*.md`): Reference for team, board presentations, understanding customer segments
- **Compact Prompts** (`/config/personas/prompts/*.md`): Actual LLM prompting for product selection

---

*Created: May 5, 2026*  
*Extracted from full CREST persona files (500+ lines each)*  
*Optimized for token efficiency while preserving key decision markers*
