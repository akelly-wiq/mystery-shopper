# Mystery Shopping Agent - Project Brief

**Last Updated:** May 4, 2026  
**Project Team:** CustomerX, Group Customer, B&M, Commercial, Data Science  
**Key Stakeholders:** CEO, Owen Lim, Alexa Kelly, Daria Volkova, Matt  
**Board Presentation:** May 18, 2026

---

## Executive Summary

The Mystery Shopping Agent is an automated, AI-powered solution that compares basket costs from a customer perspective across Woolworths, Coles, and Aldi. The system addresses critical price perception challenges where customer surveys indicate deteriorating price perception relative to Coles, despite price indices showing competitive parity. Using Claude-based AI agents and digital personas, the system executes 120 automated mystery shops (8 missions × 5 personas × 3 retailers) to understand how different customer segments experience pricing differences across retailers.

**Board Demonstration:** May 18, 2026 - Live demonstration with interactive front-end interface showcasing competitive pricing insights and AI decision-making capabilities.

---

## Business Context

### The Problem

**Price Perception Gap:**
- Woolworths' quantitative price index data shows parity with Coles
- Customer brand tracking reveals customers perceive Coles as cheaper
- Price perception vs. Coles remains challenged and is deteriorating
- Price competitiveness is the most important factor as costs to consumers re-accelerate in 2026

**Limitations of Current Approach:**
- Aggregated price indices are useful but may mask cost differences that customers actually experience
- Individual shopping experiences vary based on:
  - Product availability
  - Pack size differences
  - Promotional timing and placement
  - Customer personas and shopping behaviors
  - Substitution patterns

**Real-World Example:**
Pre-Christmas testing showed a specific recipe shop cost 50% more at Woolworths than competitors because Woolworths didn't carry certain on-brand SKUs, forcing customers to choose alternatives.

---

## Project Goals

### Primary Objectives
1. Build a scaled, automated mystery shopping measurement tool
2. Track price differences across retailers from actual customer perspectives
3. Understand drivers of price deltas (pack sizes, promotions, availability, range)
4. Improve customer price perception through data-driven insights
5. Run daily automated comparisons to build historical pricing database
6. Identify price trends over time and understand what drives price increases

### Success Criteria
- Robust, confident pricing data that captures promotional activity
- Accurate persona modeling that reflects real customer decision-making
- Automated daily execution across all retailers
- Centralized database for ongoing tracking and analysis
- Actionable insights for pricing strategy

---

## Current Status (as of May 4, 2026)

### System Architecture Built

**Technical Foundation:**
- Multi-agent voting system with Claude-based AI agents
- RAG retrieval component using BigQuery tables
- Semantic search with hybrid keyword + embedding approach
- Evaluation agent with rejection limits to prevent infinite loops
- Data pipeline: product extraction → agent voting → evaluation → BigQuery storage

**Personas Deployed:**
- Saver persona: Operational with agent reasoning
- Traditional persona: Operational with agent reasoning  
- Conscious, Refined, Essential personas: In development (Week 3)

**Data Infrastructure:**
- Woolworths internal table (attributes, allergens, promotions, Promise info)
- Coles competitor table (ingredients, allergens, nutrition, promotional data)
- Aldi base table + selective enrichment for 100 products across 8 missions
- Semantic search validated with 348-dimension embeddings
- Hybrid search combining keywords and semantic matching

**Board Presentation Preparation:**
- Interactive front-end interface in development
- Historical data integration via `gcp-wow-ent-im-tbl-prod.adp_dm_betterbuying_view.Sell_Price_Weekly_v`
- Demonstration scheduled for May 18, 2026

### Data Quality Resolution

**Solutions Implemented:**
- Coles product descriptions: 50% missing, LLM fallback logic implemented
- Price averaging: Using Mascot postcode as default location for consistency
- Aldi data: Pre-scraping approach (not on-the-fly) coordinated with pricing team
- Woolworths attributes: Confirmed allergens, country of origin, Australian ingredient % available
- Price validation: Multi-source validation with confidence scoring

