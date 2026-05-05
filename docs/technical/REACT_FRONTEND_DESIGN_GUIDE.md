# Mystery Shopping Agent - React Frontend Design Guide
**Version:** 1.0  
**Date:** May 1, 2026  
**Author:** Alexa Kelly  
**Stack:** React 19 + TypeScript + Tailwind + ECharts

---

## Table of Contents
1. [Technical Stack Integration](#technical-stack-integration)
2. [Design System Implementation](#design-system-implementation)
3. [Component Architecture](#component-architecture)
4. [Screen Components](#screen-components)
5. [Data Layer](#data-layer)
6. [Styling Patterns](#styling-patterns)
7. [Testing Strategy](#testing-strategy)

---

## 1. Project Context & Requirements

### 1.1 Business Context

**Purpose:** AI-powered automated mystery shopping system comparing basket prices across Woolworths, Coles, and Aldi from a customer perspective.

**Key Business Requirements:**
- ✅ Show basket-by-basket comparison (matches manual Feb 2026 baseline format)
- ✅ Identify WHY price differences exist (pack sizes, promotions, product selection)
- ✅ Enable persona-based insights (how different customer segments experience pricing)
- ✅ Track trends over time (weekly price changes)
- ✅ Export to Google Sheets for presentations

**Users:**
- **Primary:** Owen Lim (CEO), Commercial team
- **Secondary:** Pricing team, CustomerX team
- **Use case:** Weekly review of competitive pricing (5-10 min sessions, Friday afternoons)

**Scope:** 120 automated mystery shops per week (8 missions × 5 personas × 3 retailers)

### 1.2 User Flows

**Primary Flow: Weekly Review (5-10 minutes)**
```
1. Land on Executive Dashboard
   ↓
2. Review week summary (total across all shops)
   ↓
3. Identify missions where WW is more expensive
   ↓
4. Drill into specific mission (e.g., "Spaghetti Bolognese")
   ↓
5. View basket comparison for Saver persona
   ↓
6. Examine item-by-item differences
   ↓
7. Check "Primary Cost Drivers" section
   ↓
8. Click product to see why it was selected
   ↓
9. Export basket to Google Sheets
   ↓
10. Repeat for other missions/personas
```

**Secondary Flow: Trend Analysis (3-5 minutes, monthly)**
```
1. Navigate to "Trend Analysis" tab
   ↓
2. Select mission + persona
   ↓
3. View 8-week price trend chart
   ↓
4. Identify week where prices spiked
   ↓
5. Review "Top Price Movers" section
   ↓
6. Note insights for pricing strategy
```

**Secondary Flow: Persona Comparison (3-5 minutes, monthly)**
```
1. Navigate to "Persona Insights" tab
   ↓
2. Select mission
   ↓
3. View how all 5 personas shop the same basket
   ↓
4. Identify which personas find WW more expensive
   ↓
5. Review different product selections per persona
```

---

## 2. Technical Stack Integration

### 1.1 Project Setup

**Existing Stack Compatibility:**
- ✅ React 19.1.0 + TypeScript 5.8.3
- ✅ Tailwind CSS 3.4.13 (matches existing design system)
- ✅ ECharts 6.0.0 for data visualization
- ✅ Radix UI for accessible components
- ✅ React Query 5.84.2 for BigQuery data fetching
- ✅ Zustand 5.0.6 for local UI state

**New Dependencies Needed:**
```json
{
  "dependencies": {
    "@google-cloud/bigquery": "^7.0.0",  // Backend API integration
    "react-table": "^8.0.0",              // Data tables (or @tanstack/react-table)
    // All other dependencies already in your stack
  }
}
```

---

### 1.2 Directory Structure

```
src/
├── features/
│   └── mystery-shopper/
│       ├── components/
│       │   ├── ExecutiveDashboard.tsx
│       │   ├── BasketComparison.tsx
│       │   ├── ProductDetailModal.tsx
│       │   ├── TrendAnalysis.tsx
│       │   ├── PersonaComparison.tsx
│       │   └── shared/
│       │       ├── PriceBadge.tsx
│       │       ├── PromotionIndicator.tsx
│       │       ├── MissionCard.tsx
│       │       ├── DataTable.tsx
│       │       └── FilterBar.tsx
│       ├── hooks/
│       │   ├── useShopData.ts
│       │   ├── useBasketSummaries.ts
│       │   ├── useCostDrivers.ts
│       │   └── useTrendData.ts
│       ├── services/
│       │   └── bigquery.ts
│       ├── types/
│       │   ├── shop.types.ts
│       │   └── basket.types.ts
│       └── store/
│           └── mystery-shopper.store.ts
├── lib/
│   └── bigquery-client.ts
└── styles/
    └── mystery-shopper.css  // Additional styles if needed
```

---

### 1.3 Navigation Integration

**Add to existing tab navigation (Zustand store):**

```typescript
// src/store/navigation.store.ts (existing)
interface NavigationState {
  currentPage: 'overview' | 'deep-dive' | 'ask-ai' | 'alerts' | 
               'developer' | 'mystery-shopper';  // Add this
  setCurrentPage: (page: NavigationState['currentPage']) => void;
}

// In your main navigation component, add:
<NavigationTab
  icon={ShoppingCart}
  label="Mystery Shopper"
  active={currentPage === 'mystery-shopper'}
  onClick={() => setCurrentPage('mystery-shopper')}
/>
```

---

## 2. Design System Implementation

### 2.1 Typography Usage (Your Existing System)

**Apply semantic typography classes:**

```tsx
// Page titles
<h1 className="typography-display">Mystery Shopping Agent</h1>

// Section headers
<h2 className="typography-h2">Week Summary</h2>

// Subsections
<h3 className="typography-h3">Basket Comparison</h3>

// Body text
<p className="typography-body">
  Across all missions/personas, Woolworths is cheaper in 2 of 8 missions
</p>

// Small labels
<span className="typography-label">Last updated: 2 hours ago</span>
```

**Price Display (Monospace):**
```tsx
<span className="font-mono text-xl font-semibold">$63.90</span>
<span className="font-mono text-sm text-muted-foreground">$12.00/kg</span>
```

---

### 2.2 Color System (Your Existing Variables)

**Semantic Colors for Mystery Shopper:**

```tsx
// ✅ Cheaper than Woolworths (use success/positive)
<div className="text-[hsl(var(--success))]">  // Green
  💚 -4.5%
</div>

// ❌ More expensive than Woolworths (use destructive/negative)
<div className="text-[hsl(var(--destructive))]">  // Red
  ❌ +2.1%
</div>

// Promotion indicators (use chart colors)
<div className="text-[hsl(var(--chart-1))]">🔴 Red EDLP</div>      // Red
<div className="text-[hsl(var(--chart-2))]">🟡 Yellow Ticket</div>  // Yellow
<div className="text-[hsl(var(--chart-3))]">🟠 Half Price</div>     // Orange
```

**Define in your Tailwind config if not present:**
```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        success: 'hsl(var(--success))',      // Green for cheaper
        destructive: 'hsl(var(--destructive))', // Red for expensive
        warning: 'hsl(var(--warning))',      // Orange
      }
    }
  }
}
```

**Add to globals.css:**
```css
:root {
  --success: 142 76% 36%;      /* #00A862 Woolworths green */
  --destructive: 0 84% 60%;    /* #E31837 Woolworths red */
  --warning: 25 95% 53%;       /* #F58220 Woolworths orange */
}

.dark {
  --success: 142 76% 36%;
  --destructive: 0 84% 60%;
  --warning: 25 95% 53%;
}
```

---

### 2.3 Glass Morphism Components (Your Existing Pattern)

**Use your existing .glass-card classes:**

```tsx
// Week Summary Cards
<div className="glass-card p-6">
  <div className="typography-h3">Woolworths</div>
  <div className="font-mono text-3xl font-bold mt-2">$449.20</div>
  <div className="typography-body-sm mt-1 text-muted-foreground">Baseline</div>
</div>

// Mission Cards
<div className="glass-card-interactive p-4 cursor-pointer">
  <div className="typography-h4">Spaghetti Bolognese</div>
  {/* ... */}
</div>

// Product Detail Modal
<Dialog>
  <DialogContent className="glass-dialog max-w-2xl">
    {/* Product detail content */}
  </DialogContent>
</Dialog>
```

---

## 3. Component Architecture

### 3.1 TypeScript Types

**Create `src/features/mystery-shopper/types/shop.types.ts`:**

```typescript
// shop.types.ts
export interface ShopResult {
  shop_id: string;
  date_executed: string;
  week_label: string;
  mission_name: string;
  mission_description: string;
  persona: Persona;
  persona_description: string;
  list_item: string;
  item_order: number;
  retailer: Retailer;
  product_id: string;
  product_name: string;
  brand: string;
  pack_size: string;
  pack_size_normalized: number;
  price: number;
  unit_price: number;
  promotion_type: PromotionType;
  promotion_description: string;
  promotion_flag: boolean;
  product_url: string;
  agent_reasoning: string;
  confidence_score: number;
  alternative_product_id?: string;
  quality_markers?: string[];
  own_brand_flag: boolean;
  status: 'found' | 'not_found';
}

export interface BasketSummary {
  shop_id: string;
  date_executed: string;
  week_label: string;
  mission_name: string;
  persona: Persona;
  retailer: Retailer;
  total_basket_price: number;
  num_items: number;
  num_items_found: number;
  coverage_pct: number;
  num_promoted_items: number;
  num_own_brand_items: number;
  woolworths_basket_price: number;
  price_vs_woolworths_dollar: number;
  price_vs_woolworths_pct: number;
  is_woolworths_cheaper: boolean;
  execution_time_seconds: number;
}

export interface CostDriver {
  shop_id: string;
  date_executed: string;
  mission_name: string;
  persona: Persona;
  retailer_comparison: string;
  driver_type: 'promotion' | 'pack_size' | 'base_price' | 'product_selection';
  driver_description: string;
  item_name: string;
  dollar_impact: number;
  pct_of_total_diff: number;
}

export type Persona = 'Saver' | 'Traditional' | 'Conscious' | 'Refined' | 'Essential';
export type Retailer = 'Woolworths' | 'Coles' | 'Aldi';
export type PromotionType = 'red_edlp' | 'yellow_ticket' | 'half_price' | 'none';

export interface MissionConfig {
  name: string;
  description: string;
  items: string[];
}

export interface PersonaConfig {
  name: Persona;
  description: string;
  behaviors: string[];
}
```

---

### 3.2 React Query Hooks

**Create `src/features/mystery-shopper/hooks/useShopData.ts`:**

```typescript
import { useQuery } from '@tanstack/react-query';
import { bigQueryService } from '../services/bigquery';
import type { ShopResult, BasketSummary, CostDriver } from '../types/shop.types';

export const useShopResults = (
  weekDate: string,
  missionName?: string,
  persona?: string
) => {
  return useQuery({
    queryKey: ['shop-results', weekDate, missionName, persona],
    queryFn: () => bigQueryService.getShopResults(weekDate, missionName, persona),
    staleTime: 1000 * 60 * 5, // 5 minutes
    gcTime: 1000 * 60 * 30,   // 30 minutes (formerly cacheTime)
  });
};

export const useBasketSummaries = (weekDate: string) => {
  return useQuery<BasketSummary[]>({
    queryKey: ['basket-summaries', weekDate],
    queryFn: () => bigQueryService.getBasketSummaries(weekDate),
    staleTime: 1000 * 60 * 5,
  });
};

export const useCostDrivers = (
  weekDate: string,
  missionName: string,
  persona: string
) => {
  return useQuery<CostDriver[]>({
    queryKey: ['cost-drivers', weekDate, missionName, persona],
    queryFn: () => bigQueryService.getCostDrivers(weekDate, missionName, persona),
    staleTime: 1000 * 60 * 5,
  });
};

export const useWeekSummary = (weekDate: string) => {
  return useQuery({
    queryKey: ['week-summary', weekDate],
    queryFn: () => bigQueryService.getWeekSummary(weekDate),
    staleTime: 1000 * 60 * 5,
  });
};

export const useTrendData = (
  missionName: string,
  persona: string,
  weeksBack: number = 8
) => {
  return useQuery({
    queryKey: ['trend-data', missionName, persona, weeksBack],
    queryFn: () => bigQueryService.getTrendData(missionName, persona, weeksBack),
    staleTime: 1000 * 60 * 10, // 10 minutes for trends
  });
};
```

---

### 3.3 Zustand Store (UI State)

**Create `src/features/mystery-shopper/store/mystery-shopper.store.ts`:**

```typescript
import { create } from 'zustand';
import { persist } from 'zustand/middleware';
import type { Persona, Retailer } from '../types/shop.types';

interface MysteryShopperState {
  // Filters
  selectedWeek: string;
  selectedMission: string | null;
  selectedPersona: Persona | null;
  selectedRetailer: Retailer | null;
  
  // UI State
  expandedMissions: Set<string>;
  activeTab: 'dashboard' | 'basket' | 'trends' | 'personas';
  productDetailOpen: boolean;
  selectedProduct: {
    shopId: string;
    listItem: string;
    retailer: Retailer;
  } | null;
  
  // Actions
  setSelectedWeek: (week: string) => void;
  setSelectedMission: (mission: string | null) => void;
  setSelectedPersona: (persona: Persona | null) => void;
  setSelectedRetailer: (retailer: Retailer | null) => void;
  toggleMissionExpanded: (mission: string) => void;
  setActiveTab: (tab: MysteryShopperState['activeTab']) => void;
  openProductDetail: (product: MysteryShopperState['selectedProduct']) => void;
  closeProductDetail: () => void;
  resetFilters: () => void;
}

export const useMysteryShopperStore = create<MysteryShopperState>()(
  persist(
    (set) => ({
      // Initial state
      selectedWeek: new Date().toISOString().split('T')[0], // Today
      selectedMission: null,
      selectedPersona: null,
      selectedRetailer: null,
      expandedMissions: new Set(['Spaghetti Bolognese', 'School Lunches']), // Default expanded
      activeTab: 'dashboard',
      productDetailOpen: false,
      selectedProduct: null,
      
      // Actions
      setSelectedWeek: (week) => set({ selectedWeek: week }),
      setSelectedMission: (mission) => set({ selectedMission: mission }),
      setSelectedPersona: (persona) => set({ selectedPersona: persona }),
      setSelectedRetailer: (retailer) => set({ selectedRetailer: retailer }),
      
      toggleMissionExpanded: (mission) =>
        set((state) => {
          const newExpanded = new Set(state.expandedMissions);
          if (newExpanded.has(mission)) {
            newExpanded.delete(mission);
          } else {
            newExpanded.add(mission);
          }
          return { expandedMissions: newExpanded };
        }),
      
      setActiveTab: (tab) => set({ activeTab: tab }),
      
      openProductDetail: (product) =>
        set({ productDetailOpen: true, selectedProduct: product }),
      
      closeProductDetail: () =>
        set({ productDetailOpen: false, selectedProduct: null }),
      
      resetFilters: () =>
        set({
          selectedMission: null,
          selectedPersona: null,
          selectedRetailer: null,
        }),
    }),
    {
      name: 'mystery-shopper-storage',
      partialize: (state) => ({
        selectedWeek: state.selectedWeek,
        expandedMissions: Array.from(state.expandedMissions), // Serialize Set
      }),
    }
  )
);
```

---

### 3.4 Export to Google Sheets Functionality

**Export Format Specification:**

**Filename:** `Mystery_Shop_[Mission]_[Persona]_[Week].xlsx`  
**Example:** `Mystery_Shop_Spaghetti_Bolognese_Saver_2026-05-16.xlsx`

**Sheet 1: Basket Comparison**

| Ingredient | Woolworths Product | WW Price | WW Unit Price | WW Promo | Coles Product | Coles Price | Coles Unit Price | Coles Promo | Aldi Product | Aldi Price | Aldi Unit Price | Aldi Promo |
|------------|-------------------|----------|---------------|----------|---------------|-------------|------------------|-------------|--------------|------------|-----------------|------------|
| Lean mince beef, 1kg | WW Own Brand | $12.00 | $12.00/kg | Red EDLP | Coles 3-Star | $10.00 | $10.00/kg | Yellow Ticket | Standard | $11.50 | $11.50/kg | - |
| **TOTAL** | | **$63.90** | | | | **$60.92** | | | | **$57.60** | | |
| **vs WW** | | | | | | **-$2.98 (-4.5%)** | | | | **-$6.30 (-9.9%)** | | |

**Sheet 2: Cost Drivers**

| Retailer | Driver | $ Impact |
|----------|--------|----------|
| Coles | Lean mince beef $2.00 cheaper (Yellow ticket promo) | -$2.00 |
| Aldi | Overall basket: Lower base prices across most items | -$6.30 |

**Implementation:**
```typescript
// In bigQueryService
async exportToSheets(
  weekDate: string,
  missionName: string,
  persona: string
): Promise<{ url: string }> {
  const response = await axios.post<{ url: string }>(
    `${API_BASE}/export`,
    { 
      week: weekDate, 
      mission: missionName, 
      persona,
      format: 'xlsx' // Backend generates Excel file
    }
  );
  return response.data;
}
```

---

### 3.5 BigQuery Service Layer

**Create `src/features/mystery-shopper/services/bigquery.ts`:**

```typescript
import axios from 'axios';
import type { ShopResult, BasketSummary, CostDriver } from '../types/shop.types';

const API_BASE = '/api/mystery-shopper';

class BigQueryService {
  async getShopResults(
    weekDate: string,
    missionName?: string,
    persona?: string
  ): Promise<ShopResult[]> {
    const params = new URLSearchParams();
    params.append('week', weekDate);
    if (missionName) params.append('mission', missionName);
    if (persona) params.append('persona', persona);
    
    const response = await axios.get<ShopResult[]>(
      `${API_BASE}/shop-results?${params}`
    );
    return response.data;
  }
  
  async getBasketSummaries(weekDate: string): Promise<BasketSummary[]> {
    const response = await axios.get<BasketSummary[]>(
      `${API_BASE}/basket-summaries`,
      { params: { week: weekDate } }
    );
    return response.data;
  }
  
  async getCostDrivers(
    weekDate: string,
    missionName: string,
    persona: string
  ): Promise<CostDriver[]> {
    const response = await axios.get<CostDriver[]>(
      `${API_BASE}/cost-drivers`,
      { params: { week: weekDate, mission: missionName, persona } }
    );
    return response.data;
  }
  
  async getWeekSummary(weekDate: string) {
    const response = await axios.get(`${API_BASE}/week-summary`, {
      params: { week: weekDate }
    });
    return response.data;
  }
  
  async getTrendData(missionName: string, persona: string, weeksBack: number) {
    const response = await axios.get(`${API_BASE}/trends`, {
      params: { mission: missionName, persona, weeks: weeksBack }
    });
    return response.data;
  }
  
  async exportToSheets(
    weekDate: string,
    missionName: string,
    persona: string
  ): Promise<{ url: string }> {
    const response = await axios.post<{ url: string }>(
      `${API_BASE}/export`,
      { week: weekDate, mission: missionName, persona }
    );
    return response.data;
  }
}

export const bigQueryService = new BigQueryService();
```

---

## 4. Screen Components

### 4.1 Executive Dashboard Component

**`src/features/mystery-shopper/components/ExecutiveDashboard.tsx`:**

```tsx
import { useMysteryShopperStore } from '../store/mystery-shopper.store';
import { useBasketSummaries, useWeekSummary } from '../hooks/useShopData';
import { WeekSummaryCards } from './WeekSummaryCards';
import { FilterBar } from './shared/FilterBar';
import { MissionExplorer } from './MissionExplorer';
import { Skeleton } from '@/components/ui/skeleton';

export const ExecutiveDashboard = () => {
  const { selectedWeek } = useMysteryShopperStore();
  
  const { data: weekSummary, isLoading: summaryLoading } = useWeekSummary(selectedWeek);
  const { data: baskets, isLoading: basketsLoading } = useBasketSummaries(selectedWeek);
  
  if (summaryLoading || basketsLoading) {
    return <DashboardSkeleton />;
  }
  
  return (
    <div className="space-y-8">
      {/* Header */}
      <div className="flex items-center justify-between">
        <div>
          <h1 className="typography-display">Mystery Shopping Agent</h1>
          <p className="typography-body-sm mt-2">
            Last updated: {new Date().toLocaleTimeString()}
          </p>
        </div>
        <WeekSelector />
      </div>
      
      {/* Week Summary Cards */}
      <WeekSummaryCards data={weekSummary} />
      
      {/* Summary Stats */}
      <div className="glass-card p-6">
        <p className="typography-body">
          Across all missions/personas, Woolworths is:
        </p>
        <ul className="mt-3 space-y-2">
          <li className="typography-body flex items-center gap-2">
            <span className="text-success">💚</span>
            Cheaper in {weekSummary?.ww_wins} of 8 missions ({Math.round(weekSummary?.ww_wins / 8 * 100)}%)
          </li>
          <li className="typography-body flex items-center gap-2">
            <span className="text-destructive">❌</span>
            More expensive in {8 - weekSummary?.ww_wins} of 8 missions ({Math.round((8 - weekSummary?.ww_wins) / 8 * 100)}%)
          </li>
        </ul>
      </div>
      
      {/* Filters */}
      <FilterBar />
      
      {/* Mission Explorer */}
      <div>
        <h2 className="typography-h2 mb-4">🎯 Mission Explorer</h2>
        <MissionExplorer baskets={baskets || []} />
      </div>
    </div>
  );
};

const DashboardSkeleton = () => (
  <div className="space-y-8">
    <Skeleton className="h-16 w-96" />
    <div className="grid grid-cols-3 gap-6">
      <Skeleton className="h-32" />
      <Skeleton className="h-32" />
      <Skeleton className="h-32" />
    </div>
    <Skeleton className="h-64" />
  </div>
);
```

---

### 4.2 Week Summary Cards Component

**`src/features/mystery-shopper/components/WeekSummaryCards.tsx`:**

```tsx
import { Retailer } from '../types/shop.types';

interface WeekSummaryData {
  woolworths: number;
  coles: { total: number; vs_ww_dollar: number; vs_ww_pct: number };
  aldi: { total: number; vs_ww_dollar: number; vs_ww_pct: number };
}

export const WeekSummaryCards = ({ data }: { data: WeekSummaryData }) => {
  return (
    <div className="grid grid-cols-3 gap-6">
      {/* Woolworths (Baseline) */}
      <div className="glass-card p-6">
        <div className="typography-h3 text-muted-foreground">Woolworths</div>
        <div className="font-mono text-4xl font-bold mt-3">
          ${data.woolworths.toFixed(2)}
        </div>
        <div className="typography-body-sm mt-2 text-muted-foreground">
          Baseline
        </div>
      </div>
      
      {/* Coles */}
      <div className="glass-card p-6">
        <div className="typography-h3 text-muted-foreground">Coles</div>
        <div className="font-mono text-4xl font-bold mt-3">
          ${data.coles.total.toFixed(2)}
        </div>
        <div className="typography-body-sm mt-2 flex items-center gap-2">
          {data.coles.vs_ww_pct < 0 ? (
            <>
              <span className="text-success">✅</span>
              <span className="text-success font-medium">
                {data.coles.vs_ww_pct.toFixed(1)}%
              </span>
              <span className="text-muted-foreground">
                (${Math.abs(data.coles.vs_ww_dollar).toFixed(2)})
              </span>
            </>
          ) : (
            <>
              <span className="text-destructive">❌</span>
              <span className="text-destructive font-medium">
                +{data.coles.vs_ww_pct.toFixed(1)}%
              </span>
              <span className="text-muted-foreground">
                (+${data.coles.vs_ww_dollar.toFixed(2)})
              </span>
            </>
          )}
        </div>
      </div>
      
      {/* Aldi */}
      <div className="glass-card p-6">
        <div className="typography-h3 text-muted-foreground">Aldi</div>
        <div className="font-mono text-4xl font-bold mt-3">
          ${data.aldi.total.toFixed(2)}
        </div>
        <div className="typography-body-sm mt-2 flex items-center gap-2">
          {data.aldi.vs_ww_pct < 0 ? (
            <>
              <span className="text-success">✅</span>
              <span className="text-success font-medium">
                {data.aldi.vs_ww_pct.toFixed(1)}%
              </span>
              <span className="text-muted-foreground">
                (${Math.abs(data.aldi.vs_ww_dollar).toFixed(2)})
              </span>
            </>
          ) : (
            <>
              <span className="text-destructive">❌</span>
              <span className="text-destructive font-medium">
                +{data.aldi.vs_ww_pct.toFixed(1)}%
              </span>
              <span className="text-muted-foreground">
                (+${data.aldi.vs_ww_dollar.toFixed(2)})
              </span>
            </>
          )}
        </div>
      </div>
    </div>
  );
};
```

---

### 4.3 Mission Explorer Component

**`src/features/mystery-shopper/components/MissionExplorer.tsx`:**

```tsx
import { useMysteryShopperStore } from '../store/mystery-shopper.store';
import { MissionCard } from './shared/MissionCard';
import type { BasketSummary } from '../types/shop.types';

interface MissionExplorerProps {
  baskets: BasketSummary[];
}

export const MissionExplorer = ({ baskets }: MissionExplorerProps) => {
  const { expandedMissions, toggleMissionExpanded } = useMysteryShopperStore();
  
  // Group baskets by mission
  const missionGroups = baskets.reduce((acc, basket) => {
    if (!acc[basket.mission_name]) {
      acc[basket.mission_name] = [];
    }
    acc[basket.mission_name].push(basket);
    return acc;
  }, {} as Record<string, BasketSummary[]>);
  
  return (
    <div className="space-y-4">
      {Object.entries(missionGroups).map(([missionName, baskets]) => (
        <MissionCard
          key={missionName}
          missionName={missionName}
          baskets={baskets}
          isExpanded={expandedMissions.has(missionName)}
          onToggle={() => toggleMissionExpanded(missionName)}
        />
      ))}
    </div>
  );
};
```

---

### 4.4 Mission Card Component (Collapsible)

**`src/features/mystery-shopper/components/shared/MissionCard.tsx`:**

```tsx
import { ChevronDown, ChevronRight, ExternalLink } from 'lucide-react';
import { useMysteryShopperStore } from '../../store/mystery-shopper.store';
import { PriceBadge } from './PriceBadge';
import type { BasketSummary, Persona } from '../../types/shop.types';

interface MissionCardProps {
  missionName: string;
  baskets: BasketSummary[];
  isExpanded: boolean;
  onToggle: () => void;
}

export const MissionCard = ({
  missionName,
  baskets,
  isExpanded,
  onToggle,
}: MissionCardProps) => {
  const { setActiveTab, setSelectedMission, setSelectedPersona } = useMysteryShopperStore();
  
  // Group baskets by persona and retailer
  const basketsByPersona = baskets.reduce((acc, basket) => {
    if (!acc[basket.persona]) {
      acc[basket.persona] = {};
    }
    acc[basket.persona][basket.retailer] = basket;
    return acc;
  }, {} as Record<Persona, Record<string, BasketSummary>>);
  
  const handleViewDetails = (persona: Persona) => {
    setSelectedMission(missionName);
    setSelectedPersona(persona);
    setActiveTab('basket');
  };
  
  return (
    <div className="glass-card-interactive overflow-hidden">
      {/* Header - Clickable to expand/collapse */}
      <button
        onClick={onToggle}
        className="w-full flex items-center justify-between p-5 hover:bg-accent/50 transition-colors"
      >
        <div className="flex items-center gap-3">
          {isExpanded ? (
            <ChevronDown className="h-5 w-5 text-muted-foreground" />
          ) : (
            <ChevronRight className="h-5 w-5 text-muted-foreground" />
          )}
          <h3 className="typography-h4">Mission: {missionName}</h3>
        </div>
      </button>
      
      {/* Expandable Content */}
      {isExpanded && (
        <div className="border-t border-border/60 p-5">
          <div className="overflow-x-auto">
            <table className="w-full">
              <thead>
                <tr className="border-b border-border/60">
                  <th className="typography-label text-left pb-3 pr-4">Persona</th>
                  <th className="typography-label text-right pb-3 px-4">Woolworths</th>
                  <th className="typography-label text-right pb-3 px-4">Coles</th>
                  <th className="typography-label text-right pb-3 px-4">Aldi</th>
                  <th className="w-32 pb-3"></th>
                </tr>
              </thead>
              <tbody>
                {Object.entries(basketsByPersona).map(([persona, retailers]) => (
                  <tr key={persona} className="border-b border-border/30 last:border-0">
                    <td className="typography-body py-4 pr-4">{persona}</td>
                    <td className="py-4 px-4">
                      <PriceBadge
                        price={retailers['Woolworths']?.total_basket_price || 0}
                        isBaseline
                      />
                    </td>
                    <td className="py-4 px-4">
                      <PriceBadge
                        price={retailers['Coles']?.total_basket_price || 0}
                        vsWoolworthsPct={retailers['Coles']?.price_vs_woolworths_pct || 0}
                      />
                    </td>
                    <td className="py-4 px-4">
                      <PriceBadge
                        price={retailers['Aldi']?.total_basket_price || 0}
                        vsWoolworthsPct={retailers['Aldi']?.price_vs_woolworths_pct || 0}
                      />
                    </td>
                    <td className="py-4 text-right">
                      <button
                        onClick={() => handleViewDetails(persona as Persona)}
                        className="typography-body-sm text-primary hover:text-primary/80 
                                   flex items-center gap-1 transition-colors"
                      >
                        View
                        <ExternalLink className="h-3 w-3" />
                      </button>
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      )}
    </div>
  );
};
```

---

### 4.5 Price Badge Component

**`src/features/mystery-shopper/components/shared/PriceBadge.tsx`:**

```tsx
interface PriceBadgeProps {
  price: number;
  vsWoolworthsPct?: number;
  isBaseline?: boolean;
}

export const PriceBadge = ({ price, vsWoolworthsPct, isBaseline }: PriceBadgeProps) => {
  const isCheaper = vsWoolworthsPct && vsWoolworthsPct < 0;
  
  return (
    <div className="text-right">
      <div className="font-mono text-base font-semibold">
        ${price.toFixed(2)}
      </div>
      {isBaseline ? (
        <div className="typography-body-sm text-muted-foreground mt-0.5">
          Baseline
        </div>
      ) : vsWoolworthsPct !== undefined ? (
        <div className="typography-body-sm mt-0.5 flex items-center justify-end gap-1">
          {isCheaper ? (
            <>
              <span className="text-success">✅</span>
              <span className="text-success font-medium">
                {vsWoolworthsPct.toFixed(1)}%
              </span>
            </>
          ) : (
            <>
              <span className="text-destructive">❌</span>
              <span className="text-destructive font-medium">
                +{vsWoolworthsPct.toFixed(1)}%
              </span>
            </>
          )}
        </div>
      ) : null}
    </div>
  );
};
```

---

### 4.6 Promotion Indicator Component

**`src/features/mystery-shopper/components/shared/PromotionIndicator.tsx`:**

```tsx
import type { PromotionType } from '../../types/shop.types';

interface PromotionIndicatorProps {
  type: PromotionType;
  description?: string;
}

const PROMOTION_STYLES: Record<PromotionType, {
  icon: string;
  color: string;
  label: string;
}> = {
  red_edlp: {
    icon: '🔴',
    color: 'text-[hsl(var(--chart-1))]',
    label: 'Red EDLP',
  },
  yellow_ticket: {
    icon: '🟡',
    color: 'text-[hsl(var(--chart-2))]',
    label: 'Yellow Ticket',
  },
  half_price: {
    icon: '🟠',
    color: 'text-[hsl(var(--chart-3))]',
    label: 'Half Price',
  },
  none: {
    icon: '',
    color: '',
    label: '',
  },
};

export const PromotionIndicator = ({ type, description }: PromotionIndicatorProps) => {
  if (type === 'none') return null;
  
  const style = PROMOTION_STYLES[type];
  
  return (
    <div className="inline-flex items-center gap-1.5">
      <span className={style.color}>{style.icon}</span>
      <span className="typography-body-sm">
        {description || style.label}
      </span>
    </div>
  );
};
```

---

### 4.7 Basket Comparison Component

**`src/features/mystery-shopper/components/BasketComparison.tsx`:**

```tsx
import { useShopResults, useCostDrivers } from '../hooks/useShopData';
import { useMysteryShopperStore } from '../store/mystery-shopper.store';
import { ArrowLeft, Download, Share2 } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { BasketComparisonTable } from './BasketComparisonTable';
import { CostDriversSection } from './CostDriversSection';
import { bigQueryService } from '../services/bigquery';
import { useToast } from '@/components/ui/use-toast';

export const BasketComparison = () => {
  const {
    selectedWeek,
    selectedMission,
    selectedPersona,
    setActiveTab,
  } = useMysteryShopperStore();
  
  const { toast } = useToast();
  
  const { data: shopResults, isLoading } = useShopResults(
    selectedWeek,
    selectedMission!,
    selectedPersona!
  );
  
  const { data: costDrivers } = useCostDrivers(
    selectedWeek,
    selectedMission!,
    selectedPersona!
  );
  
  const handleExport = async () => {
    try {
      const { url } = await bigQueryService.exportToSheets(
        selectedWeek,
        selectedMission!,
        selectedPersona!
      );
      
      toast({
        title: 'Export successful',
        description: 'Opening Google Sheets...',
      });
      
      window.open(url, '_blank');
    } catch (error) {
      toast({
        title: 'Export failed',
        description: 'Please try again',
        variant: 'destructive',
      });
    }
  };
  
  const handleShare = () => {
    const url = `${window.location.origin}?week=${selectedWeek}&mission=${selectedMission}&persona=${selectedPersona}`;
    navigator.clipboard.writeText(url);
    
    toast({
      title: 'Link copied',
      description: 'Share link copied to clipboard',
    });
  };
  
  if (isLoading) {
    return <div>Loading...</div>;
  }
  
  return (
    <div className="space-y-8">
      {/* Header */}
      <div className="flex items-start justify-between">
        <div className="space-y-2">
          <button
            onClick={() => setActiveTab('dashboard')}
            className="flex items-center gap-2 typography-body-sm text-muted-foreground 
                       hover:text-foreground transition-colors"
          >
            <ArrowLeft className="h-4 w-4" />
            Back to Dashboard
          </button>
          
          <div className="flex items-center gap-4">
            <h1 className="typography-h1">Mission: {selectedMission}</h1>
            <span className="typography-body text-muted-foreground">|</span>
            <span className="typography-body">Persona: {selectedPersona}</span>
            <span className="typography-body text-muted-foreground">|</span>
            <span className="typography-body-sm text-muted-foreground">
              Week: {new Date(selectedWeek).toLocaleDateString()}
            </span>
          </div>
          
          <p className="typography-body-sm text-muted-foreground max-w-2xl">
            "{getPersonaDescription(selectedPersona!)}"
          </p>
        </div>
        
        <div className="flex gap-3">
          <Button variant="outline" size="sm" onClick={handleExport}>
            <Download className="h-4 w-4 mr-2" />
            Export to Sheets
          </Button>
          <Button variant="outline" size="sm" onClick={handleShare}>
            <Share2 className="h-4 w-4 mr-2" />
            Share Link
          </Button>
        </div>
      </div>
      
      {/* Basket Comparison Table */}
      <div>
        <h2 className="typography-h2 mb-4">Basket Comparison</h2>
        <BasketComparisonTable shopResults={shopResults || []} />
      </div>
      
      {/* Cost Drivers */}
      <div>
        <h2 className="typography-h2 mb-4">📌 Primary Cost Drivers</h2>
        <CostDriversSection costDrivers={costDrivers || []} />
      </div>
    </div>
  );
};

const getPersonaDescription = (persona: string): string => {
  const descriptions: Record<string, string> = {
    Saver: 'Seek weekly specials, shop by unit prices, will consider own brand for lower price',
    Traditional: 'Stick to tried & tested brands, consistency gives comfort',
    Conscious: 'Conscious of ingredients, environmental impact, and brand values',
    Refined: 'Prioritize quality over price, seek best ingredients',
    Essential: 'Limited means, carefully manage budgets, focus on essentials',
  };
  return descriptions[persona] || '';
};
```

---

### 4.8 Basket Comparison Table

**`src/features/mystery-shopper/components/BasketComparisonTable.tsx`:**

```tsx
import { useMysteryShopperStore } from '../store/mystery-shopper.store';
import { PromotionIndicator } from './shared/PromotionIndicator';
import type { ShopResult } from '../types/shop.types';
import { ExternalLink } from 'lucide-react';

interface BasketComparisonTableProps {
  shopResults: ShopResult[];
}

export const BasketComparisonTable = ({ shopResults }: BasketComparisonTableProps) => {
  const { openProductDetail } = useMysteryShopperStore();
  
  // Group by item
  const itemGroups = shopResults.reduce((acc, result) => {
    if (!acc[result.list_item]) {
      acc[result.list_item] = [];
    }
    acc[result.list_item].push(result);
    return acc;
  }, {} as Record<string, ShopResult[]>);
  
  // Calculate totals
  const totals = {
    Woolworths: shopResults.filter(r => r.retailer === 'Woolworths')
      .reduce((sum, r) => sum + r.price, 0),
    Coles: shopResults.filter(r => r.retailer === 'Coles')
      .reduce((sum, r) => sum + r.price, 0),
    Aldi: shopResults.filter(r => r.retailer === 'Aldi')
      .reduce((sum, r) => sum + r.price, 0),
  };
  
  const colesDiff = totals.Coles - totals.Woolworths;
  const aldiDiff = totals.Aldi - totals.Woolworths;
  
  return (
    <div className="glass-card overflow-hidden">
      <div className="overflow-x-auto">
        <table className="w-full">
          <thead className="bg-muted/50">
            <tr className="border-b border-border">
              <th className="typography-label text-left p-4 w-48">Ingredient</th>
              <th className="typography-label text-left p-4">Woolworths</th>
              <th className="typography-label text-left p-4">Coles</th>
              <th className="typography-label text-left p-4">Aldi</th>
            </tr>
          </thead>
          <tbody>
            {Object.entries(itemGroups).map(([itemName, items], idx) => {
              const wwItem = items.find(i => i.retailer === 'Woolworths');
              const colesItem = items.find(i => i.retailer === 'Coles');
              const aldiItem = items.find(i => i.retailer === 'Aldi');
              
              // Find cheapest price
              const prices = items.map(i => i.price);
              const minPrice = Math.min(...prices);
              
              return (
                <tr
                  key={itemName}
                  className={`border-b border-border/30 ${
                    idx % 2 === 0 ? 'bg-background' : 'bg-muted/20'
                  }`}
                >
                  {/* Ingredient */}
                  <td className="p-4">
                    <div className="typography-body font-medium">{itemName}</div>
                  </td>
                  
                  {/* Woolworths */}
                  <td className="p-4">
                    {wwItem && (
                      <ProductCell
                        item={wwItem}
                        isCheapest={wwItem.price === minPrice}
                        onViewDetails={() => openProductDetail({
                          shopId: wwItem.shop_id,
                          listItem: itemName,
                          retailer: 'Woolworths',
                        })}
                      />
                    )}
                  </td>
                  
                  {/* Coles */}
                  <td className="p-4">
                    {colesItem && (
                      <ProductCell
                        item={colesItem}
                        isCheapest={colesItem.price === minPrice}
                        onViewDetails={() => openProductDetail({
                          shopId: colesItem.shop_id,
                          listItem: itemName,
                          retailer: 'Coles',
                        })}
                      />
                    )}
                  </td>
                  
                  {/* Aldi */}
                  <td className="p-4">
                    {aldiItem && (
                      <ProductCell
                        item={aldiItem}
                        isCheapest={aldiItem.price === minPrice}
                        onViewDetails={() => openProductDetail({
                          shopId: aldiItem.shop_id,
                          listItem: itemName,
                          retailer: 'Aldi',
                        })}
                      />
                    )}
                  </td>
                </tr>
              );
            })}
            
            {/* Totals Row */}
            <tr className="bg-success/10 font-semibold">
              <td className="p-4 typography-body font-bold">TOTAL BASKET</td>
              <td className="p-4">
                <div className="font-mono text-lg font-bold">
                  ${totals.Woolworths.toFixed(2)}
                </div>
                <div className="typography-body-sm text-muted-foreground">
                  Baseline
                </div>
              </td>
              <td className="p-4">
                <div className="font-mono text-lg font-bold">
                  ${totals.Coles.toFixed(2)}
                  {colesDiff < 0 && <span className="text-success ml-2">💚</span>}
                </div>
                <div className={`typography-body-sm ${
                  colesDiff < 0 ? 'text-success' : 'text-destructive'
                }`}>
                  {colesDiff < 0 ? '' : '+'}{colesDiff.toFixed(2)} 
                  ({(colesDiff / totals.Woolworths * 100).toFixed(1)}%)
                </div>
              </td>
              <td className="p-4">
                <div className="font-mono text-lg font-bold">
                  ${totals.Aldi.toFixed(2)}
                  {aldiDiff < 0 && <span className="text-success ml-2">💚</span>}
                </div>
                <div className={`typography-body-sm ${
                  aldiDiff < 0 ? 'text-success' : 'text-destructive'
                }`}>
                  {aldiDiff < 0 ? '' : '+'}{aldiDiff.toFixed(2)} 
                  ({(aldiDiff / totals.Woolworths * 100).toFixed(1)}%)
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  );
};

interface ProductCellProps {
  item: ShopResult;
  isCheapest: boolean;
  onViewDetails: () => void;
}

const ProductCell = ({ item, isCheapest, onViewDetails }: ProductCellProps) => (
  <div className="space-y-2">
    <div className="flex items-start justify-between gap-2">
      <div className="typography-body-sm">
        {item.brand && <span className="font-medium">{item.brand}</span>}
        {item.brand && ' - '}
        {item.product_name}
      </div>
      {isCheapest && <span className="text-success shrink-0">💚</span>}
    </div>
    
    {item.promotion_type !== 'none' && (
      <PromotionIndicator
        type={item.promotion_type}
        description={item.promotion_description}
      />
    )}
    
    <div className="flex items-baseline gap-3">
      <span className="font-mono text-base font-semibold">
        ${item.price.toFixed(2)}
      </span>
      <span className="font-mono text-sm text-muted-foreground">
        ${item.unit_price.toFixed(2)}/{item.pack_size_normalized >= 1000 ? 'kg' : 'g'}
      </span>
    </div>
    
    <button
      onClick={onViewDetails}
      className="typography-body-sm text-primary hover:text-primary/80 
                 flex items-center gap-1 transition-colors"
    >
      View product
      <ExternalLink className="h-3 w-3" />
    </button>
  </div>
);
```

---

### 4.9 Product Detail Modal (Radix Dialog)

**`src/features/mystery-shopper/components/ProductDetailModal.tsx`:**

```tsx
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog';
import { useMysteryShopperStore } from '../store/mystery-shopper.store';
import { useShopResults } from '../hooks/useShopData';
import { ExternalLink } from 'lucide-react';
import { PromotionIndicator } from './shared/PromotionIndicator';

export const ProductDetailModal = () => {
  const {
    productDetailOpen,
    closeProductDetail,
    selectedProduct,
    selectedWeek,
    selectedMission,
    selectedPersona,
  } = useMysteryShopperStore();
  
  const { data: shopResults } = useShopResults(
    selectedWeek,
    selectedMission!,
    selectedPersona!
  );
  
  if (!selectedProduct || !shopResults) return null;
  
  const product = shopResults.find(
    r => r.list_item === selectedProduct.listItem &&
         r.retailer === selectedProduct.retailer
  );
  
  if (!product) return null;
  
  // Get competitor products
  const competitors = shopResults.filter(
    r => r.list_item === selectedProduct.listItem &&
         r.retailer !== selectedProduct.retailer
  );
  
  return (
    <Dialog open={productDetailOpen} onOpenChange={closeProductDetail}>
      <DialogContent className="glass-dialog max-w-2xl max-h-[80vh] overflow-y-auto">
        <DialogHeader>
          <DialogTitle className="typography-h2">
            {product.product_name}
          </DialogTitle>
          <p className="typography-body-sm text-muted-foreground">
            Selected for: {selectedMission} | {selectedPersona} persona
          </p>
        </DialogHeader>
        
        <div className="space-y-6 mt-4">
          {/* Product Info */}
          <div className="glass-card p-4 space-y-2">
            <h3 className="typography-h4">Product Info</h3>
            <ul className="typography-body-sm space-y-1.5 text-muted-foreground">
              <li>• Pack size: {product.pack_size}</li>
              <li>• Price: <span className="font-mono">${product.price.toFixed(2)}</span> 
                  (<span className="font-mono">${product.unit_price.toFixed(2)}/kg</span>)
              </li>
              <li>
                • Promotion: {' '}
                {product.promotion_type === 'none' ? (
                  'None'
                ) : (
                  <PromotionIndicator
                    type={product.promotion_type}
                    description={product.promotion_description}
                  />
                )}
              </li>
              <li>• Brand: {product.brand}</li>
              {product.quality_markers && product.quality_markers.length > 0 && (
                <li>• Quality: {product.quality_markers.join(', ')}</li>
              )}
              <li>
                • URL: {' '}
                <a
                  href={product.product_url}
                  target="_blank"
                  rel="noopener noreferrer"
                  className="text-primary hover:text-primary/80 inline-flex items-center gap-1"
                >
                  {product.retailer} product page
                  <ExternalLink className="h-3 w-3" />
                </a>
              </li>
            </ul>
          </div>
          
          {/* Agent Reasoning */}
          <div className="glass-card p-4 space-y-2">
            <h3 className="typography-h4">Why This Product Was Selected</h3>
            <p className="typography-body text-muted-foreground italic">
              Agent reasoning ({selectedPersona} persona):
            </p>
            <p className="typography-body">
              "{product.agent_reasoning}"
            </p>
            
            <div className="flex items-center gap-4 mt-3 pt-3 border-t border-border/60">
              <div>
                <span className="typography-body-sm text-muted-foreground">Confidence:</span>
                <span className={`typography-body ml-2 font-medium ${
                  product.confidence_score >= 0.8 ? 'text-success' :
                  product.confidence_score >= 0.5 ? 'text-warning' :
                  'text-destructive'
                }`}>
                  {(product.confidence_score * 100).toFixed(0)}% 
                  {product.confidence_score >= 0.8 ? ' (High)' :
                   product.confidence_score >= 0.5 ? ' (Medium)' :
                   ' (Low)'}
                </span>
              </div>
              
              {product.alternative_product_id && (
                <div>
                  <span className="typography-body-sm text-muted-foreground">
                    Alternative if unavailable
                  </span>
                </div>
              )}
            </div>
          </div>
          
          {/* Competitor Comparison */}
          {competitors.length > 0 && (
            <div className="glass-card p-4 space-y-3">
              <h3 className="typography-h4">Competitor Comparison</h3>
              {competitors.map(comp => {
                const diff = comp.price - product.price;
                return (
                  <div key={comp.retailer} className="border-b border-border/30 last:border-0 pb-3 last:pb-0">
                    <div className="typography-body font-medium">{comp.product_name}</div>
                    <div className="flex items-center gap-3 mt-1">
                      <span className="font-mono text-sm">
                        ${comp.price.toFixed(2)} (${comp.unit_price.toFixed(2)}/kg)
                      </span>
                      {comp.promotion_type !== 'none' && (
                        <PromotionIndicator type={comp.promotion_type} />
                      )}
                    </div>
                    <div className={`typography-body-sm mt-1 ${
                      diff < 0 ? 'text-success' : diff > 0 ? 'text-destructive' : 'text-muted-foreground'
                    }`}>
                      {diff < 0 ? '💚 ' : diff > 0 ? '❌ ' : ''}
                      ${Math.abs(diff).toFixed(2)} {diff < 0 ? 'cheaper' : diff > 0 ? 'more expensive' : 'same price'}
                    </div>
                  </div>
                );
              })}
            </div>
          )}
        </div>
      </DialogContent>
    </Dialog>
  );
};
```

---

### 4.10 Trend Analysis with ECharts

**`src/features/mystery-shopper/components/TrendAnalysis.tsx`:**

```tsx
import { useMemo } from 'react';
import ReactECharts from 'echarts-for-react';
import { useTrendData } from '../hooks/useShopData';
import { useMysteryShopperStore } from '../store/mystery-shopper.store';
import { FilterBar } from './shared/FilterBar';
import { ArrowLeft } from 'lucide-react';

export const TrendAnalysis = () => {
  const {
    selectedMission,
    selectedPersona,
    setActiveTab,
  } = useMysteryShopperStore();
  
  const { data: trendData, isLoading } = useTrendData(
    selectedMission!,
    selectedPersona!,
    8 // 8 weeks
  );
  
  const chartOption = useMemo(() => {
    if (!trendData) return {};
    
    return {
      tooltip: {
        trigger: 'axis',
        backgroundColor: 'rgba(0, 0, 0, 0.8)',
        borderColor: 'rgba(255, 255, 255, 0.2)',
        textStyle: {
          color: '#fff',
          fontFamily: 'Amiko, sans-serif',
        },
        formatter: (params: any) => {
          const date = params[0].axisValue;
          let tooltip = `<div style="font-weight: 600; margin-bottom: 8px;">${date}</div>`;
          params.forEach((param: any) => {
            tooltip += `
              <div style="display: flex; justify-content: space-between; gap: 20px;">
                <span>${param.marker} ${param.seriesName}</span>
                <span style="font-family: 'Inconsolata', monospace;">$${param.value.toFixed(2)}</span>
              </div>
            `;
          });
          return tooltip;
        },
      },
      legend: {
        data: ['Woolworths', 'Coles', 'Aldi'],
        textStyle: {
          fontFamily: 'Amiko, sans-serif',
          fontSize: 14,
        },
      },
      grid: {
        left: '3%',
        right: '4%',
        bottom: '3%',
        containLabel: true,
      },
      xAxis: {
        type: 'category',
        boundaryGap: false,
        data: trendData.weeks,
        axisLabel: {
          fontFamily: 'Amiko, sans-serif',
          fontSize: 12,
        },
      },
      yAxis: {
        type: 'value',
        name: 'Basket Price ($)',
        axisLabel: {
          formatter: '${value}',
          fontFamily: 'Inconsolata, monospace',
        },
        nameTextStyle: {
          fontFamily: 'Amiko, sans-serif',
        },
      },
      series: [
        {
          name: 'Woolworths',
          type: 'line',
          data: trendData.woolworths,
          smooth: true,
          lineStyle: { width: 3 },
          itemStyle: { color: 'hsl(var(--chart-1))' },
          symbol: 'circle',
          symbolSize: 8,
        },
        {
          name: 'Coles',
          type: 'line',
          data: trendData.coles,
          smooth: true,
          lineStyle: { width: 3 },
          itemStyle: { color: 'hsl(var(--chart-2))' },
          symbol: 'rect',
          symbolSize: 8,
        },
        {
          name: 'Aldi',
          type: 'line',
          data: trendData.aldi,
          smooth: true,
          lineStyle: { width: 3 },
          itemStyle: { color: 'hsl(var(--chart-3))' },
          symbol: 'triangle',
          symbolSize: 10,
        },
      ],
    };
  }, [trendData]);
  
  if (isLoading) {
    return <div>Loading trend data...</div>;
  }
  
  return (
    <div className="space-y-8">
      {/* Header */}
      <div>
        <button
          onClick={() => setActiveTab('dashboard')}
          className="flex items-center gap-2 typography-body-sm text-muted-foreground 
                     hover:text-foreground transition-colors mb-4"
        >
          <ArrowLeft className="h-4 w-4" />
          Back to Dashboard
        </button>
        
        <h1 className="typography-display">Price Trends</h1>
      </div>
      
      {/* Filters */}
      <FilterBar />
      
      {/* Chart */}
      <div className="glass-card p-6">
        <h2 className="typography-h2 mb-6">Total Basket Trend</h2>
        <ReactECharts
          option={chartOption}
          style={{ height: '400px' }}
          opts={{ renderer: 'svg' }}
        />
      </div>
      
      {/* Key Insights */}
      <div className="glass-card p-6">
        <h3 className="typography-h3 mb-4">Key Insights</h3>
        <ul className="typography-body space-y-2 text-muted-foreground">
          {trendData?.insights?.map((insight, idx) => (
            <li key={idx}>• {insight}</li>
          ))}
        </ul>
      </div>
      
      {/* Top Price Movers */}
      <div className="glass-card p-6">
        <h3 className="typography-h3 mb-4">Top Price Movers (Last 8 weeks)</h3>
        <div className="space-y-4">
          {trendData?.price_movers?.map((mover, idx) => (
            <div key={idx} className="border-b border-border/30 last:border-0 pb-4 last:pb-0">
              <div className="flex items-start justify-between">
                <div>
                  <div className="typography-body font-medium">
                    {idx + 1}. {mover.item_name} ({mover.retailer})
                  </div>
                  <div className="typography-body-sm text-muted-foreground mt-1">
                    {mover.explanation}
                  </div>
                </div>
                <div className={`typography-body font-mono ${
                  mover.change > 0 ? 'text-destructive' : 'text-success'
                }`}>
                  {mover.change > 0 ? '+' : ''}${mover.change.toFixed(2)} 
                  ({mover.change_pct > 0 ? '+' : ''}{mover.change_pct.toFixed(1)}%)
                </div>
              </div>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};
```

---

## 5. Data Layer

### 5.1 Backend API Endpoints (Express/Python)

**You'll need to create these endpoints in your backend:**

```typescript
// Express.js example
app.get('/api/mystery-shopper/shop-results', async (req, res) => {
  const { week, mission, persona } = req.query;
  
  const query = `
    SELECT * FROM \`mystery_shopper.shop_results\`
    WHERE date_executed = @week
    ${mission ? 'AND mission_name = @mission' : ''}
    ${persona ? 'AND persona = @persona' : ''}
    ORDER BY item_order, retailer
  `;
  
  const [rows] = await bigquery.query({
    query,
    params: { week, mission, persona },
  });
  
  res.json(rows);
});

app.get('/api/mystery-shopper/basket-summaries', async (req, res) => {
  const { week } = req.query;
  
  const query = `
    SELECT * FROM \`mystery_shopper.basket_summaries\`
    WHERE date_executed = @week
    ORDER BY mission_name, persona, retailer
  `;
  
  const [rows] = await bigquery.query({ query, params: { week } });
  res.json(rows);
});

app.get('/api/mystery-shopper/week-summary', async (req, res) => {
  const { week } = req.query;
  
  const query = `
    SELECT 
      retailer,
      SUM(total_basket_price) as total,
      SUM(price_vs_woolworths_dollar) as vs_ww_dollar,
      AVG(price_vs_woolworths_pct) as vs_ww_pct
    FROM \`mystery_shopper.basket_summaries\`
    WHERE date_executed = @week
    GROUP BY retailer
  `;
  
  const [rows] = await bigquery.query({ query, params: { week } });
  res.json(rows);
});

app.post('/api/mystery-shopper/export', async (req, res) => {
  const { week, mission, persona } = req.body;
  
  // Generate Google Sheets export
  const url = await generateSheetsExport(week, mission, persona);
  res.json({ url });
});
```

---

## 6. Styling Patterns

### 6.1 Glass Morphism Application

**Use your existing classes throughout:**

```tsx
// Primary containers
<div className="glass-card p-6">
  {/* Week summary cards, basket tables */}
</div>

// Interactive cards (missions, product cards)
<div className="glass-card-interactive p-4 cursor-pointer">
  {/* Clickable mission cards */}
</div>

// Modals
<DialogContent className="glass-dialog max-w-2xl">
  {/* Product detail modal */}
</DialogContent>
```

---

### 6.2 Typography System Application

**Consistent hierarchy:**

```tsx
// Page titles
<h1 className="typography-display">Mystery Shopping Agent</h1>

// Section headers
<h2 className="typography-h2">Week Summary</h2>

// Subsections
<h3 className="typography-h3">Basket Comparison</h3>

// Component titles
<h4 className="typography-h4">Mission: Spaghetti Bolognese</h4>

// Body text
<p className="typography-body">Description text</p>

// Labels
<span className="typography-label">Last updated</span>

// Small text
<span className="typography-body-sm text-muted-foreground">Metadata</span>
```

---

### 6.3 Chart Color Variables

**Define in your tailwind.config.js if not present:**

```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      colors: {
        chart: {
          1: 'hsl(var(--chart-1))', // Red (Woolworths, Red EDLP)
          2: 'hsl(var(--chart-2))', // Yellow (Coles, Yellow Ticket)
          3: 'hsl(var(--chart-3))', // Orange (Aldi, Half Price)
          4: 'hsl(var(--chart-4))',
          5: 'hsl(var(--chart-5))',
        },
      },
    },
  },
};
```

**Use in ECharts:**

```typescript
series: [
  {
    name: 'Woolworths',
    itemStyle: { color: 'hsl(var(--chart-1))' }, // Use CSS variable
  },
  {
    name: 'Coles',
    itemStyle: { color: 'hsl(var(--chart-2))' },
  },
  {
    name: 'Aldi',
    itemStyle: { color: 'hsl(var(--chart-3))' },
  },
]
```

---

## 7. Testing Strategy

### 7.1 Component Tests (Vitest + Testing Library)

**Example test for PriceBadge:**

```typescript
// src/features/mystery-shopper/components/shared/__tests__/PriceBadge.test.tsx
import { describe, it, expect } from 'vitest';
import { render, screen } from '@testing-library/react';
import { PriceBadge } from '../PriceBadge';

describe('PriceBadge', () => {
  it('renders baseline price correctly', () => {
    render(<PriceBadge price={63.90} isBaseline />);
    
    expect(screen.getByText('$63.90')).toBeInTheDocument();
    expect(screen.getByText('Baseline')).toBeInTheDocument();
  });
  
  it('renders cheaper competitor price', () => {
    render(<PriceBadge price={60.92} vsWoolworthsPct={-4.5} />);
    
    expect(screen.getByText('$60.92')).toBeInTheDocument();
    expect(screen.getByText('-4.5%')).toBeInTheDocument();
    expect(screen.getByText('✅')).toBeInTheDocument();
  });
  
  it('renders more expensive competitor price', () => {
    render(<PriceBadge price={72.50} vsWoolworthsPct={2.1} />);
    
    expect(screen.getByText('$72.50')).toBeInTheDocument();
    expect(screen.getByText('+2.1%')).toBeInTheDocument();
    expect(screen.getByText('❌')).toBeInTheDocument();
  });
});
```

---

### 7.2 Integration Tests

**Example test for BasketComparison:**

```typescript
// src/features/mystery-shopper/components/__tests__/BasketComparison.test.tsx
import { describe, it, expect, beforeEach } from 'vitest';
import { render, screen, waitFor } from '@testing-library/react';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { BasketComparison } from '../BasketComparison';
import { useMysteryShopperStore } from '../../store/mystery-shopper.store';

describe('BasketComparison', () => {
  let queryClient: QueryClient;
  
  beforeEach(() => {
    queryClient = new QueryClient({
      defaultOptions: { queries: { retry: false } },
    });
    
    // Set store state
    useMysteryShopperStore.setState({
      selectedWeek: '2026-05-16',
      selectedMission: 'Spaghetti Bolognese',
      selectedPersona: 'Saver',
    });
  });
  
  it('renders basket comparison table', async () => {
    render(
      <QueryClientProvider client={queryClient}>
        <BasketComparison />
      </QueryClientProvider>
    );
    
    await waitFor(() => {
      expect(screen.getByText('Mission: Spaghetti Bolognese')).toBeInTheDocument();
      expect(screen.getByText('Persona: Saver')).toBeInTheDocument();
    });
  });
  
  it('displays cost drivers section', async () => {
    render(
      <QueryClientProvider client={queryClient}>
        <BasketComparison />
      </QueryClientProvider>
    );
    
    await waitFor(() => {
      expect(screen.getByText('📌 Primary Cost Drivers')).toBeInTheDocument();
    });
  });
});
```

---

### 7.3 Test Coverage Requirements

**Per your CLAUDE.md: 70% minimum coverage**

```bash
# Run tests with coverage
npm test -- --coverage

# Coverage report should show:
# - Statements: ≥70%
# - Branches: ≥70%
# - Functions: ≥70%
# - Lines: ≥70%
```

---

## 8. Implementation Checklist

### Week 3 (May 12-16): Build Frontend

**Day 1-2 (May 12-13):** Core Infrastructure
- [ ] Create directory structure
- [ ] Set up TypeScript types
- [ ] Create Zustand store
- [ ] Create React Query hooks
- [ ] Create BigQuery service layer
- [ ] Add navigation tab to existing app

**Day 3-4 (May 14-15):** Main Components
- [ ] ExecutiveDashboard component
- [ ] WeekSummaryCards component
- [ ] MissionExplorer component
- [ ] MissionCard component (collapsible)
- [ ] PriceBadge component
- [ ] PromotionIndicator component
- [ ] FilterBar component

**Day 5 (May 16):** Detail Views
- [ ] BasketComparison component
- [ ] BasketComparisonTable component
- [ ] ProductDetailModal component
- [ ] CostDriversSection component

### Week 4 (May 19-21): Polish & Deploy

**Day 1 (May 19):** Advanced Features
- [ ] TrendAnalysis component with ECharts
- [ ] PersonaComparison component
- [ ] Export to Sheets functionality
- [ ] Share link functionality

**Day 2 (May 20):** Testing & Polish
- [ ] Write component tests (70% coverage)
- [ ] Integration testing
- [ ] Dark mode testing
- [ ] Responsive testing
- [ ] Error boundary testing

**Day 3 (May 21):** Deploy & Handoff
- [ ] Production build
- [ ] Deploy to staging
- [ ] Stakeholder UAT
- [ ] Fix bugs
- [ ] Deploy to production
- [ ] Documentation

---

## 9. Anti-Patterns to Avoid

### ❌ DON'T

1. **Don't duplicate glass-card styles:**
   ```tsx
   // ❌ BAD
   <div className="bg-card/40 backdrop-blur-xl border border-border/60">
   
   // ✅ GOOD
   <div className="glass-card">
   ```

2. **Don't hardcode chart colors:**
   ```tsx
   // ❌ BAD
   itemStyle: { color: '#E31837' }
   
   // ✅ GOOD
   itemStyle: { color: 'hsl(var(--chart-1))' }
   ```

3. **Don't manually style typography:**
   ```tsx
   // ❌ BAD
   <h2 className="text-2xl font-serif font-medium">
   
   // ✅ GOOD
   <h2 className="typography-h2">
   ```

4. **Don't use magic numbers:**
   ```tsx
   // ❌ BAD
   <div className="rounded-lg shadow-lg">
   
   // ✅ GOOD
   <div className="rounded-[var(--radius)] shadow-md">
   ```

5. **Don't skip semantic HTML:**
   ```tsx
   // ❌ BAD
   <div onClick={handleClick}>Click me</div>
   
   // ✅ GOOD
   <button onClick={handleClick}>Click me</button>
   ```

---

## 10. Performance Optimization

### 10.1 Code Splitting

**Already configured in your Vite setup:**

```javascript
// vite.config.ts (existing)
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        'mystery-shopper': [
          './src/features/mystery-shopper',
        ],
        'echarts': ['echarts', 'echarts-for-react'],
      },
    },
  },
},
```

### 10.2 React Query Optimization

**Use your existing React Query setup:**

```typescript
// Optimized query configuration
const { data, isLoading } = useBasketSummaries(selectedWeek, {
  staleTime: 1000 * 60 * 5,  // 5 minutes (data doesn't change often)
  gcTime: 1000 * 60 * 30,    // 30 minutes cache
  refetchOnWindowFocus: false, // Don't refetch on window focus
  refetchOnReconnect: false,   // Don't refetch on reconnect
});
```

### 10.3 ECharts Performance

**Use SVG renderer for better performance:**

```tsx
<ReactECharts
  option={chartOption}
  style={{ height: '400px' }}
  opts={{ renderer: 'svg' }}  // SVG is faster for line charts
  lazyUpdate={true}            // Batch updates
