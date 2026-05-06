# Accuracy Verification Plan
## Multi-Stage Quality Assurance for Mystery Shopping Agent

**Created:** May 6, 2026  
**Purpose:** Ensure board-ready accuracy for May 18 deliverable  
**Approach:** Multi-layered verification with quantitative metrics

---

## Why Accuracy Matters for This Project

Mystery shopping results directly inform **pricing strategy decisions** presented to the board. Inaccurate product selections or pricing data could lead to:
- ❌ Incorrect competitive positioning analysis
- ❌ Flawed pricing recommendations
- ❌ Loss of stakeholder confidence in AI-driven insights

**Therefore:** Accuracy is not just a technical metric—it's a **business-critical requirement** for board credibility.

---

## Our Accuracy Framework

### 1. **Product Search Accuracy** (Week 1-2)

**What we measure:**
- Can the system find the RIGHT products for each ingredient?

**How we verify:**
- **Baseline comparison:** Feb 20 manual shopping data (human shoppers)
- **Top-3 recall metric:** Correct product appears in top 3 search results
- **Target:** >95% top-3 recall

**Example:**
- Ingredient: "Lean mince beef, 1kg"
- Expected: Woolworths Lean Beef Mince 1kg
- Agent returns top 3 products
- ✅ PASS if expected product is in top 3
- ❌ FAIL if expected product not in top 3

**Why this matters:** If search fails, the LLM can't select the right product downstream.

---

### 2. **Price Accuracy** (Week 2-3)

**What we measure:**
- Are the prices in our data accurate vs reality?

**How we verify:**
- **Cross-validation:** Compare PCB competitor data vs Woolworths internal pricing API
- **Acceptable variance:** Within $0.50
- **Target:** >98% of products within tolerance

**Example:**
- Product: Woolworths Own Brand Mince 1kg
- PCB table price: $12.00
- Woolworths API price: $12.20
- Difference: $0.20 ✅ PASS (within $0.50)

**Why this matters:** Board decisions on price gaps require accurate baseline data.

---

### 3. **Persona Decision Quality** (Week 2-3)

**What we measure:**
- Does the LLM make decisions that MATCH the persona's expected behavior?

**How we verify:**
- **Expert-defined expected behavior:** For each persona, we define decision rules
- **Agent decision alignment:** Compare agent selections to expected choices
- **Target:** >90% alignment for Saver, >85% for other personas

**Example (Saver Persona):**
- Ingredient: "Mince beef, 1kg"
- Options:
  - A: WW Own Brand Mince $12 (Yellow Ticket)
  - B: Premium Grass-Fed Mince $18
  - C: Mid-range Brand Mince $15
- Expected: A (Saver prioritizes price + promotions)
- Agent selects: A ✅ PASS
- Agent selects: B ❌ FAIL (not aligned with Saver behavior)

**Why this matters:** Persona differentiation is core to the value proposition—if all personas make the same choices, the system fails.

---

### 4. **Multi-Agent Consistency** (Week 2-3)

**What we measure:**
- Do multiple agents agree on the same selection?

**How we verify:**
- **3-agent voting:** Each ingredient evaluated by 3 independent agents
- **Majority vote required:** At least 2 out of 3 agents must agree
- **Target:** >80% majority agreement

**Example:**
- Ingredient: "Pasta, 500g"
- Agent 1: Selects San Remo Pasta $2.50
- Agent 2: Selects San Remo Pasta $2.50
- Agent 3: Selects Barilla Pasta $3.00
- Majority: San Remo (2/3 votes) ✅ PASS

**Why this matters:** Consistency indicates the LLM decision is robust, not random.

---

### 5. **Human Expert Validation** (Week 4)

**What we measure:**
- Do human expert shoppers agree with AI selections?

**How we verify:**
- **Sample validation:** Human experts review 20-30% of AI selections
- **Agreement scoring:** % of AI selections that human would make
- **Target:** >85% agreement

**Process:**
1. AI completes a mission (e.g., Spaghetti Bolognese for Saver persona at Woolworths)
2. Human expert reviews each product selection
3. Human rates: "Would I have selected this product for this persona?" Yes/No
4. Calculate agreement %

**Why this matters:** Human validation is the ultimate credibility check for board stakeholders.

---

### 6. **Coverage & Completeness** (Week 3)

**What we measure:**
- Can the system find products for ALL ingredients across ALL retailers?

**How we verify:**
- **Coverage metric:** % of ingredients successfully matched to products
- **Target:** >95% coverage

**Example:**
- Mission: Spaghetti Bolognese (10 ingredients)
- Woolworths: 10/10 found ✅ 100% coverage
- Coles: 9/10 found (missing specialty item) ⚠️ 90% coverage
- Aldi: 8/10 found (limited range) ⚠️ 80% coverage

**Why this matters:** Incomplete baskets reduce the value of competitive comparisons.

---

## Weekly Accuracy Checkpoints

### Week 1 Checkpoint (April 25)
**Focus:** Data quality and search accuracy
- ✅ Data sources validated (Woolworths, Coles, Aldi)
- ✅ Search returns relevant products
- ✅ Top-3 recall >95% on test set
- **GO/NO-GO:** If recall <90%, pause and fix search before Week 2

