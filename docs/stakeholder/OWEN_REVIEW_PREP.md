# Owen Review - Presentation Prep Guide

**Date:** Friday, April 25, 2026  
**Presenter:** Alexa Kelly (with Daria support)  
**Audience:** Owen Lim (key stakeholder)  
**Duration:** 45-60 minutes recommended  
**Format:** Slide presentation + Q&A

---

## Presentation Flow

### Section 1: Executive Summary (5 mins)
**Slides 1-7**

**Key Messages:**
- We're building an AI agent that shops like real customers
- 120 shops per day (8 missions × 5 personas × 3 retailers)
- 4-week timeline, go-live May 19
- Data sources confirmed (better than expected!)

**Talking Points:**
- This gives CEO daily competitive intelligence on basket pricing
- Automated and scalable (vs manual Feb 20 baseline)
- Realistic persona decisions based on CREST profiles

---

### Section 2: Problem & Opportunity (10 mins)
**Slides 8-12**

**Key Messages:**
- The challenge: How does AI decide what to buy?
- Example: Saver choosing between mince options
- Agent needs to think like the customer
- Technical questions: product selection, pricing accuracy, product matching, persona modeling

**Talking Points:**
- Show the example table (3 mince options, different prices/promotions)
- Explain how agent uses persona rules to make realistic choices
- Emphasize validation against Feb 20 baseline

**Potential Questions:**
- Q: "How do you know the agent will choose correctly?"
  - A: Validated against Feb 20 manual shops, >90% accuracy target
- Q: "What if products aren't available?"
  - A: Agent selects alternatives based on persona rules (like real customers)

---

### Section 3: Data Sources (10 mins)
**Slides 13-19**

**Key Messages:**
- ✅ Woolworths data is EXCELLENT (via Stuart)
  - Allergens, country of origin, Australian %, Promise info
  - 3 separate promotion flags
- ✅ Coles data is COMPREHENSIVE (via Nissan)
  - Ingredients, allergens, nutrition
  - But has multi-postcode pricing issue
- 🔄 Aldi needs selective enrichment (~100 products)

**Talking Points:**
- This is BETTER than we expected (especially Woolworths attributes)
- Rich data enables quality persona decisions
- Only Aldi needs extra work (small scope)

**Critical Slide:** Slide 19 (Data Quality Assessment table)
- Show Owen the comparison across retailers
- Highlight that WW & Coles need NO enrichment

**Potential Questions:**
- Q: "Why is Aldi data worse?"
  - A: Third-party scraping has limitations, but we're enriching just ~100 products for MVP
- Q: "What's the Mascot store ID issue?"
  - A: Daria is confirming with Stuart to ensure we have full product access

---

### Section 4: Solution Architecture (10 mins)
**Slides 20-24**

**Key Messages:**
- Production-grade semantic search (not keyword matching)
- LLM-based persona decision engine (Gemini 1.5 Pro)
- Multi-stage: Search → LLM decision → Validation
- Example walkthrough (Saver buying mince)

**Talking Points:**
- Walk through the architecture diagram (Slide 20)
- Show the "How It Works" example (Slide 22-23)
- Emphasize: structured reasoning, confidence scores, auditable

**Critical Slide:** Slide 22 (How It Works example)
- Walk Owen through the full flow
- Show actual agent response with reasoning

**Potential Questions:**
- Q: "How expensive is the LLM?"
  - A: ~$100/month for daily automation (covered in cost section later)
- Q: "What if the LLM makes mistakes?"
  - A: Confidence scores, validation against baseline, human review in Week 4

---

### Section 5: 4-Week Plan (15 mins)
**Slides 25-46**

**Key Messages:**
- Week 1: Foundation & data quality (✅ on track)
- Week 2: LLM decision engine & personas (Saver, Traditional)
- Week 3: Scale to all 8 missions × 5 personas
- Week 4: Validation, analytics, documentation

**Talking Points:**
- Show weekly overview slides first (25, 33, 39, 43)
- Drill into daily breakdowns if Owen wants detail
- Emphasize weekly checkpoints (go/no-go criteria)

**Critical Slides:**
- Slide 26 (Week 1 overview) - show we're on track
- Slides 44-45 (Week 4 validation) - human vs agent comparison

**Potential Questions:**
- Q: "Is 4 weeks realistic?"
  - A: Yes, with confirmed data sources and clear checkpoints. Week 1 already in progress.
- Q: "What if you fall behind?"
  - A: Weekly go/no-go decisions. If Week 2 accuracy is low, we pause and fix.
- Q: "When can I see results?"
  - A: Week 2 checkpoint (May 2) you'll see first shops. Week 4 (May 16) full demo.

