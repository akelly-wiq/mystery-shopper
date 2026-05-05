# Mystery Shopping Agent - FINAL Implementation Plan
## CEO-Grade Solution Using PCB Price Tables

**Last Updated:** April 22, 2026  
**Timeline:** 4 weeks  
**Data Source:** PCB competitor price tables + Woolworths internal tables (primary)  

---

## Executive Summary

**Approach:** Build AI agent using existing data tables for product/pricing data, with semantic search and LLM-based persona decision-making.

**Key Findings from Apr 22 Meeting:**
- ✅ Woolworths data table available (via Stuart) with product attributes, allergens, promotion flags
- ✅ Coles competitor table (via Nissan) with ingredients, allergens, nutrition, promotions
- ⚠️ Aldi requires selective enrichment (~100 products across 8 missions)
- 🔍 Decision needed: Price averaging strategy for multi-postcode products

**Success Criteria:**
- Search precision: >95% top-3 recall vs Feb 20 baseline
- Price accuracy: >98% within $0.50
- Persona decision quality: >90% alignment with expected behavior
- Coverage: >95% of mission items found across all retailers

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                     DATA SOURCE LAYER                            │
│                                                                  │
│  Woolworths Data (via Stuart contact):                          │
│  - Recent website information with historical data              │
│  - Product attributes: allergens, country of origin,            │
│    Australian ingredient %, Promise info                        │
│  - Promotion flags: half price, low price special,              │
│    price drop special                                           │
│  - Store: Default location TBD (check Mascot service center)    │
│                                                                  │
│  Coles Data (Competitor table via Nissan):                      │
│  - Product details, ingredients, allergens, nutrition           │
│  - Promotional information                                      │
│  - Web scrape (30 stores, weekly refresh)                       │
│                                                                  │
│  Aldi Data Strategy:                                            │
│  - Base: PCB competitor table (in-store scrape, ~99% accuracy)  │
│  - Enrichment: ~100 products for 8 missions only                │
│  - Method: Pre-scrape product descriptions (not on-the-fly)     │
│  - Coordinate with pricing team on scraping approach            │
│                                                                  │
│  Refresh: Weekly (Fridays)                                      │
│  Coverage: ~50,000 products across NSW                          │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│                  PRODUCT INTELLIGENCE LAYER                      │
│                                                                  │
│  ┌────────────────────────────────────────────────────┐        │
│  │  Unified Product Catalog (BigQuery)                │        │
│  │                                                      │        │
│  │  - Product ID, name, brand, category               │        │
│  │  - Pack size (normalized to grams/ml)              │        │
│  │  - Current price, unit price                       │        │
│  │  - Promotion flags: half_price, low_price_special, │        │
│  │    price_drop_special (Woolworths)                 │        │
│  │  - Product attributes: allergens, country_of_origin,│       │
│  │    australian_ingredient_pct, Promise info          │        │
│  │  - Ingredients, nutrition (Coles)                  │        │
│  │  - Quality indicators (extracted or enriched)      │        │
│  │  - Vector embedding (semantic search)              │        │
│  └────────────────────────────────────────────────────┘        │
│                                                                  │
│  ┌────────────────────────────────────────────────────┐        │
│  │  Semantic Search Engine                            │        │
│  │                                                      │        │
│  │  - Vertex AI text-embedding-004                    │        │
│  │  - BigQuery ML vector similarity                   │        │
│  │  - Multi-stage retrieval & reranking               │        │
│  └────────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│                  PERSONA DECISION ENGINE                         │
│                                                                  │
│  ┌────────────────────────────────────────────────────┐        │
│  │  Gemini 1.5 Pro with Structured Prompts           │        │
│  │                                                      │        │
│  │  Input: Persona rules + Product candidates         │        │
│  │  Output: Selection + Reasoning + Confidence        │        │
│  │                                                      │        │
│  │  Personas: Saver, Traditional, Conscious,          │        │
│  │            Refined, Essential                       │        │
│  └────────────────────────────────────────────────────┘        │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│              VALIDATION & QUALITY ASSURANCE                      │
│                                                                  │
│  - Multi-source price validation                                │
│  - Ground truth comparison (Feb 20 baseline)                    │
│  - Confidence scoring on all decisions                          │
│  - Automated flagging of anomalies                              │
└─────────────────────────────────────────────────────────────────┘
                            │
                            ↓
