# Mystery Shopping Agent Architecture - REVISED Design

**Created:** May 5, 2026  
**Last Updated:** May 6, 2026  
**Status:** Current Architecture

---

## Executive Summary

**Architecture:** Per-vendor agents with ingredient-level evaluation and justification.

**Key Flow:**
1. **Vendor Agent** (one per retailer: WW, Coles, Aldi)
2. **Vertex AI Search** retrieves top 3 candidate products per ingredient
3. **LLM with Persona Prompt** makes product decisions from top 3
4. **Evaluation Agent** provides justification per ingredient
5. **Rich Output to BigQuery** with full traceability and decisioning attributes

**Recent Changes (May 6):**
- Switched to Vertex AI Search for all retailers (was local embedding for Coles/Aldi)
- Top 3 products per ingredient (was 5)
- Output schema enriched with justification text and decisioning attributes
- Unit normalization pre-processing (planned)

---

## High-Level Architecture

```mermaid
flowchart TD
    Start([Start: Mission + Persona]) --> VendorSplit{Create Vendor Agents}
    
    %% Three Parallel Vendor Agents
    VendorSplit --> WW[Woolworths Agent<br/>INPUT:<br/>- Recipe/Mission<br/>- WW Product Catalog<br/>- Persona Prompt]
    VendorSplit --> Coles[Coles Agent<br/>INPUT:<br/>- Recipe/Mission<br/>- Coles Product Catalog<br/>- Persona Prompt]
    VendorSplit --> Aldi[Aldi Agent<br/>INPUT:<br/>- Recipe/Mission<br/>- Aldi Product Catalog<br/>- Persona Prompt]
    
    %% Each processes ingredients
    WW --> WWProcess[Process All Ingredients<br/>Sequentially]
    Coles --> ColesProcess[Process All Ingredients<br/>Sequentially]
    Aldi --> AldiProcess[Process All Ingredients<br/>Sequentially]
    
    %% Send to evaluation
    WWProcess --> WWResults[WW Basket Complete]
    ColesProcess --> ColesResults[Coles Basket Complete]
    AldiProcess --> AldiResults[Aldi Basket Complete]
    
    %% Output to BigQuery
    WWResults --> BQ[BigQuery Output<br/>Rich Data Table]
    ColesResults --> BQ
    AldiResults --> BQ
    
    BQ --> Complete([Complete:<br/>3 Baskets for Mission])
    
    style WW fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style Coles fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    style Aldi fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    style BQ fill:#f3e5f5,stroke:#7b1fa2,stroke-width:3px
```

---

## Detailed Per-Vendor Agent Flow

### Single Vendor Agent (e.g., Woolworths)

```mermaid
flowchart TD
    Input[VENDOR AGENT INPUT] --> Recipe[Recipe/Mission:<br/>Spaghetti Bolognese<br/>10 ingredients]
    Input --> VertexAI[Vertex AI Search:<br/>Woolworths Index<br/>4 weeks historical data]
    Input --> Persona[Persona Prompt:<br/>SAVER decision framework]
    
    Recipe --> IngredientLoop{For Each<br/>Ingredient}
    
    IngredientLoop -->|Ingredient 1| Search1[Vertex AI Search<br/>Query: 'Mince beef 1kg'<br/>Returns: Top 3 candidates]
    
    Search1 --> LLM1[LLM Decision Engine<br/>Persona: SAVER<br/>Ingredient: Mince beef 1kg<br/>Available: Top 3 products]
    
    LLM1 --> Decision1[Product Selected:<br/>WW Own Brand Mince $12/kg]
    
    Decision1 --> Eval1[Evaluation Agent<br/>INPUT:<br/>- Selected product<br/>- Ingredient requirement<br/>- Persona prompt<br/>OUTPUT:<br/>- Justification<br/>- Confidence score]
    
    Eval1 --> Justification1[JUSTIFICATION:<br/>Yellow ticket special<br/>Best unit price for Saver<br/>Meets 1kg requirement<br/>Own brand = Saver default]
    
    Justification1 --> Store1[Store to Basket:<br/>+ Product details<br/>+ Justification<br/>+ Confidence]
    
    Store1 --> IngredientLoop
    
    IngredientLoop -->|Ingredient 2| LLM2[LLM Decision Engine<br/>Ingredient: Pasta 500g]
    LLM2 --> Decision2[Product Selected]
    Decision2 --> Eval2[Evaluation Agent]
    Eval2 --> Store2[Store to Basket]
    Store2 --> IngredientLoop
    
    IngredientLoop -->|All Done| Output[OUTPUT TO BIGQUERY<br/>Complete Basket<br/>All Justifications]
    
    style LLM1 fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style LLM2 fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
    style Eval1 fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style Eval2 fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style Output fill:#e1bee7,stroke:#7b1fa2,stroke-width:3px
```