---

### Section 6: DECISIONS NEEDED (15 mins - CRITICAL)
**Slides 47-56**

**This is the most important section - Owen needs to make decisions**

#### Decision 1: Multi-Postcode Pricing (Slides 48-54)

**The Problem:**
- Some Coles products have different prices across postcodes
- Example: Same milk is $6.50 in Mascot, $7.00 in Bondi

**The Options:**
1. Average price
2. Median price
3. Specific postcode (Mascot/Inner Sydney) ← RECOMMENDED
4. Flag separately

**Your Recommendation:** Option 3 (Mascot/Inner Sydney postcode)
- Real price from real store
- Matches customer experience
- Comparable to WW Mascot service center
- Simplest implementation

**Ask Owen:** "We recommend using Mascot/Inner Sydney postcode for Coles pricing. Does this approach work for you?"

**Prepare for:**
- Q: "What's the price variance?"
  - A: We'll analyze in Week 1 Day 2 and share data (Daria to provide examples)
- Q: "Why not average?"
  - A: Average isn't a real price any customer would see. Specific postcode matches reality.

---

#### Decision 2: Mascot Store ID (Slide 55)

**The Problem:**
- Need to confirm Woolworths data is using correct store
- Mascot service center may/may not have store ID in tables

**Status:** Daria following up with Stuart

**Ask Owen:** "Just flagging this as a potential data access constraint. Daria is confirming with Stuart."

**No decision needed, just awareness**

---

#### Decision 3: Aldi Enrichment (Slide 56)

**The Proposal:**
- Selectively enrich ~100 Aldi products (not full catalog)
- Pre-scrape descriptions before Week 2
- Coordinate with pricing team on methodology

**Your Recommendation:** Proceed with selective enrichment
- Small scope (~100 products for 8 missions)
- Pre-caching avoids runtime risk
- Critical for Conscious/Refined persona quality

**Ask Owen:** "We recommend proceeding with Aldi enrichment (~100 products). Do you approve this approach?"

**Prepare for:**
- Q: "Why can't you just use product names?"
  - A: Aldi names lack quality indicators (organic, free range, etc.) needed for Conscious/Refined personas
- Q: "What if scraping fails?"
  - A: We're coordinating with pricing team on their proven methodology, plus pre-scraping means we cache results

---

### Section 7: Success Criteria & Risks (5 mins)
**Slides 57-63**

**Key Messages:**
- Clear success metrics (>95% search precision, >90% persona accuracy)
- Risks identified and mitigated
- Weekly checkpoints with go/no-go criteria

**Talking Points:**
- Show the metrics table (Slide 57-58)
- Walk through risk assessment (Slides 59-61)
- Emphasize mitigation strategies already in place

**Potential Questions:**
- Q: "What's the biggest risk?"
  - A: 4-week timeline. Mitigated by weekly checkpoints and clear go/no-go criteria.
- Q: "What if persona accuracy is too low?"
  - A: We pause, iterate on prompts, validate again. Week 2 checkpoint catches this early.

---

### Section 8: Costs (5 mins)
**Slides 64-66**

**Key Messages:**
- MVP development: ~$70 (mostly internal resource)
- Ongoing automation: ~$125/month (~$1,500/year)
- Very cost-effective for daily competitive intelligence

**Talking Points:**
- Emphasize low external costs (~$70 for MVP)
- LLM costs are reasonable (~$100/month for 120 daily shops)
- Phase 2 optimization can reduce to ~$60-80/month

**Potential Questions:**
- Q: "Why so cheap?"
  - A: Using existing data sources, modern LLM APIs are very affordable
- Q: "What if costs increase?"
  - A: We'll monitor and optimize (prompt caching, batch processing)

---

### Section 9: Next Steps (5 mins)
**Slides 67-73**

**Key Messages:**
- Need Owen's decisions today (multi-postcode, Aldi enrichment)
- Weekly checkpoints every Friday 4pm
- Go-live target: May 19, 2026

**Talking Points:**
- Review immediate actions (Slide 67)
- Confirm weekly checkpoint schedule (Slides 68-71)
- Communication plan (Slide 72)

**Close:**
- "We're on track for Week 1, data sources are better than expected"
- "Need your decisions on multi-postcode pricing and Aldi enrichment"
- "Weekly updates every Friday to keep you informed"

---

## Questions & Answers Preparation

### Likely Questions

**Technical:**
1. "How do you ensure the agent makes realistic decisions?"
   - A: Persona prompts based on CREST profiles, validated against Feb 20 baseline, low LLM temperature (0.3) for consistency