┌─────────────────────────────────────────────────────────────────┐
│                  DATA PERSISTENCE (BigQuery)                     │
│                                                                  │
│  - raw_shops: Every product selection with metadata             │
│  - validation_logs: Price validation results                    │
│  - decision_audit: LLM reasoning trail                          │
│  - analytics_views: Comparative basket summaries                │
└─────────────────────────────────────────────────────────────────┘
```

---

## Key Decisions & Action Items (Apr 22 Meeting)

### ✅ Data Sources Confirmed

**Woolworths Data (via Stuart):**
- Table contains: recent website info, historical data, Promise information
- Product attributes available: allergens, country of origin, Australian ingredient %
- Promotion flags: half price, low price special, price drop special
- **Action:** Daria to confirm Mascot service center store ID (ensure not excluded from analytics)

**Coles Data (Competitor table via Nissan):**
- Comprehensive: product details, ingredients, allergens, nutrition, promotional info
- Better than expected - includes rich product metadata

**Aldi Data:**
- Base table available but needs enrichment for ~100 products (8 missions)
- **Action:** Daria to coordinate with pricing team (Nissan) on web scraping methodology
- **Decision:** Pre-scrape descriptions (not on-the-fly) to avoid runtime failures

### 🔍 Open Decisions for Owen (Friday Review)

1. **Price Averaging Strategy:**
   - Question: When a product has different prices across postcodes in competitor table, should we:
     - Use average price?
     - Use median price?
     - Use price from specific postcode (which one)?
     - Flag as multi-price and handle separately?
   - **Impact:** Affects price accuracy metrics and competitor comparisons

2. **Mascot Service Center Location:**
   - Confirm if default location has store ID in data tables
   - Risk: May exclude products from analytics if not properly tracked

3. **Aldi Enrichment Scope:**
   - Confirmed: ~100 products across 8 missions (not full catalog)
   - Method: Pre-scrape before Week 2, cache results
   - Coordinate scraping approach with pricing team to avoid bot detection

### 📋 Next Steps

**Daria's Actions:**
- [ ] Double-check Mascot service center store ID with Stuart
- [ ] Contact Nissan/pricing team about web scraping methods for Aldi
- [ ] Prepare data overview document showing available fields
- [ ] Compile sample data for Day 1 richness assessment

**Alexa's Actions:**
- [ ] Prepare packet for Owen review (Friday)
- [ ] Tag Daria in Owen review materials
- [ ] Update implementation plan with meeting outcomes

---

## Week 1: Foundation & Critical Decision Point

### Day 1 (Tuesday, April 22): Data Assessment & Setup

**UPDATED:** Based on Apr 22 meeting, we now have confirmed data sources. Focus shifts to data quality assessment.

**CRITICAL TASK: Assess Available Data Quality & Richness**

This determines the enrichment strategy for Aldi products.

#### Morning: Query All Data Sources

**1. Woolworths Table (via Stuart):**
```sql
-- Query Woolworths internal table
SELECT 
    product_id,
    product_name,
    brand,
    category,
    subcategory,
    pack_size,
    price,
    unit_price,
    allergens,
    country_of_origin,
    australian_ingredient_pct,
    promise_info,
    promotion_half_price,
    promotion_low_price_special,
    promotion_price_drop_special,
    store_id  -- VERIFY Mascot service center has ID
FROM `woolworths.product_catalog_table`  -- Table name TBD by Stuart
WHERE store_id = 'MASCOT_ID'  -- Confirm with Stuart
AND category IN ('Meat', 'Dairy', 'Eggs', 'Produce', 'Pantry')
LIMIT 200
```

**2. Coles Competitor Table (via Nissan):**
```sql
-- Query Coles competitor data
SELECT 
    product_id,
    product_name,
    brand,
    category,
    subcategory,
    pack_size,
    price,
    unit_price,
    ingredients,
    allergens,
    nutrition_info,
    promotion_flag,
    promotion_description,
    product_url,
    price_date,
    postcode  -- CHECK for multi-postcode pricing
FROM `your-project.pcb_data.coles_competitor_table`
WHERE state = 'NSW'
AND deleted = 'n'
AND price_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
AND category IN ('Meat', 'Dairy', 'Eggs', 'Produce', 'Pantry')
ORDER BY category, price DESC
LIMIT 200
```

**3. Aldi Competitor Table:**
```sql
-- Query Aldi base data
SELECT 
    product_id,
    product_name,
    brand,
    category,
    pack_size,
    price,
    unit_price,
    product_url,
    price_date
