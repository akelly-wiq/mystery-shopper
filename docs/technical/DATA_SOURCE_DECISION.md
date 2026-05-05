# Data Source Strategy: Price Tables vs Web Scraping

## DECISION: Use PCB Price Tables as Primary Source

**Rationale:** The existing PCB competitor price tables are already validated, trusted by Commercial for price index reporting, and have ~99% accuracy for in-store data. Web scraping introduces unnecessary risk, complexity, and maintenance burden for a CEO-level project.

---

## Comparison Analysis

| Factor | PCB Price Tables | Web Scraping |
|--------|-----------------|--------------|
| **Accuracy** | ~99% (validated by PCB team) | Unknown, needs validation |
| **Reliability** | High (weekly refresh, proven process) | Low (sites change, blocking, rate limits) |
| **Legal Risk** | None (internal data) | Medium-High (ToS violations, blocking) |
| **Maintenance** | None (PCB team owns it) | High (constant scraper updates) |
| **Speed to Build** | Fast (data already exists) | Slow (2-3 weeks per retailer) |
| **Data Freshness** | Weekly (Fridays) | Real-time |
| **CEO Trust** | High (already used for price index) | Low (unproven, needs validation) |
| **Coverage** | Broad (30 stores for Coles, in-store for Aldi) | Single location per scrape |
| **Cost** | $0 (already paying for it) | Development time + monitoring |

**Winner: PCB Price Tables**

---

## Known Issues with PCB Tables (and How to Address Them)

### Issue 1: Price Discrepancies
**Problem:** Daria found 50¢-$1 differences between web API and BigQuery table  
**Impact:** Medium  
**Solution:**
```python
# Multi-source validation with confidence scoring
def validate_price(product_id, pcb_price):
    sources = [
        {'source': 'pcb_table', 'price': pcb_price},
        {'source': 'web_spot_check', 'price': spot_check_web(product_id)}  # Sample only
    ]
    
    variance = max(s['price'] for s in sources) - min(s['price'] for s in sources)
    
    if variance <= 0.50:
        return pcb_price, confidence=0.95  # Acceptable variance
    else:
        return pcb_price, confidence=0.70  # Flag for review
```

**Mitigation:**
- Use PCB price as source of truth
- Spot-check web for 10% of products weekly
- Flag high-variance items (>$0.50) for manual review
- Track variance patterns to improve PCB data feed

### Issue 2: Missing Products in Matched View
**Problem:** Some Aldi products exist in "competitors only" table but missing from matched view  
**Impact:** Low (we need competitors-only products anyway)  
**Solution:**
```sql
-- Use BOTH tables in product catalog build

-- Table 1: Matched products (Woolworths + competitor equivalent)
SELECT * FROM pcb_data.price_index_matched_view
WHERE retailer IN ('Coles', 'Aldi')

UNION ALL

-- Table 2: Competitor-only products (no WW equivalent)
SELECT * FROM pcb_data.competitor_only_view
WHERE retailer IN ('Coles', 'Aldi')
```

**Mitigation:**
- Request both views from PCB team
- Handle matching logic ourselves when needed
- This gives us MORE coverage, not less

### Issue 3: Aldi Online/In-Store Price Differences
**Problem:** Aldi prices differ between online and in-store  
**Impact:** Medium for Aldi specifically  
**Solution:**
```python
# Use in-store prices (what PCB scrapes) as source of truth for Aldi
# Document this in methodology

RETAILER_PRICE_SOURCES = {
    'Woolworths': 'web_api',      # Real-time online
    'Coles': 'pcb_web_scrape',    # 30 stores scraped weekly
    'Aldi': 'pcb_instore_scrape'  # In-store scraping (~99% accurate)
}
```

**Mitigation:**
- Use PCB in-store prices for Aldi (most accurate)
- Note in reporting: "Aldi prices reflect in-store, may differ from website"
- CEO will understand this (common knowledge)

### Issue 4: Weekly Lag (Data Published Fridays)
**Problem:** Prices updated Wednesday, but data available Friday  
**Impact:** Low (acceptable for trend tracking)  
**Solution:**
```python
# Run mystery shops on Fridays or Saturdays
# This ensures:
# 1. Latest price data from PCB (published Friday)
# 2. Matches when most customers shop (weekend)

OPTIMAL_SHOP_SCHEDULE = {
    'day_of_week': 'Friday',  # After PCB refresh
    'time': '10:00 AM',       # After data pipeline completes
    'note': '48-72 hour lag from price change is acceptable for CEO reporting'
}
```

**Mitigation:**
- Schedule daily shops for Friday mornings
- This gives us freshest data from weekly refresh
- 48-hour lag is acceptable for strategic pricing insights (not real-time trading)

