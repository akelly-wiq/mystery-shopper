# May 6 Meeting - Action Items Detailed Breakdown
## Task Explanations and Implementation Guide

**Meeting Date:** May 6, 2026  
**Attendees:** Alexa Kelly, Daria Volkova  
**Context:** Follow-up discussion on agent prompts, search strategy, and data pipeline

---

## Alexa Kelly's Tasks

### Task 1: Review Persona Prompts and Descriptions

**What:**
Review all updated persona prompts and descriptions that Daria has completed. Assess whether any changes are required.

**Context:**
- Daria updated all prompts with four descriptions
- Added 'retry context' for the evaluation agent (injected if decision key returns 'false')
- All persona simulations are complete
- Prompts are in the 'parallel agents' branch (NOT main branch)

**How to Approach:**
1. Switch to 'parallel agents' branch: `git checkout parallel-agents`
2. Navigate to persona prompts location (check with Daria for exact path)
3. Review each of the 5 persona prompts:
   - Saver
   - Traditional
   - Conscious
   - Refined
   - Essential
4. For each prompt, assess:
   - Are the decision rules clear and specific?
   - Do the four descriptions provide enough context?
   - Is the retry context appropriate for failed decisions?
   - Will this differentiate from other personas?
5. Provide feedback to Daria on any needed changes

**Success Criteria:**
- All 5 persona prompts reviewed
- Feedback documented (changes needed or approved as-is)
- Retry context validated for evaluation agent

**Priority:** HIGH (blocks persona testing)

**Dependencies:** Daria must push latest changes first

---

### Task 2: Test Web Application Locally

**What:**
Run the ADK (Agent Development Kit) web application locally and check the responses returned by the application.

**Context:**
- Application is in 'parallel agents' branch
- Need to test that agents are responding correctly
- Testing location: `cd app` directory
- Can select specific personas to test

**How to Approach:**
1. Switch to 'parallel agents' branch
2. Navigate to app directory: `cd app`
3. Start the web application locally (check for README or startup script)
4. Test each persona:
   - Select a persona from available options
   - Input a test mission/ingredient
   - Observe the response
5. Validate:
   - Does the agent return a product selection?
   - Is the justification text included?
   - Does the decision align with persona behavior?
   - Are confidence scores present?
6. Document any issues or unexpected behaviors

**Success Criteria:**
- Web app runs successfully locally
- Able to test all 5 personas
- Responses include expected fields (product, justification, confidence)
- Any bugs or issues documented for Daria

**Priority:** MEDIUM (testing and validation)

**Dependencies:** 
- Task 1 (persona prompts reviewed)
- Daria's pipeline build (to have full functionality)

**Note from meeting:** Daria offered to send example of input structure required to test agents

---

### Task 3: Develop Evaluation Agent Prompt Structure and Retry Context

**What:**
Develop the evaluation agent prompt structure and the retry context mechanism.

**Context:**
- Evaluation agent validates product selections
- If validation fails (decision key = 'false'), retry context is injected
- Need to define what triggers retry and what context to provide
- Daria mentioned she'll help with this

**How to Approach:**
1. **Define evaluation criteria:**
   - Does product match ingredient requirement?
   - Does selection align with persona rules?
   - Is price reasonable for persona?
   - Is pack size appropriate?

2. **Design prompt structure:**
   ```
   You are an evaluation agent. Review this product selection:
   
   Ingredient Required: {ingredient}
   Persona: {persona_name} ({persona_rules})
   Product Selected: {product_name}
   Price: {price}
   Justification: {agent_justification}
   
   Evaluate:
   1. Does this product match the ingredient requirement?
   2. Is this selection aligned with {persona_name} behavior?
   3. Is the price reasonable for this persona?
   
   Return: decision (true/false), reasoning
   ```

3. **Design retry context:**
   - If decision = false, what feedback to provide?
   - Example: "Previous selection failed because: {failure_reason}. Please reconsider the available products."
   - Limit retries (max 2) to avoid infinite loops

