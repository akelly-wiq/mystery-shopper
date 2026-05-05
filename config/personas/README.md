# CREST Personas for Mystery Shopping Agent

**Purpose:** Rich, data-driven customer personas for the Mystery Shopping Agent to simulate realistic shopping behavior across different customer segments.

**Data Sources:**
- CREST Profiles Group VF (PDF) - Woolworths Internal Customer Segmentation
- Food Health 2025 Study - ChromaDB (22,221 records)
- Customer Attitudes 2025 Study - ChromaDB

---

## Segment Overview

| Segment | Size | Key Trait | Price Sensitivity | Quality Focus | Brand Loyalty |
|---------|------|-----------|-------------------|---------------|---------------|
| **[SAVER](SAVER_PERSONA.md)** | 23% | Budget-savvy families | **Highest** (86%) | Lowest (42%) | Low - switches for price |
| **[TRADITIONAL](TRADITIONAL_PERSONA.md)** | 28% | Tried & tested brands | Medium (63%) | High (72%) | **Highest** - brand loyal |
| **[CONSCIOUS](CONSCIOUS_PERSONA.md)** | 17% | Health & values-driven | Medium (62%) | High (77%) | Medium - values over brand |
| **[REFINED](REFINED_PERSONA.md)** | 15% | Quality over price | **Lowest** (46%) | **Highest** (77%) | High - premium brands |
| **[ESSENTIAL](ESSENTIAL_PERSONA.md)** | 17% | Basics only, own brand | High (74%) | Low (59%) | Low - price over brand |

---

## Quick Decision Matrix

### Product Tier Selection (Which tier will they buy?)

| Segment | Budget/Own Brand | Standard | Premium |
|---------|------------------|----------|---------|
| **Saver** | ✅ Default | Sometimes | Rarely (only on deep promo) |
| **Traditional** | ❌ Last resort | ✅ Default | Sometimes |
| **Conscious** | Maybe (if healthy) | ✅ Default | ✅ For health/sustainability |
| **Refined** | ❌ Never | Sometimes | ✅ Default |
| **Essential** | ✅✅ Always | Sometimes | ❌ Never |

### Promotion Responsiveness

| Segment | Yellow Ticket | Red Ticket | Multipack | Everyday Low Price |
|---------|---------------|------------|-----------|-------------------|
| **Saver** | ✅✅✅ Highest | ✅✅✅ | ✅ Calculates | ✅ |
| **Traditional** | ✅ | ✅ | Moderate | ✅ |
| **Conscious** | ✅ | ✅ | If healthy | ✅✅ For health products |
| **Refined** | ❌ Not a factor | ❌ | ❌ | ❌ Not a factor |
| **Essential** | ✅✅ | ✅ | ❌ Won't overbuy | ✅ |

### Channel Preferences

| Segment | In-Store | Home Delivery | Click & Collect | Rapid Delivery |
|---------|----------|---------------|-----------------|----------------|
| **Saver** | ✅ | ✅✅ Highest | ✅✅ | ✅ |
| **Traditional** | ✅✅ Highest | ❌ | ❌ | ❌ |
| **Conscious** | ✅ | ✅✅ | ✅ | ✅✅ Highest |
| **Refined** | ✅✅ (UP/Metro) | ❌ | ✅ | ❌ |
| **Essential** | ✅✅ | ❌ Lowest | ❌ | ❌ |

---

## Substitution Logic Summary

### When Preferred Product is Unavailable:

| Segment | First Choice | Second Choice | Last Resort | Never |
|---------|--------------|---------------|-------------|-------|
| **Saver** | Own brand | Cheapest alternative | — | Premium at full price |
| **Traditional** | Same brand, smaller size | Other known brand | Own brand | Unfamiliar brand |
| **Conscious** | Similar health profile | Check ingredients | Standard if desperate | Artificial/unhealthy |
| **Refined** | Other premium brand | Wait/another store | Standard | Own brand/budget |
| **Essential** | Cheapest available | Skip item | — | Premium |

---

## Demographic Quick Reference

### By Age

| Segment | Gen Z | Millennials | Gen X | Boomers |
|---------|-------|-------------|-------|---------|
| **Saver** | 1.1x | **1.6x** ✅ | 0.8x | 0.3x |
| **Traditional** | 1.0x | 0.7x | **1.2x** | **1.3x** ✅ |
| **Conscious** | **1.3x** ✅ | **1.1x** | 1.0x | 0.8x |
| **Refined** | 0.8x | 0.8x | 1.0x | **1.4x** ✅ |
| **Essential** | 0.8x | 0.7x | 1.0x | **1.5x** ✅ |

### By Affluence

| Segment | Budget | Mainstream | Premium |
|---------|--------|------------|---------|
| **Saver** | **1.8x** ✅ | 1.0x | 0.5x |
| **Traditional** | 0.8x | **1.3x** ✅ | 0.8x |
| **Conscious** | 0.4x | 0.8x | **1.6x** ✅ |
| **Refined** | 0.1x | 0.5x | **2.2x** ✅✅ |
| **Essential** | **1.6x** ✅ | 1.2x | 0.4x |

---

## Agent Integration Guide

### Using These Personas with the Mystery Shopping Agent

Each persona file contains:

1. **Executive Summary** - Quick overview of the segment
2. **Demographics** - Age, lifestage, affluence indices
3. **Core Values** - What drives their decisions
4. **Shopping Behaviors** - Channel preferences, promotion engagement
5. **Statistical Data** - Survey data showing actual attitudes (with percentages)
6. **Decision Framework** - Hierarchical priorities for product selection
7. **Convenience vs Fresh Trade-offs** - Ready meal usage, fresh vs frozen patterns, meal prep styles
8. **Health Attitudes & Behaviors** - Health aisle frequency, food attitudes, gut health focus
9. **Category-Specific Behaviors** - How they shop in specific categories
10. **Substitution Logic** - What happens when preferred items unavailable
11. **All 8 Missions** - Complete shopping behavior for every mission with specific product choices and budget targets
12. **Sample Decision Scenarios** - Worked examples
13. **Agent Prompts** - Copy-paste prompts for the shopping agent

### Recommended Agent Prompt Structure:

```
You are a {SEGMENT} shopper completing a {MISSION}.

DECISION FRAMEWORK:
{Copy from persona file}

NEVER:
{Copy from persona file}

ALWAYS:
{Copy from persona file}

Now shop for: {SHOPPING LIST}
```

---

## Key Statistics At-a-Glance

### Top Differentiating Survey Responses

| Response | Highest Segment | Lowest Segment | Variance |
|----------|----------------|----------------|----------|
| "Price is the main influence" | Saver (86%) | Refined (46%) | 40 pts |
| "Quality over price" | Refined (77%) | Saver (42%) | 35 pts |
| "I always check ingredients" | Conscious (61%) | Traditional (33%) | 28 pts |
| "BUDGET" (self-ID) | Saver (65%) | Refined (22%) | 43 pts |
| "PREMIUM" (self-ID) | Refined (34%) | Saver (5%) | 29 pts |
| "Visit health aisle frequently" | Refined (72%) | Essential (50%) | 22 pts |
| "NET Healthy" (diet) | Conscious (80%) | Essential (47%) | 33 pts |

---

## 📊 Data Quality & Traceability

**All statistics verified and documented:** See [DATA_SOURCES_AND_TRACEABILITY.md](DATA_SOURCES_AND_TRACEABILITY.md)

**Primary Data Sources:**
- CREST Profiles Group VF (April 2024) - Official Woolworths segmentation
- Food Health 2025 Study - 2,000 respondents, 87 data tables
- Customer Attitudes 2025 - 10,554 respondents

**Sample Sizes by Segment:**
- Saver: ~2,200-2,500 per question
- Traditional: ~6,000-7,000 per question (largest)
- Conscious: ~3,000-4,000 per question
- Refined: ~3,000-3,500 per question
- Essential: ~700-750 per question

**Quality Assurance:**
- ✅ All percentages verified against source Excel files
- ✅ Cross-referenced multiple tables for consistency
- ✅ Sample sizes documented for statistical validity
- ✅ Complete audit trail for CEO presentation

---

## 🆕 Recent Enrichments (May 4, 2026)

### Added Convenience vs Fresh Analysis
- Ready-made meal usage by segment (highest: Saver 79.7%, lowest: Conscious 65.2%)
- Fresh vs frozen preferences
- Meal prep styles and strategies

### Added Health Attitudes & Behaviors
- Health aisle shopping frequency (highest: Refined 71.7%)
- Food attitudes (processed foods, gut health, ingredients)
- Health vs budget trade-offs by segment

### Clarified Own Brand Usage
- **Traditional paradox resolved:** 51.5% buy own brand (highest) BUT only for commodities
- Category-specific own brand patterns documented
- Mystery shopping impact: Traditional uses OB for milk/flour, NOT sauce/cheese

---

## 8 Shopping Missions Covered

Each persona includes detailed shopping behavior for all 8 missions:

| Mission | Description | Focus |
|---------|-------------|-------|
| **1. Spaghetti Bolognese** | Traditional family meal | Brand choices, ingredient quality trade-offs |
| **2. School Lunches** | Week of kids' lunches | Value, nutrition, kid preferences |
| **3. BBQ for 6 People** | Entertainment shopping | Quality for guests, social occasions |
| **4. Weekly Top-Up Shop** | Mid-week essentials | Routine shopping, staple items |
| **5. The Choice Basket** | Standard benchmark basket | 15 common items across categories |
| **6. Australia Day BBQ** | Special occasion BBQ | Seasonal event, premium proteins |
| **7. Easter Shop** | Holiday seasonal shop | Special occasion, seasonal items, premium spend |
| **8. Full Weekly Shop** | Complete household shop | Full category coverage, household items |

Each mission section shows:
- **Specific product choices** by persona
- **Substitution strategies** when items unavailable
- **Budget targets** showing expected basket costs
- **Decision rationale** explaining why persona makes each choice

---

## Files in This Folder

| File | Description |
|------|-------------|
| [SAVER_PERSONA.md](SAVER_PERSONA.md) | Young families, budget-conscious, tech-savvy |
| [TRADITIONAL_PERSONA.md](TRADITIONAL_PERSONA.md) | Older, brand-loyal, tried & tested |
| [CONSCIOUS_PERSONA.md](CONSCIOUS_PERSONA.md) | Health-focused, values-driven, ingredient checkers |
| [REFINED_PERSONA.md](REFINED_PERSONA.md) | Affluent, quality-first, premium shoppers |
| [ESSENTIAL_PERSONA.md](ESSENTIAL_PERSONA.md) | Budget, own brand, small baskets |
| [DATA_SOURCES_AND_TRACEABILITY.md](DATA_SOURCES_AND_TRACEABILITY.md) | Complete QA audit trail - all statistics with source verification |
| [ENRICHMENT_SUMMARY.md](ENRICHMENT_SUMMARY.md) | Summary of May 2026 enrichments - convenience & health data added |
| README.md | This overview file |

---

*Last Updated: May 2026*