---

## Recommended Architecture: Hybrid Approach

### Primary Data Source: PCB Tables (95% of product/price data)

```python
class ProductDataSource:
    """
    Primary: PCB tables for all retailers
    Secondary: Web spot-checks for validation
    """
    
    def get_product_price(self, retailer, product_id):
        # Always use PCB table as source of truth
        pcb_price = self.query_pcb_table(retailer, product_id)
        
        # Spot-check web for sample validation (10% of queries)
        if random.random() < 0.10:
            web_price = self.spot_check_web(retailer, product_id)
            self.log_variance(product_id, pcb_price, web_price)
        
        return pcb_price
```

### Secondary Validation: Web Spot-Checks (5% sample)

**Purpose:** Validate PCB data accuracy, NOT replace it

```python
# Weekly validation routine
def weekly_validation():
    """
    Each week:
    1. Sample 20 products per retailer (60 total)
    2. Check web prices
    3. Compare to PCB table
    4. Report variance
    
    If variance >5% of products have >$0.50 difference:
    → Escalate to PCB team for investigation
    """
    pass
```

---

## Product Catalog Build Strategy

### Step 1: Extract from PCB Custom Views (Day 1)

```python
# src/data/pcb_connector.py

class PCBDataConnector:
    """
    Connect to PCB custom data views
    (Requested from Karthik/Mithun/Alexander team)
    """
    
    def fetch_competitor_products(self, retailer: str) -> pd.DataFrame:
        """
        Fetch from custom view created by PCB team
        """
        
        query = f"""
        SELECT 
            -- Product identity
            competitor_product_id as product_id,
            product_name as name,
            brand,
            category,
            subcategory,
            
            -- Pack size
            pack_size,
            pack_size_unit,
            
            -- Pricing
            price as current_price,
            unit_price,
            promotion_flag as promotion_type,
            promotion_description,
            
            -- Metadata
            price_date,
            in_stock,
            product_url as url,
            
            -- Location
            state,
            store_location
            
        FROM `your-project.pcb_data.mystery_shopper_custom_view`
        WHERE retailer = '{retailer}'
        AND state = 'NSW'
        AND deleted = 'n'
        AND price_date >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
        ORDER BY price_date DESC
        """
        
        return self.bq_client.query(query).to_dataframe()
    
    def fetch_woolworths_products(self) -> pd.DataFrame:
        """
        Fetch Woolworths products
        
        Options:
        1. Internal product catalog
        2. Web API
        3. PCB table (if they scrape WW too)
        
        Choose based on what's available
        """
        # TODO: Determine best Woolworths source
        pass
```

### Step 2: Enrich with Embeddings (Day 1-2)

```python
# Same embedding approach as before, but source is PCB tables
catalog = pcb_connector.fetch_competitor_products('Coles')
embeddings = generate_embeddings(catalog['name'] + ' ' + catalog['category'])
catalog['embedding'] = embeddings
```

### Step 3: Build Unified Search Index (Day 2)

```python
# Vector search works exactly the same
# Just different data source (PCB tables instead of web scraping)
```

---

## Addressing the Bright Data Question

**From brainstorming notes:**
> "Bright Data and Amplify with AI agents" suggested as web scraping tool

**Evaluation:**

**Bright Data:**
- Commercial web scraping service
- Costs: $500-$2000/month for retail scraping
- Handles blocking, proxies, CAPTCHA
- Legal compliance features

**Recommendation:** Not needed for MVP because:
1. We already have PCB data (same/better quality, free)
2. $500+/month not justified when PCB data is ~99% accurate
3. Adds complexity without clear benefit
4. Can evaluate in Phase 2 if PCB data proves insufficient

**Future consideration:** If CEO wants real-time pricing (not trend analysis), then consider Bright Data for Phase 2.

---

## Data Source Strategy by Retailer

### Woolworths
**Primary Source:** Internal product catalog + Web API (real-time)  
**Why:** We have direct access, most accurate, real-time  
**Backup:** PCB table if they scrape WW (confirm with PCB team)

### Coles
**Primary Source:** PCB web scrape table (30 stores, weekly refresh)  
**Why:** Validated, trusted, broad coverage  
**Validation:** Spot-check web for 10% of products weekly

### Aldi
**Primary Source:** PCB in-store scrape table (~99% accuracy)  
**Why:** In-store prices are what customers actually pay  
**Note:** Online prices differ, in-store is ground truth for Aldi

---

## Implementation Timeline with PCB Tables

### Day 1 (April 21): Data Access
- ✅ We already have access to PCB tables
- Email PCB team for custom view specifications (simpler than original plan)
- Confirm we can query both matched and competitor-only views
- **No scraper development needed** → Save 1-2 weeks

