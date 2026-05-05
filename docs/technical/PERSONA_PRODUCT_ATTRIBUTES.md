# Product Attributes Required for CREST Personas
## Mystery Shopping Agent - Data Requirements

**Last Updated:** April 22, 2026  
**Source:** CREST Profiles - Group vf (April 2024)

---

## Overview

This document maps the specific product attributes needed in our catalog to support authentic persona-based decision-making for the Mystery Shopping Agent.

**Critical for MVP:** These attributes must be extractable from product names, descriptions, or enriched data to enable the LLM to make realistic shopping decisions.

---

## Main CREST Segments

### 1. CONSCIOUS (17% of customers)

**Who they are:** Younger customers (over-index Gen Z) who value health and sustainability. Balancing busy lives with better choices.

#### Required Product Attributes:

**Sustainability & Ethics (HIGH PRIORITY):**
- `organic` - Organic certification
- `free_range` - Free range (eggs, poultry)
- `rspca_approved` - RSPCA approved
- `grass_fed` - Grass fed (meat)
- `msc_certified` - MSC certified (seafood)
- `fair_trade` - Fair trade certification
- `sustainable` - Sustainably sourced
- `ethical` - Ethical brand markers

**Health & Dietary:**
- `gluten_free` - Gluten free
- `dairy_free` - Dairy free/lactose free
- `vegan` - Vegan
- `plant_based` - Plant based
- `no_added_sugar` - No added sugar
- `low_fat` - Low fat/reduced fat
- `high_protein` - High protein
- `no_additives` - No artificial additives/preservatives
- `whole_grain` - Whole grain

**Processing Level:**
- `fresh` - Fresh (not processed)
- `raw_ingredients` - Raw/unprocessed ingredients
- `minimal_processing` - Minimally processed

**Brand Values:**
- `brand_ethics` - Brand known for ethical practices
- `australian_made` - Australian made (moderate importance)

**Decision Logic:**
- Willing to pay more for health/sustainability (40% say they do)
- Research beforehand to maximize success (high ROBIS)
- Seek convenient solutions (e-com, rapid delivery)
- Balance price with values (not unlimited budget)

---

#### Sub-segment: DIETARY CHECKERS (6% of customers)

**Additional Attributes:**
- `specific_dietary_tags` - Caters to medical/lifestyle dietary needs
- `allergen_free` - Free from specific allergens
- `specialized_diet` - Keto, paleo, low-FODMAP, etc.
- `health_food` - Specialized health foods

**Decision Logic:**
- Health/wellbeing is fundamental to how they operate
- Highly over-index in Healthy Life and health aisle
- Willing to pay premium for dietary-specific needs
- Variety in options important

---

#### Sub-segment: FAST FUELER (11% of customers)

**Additional Attributes:**
- `high_protein` - High protein (very important)
- `convenience` - Quick/convenient preparation
- `ready_meals` - Ready-to-eat meals
- `protein_enriched` - Protein enriched products
- `meal_solutions` - Complete meal solutions

**Decision Logic:**
- Youngest segment - convenience crucial
- Value physical health but cooking is a chore
- Will pay more for convenient meal solutions
- High protein products popular
- Over-index dairy and gluten-free

---

### 2. REFINED (15% of customers)

**Who they are:** Affluent and older (60% premium affluence) who prioritize quality over price. Most financially stable segment.

#### Required Product Attributes:

**Quality Indicators (HIGHEST PRIORITY):**
- `premium` - Premium quality designation
- `select` - Select/finest range
- `gourmet` - Gourmet designation
- `artisan` - Artisan/craft production
- `imported` - Imported specialty items
- `award_winning` - Award-winning products

**Brand Status:**
- `well_known_brand` - Established, trusted brands (Leggos, Heinz, etc.)
- `premium_brand` - Premium brand positioning
- `specialty_brand` - Specialty/boutique brands
- `department_store_brand` - David Jones, Myer brands