---

## Solution Approach

### Mission-Based Shopping

Rather than bottom-up data-driven basket construction, the team has adopted a descriptive, mission-based approach:

**8 Defined Missions:**
1. I need to make Spaghetti Bolognese
2. I need to pack school lunches for next week
3. I'm hosting a BBQ for 6 people
4. Saver family weekly top-up shop
5. The Choice basket (well-known reference)
6. Sunrise Australia Day BBQ (recent TV publicity)
7. Easter shop (including confectionary)
8. Full weekly shop

**Why Mission-Based:**
- Easier for executives to understand
- Missions are interchangeable across retailers
- Reflects how real customers shop (goal-oriented)
- Allows agent to make decisions based on persona priorities

### Customer Personas

The project has defined detailed customer personas based on Woolworths' customer segmentation:

#### 1. **Saver**
- Time and cash poor, typically younger families with limited incomes
- Budget-focused, will stick to it
- Scan for Yellow/Red Tickets, look for cheapest or best value
- Will buy own brand and switch products/brands when on promotion
- Pack sizes critical for key items (need minimum quantity for meal planning)
- Will go to larger sizes if unit price is cheaper (batch cooking, freezing)
- Multibuy promotions must offer substantial discounts

#### 2. **Traditional**
- Stick to "tried & tested" brands for consistency and comfort
- Less likely to experiment with niche brands or cheaper alternatives
- Seek well-known, familiar brands (Leggos, Heinz, Cheer cheese)
- If preferred brand unavailable, prefer smaller pack size over brand switching
- Own brand is last resort
- Confident cooks who stick to familiar recipes

#### 3. **Conscious**
- Conscious of ingredients, environmental impact, and brand values
- Younger singles/couples or families with primary school age children
- Lead busy lives but balance with better choices (health, environment, social outcomes)
- Will pay for health, sustainability, and convenience if affordable
- Highly attuned to health trends, early adopters
- Check labels for ingredients
- Include dietary restriction shoppers (vegetarian, gluten-free, lactose intolerant)
- Convenience is important (ready meals, heat-and-eat)
- When budgets stretched, price often wins over eco-friendly products

#### 4. **Refined**
- Most affluent customers
- Prioritize quality over price
- Years of experience developing and refining tastes
- Maintaining sustainable practices important
- Willing to pay more for what matters
- Seek best ingredients and best products
- Promotions don't necessarily sway decisions (quality standards first)
- Less likely to buy own brand, more likely to sample premium
- Savvy shoppers who buy "mainstream" for basics, premium for special meals

#### 5. **Essential**
- Limited means, carefully manage budgets
- Less adventurous, prioritize price over quality
- Focus on essentials with smaller baskets
- Not overly savvy with saving strategies
- Buy own brand, keep it simple, stick to what they know
- Two types:
  - Top-up shoppers who shop around for bargains
  - Older customers with limited means and small diets
- High promotional engagement (yellow tickets)
- Less likely to stretch into larger packs or new products

### Agent Architecture

**Multi-Agent Voting System:**
The system uses three independent AI agents per ingredient to ensure consistency and reduce decision errors. Each agent evaluates product candidates using persona-specific decision logic, then votes on the best option. The product with the highest vote count is selected.

**Example Flow:**
```
Ingredient: "Lean mince beef, 1kg" (Saver persona)
→ Agent 1: Woolworths own brand (Red EDLP)
→ Agent 2: Woolworths own brand (Red EDLP)  
→ Agent 3: Coles Yellow ticket option
→ Result: Woolworths own brand selected (2/3 votes)
```

**Component 1: RAG Retrieval System**
- Queries BigQuery tables (Woolworths, Coles, Aldi) per ingredient
- Returns top candidate products with pricing, promotions, and attributes
- Filters by price range, pack size, category, and availability
- Uses hybrid semantic search (keywords + embeddings)

