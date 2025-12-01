# Ad Analytics Dashboard Plan

## 1. Assumptions & Clarifications

Before defining the detailed plan, here are the key assumptions and open questions that shape this V1 proposal:

- **Currency**: Assuming all monetary values are in **USD** for V1. Multi-currency support can be added later.
- **Bonus Logic**: Assumed to be a simple tiered system where hitting a _monthly_ shipped revenue target unlocks a specific bonus amount. The bonus applies to the entire month's performance for ROAS calculation.
- **Attribution**: We are relying on the data entered (manually or later via API) as the "source of truth." We aren't building an attribution engine; we are visualizing the attributed data provided by Meta/Amazon.
- **Org Structure**: A user belongs to one organization. Multi-org membership is a "nice to have" for later.
- **Data Grain**: Daily performance per Ad is the finest grain required. Intraday data is not needed for V1.

## 2. Product Overview & Personas

### Core Personas

1.  **The Founder/Owner**:
    - **Goal**: High-level visibility. "Are we making money?" "What is our true ROAS after bonuses?"
    - **Needs**: Clear, big-picture dashboard. Monthly projections.
2.  **The Ad Manager**:
    - **Goal**: Optimization. "Which ad creative is winning?" "What should I kill?"
    - **Needs**: Granular performance views (Campaign/Ad level). Fast data entry. Comparison tools.
3.  **The Analyst/Editor** (Optional V1 Role):
    - **Goal**: Data integrity.
    - **Needs**: Ability to input data, configure bonus thresholds, and ensure numbers match external platforms.

### Jobs to Be Done

- **Monitor**: Check daily/monthly spend vs. revenue + bonuses.
- **Decide**: Identify underperforming ads to cut and high-performing ads to scale.
- **Forecast**: Estimate end-of-month bonus tier to adjust spend strategy (e.g., "push spend to hit the next bonus tier").

## 3. Feature Roadmap

### MVP (Must-Have for V1)

- **Auth & Org Management**: Sign up, create org, invite team members (Clerk).
- **Entity Management**: CRUD for Campaigns and Ads.
- **Bonus Configuration**: Define monthly revenue thresholds and associated bonus amounts.
- **Manual Data Entry**: Grid-like interface to input Daily Ad Spend, Affiliate Revenue, Shipped Revenue.
- **Core Dashboard**:
  - Date range picker (Today, Yesterday, Last 7/30 days, Month to Date, Custom).
  - Summary Cards: Total Spend, Revenue, Bonus (Pro-rated/Actual), True ROAS.
  - Main Chart: Daily Spend vs. Revenue over time.
  - Data Table: Breakdown by Campaign/Ad with sorting/filtering.
- **Calculated Metrics**: Real-time ROAS calculation including step bonuses based on current/projected totals.

### Near-Term (V1.1)

- **Meta Ads Integration**: Auto-import Spend, Impressions, Clicks (via Marketing API).
- **Amazon Associates Integration**: CSV import or scraping/API if available for revenue.
- **Projections UI**: Dedicated view for "What if" scenarios for end-of-month bonuses.

### Later (Nice-to-Have)

- **Creative Analysis**: Aggregated view by creative ID/type (image vs video).
- **Contextual Comments (Pinned Discussions)**: Comments on specific days, ads, or charts.
- **Multi-channel Support**: Google Ads, TikTok Ads, etc.

## 4. Data Model & Metrics Design

### Schema Definitions (Drizzle ORM)

Use `drizzle-orm/pg-core` for definitions.

**1. Organizations Table** (`organizations`)
- `id`: uuid, primary key, defaultRandom
- `name`: text, not null
- `slug`: text, unique, not null
- `created_at`: timestamp, defaultNow

**2. Users Table** (`users`)
*Note: We sync essential Clerk data here for foreign key constraints, but rely on Clerk for auth.*
- `id`: text, primary key (This is the `user_id` from Clerk)
- `org_id`: uuid, references `organizations.id`
- `email`: text, not null
- `role`: text, enum ('owner', 'editor', 'viewer'), default 'viewer'
- `created_at`: timestamp, defaultNow

**3. Campaigns Table** (`campaigns`)
- `id`: uuid, primary key, defaultRandom
- `org_id`: uuid, references `organizations.id`, not null
- `name`: text, not null
- `platform`: text, enum ('meta', 'tiktok', 'google'), default 'meta'
- `status`: text, enum ('active', 'paused', 'archived'), default 'active'
- `external_id`: text (unique per platform per org)
- `created_at`: timestamp, defaultNow

**4. Ads Table** (`ads`)
- `id`: uuid, primary key, defaultRandom
- `campaign_id`: uuid, references `campaigns.id`, not null
- `name`: text, not null
- `ad_type`: text, enum ('image', 'video', 'carousel', 'other')
- `status`: text, enum ('active', 'paused', 'archived'), default 'active'
- `external_id`: text
- `created_at`: timestamp, defaultNow