4. **Collaborate with Daria** to implement in agent framework

**Success Criteria:**
- Evaluation prompt defined and tested
- Retry context mechanism documented
- Max retry limit implemented (suggest 2 retries)
- Edge cases considered (what if all retries fail?)

**Priority:** HIGH (critical for quality control)

**Dependencies:** None (can be done in parallel)

---

### Task 4: Create Unit Normalization Solution

**What:**
Develop an LLM call or separate pre-processing tool to normalize product prices for comparison.

**Context:**
- **Problem:** Products sold in different units (e.g., bananas "each" vs required "1 kilogram")
- **Impact:** Can't compare prices fairly without normalization
- **Example:** Bananas $0.50 each, but need 1kg → must estimate ~$1.00/kg
- **Decision:** Pre-processing step BEFORE sending to persona agent
- **Data location:** Use existing data frame in "B data test location"

**How to Approach:**

**Option 1: Rule-Based Pre-Processing (RECOMMENDED for MVP)**
```python
def normalize_unit_price(product):
    """
    Normalize product prices to standard units (per kg or per L)
    """
    # Define conversion rules
    conversions = {
        'each': estimate_weight_from_category,  # Banana each → ~200g
        'bunch': estimate_weight_from_category,  # Bunch → category-specific
        '100g': lambda x: x * 10,  # 100g → per kg
        'ml': lambda x: x / 1000,  # ml → per L
    }
    
    # Extract unit from pack_size
    unit = extract_unit(product['pack_size'])
    
    # Apply conversion
    if unit in conversions:
        normalized_price = conversions[unit](product['price'])
    else:
        normalized_price = product['unit_price']  # Use existing if available
    
    return {
        **product,
        'unit_normalized': normalized_price,
        'normalization_method': 'rule_based'
    }
```

**Option 2: LLM-Based Normalization (More flexible, slower)**
```python
def llm_normalize_price(product):
    """
    Use LLM to estimate normalized price for products with unclear units
    """
    prompt = f"""
    Product: {product['name']}
    Price: ${product['price']}
    Pack Size: {product['pack_size']}
    Category: {product['category']}
    
    Estimate the price per kilogram (or per liter for liquids).
    If the product is sold "each", estimate typical weight.
    
    Return: estimated_price_per_kg (or per_L)
    """
    # Call LLM
    # Return normalized price
```

**Recommended Approach:**
1. Start with rule-based for common cases (kg, g, L, ml)
2. Use LLM fallback for edge cases ("each", "bunch", "pack")
3. Test on B data test location data frame
4. Validate: Do normalized prices make sense?

**Success Criteria:**
- Normalization function implemented and tested
- Handles common units: kg, g, L, ml, each, bunch
- Adds `unit_normalized` field to data frame
- Tested on B data test location data
- Documentation of normalization logic

**Priority:** HIGH (required for fair price comparisons)

**Dependencies:** Access to B data test location data frame

**Timeline:** Need this before Week 3 full execution (by May 9)

---

## Daria Volkova's Tasks

### Task 5: Enrich BigQuery Output Schema - Add Justification Text Field

**What:**
Add a text field to the BigQuery output schema for decision justification.

**Context:**
- Currently output includes: mission, persona, ingredient, retailer, product, price, etc.
- Need to add: justification text explaining WHY this product was selected
- This provides transparency and enables human validation

**How to Approach:**
1. Update BigQuery schema definition:
   ```sql
   ALTER TABLE mystery_shopper.shop_results_detailed
   ADD COLUMN justification_text STRING;
   ```

2. Update data pipeline to populate this field:
   ```python
   result = {
       'mission_name': mission['name'],
       'persona': persona['name'],
       'ingredient': ingredient,
       'product_id': selected_product['id'],
       'price': selected_product['price'],
       'justification_text': decision['reasoning'],  # NEW FIELD
       ...
   }
   ```

