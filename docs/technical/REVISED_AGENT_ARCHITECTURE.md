# Mystery Shopping Agent Architecture - REVISED Design

**Updated:** May 5, 2026 (Post-Catchup)  
**Status:** Current Architecture  
**Previous Version:** AGENT_ARCHITECTURE_DESIGN.md

---

## Executive Summary

**Architecture:** Per-vendor agents with ingredient-level evaluation and justification.

**Key Flow:**
1. **Vendor Agent** (one per retailer: WW, Coles, Aldi)
2. **Recipe + Full Product List** as input
3. **LLM with Persona Prompt** makes product decisions
4. **Evaluation Agent** provides justification per ingredient
5. **Rich Output to BigQuery** with full traceability

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
    Input --> Products[Full Product List:<br/>Woolworths Catalog<br/>~20K products]
    Input --> Persona[Persona Prompt:<br/>SAVER decision framework]
    
    Recipe --> IngredientLoop{For Each<br/>Ingredient}
    
    IngredientLoop -->|Ingredient 1| LLM1[LLM Decision Engine<br/>Persona: SAVER<br/>Ingredient: Mince beef 1kg<br/>Available: WW product list]
    
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
        
        I3[Full Product List<br/>Woolworths Catalog:<br/>- Own brand mince $12/kg Yellow<br/>- Premium mince $18/kg<br/>- 1.5kg pack $16.50<br/>- 800g pack $11<br/>... all available options] --> LLM
        
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
    
    BQ --> Data[RICH DATA OUTPUT:<br/>- date_run: 2026-05-10<br/>- crest_persona: SAVER<br/>- recipe: Spaghetti Bolognese<br/>- vendor: Woolworths/Coles/Aldi<br/>- ingredient: Mince beef 1kg<br/>- product_id: WW123456<br/>- product_name: WW Own Brand Mince<br/>- price: $12.00<br/>- unit_price: $12/kg<br/>- promotion: Yellow Ticket<br/>- justification: 'Selected because...'<br/>- confidence: 0.94<br/>... 30 rows total for this mission]
    
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
        
        Meta[METADATA FIELDS<br/>- run_id<br/>- date_run<br/>- week_label<br/>- execution_timestamp]
        
        Mission[MISSION FIELDS<br/>- mission_id<br/>- mission_name<br/>- mission_description<br/>- persona (CREST)<br/>- persona_description]
        
        Vendor[VENDOR FIELDS<br/>- vendor<br/>- vendor_location<br/>- vendor_category]
        
        Ingredient[INGREDIENT FIELDS<br/>- ingredient_order<br/>- ingredient_description<br/>- ingredient_quantity<br/>- ingredient_category]
        
        Product[PRODUCT FIELDS<br/>- product_id<br/>- product_name<br/>- brand<br/>- pack_size<br/>- pack_size_normalized<br/>- price<br/>- unit_price<br/>- promotion_type<br/>- promotion_description<br/>- product_url]
        
        Decision[DECISION FIELDS<br/>- justification (LLM reasoning)<br/>- confidence_score<br/>- alternative_product_id<br/>- selection_timestamp]
        
        Quality[QUALITY FIELDS<br/>- organic_flag<br/>- free_range_flag<br/>- australian_made_flag<br/>- own_brand_flag<br/>- status (found/not_found)]
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

## Implementation Pseudocode

### Vendor Agent