2. "What if products aren't found?"
   - A: Agent selects alternatives based on persona rules, tracks "not found" rate, we optimize search in Week 1

3. "How accurate is the pricing data?"
   - A: Multi-source validation (Woolworths direct, Coles competitor table), weekly refresh, confidence scoring

**Business:**
4. "When can the CEO see results?"
   - A: Week 4 demo (May 16), full dashboard ready by go-live (May 19)

5. "How does this compare to manual mystery shopping?"
   - A: Automated (daily vs one-time), scalable (120 shops vs manual), consistent methodology, auditable reasoning

6. "What's the ROI?"
   - A: ~$1,500/year for daily competitive intelligence. Manual mystery shopping costs far more per shop.

**Timeline:**
7. "Is 4 weeks realistic?"
   - A: Yes, with confirmed data sources, clear checkpoints, and focused scope (8 missions × 5 personas for MVP)

8. "What if you need more time?"
   - A: Weekly go/no-go decisions. We'll flag early if Week 2 accuracy is low.

**Scope:**
9. "Why only 8 missions for MVP?"
   - A: Proven methodology, validated against Feb 20 baseline, expandable in Phase 2

10. "Can you add more retailers later?"
    - A: Yes, architecture supports it. MVP focuses on WW/Coles/Aldi (the Feb 20 baseline)

---

## Key Numbers to Remember

- **8** missions (Spaghetti Bolognese, School Lunches, etc.)
- **5** personas (Saver, Traditional, Conscious, Refined, Essential)
- **3** retailers (Woolworths, Coles, Aldi)
- **120** shops per day (8 × 5 × 3)
- **~100** Aldi products to enrich (not full catalog)
- **~50,000** total products in catalog
- **4** weeks timeline
- **>95%** search precision target
- **>90%** persona accuracy target
- **$125/month** ongoing cost

---

## Owen's Decision Checklist

By end of meeting, Owen should decide:

1. ✅ **Multi-postcode pricing approach for Coles**
   - Recommended: Mascot/Inner Sydney postcode
   - Get his approval or alternative

2. ✅ **Aldi enrichment scope**
   - Recommended: Pre-scrape ~100 products
   - Get his approval

3. ✅ **Overall 4-week plan**
   - Confirm go-ahead
   - Confirm weekly checkpoints (Friday 4pm)

---

## Materials to Bring

1. **Slide deck** (OWEN_REVIEW_SLIDES.md)
2. **Data quality examples** (Daria to prepare)
   - Sample Woolworths products showing allergens, promotions
   - Sample Coles products showing ingredients
   - Sample Aldi products needing enrichment
3. **Feb 20 baseline data** (for reference)
4. **CREST persona profiles** (if Owen wants detail)
5. **Calendar invite** for weekly checkpoints

---

## Post-Meeting Actions

**If Owen approves:**
1. Email Owen with decisions documented
2. Share with Daria immediately
3. Update project plan with any changes
4. Schedule Week 2 checkpoint (Friday May 2, 4pm)
5. Begin Week 1 Day 2 tasks

**If Owen has concerns:**
1. Document specific concerns
2. Propose solutions or adjustments
3. Schedule follow-up if needed
4. Adjust timeline if necessary

---

## Tips for Presentation

1. **Start strong:** Executive summary sets the tone
2. **Use examples:** The mince example (Slide 11) makes it real
3. **Show the data:** Data quality table (Slide 19) proves feasibility
4. **Be clear on decisions:** Slides 47-56 are critical
5. **End with confidence:** We're on track, need your decisions, ready to execute

**Time Management:**
- Plan for 45 minutes presentation
- Leave 15 minutes for Q&A
- If Owen wants deep dive on any section, have detail ready (appendix slides)

**Body Language:**
- Confidence: Data sources are confirmed, plan is solid
- Transparency: Flag risks clearly, show mitigation
- Collaboration: We need his decisions, this is a partnership

---

## Success Criteria for This Meeting

**Meeting is successful if:**
1. ✅ Owen understands the approach and timeline
2. ✅ Owen makes decisions on multi-postcode pricing and Aldi enrichment
3. ✅ Owen approves overall 4-week plan
4. ✅ Weekly checkpoints confirmed
5. ✅ No major blockers or concerns raised

**Red flags to watch for:**
- ❌ Owen questions feasibility of 4-week timeline
- ❌ Owen has concerns about data quality
- ❌ Owen wants major scope changes
- ❌ Owen questions persona approach

**If red flags appear:**
- Listen carefully to concerns
- Propose solutions or adjustments
- Don't commit to unrealistic changes
- Schedule follow-up to address properly

---

Good luck! You've got a solid plan and the data to back it up. 🎯