**Component 2: Evaluation Agent**
- Validates basket-level quality after all product selections
- Checks: coverage (all items found), balance (reasonable composition), price accuracy
- Implements rejection limits to prevent infinite processing loops
- Re-runs selection for problematic items if validation fails (max 3 attempts)

**Component 3: Basket-Level Judge**
- Cross-validates final basket price against Woolworths API
- Ensures persona alignment (selections match customer profile)
- Generates reasoning trail for transparency
- Flags low-confidence decisions for review

**Key Design Principle:**
Agents make autonomous SKU decisions based on mission and persona (like a human would), using voting for consistency rather than prescribing exact basket compositions.

---

## Scope and Constraints

### Geographic Scope
- **Limited to Sydney region** to reduce complexity and errors
- **Default location:** Mascot CFC store for Woolworths pricing
- **Competitor pricing:** Average across NSW locations
- Rationale: Prices vary by state, suburb, and product; limiting scope ensures robustness

### Data Sources

**Woolworths:**
- **Primary:** Internal product table (via Stuart contact)
- **Contains:** Product attributes, allergens, country of origin, Australian ingredient percentage, Promise information
- **Promotions:** Half price, low price special, price drop special flags
- **Historical:** `gcp-wow-ent-im-tbl-prod.adp_dm_betterbuying_view.Sell_Price_Weekly_v` for board presentation
- **Ongoing:** Real-time catalog for post-board production system

**Coles:**
- **Primary:** PCB competitor table (via Nissan)
- **Contains:** Product details, ingredients, allergens, nutrition, promotional information
- **Coverage:** ~20,000 products, NSW focus
- **Known issue:** 50% of products missing descriptions - LLM fallback logic implemented
- **Refresh:** Weekly (Fridays after PCB data refresh)

**Aldi:**
- **Primary:** PCB competitor table (in-store scraping, ~99% accuracy)
- **Enrichment:** Selective enrichment for ~100 products across 8 missions
- **Method:** Pre-scrape product descriptions (coordinated with pricing team, not on-the-fly)
- **Coverage:** ~2,000 products, NSW focus
- **Pricing:** In-store prices (most accurate), national with state/store exceptions

**Data Strategy:**
- **Board demo (May 18):** Historical data table for comprehensive showcase
- **Production (post-May 18):** Real-time PCB tables with weekly Friday refresh
- **Geographic scope:** Mascot CFC as default location for consistency
- **Refresh cycle:** Weekly Fridays (after PCB data refresh at 10:00 AM AEST)

### Technical Stack

**AI Framework:**
- Claude-based AI agents (Anthropic)
- Agent Development Kit (ADK) for multi-agent orchestration
- Multi-agent voting system with 3 agents per ingredient
- Evaluation agent for quality validation

**Data Layer:**
- BigQuery for product catalog and results storage
- Hybrid semantic search: keyword matching + 348-dimension embeddings
- RAG retrieval from BigQuery tables (not Vertex AI Search)
- Weekly data refresh pipeline via Cloud Functions

**Front-End:**
- Looker Studio (recommended - 15-20 hours, native BigQuery integration)
- Alternative: Streamlit (25-35 hours, more customization)
- Interactive dashboard for board presentation
- Export to Google Sheets for stakeholder reporting

**Infrastructure:**
- GCP project: australia-southeast1 region
- Cloud Functions for automated shop execution
- Cloud Run for Streamlit deployment (if selected)
- BigQuery ML for embedding generation

---

## Technical Requirements

### Core System Components

1. **Multi-Agent Decision Engine**
   - Claude-based agents with persona-specific prompts
   - 3-agent voting system per ingredient for consistency
   - Evaluation agent with max 3 rejection attempts
   - Confidence scoring on all decisions (0-1 scale)
   - Reasoning trail generation for transparency