### Day 2 (April 22): Extract & Load
- Query PCB tables for Coles and Aldi products
- Get Woolworths products from internal catalog/API
- **Start with real, validated data immediately**

### Day 3 (April 23): Build Search Index
- Generate embeddings from PCB data
- Build vector search index
- Test search precision

### Day 4-5 (April 24-25): Validation
- Validate against Feb 20 baseline
- Spot-check web prices for variance analysis
- Document accuracy metrics

**Result:** Production-ready search by Friday, not 3 weeks of scraper development.

---

## Decision Matrix

| Requirement | PCB Tables | Web Scraping |
|-------------|-----------|--------------|
| **Accuracy** | ✅ 99% validated | ❓ Unknown |
| **Speed to Build** | ✅ 2 days | ❌ 2-3 weeks |
| **CEO Trust** | ✅ Already used for price index | ❌ Needs validation |
| **Maintenance** | ✅ Zero (PCB owns it) | ❌ High (constant updates) |
| **Legal Risk** | ✅ Zero | ⚠️ Medium |
| **Cost** | ✅ Free | ❌ $500+/month (Bright Data) |
| **Reliability** | ✅ High | ❌ Low (blocking, changes) |
| **Real-time** | ⚠️ Weekly lag | ✅ Real-time |

**For CEO project prioritizing accuracy, trust, and speed to delivery: PCB Tables WIN**

---

## Final Recommendation

### Use PCB Price Tables with Validation Layer

```python
# Architecture

┌─────────────────────────────────────────┐
│        Product Catalog                   │
│                                          │
│  Source: PCB Tables (Coles, Aldi)       │
│         + WW Internal Catalog            │
│                                          │
│  Refresh: Weekly (Fridays)               │
│  Coverage: 95%+ of products              │
│  Accuracy: 99% (PCB validated)           │
└─────────────────────────────────────────┘
              │
              ↓
┌─────────────────────────────────────────┐
│     Validation Layer (10% sample)        │
│                                          │
│  Spot-check web prices weekly            │
│  Flag variance >$0.50                    │
│  Report to PCB team if systematic issue  │
└─────────────────────────────────────────┘
              │
              ↓
┌─────────────────────────────────────────┐
│       Vector Search Index                │
│                                          │
│  Embeddings from PCB product names       │
│  Semantic search with 95%+ precision     │
└─────────────────────────────────────────┘
```

### Benefits:
1. ✅ **Fast:** Operational in 2 days vs 3 weeks
2. ✅ **Trusted:** CEO already trusts PCB data for pricing
3. ✅ **Accurate:** 99% accuracy validated by PCB team
4. ✅ **Reliable:** No scrapers to break or maintain
5. ✅ **Legal:** No ToS violations or blocking
6. ✅ **Free:** Already paying for PCB data

### Tradeoffs Accepted:
- ⚠️ Weekly lag (acceptable for trend analysis)
- ⚠️ Some Aldi online/in-store differences (noted in methodology)
- ⚠️ Not real-time (but CEO wants daily trends, not minute-by-minute)

---

## Email to PCB Team (Updated)

**Subject:** Custom Data View for Mystery Shopper Agent - Simplified Request

Hi Karthik, Mithun, Alexander,

Following our meeting, we'd like to proceed with using the PCB competitor tables as our primary data source for the Mystery Shopper project. This avoids web scraping complexity and leverages your validated data.

**Request:** Access to BigQuery views with the following:

**View 1: Competitor Products (Coles + Aldi)**
```sql
-- All competitor products with latest pricing
-- NSW only, active products, last 7 days
```

Required columns:
- product_id, retailer, name, brand, category
- pack_size, price, unit_price
- promotion_flag, promotion_description
- price_date, in_stock, product_url
- state, store_location

**View 2: Woolworths Products (if available)**
- Same schema as above
- If not in PCB tables, we'll use internal catalog

**Refresh:** Weekly is perfect (we'll query Fridays)

**Timeline:** Needed by Friday April 25 for Week 1 testing

Can you provide:
1. Table/view names we should query
2. Any filters we should apply (deleted='n', etc.)
3. Confirmation this matches what Commercial uses for price index

Thanks,
Alexa & Daria

---

## Next Actions

**Immediate (Monday):**
1. [ ] Send simplified email to PCB team (above)
2. [ ] Confirm access to existing PCB tables
3. [ ] Query sample data to verify structure
4. [ ] Start catalog build using PCB data

**No scraper development needed.**  
**No Bright Data subscription needed.**  
**Use proven, validated, CEO-trusted data source.**

---

*Web scraping is a solution looking for a problem. We already have the data.*
