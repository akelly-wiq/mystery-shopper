# Mystery Shopping Agent - Frontend Specifications
**Version:** 1.0  
**Date:** May 1, 2026  
**Author:** Alexa Kelly  
**For:** Frontend Developer Handoff

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [Technical Stack](#technical-stack)
3. [Data Sources & Schema](#data-sources--schema)
4. [User Flows](#user-flows)
5. [Screen Specifications](#screen-specifications)
6. [Component Library](#component-library)
7. [Visual Design System](#visual-design-system)
8. [Interactions & States](#interactions--states)
9. [Acceptance Criteria](#acceptance-criteria)
10. [Non-Functional Requirements](#non-functional-requirements)

---

## 1. Project Overview

### 1.1 Purpose
An AI-powered automated mystery shopping system that compares basket prices across Woolworths, Coles, and Aldi from a customer perspective. The frontend displays 120 automated mystery shops (8 missions × 5 personas × 3 retailers) with detailed basket comparisons and insights.

### 1.2 Users
- **Primary:** Owen Lim (CEO), Commercial team
- **Secondary:** Pricing team, CustomerX team
- **Use case:** Weekly review of competitive pricing, strategic decision-making

### 1.3 Key Business Requirements
- ✅ Show basket-by-basket comparison (matches manual Feb 2026 baseline format)
- ✅ Identify WHY price differences exist (pack sizes, promotions, product selection)
- ✅ Enable persona-based insights (how different customer segments experience pricing)
- ✅ Track trends over time (weekly price changes)
- ✅ Export to Google Sheets for presentations

### 1.4 Reference Materials
- **Design reference:** `/home/akelly5_woolworths_com_au/mystery-shopper/[26-01] Mystery shop baskets - for discussion (1).pdf` (Pages 3-5)
- **Persona descriptions:** `/home/akelly5_woolworths_com_au/mystery-shopper/Mystery Shopping agent - overview.pdf` (Pages 4-6)
- **Project brief:** `PROJECT_BRIEF.md`

---

## 2. Technical Stack

### 2.1 Recommended Stack: Looker Studio (Primary Option)

**Technology:** Google Looker Studio  
**Rationale:**
- Native BigQuery integration (no backend API needed)
- Fast development (1-2 days for MVP)
- Stakeholder-friendly (Google ecosystem, familiar interface)
- Built-in export to PDF/Sheets
- Free, no licensing costs

**Architecture:**
```
BigQuery Tables
      ↓
Looker Studio Dashboard
      ↓
Shareable Google Link → Stakeholders
```

**Development Time:** 15-20 hours  
**Deployment:** Publish to Google Cloud, share link  
**Authentication:** Google Workspace SSO

---

### 2.2 Alternative Stack: Streamlit (If More Customization Needed)

**Technology:** Streamlit + Python + BigQuery  
**Use if:** Need custom interactions, agent reasoning modals, or complex filtering

**Architecture:**
```
BigQuery Tables
      ↓
Python Backend (Streamlit app)
      ↓
Cloud Run (deployed container)
      ↓
HTTPS endpoint → Stakeholders
```

**Development Time:** 25-35 hours  
**Deployment:** Docker container on Cloud Run  
**Authentication:** Google OAuth or basic auth

**Tech Stack Details:**
- **Frontend:** Streamlit (Python framework)
- **Data Layer:** `google-cloud-bigquery` Python library
- **Visualization:** Plotly, Altair (for charts)
- **Deployment:** Cloud Run (serverless)
- **Cost:** ~$5-10/month

---

### 2.3 Not Recommended: Custom React/Next.js

**Why not:**
- Overkill for 2-3 stakeholder users
- Requires backend API development (40+ hours)
- 80-100 hours total development time
- Higher maintenance burden

**Only use if:** Building for 100+ users, need mobile app, or complex interactivity

---

## 3. Data Sources & Schema

### 3.1 BigQuery Connection

**GCP Project:** `[your-project-id]`  
**Dataset:** `mystery_shopper`  
**Location:** `australia-southeast1`  
**Tables:** 3 tables (detailed below)

**Connection String (Looker Studio):**
```
Project ID: [your-project-id]
Dataset: mystery_shopper
Billing Project: [your-project-id]
```

---

### 3.2 Table 1: `shop_results` (Item-Level Detail)

**Purpose:** Detailed product selections for each mission/persona/retailer

**Schema:**
| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `shop_id` | STRING | Unique ID for each shop execution | `2026-05-16_spag_saver_ww` |
| `date_executed` | DATE | When shop was run | `2026-05-16` |
| `week_label` | STRING | Human-readable week | `Week of May 16, 2026` |
| `mission_name` | STRING | Shopping mission | `Spaghetti Bolognese` |
| `mission_description` | STRING | Mission rationale | `Most common home cooked meal` |
| `persona` | STRING | Customer segment | `Saver` |
| `persona_description` | STRING | Short persona profile | `Seek weekly specials, shop by unit prices...` |
| `list_item` | STRING | Ingredient from mission | `Lean mince beef, 1kg` |
| `item_order` | INT | Display order in basket | `1` |
| `retailer` | STRING | Retailer name | `Woolworths`, `Coles`, `Aldi` |
| `product_id` | STRING | Product identifier | `WW123456` |
| `product_name` | STRING | Actual product selected | `Woolworths Own Brand Lean Beef Mince` |
| `brand` | STRING | Product brand | `Woolworths Own`, `Leggos`, `Coles` |
| `pack_size` | STRING | Pack size as displayed | `1kg`, `500g`, `2x250g` |
| `pack_size_normalized` | FLOAT | Normalized to grams/ml | `1000.0` |
| `price` | FLOAT | Actual price (not scaled) | `12.00` |
| `unit_price` | FLOAT | Price per kg/L | `12.00` |
| `promotion_type` | STRING | Promotion category | `red_edlp`, `yellow_ticket`, `half_price`, `none` |
| `promotion_description` | STRING | Promotion details | `Red EDLP`, `Yellow Ticket - Save $2` |
| `promotion_flag` | BOOLEAN | Has any promotion | `true` |
| `product_url` | STRING | Link to product page | `https://woolworths.com.au/...` |
| `agent_reasoning` | STRING | Why this product was selected | `Selected WW own brand for $12/kg as it meets...` |
| `confidence_score` | FLOAT | Agent confidence (0-1) | `0.92` |
| `alternative_product_id` | STRING | Backup if unavailable | `COLES456789` |
| `quality_markers` | STRING | Quality attributes (comma-separated) | `organic,free_range`, `NULL` |
| `own_brand_flag` | BOOLEAN | Is own brand | `true` |
| `status` | STRING | Item found or not | `found`, `not_found` |

**Row Count:** ~1,200 rows per week (120 shops × ~10 items per basket)  
**Query Pattern:** Filter by date, mission, persona, retailer

**Sample Query:**
```sql
SELECT 
  list_item,
  product_name,
  price,
  unit_price,
  promotion_description,
  retailer
FROM `mystery_shopper.shop_results`
WHERE date_executed = '2026-05-16'
  AND mission_name = 'Spaghetti Bolognese'
  AND persona = 'Saver'
ORDER BY retailer, item_order
```

---

### 3.3 Table 2: `basket_summaries` (Aggregate View)

**Purpose:** Basket-level totals and comparisons

**Schema:**
| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `shop_id` | STRING | Unique ID for each shop | `2026-05-16_spag_saver_ww` |
| `date_executed` | DATE | When shop was run | `2026-05-16` |
| `week_label` | STRING | Human-readable week | `Week of May 16, 2026` |
| `mission_name` | STRING | Shopping mission | `Spaghetti Bolognese` |
| `persona` | STRING | Customer segment | `Saver` |
| `retailer` | STRING | Retailer name | `Woolworths`, `Coles`, `Aldi` |
| `total_basket_price` | FLOAT | Total basket cost | `63.90` |
| `num_items` | INT | Number of items in basket | `10` |
| `num_items_found` | INT | Items successfully found | `10` |
| `coverage_pct` | FLOAT | % of items found | `100.0` |
| `num_promoted_items` | INT | Items on promotion | `3` |
| `num_own_brand_items` | INT | Own brand products | `5` |
| `woolworths_basket_price` | FLOAT | WW basket price (for comparison) | `63.90` |
| `price_vs_woolworths_dollar` | FLOAT | $ difference vs WW | `-2.90` (Coles cheaper by $2.90) |
| `price_vs_woolworths_pct` | FLOAT | % difference vs WW | `-4.5` (Coles 4.5% cheaper) |
| `is_woolworths_cheaper` | BOOLEAN | WW wins on price | `false` |
| `execution_time_seconds` | FLOAT | Shop execution time | `120.5` |

**Row Count:** ~120 rows per week (8 missions × 5 personas × 3 retailers)  
**Query Pattern:** Aggregate by mission, persona, retailer

**Sample Query:**
```sql
SELECT 
  mission_name,
  persona,
  retailer,
  total_basket_price,
  price_vs_woolworths_dollar,
  price_vs_woolworths_pct
FROM `mystery_shopper.basket_summaries`
WHERE date_executed = '2026-05-16'
  AND mission_name = 'Spaghetti Bolognese'
ORDER BY persona, retailer
```

---

### 3.4 Table 3: `cost_drivers` (Insights)

**Purpose:** Explain WHY price differences exist

**Schema:**
| Column | Type | Description | Example |
|--------|------|-------------|---------|
| `shop_id` | STRING | Unique shop ID | `2026-05-16_spag_saver_coles` |
| `date_executed` | DATE | When shop was run | `2026-05-16` |
| `mission_name` | STRING | Shopping mission | `Spaghetti Bolognese` |
| `persona` | STRING | Customer segment | `Saver` |
| `retailer_comparison` | STRING | Comparison pair | `Coles vs Woolworths` |
| `driver_type` | STRING | Type of cost driver | `promotion`, `pack_size`, `base_price`, `product_selection` |
| `driver_description` | STRING | Human-readable explanation | `Lean mince beef $2 cheaper due to Yellow ticket promo` |
| `item_name` | STRING | Item causing difference | `Lean mince beef, 1kg` |
| `dollar_impact` | FLOAT | $ impact on basket | `-2.00` |
| `pct_of_total_diff` | FLOAT | % contribution to total diff | `68.9` (69% of total $2.90 difference) |

**Row Count:** ~5-10 rows per basket comparison (top cost drivers)  
**Query Pattern:** Filter by shop_id or mission/persona/retailer

**Sample Query:**
```sql
SELECT 
  driver_description,
  dollar_impact,
  pct_of_total_diff
FROM `mystery_shopper.cost_drivers`
WHERE date_executed = '2026-05-16'
  AND mission_name = 'Spaghetti Bolognese'
  AND persona = 'Saver'
  AND retailer_comparison = 'Coles vs Woolworths'
ORDER BY ABS(dollar_impact) DESC
LIMIT 5
```

---

### 3.5 Data Refresh Schedule

**Frequency:** Weekly (Fridays)  
**Timing:** Friday 10:00 AM AEST (after PCB data refresh)  
**Process:** Automated Cloud Function triggers 120 shops → writes to BigQuery  
**Historical Data:** Retained indefinitely (for trend analysis)

**Latest Data Query:**
```sql
-- Get most recent week
SELECT MAX(date_executed) as latest_date
FROM `mystery_shopper.basket_summaries`
```

---

## 4. User Flows

### 4.1 Primary User Flow: Weekly Review

```
1. User lands on Executive Dashboard
   ↓
2. Reviews week summary (total across all shops)
   ↓
3. Identifies missions where WW is more expensive
   ↓
4. Drills into specific mission (e.g., "Spaghetti Bolognese")
   ↓
5. Views basket comparison for Saver persona
   ↓
6. Examines item-by-item differences
   ↓
7. Checks "Primary Cost Drivers" section
   ↓
8. Clicks product to see why it was selected
   ↓
9. Exports basket to Google Sheets for presentation
   ↓
10. Repeats for other missions/personas of interest
```

**Time per review:** 5-10 minutes  
**Frequency:** Weekly (Friday afternoons)

---

### 4.2 Secondary User Flow: Trend Analysis

```
1. User navigates to "Trend Analysis" tab
   ↓
2. Selects mission + persona
   ↓
3. Views 8-week price trend chart
   ↓
4. Identifies week where prices spiked
   ↓
5. Reviews "Top Price Movers" section
   ↓
6. Clicks specific item to see weekly history
   ↓
7. Notes insights for pricing strategy discussion
```

**Time per analysis:** 3-5 minutes  
**Frequency:** Monthly or when issues identified

---

### 4.3 Secondary User Flow: Persona Comparison

```
1. User navigates to "Persona Insights" tab
   ↓
2. Selects mission (e.g., "Spaghetti Bolognese")
   ↓
3. Views how all 5 personas shop the same basket
   ↓
4. Identifies which personas find WW more expensive
   ↓
5. Reviews different product selections per persona
   ↓
6. Notes which personas are price-sensitive vs quality-focused
```

**Time per analysis:** 3-5 minutes  
**Frequency:** Monthly or for specific persona insights

---

## 5. Screen Specifications

### 5.1 Screen 1: Executive Dashboard (Landing Page)

**URL/Route:** `/` or `/dashboard`  
**Purpose:** High-level overview of all 120 shops for the week  
**User:** CEO, Commercial team (weekly review)

---

#### 5.1.1 Layout

```
┌─────────────────────────────────────────────────────────────────────┐
│  [Logo] Mystery Shopping Agent         Week of: [May 16, 2026 ▼]    │
│  🔄 Last updated: 2 hours ago                          [Run Now]     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  📊 WEEK SUMMARY                                                     │
│  ┌──────────────────┬──────────────────┬──────────────────┐         │
│  │   Woolworths     │      Coles       │      Aldi        │         │
│  │                  │                   │                  │         │
│  │    $449.20       │    $447.40       │    $412.00       │         │
│  │    Baseline      │   -0.4% ✅       │   -8.3% ✅       │         │
│  │                  │   (-$1.80)        │   (-$37.20)      │         │
│  └──────────────────┴──────────────────┴──────────────────┘         │
│                                                                       │
│  Across all missions/personas, Woolworths is:                        │
│  • Cheaper in 2 of 8 missions (25%)                                  │
│  • More expensive in 6 of 8 missions (75%)                           │
│                                                                       │
├─────────────────────────────────────────────────────────────────────┤
│  FILTERS                                                              │
│  [All Personas ▼] [All Missions ▼] [All Retailers ▼]                │
│                                                                       │
├─────────────────────────────────────────────────────────────────────┤
│  🎯 MISSION EXPLORER                                                 │
│                                                                       │
│  ┌───────────────────────────────────────────────────────┐          │
│  │ Mission 1: Spaghetti Bolognese                        │          │
│  │ ┌─────────────┬──────────┬─────────────┬────────────┐ │          │
│  │ │ Persona     │ WW       │ Coles       │ Aldi       │ │          │
│  │ ├─────────────┼──────────┼─────────────┼────────────┤ │          │
│  │ │ Saver       │ $63.90   │ $60.92 ✅   │ $57.60 ✅  │ │          │
│  │ │             │ Baseline │ -4.5%       │ -9.9%      │ │          │
│  │ ├─────────────┼──────────┼─────────────┼────────────┤ │          │
│  │ │ Traditional │ $71.20   │ $72.72 ❌   │ $67.50 ✅  │ │          │
│  │ │             │ Baseline │ +2.1%       │ -5.2%      │ │          │
│  │ ├─────────────┼──────────┼─────────────┼────────────┤ │          │
│  │ │ Conscious   │ $78.50   │ $77.60 ✅   │ $75.50 ✅  │ │          │
│  │ │             │ Baseline │ -1.2%       │ -3.8%      │ │          │
│  │ ├─────────────┼──────────┼─────────────┼────────────┤ │          │
│  │ │ Refined     │ $89.00   │ $93.80 ❌   │ N/A        │ │          │
│  │ │             │ Baseline │ +5.4%       │ -          │ │          │
│  │ ├─────────────┼──────────┼─────────────┼────────────┤ │          │
│  │ │ Essential   │ $61.20   │ $59.30 ✅   │ $56.00 ✅  │ │          │
│  │ │             │ Baseline │ -3.1%       │ -8.5%      │ │          │
│  │ └─────────────┴──────────┴─────────────┴────────────┘ │          │
│  │                                           [View Details >] │       │
│  └───────────────────────────────────────────────────────┘          │
│                                                                       │
│  ┌───────────────────────────────────────────────────────┐          │
│  │ Mission 2: School Lunches                             │          │
│  │ ┌─────────────┬──────────┬─────────────┬────────────┐ │          │
│  │ │ Persona     │ WW       │ Coles       │ Aldi       │ │          │
│  │ ├─────────────┼──────────┼─────────────┼────────────┤ │          │
│  │ │ Saver       │ $62.10   │ $62.35 ❌   │ $51.10 ✅  │ │          │
│  │ │             │ Baseline │ +0.4%       │ -17.6%     │ │          │
│  │ │ ...         │ ...      │ ...         │ ...        │ │          │
│  │ └─────────────┴──────────┴─────────────┴────────────┘ │          │
│  │                                           [View Details >] │       │
│  └───────────────────────────────────────────────────────┘          │
│                                                                       │
│  ... [6 more missions collapsed] ...                                 │
│  [Show All Missions]                                                 │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

#### 5.1.2 Components

**Component 1: Header**
- Logo: Woolworths logo or "Mystery Shopping Agent" text
- Week selector: Dropdown to select historical weeks
- Last updated: Timestamp of last data refresh
- Run Now button: Trigger manual shop execution (admin only)

**Component 2: Week Summary Cards**
- 3 cards (Woolworths, Coles, Aldi)
- Total basket price (sum across all 120 shops)
- $ and % difference vs Woolworths
- Color coding: Green (cheaper than WW), Red (more expensive)

**Component 3: Summary Stats**
- Text summary of WW performance
- % of missions where WW is cheaper/more expensive

**Component 4: Filters**
- Persona dropdown (All, Saver, Traditional, Conscious, Refined, Essential)
- Mission dropdown (All, [8 mission names])
- Retailer dropdown (All, Woolworths, Coles, Aldi)
- Filter behavior: Update Mission Explorer table

**Component 5: Mission Explorer Table**
- Accordion/collapsible sections (one per mission)
- Default: First 2 missions expanded, rest collapsed
- Table: Persona × Retailer grid
  - Columns: Persona, WW, Coles, Aldi
  - Rows: 5 personas
  - Cell content: Price + % difference
  - Color coding: Green ✅ (competitor cheaper), Red ❌ (competitor more expensive)
- "View Details" button: Navigate to Basket Comparison view

---

#### 5.1.3 Data Source

**BigQuery Query:**
```sql
-- Week summary (top cards)
SELECT 
  retailer,
  SUM(total_basket_price) as total_price,
  SUM(CASE WHEN retailer = 'Woolworths' THEN total_basket_price ELSE 0 END) as ww_total,
  SUM(total_basket_price) - ww_total as price_diff_dollar,
  (SUM(total_basket_price) - ww_total) / ww_total * 100 as price_diff_pct
FROM `mystery_shopper.basket_summaries`
WHERE date_executed = [selected_week]
GROUP BY retailer

-- Mission explorer
SELECT 
  mission_name,
  persona,
  retailer,
  total_basket_price,
  price_vs_woolworths_dollar,
  price_vs_woolworths_pct,
  is_woolworths_cheaper
FROM `mystery_shopper.basket_summaries`
WHERE date_executed = [selected_week]
ORDER BY mission_name, persona, retailer
```

---

#### 5.1.4 Interactions

| Action | Behavior |
|--------|----------|
| **Select week** | Refresh all data for selected week |
| **Apply filters** | Update Mission Explorer to show filtered missions/personas/retailers |
| **Click "View Details"** | Navigate to Basket Comparison screen (Screen 2) with mission/persona pre-selected |
| **Expand/collapse mission** | Show/hide persona breakdown table |
| **Click "Run Now"** (admin) | Trigger manual shop execution, show loading state, refresh after 5-10 min |

---

#### 5.1.5 Visual Design

**Colors:**
- ✅ Green (competitor cheaper): `#00A862` (Woolworths green)
- ❌ Red (competitor more expensive): `#E31837` (Woolworths red)
- Neutral: `#333333` (text), `#F5F5F5` (background)

**Typography:**
- Header: 24px bold, Woolworths font (or Open Sans)
- Body: 14px regular
- Table cells: 13px

**Spacing:**
- Card padding: 20px
- Table row height: 40px
- Section margin: 30px vertical

**Icons:**
- ✅ Checkmark (competitor cheaper)
- ❌ Cross (competitor more expensive)
- 🔄 Refresh icon
- 📊 Chart icon

---

### 5.2 Screen 2: Basket Comparison (Detailed View)

**URL/Route:** `/basket/{mission}/{persona}/{week}`  
**Example:** `/basket/spaghetti-bolognese/saver/2026-05-16`  
**Purpose:** Item-by-item basket comparison across 3 retailers  
**User:** CEO, Commercial team (detailed analysis)

---

#### 5.2.1 Layout

```
┌─────────────────────────────────────────────────────────────────────┐
│  ← Back to Dashboard                                                 │
│                                                                       │
│  Mission: Spaghetti Bolognese  |  Persona: Saver  |  Week: May 16   │
│  "Seek weekly specials, shop by unit prices, will consider own brand│
│   for lower price"                                                   │
├─────────────────────────────────────────────────────────────────────┤
│  BASKET COMPARISON                                                   │
│                                                                       │
│  ┌────────────┬─────────────────┬─────────────────┬────────────────┐│
│  │ Ingredient │  Woolworths     │     Coles       │     Aldi       ││
│  ├────────────┼─────────────────┼─────────────────┼────────────────┤│
│  │ Lean mince │ WW Own Brand    │ Coles 3-Star   │ Standard       ││
│  │ beef, 1kg  │ Red EDLP 🔴     │ Yellow 🟡      │                ││
│  │            │ $12.00          │ $10.00 💚      │ $11.50         ││
│  │            │ $12.00/kg       │ $10.00/kg 💚   │ $11.50/kg      ││
│  │            │ [View product]  │ [View product] │ [View product] ││
│  ├────────────┼─────────────────┼─────────────────┼────────────────┤│
│  │ Pasta sauce│ Leggos          │ Coles Own      │ Casa Italia    ││
│  │ 500g       │ Regular price   │ Regular        │                ││
│  │            │ $3.50           │ $2.80 💚       │ $2.99 💚       ││
│  │            │ $7.00/kg        │ $5.60/kg 💚    │ $5.98/kg 💚    ││
│  │            │ [View product]  │ [View product] │ [View product] ││
│  ├────────────┼─────────────────┼─────────────────┼────────────────┤│
│  │ Spaghetti  │ San Remo        │ Coles Own      │ Remano         ││
│  │ 500g       │ Half Price 🟠  │ Regular        │                ││
│  │            │ $1.50 💚        │ $1.75          │ $1.60          ││
│  │            │ $3.00/kg 💚     │ $3.50/kg       │ $3.20/kg       ││
│  │            │ [View product]  │ [View product] │ [View product] ││
│  ├────────────┼─────────────────┼─────────────────┼────────────────┤│
│  │ Tomato     │ WW Own          │ Coles Own      │ Casa Italia    ││
│  │ paste,     │ Regular         │ Regular        │                ││
│  │ 140g tub   │ $1.80           │ $1.65 💚       │ $1.70 💚       ││
│  │            │ $12.86/kg       │ $11.79/kg 💚   │ $12.14/kg      ││
│  │            │ [View product]  │ [View product] │ [View product] ││
│  ├────────────┼─────────────────┼─────────────────┼────────────────┤│
│  │ ... [6 more items] ...                                          ││
│  ├────────────┼─────────────────┼─────────────────┼────────────────┤│
│  │ TOTAL      │   $63.90        │ $60.92 💚      │ $57.60 💚      ││
│  │ BASKET     │   Baseline      │ -$2.98 (-4.5%) │-$6.30 (-9.9%)  ││
│  └────────────┴─────────────────┴─────────────────┴────────────────┘│
│                                                                       │
│  📌 PRIMARY COST DRIVERS                                             │
│  ┌──────────────────────────────────────────────────────────────┐   │
│  │ Why is Coles cheaper?                                         │   │
│  │ • Lean mince beef: $2.00 cheaper (Yellow ticket promo)       │   │
│  │ • Pasta sauce: $0.70 cheaper (own brand selection)           │   │
│  │ • Parmesan cheese: $0.50 cheaper (promotion)                 │   │
│  │                                                                │   │
│  │ Why is Aldi cheaper?                                          │   │
│  │ • Lean mince beef: $0.50 cheaper (everyday low price)        │   │
│  │ • Overall basket: Lower base prices across most items        │   │
│  └──────────────────────────────────────────────────────────────┘   │
│                                                                       │
│  [📊 View Price Trends] [📄 Export to Sheets] [🔗 Share Link]       │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

#### 5.2.2 Components

**Component 1: Breadcrumb Navigation**
- "← Back to Dashboard" link
- Mission + Persona + Week display
- Persona description (1-2 sentence quote from persona profiles)

**Component 2: Basket Comparison Table**
- Header row: Ingredient | Woolworths | Coles | Aldi
- Data rows: One per ingredient (10-15 items typical)
- Cell content per retailer:
  - Product name (brand + description)
  - Promotion indicator (🔴 Red EDLP, 🟡 Yellow ticket, 🟠 Half price)
  - Price (actual, not scaled)
  - Unit price (per kg/L)
  - "View product" link (opens modal - Screen 2.1)
  - 💚 Green indicator if cheapest option
- Footer row: Total basket
  - WW: Total + "Baseline"
  - Coles/Aldi: Total + $ difference + % difference

**Component 3: Primary Cost Drivers**
- Section per competitor (Coles, Aldi)
- Bulleted list of top 3-5 drivers
- Format: "[Item]: $X cheaper/more expensive ([reason])"

**Component 4: Action Buttons**
- "View Price Trends": Navigate to Trend Analysis (Screen 3) with pre-filled filters
- "Export to Sheets": Download basket as Google Sheets
- "Share Link": Copy shareable URL to clipboard

---

#### 5.2.3 Data Source

**BigQuery Query:**
```sql
-- Item-level detail
SELECT 
  sr.item_order,
  sr.list_item,
  sr.retailer,
  sr.product_name,
  sr.brand,
  sr.pack_size,
  sr.price,
  sr.unit_price,
  sr.promotion_type,
  sr.promotion_description,
  sr.product_url,
  sr.agent_reasoning,
  sr.confidence_score
FROM `mystery_shopper.shop_results` sr
WHERE sr.date_executed = [selected_week]
  AND sr.mission_name = [selected_mission]
  AND sr.persona = [selected_persona]
ORDER BY sr.item_order, sr.retailer

-- Basket totals
SELECT 
  retailer,
  total_basket_price,
  price_vs_woolworths_dollar,
  price_vs_woolworths_pct
FROM `mystery_shopper.basket_summaries`
WHERE date_executed = [selected_week]
  AND mission_name = [selected_mission]
  AND persona = [selected_persona]

-- Cost drivers
SELECT 
  driver_description,
  dollar_impact
FROM `mystery_shopper.cost_drivers`
WHERE date_executed = [selected_week]
  AND mission_name = [selected_mission]
  AND persona = [selected_persona]
  AND retailer_comparison IN ('Coles vs Woolworths', 'Aldi vs Woolworths')
ORDER BY ABS(dollar_impact) DESC
LIMIT 5
```

---

#### 5.2.4 Interactions

| Action | Behavior |
|--------|----------|
| **Click "Back to Dashboard"** | Navigate to Executive Dashboard (Screen 1) |
| **Click "View product"** | Open Product Detail Modal (Screen 2.1) |
| **Click "View Price Trends"** | Navigate to Trend Analysis (Screen 3) with filters pre-filled |
| **Click "Export to Sheets"** | Download basket as Google Sheets file (see format in Section 5.2.6) |
| **Click "Share Link"** | Copy URL to clipboard, show toast "Link copied!" |

---

#### 5.2.5 Visual Design

**Promotion Indicators:**
- 🔴 Red EDLP: Red circle icon, tooltip "Red EDLP"
- 🟡 Yellow Ticket: Yellow circle icon, tooltip "Yellow Ticket - Weekly Special"
- 🟠 Half Price: Orange circle icon, tooltip "Half Price"

**Price Comparison:**
- 💚 Green checkmark: Cheapest option for that ingredient across 3 retailers
- Bold text: Cheapest price
- Regular text: Other prices

**Table Styling:**
- Alternating row colors: White, `#F9F9F9`
- Border: 1px solid `#E0E0E0`
- Header row: Bold, `#F5F5F5` background
- Footer row (totals): Bold, `#E8F5E9` background

---

#### 5.2.6 Export Format (Google Sheets)

**Filename:** `Mystery_Shop_[Mission]_[Persona]_[Week].xlsx`  
**Example:** `Mystery_Shop_Spaghetti_Bolognese_Saver_2026-05-16.xlsx`

**Sheet 1: Basket Comparison**

| Ingredient | Woolworths Product | WW Price | WW Unit Price | WW Promo | Coles Product | Coles Price | Coles Unit Price | Coles Promo | Aldi Product | Aldi Price | Aldi Unit Price | Aldi Promo |
|------------|-------------------|----------|---------------|----------|---------------|-------------|------------------|-------------|--------------|------------|-----------------|------------|
| Lean mince beef, 1kg | WW Own Brand | $12.00 | $12.00/kg | Red EDLP | Coles 3-Star | $10.00 | $10.00/kg | Yellow Ticket | Standard | $11.50 | $11.50/kg | - |
| ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... | ... |
| **TOTAL** | | **$63.90** | | | | **$60.92** | | | | **$57.60** | | |
| **vs WW** | | | | | | **-$2.98 (-4.5%)** | | | | **-$6.30 (-9.9%)** | | |

**Sheet 2: Cost Drivers**

| Retailer | Driver | $ Impact |
|----------|--------|----------|
| Coles | Lean mince beef $2.00 cheaper (Yellow ticket promo) | -$2.00 |
| Coles | Pasta sauce $0.70 cheaper (own brand) | -$0.70 |
| Aldi | Lean mince beef $0.50 cheaper (everyday low) | -$0.50 |
| ... | ... | ... |

---

### 5.3 Screen 2.1: Product Detail Modal (Overlay)

**Trigger:** Click "View product" in Basket Comparison  
**Type:** Modal overlay (not full page)  
**Purpose:** Show why a product was selected + competitor comparison

---

#### 5.3.1 Layout

```
┌───────────────────────────────────────────────────────────┐
│  PRODUCT DETAIL                                      [✕]  │
├───────────────────────────────────────────────────────────┤
│                                                            │
│  🥩 Woolworths Own Brand Lean Beef Mince                  │
│                                                            │
│  Selected for: Spaghetti Bolognese | Saver persona       │
│                                                            │
│  ┌────────────────────────────────────────────────────┐  │
│  │ PRODUCT INFO                                        │  │
│  │ • Pack size: 1kg                                   │  │
│  │ • Price: $12.00 ($12.00/kg)                        │  │
│  │ • Promotion: Red EDLP 🔴                           │  │
│  │ • Brand: Woolworths Own                            │  │
│  │ • Quality: Standard (not organic/free range)      │  │
│  │ • URL: [woolworths.com.au/...] 🔗                 │  │
│  └────────────────────────────────────────────────────┘  │
│                                                            │
│  ┌────────────────────────────────────────────────────┐  │
│  │ WHY THIS PRODUCT WAS SELECTED                      │  │
│  │                                                     │  │
│  │ Agent reasoning (Saver persona):                   │  │
│  │ "Selected Woolworths own brand lean mince for      │  │
│  │  $12.00/kg as it meets the minimum 1kg pack size  │  │
│  │  requirement and is on Red EDLP promotion. While  │  │
│  │  Coles has a yellow ticket at $10.00, this WW     │  │
│  │  option provides consistent value for weekly      │  │
│  │  shopping."                                        │  │
│  │                                                     │  │
│  │ Confidence: 0.92 (High)                            │  │
│  │ Alternative if unavailable: Coles 3-star mince     │  │
│  └────────────────────────────────────────────────────┘  │
│                                                            │
│  ┌────────────────────────────────────────────────────┐  │
│  │ COMPETITOR COMPARISON                              │  │
│  │                                                     │  │
│  │ Coles 3-Star Lean Mince (1kg)                      │  │
│  │ • $10.00 ($10.00/kg) - Yellow ticket 🟡           │  │
│  │ • $2.00 cheaper 💚                                │  │
│  │                                                     │  │
│  │ Aldi Standard Lean Mince (1kg)                     │  │
│  │ • $11.50 ($11.50/kg)                               │  │
│  │ • $0.50 cheaper 💚                                │  │
│  └────────────────────────────────────────────────────┘  │
│                                                            │
│  [Close]                                                   │
└───────────────────────────────────────────────────────────┘
```

---

#### 5.3.2 Components

**Component 1: Product Header**
- Product name (with icon if category known)
- Mission + Persona context

**Component 2: Product Info Card**
- Pack size, price, unit price
- Promotion indicator
- Brand name
- Quality markers (if any)
- Link to product page (external)

**Component 3: Agent Reasoning Card**
- Verbatim agent reasoning text
- Confidence score (0-1, displayed as High/Medium/Low)
- Alternative product mention

**Component 4: Competitor Comparison**
- List of competing products (same ingredient at other retailers)
- Price comparison ($ cheaper/more expensive)
- Promotion indicators

---

#### 5.3.3 Data Source

**BigQuery Query:**
```sql
-- Selected product detail
SELECT 
  product_name,
  brand,
  pack_size,
  price,
  unit_price,
  promotion_type,
  promotion_description,
  product_url,
  agent_reasoning,
  confidence_score,
  alternative_product_id,
  quality_markers
FROM `mystery_shopper.shop_results`
WHERE shop_id = [selected_shop_id]
  AND list_item = [selected_ingredient]
  AND retailer = [selected_retailer]

-- Competitor products (same ingredient, other retailers)
SELECT 
  retailer,
  product_name,
  pack_size,
  price,
  unit_price,
  promotion_description
FROM `mystery_shopper.shop_results`
WHERE shop_id = [selected_shop_id]
  AND list_item = [selected_ingredient]
  AND retailer != [selected_retailer]
```

---

#### 5.3.4 Interactions

| Action | Behavior |
|--------|----------|
| **Click [✕] or "Close"** | Close modal, return to Basket Comparison |
| **Click product URL** | Open product page in new tab (external link) |
| **Click anywhere outside modal** | Close modal |

---

#### 5.3.5 Visual Design

**Modal Dimensions:**
- Width: 600px
- Max height: 80vh (scrollable if content overflows)
- Background overlay: `rgba(0,0,0,0.5)`

**Confidence Score Display:**
- High (0.8-1.0): Green badge "High confidence"
- Medium (0.5-0.79): Yellow badge "Medium confidence"
- Low (<0.5): Red badge "Low confidence"

---

### 5.4 Screen 3: Trend Analysis

**URL/Route:** `/trends`  
**Purpose:** Historical price trends over time (8-week view)  
**User:** Commercial team (monthly analysis)

---

#### 5.4.1 Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  ← Back to Dashboard                                             │
│                                                                   │
│  PRICE TRENDS                                                    │
│                                                                   │
│  Mission: [Spaghetti Bolognese ▼]  Persona: [Saver ▼]          │
│  Date range: [Last 8 weeks ▼]                                   │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  TOTAL BASKET TREND                                              │
│                                                                   │
│  $70 ┐                                                           │
│      │     ●────●────●                                           │
│  $65 │    /          \●──●         ● Woolworths                  │
│      │   /              \          ■ Coles                       │
│  $60 ├──●                ●────●    ▼ Aldi                        │
│      │   ▲──▲──▲──▲──▲──▲──▲──▲                                 │
│  $55 │    ■  ■  ■  ■  ■  ■  ■  ■                                │
│      │     ▼  ▼  ▼  ▼  ▼  ▼  ▼  ▼                               │
│  $50 └─┬──┬──┬──┬──┬──┬──┬──┬──                                │
│       W1 W2 W3 W4 W5 W6 W7 W8                                   │
│     Mar5    Mar19   Apr2    Apr16   Apr30                       │
│                                                                   │
│  KEY INSIGHTS:                                                   │
│  • Woolworths basket increased $3.20 (+5.3%) in Week 5 (Apr 2)  │
│    Driver: Lean mince beef price increase ($11→$12)             │
│  • Coles consistently 4-5% cheaper for Saver persona             │
│  • Aldi price stable (low variance week to week)                 │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ TOP PRICE MOVERS (Last 8 weeks)                            │ │
│  │                                                              │ │
│  │ 1. Lean mince beef (WW): +$1.00 (+8.3%)                    │ │
│  │    • Week 5 (Apr 2): Red EDLP removed                       │ │
│  │    [View item trend →]                                      │ │
│  │                                                              │ │
│  │ 2. Pasta sauce (Coles): -$0.50 (-15.2%)                    │ │
│  │    • Week 3-7: Yellow ticket promotion                      │ │
│  │    [View item trend →]                                      │ │
│  │                                                              │ │
│  │ 3. Parmesan cheese (Aldi): +$0.80 (+13.1%)                 │ │
│  │    • Week 6 (Apr 16): Price increase across category        │ │
│  │    [View item trend →]                                      │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

#### 5.4.2 Components

**Component 1: Filters**
- Mission dropdown (8 missions)
- Persona dropdown (5 personas)
- Date range dropdown (Last 4 weeks, Last 8 weeks, Last 12 weeks, Custom)

**Component 2: Line Chart**
- X-axis: Weeks (with dates)
- Y-axis: Basket price ($)
- 3 lines: Woolworths (●), Coles (■), Aldi (▼)
- Hover tooltip: Week, retailer, price
- Y-axis starts at $50 (not 0) for better scale

**Component 3: Key Insights**
- 3-5 bullet points
- Auto-generated based on trend data
- Format: "Event description + driver"

**Component 4: Top Price Movers Table**
- Rank ordered by absolute $ change
- Item name + retailer
- $ and % change over period
- Explanation (promotion added/removed, category price change)
- "View item trend" link (opens item-level trend modal)

---

#### 5.4.3 Data Source

**BigQuery Query:**
```sql
-- Basket trend over time
SELECT 
  date_executed,
  retailer,
  total_basket_price
FROM `mystery_shopper.basket_summaries`
WHERE mission_name = [selected_mission]
  AND persona = [selected_persona]
  AND date_executed >= DATE_SUB(CURRENT_DATE(), INTERVAL 8 WEEK)
ORDER BY date_executed, retailer

-- Top price movers
WITH price_changes AS (
  SELECT 
    list_item,
    retailer,
    FIRST_VALUE(price) OVER (PARTITION BY list_item, retailer ORDER BY date_executed) as first_price,
    LAST_VALUE(price) OVER (PARTITION BY list_item, retailer ORDER BY date_executed) as last_price,
    LAST_VALUE(price) OVER (PARTITION BY list_item, retailer ORDER BY date_executed) - 
      FIRST_VALUE(price) OVER (PARTITION BY list_item, retailer ORDER BY date_executed) as price_change
  FROM `mystery_shopper.shop_results`
  WHERE mission_name = [selected_mission]
    AND persona = [selected_persona]
    AND date_executed >= DATE_SUB(CURRENT_DATE(), INTERVAL 8 WEEK)
)
SELECT 
  list_item,
  retailer,
  price_change,
  (price_change / first_price * 100) as pct_change
FROM price_changes
WHERE ABS(price_change) > 0.50
ORDER BY ABS(price_change) DESC
LIMIT 5
```

---

#### 5.4.4 Interactions

| Action | Behavior |
|--------|----------|
| **Change filters** | Refresh chart + insights |
| **Hover over data point** | Show tooltip with week, retailer, price |
| **Click "View item trend"** | Open item-level trend modal (Screen 3.1) |

---

### 5.5 Screen 4: Persona Comparison

**URL/Route:** `/personas`  
**Purpose:** Compare how different personas shop the same mission  
**User:** CustomerX team (persona insights)

---

#### 5.5.1 Layout

```
┌─────────────────────────────────────────────────────────────────┐
│  ← Back to Dashboard                                             │
│                                                                   │
│  PERSONA INSIGHTS                                                │
│                                                                   │
│  Mission: [Spaghetti Bolognese ▼]  Week: [May 16 ▼]            │
├─────────────────────────────────────────────────────────────────┤
│                                                                   │
│  HOW DIFFERENT CUSTOMER SEGMENTS SEE THIS BASKET                 │
│                                                                   │
│  ┌──────────┬──────────┬──────────┬──────────┬──────────────┐  │
│  │ Persona  │ WW       │ Coles    │ Aldi     │ Winner       │  │
│  ├──────────┼──────────┼──────────┼──────────┼──────────────┤  │
│  │ Saver    │ $63.90   │ $60.92   │ $57.60   │ Aldi 💚     │  │
│  │          │ Baseline │ -4.5% ✅ │ -9.9% ✅ │              │  │
│  │          │ Promo-focused, own brand selections            │  │
│  │          │                    [View basket comparison →] │  │
│  ├──────────┼──────────┼──────────┼──────────┼──────────────┤  │
│  │ Trad.    │ $71.20   │ $72.72   │ $67.50   │ Aldi 💚     │  │
│  │          │ Baseline │ +2.1% ❌ │ -5.2% ✅ │              │  │
│  │          │ Brand-loyal (Leggos sauce selected)            │  │
│  │          │                    [View basket comparison →] │  │
│  ├──────────┼──────────┼──────────┼──────────┼──────────────┤  │
│  │ Conscious│ $78.50   │ $77.60   │ $75.50   │ Aldi 💚     │  │
│  │          │ Baseline │ -1.2% ✅ │ -3.8% ✅ │              │  │
│  │          │ Organic mince, free-range preferences          │  │
│  │          │                    [View basket comparison →] │  │
│  ├──────────┼──────────┼──────────┼──────────┼──────────────┤  │
│  │ Refined  │ $89.00   │ $93.80   │ N/A      │ WW 💚       │  │
│  │          │ Baseline │ +5.4% ❌ │ -        │              │  │
│  │          │ Premium products (grass-fed mince)             │  │
│  │          │                    [View basket comparison →] │  │
│  ├──────────┼──────────┼──────────┼──────────┼──────────────┤  │
│  │ Essential│ $61.20   │ $59.30   │ $56.00   │ Aldi 💚     │  │
│  │          │ Baseline │ -3.1% ✅ │ -8.5% ✅ │              │  │
│  │          │ Price-first, smallest pack sizes               │  │
│  │          │                    [View basket comparison →] │  │
│  └──────────┴──────────┴──────────┴──────────┴──────────────┘  │
│                                                                   │
│  KEY INSIGHTS:                                                   │
│  • Aldi wins for 4 of 5 personas (price-sensitive segments)     │
│  • Woolworths competitive for Refined (premium product range)    │
│  • Conscious persona: Coles competitive with better organic range│
│  • Traditional persona: WW loses despite brand loyalty focus     │
│                                                                   │
│  ┌────────────────────────────────────────────────────────────┐ │
│  │ DECISION QUALITY VALIDATION                                 │ │
│  │                                                              │ │
│  │ Saver persona shopping behavior:                            │ │
│  │ • Selected promoted items: 6 of 10 (60%) ✅                │ │
│  │ • Selected own brand: 7 of 10 (70%) ✅                     │ │
│  │ • Average confidence: 0.89 ✅                               │ │
│  │ ✅ High quality - matches expected Saver behavior          │ │
│  │                                                              │ │
│  │ Conscious persona shopping behavior:                        │ │
│  │ • Selected organic/free-range: 4 of 10 (40%) ⚠️           │ │
│  │ • Selected sustainable: 2 of 10 (20%) ⚠️                  │ │
│  │ • Average confidence: 0.72 ⚠️                              │ │
│  │ ⚠️ Medium quality - may need more attribute data           │ │
│  └────────────────────────────────────────────────────────────┘ │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

---

#### 5.5.2 Components

**Component 1: Filters**
- Mission dropdown (8 missions)
- Week dropdown (historical weeks)

**Component 2: Persona Comparison Table**
- Rows: 5 personas
- Columns: Persona, WW, Coles, Aldi, Winner
- Cell content: Price, % vs WW, persona behavior summary
- "View basket comparison" link per persona

**Component 3: Key Insights**
- Auto-generated insights
- Format: "[Retailer] wins for [X] personas ([segment description])"

**Component 4: Decision Quality Validation**
- Validation per persona (show top 2-3)
- Metrics: % promoted items, % own brand, avg confidence
- Quality badge: ✅ High, ⚠️ Medium, ❌ Low

---

#### 5.5.3 Data Source

**BigQuery Query:**
```sql
-- Persona comparison
SELECT 
  persona,
  retailer,
  total_basket_price,
  price_vs_woolworths_pct,
  is_woolworths_cheaper,
  num_promoted_items,
  num_own_brand_items,
  num_items
FROM `mystery_shopper.basket_summaries`
WHERE date_executed = [selected_week]
  AND mission_name = [selected_mission]
ORDER BY persona, retailer

-- Decision quality
WITH persona_stats AS (
  SELECT 
    persona,
    retailer,
    SUM(CASE WHEN promotion_flag THEN 1 ELSE 0 END) as promo_count,
    SUM(CASE WHEN own_brand_flag THEN 1 ELSE 0 END) as own_brand_count,
    COUNT(*) as total_items,
    AVG(confidence_score) as avg_confidence
  FROM `mystery_shopper.shop_results`
  WHERE date_executed = [selected_week]
    AND mission_name = [selected_mission]
  GROUP BY persona, retailer
)
SELECT 
  persona,
  retailer,
  promo_count / total_items as promo_pct,
  own_brand_count / total_items as own_brand_pct,
  avg_confidence
FROM persona_stats
WHERE retailer = 'Woolworths'
```

---

## 6. Component Library

### 6.1 Reusable Components

**Component:** Price Badge
- **Purpose:** Display price with vs WW comparison
- **Props:** `price`, `vsWoolworthsPct`, `isWoolworthsCheaper`
- **Variants:**
  - Baseline: "$63.90" (no comparison)
  - Cheaper: "$60.92 💚 -4.5%"
  - More expensive: "$65.20 ❌ +2.1%"

**Component:** Promotion Indicator
- **Purpose:** Show promotion type
- **Props:** `promotionType`
- **Variants:**
  - Red EDLP: 🔴 red circle
  - Yellow Ticket: 🟡 yellow circle
  - Half Price: 🟠 orange circle
  - None: no icon

**Component:** Mission Card
- **Purpose:** Collapsible mission section in dashboard
- **Props:** `missionName`, `personas`, `retailers`, `data`
- **States:** Expanded, Collapsed

**Component:** Data Table
- **Purpose:** Generic data table with sorting
- **Props:** `columns`, `rows`, `sortable`
- **Features:** Sort by column, row hover, sticky header

**Component:** Filter Bar
- **Purpose:** Multi-filter selection
- **Props:** `filters` (array of filter configs)
- **Features:** Dropdown filters, "Clear all" button

---

### 6.2 Chart Library

**Recommendation:** Use Looker Studio built-in charts or Plotly for Streamlit

**Chart Types Needed:**
1. **Line chart** (Trend Analysis)
   - Multi-series (3 retailers)
   - Time-series x-axis
   - Hover tooltips
2. **Table** (Basket Comparison, Mission Explorer)
   - Sortable columns
   - Cell formatting (colors, icons)
3. **Scorecard** (Week Summary)
   - Large number display
   - Comparison indicator

---

## 7. Visual Design System

### 7.1 Color Palette

**Brand Colors:**
- Primary green: `#00A862` (Woolworths green)
- Primary red: `#E31837` (Woolworths red)
- Primary orange: `#F58220` (Woolworths orange)

**Functional Colors:**
- ✅ Success (cheaper): `#00A862`
- ❌ Error (more expensive): `#E31837`
- ⚠️ Warning: `#FFA726`
- ℹ️ Info: `#2196F3`

**Neutral Colors:**
- Text primary: `#333333`
- Text secondary: `#666666`
- Border: `#E0E0E0`
- Background: `#FFFFFF`
- Background secondary: `#F5F5F5`

**Promotion Colors:**
- 🔴 Red EDLP: `#E31837`
- 🟡 Yellow Ticket: `#FFD700`
- 🟠 Half Price: `#F58220`

---

### 7.2 Typography

**Font Family:**
- Primary: "Woolworths Sans" or Open Sans (fallback)
- Monospace (for prices): "Roboto Mono" or Courier New

**Font Sizes:**
- H1 (Page title): 28px bold
- H2 (Section title): 24px bold
- H3 (Subsection): 20px bold
- Body: 14px regular
- Small: 12px regular
- Price (large): 18px bold

**Line Heights:**
- Headers: 1.2
- Body: 1.5

---

### 7.3 Spacing

**Base unit:** 8px

**Spacing Scale:**
- xs: 4px
- sm: 8px
- md: 16px
- lg: 24px
- xl: 32px
- xxl: 48px

**Component Spacing:**
- Card padding: 24px
- Table cell padding: 12px 16px
- Button padding: 10px 20px
- Section margin: 32px vertical

---

### 7.4 Layout Grid

**Container Width:**
- Max width: 1200px
- Padding: 24px horizontal

**Columns:**
- Dashboard: 1 column (full width)
- Basket Comparison: 4 columns (Ingredient | WW | Coles | Aldi)
- Cards: 3 columns (WW | Coles | Aldi)

---

### 7.5 Icons

**Icon Library:** Material Icons or Feather Icons

**Icons Needed:**
- ✅ Checkmark (cheaper)
- ❌ Cross (more expensive)
- 🔄 Refresh
- 📊 Chart/Graph
- 📄 Document/Export
- 🔗 Link
- ← Back arrow
- ▼ Dropdown caret
- 🥩 Meat (category icons - optional)
- 🔴🟡🟠 Promotion indicators

---

## 8. Interactions & States

### 8.1 Button States

**Primary Button:**
- Default: Green background `#00A862`, white text
- Hover: Darker green `#008850`
- Active: Even darker `#006F40`
- Disabled: Gray `#CCCCCC`

**Secondary Button:**
- Default: White background, green border
- Hover: Light green background `#E8F5E9`
- Active: Slightly darker green background
- Disabled: Gray border

**Link:**
- Default: Blue `#2196F3` underlined
- Hover: Darker blue `#1976D2`
- Visited: Purple `#9C27B0`

---

### 8.2 Loading States

**Page Load:**
- Show skeleton screens (gray placeholder boxes)
- Spinner in center for initial load

**Data Refresh:**
- Show spinner next to "Last updated" timestamp
- Disable filters during refresh

**Export:**
- Show "Downloading..." spinner on button
- Replace with "Downloaded!" checkmark for 2 seconds

---

### 8.3 Error States

**Data Load Error:**
- Show error message: "Unable to load data. Please try again."
- Retry button

**No Data:**
- Show empty state: "No shops found for this week."
- Suggest action: "Run shops for this week"

**Export Error:**
- Show error toast: "Export failed. Please try again."

---

### 8.4 Responsive Behavior

**Desktop (>1024px):**
- Full layout as designed
- Multi-column grids

**Tablet (768-1024px):**
- Reduce max width to 100%
- Stack some columns

**Mobile (<768px):**
- NOT REQUIRED (stakeholders use desktop)
- If needed: Stack all columns, horizontal scroll tables

---

## 9. Acceptance Criteria

### 9.1 Functional Requirements

**Must Have:**
- ✅ Display 120 shops (8 missions × 5 personas × 3 retailers)
- ✅ Item-by-item basket comparison matching Feb 2026 PDF format
- ✅ Price comparison ($ and %) vs Woolworths
- ✅ Promotion indicators (Red EDLP, Yellow Ticket, Half Price)
- ✅ Primary cost drivers section
- ✅ Export to Google Sheets
- ✅ Filter by mission, persona, week
- ✅ Historical trend view (8 weeks)
- ✅ Product detail modal with agent reasoning

**Should Have:**
- Persona comparison view
- Share link functionality
- Decision quality validation
- Top price movers

**Nice to Have:**
- Item-level trend charts
- Custom date range
- Search functionality

---

### 9.2 Performance Requirements

**Page Load:**
- Executive Dashboard: <2 seconds
- Basket Comparison: <1 second
- Trend Analysis: <3 seconds (chart rendering)

**Data Refresh:**
- BigQuery query: <5 seconds
- UI update: <500ms

**Export:**
- Generate Google Sheets: <10 seconds

---

### 9.3 Browser Compatibility

**Supported Browsers:**
- Chrome 90+ (primary)
- Firefox 88+
- Safari 14+
- Edge 90+

**Not Supported:**
- Internet Explorer (deprecated)

---

### 9.4 Data Quality

**Validation:**
- All prices displayed to 2 decimal places
- Percentages to 1 decimal place
- Handle missing data gracefully (show "N/A")
- Handle zero values (show "$0.00", not blank)

---

## 10. Non-Functional Requirements

### 10.1 Security

**Authentication:**
- Google Workspace SSO (Looker Studio native)
- OR Basic auth (Streamlit deployment)

**Authorization:**
- Read-only access to BigQuery
- No user can modify shop data from frontend

**Data Privacy:**
- No PII displayed
- No customer data (synthetic personas only)

---

### 10.2 Accessibility

**Level:** WCAG 2.1 Level A (minimum)

**Requirements:**
- Color contrast ratio ≥4.5:1
- Keyboard navigation
- Screen reader support for tables
- Alt text for icons

---

### 10.3 Monitoring

**Logging:**
- User interactions (page views, filters applied)
- Export usage
- Error tracking

**Analytics:**
- Google Analytics or BigQuery logs
- Track: Most viewed missions, most used filters, export frequency

---

### 10.4 Deployment

**Looker Studio:**
- Publish to Google Cloud
- Share link with stakeholders
- No CI/CD needed

**Streamlit (if used):**
- Deploy to Cloud Run
- Docker container
- Environment variables for BigQuery connection
- CI/CD: GitHub Actions → Cloud Build → Cloud Run

---

## 11. Handoff Checklist

### 11.1 Assets Provided

- ✅ This specification document
- ✅ Reference PDF (Feb 2026 basket comparison)
- ✅ Persona descriptions PDF
- ✅ BigQuery schema documentation (Section 3)
- ✅ Sample data queries (in each section)

### 11.2 Access Required

**GCP:**
- BigQuery read access to `mystery_shopper` dataset
- Looker Studio access (if using)
- Cloud Run deploy permissions (if Streamlit)

**Credentials:**
- Service account JSON key for BigQuery
- OR Google Workspace account for Looker Studio

### 11.3 Development Steps

1. **Set up BigQuery connection** (Day 1)
   - Test sample queries
   - Verify data availability

2. **Build Executive Dashboard** (Days 2-3)
   - Week summary cards
   - Mission Explorer table
   - Filters

3. **Build Basket Comparison** (Days 4-5)
   - Item-by-item table
   - Cost drivers section
   - Export functionality

4. **Build Product Detail Modal** (Day 6)
   - Agent reasoning display
   - Competitor comparison

5. **Build Trend Analysis** (Days 7-8)
   - Line chart
   - Top price movers

6. **Build Persona Comparison** (Day 9-10)
   - Persona table
   - Decision quality validation

7. **Testing & Polish** (Days 11-12)
   - Cross-browser testing
   - Performance optimization
   - Stakeholder UAT

8. **Deployment** (Day 13)
   - Publish to production
   - Share with stakeholders
   - Handoff documentation

---

## 12. Contact & Support

**Project Lead:** Alexa Kelly  
**Technical Lead:** Daria Volkova (backend/data)  
**Frontend Developer:** [Your name]

**Questions:**
- Data schema: Daria
- Business requirements: Alexa
- BigQuery access: GCP admin

**Timeline:**
- **Start:** Week 3 (May 12)
- **UAT:** Week 4 Day 2 (May 20)
- **Go-live:** Week 4 Day 3 (May 21)

---

## Appendix A: Sample Data

### A.1 Sample shop_results Row

```json
{
  "shop_id": "2026-05-16_spag_saver_ww",
  "date_executed": "2026-05-16",
  "week_label": "Week of May 16, 2026",
  "mission_name": "Spaghetti Bolognese",
  "persona": "Saver",
  "list_item": "Lean mince beef, 1kg",
  "item_order": 1,
  "retailer": "Woolworths",
  "product_id": "WW123456",
  "product_name": "Woolworths Own Brand Lean Beef Mince",
  "brand": "Woolworths Own",
  "pack_size": "1kg",
  "pack_size_normalized": 1000.0,
  "price": 12.00,
  "unit_price": 12.00,
  "promotion_type": "red_edlp",
  "promotion_description": "Red EDLP",
  "promotion_flag": true,
  "product_url": "https://woolworths.com.au/shop/productdetails/123456",
  "agent_reasoning": "Selected Woolworths own brand lean mince for $12.00/kg as it meets the minimum 1kg pack size requirement and is on Red EDLP promotion. While Coles has a yellow ticket at $10.00, this WW option provides consistent value for weekly shopping.",
  "confidence_score": 0.92,
  "alternative_product_id": "COLES789",
  "quality_markers": null,
  "own_brand_flag": true,
  "status": "found"
}
```

---

### A.2 Sample basket_summaries Row

```json
{
  "shop_id": "2026-05-16_spag_saver_ww",
  "date_executed": "2026-05-16",
  "week_label": "Week of May 16, 2026",
  "mission_name": "Spaghetti Bolognese",
  "persona": "Saver",
  "retailer": "Coles",
  "total_basket_price": 60.92,
  "num_items": 10,
  "num_items_found": 10,
  "coverage_pct": 100.0,
  "num_promoted_items": 3,
  "num_own_brand_items": 5,
  "woolworths_basket_price": 63.90,
  "price_vs_woolworths_dollar": -2.98,
  "price_vs_woolworths_pct": -4.5,
  "is_woolworths_cheaper": false,
  "execution_time_seconds": 120.5
}
```

---

**END OF SPECIFICATION**

**Version History:**
- v1.0 (May 1, 2026): Initial specification