2. **Data Pipeline**
   - RAG retrieval from BigQuery product catalog
   - Hybrid semantic search (keyword + 348-dimension embeddings)
   - Weekly data refresh from PCB tables (Fridays 10 AM)
   - Real-time Woolworths API validation for price accuracy
   - Multi-source validation with confidence scoring

3. **Front-End Interface**
   - Interactive dashboard for board presentation
   - Executive summary view (120 shops aggregated)
   - Basket comparison view (item-by-item detail)
   - Cost driver analysis (why price differences exist)
   - Trend analysis (8-week historical view)
   - Persona comparison (same mission, different personas)
   - Export to Google Sheets functionality

4. **Database Schema**
   - `shop_results` table: Item-level detail (~1,200 rows/week)
   - `basket_summaries` table: Basket totals (~120 rows/week)
   - `cost_drivers` table: Price difference explanations
   - Historical retention for trend analysis
   - BigQuery ML for embedding generation

### Quality Assurance

5. **Validation Framework**
   - Pricing accuracy: >98% within $0.50 of actual price
   - Coverage: >95% of mission items found across retailers
   - Persona alignment: Decisions match expected behavior patterns
   - Agent confidence: Average >0.80 across all selections
   - Execution performance: <5 minutes per mission

### Data Output Specification

**Table 1: shop_results (Item-Level Detail)**
Primary table containing every product selection with full context and reasoning.

Key columns:
- `shop_id`, `date_executed`, `week_label`
- `mission_name`, `mission_description`
- `persona`, `persona_description`
- `list_item`, `item_order` (ingredient and basket position)
- `retailer`, `product_id`, `product_name`, `brand`
- `pack_size`, `pack_size_normalized` (in grams/ml)
- `price`, `unit_price` (actual prices, no scaling)
- `promotion_type`, `promotion_description`, `promotion_flag`
- `agent_reasoning` (why this product was selected)
- `confidence_score` (0-1, agent decision confidence)
- `alternative_product_id` (backup if unavailable)
- `quality_markers` (organic, free range, etc.)
- `product_url` (for validation)
- `status` (found/not_found)

**Table 2: basket_summaries (Aggregate View)**
Basket-level totals and comparisons for executive reporting.

Key columns:
- `shop_id`, `date_executed`, `week_label`
- `mission_name`, `persona`, `retailer`
- `total_basket_price`, `num_items`, `num_items_found`
- `coverage_pct` (% of items successfully found)
- `num_promoted_items`, `num_own_brand_items`
- `woolworths_basket_price` (for comparison baseline)
- `price_vs_woolworths_dollar`, `price_vs_woolworths_pct`
- `is_woolworths_cheaper` (boolean flag)
- `execution_time_seconds`

**Table 3: cost_drivers (Insights)**
Explains WHY price differences exist between retailers.

Key columns:
- `shop_id`, `date_executed`, `mission_name`, `persona`
- `retailer_comparison` (e.g., "Coles vs Woolworths")
- `driver_type` (promotion, pack_size, base_price, product_selection)
- `driver_description` (human-readable explanation)
- `item_name`, `dollar_impact`, `pct_of_total_diff`

**Data Integrity:**
- Capture actual costs including pack size variances (no scaling)
- Example: If Coles only sells 800g mince beef, record the 800g price, not scaled to 1kg
- All prices to 2 decimal places, percentages to 1 decimal place
- Handle missing data gracefully (NULL or "N/A" with status flag)

---

## Board Presentation Strategy (May 18, 2026)

### Demonstration Objectives

**Primary Goal:** Showcase AI-powered competitive intelligence capability to CEO and board members

**Key Messages:**
1. **Automated at scale** - 120 shops executed automatically vs 3 people × 1.5 hours manual
2. **Persona insights** - Different customer segments experience different price perceptions
3. **Transparency** - AI reasoning visible and explainable for every product choice
4. **Actionable intelligence** - Clear cost drivers identified (promotions, pack sizes, product selection)

### Demo Flow (Recommended)