FROM `your-project.pcb_data.aldi_competitor_table`
WHERE state = 'NSW'
AND deleted = 'n'
AND category IN ('Meat', 'Dairy', 'Eggs', 'Produce', 'Pantry')
ORDER BY category, price DESC
LIMIT 200
```

**Inspect Results:**
```python
# Analysis script
def assess_data_quality(ww_df, coles_df, aldi_df):
    """
    Assess data quality across all three retailers
    Focus: Quality indicators, multi-postcode pricing, enrichment needs
    """
    
    quality_keywords = [
        'organic', 'free range', 'rspca approved', 'grass fed',
        'premium', 'select', 'extra lean', 'gluten free',
        'no added hormones', 'msc certified', 'fair trade'
    ]
    
    # === WOOLWORTHS ASSESSMENT ===
    print("=" * 60)
    print("WOOLWORTHS DATA QUALITY")
    print("=" * 60)
    
    ww_stats = {
        'total': len(ww_df),
        'has_allergens': ww_df['allergens'].notna().sum(),
        'has_country_origin': ww_df['country_of_origin'].notna().sum(),
        'has_aus_ingredient_pct': ww_df['australian_ingredient_pct'].notna().sum(),
        'has_promotions': (
            ww_df['promotion_half_price'] | 
            ww_df['promotion_low_price_special'] | 
            ww_df['promotion_price_drop_special']
        ).sum()
    }
    
    print(f"Total products: {ww_stats['total']}")
    print(f"Has allergens: {ww_stats['has_allergens']/ww_stats['total']:.1%}")
    print(f"Has country of origin: {ww_stats['has_country_origin']/ww_stats['total']:.1%}")
    print(f"Has Australian %: {ww_stats['has_aus_ingredient_pct']/ww_stats['total']:.1%}")
    print(f"Has promotions: {ww_stats['has_promotions']/ww_stats['total']:.1%}")
    
    # === COLES ASSESSMENT ===
    print("\n" + "=" * 60)
    print("COLES DATA QUALITY")
    print("=" * 60)
    
    # Check for multi-postcode pricing
    coles_multi_price = coles_df.groupby('product_id').agg({
        'price': ['count', 'min', 'max', 'mean', 'std']
    }).reset_index()
    
    multi_price_products = coles_multi_price[
        coles_multi_price[('price', 'count')] > 1
    ]
    
    print(f"Total products: {len(coles_df['product_id'].unique())}")
    print(f"Multi-postcode pricing: {len(multi_price_products)} products")
    if len(multi_price_products) > 0:
        print(f"  ⚠️  ACTION NEEDED: Decide on price averaging strategy")
        print(f"  Sample variance:")
        for _, row in multi_price_products.head(5).iterrows():
            pid = row['product_id']
            min_price = row[('price', 'min')]
            max_price = row[('price', 'max')]
            mean_price = row[('price', 'mean')]
            print(f"    Product {pid}: ${min_price:.2f} - ${max_price:.2f} (avg ${mean_price:.2f})")
    
    coles_stats = {
        'total': len(coles_df),
        'has_ingredients': coles_df['ingredients'].notna().sum(),
        'has_allergens': coles_df['allergens'].notna().sum(),
        'has_nutrition': coles_df['nutrition_info'].notna().sum(),
        'has_quality': 0
    }
    
    for _, product in coles_df.iterrows():
        name = str(product['product_name']).lower()
        if any(kw in name for kw in quality_keywords):
            coles_stats['has_quality'] += 1
    
    print(f"Has ingredients: {coles_stats['has_ingredients']/coles_stats['total']:.1%}")
    print(f"Has allergens: {coles_stats['has_allergens']/coles_stats['total']:.1%}")
    print(f"Has nutrition: {coles_stats['has_nutrition']/coles_stats['total']:.1%}")
    print(f"Quality indicators in name: {coles_stats['has_quality']/coles_stats['total']:.1%}")
    
    # === ALDI ASSESSMENT ===
    print("\n" + "=" * 60)
    print("ALDI DATA QUALITY & ENRICHMENT NEEDS")
    print("=" * 60)
    
    aldi_stats = {
        'total': len(aldi_df),
        'has_quality': 0,
        'needs_enrichment': []
    }
    
    for _, product in aldi_df.iterrows():
        name = str(product['product_name']).lower()
        has_quality = any(kw in name for kw in quality_keywords)
        
        if has_quality:
            aldi_stats['has_quality'] += 1
        else:
            if product['category'] in ['Meat', 'Dairy', 'Eggs']:
                aldi_stats['needs_enrichment'].append({
                    'id': product['product_id'],
                    'name': product['product_name'],
                    'category': product['category'],
                    'url': product.get('product_url')
                })
    
    print(f"Total products: {aldi_stats['total']}")
    print(f"Quality indicators in name: {aldi_stats['has_quality']/aldi_stats['total']:.1%}")
    print(f"Products needing enrichment: {len(aldi_stats['needs_enrichment'])}")
    
    # === ENRICHMENT RECOMMENDATION ===
    print("\n" + "=" * 60)
    print("ENRICHMENT STRATEGY")
    print("=" * 60)
    
    # For MVP: only enrich ~100 Aldi products across 8 missions
    print(f"✅ Woolworths: No enrichment needed (attributes available)")
    print(f"✅ Coles: No enrichment needed (ingredients/allergens available)")
    
    if aldi_stats['has_quality'] / aldi_stats['total'] < 0.30:
        recommendation = "SELECTIVE_ENRICHMENT_ALDI"
        print(f"⚠️  Aldi: Selective enrichment needed")
        print(f"   Target: ~100 products for 8 missions")
        print(f"   Method: Pre-scrape descriptions (coordinate with pricing team)")
        print(f"   Categories to prioritize: Meat, Dairy, Eggs")
    else:
        recommendation = "NO_ENRICHMENT_NEEDED"
        print(f"✅ Aldi: Product names sufficiently rich")
    
    return recommendation, multi_price_products

# Execute assessment
ww_data = query_woolworths_table()
coles_data = query_coles_table()
aldi_data = query_aldi_table()

decision, multi_price_df = assess_data_quality(ww_data, coles_data, aldi_data)
```

**Decision Point (UPDATED Apr 22):**

| Data Source | Strategy | Rationale |
|-------------|----------|-----------|
| **Woolworths** | Use internal table directly | Has product attributes, allergens, promotion flags |
| **Coles** | Use competitor table directly | Has ingredients, allergens, nutrition, promotions |
| **Aldi** | Selective enrichment (~100 products) | Base table + pre-scraped descriptions for 8 missions |

**Key Decisions for Owen (Friday):**

1. ⚠️ **Multi-postcode pricing** - How to handle products with different prices across postcodes?
2. ⚠️ **Mascot service center** - Confirm store ID in data tables
3. ✅ **Aldi enrichment** - Pre-scrape ~100 products (coordinate with pricing team)

**END OF DAY 1 CHECKPOINT:** Review data quality assessment, confirm enrichment strategy for Aldi.

#### Afternoon: Setup Infrastructure

**Regardless of enrichment decision, setup core infrastructure:**

```bash
# 1. GCP Project Setup
gcloud config set project your-project-id

