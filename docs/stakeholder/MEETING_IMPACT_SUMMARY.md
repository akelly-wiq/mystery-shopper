# April 22 Meeting - Impact on Implementation Plan

**Date:** April 22, 2026  
**Attendees:** Alexa, Daria  
**Status:** Implementation plan updated

---

## Key Findings from Meeting

### ✅ Data Sources Confirmed (Better than Expected!)

#### 1. Woolworths Data (via Stuart Contact)
**Status:** ✅ Available - much better than anticipated

**What we have:**
- Recent website information + historical data
- **Product attributes:** allergens, country of origin, Australian ingredient %
- **Promise information** (Woolworths Promise program)
- **Promotion flags:** 
  - Half price
  - Low price special
  - Price drop special

**Action needed:**
- ⚠️ Daria to confirm Mascot service center store ID (ensure not excluded from analytics tables)

**Impact on plan:**
- ✅ No enrichment needed for Woolworths
- ✅ Rich product attributes available (allergens crucial for Conscious/Dietary Checkers personas)
- ✅ Detailed promotion tracking supports Saver persona decisions

---

#### 2. Coles Data (Competitor Table via Nissan)
**Status:** ✅ Available - comprehensive coverage

**What we have:**
- Product details
- **Ingredients** (full ingredient lists)
- **Allergens**
- **Nutrition information**
- Promotional information

**Issue identified:**
- ⚠️ Multi-postcode pricing detected
- Some products have different prices across postcodes

**Action needed:**
- 🔍 Discuss with Owen (Friday): How to handle multi-postcode pricing?
  - Average price?
  - Median price?
  - Specific postcode?
  - Flag and handle separately?

**Impact on plan:**
- ✅ No enrichment needed for Coles
- ✅ Ingredients/allergens support quality-driven personas (Conscious, Refined)
- ⚠️ Need decision on price averaging before proceeding

---

#### 3. Aldi Data Strategy
**Status:** 🔄 Requires selective enrichment

**What we have:**
- Base competitor table (product names, prices, pack sizes)
- Product URLs

**What we need:**
- Product descriptions for ~100 products across 8 missions
- Allergen information (where available)
- Quality indicators (organic, free range, etc.)