3. Example justification text:
   > "Selected WW Own Brand Mince because yellow ticket special provides best unit price at $12/kg, meets exact 1kg requirement, and own brand aligns with Saver budget-first priority."

**Success Criteria:**
- BigQuery schema updated with `justification_text` column
- Pipeline populates this field for every product selection
- Text is human-readable and explains decision reasoning

**Priority:** HIGH (transparency for board presentation)

**Dependencies:** Agent decision logic must return reasoning

---

### Task 6: Add Decisioning Attributes to Output Schema

**What:**
Include all decisioning attributes used by the product agent in the output schema.

**Context:**
- Need to show WHAT factors influenced each decision
- Examples: price, unit_price, promotion_type, quality_indicators, pack_size_match
- This enables analysis of decision patterns

**How to Approach:**
1. Define decisioning attributes to capture:
   - `price` (absolute price)
   - `unit_price` (price per kg/L)
   - `promotion_type` (yellow_ticket, red_edlp, none)
   - `pack_size_match` (does pack size meet requirement?)
   - `quality_indicators` (organic, free_range, etc.)
   - `own_brand_flag` (boolean)
   - `price_rank` (1st, 2nd, 3rd cheapest among candidates)

2. Add to BigQuery schema:
   ```sql
   ALTER TABLE mystery_shopper.shop_results_detailed
   ADD COLUMN decisioning_attributes JSON;
   ```

3. Populate with structured data:
   ```python
   decisioning_attributes = {
       'price': 12.00,
       'unit_price': 12.00,
       'promotion_type': 'yellow_ticket',
       'pack_size_match': True,
       'quality_indicators': ['own_brand'],
       'price_rank': 1,
       'confidence_score': 0.94
   }
   ```

**Success Criteria:**
- BigQuery schema updated with `decisioning_attributes` JSON column
- All relevant attributes captured for each decision
- Consistent structure across all records

**Priority:** HIGH (enables pattern analysis)

**Dependencies:** None (can be done in parallel with other tasks)

---

### Task 7: Refactor Search to Use Vertex AI Search

**What:**
Refactor the current search mechanism implementation to switch back to using Vertex AI Search (vector store method).

**Context:**
- **Previous approach:** Local embedding / simple keyword search
- **Problem identified:** Returning irrelevant results (e.g., "chicken breast bulk pack" → "nuggets")
- **New approach:** Vertex AI Search for better semantic matching
- **Applies to:** All 3 retailers (Woolworths, Coles, Aldi)

**How to Approach:**
1. **Remove/comment out old search code:**
   - Local embedding generation
   - BigQuery ML vector similarity
   - Simple keyword matching

2. **Implement Vertex AI Search client:**
   ```python
   from google.cloud import discoveryengine_v1beta as discoveryengine
   
   class VertexAIProductSearch:
       def __init__(self):
           self.client = discoveryengine.SearchServiceClient()
           self.search_configs = {
               'woolworths': 'projects/{project}/locations/{location}/dataStores/woolworths-products',
               'coles': 'projects/{project}/locations/{location}/dataStores/coles-products',
               'aldi': 'projects/{project}/locations/{location}/dataStores/aldi-products'
           }
       
       def search(self, query, retailer, max_results=3):
           serving_config = self.search_configs[retailer.lower()]
           
           request = discoveryengine.SearchRequest(
               serving_config=serving_config,
               query=query,
               page_size=max_results
           )
           
           response = self.client.search(request)
           return [self._parse_result(r) for r in response.results]
   ```

3. **Update pipeline to use new search:**
   ```python
   # OLD
   # results = semantic_search.search(ingredient, retailer, max_results=5)
   
   # NEW
   results = vertex_ai_search.search(ingredient, retailer, max_results=3)
   ```