```python
class VendorAgent:
    """
    Agent for a single vendor (Woolworths, Coles, or Aldi).
    Processes all ingredients for a mission.
    """
    
    def __init__(self, vendor_name, product_catalog, persona):
        self.vendor = vendor_name
        self.products = product_catalog  # Full catalog for this vendor
        self.persona = persona
        self.llm = Claude()
        self.eval_agent = EvaluationAgent(persona)
    
    def process_mission(self, mission):
        """
        Process all ingredients in a mission/recipe.
        
        Args:
            mission: Mission object with ingredients list
        
        Returns:
            Complete basket with justifications
        """
        basket = []
        
        for ingredient in mission.ingredients:
            # LLM makes product selection
            selected_product = self.select_product(ingredient)
            
            # Evaluation agent provides justification
            justification = self.eval_agent.justify_selection(
                selected_product=selected_product,
                ingredient=ingredient,
                persona=self.persona
            )
            
            # Store result
            basket.append({
                'vendor': self.vendor,
                'ingredient': ingredient,
                'product': selected_product,
                'justification': justification.text,
                'confidence': justification.confidence,
                'timestamp': datetime.now()
            })
        
        return basket
    
    def select_product(self, ingredient):
        """
        LLM selects best product for ingredient given persona.
        
        Args:
            ingredient: Ingredient specification (e.g., "Mince beef 1kg")
        
        Returns:
            Selected product from catalog
        """
        prompt = f"""
        You are a {self.persona.name} shopper at {self.vendor}.
        
        PERSONA DECISION FRAMEWORK:
        {self.persona.decision_framework}
        
        INGREDIENT NEEDED:
        {ingredient.description}
        Quantity: {ingredient.quantity}
        
        AVAILABLE PRODUCTS:
        {self.format_product_catalog(ingredient)}
        
        Select the best product that matches your persona's priorities.
        Return the product_id only.
        """
        
        response = self.llm.complete(prompt)
        return self.products.get(response.product_id)
    
    def format_product_catalog(self, ingredient):
        """
        Format relevant products from catalog for LLM.
        Filters by category, sorts by relevance.
        """
        relevant_products = self.products.filter(
            category=ingredient.category,
            # Could add more filters here
        )
        
        # Return formatted list with all details
        return "\n".join([
            f"- {p.id}: {p.name} | {p.brand} | {p.pack_size} | "
            f"${p.price} ({p.unit_price}) | "
            f"{'YELLOW TICKET' if p.promotion == 'yellow' else ''}"
            for p in relevant_products
        ])


class EvaluationAgent:
    """
    Provides justification for product selections.
    Validates alignment with persona and requirements.
    """
    
    def __init__(self, persona):
        self.persona = persona
        self.llm = Claude()
    
    def justify_selection(self, selected_product, ingredient, persona):
        """
        Generate justification for why product was selected.
        
        Args:
            selected_product: The product that was selected
            ingredient: What was required
            persona: Shopping persona
        
        Returns:
            Justification object with text and confidence
        """
        prompt = f"""
        A {persona.name} shopper selected this product:
        
        SELECTED: {selected_product.name}
        - Brand: {selected_product.brand}
        - Price: ${selected_product.price}
        - Unit Price: {selected_product.unit_price}
        - Pack Size: {selected_product.pack_size}
        - Promotion: {selected_product.promotion or 'None'}
        
        FOR INGREDIENT: {ingredient.description}
        
        PERSONA RULES:
        {persona.decision_framework}
        
        Provide a clear justification for why this product aligns with 
        the {persona.name} persona's priorities. Also assess confidence (0-1).
        
        Output format:
        {{
            "justification": "Selected because...",
            "confidence": 0.0-1.0,
            "checks": {{
                "price_appropriate": true/false,
                "persona_aligned": true/false,
                "requirement_met": true/false,
                "promotion_considered": true/false
            }}
        }}
        """
        
        response = self.llm.complete(prompt)
        return Justification(
            text=response.justification,
            confidence=response.confidence,
            checks=response.checks
        )


# Main execution flow
def run_mystery_shop(mission, persona, date_run):
    """
    Execute mystery shop for all vendors.
    
    Args:
        mission: Mission/recipe to shop
        persona: CREST persona (Saver, Traditional, etc.)
        date_run: Date of execution
    
    Returns:
        Combined results from all vendors
    """
    # Initialize vendor agents
    woolworths_agent = VendorAgent(
        vendor_name="Woolworths",
        product_catalog=load_woolworths_catalog(),
        persona=persona
    )
    
    coles_agent = VendorAgent(
        vendor_name="Coles",
        product_catalog=load_coles_catalog(),
        persona=persona
    )
    
    aldi_agent = VendorAgent(
        vendor_name="Aldi",
        product_catalog=load_aldi_catalog(),
        persona=persona
    )
    
    # Process mission at all vendors (can be parallel)
    results = []
    
    for agent in [woolworths_agent, coles_agent, aldi_agent]:
        basket = agent.process_mission(mission)
        
        # Add metadata
        for item in basket:
            item.update({
                'run_id': generate_run_id(),
                'date_run': date_run,
                'mission_name': mission.name,
                'persona': persona.name,
                'persona_description': persona.description
            })
        
        results.extend(basket)
    
    # Output to BigQuery
    save_to_bigquery(results, table='shop_results_detailed')
    
    return results
```

---

## Example Output Record

### Single Ingredient Result in BigQuery

