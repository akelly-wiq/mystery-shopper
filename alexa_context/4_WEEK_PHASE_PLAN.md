# Mystery Shopping Agent - 4 Week Phase Plan
## High-Level Execution Plan for Stakeholders

**Timeline:** April 22 - May 19, 2026  
**Last Updated:** May 6, 2026  
**Resource:** Daria Volkova (Data Scientist) + Alexa Kelly (Project Lead)  
**Architecture:** Multi-Agent System with Vertex AI Search and Basket-Level Validation

---

## Phase 1: Foundation & Data Intelligence
**Week 1 (Apr 22-28)**

### Objectives
Build the data foundation and product retrieval system

### Key Deliverables
✅ **Unified product catalog** (~50K products across WW, Coles, Aldi)  
✅ **Vertex AI Search** - semantic search for all retailers (UPDATED May 6)  
✅ **Data enrichment** - Aldi products (~100) with quality attributes  
✅ **Data validation** - confirm accuracy and completeness  
🔄 **Unit normalization** - pre-processing for price comparisons (PLANNED)  

### Success Criteria
- Product catalog complete with pricing, promotions, attributes
- RAG system retrieves relevant products for mission ingredients
- Data quality assessment shows >90% completeness

### Checkpoint: Friday Apr 25 (Today)
**Go/No-Go:** Data confirmed, retrieval working, Aldi enrichment plan approved

---

## Phase 2: Multi-Agent Decision System
**Week 2 (Apr 29-May 5)**

### Objectives
Build the AI decision-making system using multi-agent architecture

### Key Deliverables
✅ **ADK agent framework** - Agent Development Kit implementation  
✅ **Persona-based agents** - CREST segment decision logic (Saver, Traditional)  
✅ **Multi-agent voting** - 3 agents vote per ingredient for consistency  
✅ **Basket-level judge** - validates coverage, balance, total price  

### Success Criteria
- Agents make persona-differentiated decisions
- Multi-agent voting achieves consistency (majority agreement)
- Basket validation catches errors and inconsistencies
- Test missions complete successfully (2 personas, sample missions)

### Checkpoint: Friday May 2
**Go/No-Go:** Agent decisions align with persona rules, voting system works

---

## Phase 3: Scale to Full Scope
**Week 3 (May 6-12)**

### Objectives
Expand to complete system: all personas, all missions

### Key Deliverables
✅ **All 5 CREST personas** implemented (Conscious, Refined, Essential added)  
✅ **All 8 shopping missions** configured and tested  
✅ **Full matrix execution** - 120 shops (8 × 5 × 3)  
✅ **Price validation** - Woolworths API cross-check  
✅ **Output schema enrichment** - justification text + decisioning attributes (NEW)  
✅ **4 weeks historical data** - confirmed scope  

### Success Criteria
- All 120 automated shops complete successfully
- Coverage >95% (ingredients found across retailers)
- Price accuracy >98% (validated against Woolworths API)
- Execution time <5 minutes per mission

### Checkpoint: Friday May 9
**Go/No-Go:** Full scope working, coverage targets met, no major failures

---

## Phase 4: Production Readiness
**Week 4 (May 13-19)**

### Objectives
Validate quality, build reporting, prepare for handoff

### Key Deliverables
✅ **Human validation** - compare agent vs expert shoppers  
✅ **Analytics dashboards** - basket comparisons, insights  
✅ **Reporting interface** - Google Sheets integration  
✅ **Documentation** - system guide, runbook, known limitations  

### Success Criteria
- Human validation shows >85% agreement with agent decisions
- Stakeholder demo successful
- Analytics deliver actionable insights
- System ready for production use

### Go-Live: Monday May 19
**Owen approval and system handoff**

---

## Architecture Overview (High-Level)

### Component 1: Vertex AI Search System (UPDATED May 6)
**Purpose:** Find relevant products for each ingredient using semantic search

**Process:**
- Vertex AI Search indexes for WW, Coles, Aldi (4 weeks historical data)
- Semantic search returns top 3 candidates per ingredient
- Products include: price, promotions, pack size, quality attributes
- Pre-processing: Unit normalization for fair price comparisons (PLANNED)

**Why Changed:**
- Previous approach: Local embeddings returned irrelevant results
- Issue: "chicken breast bulk pack" → "nuggets"
- Solution: Vertex AI Search for improved accuracy

---

### Component 2: Multi-Agent Voting System
**Purpose:** Select best product per ingredient with consistency

**Process:**
- 3 independent agents evaluate candidates
- Each agent applies persona-specific decision logic
- Vote on best product (highest vote count wins)
- Ensures consistency and reduces errors

**Example:**
- Ingredient: "Lean mince beef, 1kg"
- Agent 1: Selects Woolworths own brand (Red EDLP)
- Agent 2: Selects Woolworths own brand (Red EDLP)
- Agent 3: Selects Coles Yellow ticket option
- **Winner:** Woolworths own brand (2/3 votes)

---

### Component 3: Basket-Level Judge
**Purpose:** Validate complete basket for quality and accuracy

**Validation Checks:**
1. **Coverage:** All ingredients found?
2. **Balance:** Reasonable basket composition?
3. **Price accuracy:** Cross-check with Woolworths API
4. **Persona alignment:** Selections match customer profile?