4. **Test accuracy improvement:**
   - Test problematic queries ("chicken breast bulk pack")
   - Verify top 3 results are relevant
   - Compare to old approach

**Success Criteria:**
- Old search code removed/deprecated
- Vertex AI Search implemented for all 3 retailers
- Returns top 3 results (changed from 5)
- Accuracy improvement validated on test queries

**Priority:** CRITICAL (blocks Week 2-3 progress)

**Dependencies:** Task 8 (Vertex AI Search instances must be created first)

---

### Task 8: Create Vertex AI Search Instances

**What:**
Set up 2 Vertex AI Search instances (one for Aldi data, one for Coles data).

**Context:**
- **Decision from meeting:** Use Vertex AI Search for Coles and Aldi
- **Note:** Meeting notes say "2 instances" but we have 3 retailers - clarify if Woolworths also needs one
- **Index strategy:** One index per retailer covering 4 weeks of data (TBD: combined or per-week)

**How to Approach:**

1. **Enable Vertex AI Search API:**
   ```bash
   gcloud services enable discoveryengine.googleapis.com
   ```

2. **Create data stores (3 total - assuming Woolworths also needs one):**
   ```bash
   # Woolworths
   gcloud alpha discovery-engine data-stores create woolworths-products \
       --location=global \
       --collection=default_collection \
       --industry-vertical=RETAIL
   
   # Coles
   gcloud alpha discovery-engine data-stores create coles-products \
       --location=global \
       --collection=default_collection \
       --industry-vertical=RETAIL
   
   # Aldi
   gcloud alpha discovery-engine data-stores create aldi-products \
       --location=global \
       --collection=default_collection \
       --industry-vertical=RETAIL
   ```

3. **Prepare data for indexing:**
   - Extract product catalogs from BigQuery
   - Format as JSON documents
   - Include: product_id, name, brand, category, price, promotions, pack_size
   - Historical scope: 4 weeks

4. **Index products:**
   ```bash
   gcloud alpha discovery-engine documents import \
       --data-store=woolworths-products \
       --location=global \
       --collection=default_collection \
       --gcs-uri=gs://your-bucket/woolworths_products.jsonl
   ```

5. **Configure search settings:**
   - Enable semantic search mode
   - Set boost specs for promotions
   - Test search relevance

**Success Criteria:**
- 3 Vertex AI Search data stores created (WW, Coles, Aldi)
- Product data indexed (4 weeks historical)
- Search queries return relevant results
- Latency <500ms per query

**Priority:** CRITICAL (blocks Task 7)

**Dependencies:** 
- Product catalog data in BigQuery
- GCP permissions to create Vertex AI resources

**Open Question:** Do we need separate indexes per week (12 total) or combined per retailer (3 total)? Recommend combined for simplicity.

---

### Task 9: Integrate Programmatic Data Extraction

**What:**
Integrate the programmatic solution to handle data extraction for the pipeline.

**Context:**
- Need to extract product data from BigQuery tables
- Transform into format suitable for Vertex AI Search and agent processing
- Automate rather than manual queries

**How to Approach:**
1. **Create data extraction module:**
   ```python
   # src/data/product_extractor.py
   
   class ProductDataExtractor:
       def __init__(self):
           self.bq_client = bigquery.Client()
       
       def extract_woolworths_products(self, weeks_back=4):
           """Extract Woolworths products from internal table"""
           query = """
           SELECT 
               product_id,
               product_name,
               brand,
               price,
               unit_price,
               pack_size,
               promotion_type,
               date
           FROM `gcp-wow-ent-im-tbl-prod.adp_dm_betterbuying_view.Sell_Price_Weekly_v`
           WHERE date >= DATE_SUB(CURRENT_DATE(), INTERVAL @weeks DAY)
           """
           return self.bq_client.query(query, 
               query_parameters=[bigquery.ScalarQueryParameter("weeks", "INT64", weeks_back * 7)]
           ).to_dataframe()
       
       def extract_coles_products(self, weeks_back=4):
           """Extract Coles from PCB competitor table"""
           # Similar query for Coles
           pass
       
       def extract_aldi_products(self, weeks_back=4):
           """Extract Aldi from PCB table + enrichment"""
           # Similar query for Aldi
           pass
   ```

