# Week 1 Quick-Start Actions

## Today (April 21) - Get Started Immediately

### 1. Stakeholder Alignment (2 hours)
- [ ] Review IMPLEMENTATION_PLAN.md with Owen
- [ ] Get approval on technical approach
- [ ] Confirm 4-week timeline is acceptable
- [ ] Agree on Week 1 deliverable (1 mission validated)

### 2. Email PCB Team for Data (30 mins)
**To:** Karthik, Mithun, Alexander  
**Subject:** Custom Data View Specifications for Mystery Shopper Agent

```
Hi team,

Following up from our recent meeting, we'd like to request a custom data view 
for the Mystery Shopper project. Here are our exact requirements:

Required Columns:
- woolworths_product_id (if available - for matched products)
- competitor_product_id
- retailer (Coles or Aldi)
- product_name
- brand
- category
- pack_size
- price (most recent)
- unit_price
- price_date (when price was captured)
- promotion_flag (yellow_ticket / red_edlp / none)
- in_stock_flag
- state (NSW only for MVP)

Format: BigQuery view or table
Refresh: Weekly (Fridays preferred)
Scope: NSW stores only, active products only (deleted='n')

We need two views:
1. Matched products (Woolworths product + competitor equivalent)
2. Competitor-only products (for items WW doesn't stock)

Timeline: Required by Friday April 25 to stay on track

Questions: [Your name/contact]

Thanks,
Alexa & Daria
```

### 3. Set Up GCP Environment (2 hours)

```bash
# Create BigQuery dataset
bq mk --dataset \
  --location=australia-southeast1 \
  mystery_shopper

# Create tables (copy from IMPLEMENTATION_PLAN.md)
bq mk --table mystery_shopper.raw_shops schema_raw_shops.json
bq mk --table mystery_shopper.validation_logs schema_validation_logs.json
```

### 4. Create Development Structure (1 hour)

```bash
cd /home/akelly5_woolworths_com_au/mystery-shopper

# Create project structure
mkdir -p src/{agents,data,validation,utils}
mkdir -p config
mkdir -p tests
mkdir -p notebooks  # For exploratory analysis

# Create placeholder files
touch src/agents/persona_agent.py
touch src/agents/product_search.py
touch src/data/pricing_api.py
touch src/validation/price_validator.py
touch src/utils/mission_parser.py
touch config/personas.yaml
touch config/missions.yaml
```

---

## Day 2 (April 22) - Build Pricing Validation

### Priority Task: Validate Feb 20 Baseline Data

**Goal:** Prove you can get accurate pricing for products we manually shopped

```python
# src/validation/price_validator.py

import requests
from google.cloud import bigquery
from datetime import datetime

class PricingValidator:
    def __init__(self):
        self.bq_client = bigquery.Client()
        
    def validate_woolworths_price(self, product_id):
        """
        Check Woolworths price from multiple sources
        Returns: (price, confidence, sources)
        """
        sources = []
        
        # Source 1: Web API
        try:
            web_price = self._fetch_woolworths_api(product_id)
            sources.append({
                'source': 'web_api',
                'price': web_price,
                'timestamp': datetime.now()
            })
        except Exception as e:
            print(f"Web API failed: {e}")
        
        # Source 2: Internal catalog (if you have access)
        # catalog_price = self._fetch_internal_catalog(product_id)
        
        # Calculate confidence
        if len(sources) == 0:
            return None, 0.0, []
        
        prices = [s['price'] for s in sources]
        avg_price = sum(prices) / len(prices)
        variance = max(prices) - min(prices)
        
        confidence = 1.0 if variance == 0 else (0.8 if variance < 0.5 else 0.3)
        
        return avg_price, confidence, sources
    
    def _fetch_woolworths_api(self, product_id):
        """Call Woolworths web API for price"""
        # TODO: Implement actual API call
        # url = f"https://www.woolworths.com.au/api/v1/products/{product_id}"
        # response = requests.get(url, headers={...})
        # return response.json()['price']
        pass

# Test with Feb 20 baseline
if __name__ == "__main__":
    validator = PricingValidator()
    
    # Test with spaghetti bolognese items
    test_products = {
        'lean_mince_beef_1kg': 'WW_PRODUCT_ID_HERE',
        'pasta_500g': 'WW_PRODUCT_ID_HERE',
        # ... add more from Feb 20 manual shop
    }
    
    for item_name, product_id in test_products.items():
        price, confidence, sources = validator.validate_woolworths_price(product_id)
        print(f"{item_name}: ${price:.2f} (confidence: {confidence:.2f})")
```

---

## Day 3 (April 23) - Mission Item Parser

### Build the Foundation for Product Search