**Part 1: Executive Summary (2 minutes)**
- Show weekly aggregate: WW vs Coles vs Aldi across all 120 shops
- Highlight key finding: "Woolworths cheaper in only 2 of 8 missions for Saver persona"

**Part 2: Basket Deep-Dive (3 minutes)**
- Select "Spaghetti Bolognese - Saver persona"
- Item-by-item comparison across 3 retailers
- Show cost drivers: "Coles cheaper by $2.98 due to Yellow ticket on mince beef"

**Part 3: AI Transparency (2 minutes)**
- Click into product detail for mince beef
- Show agent reasoning: "Selected WW own brand for $12/kg as it meets 1kg pack size requirement..."
- Highlight confidence scoring and multi-agent voting

**Part 4: Persona Comparison (2 minutes)**
- Show how Conscious persona shops differently than Saver
- Same mission, different product selections based on values
- Demonstrate persona-specific insights

**Part 5: Trend Analysis (1 minute)**
- 8-week price trend showing WW vs competitors
- Identify week where gap widened and why

**Total: 10 minutes presentation + Q&A**

### Historical Data Showcase

**Purpose:** Demonstrate system with rich historical context for board presentation

**Data Source:** `gcp-wow-ent-im-tbl-prod.adp_dm_betterbuying_view.Sell_Price_Weekly_v`

**Post-Presentation Transition:** Switch to real-time PCB tables for production system

### Success Indicators

- Board engagement and questions (positive signal)
- Owen approval to proceed to production
- Recognition of AI decision-making quality
- Stakeholder requests for specific insights or expansions

---

## Project Timeline

### Phase 1: Foundation & Data Intelligence (Week 1: Apr 22-28) - ✅ COMPLETE
**Objectives:** Build data foundation and product retrieval system

**Deliverables:**
- ✅ Unified product catalog (~50K products across WW, Coles, Aldi)
- ✅ RAG retrieval component with semantic search
- ✅ Data enrichment strategy (Aldi products)
- ✅ Data quality validation (>90% completeness achieved)

**Checkpoint:** Friday Apr 25 - Data confirmed, retrieval operational, enrichment plan approved

---

### Phase 2: Multi-Agent Decision System (Week 2: Apr 29-May 5) - IN PROGRESS
**Objectives:** Build AI decision-making system with multi-agent architecture

**Deliverables:**
- ✅ ADK agent framework implemented
- ✅ Persona-based agents deployed (Saver, Traditional)
- 🔄 Multi-agent voting system (3 agents per ingredient)
- 🔄 Basket-level evaluation agent with rejection limits
- 🔄 Test missions complete (2 personas, sample baskets)

**Success Criteria:**
- Agents make persona-differentiated decisions
- Multi-agent voting achieves consistency (majority agreement)
- Basket validation catches errors and inconsistencies

**Checkpoint:** Friday May 2 - Agent decisions align with persona rules, voting system operational

---

### Phase 3: Scale to Full Scope (Week 3: May 6-12)
**Objectives:** Expand to complete system - all personas, all missions

**Deliverables:**
- All 5 CREST personas implemented (add Conscious, Refined, Essential)
- All 8 shopping missions configured and tested
- Full matrix execution: 120 shops (8 missions × 5 personas × 3 retailers)
- Price validation against Woolworths API
- Front-end interface development begins

**Success Criteria:**
- All 120 automated shops complete successfully
- Coverage >95% (ingredients found across retailers)
- Price accuracy >98% (validated against Woolworths API)
- Execution time <5 minutes per mission

**Checkpoint:** Friday May 9 - Full scope operational, coverage targets met

---

### Phase 4: Board Presentation Readiness (Week 4: May 13-18)
**Objectives:** Front-end polish, validation, board demonstration

**Deliverables:**
- Interactive front-end dashboard (Looker Studio or Streamlit)
- Historical data integration for comprehensive showcase
- Analytics dashboards with basket comparisons and insights
- Google Sheets export functionality
- Board presentation materials and demonstration script
- System documentation and known limitations

