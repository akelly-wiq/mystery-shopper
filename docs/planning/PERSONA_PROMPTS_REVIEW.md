# Persona Prompts Review - personas.py
## Analysis of Daria's Implementation

**Reviewed:** May 6, 2026  
**File:** `/config/personas/prompts/personas.py`  
**Reviewer:** Alexa Kelly

---

## Executive Summary

✅ **Overall: GOOD** - Daria has done solid work converting the markdown prompts to Python strings. The structure is clean, prompts are comprehensive, and the decision frameworks are well-defined.

⚠️ **Minor Issues Found:**
1. Typos in 2 personas (REFINED, CONSCIOUS)
2. "SAVERS" vs "SAVER" naming inconsistency
3. Missing product list formatting in task section
4. Could benefit from explicit output format

✅ **Strengths:**
- Excellent priority ordering with "STRICT ORDER" emphasis
- Strong tie-breaker rules for consistency
- Good sample decisions for each persona
- Clear NEVER/ALWAYS sections
- Proper Python structure with helper functions

---

## Detailed Review by Persona

### 1. SAVERS (Should be SAVER)

**Status:** ✅ APPROVED with minor naming fix

**What's Good:**
- Decision priority is crystal clear (Price → Value → Family → Convenience)
- Tie-breaker rules cover edge cases well
- Sample decisions are practical and realistic
- "Follow this order. Do NOT skip to convenience" is excellent reinforcement

**Issues:**
- ❌ Variable name is `SAVERS` but should be `SAVER` for consistency (line 1, line 412)
- ❌ Sample decision says "SAVER PICKS" but variable is "SAVERS"

**Recommendation:** Rename `SAVERS` → `SAVER` throughout

---

### 2. TRADITIONAL

**Status:** ✅ APPROVED

**What's Good:**
- "Brand familiarity" as top priority is perfect
- "Own brand is LAST RESORT only" - very clear differentiation
- Tie-breaker for "preferred brand unavailable" scenario is excellent
- Sample decisions show brand loyalty over price

**Issues:**
- ✅ None - this is well-structured

**Observations:**
- This persona is well-differentiated from SAVER (brand over price)
- "perfers" (line 74) → should be "prefers" (typo in source .md file)

---

### 3. REFINED

**Status:** ✅ APPROVED with typo fix

**What's Good:**
- "Price is NOT a factor" is bold and clear
- "Quality is the ONLY priority" - excellent emphasis
- "Would I serve this to guests?" is a perfect framing
- Premium focus is consistent throughout

**Issues:**
- ❌ **Typo line 157:** "perfers qulaity over price" → should be "prefers quality over price"

**Recommendation:** Fix typo in line 157

---

### 4. ESSENTIAL

**Status:** ✅ APPROVED with naming fix

**What's Good:**
- "Price above all else" - clear differentiation from SAVER
- "Do I actually NEED this item?" is great framing
- "Small quantities - don't over-buy" differentiates from SAVER (who might buy bulk)
- Sample decisions emphasize "cheapest own brand"

**Issues:**
- ❌ Variable name is `ESSENTIAL` but context uses "essential shopper" (line 241)
- Note: This is consistent, but check if it should be "ESSENTIALS" or keep as "ESSENTIAL"

**Observations:**
- Well-differentiated from SAVER: ESSENTIAL is "smallest basket, own brand always" vs SAVER is "best unit price, might buy bulk if can freeze"

---

### 5. CONSCIOUS

**Status:** ✅ APPROVED with typo fix

**What's Good:**
- Health and sustainability priority is clear
- "Dietary requirements are NON-NEGOTIABLE" - excellent
- Tie-breaker "health over sustainability if budget tight" shows nuance
- "Read ingredients labels" emphasis is perfect

**Issues:**
- ❌ **Typo line 324:** "consious" → should be "conscious"

**Recommendation:** Fix typo in line 324

---

## Common Issues Across All Personas