```json
{
  "run_id": "run_20260510_001",
  "date_run": "2026-05-10",
  "week_label": "Week 1 May 2026",
  "execution_timestamp": "2026-05-10T10:15:32Z",
  
  "mission_id": "mission_001",
  "mission_name": "Spaghetti Bolognese",
  "mission_description": "Traditional family meal for 4",
  "persona": "SAVER",
  "persona_description": "Budget-conscious young families",
  
  "vendor": "Woolworths",
  "vendor_location": "Mascot CFC",
  "vendor_category": "Supermarket",
  
  "ingredient_order": 1,
  "ingredient_description": "Lean mince beef, 1kg",
  "ingredient_quantity": "1kg",
  "ingredient_category": "Meat",
  
  "product_id": "WW_123456",
  "product_name": "Woolworths Lean Beef Mince",
  "brand": "Woolworths",
  "pack_size": "1kg",
  "pack_size_normalized": 1000,
  "price": 12.00,
  "unit_price": "$12.00/kg",
  "promotion_type": "yellow_ticket",
  "promotion_description": "Special - Save $3",
  "product_url": "https://www.woolworths.com.au/...",
  
  "justification": "Selected Woolworths own brand mince because yellow ticket special provides best unit price at $12/kg (down from $15), meets exact 1kg requirement without waste, and own brand aligns with Saver budget-first priority. Alternative premium brands at $18/kg rejected as inconsistent with price-conscious persona.",
  "confidence_score": 0.94,
  "alternative_product_id": "WW_789012",
  "selection_timestamp": "2026-05-10T10:15:28Z",
  
  "organic_flag": false,
  "free_range_flag": false,
  "australian_made_flag": true,
  "own_brand_flag": true,
  "status": "found"
}
```

---

## Key Architecture Benefits

### Why This Design Works

| Feature | Benefit |
|---------|---------|
| **Per-Vendor Agents** | Clean separation, parallel processing possible |
| **Full Product Catalog Input** | LLM has complete context to make best choice |
| **Ingredient-Level Evaluation** | Rich justification for every decision |
| **Persona Prompt Injection** | Consistent decision-making aligned with CREST segments |
| **Rich BigQuery Output** | Complete traceability and analysis capability |

### Comparison to Previous Design

| Aspect | Previous Design | Current Design |
|--------|----------------|----------------|
| **Agent Scope** | Per-ingredient voting | Per-vendor, all ingredients |
| **Evaluation** | Basket-level only | Per-ingredient justification |
| **Product Context** | RAG top-15 candidates | Full vendor catalog |
| **Output Granularity** | Basket summaries | Ingredient-level detail |
| **Traceability** | Moderate | **High** - full justifications |

---

## Execution Schedule

### Weekly Batch Processing

```mermaid
gantt
    title Weekly Mystery Shop Execution
    dateFormat HH:mm
    section Friday 10 AM
    Load Catalogs         :done, 10:00, 10m
    Mission 1 (8x3=24 shops)  :active, 10:10, 20m
    Mission 2 (8x3=24 shops)  :10:30, 20m
    Mission 3 (8x3=24 shops)  :10:50, 20m
    Mission 4 (8x3=24 shops)  :11:10, 20m
    Mission 5 (8x3=24 shops)  :11:30, 20m
    Mission 6 (8x3=24 shops)  :11:50, 20m
    Mission 7 (8x3=24 shops)  :12:10, 20m
    Mission 8 (8x3=24 shops)  :12:30, 20m
    Save to BigQuery      :12:50, 10m
    section Complete
    120 Shops Done        :milestone, 13:00, 0m
```

**Total Time:** ~3 hours for 120 shops (8 missions × 5 personas × 3 vendors)

---

## Next Steps

1. ✅ **Implement VendorAgent class** with LLM integration
2. ✅ **Implement EvaluationAgent class** for justifications
3. ✅ **Load full product catalogs** into memory/database
4. ✅ **Define BigQuery schema** with rich fields
5. ✅ **Test with single mission** (e.g., Spaghetti Bolognese, Saver, Woolworths)
6. ✅ **Scale to all 120 combinations**
7. ✅ **Build front-end dashboard** to visualize justifications

---

**Last Updated:** May 5, 2026 (Post-Catchup)  
**Status:** Current Recommended Architecture  
**Supersedes:** AGENT_ARCHITECTURE_DESIGN.md (sequential basket-level approach)
