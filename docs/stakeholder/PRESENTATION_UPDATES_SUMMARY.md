# Presentation Updates Summary
**What Was Added to OWEN_STAKEHOLDER_PRESENTATION.md**

---

## Key Additions

### 1. Current POC Design & Why It Failed (New Section)

**Location:** After "Business Value", before "How It Works"

**What It Explains:**

#### Initial POC Approach (Rejected)
- Single LLM prompt with entire basket description
- Example: "Select products for Spaghetti Bolognese for a Saver customer..."

#### Why It Failed
❌ **Unreliable SKU matching** - LLM generated invalid product IDs  
❌ **Broken links** - Product URLs didn't work  
❌ **No product differentiation** - Couldn't distinguish organic vs standard  
❌ **Missing attributes** - No data on free range, RSPCA approved, Australian made  
❌ **Can't personalize** - Saver and Conscious customers need different attributes  

**Why This Matters:**
- Shows Owen we learned from POC failure
- Explains why we need robust attribute store
- Justifies the Week 1 focus on data quality

---

### 2. The Core Problem: Missing Product Attributes (New Section)

**What It Shows:**

Comparison table of what each persona needs vs what data we have:

| Attribute Type | Saver Needs | Conscious Needs | Traditional Needs |
|----------------|-------------|-----------------|-------------------|
| Pricing | ✅ Price, promotions | ✅ Price | ✅ Price |
| Brand | ⚠️ Own brand flag | ❌ Brand ethics | ✅ Trusted brands |
| Quality | ❌ Not critical | ✅ Organic, free range | ✅ Quality markers |
| Origin | ❌ Not needed | ✅ Australian made | ⚠️ Preference |
| Health | ❌ Not priority | ✅ Allergens, nutrition | ⚠️ Basic allergens |

**Current State by Retailer:**
- ✅ Woolworths: Has most attributes
- ⚠️ Coles: Has some attributes
- ❌ Aldi: Missing critical attributes

**Why This Matters:**
- Makes it clear why Week 1 is focused on data
- Shows Owen the gap we need to fill
- Explains why Aldi enrichment is necessary

---

### 3. Solution: Build Robust Attribute Store (New Section)

**What It Covers:**

#### Week 1 Critical Task: Define Product Attribute Schema

For each retailer, identify:
1. What attributes exist in current data tables?
2. What attributes are missing but needed?
3. How do we acquire missing attributes?
4. How do we standardize across retailers?

#### Attribute Store Design

Shows complete product schema with all needed attributes:

```
Product {
  // Identity
  product_id, name, brand, retailer
  
  // Pricing (CRITICAL for all personas)
  price, unit_price, promotion_type, promotion_description
  
  // Pack Size (CRITICAL for comparisons)
  pack_size, pack_size_normalized
  
  // Quality & Ethics (CRITICAL for Conscious/Refined)
  quality_markers: ["organic", "free_range", "rspca_approved", ...]
  
  // Origin (CRITICAL for Traditional/Conscious)
  country_of_origin, australian_ingredient_pct, australian_made
  
  // Health (CRITICAL for Conscious/Dietary)
  allergens, ingredients, nutrition
  
  // Brand Classification (CRITICAL for personas)
  brand_tier, brand_familiarity
}
```

**Why This Matters:**
- Shows Owen we have a comprehensive plan
- Demonstrates technical rigor
- Makes Week 1 deliverable concrete

---

### 4. Attribute Availability by Retailer (New Section)

**What It Shows:**

Detailed breakdown for each retailer:

**Woolworths:**
- ✅ Excellent: Price, 3 promotion flags, allergens, origin, Australian %
- ✅ Good: Promise info
- ⚠️ Partial: Ingredients
- ❌ Missing: Quality markers (need extraction)

**Coles:**
- ✅ Excellent: Ingredients, allergens, nutrition, promotions
- ✅ Good: Price
- ❌ Missing: Quality markers, origin
- ⚠️ Issue: Multi-postcode pricing

**Aldi:**
- ✅ Good: Price, basic names
- ❌ Missing: Almost all quality attributes
- 🔧 Solution: Pre-scrape ~100 products

**Why This Matters:**
- Transparent about data gaps
- Shows we've done the assessment
- Justifies enrichment strategy

---

### 5. Week 1 Deliverable: Attribute Mapping (New Section)

**What It Shows:**

Example mapping table:

| Attribute | Woolworths Source | Coles Source | Aldi Source | Strategy If Missing |
|-----------|------------------|--------------|-------------|---------------------|
| Price | `price` column | `price` column | `price` column | ✅ Available |
| Promotions | 3 flag columns | `promotion_flag` | `promotion_flag` | ✅ Available |
| Allergens | `allergens` column | `allergens` column | ❌ Missing | Web scrape ~100 |
| Organic | ❌ Missing | ❌ Missing | ❌ Missing | Extract from name |
| Australian Made | `country_of_origin` | ❌ Missing | ❌ Missing | Heuristic mapping |