/>
```

---

## 11. Accessibility

### 11.1 Radix UI (Already Accessible)

Your existing Radix UI components handle:
- ✅ Keyboard navigation
- ✅ ARIA attributes
- ✅ Focus management
- ✅ Screen reader support

### 11.2 Additional Accessibility

**Add to components:**

```tsx
// Tables
<table role="table" aria-label="Basket comparison">
  <thead role="rowgroup">
    <tr role="row">
      <th role="columnheader">Ingredient</th>
    </tr>
  </thead>
</table>

// Buttons
<button
  aria-label="View product details"
  aria-describedby="product-description"
>
  View product
</button>

// Charts
<ReactECharts
  option={chartOption}
  aria-label="Price trend chart showing 8-week basket prices"
/>
```

---

## 12. Summary

### Tech Stack Match
- ✅ React 19 + TypeScript (your stack)
- ✅ Tailwind CSS with semantic classes (your design system)
- ✅ ECharts for visualization (your charts library)
- ✅ Radix UI for components (your UI library)
- ✅ React Query for data fetching (your server state)
- ✅ Zustand for UI state (your state management)

### Design System Match
- ✅ Typography system (.typography-* classes)
- ✅ Glass morphism (.glass-card, .glass-dialog)
- ✅ Color variables (CSS custom properties)
- ✅ Editorial aesthetic (Kaisei Opti serif)

### Anti-Pattern Compliance
- ✅ No style duplication (use design system classes)
- ✅ No hardcoded colors (use CSS variables)
- ✅ No manual typography (use semantic classes)
- ✅ No magic numbers (use design tokens)

### Testing
- ✅ 70% coverage minimum (Vitest + Testing Library)
- ✅ Component tests
- ✅ Integration tests

### Development Timeline
- **Week 3 (May 12-16):** Build components (5 days)
- **Week 4 (May 19-21):** Test + polish + deploy (3 days)

**Total:** 8 days of focused frontend development

---

**This guide provides everything needed to build a production-ready React frontend that matches your existing design system and tech stack.**

---

## 13. Non-Functional Requirements

### 13.1 Performance Requirements

**Page Load Times:**
- Executive Dashboard: <2 seconds
- Basket Comparison: <1 second
- Trend Analysis: <3 seconds (including chart rendering)
- Product Detail Modal: <500ms

**Data Refresh:**
- BigQuery query execution: <5 seconds
- UI update after data fetch: <500ms

**Export:**
- Generate Google Sheets: <10 seconds
- Show loading state during export

**React Query Caching:**
```typescript
staleTime: 1000 * 60 * 5,  // 5 minutes (data doesn't change often)
gcTime: 1000 * 60 * 30,    // 30 minutes cache
```

### 13.2 Browser Compatibility

**Supported Browsers:**
- Chrome 90+ (primary - stakeholders use Chrome)
- Firefox 88+
- Safari 14+
- Edge 90+

**Not Supported:**
- Internet Explorer (deprecated)

**Responsive:**
- Desktop-first (>1024px primary)
- Tablet support (768-1024px) nice-to-have
- Mobile (<768px) not required (stakeholders use desktop)

### 13.3 Security

**Authentication:**
- Use existing app authentication (already handled)
- Users already authenticated to access main app

**Authorization:**
- Read-only access to BigQuery (backend enforces)
- No user can modify shop data from frontend

**Data Privacy:**
- No PII displayed
- No customer data (synthetic personas only)
- All data is aggregated pricing

### 13.4 Monitoring & Logging

**User Analytics (optional):**
- Track page views (which missions viewed most)
- Filter usage patterns
- Export frequency
- Most viewed personas

**Error Tracking:**
- React Error Boundary (already in your stack)
- Log failed BigQuery queries
- Track export failures

**React Query DevTools:**
- Available in development
- Monitor cache hit rates
- Debug query performance

---

## 14. Acceptance Criteria

### 14.1 Functional Requirements

**Must Have (MVP):**
- ✅ Display 120 shops (8 missions × 5 personas × 3 retailers)
- ✅ Item-by-item basket comparison matching Feb 2026 PDF format
- ✅ Price comparison ($ and %) vs Woolworths baseline
- ✅ Promotion indicators (🔴 Red EDLP, 🟡 Yellow Ticket, 🟠 Half Price)
- ✅ Primary cost drivers section (why price differences exist)
- ✅ Export to Google Sheets (.xlsx format)
- ✅ Filter by mission, persona, week
- ✅ Historical trend view (8 weeks minimum)
- ✅ Product detail modal with agent reasoning

**Should Have:**
- ✅ Persona comparison view
- ✅ Share link functionality
- ✅ Decision quality validation metrics
- ✅ Top price movers analysis

**Nice to Have:**
- Item-level trend charts
- Custom date range selection
- Search functionality
- Downloadable PDF reports

### 14.2 Data Quality

**Validation Rules:**
- All prices displayed to 2 decimal places: `$XX.XX`
- Percentages to 1 decimal place: `X.X%`
- Handle missing data gracefully: show "N/A" not blank cells
- Handle zero values: show "$0.00" not blank
- Date formatting: consistent across all views

**Error Handling:**
```tsx
// Missing data
{price !== null ? `$${price.toFixed(2)}` : 'N/A'}