**Product Type:**
- `fresh_unprocessed` - Fresh, unprocessed ingredients
- `specialty_ingredient` - Specialty/gourmet ingredients
- `entertaining` - Products for entertaining

**Processing & Origin:**
- `fresh` - Fresh (in-store bakery, butcher)
- `australian_made` - Australian made (moderate importance)
- `local_specialty` - Local specialty stores preference

**Pack Size & Presentation:**
- `quality_packaging` - Premium packaging indicators
- `single_serve_premium` - Premium single-serve options

**Decision Logic:**
- Quality is more important than price (61% vs 21% for price)
- Top of the line, best quality, no compromises
- Well-known products and brands preferred
- Less likely to engage with promotions
- Shop at local specialty stores and fresh markets (1.5x more likely)
- Prefer in-store shopping where good service valued
- Low digital adoption - use apps for lists/search only

---

### 3. ESSENTIAL (17% of customers)

**Who they are:** Non-family households (often singles/couples) seeking value. Prioritize price over quality, prepared to shop around.

#### Required Product Attributes:

**Value Indicators (HIGHEST PRIORITY):**
- `price` - Lowest price available
- `unit_price` - Unit price for comparison
- `own_brand` - Own brand/budget brand
- `value_range` - Value range designation
- `basic` - Basic/no-frills version

**Convenience:**
- `ready_meals` - Ready meals, frozen meals
- `convenience` - Easy to prepare
- `long_life` - Long shelf life pantry items
- `staple` - Staple/essential items

**Pack Size:**
- `small_pack` - Smaller pack sizes (smallest basket sizes - 20% smaller)
- `single_serve` - Single serve options

**Country of Origin:**
- `australian_made` - Australian made (preference, but not deciding factor)

**Decision Logic:**
- Prioritize price over quality (they focus on essentials)
- Buy own brand and look for ways to save
- Not overly savvy with saving strategies
- Stick to basics, smaller baskets
- High promotion engagement (53% sales on promo)
- Highest own brand penetration
- Digital usage lower, but use e-com when it helps

---

#### Sub-segment: TOP-UP SHOPPERS (6% of customers)

**Additional Attributes:**
- `ethnic_specialty` - Ethnic/gourmet specialty foods
- `international` - International food products
- `specialty_flour` - Specialty ingredients (flour, spices)
- `pantry_staple` - Pantry cleaning products (rice, oil)

**Decision Logic:**
- Non-family households skewing to budget
- Highly engaged on Yellow tickets (not Red)
- Shop around supermarkets for best value
- Frequent specialty stores for ethnic/value items
- Lower basket size, lower frequency

---

#### Sub-segment: COMFORT SEEKERS (11% of customers)

**Additional Attributes:**
- `frozen_meals` - Traditional frozen meals
- `comfort_food` - Comfort food items
- `long_life` - Long life pantry items
- `familiar_brands` - Familiar, trusted brands
- `easy_prep` - Easy to prepare

**Decision Logic:**
- Oldest segment - like things uncomplicated
- Value matters (financially challenged)
- Want reassured, comfort with familiar options
- Shop in store, low e-com penetration
- Seek value through Red tickets and OB (highest penetration)
- Like interaction where staff know them
- Frozen meals, traditional deli, long life pantry items

---

### 4. SAVERS (23% of customers - LARGEST SEGMENT)

**Who they are:** Young families (65% new & young families) balancing time and money. Must compromise on limited budget.

#### Required Product Attributes:

**Price & Promotions (CRITICAL):**
- `price` - Current price (must be accurate)
- `unit_price` - Unit price for comparison (CRITICAL)
- `promotion_type` - Yellow ticket / Red EDLP designation
- `promotion_description` - Promotion details
- `price_per_serve` - Price per serve for family planning
- `own_brand` - Own brand (high willingness)
- `value` - Value designation

**Pack Size (CRITICAL FOR DECISION):**
- `pack_size` - Exact pack size
- `pack_size_normalized` - Normalized to grams/ml for comparison
- `family_size` - Family pack/multi-pack
- `bulk_option` - Bulk buying option (if unit price cheaper)
- `serves` - Number of serves (for family meal planning)