### Issue 1: Product List Placeholder

**Current (lines 44, 118, 203, 284, 368):**
```python
### Products:
{product_list}
```

**Problem:** The variable name `{product_list}` is used, but there's no clear specification of how this should be formatted.

**Recommendation:** Add documentation or update to `{formatted_product_list}` and specify the expected format:

```python
### Products:
{formatted_product_list}

# Expected format:
# 1. Woolworths Own Brand Mince 1kg - $12.00 ($12/kg) - Yellow Ticket Special
# 2. Premium Grass-Fed Mince 1kg - $18.00 ($18/kg)
# 3. Mid-Range Brand Mince 1.5kg - $16.50 ($11/kg)
```

**Action:** Clarify with Daria how `product_list` is being formatted in the actual agent code.

---

### Issue 2: Output Format Not Specified

**Current:** Instructions say "Explain your reasoning" but don't specify output format.

**Recommendation:** Add explicit output format to ensure consistency:

```python
## Output Format:
Return your decision as JSON:
{
  "selected_product": "Product name from list",
  "reasoning": "Your explanation following the decision framework above",
  "confidence": 0.0-1.0
}
```

**Why:** This ensures the agent returns structured output that can be parsed programmatically.

**Action:** Check if Daria's agent framework handles this automatically or if it needs to be in the prompt.

---

### Issue 3: Missing Product Attributes in Task Context

**Current:** Sample decisions show promotions, pack sizes, unit prices

**Gap:** No explicit instruction to USE these attributes when they're provided

**Recommendation:** Add a section before "## Your Task":

```python
## Available Product Information:
You will be provided with the following details for each product:
- Product name and brand
- Price (absolute)
- Unit price (per kg or per L)
- Pack size
- Promotion type (Yellow Ticket, Red EDLP, none)
- Quality indicators (organic, free-range, etc.) if available

Use ALL available information to make your decision according to your priority framework.
```

**Action:** Confirm with Daria that all these attributes are actually being provided.

---

## Code Structure Review

### Python Structure: ✅ EXCELLENT

```python
PERSONA_TEMPLATES = {
    "savers": SAVERS,
    "traditional": TRADITIONAL,
    "refined": REFINED,
    "essential": ESSENTIAL, 
    "conscious": CONSCIOUS
}

def get_persona_template(persona_type: str) -> str:
    ...
    
def list_available_personas() -> list[str]:
    ...
```

**Strengths:**
- Clean mapping of persona names to templates
- Good error handling in `get_persona_template()` with helpful error message
- Helper function `list_available_personas()` is useful
- Type hints are present

**Issues:**
- ❌ Inconsistency: dictionary key is `"savers"` but should probably be `"saver"` (lowercase, singular)
- ⚠️ Consider: Should keys be lowercase (current: `"savers"`) or match variable names (would be `"SAVER"`)?

**Recommendation:** 
- Choose consistent naming: either all lowercase singular (`"saver"`, `"traditional"`, etc.) OR all uppercase (`"SAVER"`, `"TRADITIONAL"`)
- Update variable names to match

---

## Comparison to Original .md Files

### Changes Daria Made:

1. ✅ **Added task instructions** - "Your Task", "Instructions", clear directive to select ONE product
2. ✅ **Formatted for Python** - Converted markdown to triple-quoted strings
3. ✅ **Added product_list placeholder** - Dynamic content injection
4. ✅ **Maintained structure** - Kept Decision Priority, Framework, Tie-Breakers, NEVER/ALWAYS
5. ⚠️ **Removed some context** - CONSCIOUS lost "Key Differentiators" section with stats (60.8% ingredient checking, etc.)

**Overall Assessment:** Daria's changes are pragmatic and production-ready. The loss of statistical context is acceptable for token efficiency.

---

## Persona Differentiation Check

**Critical Question:** Will these personas make DIFFERENT decisions?

