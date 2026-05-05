# Mystery Shopping Agent
## Automated Competitive Intelligence System

**Stakeholder Review for Owen Lim**  
**Date:** April 25, 2026 (Friday)  
**Timeline:** 4 weeks (Apr 22 - May 19, 2026)  
**Presented by:** Alexa Kelly, Daria Volkova

---

## What We're Building

**AI-powered system that automatically shops like real customers**

Delivers daily competitive intelligence by comparing basket prices across:
- **8 shopping missions** (Spaghetti Bolognese, School Lunches, Weekly Shop, etc.)
- **5 customer personas** (CREST segments: Saver, Traditional, Conscious, Refined, Essential)
- **3 retailers** (Woolworths, Coles, Aldi)

**= 120 automated mystery shops available on demand**

---

## Business Value

### For the CEO:
✅ **Daily competitive intelligence** - How do our baskets compare on price?  
✅ **Customer perspective** - See pricing through different customer segment lenses  
✅ **Data-driven decisions** - Real insights for pricing and promotion strategy  
✅ **Automated & scalable** - No manual shopping required  

### Vs Current State:
❌ **Manual mystery shopping** (Feb 20) - one-time, labor intensive  
❌ **Not scalable** to daily monitoring  
❌ **Limited coverage** of personas and missions  

---

## Why the POC Approach Won't Work

### Initial POC Design (Rejected)

**Approach:** Single LLM prompt with entire basket description

**Example:**
```
"Select products for Spaghetti Bolognese recipe for a Saver customer.
Here are all the products from Woolworths, Coles, and Aldi..."
```

**Why This Failed:**
❌ **Unreliable SKU matching** - LLM generated invalid product IDs  
❌ **Broken links** - Product URLs didn't work  
❌ **No product differentiation** - Couldn't distinguish organic vs standard  
❌ **Missing attributes** - No data on free range, RSPCA approved, Australian made  
❌ **Can't personalize** - Saver and Conscious customers need different attributes  

---

## The Core Problem: Missing Product Attributes

### What We Need vs What We Have

**For Persona-Based Decisions, We Need:**

| Attribute Type | Saver Needs | Conscious Needs | Traditional Needs |
|----------------|-------------|-----------------|-------------------|
| **Pricing** | ✅ Price, promotions, unit price | ✅ Price | ✅ Price |
| **Brand** | ⚠️ Own brand flag | ❌ Brand ethics | ✅ Trusted/familiar brands |
| **Quality** | ❌ Not critical | ✅ Organic, free range, RSPCA | ✅ Consistent quality markers |
| **Origin** | ❌ Not needed | ✅ Australian made, sustainable | ⚠️ Australian made preference |
| **Health** | ❌ Not priority | ✅ Allergens, ingredients, nutrition | ⚠️ Basic allergens |

**Current State:**
- ✅ Woolworths: Has most attributes (allergens, origin, promotions)
- ⚠️ Coles: Has some (ingredients, allergens, nutrition)
- ❌ Aldi: Missing critical attributes (organic, free range, quality markers)

---

## Solution: Build Robust Attribute Store

### Week 1 Critical Task: Define Product Attribute Schema

**For Each Retailer, We Must Identify:**

1. **What attributes exist in current data tables?**
2. **What attributes are missing but needed?**
3. **How do we acquire missing attributes?**
4. **How do we standardize across retailers?**

---

### Attribute Store Design

**Core Product Attributes (All Retailers):**

```
Product {
  // Identity
  product_id: string
  name: string
  brand: string
  retailer: string
  
  // Pricing (CRITICAL for all personas)
  price: float
  unit_price: float  // per kg/L
  promotion_type: string  // "half_price", "yellow_ticket", "red_edlp"
  promotion_description: string
  
  // Pack Size (CRITICAL for comparisons)
  pack_size: string  // e.g., "1kg", "500g"
  pack_size_normalized: float  // grams or ml for comparison
  
  // Quality & Ethics (CRITICAL for Conscious/Refined)
  quality_markers: [
    "organic",
    "free_range", 
    "rspca_approved",
    "grass_fed",
    "msc_certified",
    "fair_trade"
  ]
  
  // Origin (CRITICAL for Traditional/Conscious)
  country_of_origin: string
  australian_ingredient_pct: float
  australian_made: boolean
  
  // Health (CRITICAL for Conscious/Dietary)
  allergens: [string]  // e.g., ["milk", "gluten"]
  ingredients: string  // full ingredient list
  nutrition: object    // nutritional information
  
  // Brand Classification (CRITICAL for personas)
  brand_tier: string  // "premium", "mainstream", "own_brand", "budget"
  brand_familiarity: string  // "well_known", "specialty", "emerging"
}
```