**Success Criteria:**
- Front-end operational and stakeholder-ready
- Board demonstration successful (May 18)
- Analytics deliver actionable competitive insights
- System approved for production transition

**Go-Live:** Monday May 18 - Board presentation and demonstration

---

### Phase 5: Production Transition (Post-May 18)
**Objectives:** Transition from historical showcase to live production system

**Deliverables:**
- Switch from historical data table to real-time PCB tables
- Weekly automated execution schedule (Fridays 10 AM)
- Monitoring and alerting for data quality
- Stakeholder training and handoff
- Production runbook and support documentation

---

## Key Decisions Made

1. **Multi-agent voting architecture** - 3 agents vote per ingredient for consistency and error reduction
2. **Evaluation agent with rejection limits** - Max 3 attempts to prevent infinite loops
3. **Mission-based approach** - Goal-oriented shopping over data-driven basket construction
4. **Claude-based agents** - Using ADK framework for multi-agent orchestration
5. **PCB tables as primary data source** - Validated competitor data, weekly Friday refresh
6. **Limited geographic scope** - Mascot CFC as default location for consistency
7. **Two-phase data strategy** - Historical table for board demo, real-time for production
8. **Hybrid semantic search** - Keywords + embeddings (not Vertex AI Search)
9. **Selective Aldi enrichment** - Pre-scrape ~100 products for 8 missions (not on-the-fly)
10. **LLM fallback for missing data** - Handle 50% missing Coles descriptions gracefully
11. **Capture actual prices** - Including pack size variances (no scaling for comparability)
12. **Front-end for board presentation** - Interactive dashboard to showcase AI capabilities
13. **May 18 board deadline** - Compressed 4-week timeline with weekly checkpoints

---

## Risk Considerations

### Data Quality Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| **50% Coles products missing descriptions** | Medium | LLM fallback logic for attribute extraction |
| **Price discrepancies** ($0.50-$1.00 variance) | Medium | Multi-source validation with confidence scoring |
| **Aldi online vs in-store pricing** | Low | Use in-store prices (most accurate, PCB source) |
| **Multi-postcode pricing (Coles)** | Medium | Use Mascot postcode as consistent default |
| **Missing Woolworths attributes** | Low | Confirmed available: allergens, origin, promotions |

### Technical Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Agent voting inconsistency** | High | 3-agent voting + basket evaluation agent |
| **Infinite evaluation loops** | Medium | Rejection limits (max 3 attempts) |
| **Low agent confidence scores** | Medium | Track confidence, flag <0.70 for review |
| **Front-end development timeline** | Medium | Looker Studio option (faster) vs Streamlit (flexible) |
| **Execution performance** | Low | Target <5 min/mission, parallel processing |

### Strategic Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| **May 18 board deadline** | High | Weekly checkpoints, compressed timeline, clear go/no-go criteria |
| **External competitive development** | Medium | Accelerated delivery, showcase Claude differentiation |
| **Board approval uncertainty** | Medium | Strong demonstration, clear business value, interactive demo |
| **Production transition complexity** | Low | Phased approach: historical demo → production system |

### Mitigation Strategies
- **Weekly checkpoints:** Friday reviews with clear go/no-go criteria
- **Multi-agent voting:** Reduces individual agent error rate
- **Evaluation agent:** Catches systematic quality issues
- **Phased rollout:** Historical showcase first, production system second
- **Front-end flexibility:** Two options (Looker Studio fast track, Streamlit customization)

---

## Current Focus (Week 2: May 6-12)

### Technical Priorities

1. **Complete Multi-Agent Voting System**
   - Finalize 3-agent voting implementation
   - Test consistency across multiple ingredients
   - Validate majority-wins logic

2. **Deploy Evaluation Agent**
   - Implement rejection limits (max 3 attempts)
   - Build basket-level validation checks
   - Test coverage, balance, and price accuracy validation