---

## LLM Decision Engine Detail

### How Product Selection Works

```mermaid
flowchart LR
    subgraph DecisionEngine["🤖 LLM Decision Engine (Per Ingredient)"]
        direction TB
        
        I1[Persona Prompt<br/>SAVER:<br/>- Price first<br/>- Unit price comparison<br/>- Own brand default<br/>- Yellow/Red tickets<br/>- Bulk if better $/unit] --> LLM
        
        I2[Ingredient Requirement<br/>'Lean mince beef, 1kg'] --> LLM
        
        I3[Top 3 from Vertex AI:<br/>Woolworths Candidates:<br/>1. Own brand mince $12/kg Yellow<br/>2. Premium mince $18/kg<br/>3. 1.5kg pack $16.50] --> LLM
        
        LLM[Claude LLM<br/>Product Selection] --> Selection[SELECTED PRODUCT<br/>WW Own Brand 1kg $12]
        
        Selection --> ToEval[Send to Evaluation Agent]
    end
    
    style DecisionEngine fill:#e8f5e9,stroke:#2e7d32,stroke-width:3px
    style LLM fill:#a5d6a7,stroke:#388e3c,stroke-width:2px
    style Selection fill:#fff59d,stroke:#f57f17,stroke-width:2px
```

---

## Evaluation Agent Detail

### Per-Ingredient Justification

```mermaid
flowchart TD
    subgraph EvalAgent["🔍 Evaluation Agent (Per Ingredient)"]
        direction TB
        
        Input[EVALUATION INPUT] --> Selected[Selected Product:<br/>WW Own Brand Mince 1kg $12]
        Input --> Ingredient[Ingredient Requirement:<br/>'Lean mince beef, 1kg']
        Input --> PersonaCheck[Persona Prompt:<br/>SAVER decision rules]
        
        Selected --> Validate{Validate Selection}
        Ingredient --> Validate
        PersonaCheck --> Validate
        
        Validate --> Check1[✅ Price Check:<br/>$12/kg competitive?]
        Validate --> Check2[✅ Persona Alignment:<br/>Own brand = Saver default?]
        Validate --> Check3[✅ Requirement Match:<br/>1kg = exactly what's needed?]
        Validate --> Check4[✅ Promotion Check:<br/>Yellow ticket caught?]
        
        Check1 --> Generate
        Check2 --> Generate
        Check3 --> Generate
        Check4 --> Generate
        
        Generate[Generate Justification] --> Output[OUTPUT:<br/><br/>JUSTIFICATION:<br/>'Selected WW Own Brand mince<br/>because yellow ticket special<br/>provides best unit price at $12/kg,<br/>meets exact 1kg requirement,<br/>and own brand aligns with<br/>Saver budget-first priority'<br/><br/>CONFIDENCE: 0.94]
    end
    
    style EvalAgent fill:#fff3e0,stroke:#e65100,stroke-width:3px
    style Generate fill:#ffecb3,stroke:#ff6f00,stroke-width:2px
    style Output fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
```

---

## Complete Data Flow

### From Mission to BigQuery