```python
# src/utils/mission_parser.py

import re
from typing import Dict, Optional

class MissionItemParser:
    """
    Parse mission items into searchable components
    
    Example:
    "Lean Mince beef, 1kg" → {
        'category': 'meat',
        'type': 'mince beef',
        'quality': 'lean',
        'pack_size': '1kg',
        'pack_size_value': 1.0,
        'pack_size_unit': 'kg'
    }
    """
    
    def parse(self, mission_item: str) -> Dict:
        """Parse mission item into components"""
        
        result = {
            'original': mission_item,
            'category': None,
            'type': None,
            'quality': None,
            'brand': None,
            'pack_size': None,
            'pack_size_value': None,
            'pack_size_unit': None
        }
        
        # Extract pack size (e.g., "1kg", "500g", "2x250g")
        pack_size_match = re.search(r'(\d+(?:x\d+)?)\s*(kg|g|L|ml|pack)', mission_item, re.IGNORECASE)
        if pack_size_match:
            result['pack_size'] = pack_size_match.group(0)
            result['pack_size_value'] = self._parse_pack_size_value(pack_size_match.group(0))
            result['pack_size_unit'] = pack_size_match.group(2).lower()
        
        # Extract quality indicators
        quality_terms = ['lean', 'extra lean', 'premium', 'free range', 'organic']
        for term in quality_terms:
            if term.lower() in mission_item.lower():
                result['quality'] = term
                break
        
        # Extract brand (if specified)
        # TODO: Add brand detection logic
        
        # Extract product type (remaining text after removing modifiers)
        type_text = mission_item
        if result['pack_size']:
            type_text = type_text.replace(result['pack_size'], '').strip()
        if result['quality']:
            type_text = type_text.replace(result['quality'], '').strip()
        
        result['type'] = type_text.strip(' ,')
        
        # Determine category (simple rules for now)
        result['category'] = self._infer_category(result['type'])
        
        return result
    
    def _parse_pack_size_value(self, pack_size_str: str) -> float:
        """Convert pack size to normalized value in grams or ml"""
        # Handle "2x250g" → 500g
        if 'x' in pack_size_str:
            parts = pack_size_str.split('x')
            multiplier = float(re.search(r'\d+', parts[0]).group())
            value = float(re.search(r'\d+', parts[1]).group())
            return multiplier * value
        
        # Extract number
        value = float(re.search(r'\d+', pack_size_str).group())
        
        # Convert to base unit (g or ml)
        if 'kg' in pack_size_str.lower():
            return value * 1000
        elif 'l' in pack_size_str.lower() and 'ml' not in pack_size_str.lower():
            return value * 1000
        else:
            return value
    
    def _infer_category(self, type_text: str) -> str:
        """Simple category inference"""
        categories = {
            'meat': ['mince', 'beef', 'chicken', 'pork', 'sausage', 'lamb'],
            'dairy': ['milk', 'cheese', 'yoghurt', 'butter', 'cream'],
            'produce': ['banana', 'apple', 'onion', 'tomato', 'carrot', 'mushroom'],
            'pantry': ['pasta', 'sauce', 'oil', 'bread'],
            # ... add more
        }
        
        type_lower = type_text.lower()
        for category, keywords in categories.items():
            if any(kw in type_lower for kw in keywords):
                return category
        
        return 'other'

# Test
if __name__ == "__main__":
    parser = MissionItemParser()
    
    test_items = [
        "Lean Mince beef, 1kg",
        "Spaghetti / Dry pasta, 500g",
        "Strawberries, 2x250g",
        "Full cream milk, 3L"
    ]
    
    for item in test_items:
        parsed = parser.parse(item)
        print(f"\n{item}")
        print(f"  → Type: {parsed['type']}")
        print(f"  → Category: {parsed['category']}")
        print(f"  → Pack size: {parsed['pack_size']} ({parsed['pack_size_value']}{parsed['pack_size_unit']})")
        print(f"  → Quality: {parsed['quality']}")
```

---

## Day 4 (April 24) - Simple Product Search

### Test Product Retrieval (Before RAG)