3. **Expand Persona Coverage**
   - Add Conscious persona (values-driven, sustainability focus)
   - Add Refined persona (quality over price)
   - Add Essential persona (budget-constrained, simplicity)

4. **Prompt Engineering**
   - Refine persona-specific decision prompts
   - Develop examples for each customer segment
   - Test prompt effectiveness across product categories

### Data & Infrastructure

5. **Complete Aldi Enrichment**
   - Pre-scrape descriptions for ~100 priority products
   - Coordinate with pricing team on scraping methodology
   - Cache enriched data in BigQuery

6. **Historical Data Integration**
   - Connect to `gcp-wow-ent-im-tbl-prod.adp_dm_betterbuying_view.Sell_Price_Weekly_v`
   - Validate historical data quality and completeness
   - Prepare data views for board presentation

7. **Front-End Development Kickoff**
   - Finalize technology choice (Looker Studio vs Streamlit)
   - Begin dashboard layout and component design
   - Connect to BigQuery data tables

### Board Presentation Preparation

8. **Demo Script Development**
   - Identify key insights to showcase
   - Prepare compelling narratives from basket comparisons
   - Highlight AI decision-making transparency (reasoning trails)

9. **Stakeholder Coordination**
   - Matthew: Address project information sharing concerns
   - Owen: Weekly Friday checkpoint and approval
   - PCB team: Data quality validation and support

---

## Resources and Links

**Documentation:**
- Master pack presentation (Google Slides)
- Mystery shop basket items (Google Sheet)
- Existing Gem version
- Mystery shopping agent Google Drive

**Key Data Sources:**
- Competitor price index tables (PCB team)
- Woolworths internal product catalog
- Web APIs for Woolworths, Coles, Aldi

**Technology Stack:**
- Google Gemini agents
- Google Gems (current beta)
- Vertex AI Search (proposed)
- GCP database
- BigQuery for data analysis

---

## Success Metrics

### Board Presentation (May 18, 2026)

**Quality Metrics:**
- Coverage: >95% of mission items found across all retailers
- Price accuracy: >98% within $0.50 of actual retail price
- Agent confidence: Average >0.80 across all product selections
- Persona alignment: Decisions match expected behavior patterns

**System Performance:**
- Execution time: <5 minutes per mission (complete basket)
- System reliability: 120 shops complete successfully
- Data freshness: Weekly refresh cycle operational

**Stakeholder Metrics:**
- Interactive dashboard operational and responsive
- Cost driver insights clear and actionable
- Board demonstration successful
- Owen approval for production transition

### Production System (Post-May 18)

**Operational Metrics:**
- Weekly automated execution: Every Friday 10 AM AEST
- Historical database: Trend analysis over 8+ weeks
- Data quality monitoring: Automated alerts for anomalies
- Stakeholder usage: Regular access and exports to Google Sheets

**Business Impact Metrics:**
- Actionable competitive insights: Price gap drivers identified
- Persona-specific insights: Different customer experiences quantified
- Time savings: Automated vs manual mystery shopping (3 people × 1.5 hours)
- Strategic value: Integration with pricing strategy decisions

### Long-Term Vision (6-12 Months)

- Daily automated execution (if business need identified)
- Expansion to additional missions or retailers
- Integration with pricing mechanisms and decision workflows
- Measurable improvement in customer price perception tracking
- Regional expansion (beyond Sydney/Mascot if validated)

---

## Appendix: Basket Compositions

### Mission 1: Spaghetti Bolognese (Traditional Persona)
- Lean mince beef, 1kg
- Spaghetti/Dry pasta, 500g
- Pasta sauce, 500g
- Tomato paste, 140g tub
- Brown onions, 1kg
- Mushrooms, 200g
- Carrots, 1kg
- Parmesan cheese, shaved 200g
- Garlic bread, twin pack ~450g
- Olive oil, 700ml

