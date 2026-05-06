# Mystery Shopper Project - Context for Claude Agent
## Essential Documentation Package

**Created:** May 6, 2026  
**Purpose:** Complete context for working on Mystery Shopper project in separate VS Code instance  
**Status:** Week 2 - Multi-Agent Decision System Development

---

## What This Folder Contains

This folder contains all the essential documentation needed to work on the Mystery Shopper project with full context. Documents are organized by priority.

---

## 📌 START HERE - Core Documents (Read First)

### 1. **PROJECT_BRIEF.md** ⭐ CRITICAL
**What:** Complete project overview, objectives, architecture, and timeline  
**Why read:** Understand the business problem, solution approach, and 4-week delivery timeline  
**Key sections:**
- Executive summary (what we're building and why)
- Architecture overview
- 4-week timeline
- Success metrics

### 2. **PROJECT_README.md** 
**What:** Repository structure and quick reference guide  
**Why read:** Navigate the codebase and understand current status  
**Key sections:**
- Current deliverable status
- Directory structure
- Key contacts
- Data sources

### 3. **AGENT_ARCHITECTURE.md** ⭐ CRITICAL (Updated May 6)
**What:** Technical architecture with Mermaid diagrams  
**Why read:** Understand how the multi-agent system works  
**Key sections:**
- Per-vendor agent flow
- Vertex AI Search architecture (NEW - May 6 change)
- LLM decision engine details
- Evaluation agent
- BigQuery output schema
- Unit normalization (planned)

**Recent changes:** Switched from local embeddings to Vertex AI Search, top 5→3 products, enriched output schema

---

## 📋 Planning & Timeline Documents

### 4. **4_WEEK_PHASE_PLAN.md** ⭐ CRITICAL
**What:** High-level weekly execution plan for stakeholders  
**Why read:** Understand what needs to be delivered each week  
**Key sections:**
- Week 1: Foundation & Data (✅ Complete)
- Week 2: Multi-Agent System (🔄 Current)
- Week 3: Scale to 120 shops
- Week 4: Board preparation
- Weekly checkpoint criteria

### 5. **FINAL_IMPLEMENTATION_PLAN.md**
**What:** Detailed technical implementation guide  
**Why read:** Day-by-day breakdown of what to build  
**Key sections:**
- Data source decisions
- Vertex AI Search setup (updated May 6)
- Week-by-week deliverables
- Code examples
- Success criteria

### 6. **MAY6_ACTION_ITEMS_DETAILED.md** ⭐ CRITICAL
**What:** Detailed breakdown of all tasks from May 6 meeting with Daria  
**Why read:** Know exactly what Alexa and Daria need to deliver this week  
**Key sections:**
- Alexa's 4 tasks (prompts review, testing, evaluation agent, normalization)
- Daria's 7 tasks (schema, Vertex AI, pipeline)
- Dependencies and critical path
- Timeline (immediate/this week/end of week)

**YOUR CURRENT TASKS (Alexa):**
1. Review persona prompts (HIGH - May 7)
2. Test web application (MEDIUM - May 9)
3. Evaluation agent prompt (HIGH - May 8)
4. Unit normalization tool (HIGH - May 9)

---

## 🎭 Persona & Prompts Documents

### 7. **PERSONA_PROMPTS_REVIEW.md** ⭐ CRITICAL (Task 1)
**What:** Review of Daria's personas.py implementation  
**Why read:** Know what's good, what needs fixing, and testing approach  
**Key sections:**
- Issues found (4 typos, naming inconsistency)
- Persona differentiation analysis
- Questions for Daria
- Testing checklist

**Action required:** Fix typos and validate differentiation

### 8. **persona_prompts/personas.py** ⭐ CRITICAL
**What:** The actual prompt templates Daria created for all 5 personas  
**Why read:** See the production prompts the LLM will use  
**Contains:**
- SAVER, TRADITIONAL, REFINED, ESSENTIAL, CONSCIOUS prompts
- Decision frameworks
- Tie-breaker rules
- Sample decisions
- Python helper functions

### 9. **persona_prompts/*.md files**
**What:** Original markdown prompt files (SAVER_PROMPT.md, etc.)  
**Why read:** See the source material Daria converted to Python  
**Use for:** Comparing against personas.py to validate conversion

### 10. **PERSONA_DECISION_QUALITY.md**
**What:** How to ensure persona decisions are high quality  
**Why read:** Framework for validating agent decisions  
**Key sections:**
- Quality dimensions
- Differentiation testing
- Validation approach

---

## 📝 Meeting Notes

### 11. **daria_project_discussion_may6.txt** ⭐ IMPORTANT
**What:** Detailed meeting notes from May 6 session with Daria  
**Why read:** Context on recent decisions and direction changes  
**Key decisions:**
- Switch to Vertex AI Search for all retailers
- Top 3 products (not 5)
- 4 weeks historical data
- Output schema enrichment
- Unit normalization needed
- Testing on 'parallel agents' branch

**Action items extracted in:** MAY6_ACTION_ITEMS_DETAILED.md

---

## 🗂️ Document Organization

```
alexa_context/
├── README_CONTEXT.md (this file)
│
├── 📌 CORE (read these first)
│   ├── PROJECT_BRIEF.md
│   ├── PROJECT_README.md
│   └── AGENT_ARCHITECTURE.md
│
├── 📋 PLANNING
│   ├── 4_WEEK_PHASE_PLAN.md
│   ├── FINAL_IMPLEMENTATION_PLAN.md
│   └── MAY6_ACTION_ITEMS_DETAILED.md
│
├── 🎭 PERSONAS
│   ├── PERSONA_PROMPTS_REVIEW.md
│   ├── PERSONA_DECISION_QUALITY.md
│   └── persona_prompts/
│       ├── personas.py
│       ├── SAVER_PROMPT.md
│       ├── TRADITIONAL_PROMPT.md
│       ├── REFINED_PROMPT.md
│       ├── ESSENTIAL_PROMPT.md
│       └── CONSCIOUS_PROMPT.md
│
└── 📝 MEETINGS
    └── daria_project_discussion_may6.txt
```

---

## 🎯 Current Project Status (May 6, 2026)

**Week:** 2 of 4  
**Focus:** Multi-Agent Decision System  
**Board Presentation:** May 18, 2026

**✅ Completed:**
- Week 1: Data foundation, product catalog (Woolworths, Coles, Aldi)
- Persona prompts created by Daria (needs review)
- Architecture decision: Vertex AI Search for all retailers

**🔄 In Progress (This Week):**
- Alexa: Review persona prompts, design evaluation agent, build unit normalization
- Daria: Set up Vertex AI Search, refactor pipeline, enrich output schema

**⏳ Coming Next:**
- Week 3: Execute 120 shops (8 missions × 5 personas × 3 retailers)
- Week 4: Validation and board preparation

---

## 🚨 Critical Context You Need to Know

### Architecture Change (May 6)
**OLD:** Local embeddings for Coles/Aldi, top 5 products  
**NEW:** Vertex AI Search for ALL retailers (WW, Coles, Aldi), top 3 products  
**Reason:** Better accuracy - was returning "nuggets" for "chicken breast bulk pack"

### Current Branch
**DO NOT USE:** `main` branch (deprecated)  
**USE:** `parallel-agents` branch  
**Testing location:** `cd app` directory

### Key Dependencies
- BigQuery: `gcp-wow-ent-im-tbl-prod.adp_dm_betterbuying_view.Sell_Price_Weekly_v`
- Historical scope: 4 weeks
- Vertex AI Search indexes: WW, Coles, Aldi (Daria creating this week)

### Accuracy Approach (Pragmatic)
**NOT:** Absolute product-level precision (98% targets)  
**YES:** Pragmatic validation:
- Consistency checks (same persona behaves predictably)
- Sanity checks (align with known signals: promos, own-brand)
- Spot reviews (sample outputs make sense)

**Focus:** Personas differentiate meaningfully, patterns are consistent

---

## 🔧 Key Technical Decisions

1. **Search:** Vertex AI Search (not local embeddings)
2. **Products per query:** 3 (not 5)
3. **Historical data:** 4 weeks
4. **Output schema:** Enriched with justification_text + decisioning_attributes
5. **Unit normalization:** Pre-processing step (Alexa building)
6. **Evaluation agent:** Retry context if validation fails (max 2 retries)

---

## 👥 Team & Contacts

**Alexa Kelly** - Project Lead (you)  
**Daria Volkova** - Data Scientist/Technical Lead  
**Owen Lim** - Project Sponsor  
**Sanjana** - Stakeholder (meeting May 7)

**Data Contacts:**
- Stuart: Woolworths data
- Nissan: PCB competitor data (Coles/Aldi)
- Mel/Mtho: Woolworths Offer ID descriptions

---

## 📊 Data Sources Quick Reference

| Retailer | Source | Products | Historical |
|----------|--------|----------|------------|
| **Woolworths** | Internal table | ~20K | 4 weeks |
| **Coles** | PCB competitor table | ~20K | 4 weeks |
| **Aldi** | PCB + enrichment | ~2K (~100 enriched) | 4 weeks |

---

## 🎯 Success Metrics

**Must-Have for Board (May 18):**
- 120 shops executable (8 missions × 5 personas × 3 retailers)
- Persona differentiation working (Saver ≠ Conscious ≠ Refined)
- Consistent decisions (same persona, same scenario = same choice)
- Output in board-ready format (Google Sheet like COMPARISON tab)

**Nice-to-Have (Phase 2):**
- Front-end dashboard
- Automated daily execution
- Historical trending

---

## 🤔 Questions to Ask When Working

If you're unclear on anything, these documents should answer:

**"What are we building?"** → PROJECT_BRIEF.md  
**"How does it work technically?"** → AGENT_ARCHITECTURE.md  
**"What do I need to do this week?"** → MAY6_ACTION_ITEMS_DETAILED.md  
**"What's the timeline?"** → 4_WEEK_PHASE_PLAN.md  
**"How are the prompts structured?"** → personas.py + PERSONA_PROMPTS_REVIEW.md  
**"What decisions were made recently?"** → daria_project_discussion_may6.txt  
**"How do I validate quality?"** → PERSONA_DECISION_QUALITY.md

---

## 📌 First Actions for New Claude Instance

1. **Read in this order:**
   - PROJECT_BRIEF.md (10 min)
   - AGENT_ARCHITECTURE.md (15 min)
   - MAY6_ACTION_ITEMS_DETAILED.md (10 min)
   - PERSONA_PROMPTS_REVIEW.md (10 min)

2. **Understand current tasks:**
   - You (Alexa) have 4 tasks due May 7-9
   - See MAY6_ACTION_ITEMS_DETAILED.md for details

3. **Review the code:**
   - Check personas.py implementation
   - Note the 4 typos identified
   - Understand what needs fixing

4. **Clarify with user:**
   - Which task to prioritize first?
   - Any blockers or new information?
   - Access to required resources (B data test location, etc.)?

---

## 🔄 Document Versions

All documents in this folder are current as of **May 6, 2026**.

**Recent updates:**
- AGENT_ARCHITECTURE.md: Updated May 6 with Vertex AI Search
- 4_WEEK_PHASE_PLAN.md: Updated May 6 with new search approach
- FINAL_IMPLEMENTATION_PLAN.md: Updated May 6 with Vertex AI details
- PERSONA_PROMPTS_REVIEW.md: Created May 6 after reviewing personas.py
- MAY6_ACTION_ITEMS_DETAILED.md: Created May 6 from meeting

---

## 💡 Tips for Working with This Context

**✅ DO:**
- Start with the "START HERE" documents
- Cross-reference between architecture and implementation plans
- Use MAY6_ACTION_ITEMS as your current task list
- Check meeting notes for recent decision context

**❌ DON'T:**
- Skip PROJECT_BRIEF.md (you'll miss critical business context)
- Assume main branch is current (use `parallel-agents`)
- Implement without checking latest architecture decisions
- Forget the May 6 changes (Vertex AI Search is NEW direction)

---

**Last Updated:** May 6, 2026  
**Context Package Version:** 1.0  
**Status:** Ready for separate VS Code instance

*For questions about this context package, refer back to the main repository or ask Alexa Kelly.*