**5. Daily Metrics Table** (`daily_ad_metrics`)
- `id`: uuid, primary key, defaultRandom
- `ad_id`: uuid, references `ads.id`, not null
- `date`: date, not null (Index this for range queries)
- `spend`: numeric(10, 2), default 0
- `affiliate_revenue`: numeric(10, 2), default 0 (Direct Commission)
- `shipped_revenue`: numeric(10, 2), default 0 (For Bonus Calc)
- `impressions`: integer, default 0
- `clicks`: integer, default 0
- `roas`: numeric(10, 2), generated always as (affiliate_revenue / NULLIF(spend, 0)) stored (Optional, or compute on read)
- *Constraint*: Unique constraint on `(ad_id, date)`

**6. Bonus Configurations Table** (`bonus_configs`)
- `id`: uuid, primary key
- `org_id`: uuid, references `organizations.id`
- `year_month`: date (store as 1st of month, e.g., '2023-10-01')
- `tiers`: jsonb
  - *Structure*: `[{ "threshold": 10000, "bonus_amount": 500 }, { "threshold": 25000, "bonus_amount": 1500 }]`
  - *Validation*: Ensure sorted by threshold ascending.

### Key Metrics & Calculations

1.  **Direct ROAS**:
    $$ \text{Direct ROAS} = \frac{\sum \text{Affiliate Revenue}}{\sum \text{Spend}} $$

2.  **Projected Bonus**:
    - Get `Current Shipped Revenue` (Sum of `shipped_revenue` for the month).
    - Get `Days Passed` and `Days Remaining` in month.
    - `Avg Daily Shipped` = `Current Shipped` / `Days Passed`.
    - `Projected Shipped Total` = `Current Shipped` + (`Avg Daily Shipped` * `Days Remaining`).
    - **Lookup**: Iterate through `bonus_configs.tiers` for the current month. Find the highest `threshold` <= `Projected Shipped Total`. That is the `Projected Bonus`.

3.  **True ROAS (with Bonus)**:
    - **Historical (Past Months)**:
       $$ \frac{\text{Total Affiliate Rev} + \text{Final Bonus Achieved}}{\text{Total Spend}} $$
    - **Current Month (Real-time)**:
       $$ \frac{\text{Total Affiliate Rev (MTD)} + \text{Projected Bonus}}{\text{Total Spend (MTD)}} $$
    - *Note*: When filtering by specific campaigns/ads, the bonus usually isn't included because it's an account-level metric. *Decision*: Only show "True ROAS" on the Account/Global view. For Campaign/Ad views, show "Direct ROAS".

## 5. UX / UI Design & Key Flows

**Style**: Minimal, "Linear-esque" or Vercel-style. High contrast, data-dense but legible.

### Component Hierarchy

**1. Layout (`app/(dashboard)/layout.tsx`)**
- **Sidebar**:
  - Logo & Org Switcher
  - Navigation Links (Dashboard, Campaigns, Data Entry, Settings)
  - User Profile (Clerk)
- **Main Content Area**: `flex-1 p-6 overflow-auto`

**2. Dashboard Page (`app/(dashboard)/overview/page.tsx`)**
- **Controls**:
  - `DateRangePicker` (Global state via URL search params `?from=...&to=...`)
  - `MetricToggle`: "Show Projections" (Switch)
- **KPI Grid**: 4 Cards.
  - Spend: `$12,450` (vs prev period)
  - Direct Revenue: `$15,200`
  - **True ROAS**: `1.45x` (Color coded: Green >= 1.2)
  - Bonus Progress: Progress bar showing `$85k / $100k` shipped.
- **Main Chart**:
  - `AreaChart` (Gradient fill). X-Axis: Date.
  - Series 1: Revenue (Green)
  - Series 2: Spend (Red)
  - Tooltip: Shows exact numbers + ROAS for that day.
- **Breakdown Table**:
  - Columns: Name, Spend, Direct Rev, Shipped Rev, ROAS.
  - Row Action: "Edit Data" (Opens drawer/modal).

**3. Data Entry Page (`app/(dashboard)/data-entry/page.tsx`)**
- *Goal*: High speed input.
- **Date Selector**: Single day picker (Defaults to yesterday).
- **Grid**:
  - Rows: Active Ads (Grouped by Campaign).
  - Columns: `Spend`, `Affiliate Rev`, `Shipped Rev`.
  - **Interaction**:
    - `react-hook-form` + `useFieldArray`.
    - `onBlur` autosave or "Save All" button at bottom floating.
    - "Fill from previous" button to copy yesterday's values as a starting point.
  - **Validation**: Prevent negative numbers. Warn on unusually high/low values (e.g., > 2x avg).

### Components (shadcn/ui)
- `Button`, `Input`, `Table`, `Card`, `Select`, `Form`, `Dialog`, `Sheet` (for side editing), `Progress` (for bonus tracking).
- **Charts**: Use `shadcn/charts` wrapper around Recharts.