**Family-Friendly:**
- `kid_friendly` - Kid friendly products
- `baby_food` - Baby food/formula
- `kids_snacks` - Kids snacks
- `family_meal` - Family meal solutions
- `healthy_kids` - Healthy options for kids

**Convenience:**
- `quick_prep` - Quick/easy to prepare
- `batch_cooking` - Suitable for batch cooking/freezing
- `freezer_friendly` - Freezer friendly

**Health (Secondary):**
- `healthy` - Healthy options (important but balanced with price)
- `reduced_sugar` - Reduced sugar (for kids)

**Decision Logic:**
- Pay close attention to price - seek specials and discounts
- Highest engagement with Yellow tickets (2nd on Red EDLP, OB)
- Employ value hacks more than other segments
- Use catalogues, compare prices, use loyalty
- Highest e-com penetration, use all WOW app functions
- Kids drive choices (keeping kids happy & healthy)
- Time is precious - seek value + convenience
- Will buy larger pack if unit price cheaper (for batch cooking)

---

#### Sub-segment: KID JUGGLERS (12% of customers)

**Additional Attributes:**
- `kids_meals` - Kid-friendly meals
- `easy_meal_prep` - Oven-ready, freezer meals
- `kids_lunchbox` - Lunchbox items
- `kids_yoghurt` - Kids yoghurt/dairy
- `convenience_meal` - Convenience meal solutions

**Decision Logic:**
- Time, budgeting, and responsibility juggling
- Research online, use e-com with pickup (no delivery fee)
- Frequent boosters of value hacks
- Savvy with saving, know how to find value
- Easy to prepare meal solutions important
- Kids items in baskets (yoghurts, berries, freezer meals)

---

#### Sub-segment: BASIC MEAL BUILDERS (11% of customers)

**Additional Attributes:**
- `school_lunch` - School lunch items
- `after_school_snacks` - After school snacks
- `budget_meal` - Budget meal ingredients
- `lunch_filler` - Lunch box fillers
- `soft_drink` - Soft drinks (high consumption)

**Decision Logic:**
- Most financially challenged segment
- Low price, great taste, easy to prepare, kids like it
- Highest budget + lowest premium affluence
- Pay for additional benefits (health, sustainability) if can save elsewhere
- Convenience important (working families, earn time back)
- Highly engaged red + yellow promotions, value hacks
- Seek cheap, easy foods (frozen meals, lunch fillers)

---

### 5. TRADITIONAL (28% of customers - 2ND LARGEST)

**Who they are:** Mature cohort (60% Boomers/Gen X) who value familiarity and community. Seek consistency.

#### Required Product Attributes:

**Brand Familiarity (CRITICAL):**
- `brand_name` - Well-known, trusted brand name
- `familiar_brand` - Brand they recognize and trust
- `established_brand` - Long-established brands
- `australian_brand` - Australian brand (moderate importance)

**Quality Consistency:**
- `quality` - Quality indicators
- `consistent` - Consistent quality/reliable
- `tried_tested` - Tried and tested products
- `well_known` - Well-known product

**Product Type:**
- `fresh_ingredients` - Fresh ingredients (meat, seafood)
- `raw_ingredients` - Raw/unprocessed for home cooking
- `traditional_format` - Traditional product formats
- `basic_ingredients` - Basic meal ingredients

**Origin:**
- `australian_made` - Australian made (preference)
- `local` - Supporting local

**Preparation:**
- `scratch_cooking` - Suitable for cooking from scratch
- `traditional_recipe` - Traditional recipe ingredients

**Decision Logic:**
- Consistency and certainty in choice (stick to tried & tested)
- Familiar choices they know well, can easily get and keep things uncomplicated
- Less inclined to shop online (prefer in-store with friendly faces)
- Quality matters - food should taste great, be made from fresh ingredients
- Delivering basics shouldn't be too much to ask
- Community matters - value connection to local
- Not as digitally engaged (but reasonably so)