**Action if failed:** Re-run agents for problematic items

---

### Component 4: Reporting & Analytics
**Purpose:** Deliver actionable competitive insights

**Outputs:**
- Basket comparison tables (WW vs Coles vs Aldi)
- Price gap analysis by mission and persona
- Promotion effectiveness insights
- Persona shopping behavior patterns

---

## Scope Summary

### Personas (CREST Segments)
1. **Saver** - Budget-focused, promotion-seeking
2. **Traditional** - Brand-loyal, consistency-focused
3. **Conscious** - Health/sustainability-focused
4. **Refined** - Quality over price
5. **Essential** - Price-conscious, simplicity

### Missions
1. Spaghetti Bolognese
2. School Lunches (1 week)
3. Weekly Shop
4. BBQ
5. Easy Midweek Dinner
6. Party Food
7. Quick Breakfast
8. Healthy Snacks

### Retailers
- Woolworths
- Coles
- Aldi

**Total Scope:** 8 missions × 5 personas × 3 retailers = **120 automated mystery shops**

---

## Weekly Checkpoints

| Week | Date | Focus | Go Criteria | No-Go Trigger |
|------|------|-------|-------------|---------------|
| **1** | Apr 25 | Data & RAG | Data quality >90%, retrieval working | Data access blocked, quality <80% |
| **2** | May 2 | Multi-Agent System | Agents work, voting consistent | Agent decisions random/illogical |
| **3** | May 9 | Full Scope | 120 shops complete, coverage >95% | Major failures, coverage <85% |
| **4** | May 16 | Validation | Human agreement >85%, demo ready | Accuracy <75%, major bugs |

---

## Risk Management

### Key Risks

| Risk | Impact | Mitigation |
|------|--------|------------|
| **Multi-postcode pricing (Coles)** | Price comparison accuracy | Owen decision (use Mascot postcode) |
| **Aldi data sparse** | Poor Conscious/Refined decisions | Pre-enrich ~100 products |
| **Agent voting inconsistency** | Unreliable selections | 3-agent voting, basket judge validation |
| **4-week timeline** | Quality vs speed trade-off | Weekly checkpoints, clear go/no-go criteria |

### Mitigation Strategy
- **Weekly checkpoints:** Pause and fix if metrics below threshold
- **Multi-agent voting:** Reduces individual agent errors
- **Basket judge:** Catches systematic issues
- **Human validation:** Final quality gate (Week 4)

---

## Success Definition

### Must-Have (MVP)
✅ All 120 shops executable on demand  
✅ Basket comparisons across 3 retailers  
✅ Persona-differentiated decisions  
✅ Price accuracy >95%  
✅ Stakeholder approval for production  

### Nice-to-Have (Phase 2)
🔄 Daily automated execution  
🔄 Historical trending  
🔄 Alert system for competitive changes  
🔄 Scenario modeling  

---

## Phase 2 Preview (Post-MVP)

**After May 19 go-live, next phase would include:**

### Automation
- Scheduled daily/weekly execution
- Cloud Functions for orchestration
- Monitoring and alerting

### Analytics Enhancement
- Historical price trending
- Promotion effectiveness tracking
- Competitive positioning insights
- Scenario modeling ("what-if" analysis)

### Expansion
- Additional missions
- Regional variations (different postcodes)
- More retailers (if needed)

**Estimated effort:** 2-3 weeks  
**Estimated cost:** ~$125/month ongoing operations

---

## Communication Plan

### Weekly Updates (Every Friday 4pm)
**To:** Owen Lim  
**Format:** Email + metrics dashboard  
**Contents:**
- Progress vs plan
- Key metrics (coverage, accuracy, completion)
- Blockers or decisions needed
- Go/no-go recommendation for next week

### Escalation
- **Minor issues:** Daria → Alexa (resolve within team)
- **Major blockers:** Immediate escalation to Owen
- **Data access issues:** Coordinate with Stuart/Nissan, inform Owen

### Stakeholder Demo
- **When:** Week 4, May 19 (go-live)
- **Format:** Live demonstration + Q&A
- **Attendees:** Owen + relevant stakeholders
- **Content:** System walkthrough, results showcase, handoff

---

## Key Dependencies

### Data Access
✅ **Woolworths tables** (via Stuart) - confirmed  
✅ **Coles competitor data** (via Nissan) - confirmed  
🔄 **Aldi enrichment** - coordinate with pricing team  

### Decisions
⚠️ **Multi-postcode pricing** - Owen decision needed (Apr 25)  
⚠️ **Aldi enrichment scope** - Owen approval needed (Apr 25)  

### Technical
✅ **ADK (Agent Development Kit)** - framework available  
✅ **BigQuery access** - tables identified  
✅ **Vertex AI / Gemini API** - approved for use  

---

## Deliverables Summary

| Week | Deliverable | Stakeholder Value |
|------|-------------|-------------------|
| **1** | Product catalog + RAG retrieval | Foundation for accurate product matching |
| **2** | Multi-agent decision system | Consistent, persona-based selections |
| **3** | 120 automated shops | Full competitive intelligence capability |
| **4** | Validated system + dashboards | Production-ready competitive insights |

---

**Status:** Week 1 in progress, on track for May 19 go-live  
**Next Checkpoint:** Friday April 25 (today) - Data & RAG validation
