# Mystery Shopping Agent

**Board Presentation:** May 18, 2026  
**Project Lead:** Alexa Kelly  
**Technical Lead:** Daria Volkova  
**Status:** Week 2 - Multi-Agent System Development

---

## Quick Start

**📋 Project Overview:** See [PROJECT_BRIEF.md](PROJECT_BRIEF.md) for complete project scope, architecture, and timeline.

**🎯 Current Deliverable:** AI-powered automated mystery shopping system executing 120 shops (8 missions × 5 personas × 3 retailers) with interactive front-end for board demonstration.

---

## Repository Structure

```
mystery-shopper/
├── PROJECT_BRIEF.md          # Master project plan and specifications
├── README.md                 # This file
│
├── config/                   # Configuration and persona definitions
│   ├── personas/            # Detailed CREST persona specifications
│   │   ├── SAVER_PERSONA.md
│   │   ├── TRADITIONAL_PERSONA.md
│   │   ├── CONSCIOUS_PERSONA.md
│   │   ├── REFINED_PERSONA.md
│   │   └── ESSENTIAL_PERSONA.md
│   └── PERSONA_REVIEW.md    # Persona validation and quality review
│
├── docs/                     # Documentation
│   ├── planning/            # Project planning documents
│   │   ├── 4_WEEK_PHASE_PLAN.md
│   │   ├── FINAL_IMPLEMENTATION_PLAN.md
│   │   ├── TIMELINE_COMPRESSION_ANALYSIS.md
│   │   └── WEEK1_ACTIONS.md
│   │
│   ├── technical/           # Technical specifications
│   │   ├── DATA_SOURCE_DECISION.md
│   │   ├── PERSONA_DECISION_QUALITY.md
│   │   ├── PERSONA_PRODUCT_ATTRIBUTES.md
│   │   ├── FRONTEND_SPECIFICATIONS.md
│   │   └── REACT_FRONTEND_DESIGN_GUIDE.md
│   │
│   └── stakeholder/         # Stakeholder presentations and updates
│       ├── OWEN_ONE_PAGE_SUMMARY.md
│       ├── OWEN_STAKEHOLDER_PRESENTATION.md
│       ├── OWEN_MEETING_PREREAD.md
│       ├── OWEN_REVIEW_PREP.md
│       ├── MEETING_IMPACT_SUMMARY.md
│       └── PRESENTATION_UPDATES_SUMMARY.md
│
├── meetings/                 # Meeting notes and transcripts
│   ├── ak dv catch up may4.txt
│   ├── dv ak catch up apr29 apr30.txt
│   ├── alexa daria apr22.txt
│   └── ... (other meeting notes)
│
└── reference/               # Reference materials and PDFs
    ├── Mystery Shopping agent - overview.pdf
    ├── [26-01] Mystery shop baskets - for discussion (1).pdf
    └── CREST Profiles - Group vf.pdf
```

---

## Key Documents

### For Project Overview
- **[PROJECT_BRIEF.md](PROJECT_BRIEF.md)** - Complete project plan, architecture, timeline, and success metrics

### For Implementation
- **[4_WEEK_PHASE_PLAN.md](docs/planning/4_WEEK_PHASE_PLAN.md)** - Weekly deliverables and checkpoints
- **[FINAL_IMPLEMENTATION_PLAN.md](docs/planning/FINAL_IMPLEMENTATION_PLAN.md)** - Detailed technical implementation approach
- **[FRONTEND_SPECIFICATIONS.md](docs/technical/FRONTEND_SPECIFICATIONS.md)** - Front-end dashboard specifications

### For Board Presentation
- **[OWEN_STAKEHOLDER_PRESENTATION.md](docs/stakeholder/OWEN_STAKEHOLDER_PRESENTATION.md)** - Board presentation strategy and demo flow
- **[OWEN_ONE_PAGE_SUMMARY.md](docs/stakeholder/OWEN_ONE_PAGE_SUMMARY.md)** - Executive summary for Owen

### For Persona Configuration
- **[config/personas/](config/personas/)** - Individual persona specifications (Saver, Traditional, Conscious, Refined, Essential)

---

## System Architecture

**Multi-Agent Voting System:**
- 3 independent Claude-based agents vote per ingredient
- Evaluation agent validates basket-level quality
- Vertex AI Search for semantic product retrieval (UPDATED May 6)
- Top 3 candidates per ingredient from 4 weeks historical data

**Data Pipeline:**
Vertex AI Search (top 3) → Unit normalization (planned) → Multi-agent voting → Evaluation → BigQuery storage (enriched schema)

**Recent Updates (May 6, 2026):**
- Switched to Vertex AI Search for all retailers (improved accuracy)
- Top 3 products per ingredient (was 5)
- Output schema enriched with justification text and decisioning attributes
- Unit normalization pre-processing in development

**Front-End:**
Interactive Looker Studio or Streamlit dashboard for board demonstration

---

## Timeline

| Phase | Week | Dates | Status | Focus |
|-------|------|-------|--------|-------|
| **1** | Week 1 | Apr 22-28 | ✅ Complete | Data foundation, RAG retrieval |
| **2** | Week 2 | Apr 29-May 5 | 🔄 In Progress | Multi-agent system, personas |
| **3** | Week 3 | May 6-12 | ⏳ Upcoming | Scale to 120 shops, front-end |
| **4** | Week 4 | May 13-18 | ⏳ Upcoming | Board demo preparation |

**Board Presentation:** May 18, 2026

---

## Current Status (Week 2)

✅ **Completed:**
- Unified product catalog (Woolworths, Coles, Aldi)
- Semantic search with hybrid approach
- Saver and Traditional personas deployed
- Data pipeline operational

🔄 **In Progress:**
- Multi-agent voting implementation
- Evaluation agent with rejection limits and retry context
- Conscious, Refined, Essential personas (prompts complete, need review)
- Front-end dashboard development
- Vertex AI Search index setup (3 retailers)
- Unit normalization pre-processing tool

⏳ **Next:**
- Scale to all 120 shops
- Historical data integration
- Board presentation materials

---

## Key Contacts

**Project Team:**
- **Alexa Kelly** - Project Lead
- **Daria Volkova** - Technical Lead (Backend/Data)
- **Matt** - Frontend Development Support

**Stakeholders:**
- **Owen Lim** - Project Sponsor, CEO presentation
- **Stuart** - Woolworths data source contact
- **Nissan** - Competitor data (PCB team)

---

## Quick Reference

**Data Sources:**
- Woolworths: Internal product table + historical data (4 weeks) (`gcp-wow-ent-im-tbl-prod.adp_dm_betterbuying_view.Sell_Price_Weekly_v`)
- Coles: PCB competitor table (~20K products, 4 weeks historical)
- Aldi: PCB table + selective enrichment (~100 products for 8 missions)

**Search Infrastructure:**
- Vertex AI Search indexes (3 total: WW, Coles, Aldi)
- 4 weeks historical data per retailer
- Returns top 3 candidates per ingredient query

**Scope:**
- 8 shopping missions (Spaghetti Bolognese, School Lunches, Weekly Shop, BBQ, etc.)
- 5 CREST personas (Saver, Traditional, Conscious, Refined, Essential)
- 3 retailers (Woolworths, Coles, Aldi)
- **Total: 120 automated mystery shops**

**Success Metrics:**
- Coverage: >95% of items found
- Price accuracy: >98% within $0.50
- Agent confidence: >0.80 average
- Execution time: <5 minutes per mission

---

*Last updated: May 4, 2026*