---

#### Sub-segment: CLASSIC COOKS (21% of customers - LARGEST SUB-SEGMENT)

**Additional Attributes:**
- `raw_ingredients` - Raw ingredients for cooking
- `fresh_meat_seafood` - Fresh meat & seafood
- `deli_items` - Deli meats, bakery
- `cooking_ingredients` - Ingredients for scratch cooking
- `traditional_products` - Traditional Australian products

**Decision Logic:**
- Older segment, love to shop in store (familiar environment)
- Quality ahead of price (seek quality but not deliberate)
- Cooking is not a chore (confident cooks who cook from scratch)
- Engaged with EDR, relatively low WOW app usage
- Look for value but not as deliberate as other segments
- Large baskets, low e-com usage
- Prefer in-store for interaction, value connection
- Over-index processed deli meats, dairy, bakery

---

#### Sub-segment: SAVVY COOKS (7% of customers)

**Additional Attributes:**
- `natural` - Natural products (raw, fresh, unprocessed)
- `healthy` - Healthy/wholesome
- `quality_meat` - Quality bulk meat & staples
- `serviced_meat` - Serviced meat counter (roasts, steaks, offcuts)
- `seafood` - Seafood
- `asian_ingredients` - Asian foods

**Decision Logic:**
- Balance price and quality in approach to food
- Health important, manage it by eating fresh and exercising
- Most financially stable (in their view) but doesn't mean carefree
- Love to cook from scratch (reasonably confident cooks)
- Look for natural products (raw, fresh, unprocessed, healthy, high quality)
- Weekly specials highly engaged, more engaged with yellow
- Lowest e-com usage (prefer in-store, find shopping enjoyable)
- Seek weekly specials on quality products

---

## Product Attribute Schema for Database

Based on the above analysis, the `product_catalog` table needs these fields:

```sql
-- Core Product Info
product_id STRING
retailer STRING
name STRING
brand STRING
category STRING
subcategory STRING

-- Pricing (CRITICAL for Savers, Essential, Comfort Seekers)
current_price FLOAT64
unit_price FLOAT64  -- CRITICAL for Savers
promotion_type STRING  -- 'yellow_ticket', 'red_edlp', NULL
promotion_description STRING

-- Pack Size (CRITICAL for Savers)
pack_size STRING  -- e.g., "1kg", "500g"
pack_size_normalized FLOAT64  -- grams or ml for comparison
pack_size_unit STRING  -- 'g' or 'ml'
serves INT  -- estimated serves (for family planning)

-- Quality Indicators (CRITICAL for Refined, Conscious)
quality_indicators ARRAY<STRING>
  -- Examples:
  -- Quality: 'premium', 'select', 'gourmet', 'artisan'
  -- Sustainability: 'organic', 'free_range', 'rspca_approved', 
  --                 'grass_fed', 'msc_certified', 'fair_trade'
  -- Health: 'gluten_free', 'dairy_free', 'vegan', 'plant_based',
  --         'low_fat', 'high_protein', 'no_additives'
  -- Processing: 'fresh', 'unprocessed', 'raw_ingredients'
  -- Convenience: 'ready_meal', 'quick_prep', 'freezer_friendly'
  -- Family: 'kid_friendly', 'family_size'
  -- Origin: 'australian_made', 'imported'
  -- Brand tier: 'own_brand', 'budget', 'premium_brand', 'well_known_brand'

-- Brand Recognition (CRITICAL for Traditional, Refined)
brand_tier STRING  -- 'premium', 'mainstream', 'own_brand', 'budget'
brand_recognition STRING  -- 'well_known', 'specialty', 'emerging', 'unknown'

-- Product Characteristics
product_type STRING  -- 'fresh', 'frozen', 'pantry', 'chilled'
preparation_level STRING  -- 'ready_to_eat', 'ready_to_cook', 'raw_ingredient'

-- Availability
in_stock BOOLEAN
url STRING
```