```mermaid
flowchart TD
    Mission[MISSION:<br/>Spaghetti Bolognese<br/>Persona: SAVER<br/>Date: 2026-05-10] --> Process{Process<br/>3 Vendors}
    
    %% Woolworths
    Process --> WW[WOOLWORTHS AGENT]
    WW --> WWIng1[Ingredient 1: Mince<br/>LLM → Select product<br/>Eval → Justify]
    WWIng1 --> WWIng2[Ingredient 2: Pasta<br/>LLM → Select product<br/>Eval → Justify]
    WWIng2 --> WWIng3[... 10 ingredients ...]
    WWIng3 --> WWBasket[Woolworths Basket Complete]
    
    %% Coles
    Process --> Coles[COLES AGENT]
    Coles --> ColesIng1[Ingredient 1: Mince<br/>LLM → Select product<br/>Eval → Justify]
    ColesIng1 --> ColesIng2[Ingredient 2: Pasta<br/>LLM → Select product<br/>Eval → Justify]
    ColesIng2 --> ColesIng3[... 10 ingredients ...]
    ColesIng3 --> ColesBasket[Coles Basket Complete]
    
    %% Aldi
    Process --> Aldi[ALDI AGENT]
    Aldi --> AldiIng1[Ingredient 1: Mince<br/>LLM → Select product<br/>Eval → Justify]
    AldiIng1 --> AldiIng2[Ingredient 2: Pasta<br/>LLM → Select product<br/>Eval → Justify]
    AldiIng2 --> AldiIng3[... 10 ingredients ...]
    AldiIng3 --> AldiBasket[Aldi Basket Complete]
    
    %% BigQuery Output
    WWBasket --> BQ[BIGQUERY TABLE]
    ColesBasket --> BQ
    AldiBasket --> BQ
    
    BQ --> Data[RICH DATA OUTPUT:<br/>- date_run: 2026-05-10<br/>- crest_persona: SAVER<br/>- recipe: Spaghetti Bolognese<br/>- vendor: Woolworths/Coles/Aldi<br/>- ingredient: Mince beef 1kg<br/>- product_id: WW123456<br/>- product_name: WW Own Brand Mince<br/>- price: $12.00<br/>- unit_price: $12/kg<br/>- unit_normalized: $12/kg (NEW)<br/>- promotion: Yellow Ticket<br/>- justification_text: 'Selected because...' (NEW)<br/>- decisioning_attributes: JSON (NEW)<br/>- confidence: 0.94<br/>... 30 rows total for this mission]
    
    style WW fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style Coles fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    style Aldi fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    style BQ fill:#f3e5f5,stroke:#7b1fa2,stroke-width:3px
    style Data fill:#e1bee7,stroke:#6a1b9a,stroke-width:3px
```

---

## BigQuery Output Schema

### Rich Data Table Structure

```mermaid
graph TD
    subgraph BigQueryTable["shop_results_detailed Table"]
        direction TB
        
        Meta["METADATA FIELDS<br/>- run_id<br/>- date_run<br/>- week_label<br/>- execution_timestamp"]
        
        Mission["MISSION FIELDS<br/>- mission_id<br/>- mission_name<br/>- mission_description<br/>- persona_crest<br/>- persona_description"]
        
        Vendor["VENDOR FIELDS<br/>- vendor<br/>- vendor_location<br/>- vendor_category"]
        
        Ingredient["INGREDIENT FIELDS<br/>- ingredient_order<br/>- ingredient_description<br/>- ingredient_quantity<br/>- ingredient_category"]
        
        Product["PRODUCT FIELDS<br/>- product_id<br/>- product_name<br/>- brand<br/>- pack_size<br/>- pack_size_normalized<br/>- price<br/>- unit_price<br/>- promotion_type<br/>- promotion_description<br/>- product_url"]
        
        Decision["DECISION FIELDS<br/>- justification_text (NEW)<br/>- decisioning_attributes (NEW)<br/>- confidence_score<br/>- alternative_product_id<br/>- selection_timestamp<br/>- decision_metadata"]
        
        Quality["QUALITY FIELDS<br/>- organic_flag<br/>- free_range_flag<br/>- australian_made_flag<br/>- own_brand_flag<br/>- status_found"]
    end
    
    style Meta fill:#e3f2fd,stroke:#1565c0,stroke-width:2px
    style Mission fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px
    style Vendor fill:#fff3e0,stroke:#ef6c00,stroke-width:2px
    style Ingredient fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    style Product fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style Decision fill:#ffcdd2,stroke:#c62828,stroke-width:2px
    style Quality fill:#b2dfdb,stroke:#00695c,stroke-width:2px
```