---

### Attribute Availability by Retailer

**Woolworths (via Stuart):**
- ✅ **Excellent:** Price, promotions (3 flags), allergens, origin, Australian %
- ✅ **Good:** Promise info (quality program)
- ⚠️ **Partial:** Ingredients (some products)
- ❌ **Missing:** Quality markers (organic, free range) - need extraction from names

**Coles (via Nissan):**
- ✅ **Excellent:** Ingredients, allergens, nutrition, promotions
- ✅ **Good:** Price data
- ❌ **Missing:** Quality markers, origin info - need extraction from descriptions
- ⚠️ **Issue:** Multi-postcode pricing (different prices across suburbs)

**Aldi:**
- ✅ **Good:** Price, basic product names
- ❌ **Missing:** Almost all attributes needed for quality personas
- 🔧 **Solution:** Pre-scrape descriptions for ~100 products in 8 missions

---

### Week 1 Deliverable: Attribute Mapping

**Critical Task:** Map available attributes for each retailer

**Example Mapping Table:**

| Attribute | Woolworths Source | Coles Source | Aldi Source | Strategy If Missing |
|-----------|------------------|--------------|-------------|---------------------|
| **Price** | `price` column | `price` column | `price` column | ✅ Available |
| **Promotions** | 3 flag columns | `promotion_flag` | `promotion_flag` | ✅ Available |
| **Allergens** | `allergens` column | `allergens` column | ❌ Missing | Web scrape ~100 |
| **Organic** | ❌ Missing | ❌ Missing | ❌ Missing | Extract from product name |
| **Free Range** | ❌ Missing | ❌ Missing | ❌ Missing | Extract from product name |
| **Australian Made** | `country_of_origin` | ❌ Missing | ❌ Missing | Heuristic mapping |
| **Brand Tier** | ❌ Missing | ❌ Missing | ❌ Missing | Custom classification |

**Strategy:**
1. **Use existing columns** where available (Woolworths best coverage)
2. **Extract from product names** for quality markers (organic, free range, premium)
3. **Heuristic mappings** for brand classification (own brand, trusted brands)
4. **Selective enrichment** for critical gaps (Aldi ~100 products)

---

### Why This Matters for Robustness

**Without Proper Attributes:**
❌ Can't differentiate Conscious from Saver (both pick cheapest)  
❌ Can't distinguish organic from standard (all look the same)  
❌ Can't identify familiar brands for Traditional customers  
❌ Decisions are price-only, not persona-based  

**With Proper Attribute Store:**
✅ Saver picks promoted own-brand items  
✅ Conscious picks organic free-range options  
✅ Traditional picks Leggos (trusted brand)  
✅ Refined picks premium grass-fed products  
✅ Decisions are realistic and differentiated  

---

## How It Works

### Simple 3-Step Process:

**STEP 1: Find Products**
- System searches for each ingredient (e.g., "lean mince beef, 1kg")
- Returns relevant product options from each retailer
- Includes: price, promotions, pack size, product attributes

**STEP 2: Make Decisions** 
- AI agents select products based on customer persona rules
- Example: Saver picks promoted items, Traditional picks familiar brands
- Multiple agents vote to ensure consistency

**STEP 3: Validate & Report**
- System validates basket (coverage, balance, total price)
- Cross-checks prices for accuracy
- Outputs comparison: Woolworths vs Coles vs Aldi

---

## Example Output

### Mission: Spaghetti Bolognese | Persona: Saver