| Scenario | SAVER | TRADITIONAL | REFINED | ESSENTIAL | CONSCIOUS |
|----------|-------|-------------|---------|-----------|-----------|
| **Leggo's $5.50 vs WW Own $3.20** | Own brand (price) | Leggo's (familiar brand) | Neither (look for premium) | Own brand (cheapest) | Check ingredients first |
| **1kg bulk vs 500g small pack (same $/kg)** | 1kg (can freeze) | Depends on usual | Best quality regardless | 500g (don't over-buy) | 1kg if health-aligned |
| **Organic $7 vs Standard $4** | Standard (price) | Standard (familiar) | Organic (quality) | Standard (price) | Organic (health) |
| **Yellow ticket premium vs Own brand** | Whichever cheaper | Prefer premium even full price | Premium (ignore promotion) | Own brand (absolute cheapest) | Check ingredients, then decide |

**Result:** ✅ YES - Personas are well-differentiated and will make meaningfully different decisions.

---

## Recommendations Summary

### High Priority (Fix Before Testing)

1. **Fix typos:**
   - Line 157: "perfers qulaity" → "prefers quality"
   - Line 74: "perfers consistency" → "prefers consistency"
   - Line 241: "perfers smalles" → "prefers smallest"
   - Line 324: "consious" → "conscious"

2. **Fix naming inconsistency:**
   - Decide: `SAVER` or `SAVERS`? (Recommend: `SAVER`)
   - Decide: Dictionary keys `"saver"` or `"savers"`? (Recommend: `"saver"`)
   - Make consistent throughout file

3. **Clarify product_list format:**
   - Document expected format for `{product_list}` placeholder
   - Or add formatting specification in prompt

### Medium Priority (Improve Consistency)

4. **Add explicit output format:**
   - Specify JSON structure expected
   - Include confidence score requirement
   - Ensure parseable by agent framework

5. **Add product attributes section:**
   - List what information will be provided
   - Ensure persona uses all available data

### Low Priority (Nice-to-Have)

6. **Consider adding:**
   - Maximum response length guidance
   - Fallback instruction if no products match criteria
   - Explicit "if all options equally bad" scenario

---

## Testing Checklist for Alexa

When testing these prompts (Task 2 from May 6 meeting), validate:

- [ ] All 5 personas respond to same product list
- [ ] Responses are different across personas (differentiation working?)
- [ ] Output is parseable (JSON or structured format)
- [ ] Confidence scores are included
- [ ] Reasoning aligns with decision priority
- [ ] Tie-breaker rules are applied when appropriate
- [ ] Sample decisions match actual decisions when tested
- [ ] No errors with `{product_list}` placeholder substitution

---

## Final Verdict

**Status:** ✅ **APPROVED FOR TESTING** with minor fixes

**Action Items:**
1. Fix 4 typos identified above
2. Resolve SAVER/SAVERS naming inconsistency
3. Clarify product_list format with Daria
4. Add explicit output format requirement
5. Test all 5 personas with same product list
6. Validate differentiation is working

**Overall Assessment:** 
Daria has done **excellent work** converting the prompts. The structure is solid, decision frameworks are clear, and personas are well-differentiated. With the minor typo fixes and naming consistency updates, these prompts are production-ready.

**Estimated time to address issues:** 15-30 minutes

---

## Notes for Discussion with Daria

**Questions to ask:**

1. How is `{product_list}` being formatted when injected? 
   - Is it a numbered list? 
   - Are all attributes included (price, unit price, promotion)?

2. What output format does the agent framework expect?
   - JSON?
   - Plain text?
   - Is there a parser?

3. Should dictionary keys be singular (`"saver"`) or plural (`"savers"`)?

4. Are we tracking confidence scores? If yes, should it be in the prompt?

5. Did you implement the retry context mentioned in the meeting? If so, where?

---

*Original .md files reviewed: SAVER_PROMPT.md, CONSCIOUS_PROMPT.md*  
*Additional context: May 6 meeting notes, Task 1 from action items*