```python
# src/agents/product_search.py

from typing import List, Dict
from src.utils.mission_parser import MissionItemParser

class ProductSearcher:
    """
    Simple keyword-based product search
    (We'll upgrade to RAG in Week 3 if needed)
    """
    
    def __init__(self):
        self.parser = MissionItemParser()
    
    def search_woolworths(self, mission_item: str, max_results: int = 10) -> List[Dict]:
        """
        Search Woolworths for products matching mission item
        Returns list of candidate products
        """
        
        # Parse mission item
        parsed = self.parser.parse(mission_item)
        
        # For now, use Woolworths web search
        # TODO: Replace with proper API or catalog search
        search_query = parsed['type']
        
        candidates = self._fetch_woolworths_search(search_query, max_results)
        
        # Filter and rank
        filtered = self._filter_candidates(candidates, parsed)
        ranked = self._rank_candidates(filtered, parsed)
        
        return ranked[:max_results]
    
    def _fetch_woolworths_search(self, query: str, limit: int) -> List[Dict]:
        """
        Fetch search results from Woolworths
        TODO: Implement actual search API call
        """
        # Placeholder - replace with actual implementation
        # url = "https://www.woolworths.com.au/api/v3/ui/search"
        # params = {'searchTerm': query}
        # response = requests.get(url, params=params)
        # return response.json()['products']
        
        return []  # Placeholder
    
    def _filter_candidates(self, candidates: List[Dict], parsed: Dict) -> List[Dict]:
        """Filter candidates based on parsed requirements"""
        filtered = []
        
        for product in candidates:
            # Check category match
            if parsed['category'] and product.get('category') != parsed['category']:
                continue
            
            # Check pack size is within reasonable range
            if parsed['pack_size_value']:
                product_size = self._extract_pack_size(product)
                if not self._is_size_match(product_size, parsed['pack_size_value']):
                    continue
            
            filtered.append(product)
        
        return filtered
    
    def _rank_candidates(self, candidates: List[Dict], parsed: Dict) -> List[Dict]:
        """Rank candidates by relevance"""
        # Simple ranking: exact pack size match first, then closest size
        
        for product in candidates:
            score = 0
            
            # Exact pack size match
            product_size = self._extract_pack_size(product)
            if product_size == parsed['pack_size_value']:
                score += 10
            else:
                # Closer is better
                size_diff = abs(product_size - parsed['pack_size_value'])
                score += max(0, 5 - size_diff/100)
            
            # Quality match
            if parsed['quality'] and parsed['quality'].lower() in product.get('name', '').lower():
                score += 5
            
            product['relevance_score'] = score
        
        return sorted(candidates, key=lambda x: x['relevance_score'], reverse=True)
    
    def _extract_pack_size(self, product: Dict) -> float:
        """Extract pack size from product data"""
        # TODO: Implement based on actual product data structure
        return 0.0
    
    def _is_size_match(self, product_size: float, target_size: float) -> bool:
        """Check if pack sizes are reasonably close"""
        # Within 50% of target size
        return target_size * 0.5 <= product_size <= target_size * 1.5
```

---

## Day 5 (April 25) - Week 1 Checkpoint

### Prepare Demo & Decision Point

**What to Show:**
1. Pricing validation working for 10 products from Feb 20 baseline
2. Mission item parser correctly extracting components
3. Product search returning relevant candidates for 3 items
4. BigQuery tables set up and ready

**Decision:** Can we proceed to Week 2?
- ✅ Yes if: Pricing validation >80% accurate, product search finds matches
- ⚠️ Adjust if: Major data source issues, need to pivot approach

**Document:**
- What worked well
- What needs attention
- Risks for Week 2

---

## Quick Reference: Files to Create

```
mystery-shopper/
├── src/
│   ├── agents/
│   │   ├── __init__.py
│   │   ├── persona_agent.py        # Week 2
│   │   └── product_search.py       # Week 1 (Day 4)
│   ├── data/
│   │   ├── __init__.py
│   │   ├── pricing_api.py          # Week 1 (Day 2)
│   │   └── pcb_connector.py        # Week 1 (after data view ready)
│   ├── validation/
│   │   ├── __init__.py
│   │   └── price_validator.py      # Week 1 (Day 2)
│   └── utils/
│       ├── __init__.py
│       └── mission_parser.py       # Week 1 (Day 3)
├── config/
│   ├── personas.yaml               # Week 2
│   ├── missions.yaml               # Week 1 (Day 3)
│   └── database_schemas.json       # Week 1 (Day 1)
├── tests/
│   ├── test_price_validator.py     # Week 1 (Day 2)
│   ├── test_mission_parser.py      # Week 1 (Day 3)
│   └── test_product_search.py      # Week 1 (Day 4)
├── notebooks/
│   └── feb20_baseline_analysis.ipynb  # Week 1 (Day 2)
├── IMPLEMENTATION_PLAN.md
├── PROJECT_BRIEF.md
└── WEEK1_ACTIONS.md (this file)
```

---

## Key Contacts & Resources

**Data Sources:**
- PCB Team: Karthik, Mithun, Alexander (competitor data)
- Woolworths API: [Need contact for API access/keys]

**Stakeholders:**
- Owen Lim: Project sponsor
- Ben Kress: Stakeholder

**Technical Resources:**
- GCP Project: [Project ID needed]
- BigQuery Dataset: mystery_shopper
- Repository: [GitHub repo to be created]

---

## Questions to Answer This Week

- [ ] Do we have access to Woolworths API? Need API keys?
- [ ] When will PCB custom data view be ready?
- [ ] Can we access Feb 20 baseline data in structured format?
- [ ] What's the approval process for Gemini API usage?

---

**End of Week 1, you should have:**
✅ Validated pricing for Spaghetti Bolognese mission  
✅ Working product search returning candidates  
✅ BigQuery database populated with test data  
✅ Confidence to proceed to Week 2 (LLM agent)