| Ingredient | Woolworths | Coles | Aldi |
|------------|-----------|-------|------|
| Lean mince beef, 1kg | Own brand, Red EDLP $12.00 | 3-star, Yellow ticket $10.00 | Standard $11.50 |
| Pasta sauce, 500g | Leggos $3.50 | Own brand $2.80 | Casa Italia $2.99 |
| ... | ... | ... | ... |
| **TOTAL BASKET** | **$45.20** | **$41.30** | **$43.80** |

**Insight:** Coles basket is $3.90 cheaper for Saver customers on this mission

---

## Data Sources Confirmed (Apr 22)

### ✅ Better Than Expected

**Woolworths** (via Stuart):
- Product attributes: allergens, country of origin, Australian ingredient %
- Detailed promotion flags (half price, low price special, price drop)
- Promise program information

**Coles** (via Nissan):
- Ingredients, allergens, nutrition information
- Comprehensive promotional data
- ⚠️ Issue: Multi-postcode pricing (different prices across suburbs)

**Aldi**:
- Base product data available
- Need to enrich ~100 products with descriptions
- Coordinate with pricing team on methodology

---

## System Architecture

```
┌─────────────────────────────────────────┐
│     DATA LAYER                          │
│  • Woolworths internal tables           │
│  • Coles competitor data                │
│  • Aldi competitor data + enrichment    │
│                                          │
│  Weekly refresh from pricing team       │
└──────────────┬──────────────────────────┘
               ↓
┌──────────────────────────────────────────┐
│     PRODUCT SEARCH                       │
│  • Find relevant products per ingredient │
│  • Return top candidates with metadata   │
│  • Filter by price, pack size, promotions│
└──────────────┬──────────────────────────┘
               ↓
┌──────────────────────────────────────────┐
│     AI DECISION ENGINE                   │
│  • Multiple agents vote per ingredient   │
│  • Based on CREST persona rules          │
│  • Ensures consistent selections         │
└──────────────┬──────────────────────────┘
               ↓
┌──────────────────────────────────────────┐
│     VALIDATION & REPORTING               │
│  • Basket-level validation               │
│  • Price accuracy checks                 │
│  • Comparison dashboards                 │
│  • Analytics and insights                │
└──────────────────────────────────────────┘
```

---

## 4-Week Timeline

### Week 1: Foundation (Apr 22-28)
**Build the data and search foundation**