---

## Vertex AI Search Architecture (NEW - May 6, 2026)

### Search Strategy

**Decision:** Use Vertex AI Search for all three retailers to improve accuracy.

**Rationale:**
- Issue identified: Simple keyword search returned irrelevant results (e.g., "chicken breast bulk pack" → "nuggets")
- Solution: Vertex AI Search provides semantic search with better ingredient matching
- Applies to: Woolworths, Coles, and Aldi

**Implementation:**
```mermaid
flowchart LR
    subgraph VertexAI["Vertex AI Search Infrastructure"]
        direction TB
        
        WW[Woolworths Index<br/>~20K products<br/>4 weeks historical]
        Coles[Coles Index<br/>~20K products<br/>4 weeks historical]
        Aldi[Aldi Index<br/>~2K products<br/>4 weeks historical]
        
        WW --> Query[Search Query:<br/>'Lean mince beef 1kg']
        Coles --> Query
        Aldi --> Query
        
        Query --> Top3[Return Top 3<br/>Most Relevant Products]
        Top3 --> Agent[Send to LLM<br/>Decision Engine]
    end
    
    style VertexAI fill:#e8f5e9,stroke:#2e7d32,stroke-width:3px
    style Top3 fill:#fff59d,stroke:#f57f17,stroke-width:2px
    style Agent fill:#a5d6a7,stroke:#388e3c,stroke-width:2px
```

**Index Configuration:**
- Index count: TBD (awaiting decision on per-retailer vs per-week indexing)
- Historical scope: 4 weeks
- Product selection: Top 3 candidates per ingredient
- Refresh strategy: Weekly

---

## Unit Normalization Pre-Processing (PLANNED)

### Challenge

Products sold in different units (e.g., bananas sold "each" vs required "1 kilogram") require price normalization for fair comparison.

### Solution (In Development)

**Pre-processing step** to normalize units BEFORE sending to persona agent:

```mermaid
flowchart TD
    Raw[Raw Product Data<br/>Bananas: $0.50 each<br/>Mince: $12 per 1kg<br/>Milk: $3.50 per 2L] --> Normalize[Unit Normalization<br/>LLM or Pre-processing Tool]
    
    Normalize --> Normalized[Normalized Data<br/>Bananas: ~$1.00/kg<br/>Mince: $12/kg<br/>Milk: $1.75/L]
    
    Normalized --> Search[Vertex AI Search]
    Search --> Agent[Persona Agent<br/>Makes Decision on<br/>Normalized Prices]
    
    style Normalize fill:#fff9c4,stroke:#f57f17,stroke-width:2px
    style Normalized fill:#c8e6c9,stroke:#2e7d32,stroke-width:2px
```

**Approach Options:**
1. LLM-based normalization call (flexible, handles edge cases)
2. Rule-based pre-processing tool (faster, deterministic)
3. Hybrid (rules + LLM fallback)

**Status:** Planning phase - implementation approach TBD

**Action Item:** Alexa to develop normalization solution using existing data frame in B data test location

---

## Recent Architecture Changes

### May 6, 2026 Updates

**Search Strategy:**
- ✅ **CHANGED:** Local embedding → Vertex AI Search for all retailers
- ✅ **CHANGED:** Top 5 products → Top 3 products
- **Reason:** Improved accuracy for ingredient matching

**Output Schema:**
- ✅ **ADDED:** `justification_text` field for decision transparency
- ✅ **ADDED:** `decisioning_attributes` JSON field for all attributes used in decision
- ✅ **PLANNED:** `unit_normalized` field for price comparisons

**Data Pipeline:**
- ✅ **CONFIRMED:** 4 weeks historical data (12 weeks technically available)
- ✅ **PLANNED:** Unit normalization pre-processing step

**Testing:**
- ✅ Use 'parallel agents' branch for testing (main branch deprecated)
- ✅ Test location: `cd app` directory