2. **Build extraction pipeline:**
   - Extract from all 3 retailers
   - Deduplicate products
   - Validate data quality
   - Save to staging table or files

3. **Integrate with Vertex AI indexing:**
   - Transform to JSONL format
   - Upload to GCS bucket
   - Trigger Vertex AI import

**Success Criteria:**
- Extraction module implemented
- Can pull 4 weeks of data for all 3 retailers
- Automated pipeline (not manual queries)
- Data quality validation included

**Priority:** HIGH (required for pipeline)

**Dependencies:** Access to BigQuery tables confirmed

---

### Task 10: Build Data Pipeline

**What:**
Build the data processing pipeline that generates a data frame and sends it to the agent for decision making.

**Context:**
- **Pipeline flow:** 
  1. Search Vertex AI indexes → top 3 products
  2. (Optional) Unit normalization
  3. Send to persona agent for decision
  4. Evaluation agent validates
  5. Save to BigQuery

**How to Approach:**
1. **Define pipeline orchestration:**
   ```python
   # src/pipeline/mission_pipeline.py
   
   class MissionPipeline:
       def __init__(self):
           self.search = VertexAIProductSearch()
           self.normalizer = UnitNormalizer()  # Alexa's task
           self.agent = PersonaAgent()
           self.evaluator = EvaluationAgent()
       
       def execute_mission(self, mission, persona, retailer):
           results = []
           
           for ingredient in mission['ingredients']:
               # Step 1: Search
               candidates = self.search.search(ingredient, retailer, max_results=3)
               
               # Step 2: Normalize (if needed)
               candidates = [self.normalizer.normalize(c) for c in candidates]
               
               # Step 3: Agent decision
               decision = self.agent.select_product(ingredient, candidates, persona)
               
               # Step 4: Evaluation
               validation = self.evaluator.validate(decision, ingredient, persona)
               
               if not validation['approved']:
                   # Retry with context
                   decision = self.agent.select_product(
                       ingredient, 
                       candidates, 
                       persona,
                       retry_context=validation['feedback']
                   )
               
               # Step 5: Save
               results.append({
                   'mission': mission['name'],
                   'persona': persona['name'],
                   'ingredient': ingredient,
                   'product': decision['product'],
                   'justification': decision['reasoning'],
                   'decisioning_attributes': decision['attributes'],
                   'confidence': decision['confidence']
               })
           
           return pd.DataFrame(results)
   ```

2. **Build data frame output:**
   - Structured format matching BigQuery schema
   - Include all required fields
   - Validation before saving

3. **Integration testing:**
   - Test with 1 mission
   - Validate all steps execute
   - Check output format

**Success Criteria:**
- Pipeline executes end-to-end for a single mission
- Generates data frame with all required fields
- Integrates search, normalization, agent, evaluation
- Error handling for failures

**Priority:** CRITICAL (core deliverable)

**Dependencies:** 
- Task 7 (Vertex AI Search)
- Task 8 (Search instances)
- Alexa's Task 4 (normalization)

---

### Task 11: Find Woolworths Offer Description Data

**What:**
Search for descriptive data corresponding to Offer ID numbers, necessary for Woolworths historical implementation.

**Context:**
- Woolworths historical data has "Offer IDs" but missing descriptive text
- Need to match Offer IDs to promotion descriptions
- Contact: Mel (mentioned in meeting) or Mtho (mentioned in notes)
- Required for understanding historical promotions

**How to Approach:**
1. **Contact Mel/Mtho:**
   - Ask: "Do we have a table that maps Offer IDs to promotion descriptions?"
   - Provide example Offer IDs from historical data