// Division by zero
{denominator !== 0 ? (value / denominator * 100).toFixed(1) : '0.0'}%

// Empty arrays
{items.length > 0 ? <Table items={items} /> : <EmptyState />}
```

### 14.3 Visual Acceptance

**Design System Compliance:**
- ✅ All typography uses semantic classes (`.typography-*`)
- ✅ All cards use glass morphism (`.glass-card*`)
- ✅ All colors use CSS variables (`hsl(var(--chart-1))`)
- ✅ No magic numbers (use design tokens)
- ✅ Dark mode fully supported (`.dark` class switching)

**Visual QA Checklist:**
- [ ] Week summary cards match design system
- [ ] Mission cards collapse/expand smoothly
- [ ] Basket comparison table is readable (alternating rows)
- [ ] Promotion indicators visible and consistent
- [ ] Product modal overlays properly
- [ ] Trend chart renders correctly on all screen sizes
- [ ] All buttons have hover states
- [ ] Loading skeletons match content layout

---

## 15. Sample Data Reference

### 15.1 Sample `shop_results` Row (JSON)

```json
{
  "shop_id": "2026-05-16_spag_saver_ww",
  "date_executed": "2026-05-16",
  "week_label": "Week of May 16, 2026",
  "mission_name": "Spaghetti Bolognese",
  "mission_description": "Most common home cooked meal",
  "persona": "Saver",
  "persona_description": "Seek weekly specials, shop by unit prices, will consider own brand for lower price",
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

### 15.2 Sample `basket_summaries` Row (JSON)

```json
{
  "shop_id": "2026-05-16_spag_saver_coles",
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
  "price_vs_woolworths_pct": -4.7,
  "is_woolworths_cheaper": false,
  "execution_time_seconds": 120.5
}
```

---

## 16. Final Handoff Checklist

### 16.1 Assets Provided

- ✅ This complete implementation guide
- ✅ Reference PDF (Feb 2026 basket comparison) - in project folder
- ✅ Persona descriptions PDF - in project folder
- ✅ BigQuery schema documentation (Section 3.4 in REACT_FRONTEND_DESIGN_GUIDE)
- ✅ Sample data queries
- ✅ Complete TypeScript types
- ✅ React component implementations
- ✅ Zustand store setup
- ✅ React Query hooks
- ✅ ECharts configuration

### 16.2 Access Required

**GCP:**
- BigQuery read access to `mystery_shopper` dataset
- Service account JSON key for BigQuery (backend)

**Development:**
- Access to existing React codebase
- Node.js 18+ installed
- npm or yarn package manager

### 16.3 Integration Points

**Add Mystery Shopper to existing navigation:**
```typescript
// In your existing navigation store
currentPage: 'mystery-shopper'  // Add to union type

// In navigation component
<NavigationTab
  icon={ShoppingCart}
  label="Mystery Shopper"
  active={currentPage === 'mystery-shopper'}
  onClick={() => setCurrentPage('mystery-shopper')}
/>
```

**Add to routing (if you add routing later):**
```typescript
// Example if you add React Router
<Route path="/mystery-shopper" element={<MysteryShopperPage />}>
  <Route index element={<ExecutiveDashboard />} />
  <Route path="basket" element={<BasketComparison />} />
  <Route path="trends" element={<TrendAnalysis />} />
  <Route path="personas" element={<PersonaComparison />} />
</Route>
```

### 16.4 Development Timeline

**Week 3 (May 12-16) - 5 days:**
- Day 1-2: Infrastructure (types, hooks, store, service layer)
- Day 3-4: Core components (dashboard, mission explorer, basket table)
- Day 5: Detail views (product modal, filters)

**Week 4 (May 19-21) - 3 days:**
- Day 1: Advanced features (trends, persona comparison, export)
- Day 2: Testing (70% coverage) + dark mode QA
- Day 3: Deploy + UAT + bug fixes

**Total:** 8 days

### 16.5 Questions & Support

**For questions about:**
- Data schema / BigQuery: Daria Volkova (backend lead)
- Business requirements: Alexa Kelly (project lead)
- Design system: Refer to existing codebase patterns
- GCP access: Your GCP admin

**Timeline:** Ready for UAT by May 20, deploy May 21

---

**END OF GUIDE**

**Version:** 2.0 (Merged with FRONTEND_SPECIFICATIONS.md intelligence)  
**Last Updated:** May 1, 2026