# 2. Enable APIs
gcloud services enable \
    bigquery.googleapis.com \
    aiplatform.googleapis.com \
    storage.googleapis.com

# 3. Create BigQuery Dataset
bq mk --dataset \
    --location=australia-southeast1 \
    --description="Mystery Shopper Agent Data" \
    mystery_shopper

# 4. Create tables
bq mk --table mystery_shopper.product_catalog schema/catalog.json
bq mk --table mystery_shopper.raw_shops schema/shops.json
bq mk --table mystery_shopper.validation_logs schema/validation.json
bq mk --table mystery_shopper.price_history schema/price_history.json
```

**Database Schema (UPDATED Apr 22):**

```sql
-- product_catalog table
CREATE TABLE mystery_shopper.product_catalog (
    product_id STRING NOT NULL,
    retailer STRING NOT NULL,
    name STRING NOT NULL,
    brand STRING,
    category STRING,
    subcategory STRING,
    
    -- Pack size
    pack_size STRING,
    pack_size_normalized FLOAT64,  -- grams or ml
    pack_size_unit STRING,  -- 'g' or 'ml'
    
    -- Pricing
    current_price FLOAT64,
    unit_price FLOAT64,
    price_date DATE,
    postcode STRING,  -- For tracking multi-postcode pricing (Coles)
    
    -- Promotions (Woolworths specific flags)
    promotion_half_price BOOLEAN,
    promotion_low_price_special BOOLEAN,
    promotion_price_drop_special BOOLEAN,
    promotion_type STRING,  -- Generic: 'yellow_ticket', 'red_edlp', etc.
    promotion_description STRING,
    
    -- Product Attributes (Woolworths)
    allergens STRING,  -- Comma-separated or JSON
    country_of_origin STRING,
    australian_ingredient_pct FLOAT64,
    promise_info STRING,  -- Woolworths Promise information
    
    -- Product Details (Coles)
    ingredients STRING,
    nutrition_info STRING,  -- JSON or structured text
    
    -- Metadata
    quality_indicators ARRAY<STRING>,  -- ['organic', 'free range']
    description STRING,  -- Rich text (if enriched)
    in_stock BOOLEAN,
    url STRING,
    
    -- Search
    search_document STRING,  -- Text used for embedding
    embedding ARRAY<FLOAT64>,  -- Vector embedding
    
    -- Audit
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    data_source STRING  -- 'pcb_table' or 'pcb_table_enriched'
);

-- Create indexes
CREATE INDEX idx_retailer_category 
ON mystery_shopper.product_catalog(retailer, category);
```

---

### Day 2 (Tuesday, April 22): Build Product Catalog

**Path A: PCB Only (if richness >50%)**

```python
# src/data/catalog_builder.py