### Week 2 Checkpoint (May 2)
**Focus:** Persona decision quality
- ✅ Saver persona >90% alignment
- ✅ Traditional persona >85% alignment
- ✅ All decisions include reasoning + confidence scores
- **GO/NO-GO:** If alignment <80%, refine persona prompts before Week 3

### Week 3 Checkpoint (May 9)
**Focus:** Full-scale execution and coverage
- ✅ 120 shops executed successfully
- ✅ Coverage >95%
- ✅ Price accuracy >98%
- **GO/NO-GO:** If major failures (coverage <85%), extend timeline or reduce scope

### Week 4 Checkpoint (May 16)
**Focus:** Human validation and board readiness
- ✅ Human expert agreement >85%
- ✅ Output formatted for board presentation
- ✅ Stakeholder approval
- **GO/NO-GO:** Final quality gate before May 18 submission

---

## Built-In Quality Controls

### 1. Confidence Scoring
Every product selection includes:
- **LLM confidence score** (0.0 - 1.0): How confident is the agent in this choice?
- **Price validation confidence** (0.0 - 1.0): How confident are we in the price data?
- **Low confidence flag:** Selections with confidence <0.7 flagged for human review

### 2. Justification Transparency
Every selection includes:
- **Justification text:** "Selected WW Own Brand Mince because yellow ticket provides best unit price at $12/kg, meets exact 1kg requirement, and own brand aligns with Saver budget priority."
- **Decisioning attributes:** JSON of factors considered (price, promotion, pack size, quality indicators)
- **Enables:** Human reviewers to validate reasoning, not just outcome

### 3. Evaluation Agent (Retry Mechanism)
- **Evaluation agent** reviews each basket for quality
- **Validation checks:**
  - All ingredients found?
  - Selections align with persona?
  - Prices reasonable?
- **If validation fails:** Inject retry context and re-run selection
- **Limit:** Max 2 retries per ingredient to avoid infinite loops

### 4. Multi-Agent Voting
- **3 independent agents** evaluate each ingredient
- **Voting mechanism:** Majority wins (2/3 required)
- **Benefit:** Reduces single-agent errors, increases reliability

---

## Accuracy Metrics Dashboard

We will track and report these metrics weekly:

| Metric | Week 1 Target | Week 2 Target | Week 3 Target | Week 4 Target |
|--------|---------------|---------------|---------------|---------------|
| **Search Top-3 Recall** | >95% | >95% | >95% | >95% |
| **Price Accuracy** | >98% | >98% | >98% | >98% |
| **Persona Alignment (Saver)** | N/A | >90% | >90% | >90% |
| **Persona Alignment (Other)** | N/A | >85% | >85% | >85% |
| **Coverage** | >90% | >95% | >95% | >95% |
| **Multi-Agent Agreement** | N/A | >80% | >80% | >80% |
| **Human Expert Agreement** | N/A | N/A | N/A | >85% |
| **Avg Confidence Score** | N/A | >0.75 | >0.80 | >0.80 |

---

## Risk Mitigation

### Risk: Low Search Accuracy
**Mitigation:** 
- Vertex AI Search (implemented May 6) for semantic matching
- Continuous testing against Feb 20 baseline
- Weekly checkpoint: pause if recall <90%

### Risk: Poor Persona Differentiation
**Mitigation:**
- Detailed persona prompts with decision rules
- Weekly alignment testing
- Human expert review in Week 4

### Risk: Data Quality Issues
**Mitigation:**
- Multi-source price validation
- Confidence scoring flags low-quality data
- Manual review of flagged items

### Risk: Incomplete Coverage
**Mitigation:**
- Aldi selective enrichment (~100 products for 8 missions)
- Fallback logic for missing items
- Weekly coverage metrics

---

## Board Presentation Confidence

**By May 18, we will demonstrate:**

1. **Validated Accuracy:** All metrics above target thresholds
2. **Human Expert Approval:** >85% agreement from domain experts
3. **Transparency:** Full justification and confidence scores for every decision
4. **Reproducibility:** Clear audit trail from ingredient → search → decision → output
5. **Quality Controls:** Multi-layer verification (search → LLM → evaluation → human)

**This multi-stage QA approach ensures the board receives trustworthy, defensible competitive intelligence.**

---

## Summary: "Yes, We Can Deliver Accurate Results"

**Professionally stated:**

"We have designed a comprehensive accuracy verification framework with quantitative targets at each stage:
- Search accuracy (>95% top-3 recall)
- Price accuracy (>98% within $0.50)
- Persona alignment (>90% for Saver, >85% for others)
- Human expert validation (>85% agreement)

This multi-layered approach, combined with weekly checkpoints and built-in quality controls (confidence scoring, multi-agent voting, evaluation agent), ensures we deliver board-ready results with documented accuracy by May 18."

---

*For implementation details, see [4_WEEK_PHASE_PLAN.md](../planning/4_WEEK_PHASE_PLAN.md)*