---

## LLM Prompt Integration

When the LLM makes product selections, it needs access to these attributes formatted clearly:

### Example for Conscious customer:

```
You are a Conscious customer shopping for: Lean mince beef, 1kg

Your profile:
- You care about sustainability and health
- You look for: organic, free range, grass fed, ethical certifications
- You're willing to pay more for products that align with your values
- You prefer fresh, unprocessed ingredients

Available products:
| # | Product | Brand | Pack | Price | Unit $/kg | Quality Indicators | Promo |
|---|---------|-------|------|-------|-----------|-------------------|-------|
| 1 | Coles RSPCA Approved Grass Fed Beef Mince | Coles | 500g | $7.00 | $14.00 | grass_fed, rspca_approved | None |
| 2 | Woolworths Lean Beef Mince | Woolworths | 1kg | $12.00 | $12.00 | lean | None |
| 3 | Macro Organic Grass Fed Beef Mince | Macro | 500g | $9.50 | $19.00 | organic, grass_fed | None |

Which would you choose and why?
```

### Example for Saver customer:

```
You are a Saver customer shopping for: Lean mince beef, 1kg

Your profile:
- You're on a tight budget with young family to feed
- You actively look for Yellow tickets and Red EDLP promotions
- You'll buy own brand or switch brands to save money
- Unit price is critical - you'll buy larger pack if cheaper per kg
- You might batch cook and freeze to save money

Available products:
| # | Product | Brand | Pack | Price | Unit $/kg | Promo | Own Brand |
|---|---------|-------|------|-------|-----------|-------|-----------|
| 1 | Woolworths Lean Beef Mince | Woolworths | 1kg | $12.00 | $12.00 | Red EDLP | Yes |
| 2 | Coles Beef Mince | Coles | 1kg | $13.50 | $13.50 | None | Yes |
| 3 | Coles 3 Star Beef Mince | Coles | 900g | $10.00 | $11.11 | Yellow | Yes |
| 4 | Premium 5 Star Beef Mince | Brand | 500g | $9.00 | $18.00 | None | No |

Which would you choose and why?
```

---

## Data Extraction Priority

### Week 1 (Essential for MVP):
1. ✅ Price, unit price, promotion type
2. ✅ Pack size (normalized)
3. ✅ Brand name
4. ✅ Basic quality indicators from product name
5. ✅ Own brand flag

### Week 2 (Enhance decision quality):
6. ⚠️ Quality indicators (organic, free range, etc.) - **from name or enrichment**
7. ⚠️ Brand tier classification
8. ⚠️ Product type (fresh, frozen, etc.)

### Week 3 (Optional enhancement):
9. 🔄 Detailed sustainability certifications
10. 🔄 Health/dietary tags
11. 🔄 Preparation level classification

---

## Validation Against Feb 20 Baseline

For each persona test, validate that:

1. **Saver** selections show:
   - Preference for promotions (Yellow > Red > None)
   - Own brand acceptance
   - Unit price optimization
   - Larger packs if cheaper per unit

2. **Conscious** selections show:
   - Preference for quality indicators (organic, free range, etc.)
   - Willingness to pay premium
   - Sustainability certifications valued

3. **Refined** selections show:
   - Premium brand preference
   - Quality over price
   - Well-known brands
   - Fresh/specialty items

4. **Traditional** selections show:
   - Familiar brand consistency
   - Australian made preference
   - Quality matters
   - Traditional formats

5. **Essential** selections show:
   - Lowest price priority
   - Own brand acceptance
   - Value seeking
   - Smaller pack sizes

---

## Next Steps

1. **Day 1 PCB Assessment:** Check if product names contain quality indicators
2. **If names are sparse:** Plan selective enrichment for key categories
3. **Build extraction logic:** Parse quality indicators from names
4. **Test against baseline:** Validate that extracted attributes enable accurate persona decisions