✅ **CRITICAL:** Define product attribute schema for all retailers  
✅ Map available attributes (what exists, what's missing, how to get it)  
✅ Build product catalog (~50K products) with standardized attributes  
✅ Implement product search capability  
✅ Enrich Aldi data (~100 products) with quality markers  

**Key Deliverable:** Attribute mapping table showing:
- What attributes each retailer has
- Extraction strategy for missing attributes (name parsing, heuristics, enrichment)
- Standardized schema across all 3 retailers

**Checkpoint:** Friday Apr 25 (today) - Data confirmed, attribute store defined, search working

---

### Week 2: AI Decision System (Apr 29-May 5)
**Build the agent decision-making capability**

✅ Implement CREST persona logic  
✅ Build multi-agent voting system  
✅ Test with 2 personas on sample missions  
✅ Validate decisions make sense  

**Checkpoint:** Friday May 2 - Agent decisions align with persona rules

---

### Week 3: Scale to Full Scope (May 6-12)
**Expand to all personas and missions**

✅ Implement all 5 CREST personas  
✅ Configure all 8 shopping missions  
✅ Run full matrix: 120 automated shops  
✅ Build basket-level validation  

**Checkpoint:** Friday May 9 - All 120 shops successful, >95% coverage

---

### Week 4: Production Ready (May 13-19)
**Validate, document, and go live**

✅ Human validation (compare agent vs manual shops)  
✅ Build analytics dashboards  
✅ Complete documentation  
✅ Stakeholder demo and handoff  

**Go-Live:** May 19, 2026

---

## Success Metrics

### Technical Quality

| Metric | Target |
|--------|--------|
| **Product Coverage** | >95% of ingredients found |
| **Price Accuracy** | >98% within $0.50 |
| **Decision Quality** | Personas make differentiated choices |
| **Execution Speed** | <5 minutes per mission |

### Business Outcomes

| Metric | Target |
|--------|--------|
| **Daily Shops Available** | 120 (8 missions × 5 personas × 3 retailers) |
| **Human Validation** | >85% agreement with expert judgment |
| **Insight Generation** | Clear patterns in competitive pricing |
| **Stakeholder Satisfaction** | CEO approval for production use |

---

## Decisions Needed from Owen

---

### Decision 1: Multi-Postcode Pricing (Coles)

**The Issue:**  
Some Coles products have different prices across Sydney suburbs

**Example:**
- Mascot (2020): $6.50
- Bondi (2026): $7.00  
- Parramatta (2145): $6.80

**Question:** Which price should we use for comparisons?

---

### Options for Multi-Postcode Pricing

**Option A: Use average price across suburbs**
- Simple, fair representation
- ❌ Not a real price any customer sees

**Option B: Use specific suburb (Mascot/Inner Sydney)** ← **RECOMMENDED**
- Real price from real store
- ✅ Matches actual customer experience
- ✅ Comparable to Woolworths Mascot location

**Option C: Flag these products separately**
- Transparent about variations
- ❌ More complex reporting

---

### Our Recommendation: Option B

**Use Mascot/Inner Sydney postcode for Coles pricing**

**Why:**
- Matches real customer shopping experience
- Comparable location to Woolworths data
- Simplest to implement and explain
- Inner Sydney demographics align with target personas

**What we need:** Your approval to proceed with this approach

---

### Decision 2: Aldi Data Enrichment

**The Issue:**  
Aldi base data lacks product descriptions needed for quality-focused personas

**Proposal:**
- Pre-scrape descriptions for ~100 Aldi products
- Focus on products in 8 missions only (not full catalog)
- Cache results (one-time enrichment, not daily scraping)
- Coordinate with pricing team on methodology

**Why we need this:**
- Conscious/Refined personas need quality attributes (organic, free range, etc.)
- Aldi product names alone don't have this detail
- Small scope (only ~100 products for MVP)

**What we need:** Your approval to proceed with selective enrichment

---

### Decision 3: Overall Plan Approval

**Confirming:**
- ✅ 4-week timeline (go-live May 19)
- ✅ Weekly Friday checkpoints at 4pm
- ✅ Scope: 8 missions × 5 personas × 3 retailers

**What we need:** Your sign-off to proceed

---

## Cost Estimate

### Development (4 weeks)
**~$70 total**
- Internal resource: 1 Data Scientist (Daria)
- External costs minimal (LLM API, cloud services)

### Ongoing Operations (Daily automation)
**~$125/month** (~$1,500/year)

| Component | Monthly Cost |
|-----------|-------------|
| AI API (Gemini) | ~$100 |
| Data storage (BigQuery) | ~$20 |
| Cloud infrastructure | ~$5 |

**Cost per shop:** ~$1 per automated mystery shop  
**Value:** Daily competitive intelligence vs manual mystery shopping

---

## Risk Management

### Key Risks & Mitigation

| Risk | Mitigation |
|------|------------|
| **4-week timeline tight** | Weekly go/no-go checkpoints |
| **Multi-postcode pricing** | Owen decision today |
| **Aldi enrichment** | Pre-scrape, pricing team coordination |
| **AI decision quality** | Multi-agent voting, human validation |
| **Data access issues** | Confirmed with Stuart/Nissan |

### Weekly Checkpoints

Clear criteria for proceeding or pausing:
- Week 1: Data quality >90%, search working
- Week 2: Persona logic validated
- Week 3: Full scope (120 shops) complete
- Week 4: Human validation >85%

---

## What Happens Next

### This Week (Apr 22-28)

**If approved today:**

✅ Daria continues catalog build  
✅ Coordinate Aldi enrichment with pricing team  
✅ Implement product search system  
✅ Weekly update Friday 4pm  

**Next checkpoint:** Friday May 2 (Week 2 review)

---

### Weekly Communication

**Every Friday 4pm:**
- Progress update email
- Metrics dashboard
- Blockers or decisions needed
- Go/no-go recommendation

**Escalation:**
- Minor issues: Daria → Alexa
- Major blockers: Immediate escalation to Owen

---

## Expected Outcomes

### By May 19 (Go-Live):

✅ **120 automated mystery shops** ready to run on demand  
✅ **Comparison dashboards** showing WW vs Coles vs Aldi  
✅ **Persona insights** (how different segments view our value)  
✅ **Analytics capability** for pricing strategy  

### Phase 2 (Post-MVP):

🔄 Daily automated execution  
🔄 Historical trending over time  
🔄 Scenario modeling (what-if analysis)  
🔄 Alert system for competitive changes  

---

## Summary

### What We're Delivering

**4-Week MVP:**
- AI system that shops like 5 different customer types
- 8 missions × 5 personas × 3 retailers = 120 shops
- Automated, scalable, consistent methodology
- CEO-ready competitive intelligence

**Business Value:**
- Daily basket price comparisons
- Customer segment perspectives
- Data-driven pricing insights
- Foundation for ongoing monitoring

**Investment:**
- 4 weeks development (~$70)
- ~$125/month ongoing operations
- High ROI vs manual mystery shopping

---

## Decisions Needed Today

### 1. Multi-Postcode Pricing
**Recommended:** Use Mascot/Inner Sydney postcode for Coles  
**Your decision:** Approve or alternative approach?

### 2. Aldi Enrichment  
**Recommended:** Pre-scrape ~100 products  
**Your decision:** Approve this scope?

### 3. Overall Plan
**Recommended:** Proceed with 4-week timeline  
**Your decision:** Sign-off to begin execution?

---

## Questions?

**Ready to answer:**
- Business value and ROI
- Technical approach (high level)
- Timeline and checkpoints
- Risk mitigation
- Cost estimates
- Phase 2 opportunities

**Next steps:**
1. Your decisions on pricing and enrichment
2. Weekly check-ins every Friday 4pm
3. Demo and go-live May 19

---

## Thank You

**Let's discuss your questions and get approval to proceed.**

---

# Appendix

---

## CREST Persona Overview

### Saver (23% of customers)
**Young families, budget-focused**
- Seeks promotions (Yellow/Red tickets)
- Willing to switch brands for savings
- Unit price is critical
- Will buy larger packs if cheaper per unit

### Traditional (28% of customers)  
**Mature, value familiarity**
- Sticks to known, trusted brands
- Quality and consistency matter
- Prefers Australian made
- Less promotion-driven

---

### Conscious (17% of customers)
**Younger, health & sustainability focused**
- Seeks organic, free range, RSPCA approved
- Checks ingredients and labels
- Willing to pay more for values
- Convenience important (e-com)

### Refined (15% of customers)
**Affluent, quality over price**
- Premium brands and specialty products
- Quality is non-negotiable
- Price is secondary
- Shops specialty stores

### Essential (17% of customers)
**Singles/couples, price conscious**
- Lowest price priority
- Own brand acceptance
- Smaller baskets, essentials only
- Ready meals, convenience

---

## Mission Examples

### 1. Spaghetti Bolognese
Lean mince beef, pasta, sauce, vegetables, parmesan, garlic bread, olive oil

### 2. School Lunches (1 week)
Bread, deli meats, cheese, fruit, snacks, yoghurt, juice, crackers

### 3. Weekly Shop
Full weekly grocery basket for family

### 4. BBQ
Sausages, meat, bread rolls, salads, drinks, condiments

### 5. Easy Midweek Dinner
Quick meal solution, 30-minute prep

### 6-8. Party Food, Quick Breakfast, Healthy Snacks
Various occasion-based baskets

---

## Contact Information

**Project Team:**
- **Alexa Kelly** - Project Lead
- **Daria Volkova** - Technical Implementation

**Data Sources:**
- **Stuart** - Woolworths internal tables
- **Nissan** - Coles/Aldi competitor data

**Questions before go-live?**  
Reach out to Alexa or Daria

---

**End of Presentation**