### Mission 2: School Lunches (Saver Persona)
- Sandwich bread, 1 loaf ~700g
- Wraps, pack of 8
- Deli ham, 600g
- Apples, 1kg
- Bananas, 1kg
- White seedless grapes, 1kg
- Tasty cheese block, 500g
- Yoghurt pouches, 12 x 100g
- Muesli bars, 6 bar box (185g)
- Multipack chips, 6-pack
- Juice boxes, 6x250ml pack
- Sandwich bags, 40 pack box

### Mission 3: BBQ for 6 People (Conscious Persona)
- Beef sausages, 1kg
- Beef hamburger patties, 6 pack
- Sandwich bread, 1 loaf ~700g
- Hamburger rolls, x6
- Tomato sauce, 500ml
- BBQ sauce, 500ml
- Coleslaw salad kit, ~450g
- Potato salad, ~500g pre pack
- Paper plates, 20 pack
- Paper towels, 2 pack
- Soft drink, multipack cans 10 x 375ml
- Potato chips, 2x 165g

### Mission 4: Saver Family Weekly Shop (Saver Persona)
- Sandwich bread, 1 loaf ~700g
- Bananas, 1kg
- Strawberries, 2x250g
- Onions, 1kg
- Tomatoes, 1kg
- Apples, 1kg
- Minced beef, 1kg
- Chicken breast, 1kg
- Full cream milk, 3L
- Kids yoghurt pouches, 10x70g
- Dry pasta, 500g
- Bolognese pasta sauce, 500g
- Loose washed potatoes, 1kg
- Potato chips, 2x 165g
- Muesli bars, 6pk, 185g

### Mission 5: The Choice Basket (Traditional Persona)
- Chicken breasts, bulk pack (1kg)
- Carrots, 1kg
- Royal Gala apples, 1kg
- Cavendish bananas, 1kg
- Strawberries, 250g
- Full cream milk, 3L
- Weet-bix, 1kg
- Beef mince, 1kg
- Block of tasty cheese, 1kg
- Sliced white bread, 650g
- Penne pasta, 500g
- Tinned diced tomatoes, 400g
- Frozen peas, 1kg
- Tea bags, 100 pack
- Brown onions, 1kg

### Mission 6: Sunrise Australia Day BBQ (Saver Persona)
- Lamb cutlets, 1kg
- Beef sausages, 1kg
- Beef burger patties, 6 pack
- Chicken drumsticks, 1kg
- Sunscreen, 250ml tube
- Paddle pops, 8 pack
- Kettle chips, 175g
- White bread, 650g loaf
- Tomato sauce, 500ml
- Coca-Cola, 1.25L

### Mission 7: Easter Shop (Saver Persona)
- Hot cross buns (Traditional), 6 pack
- Hot cross buns (Chocolate), 6 pack
- Solid chocolate eggs, 125g-150g
- Hollow hunting easter eggs, 170g (10 pack)
- Chocolate rabbit (gold bunny), 100g
- Double brie cheese, 200g
- Strawberries, 250g
- Pancake shaker, 325g
- Maple syrup, 250ml
- Fresh salmon fillet, 200g
- Prawns (thawed), 1kg
- Boneless Pork Leg Roast, 1kg

### Mission 8: Full Weekly Shop (Saver Persona)
Complete household shop including:
- Fresh produce (bread, bananas, strawberries, onions, tomatoes, apples, avocado)
- Proteins (minced beef, chicken breast)
- Dairy (milk, yoghurt pouches, butter, cheese, eggs, cream)
- Pantry (pasta, sauce, potatoes, chips, muesli bars, tuna, Weet-bix, juice, water)
- Household (laundry liquid, dishwashing liquid, dishwashing tabs, toilet tissue)
- Baby items (nappies, baby wipes)
- Frozen meals

---

*This brief synthesizes information from meetings held Feb 25 - April 20, 2026, and project documentation including the Comparative Basket Views presentation and Mystery Shopping Agent Overview.*