class ProductCatalogBuilder:
    """
    Build unified product catalog from PCB tables
    """
    
    def __init__(self):
        self.bq_client = bigquery.Client()
        self.embedding_model = TextEmbeddingModel.from_pretrained("text-embedding-004")
    
    def build_catalog(self):
        """
        Main catalog build process
        """
        print("Step 1: Extract from PCB tables...")
        products = self.extract_from_pcb()
        
        print("Step 2: Normalize pack sizes...")
        products = self.normalize_pack_sizes(products)
        
        print("Step 3: Extract quality indicators from product names...")
        products = self.extract_quality_indicators(products)
        
        print("Step 4: Generate embeddings...")
        products = self.generate_embeddings(products)
        
        print("Step 5: Save to BigQuery...")
        self.save_to_bigquery(products)
        
        print(f"✅ Catalog built: {len(products)} products")
        return products
    
    def extract_from_pcb(self):
        """
        Extract products from PCB competitor tables
        """
        query = """
        WITH competitor_products AS (
            -- Get Coles and Aldi from PCB tables
            SELECT 
                competitor_product_id as product_id,
                retailer,
                product_name as name,
                brand,
                category,
                subcategory,
                pack_size,
                price as current_price,
                unit_price,
                promotion_flag as promotion_type,
                promotion_description,
                price_date,
                in_stock,
                product_url as url
            FROM `your-project.pcb_data.competitor_price_index_view`
            WHERE retailer IN ('Coles', 'Aldi')
            AND state = 'NSW'
            AND deleted = 'n'
            AND price_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
            
            UNION ALL
            
            -- Get competitor-only products (no WW match)
            SELECT 
                competitor_product_id as product_id,
                retailer,
                product_name as name,
                brand,
                category,
                subcategory,
                pack_size,
                price as current_price,
                unit_price,
                promotion_flag as promotion_type,
                promotion_description,
                price_date,
                in_stock,
                product_url as url
            FROM `your-project.pcb_data.competitor_only_view`
            WHERE retailer IN ('Coles', 'Aldi')
            AND state = 'NSW'
            AND deleted = 'n'
        ),
        
        woolworths_products AS (
            -- Get Woolworths from internal catalog
            SELECT 
                product_id,
                'Woolworths' as retailer,
                name,
                brand,
                category,
                subcategory,
                pack_size,
                current_price,
                unit_price,
                promotion_type,
                promotion_description,
                CURRENT_DATE() as price_date,
                in_stock,
                url
            FROM `your-project.woolworths_catalog.products`
            WHERE active = TRUE
        )
        
        SELECT * FROM competitor_products
        UNION ALL
        SELECT * FROM woolworths_products
        ORDER BY retailer, category, name
        """
        
        df = self.bq_client.query(query).to_dataframe()
        print(f"Extracted {len(df)} products")
        print(f"  Woolworths: {len(df[df.retailer == 'Woolworths'])}")
        print(f"  Coles: {len(df[df.retailer == 'Coles'])}")
        print(f"  Aldi: {len(df[df.retailer == 'Aldi'])}")
        
        return df
    
    def extract_quality_indicators(self, products):
        """
        Extract quality indicators from product names
        (No web scraping needed if names are rich)
        """
        quality_keywords = {
            'organic': ['organic', 'bio'],
            'free_range': ['free range', 'free-range'],
            'rspca_approved': ['rspca approved', 'rspca'],
            'grass_fed': ['grass fed', 'grass-fed', 'pasture raised'],
            'premium': ['premium', 'select', 'finest', 'signature'],
            'lean': ['lean', 'extra lean', 'trim'],
            'gluten_free': ['gluten free', 'gluten-free'],
            'vegan': ['vegan', 'plant based', 'plant-based'],
            'sustainable': ['msc certified', 'sustainable', 'responsibly sourced'],
            'hormone_free': ['no added hormones', 'hormone free']
        }
        
        def extract_indicators(product_name):
            if not product_name:
                return []
            
            name_lower = product_name.lower()
            indicators = []
            
            for indicator, keywords in quality_keywords.items():
                if any(kw in name_lower for kw in keywords):
                    indicators.append(indicator)
            
            return indicators
        
        products['quality_indicators'] = products['name'].apply(extract_indicators)
        
        # Stats
        has_indicators = products['quality_indicators'].apply(len) > 0
        pct = has_indicators.sum() / len(products) * 100
        print(f"Quality indicators found in {pct:.1f}% of products")
        
        return products
    
    def generate_embeddings(self, products):
        """
        Generate vector embeddings for semantic search
        """
        # Create search document (text to embed)
        def create_search_doc(row):
            parts = [
                row['name'],
                row['brand'] or '',
                row['category'] or '',
                ' '.join(row.get('quality_indicators', []))
            ]
            return ' '.join(filter(None, parts))
        
        products['search_document'] = products.apply(create_search_doc, axis=1)
        
        # Generate embeddings in batches
        embeddings = []
        batch_size = 250
        
        for i in range(0, len(products), batch_size):
            batch = products['search_document'].iloc[i:i+batch_size].tolist()
            batch_embeddings = self.embedding_model.get_embeddings(batch)
            embeddings.extend([emb.values for emb in batch_embeddings])
            
            if (i // batch_size + 1) % 10 == 0:
                print(f"  Generated {i + len(batch)} / {len(products)} embeddings...")
        
        products['embedding'] = embeddings
        
        return products
```

**Aldi Selective Enrichment (UPDATED Apr 22)**

Based on meeting decision: Enrich ~100 Aldi products for 8 missions only.

```python
# Aldi-specific enrichment for MVP missions

class AldiProductEnricher:
    """
    Enrich specific Aldi products for 8 MVP missions
    Pre-scrape approach (not on-the-fly) to avoid runtime failures
    Coordinate with pricing team on scraping methodology
    """
    
    def __init__(self, missions_config):
        """
        missions_config: List of 8 missions with their item lists
        """
        self.missions = missions_config
        self.pricing_team_method = None  # To be confirmed with Nissan
    
    def identify_products_to_enrich(self, aldi_products):
        """
        Identify ~100 Aldi products that match mission items
        """
        # Extract all unique mission items across 8 missions
        mission_items = set()
        for mission in self.missions:
            mission_items.update(mission['items'])
        
        print(f"Total mission items: {len(mission_items)}")
        
        # Match Aldi products to mission items
        products_to_enrich = []
        
        for item in mission_items:
            # Simple keyword matching (will be improved with semantic search)
            item_keywords = item.lower().split()
            
            for _, product in aldi_products.iterrows():
                product_name = product['name'].lower()
                
                # Check if product matches mission item
                if any(kw in product_name for kw in item_keywords):
                    products_to_enrich.append({
                        'product_id': product['product_id'],
                        'product_name': product['name'],
                        'mission_item': item,
                        'url': product['url'],
                        'category': product['category']
                    })
        
        # Deduplicate by product_id
        unique_products = pd.DataFrame(products_to_enrich).drop_duplicates('product_id')
        
        print(f"Products to enrich: {len(unique_products)}")
        print(f"Categories: {unique_products['category'].value_counts()}")
        
        return unique_products
    
    def enrich_aldi_products(self, products_to_enrich):
        """
        Scrape Aldi product descriptions using pricing team methodology
        Pre-scrape all ~100 products, cache results
        """
        print(f"Enriching {len(products_to_enrich)} Aldi products...")
        
        # COORDINATION WITH PRICING TEAM
        # Action: Daria to ask Nissan about scraping approach
        # Need: Method to avoid bot detection
        # Timeline: Complete before Week 2 (LLM decision engine)
        
        enriched = []
        failed = []
        
        for idx, product in products_to_enrich.iterrows():
            if idx % 10 == 0:
                print(f"  Enriched {idx} / {len(products_to_enrich)}...")
            
            try:
                # Use pricing team's scraping method (TBD)
                description, allergens, attrs = self.scrape_aldi_product(
                    product['url']
                )
                
                # Extract quality indicators
                quality_indicators = self.extract_quality_indicators(
                    description, product['product_name']
                )
                
                enriched.append({
                    'product_id': product['product_id'],
                    'description': description,
                    'allergens': allergens,
                    'quality_indicators': quality_indicators,
                    'enrichment_date': pd.Timestamp.now()
                })
                
            except Exception as e:
                print(f"  ⚠️  Failed to enrich {product['product_id']}: {e}")
                failed.append({
                    'product_id': product['product_id'],
                    'error': str(e)
                })
                continue
        
        enriched_df = pd.DataFrame(enriched)
        
        print(f"\n✅ Successfully enriched: {len(enriched)}/{len(products_to_enrich)}")
        print(f"❌ Failed: {len(failed)}")
        
        # Cache results to avoid re-scraping
        enriched_df.to_csv('data/aldi_enriched_products.csv', index=False)
        
        return enriched_df
    
    def scrape_aldi_product(self, url):
        """
        Scrape Aldi product using pricing team methodology
        PLACEHOLDER - to be replaced with actual method from Nissan
        """
        import time
        import requests
        from bs4 import BeautifulSoup
        
        # Rate limiting - coordinate with pricing team on acceptable rate
        time.sleep(0.5)
        
        # Use pricing team's headers/method to avoid bot detection
        response = requests.get(url, headers={
            'User-Agent': 'TBD_from_pricing_team'  # Coordinate with Nissan
        })
        
        if response.status_code != 200:
            raise Exception(f"HTTP {response.status_code}")
        
        soup = BeautifulSoup(response.content, 'html.parser')
        
        # Extract description
        # Selectors to be confirmed based on Aldi website structure
        description = soup.find('div', class_='product-description')
        allergens = soup.find('div', class_='allergen-info')
        
        return (
            description.get_text() if description else '',
            allergens.get_text() if allergens else '',
            {}  # Additional attributes
        )
    
    def extract_quality_indicators(self, description, product_name):
        """
        Extract quality indicators from description and product name
        """
        quality_keywords = {
            'organic': ['organic', 'bio'],
            'free_range': ['free range', 'free-range'],
            'grass_fed': ['grass fed', 'grass-fed'],
            'premium': ['premium', 'select', 'finest'],
            'lean': ['lean', 'extra lean'],
            'gluten_free': ['gluten free', 'gluten-free'],
            'vegan': ['vegan', 'plant based']
        }
        
        text = f"{product_name} {description}".lower()
        
        indicators = []
        for indicator, keywords in quality_keywords.items():
            if any(kw in text for kw in keywords):
                indicators.append(indicator)
        
        return indicators

# Execute Aldi enrichment
# Note: Run this AFTER Day 1 assessment, BEFORE Week 2
if __name__ == "__main__":
    # Load missions config
    missions = load_missions_config()  # 8 missions
    
    # Load Aldi products from base table
    aldi_products = load_aldi_base_products()
    
    # Identify products to enrich
    enricher = AldiProductEnricher(missions)
    products_to_enrich = enricher.identify_products_to_enrich(aldi_products)
    
    # Coordinate with pricing team first!
    print("⚠️  BEFORE RUNNING: Coordinate with Nissan on scraping methodology")
    
    # Enrich products
    enriched = enricher.enrich_aldi_products(products_to_enrich)
```

**Key Differences from Original Approach:**
- ✅ **Scope:** Only ~100 products (not full catalog)
- ✅ **Timing:** Pre-scrape before Week 2 (not on-the-fly)
- ✅ **Method:** Coordinate with pricing team (Nissan) on scraping approach
- ✅ **Risk mitigation:** Pre-cache to avoid runtime failures

---

### Day 3 (Wednesday, April 23): Semantic Search

```python
# src/agents/semantic_product_search.py
# (Same as in REVISED_IMPLEMENTATION_PLAN.md)

class SemanticProductSearch:
    """
    Production-grade semantic search using vector embeddings
    """
    
    def search(self, query, retailer, category=None, 
               pack_size_target=None, max_results=20):
        """
        Multi-stage search:
        1. Generate query embedding
        2. Vector similarity search (BigQuery ML)
        3. Metadata filtering
        4. Reranking
        """
        # Implementation as before
        pass
```

---

### Day 4 (Thursday, April 24): Ground Truth Validation

```python
# tests/test_search_precision.py

class TestSearchPrecision:
    """
    Validate against Feb 20 manual baseline
    Success criteria: >95% top-3 recall
    """
    
    @pytest.fixture
    def feb20_baseline(self):
        """
        Load Feb 20 manual shop data
        What humans actually bought
        """
        return [
            {
                'mission_item': 'Lean Mince beef, 1kg',
                'retailer': 'Woolworths',
                'expected_product_name': 'Woolworths Lean Beef Mince',
                'expected_price': 12.00,
                'expected_pack_size': '1kg',
                'shopper': 'Person 1'
            },
            # ... all items from Feb 20 manual shops
        ]
    
    def test_top3_recall(self, feb20_baseline):
        """
        Test: Correct product in top 3 results?
        Target: >95%
        """
        searcher = SemanticProductSearch()
        correct = 0
        total = len(feb20_baseline)
        
        for item in feb20_baseline:
            results = searcher.search(
                query=item['mission_item'],
                retailer=item['retailer']
            )
            
            # Check if expected product in top 3
            top3_names = [r.name for r in results[:3]]
            if item['expected_product_name'] in top3_names:
                correct += 1
            else:
                print(f"❌ {item['mission_item']}")
                print(f"   Expected: {item['expected_product_name']}")
                print(f"   Got: {top3_names}")
        
        recall = correct / total
        print(f"\nTop-3 Recall: {recall:.1%} ({correct}/{total})")
        
        assert recall >= 0.95, f"Recall {recall:.1%} below 95% threshold"
```

---

### Day 5 (Friday, April 25): Week 1 Checkpoint

**Deliverables:**

1. ✅ Product catalog built (all 3 retailers)
2. ✅ Semantic search operational
3. ✅ Ground truth validation passing (>95% top-3 recall)
4. ✅ Full Spaghetti Bolognese mission searchable
5. ✅ Documentation complete

**Demo to Stakeholders:**
- Show search precision metrics
- Demo semantic search ("lean beef mince" → relevant products)
- Show price accuracy validation
- Confirm path for Week 2

**GO/NO-GO Decision:**
- ✅ GO if all metrics passing
- ⚠️ NO-GO if recall <90% → debug over weekend

---

## Week 2: LLM Decision Engine & Persona Logic

### Day 1-2 (Monday-Tuesday, April 28-29): Persona Prompt Engineering

```python
# config/personas.yaml

personas:
  saver:
    name: "Saver"
    profile: |
      These customers are time and cash poor. Typically younger families with
      limited incomes. Budget-focused and will stick to it.
    
    decision_rules:
      price_priority: "highest"
      promotion_response: "actively seeks yellow/red tickets"
      brand_flexibility: "high - will switch for savings"
      pack_size_logic: "choose minimum needed OR larger if unit price cheaper for batch cooking"
      quality_threshold: "acceptable quality, not premium"
    
    prompt_template: |
      You are a Saver customer shopping for: {mission_item}
      
      Your Profile:
      - You're on a tight budget and need to save money
      - You actively look for promotions (yellow tickets, red EDLP)
      - You'll buy own brand or switch brands to save money
      - Pack size matters: you need the minimum quantity to feed your family,
        but will buy larger if the unit price is cheaper (for batch cooking/freezing)
      - Quality needs to be acceptable, but you prioritize price over premium
      
      Available products:
      {format_candidates_table(candidates)}
      
      Which product would you choose and why?
      
      Think step-by-step:
      1. Which products are on promotion? (promotions are very appealing to you)
      2. What's the unit price? (critical for comparison)
      3. Does the pack size meet your needs? (minimum quantity required)
      4. Is there a cheaper alternative that's "good enough"?
      
      Return JSON:
      {{
        "selected_product_id": "...",
        "reasoning": "Step-by-step explanation of your choice",
        "confidence": 0.0-1.0,
        "alternative_if_unavailable": "..."
      }}

  conscious:
    name: "Conscious"
    profile: |
      Conscious of ingredients, environmental impact, and brand values.
      Younger singles/couples or families. Lead busy lives but try to balance
      with better choices (health, environment, social outcomes).
    
    decision_rules:
      quality_priority: "high"
      sustainability: "important - look for certifications"
      health_focus: "check labels, prefer fresh ingredients"
      price_sensitivity: "medium - will pay more for values alignment"
      brand_ethics: "important"
    
    prompt_template: |
      You are a Conscious customer shopping for: {mission_item}
      
      Your Profile:
      - You care about what you put in your body and impact on environment
      - You look for quality indicators: organic, free range, RSPCA approved,
        sustainable certifications (MSC, Fair Trade)
      - You check labels and prefer products with clear ingredient lists
      - You'll pay more for products that align with your values
      - When budget is tight, you prioritize, but quality matters
      
      Available products:
      {format_candidates_table(candidates)}
      
      Which product would you choose and why?
      
      Think step-by-step:
      1. Which products have quality certifications? (organic, free range, etc.)
      2. Which brands align with my values?
      3. Are there health benefits? (low fat, high protein, no additives)
      4. Is the price reasonable for the quality?
      
      Return JSON:
      {{
        "selected_product_id": "...",
        "reasoning": "Step-by-step explanation prioritizing quality/values",
        "confidence": 0.0-1.0,
        "alternative_if_unavailable": "..."
      }}
```

```python
# src/agents/persona_agent.py

class PersonaDecisionEngine:
    """
    LLM-based decision engine for persona shopping behavior
    """
    
    def __init__(self, persona_config):
        self.persona = persona_config
        self.llm = genai.GenerativeModel('gemini-1.5-pro')
    
    def select_product(self, mission_item, candidates):
        """
        Given candidates, select product based on persona
        """
        # Build prompt
        prompt = self.persona['prompt_template'].format(
            mission_item=mission_item,
            candidates=self.format_candidates(candidates)
        )
        
        # Call LLM with structured output
        response = self.llm.generate_content(
            prompt,
            generation_config={
                'temperature': 0.3,  # Consistent but nuanced
                'response_mime_type': 'application/json'
            }
        )
        
        # Parse response
        decision = json.loads(response.text)
        
        # Validate
        selected = self.find_product(decision['selected_product_id'], candidates)
        
        return {
            'product': selected,
            'reasoning': decision['reasoning'],
            'confidence': decision['confidence'],
            'alternative': decision.get('alternative_if_unavailable')
        }
    
    def format_candidates(self, candidates):
        """
        Format candidates as markdown table for LLM
        """
        table = "| # | Product | Brand | Pack Size | Price | Unit Price | Promotion |\n"
        table += "|---|---------|-------|-----------|-------|------------|----------|\n"
        
        for i, c in enumerate(candidates, 1):
            promo = c.promotion_type or 'None'
            if c.promotion_description:
                promo += f" ({c.promotion_description})"
            
            table += f"| {i} | {c.name} | {c.brand} | {c.pack_size} | "
            table += f"${c.price:.2f} | ${c.unit_price:.2f}/kg | {promo} |\n"
        
        return table
```

### Day 3-5: Integration & Testing

```python
# src/agents/mission_executor.py

class MissionExecutor:
    """
    Execute full mission: search → LLM decision → validation
    """
    
    def execute_mission(self, mission, persona, retailer):
        """
        Execute complete mission for given persona at retailer
        """
        results = []
        
        for item in mission['items']:
            print(f"\nShopping for: {item}")
            
            # Stage 1: Semantic search
            candidates = self.searcher.search(
                query=item,
                retailer=retailer,
                max_results=10
            )
            
            if not candidates:
                print(f"  ❌ No products found")
                results.append({
                    'item': item,
                    'status': 'not_found',
                    'product': None
                })
                continue
            
            # Stage 2: Persona decision
            decision = self.persona_engine.select_product(item, candidates)
            
            # Stage 3: Validate price
            validation = self.validator.validate_price(
                retailer,
                decision['product'].product_id,
                decision['product'].price
            )
            
            # Stage 4: Store result
            result = {
                'mission_name': mission['name'],
                'persona': persona['name'],
                'item': item,
                'retailer': retailer,
                'product': decision['product'],
                'reasoning': decision['reasoning'],
                'llm_confidence': decision['confidence'],
                'price_confidence': validation['confidence'],
                'status': 'success'
            }
            
            results.append(result)
            self.save_to_bigquery(result)
            
            print(f"  ✅ Selected: {decision['product'].name}")
            print(f"     Price: ${decision['product'].price:.2f}")
            print(f"     Confidence: {decision['confidence']:.2f}")
        
        return results
```

**Week 2 Success Criteria:**
- [ ] Saver persona: >90% alignment with Feb 20 baseline
- [ ] Traditional persona: >85% alignment  
- [ ] 2 full missions completed successfully
- [ ] All decisions have reasoning + confidence scores

---

## Week 3: Scale to All Missions & Personas

(Continue with all 8 missions, 5 personas, 3 retailers = 120 shops)

---

## Week 4: Polish & Production Readiness

(Validation, reporting, documentation, handoff)

---

## Decision Tree Summary

```
Day 1: Query PCB Tables
    │
    ├─> Product names rich (>50% quality indicators)?
    │   │
    │   ├─> YES: Path A - PCB Only
    │   │   └─> Week 1-4: Build with PCB data
    │   │       └─> 4 weeks total ✅
    │   │
    │   └─> NO: Path B/C - Need Enrichment
    │       └─> Week 1-2: Build with PCB data
    │           └─> Week 3 Day 1-3: Enrich via web scraping (one-time)
    │               └─> Week 3-4: Complete build
    │                   └─> 4.5 weeks total ✅
    │
    └─> Ongoing: Always use PCB for daily price updates
                 (no continuous web scraping)
```

---

## Cost Summary

**Week 1:**
- Embeddings: $0.50 (one-time)
- BigQuery: $5 (storage + queries)
- Optional web scraping: $0-50 (if needed)
- **Total: $5-55**

**Ongoing (Monthly):**
- Embeddings refresh: $0.50
- BigQuery: $20
- Gemini API: $100
- **Total: ~$120/month**

---

## Success Metrics (CEO-Ready)

**Week 1 Checkpoint:**
- Search precision: 95%+ top-3 recall ✅
- Price accuracy: 98%+ within $0.50 ✅
- Coverage: 95%+ items found ✅

**Week 2 Checkpoint:**
- Persona alignment: 90%+ for Saver ✅
- Decision quality: All have reasoning ✅
- 2 missions completed ✅

**Week 4 Final:**
- 8 missions × 5 personas × 3 retailers = 120 shops ✅
- Stakeholder approval ✅
- Production deployment plan ✅

---

*This plan reflects the PCB table decision with Day 1 richness check as critical decision point.*
