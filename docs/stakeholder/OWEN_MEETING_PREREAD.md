# Mystery Shopping Agent - Pre-Read for Owen
## Friday Meeting (April 25, 2026)

**From:** Alexa Kelly  
**Subject:** Mystery Shopping Agent - 4 Week Plan Review  
**Meeting:** Friday, April 25, 4:00 PM

---

## What We're Building

**AI agent that autonomously shops like real customers**

- 8 missions (Spaghetti Bolognese, School Lunches, etc.) × 5 personas (CREST segments) × 3 retailers
- **= 120 shops per day for competitive intelligence**
- Validated against Feb 20 manual baseline
- Go-live: May 19, 2026 (4 weeks from now)

---

## Why This Matters

**For the CEO:**
- Daily basket price comparisons: Woolworths vs Coles vs Aldi
- Understand how different customer segments perceive our value
- Automated, scalable, consistent methodology
- Historical trending and competitive analysis

**Current State:** Manual mystery shopping (Feb 20 baseline) - one-time, labor-intensive

**Future State:** Automated daily mystery shops with persona-based insights

---

## Key Update: Data Sources Confirmed (Apr 22)

**Better than expected!**

✅ **Woolworths** (via Stuart):
- Product attributes: allergens, country of origin, Australian ingredient %
- Promise information
- **3 separate promotion flags** (half price, low price special, price drop special)

✅ **Coles** (via Nissan competitor table):
- Ingredients, allergens, nutrition information
- Promotional details
- ⚠️ Issue: Multi-postcode pricing (different prices across postcodes)

🔄 **Aldi**:
- Base table available
- Need to enrich ~100 products (not full catalog) for 8 missions
- Pre-scrape approach to avoid runtime failures

---

## Technical Approach

**1. Semantic Product Search**
- Vector embeddings for intelligent matching
- Handles variations in pack sizes, brands, quality grades
- Target: >95% recall vs Feb 20 baseline

**2. LLM Persona Decision Engine**
- Gemini 1.5 Pro makes decisions based on CREST persona rules
- Structured reasoning (auditable decisions)
- Confidence scoring on every selection

**3. Validation & Quality Assurance**
- Multi-source price validation
- Ground truth comparison (Feb 20 baseline)
- Human validation in Week 4

---

## 4-Week Timeline

**Week 1 (Apr 22-28):** Foundation & data quality assessment
- Build product catalog (~50K products)
- Semantic search operational
- Aldi enrichment (~100 products)
- ✅ Checkpoint: Friday Apr 25 (today's meeting)

**Week 2 (Apr 29-May 5):** LLM decision engine & personas
- Saver and Traditional personas implemented
- 2 missions completed (12 shops)
- ✅ Checkpoint: Friday May 2

**Week 3 (May 6-12):** Scale to all missions & personas
- All 5 personas operational
- All 8 missions configured
- 120 successful shop executions
- ✅ Checkpoint: Friday May 9

**Week 4 (May 13-19):** Validation & production readiness
- Human validation (agent vs manual shops)
- Analytics dashboards
- Documentation
- **🎯 Go-Live: May 19**

---

## Decisions Needed from You (Friday Meeting)

### 1. Multi-Postcode Pricing (Coles)

**Problem:** Some Coles products have different prices across postcodes

**Options:**
- Average price
- Median price
- **Specific postcode (Mascot/Inner Sydney)** ← We recommend this
- Flag separately

**Question:** Which approach should we use?

**Our recommendation:** Mascot/Inner Sydney postcode
- Real price from real store (matches customer experience)
- Comparable to WW Mascot service center
- Simplest implementation

---

### 2. Aldi Enrichment Approval

**Proposal:** Pre-scrape descriptions for ~100 Aldi products across 8 missions

**Approach:**
- NOT on-the-fly scraping (avoid runtime failures)
- Pre-scrape before Week 2, cache results
- Coordinate with pricing team on methodology
- Focus: Meat, Dairy, Eggs categories

**Question:** Do you approve this selective enrichment approach?

**Our recommendation:** Yes, proceed
- Small scope (only ~100 products, not full catalog)
- Critical for Conscious/Refined persona decision quality
- Mitigates risk (pre-cached, no runtime dependency)

---

### 3. Overall Plan Approval

**Question:** Are you comfortable with the 4-week timeline and approach?

**Our recommendation:** Proceed as planned
- Data sources confirmed (better than expected)
- Clear weekly checkpoints with go/no-go criteria
- Validated against Feb 20 baseline
- Scope is focused and achievable

---

## Success Criteria

| Metric | Target |
|--------|--------|
| Search precision | >95% top-3 recall vs Feb 20 |
| Price accuracy | >98% within $0.50 |
| Persona decision quality | >90% alignment with expected behavior |
| Coverage | >95% of mission items found |
| Execution time | <5 minutes per mission |

---

## Costs

**MVP Development (4 weeks):** ~$70
- Mainly internal resource (Daria)
- Minimal external spend (LLM API, BigQuery)

**Ongoing Automation (daily):** ~$125/month (~$1,500/year)
- Gemini 1.5 Pro API: ~$100/month
- BigQuery: ~$20/month
- Cloud Functions: ~$5/month

**Very cost-effective for daily competitive intelligence**

---

## Risks & Mitigation

| Risk | Mitigation |
|------|------------|
| 4-week timeline | Weekly checkpoints with go/no-go criteria |
| Multi-postcode pricing | Your decision at Friday meeting |
| Aldi enrichment failures | Pre-scrape + pricing team coordination |
| LLM inconsistency | Low temperature (0.3), structured prompts |
| Search precision | Ground truth validation, iterative refinement |

---

## What Happens After Friday Meeting

**If approved:**
1. Daria continues Week 1 execution (catalog build, semantic search)
2. Coordinate Aldi enrichment with pricing team
3. Weekly checkpoints every Friday 4pm
4. Email updates on progress and any blockers

**Weekly checkpoints:**
- Week 1 (Apr 25 - TODAY): Data quality, search foundation
- Week 2 (May 2): Persona accuracy vs baseline
- Week 3 (May 9): Full scope execution (120 shops)
- Week 4 (May 16): Human validation, analytics

---

## Questions to Think About Before Meeting

1. **Multi-postcode pricing:** Comfortable with Mascot/Inner Sydney approach?
2. **Aldi enrichment:** Approve ~100 product pre-scraping?
3. **Weekly checkpoints:** Available Fridays 4pm for updates?
4. **Stakeholder demo:** Who else should see Week 4 demo (May 19)?
5. **Phase 2 automation:** Interest in daily automation post-MVP?

---

## Meeting Agenda (45-60 mins)

1. Executive summary (5 mins)
2. Problem & opportunity (10 mins)
3. Data sources confirmed (10 mins)
4. Solution architecture (10 mins)
5. **4-week detailed plan** (15 mins)
6. **Decisions needed from you** (15 mins) ← Key section
7. Success criteria & risks (5 mins)
8. Next steps & Q&A (remaining time)

---

## Contact

**Questions before Friday?**
- Alexa Kelly (project lead)
- Daria (technical implementation)

**See you Friday at 4pm!**

We're excited to walk you through the plan and get your approval to proceed. The data sources are better than we expected, and we're on track for a successful 4-week delivery.

---

**Attachments for Meeting:**
- Full slide deck (will be shared)
- Sample data quality examples
- CREST persona summaries
- Detailed task breakdowns