2. **Search existing tables:**
   ```sql
   -- Look for promotion/offer tables
   SELECT table_name 
   FROM `gcp-wow-ent-im-tbl-prod`.INFORMATION_SCHEMA.TABLES
   WHERE table_name LIKE '%offer%' OR table_name LIKE '%promo%'
   ```

3. **If found, join to historical data:**
   ```sql
   SELECT 
       h.product_id,
       h.offer_id,
       o.offer_description,
       o.offer_type
   FROM historical_table h
   LEFT JOIN offer_table o ON h.offer_id = o.offer_id
   ```

4. **If not found:**
   - Document as limitation
   - Use offer_id as-is (e.g., "OFFER_12345")
   - Or use promotion flags (half_price, low_price_special) as proxy

**Success Criteria:**
- Offer description data source identified (or confirmed as unavailable)
- If available: join logic implemented
- If not available: workaround documented

**Priority:** MEDIUM (nice-to-have for historical analysis)

**Dependencies:** Contact with Mel/Mtho

**Fallback:** Use existing promotion flags if offer descriptions unavailable

---

## Task Summary Table

| # | Task | Owner | Priority | Dependencies | Deadline |
|---|------|-------|----------|--------------|----------|
| 1 | Review persona prompts | Alexa | HIGH | Daria pushes changes | May 7 |
| 2 | Test web application | Alexa | MEDIUM | Task 1, Daria's pipeline | May 9 |
| 3 | Evaluation agent prompt | Alexa | HIGH | None | May 8 |
| 4 | Unit normalization | Alexa | HIGH | B data access | May 9 |
| 5 | Add justification field to schema | Daria | HIGH | None | May 7 |
| 6 | Add decisioning attributes | Daria | HIGH | None | May 7 |
| 7 | Refactor to Vertex AI Search | Daria | CRITICAL | Task 8 | May 9 |
| 8 | Create Vertex AI instances | Daria | CRITICAL | GCP access | May 8 |
| 9 | Integrate data extraction | Daria | HIGH | BQ access | May 8 |
| 10 | Build pipeline | Daria | CRITICAL | Tasks 7,8, Alexa Task 4 | May 10 |
| 11 | Find offer description data | Daria | MEDIUM | Mel/Mtho contact | May 10 |

---

## Coordination Points

### Alexa ↔ Daria Dependencies

**Alexa needs from Daria:**
- Latest code pushed to 'parallel agents' branch (for Task 1, 2)
- Example input structure for testing (for Task 2)
- Collaboration on evaluation agent design (for Task 3)
- B data test location data frame (for Task 4)

**Daria needs from Alexa:**
- Persona prompt feedback (from Task 1) → to finalize prompts
- Unit normalization function (from Task 4) → to integrate in pipeline
- Evaluation agent prompt (from Task 3) → to integrate in agent framework

### Critical Path
1. Task 8 (Create Vertex AI instances) → Unblocks Task 7
2. Task 7 (Refactor search) → Unblocks Task 10
3. Task 4 (Normalization) → Needed for Task 10
4. Task 10 (Pipeline) → Core deliverable for Week 2 completion

---

## Next Steps

**Immediate (May 7):**
- Alexa: Task 1 (review prompts)
- Daria: Tasks 5, 6 (schema enrichment)
- Daria: Task 8 (create Vertex AI instances)

**This Week (May 7-9):**
- Alexa: Tasks 3, 4 (evaluation prompt, normalization)
- Daria: Tasks 7, 9 (refactor search, data extraction)

**End of Week (May 9-10):**
- Daria: Task 10 (complete pipeline)
- Alexa: Task 2 (test web app)
- Both: Integration testing

**Lower Priority (May 10+):**
- Daria: Task 11 (offer data - nice-to-have)

---

*For meeting context, see: [daria_project_discussion_may6.txt](../../meetings/daria_project_discussion_may6.txt)*