**Enrichment Approach (UPDATED):**
- ❌ **NOT** on-the-fly scraping (Alexa's concern: runtime failure risk)
- ✅ **Pre-scrape** ~100 products before Week 2
- ✅ Cache results (one-time operation)
- ✅ Coordinate with pricing team (Nissan) on scraping methodology to avoid bot detection

**Action needed:**
- 📋 Daria to contact Nissan/pricing team about web scraping approach
- 📋 Identify which ~100 products from 8 missions need enrichment
- 📋 Complete enrichment before Week 2 (LLM decision engine)

**Impact on plan:**
- ✅ Reduced scope: Only ~100 products (not full catalog)
- ✅ Timeline preserved: Pre-scrape doesn't delay main development
- ✅ Risk mitigated: Pre-cached data avoids runtime failures

---

## Changes to Implementation Plan

### Architecture Updates

**OLD:** PCB competitor price tables (generic)

**NEW:** Three distinct data sources with different attributes

```
Woolworths Table (via Stuart)
├── Product attributes (allergens, country_of_origin, australian_ingredient_pct)
├── Promise information
└── Promotion flags (half_price, low_price_special, price_drop_special)

Coles Competitor Table (via Nissan)
├── Ingredients (full lists)
├── Allergens
├── Nutrition information
├── Promotional information
└── ⚠️ Multi-postcode pricing (decision needed)

Aldi Competitor Table
├── Base: Product names, prices, pack sizes
└── Enrichment: ~100 products pre-scraped
```

---

### Database Schema Changes

**Added fields to `product_catalog` table:**

```sql
-- Woolworths-specific
promotion_half_price BOOLEAN,
promotion_low_price_special BOOLEAN,
promotion_price_drop_special BOOLEAN,
allergens STRING,
country_of_origin STRING,
australian_ingredient_pct FLOAT64,
promise_info STRING,

-- Coles-specific
ingredients STRING,
nutrition_info STRING,
postcode STRING,  -- For tracking multi-postcode pricing

-- Aldi-specific
description STRING,  -- From enrichment
enrichment_date TIMESTAMP,  -- Track when enriched
```

---

### Week 1 Day 1 Changes

**OLD:** Query PCB tables and assess richness

**NEW:** Query all three data sources and assess quality

**New assessment focus:**
1. ✅ Woolworths: Verify Mascot service center store ID
2. ✅ Coles: Check for multi-postcode pricing variance
3. ✅ Aldi: Identify ~100 products needing enrichment

**New deliverables:**
- Data quality report for all 3 retailers
- Multi-postcode pricing analysis (for Owen decision)
- List of Aldi products to enrich

---

### Persona Decision-Making Enhancements

**Better support for personas based on new data:**

#### Conscious / Dietary Checkers
- ✅ Allergens available (Woolworths, Coles)
- ✅ Ingredients available (Coles)
- ✅ Country of origin (Woolworths)
- ✅ Australian ingredient % (Woolworths)

#### Savers
- ✅ Detailed promotion flags (Woolworths)
  - Can distinguish: half price vs low price special vs price drop
- ✅ Promotional descriptions (all retailers)

#### Refined
- ✅ Promise information (Woolworths quality program)
- ✅ Ingredients for quality assessment (Coles)
- ✅ Nutrition for quality products (Coles)

#### Traditional
- ✅ Australian ingredient % supports "Australian made" preference
- ✅ Country of origin supports local preference

---

## Action Items

### Immediate (This Week)

**Daria:**
- [ ] Confirm Mascot service center store ID with Stuart
- [ ] Contact Nissan about web scraping methodology for Aldi
- [ ] Prepare data overview showing available fields from all 3 sources
- [ ] Extract sample data for Day 1 quality assessment

**Alexa:**
- [ ] Prepare packet for Owen review (Friday)
- [ ] Tag Daria in Owen review materials
- [x] Update implementation plan with meeting outcomes

### Before Week 2

**Daria:**
- [ ] Identify ~100 Aldi products for 8 missions
- [ ] Coordinate scraping approach with pricing team
- [ ] Execute Aldi enrichment (pre-scrape)
- [ ] Cache enriched Aldi data

### Owen Review (Friday)

**Decisions needed:**
1. 🔍 Multi-postcode pricing strategy for Coles products
2. ✅ Confirm Mascot service center approach for Woolworths data
3. ✅ Approve Aldi selective enrichment scope (~100 products)

---

## Risk Mitigation

### Risks Addressed by Meeting Outcomes

| Risk | Original Concern | Resolution |
|------|------------------|------------|
| **Insufficient product attributes** | Might need extensive enrichment | ✅ Woolworths & Coles have rich attributes |
| **Runtime scraping failures** | On-the-fly scraping unreliable | ✅ Pre-scrape Aldi products |
| **Price accuracy** | Unknown data quality | ✅ Multiple promotion flags, can validate |
| **Persona decisions** | Missing allergens/ingredients | ✅ Available for WW & Coles |

### New Risks Identified

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Multi-postcode pricing** | Price comparison accuracy | Owen decision on averaging strategy |
| **Mascot store ID** | Limited product coverage | Confirm with Stuart before proceeding |
| **Aldi scraping** | Bot detection / failures | Coordinate with pricing team on methodology |

---

## Timeline Impact

**Original:** 4 weeks

**Updated:** Still 4 weeks ✅

**Why no delay:**
- Aldi enrichment (~100 products) can run in parallel with Week 1 setup
- Better data sources mean less overall enrichment work
- Pre-scraping approach doesn't block main development

**Schedule adjustment:**
- Week 1 Day 1: Expanded data assessment (all 3 sources)
- Week 1 Days 2-5: Aldi enrichment in parallel with semantic search dev
- Week 2: Proceed as planned (enriched Aldi data ready)

---

## Success Criteria Updates

**No changes to core metrics:**
- Search precision: >95% top-3 recall vs Feb 20 baseline
- Price accuracy: >98% within $0.50
- Persona decision quality: >90% alignment with expected behavior
- Coverage: >95% of mission items found across all retailers

**Enhanced validation:**
- ✅ Allergen accuracy (Conscious persona decisions)
- ✅ Promotion flag accuracy (Saver persona decisions)
- ✅ Ingredient/nutrition use (Refined/Conscious decisions)

---

## Summary

### What Changed
1. ✅ Three confirmed data sources (better than expected)
2. ✅ Rich product attributes available (allergens, ingredients, nutrition)
3. ✅ Detailed promotion tracking (3 separate flags for Woolworths)
4. 🔄 Aldi enrichment scoped to ~100 products (not full catalog)
5. ⚠️ Multi-postcode pricing issue identified (needs Owen decision)

### What Stayed the Same
1. ✅ 4-week timeline preserved
2. ✅ Semantic search approach unchanged
3. ✅ LLM persona decision engine unchanged
4. ✅ Success criteria unchanged

### Next Steps
1. Daria confirms data access details
2. Owen decides on multi-postcode pricing
3. Coordinate with pricing team on Aldi scraping
4. Execute Day 1 data quality assessment

---

**Status:** Plan updated and ready for execution ✅