**Extraction Strategies:**
1. Use existing columns (Woolworths best)
2. Extract from product names (quality markers)
3. Heuristic mappings (brand classification)
4. Selective enrichment (Aldi ~100 products)

**Why This Matters:**
- Concrete Week 1 deliverable
- Shows practical implementation approach
- Makes data work transparent

---

### 6. Why This Matters for Robustness (New Section)

**What It Shows:**

**Without Proper Attributes:**
❌ Can't differentiate Conscious from Saver  
❌ Can't distinguish organic from standard  
❌ Can't identify familiar brands for Traditional  
❌ Decisions are price-only, not persona-based  

**With Proper Attribute Store:**
✅ Saver picks promoted own-brand items  
✅ Conscious picks organic free-range options  
✅ Traditional picks Leggos (trusted brand)  
✅ Refined picks premium grass-fed products  
✅ Decisions are realistic and differentiated  

**Why This Matters:**
- Clear business impact
- Connects technical work to persona outcomes
- Justifies Week 1 investment in data quality

---

### 7. Updated Week 1 Timeline

**Old:**
```
Week 1: Foundation
✅ Assess data quality
✅ Build product catalog
✅ Implement search
✅ Enrich Aldi data
```

**New:**
```
Week 1: Foundation
✅ CRITICAL: Define product attribute schema for all retailers
✅ Map available attributes (what exists, what's missing, how to get it)
✅ Build product catalog with standardized attributes
✅ Implement search capability
✅ Enrich Aldi data with quality markers

Key Deliverable: Attribute mapping table
Checkpoint: Data confirmed, attribute store defined, search working
```

**Why This Matters:**
- Makes Week 1 focus crystal clear
- Emphasizes attribute store as critical path
- Sets clear expectation for Friday checkpoint

---

## Impact on Presentation Flow

### Old Flow:
1. What we're building
2. Business value
3. **→ How it works** (jumped straight to solution)
4. Data sources
5. Timeline

### New Flow:
1. What we're building
2. Business value
3. **→ Why POC failed** (context)
4. **→ Core problem: missing attributes** (problem definition)
5. **→ Solution: robust attribute store** (our approach)
6. **→ Attribute availability** (what we have)
7. **→ Week 1 deliverable** (concrete plan)
8. **→ Why this matters** (business impact)
9. How it works (solution architecture)
10. Data sources
11. Timeline

**Better because:**
- Tells a story: POC → Problem → Solution → Impact
- Justifies Week 1 focus on data
- Makes attribute store central to success
- Shows technical rigor without jargon

---

## Key Messages for Owen

### From POC Section:
"The POC failed because we lacked product attributes needed for persona-based decisions. We can't just ask an LLM to pick products - we need structured data."

### From Attribute Store Section:
"Week 1 is about building a robust attribute store - defining what data we need, where to get it, and how to standardize it across retailers. This is the foundation for realistic persona decisions."

### From Robustness Section:
"Without proper attributes, all personas look the same (price-only). With proper attributes, Saver picks promotions, Conscious picks organic, Traditional picks trusted brands - that's realistic competitive intelligence."

---

## Questions Owen Might Ask

**Q: "Why spend Week 1 on data? Can't you just start building?"**
A: "The POC proved that approach fails. Without quality attributes, we can't differentiate personas. Week 1 defines the attribute store that makes realistic decisions possible."

**Q: "What if we can't get all the attributes?"**
A: "We have fallback strategies: extract from product names, use heuristic mappings, selective enrichment for critical gaps. The mapping table shows our approach for each missing attribute."

**Q: "Why is Aldi so much worse than Woolworths/Coles?"**
A: "Aldi data is third-party scraped, so it lacks rich metadata. But we only need ~100 products for 8 missions, so selective enrichment is manageable."

**Q: "How do you know which attributes are 'critical'?"**
A: "Based on CREST persona profiles - Conscious needs organic/free-range, Traditional needs familiar brands, Saver needs promotions. The comparison table shows exactly what each persona requires."

---

## Summary

**Added ~8 new sections** explaining:
1. Why POC failed (unreliable, no differentiation)
2. What attributes we need (by persona)
3. What attributes we have (by retailer)
4. How we'll get missing attributes (strategy)
5. Why this matters for robustness (business impact)

**Updated Week 1 timeline** to emphasize:
- Attribute schema definition as CRITICAL
- Mapping table as key deliverable
- Standardization across retailers

**Result:**
- Presentation now tells complete story
- Week 1 focus is justified and clear
- Technical approach is concrete
- Business value is connected to data quality