## 6. Contextual Comments (Pinned Discussions) - *Future/Nice-to-Have*

This feature allows treating comments like "pins" anchored to specific elements (chart, card, table cell) via right-click, rendered as small hotspots.

### Concept & Interaction
- **Add Comment**: Right-click on a dashboard element -> "Add comment". Opens a small composer at the click location.
- **Pinned Hotspot**: After saving, a dot/badge appears at the click offset or top-right of the element.
- **View Thread**: Hovering or clicking the hotspot opens the thread overlay with replies and resolve state.

### Data Shape (Draft)
```typescript
type CommentAnchor =
  | { kind: 'widget'; widgetId: string; xPct?: number; yPct?: number }
  | { kind: 'tableCell'; tableId: string; rowKey: string; columnKey: string };

type Comment = {
  id: string;
  anchor: CommentAnchor;
  text: string;
  createdBy: string; // Clerk ID
  createdAt: string;
  resolved: boolean;
  parentId?: string; // for replies
};
```

### Implementation Strategy
- **Hook**: `useAnchoredComments` to fetch and filter comments for a specific `widgetId`.
- **Overlay Component**: `CommentPinsOverlay` which renders absolute-positioned dots over the widget container.
- **Backend**: A `comments` table in Postgres with a JSONB `anchor` column.

## 7. Implementation Guidance

### Tech Stack Specifics
- **Framework**: Next.js 14+ (App Router).
- **Auth**: Clerk (`@clerk/nextjs`).
- **DB**: Postgres (Neon/Supabase) + Drizzle ORM (`drizzle-orm`, `drizzle-kit`, `postgres`).
- **Styling**: Tailwind CSS + `clsx` + `tailwind-merge`.
- **Icons**: `lucide-react`.
- **State**: `nuqs` for URL search params (filters/dates), `react-hook-form` + `zod` for forms, `@tanstack/react-query` for data fetching.

### Architecture & Folder Structure

```text
/src
  /app
    /(auth)              # sign-in / sign-up
    /(dashboard)
      /layout.tsx        # Sidebar + Auth check
      /overview/page.tsx # Main Dashboard
      /campaigns/page.tsx
      /data-entry/page.tsx
      /settings/page.tsx
    /api
      /webhooks/clerk    # Sync user data
  /components
    /ui                  # shadcn primitives
    /domain              # Domain-specific components
      /dashboard-kpi-card.tsx
      /bonus-progress-bar.tsx
      /data-entry-grid.tsx
      /comments          # Future comments feature
        /comment-pins-overlay.tsx
        /comment-composer.tsx
  /db
    /schema.ts           # All table definitions
    /index.ts            # DB connection
  /lib
    /utils.ts            # cn() helper
    /constants.ts        # Currency formatting rules
  /features              # Feature-based logic
    /analytics
      /actions.ts        # calculateRoas, getMetrics
      /utils.ts          # Math helpers
    /campaigns
      /actions.ts        # createCampaign, updateAd
    /bonuses
      /actions.ts        # updateBonusConfig
  /types                 # Global types
```

### Step-by-Step Implementation Plan for LLM

**Phase 1: Foundation**
1.  **Scaffold**: `npx create-next-app@latest` with TypeScript, Tailwind, ESLint.
2.  **Install Core Deps**: `shadcn-ui`, `lucide-react`, `clsx`, `tailwind-merge`.
3.  **Database Setup**: Install `drizzle-orm` & `postgres`. Configure `drizzle.config.ts`. Create `src/db/schema.ts` with the tables defined above. Run migration.
4.  **Auth Setup**: Install `@clerk/nextjs`. Wrap root layout with `<ClerkProvider>`. Add `middleware.ts` to protect dashboard routes.

**Phase 2: Core Data & Entry**
5.  **Seed/CRUD Actions**: Create Server Actions for creating Campaigns and Ads.
6.  **Data Entry UI**: Build the `DataEntryGrid` component using `react-hook-form`. Implement the "Save" server action (`upsertDailyMetrics`).
7.  **Bonus Logic**: Implement `BonusConfiguration` settings page. Write the utility function `calculateProjectedBonus(metrics, config)`.

**Phase 3: Visualization**
8.  **KPI Calculation**: Write aggregation queries (SQL `SUM` grouped by date) in `src/features/analytics/actions.ts`.
9.  **Dashboard UI**: Build the KPI cards and Main Chart. Connect them to the aggregated data.
10. **Filtering**: Implement `nuqs` to drive the dashboard date range and filter state.

**Phase 4: Polish**
11. **Formatting**: Ensure currency ($) and percentages (%) are formatted consistently.
12. **Edge Cases**: Handle empty states (no data for selected range).
13. **Responsive Check**: Ensure data tables scroll horizontally on mobile.
