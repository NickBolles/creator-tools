# Ad Analytics Dashboard - Comprehensive Project Plan

## Executive Summary

Building a web dashboard to analyze ad spend vs. revenue/bonuses for a social-media-driven affiliate business. The platform will help calculate "true ROAS" including Amazon stepped bonuses and support daily ad management decisions.

**Timeline Estimate**: MVP in 6-8 weeks, v1.1 in 10-12 weeks total

---

## 1. Assumptions & Clarifications

### Assumptions Made:

- Amazon Associates provides monthly shipped revenue reports (manually downloadable)
- Meta Ads provides daily spend/performance data (initially manual, later API)
- Affiliate commissions are tracked separately from shipped revenue bonuses
- Primary users: 1-5 person team, desktop-first usage
- Data retention: Minimum 24 months of historical data
- Bonus thresholds are monthly and reset each month
- Single currency (USD) for V1
- Bonus structure: Known thresholds (e.g., $1K shipped = $100 bonus, $5K = $500, etc.)

### Clarifying Questions for Implementation:

1. **Bonus Calculation**: Are bonuses incremental (earn each tier) or replacement (higher tier replaces lower)?
2. **Attribution Window**: What's the typical delay between ad click and Amazon commission/shipped revenue?
3. **Data Access**: Do you currently have API access to Meta Ads Manager?
4. **Historical Data**: How much historical data exists to migrate?

---

## 2. Product Overview & User Personas

### Core User Personas

#### 1. **Founder/Owner** (Primary)

**Jobs to be Done:**

- Understand true profitability of ad spend
- Decide monthly ad budget allocation
- Track progress toward bonus thresholds
- Assess overall business health

**Key Needs:**

- Quick "are ads profitable?" answer
- Projection to end-of-month
- Historical trend comparison

#### 2. **Ad Manager** (Secondary)

**Jobs to be Done:**

- Daily performance monitoring
- Campaign optimization decisions
- Enter/update daily ad data
- Identify underperforming ads to pause

**Key Needs:**

- Fast data entry workflow
- Ad-level performance comparison
- Clear spend vs. return metrics

#### 3. **Team Member/Analyst** (Tertiary)

**Jobs to be Done:**

- View performance reports
- Understand campaign context
- Support decision-making with data

**Key Needs:**

- Read-only dashboard access
- Filtered views by campaign
- Shareable reports

---

## 3. Phased Development Roadmap

### **Phase 1: MVP (Weeks 1-8)** 🎯

**Goal**: Ship a functional dashboard that answers "are ads worth it?" with manual data entry.

#### Core Features:

1. **Auth & Organizations**

   - Clerk integration with organization support
   - Basic role system (Owner, Editor, Viewer)
   - Invite team members

2. **Campaign & Ad Management**

   - CRUD operations for campaigns
   - CRUD operations for ads (linked to campaigns)
   - Basic metadata (name, status, platform, type)

3. **Manual Data Entry**

   - Daily performance input form
   - Bulk entry for multiple ads/days
   - CSV import for historical data

4. **Bonus Threshold Configuration**

   - Define monthly shipped revenue thresholds
   - Associate bonus amounts/percentages
   - Preview bonus calculation logic

5. **Main Dashboard**

   - Current month summary cards (spend, revenue, ROAS, bonus progress)
   - Monthly trend chart (last 12 months)
   - Campaign performance table
   - Simple filters (date range, campaign, status)

6. **Basic Metrics**
   - ROAS (without bonuses)
   - True ROAS (including bonuses)
   - Month-to-date vs. projected end-of-month
   - Spend efficiency by campaign

#### MVP Success Criteria:

- ✅ Can track 10+ campaigns with daily granularity
- ✅ Calculate bonus-inclusive ROAS accurately
- ✅ Enter a month's data in <15 minutes
- ✅ Answer "should we increase/decrease spend?" in <30 seconds

---

### **Phase 2: Enhanced Features (Weeks 9-12)** 📈

**Goal**: Reduce manual work and add decision-support features.

#### Features:

1. **Meta Ads Integration**

   - OAuth connection to Meta Business
   - Automated daily data sync
   - Conflict resolution (manual vs. automated data)
   - Sync status indicators

2. **Advanced Dashboard Views**

   - Ad-level detail pages
   - Campaign comparison mode
   - Stacked charts (by campaign, ad type)
   - Custom date range comparisons

3. **Projections & Forecasting**

   - Linear projection to month-end
   - "Days until next bonus threshold"
   - Scenario modeling ("what if we cut/scale X?")

4. **Enhanced Permissions**

   - Ad Manager role (edit ads, enter data only)
   - Finance role (view all, manage bonuses)
   - Granular permissions matrix

5. **Export & Reporting**

   - PDF monthly reports
   - CSV data export
   - Scheduled email summaries

6. **Deep Links & Sharing**
   - Shareable URLs with filters preserved
   - Bookmark key views
   - Team dashboard presets

---

### **Phase 3: Collaboration & Scale (Weeks 13+)** 🚀

**Goal**: Multi-team support and additional affiliate networks.

#### Features:

1. **Anchored Comments & Collaboration** (Inspired by Google Sheets/Figma)

   - **Right-click to add comments** on any dashboard element
   - **Pinned hotspots** with visual indicators (dots/badges)
   - **Context anchoring** to charts, cards, table cells, specific data points
   - **Comment threads** with replies and resolve state
   - **@mentions** with email/Slack notifications
   - **Activity log** tracking all comment activity
   - **Real-time updates** via WebSockets (optional)

   **Technical Implementation:**

   - Anchor types: widget, chart, tableCell, metricCard, dataPoint
   - Store anchor metadata: widgetId, x/y percentages, row/column keys
   - Overlay layer renders hotspots relative to anchored elements
   - `useAnchoredComments` hook for comment management
   - `CommentPinsOverlay` component for visualization

   **Data Model:**

   ```typescript
   type CommentAnchor =
     | { kind: "widget"; widgetId: string; xPct?: number; yPct?: number }
     | {
         kind: "chart";
         chartId: string;
         dataPoint?: string;
         xPct?: number;
         yPct?: number;
       }
     | { kind: "tableCell"; tableId: string; rowKey: string; columnKey: string }
     | { kind: "metricCard"; cardId: string };

   type Comment = {
     id: string;
     organizationId: string;
     anchor: CommentAnchor;
     text: string;
     createdBy: string;
     createdAt: timestamp;
     resolved: boolean;
     parentId?: string; // for replies
     mentions?: string[]; // mentioned user IDs
   };
   ```

   **Implementation Details:**

   - **Context Menu Integration**: Use shadcn/ui `DropdownMenu` triggered on right-click (contextmenu event) for each dashboard widget
   - **Comment Composer**: Small modal/sheet positioned near click location using `Popover` or `Sheet` component
   - **Hotspot Rendering**: Absolute positioned elements within widget containers, using x/y percentages for precise positioning
   - **Overlay Layer**: Separate React component (`CommentPinsOverlay`) that queries comments by anchor and renders hotspots
   - **Real-time Sync**: Optional WebSocket integration via Supabase Realtime, or polling fallback

   **Pre-Implementation Decisions Needed:**

   - Front-end framework confirmation (React/Next.js assumed)
   - Existing context menu/overlay system to integrate with
   - Current commenting API status (new feature vs. extending existing)
   - Dashboard widget structure (what are the "items" - chart widgets, tiles, table cells?)
   - Real-time requirements (WebSockets vs. refresh-to-see)

2. **Additional Affiliate Networks**

   - Multi-affiliate support (Amazon, Impact, ShareASale, etc.)
   - Network-specific bonus structures
   - Cross-network attribution

3. **Additional Ad Platforms**

   - Google Ads integration
   - TikTok Ads integration
   - Unified cross-platform view

4. **Advanced Analytics**

   - Cohort analysis
   - Retention curves
   - LTV estimation
   - Creative performance patterns

5. **Automation & Alerts**

   - Slack/email notifications
   - Threshold proximity alerts
   - Anomaly detection
   - Automated budget recommendations

6. **Modular Extensions**
   - Creative testing module
   - Content calendar module
   - Competitor tracking module

---

## 4. Data Model & Metrics Design

### Core Entities

```
Organizations
├── Users (via Clerk)
├── Campaigns
│   └── Ads
├── BonusStructures
├── DailyPerformance (fact table)
└── Settings
```

### Detailed Schema

#### **organizations**

```sql
id: uuid (PK)
name: string
created_at: timestamp
settings: jsonb (currency, timezone, etc.)
```

#### **campaigns**

```sql
id: uuid (PK)
organization_id: uuid (FK)
name: string
platform: enum (meta, google, tiktok, other)
status: enum (active, paused, archived)
start_date: date
end_date: date (nullable)
created_by: uuid (FK to users via Clerk)
created_at: timestamp
metadata: jsonb (optional custom fields)
```

#### **ads**

```sql
id: uuid (PK)
campaign_id: uuid (FK)
external_id: string (nullable, for API integrations)
name: string
platform_ad_type: string (image, video, carousel, etc.)
status: enum (active, paused, archived)
created_at: timestamp
metadata: jsonb
```

#### **daily_performance** (Fact Table - Grain: ad × date)

```sql
id: uuid (PK)
ad_id: uuid (FK)
date: date
spend: decimal(12,2)
impressions: integer
clicks: integer
conversions: integer (optional)
affiliate_revenue: decimal(12,2) -- direct attributed commission
data_source: enum (manual, meta_api, google_api)
created_at: timestamp
updated_at: timestamp
notes: text (nullable)

UNIQUE (ad_id, date, data_source)
```

#### **shipped_revenue** (Separate from daily_performance)

```sql
id: uuid (PK)
organization_id: uuid (FK)
affiliate_network: string (amazon, impact, etc.)
month: date (stored as first of month)
shipped_revenue: decimal(12,2)
commission_revenue: decimal(12,2)
created_at: timestamp
updated_at: timestamp

UNIQUE (organization_id, affiliate_network, month)
```

#### **bonus_structures**

```sql
id: uuid (PK)
organization_id: uuid (FK)
affiliate_network: string
effective_from: date
effective_to: date (nullable)
tiers: jsonb -- [{"threshold": 1000, "bonus": 100}, {...}]
calculation_type: enum (incremental, replacement)
created_at: timestamp
```

#### **roles_permissions** (Clerk + custom)

```sql
user_id: string (Clerk user ID)
organization_id: uuid (FK)
role: enum (owner, editor, ad_manager, finance, viewer)
created_at: timestamp

UNIQUE (user_id, organization_id)
```

### Key Metrics Definitions

#### 1. **Basic ROAS**

```
ROAS = Total Affiliate Revenue / Total Ad Spend
```

#### 2. **Bonus-Inclusive ROAS ("True ROAS")**

```
True ROAS = (Total Affiliate Revenue + Bonus Earned) / Total Ad Spend
```

#### 3. **Projected Month-End Metrics**

```
Projected Spend = Current Spend + (Avg Daily Spend × Days Remaining)
Projected Revenue = Current Revenue + (Avg Daily Revenue × Days Remaining)
Projected Shipped Revenue = Current + (linear extrapolation)
Projected Bonus = bonus_for_tier(Projected Shipped Revenue)
```

#### 4. **Bonus Progress**

```
Next Threshold = lowest threshold > current_shipped_revenue
Progress % = (current_shipped_revenue / Next Threshold) × 100
Days to Next Threshold = (Next Threshold - Current) / Avg Daily Shipped Revenue
```

#### 5. **Campaign Efficiency**

```
Campaign ROAS = sum(campaign_revenue) / sum(campaign_spend)
Ad ROAS = ad_revenue / ad_spend
```

### Data Grain Decisions

**Primary Grain**: `ad × date` (daily)

- Allows flexibility for aggregation
- Supports day-level trend analysis
- Manageable data volume (<50K rows/year for 100 ads)

**Shipped Revenue Grain**: `organization × affiliate_network × month`

- Matches Amazon reporting cadence
- Simplifies bonus calculation
- Allocated proportionally to ads based on attributed revenue

---

## 5. UX/UI Design

### Main Dashboard Layout

```
┌─────────────────────────────────────────────────────────┐
│ [Logo]  Dashboard  Campaigns  Settings     [User Menu] │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │ Current ROAS │  │   Spend MTD  │  │ Revenue MTD  │ │
│  │    2.4x      │  │   $12,450    │  │   $29,880    │ │
│  │ ▲ 0.3 vs LM  │  │ Proj: $18K   │  │ Proj: $43K   │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐ │
│  │  True ROAS   │  │ Shipped MTD  │  │ Next Bonus   │ │
│  │    3.1x      │  │   $8,200     │  │   $10,000    │ │
│  │ +$6,500 bonus│  │ 82% to next  │  │ ~4 days      │ │
│  └──────────────┘  └──────────────┘  └──────────────┘ │
│                                                          │
│  Filters: [Month ▼] [Campaign: All ▼] [Status ▼]       │
│                                                          │
│  ┌────────────────────────────────────────────────────┐│
│  │         Monthly Spend vs Revenue Trend             ││
│  │  $50K ┤                                            ││
│  │       │     ████ Revenue                           ││
│  │  $30K ┤    ████████                                ││
│  │       │   ████████████                             ││
│  │  $10K ┤──██████████████──── Spend                 ││
│  │       └─────────────────────────────────────       ││
│  │          Jan  Feb  Mar  Apr  May  Jun             ││
│  └────────────────────────────────────────────────────┘│
│                                                          │
│  ┌────────────────────────────────────────────────────┐│
│  │            Campaign Performance Table              ││
│  ├──────────┬────────┬─────────┬────────┬──────────┬─┤│
│  │ Campaign │ Spend  │ Revenue │ ROAS   │ Status   │▼││
│  ├──────────┼────────┼─────────┼────────┼──────────┼─┤│
│  │ Product1 │ $8,200 │ $24,600 │ 3.0x ⬆│ Active   │●││
│  │ Product2 │ $3,100 │  $6,510 │ 2.1x   │ Active   │●││
│  │ Product3 │ $1,150 │  $1,770 │ 1.5x ⬇│ Paused   │○││
│  └──────────┴────────┴─────────┴────────┴──────────┴─┘│
│                                                          │
│  [+ New Campaign]  [Enter Daily Data]                   │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### Key UI Flows

#### **Flow 1: Daily Data Entry**

```
1. Click "Enter Daily Data" button
2. Modal/page opens with:
   - Date selector (defaults to yesterday)
   - Campaign filter (optional)
   - Table: [Ad Name | Spend | Revenue | Notes]
   - Auto-save on blur
   - Keyboard navigation (Tab, Enter)
3. "Save & Close" button
4. Shows confirmation toast

Speed target: 20 ads in <3 minutes
```

#### **Flow 2: Create Campaign & Ads**

```
1. Click "+ New Campaign"
2. Slide-over form:
   - Campaign name
   - Platform (dropdown)
   - Start date
   - [Save Campaign]
3. Immediately show "Add Ads" option
4. Quick add multiple ads (spreadsheet-style)
5. Redirect to campaign detail page

Speed target: Campaign + 5 ads in <2 minutes
```

#### **Flow 3: Configure Bonus Structure**

```
1. Settings → Bonus Structures
2. Card per affiliate network
3. "+ Add Tier" button
4. Table: [Shipped Revenue Threshold | Bonus Amount | Delete]
5. Preview calculation with example values
6. Effective date selector
7. Save

Edge case: Show warning if mid-month change
```

#### **Flow 4: View Campaign Deep Dive**

```
1. Click campaign name from table
2. Campaign detail page:
   - Header with campaign metadata + edit button
   - Summary cards (spend, revenue, ROAS for this campaign)
   - Daily trend chart for this campaign
   - Ad performance table (filterable)
   - Quick actions: pause/activate ads
   - Share button → copy URL with filters
```

### shadcn/ui Component Mapping

| UI Element       | shadcn Component                          |
| ---------------- | ----------------------------------------- |
| Navigation       | `NavigationMenu`                          |
| Summary Cards    | `Card`, `CardHeader`, `CardContent`       |
| Filters          | `Select`, `Popover` + `Calendar`          |
| Tables           | `Table` + `DataTable` pattern             |
| Charts           | `recharts` (external, common with shadcn) |
| Forms            | `Form`, `Input`, `Button`                 |
| Modals           | `Dialog` or `Sheet` (slide-overs)         |
| Date Pickers     | `Calendar` + `Popover`                    |
| Toasts           | `Toast`, `useToast`                       |
| Dropdown Menus   | `DropdownMenu`                            |
| Role Badges      | `Badge`                                   |
| Data Entry Table | `Table` + `Input` (editable cells)        |

### Design Principles

1. **Information Hierarchy**: Big numbers first, details on demand
2. **Speed**: Minimize clicks, optimize for keyboard users
3. **Clarity**: Always show both "basic ROAS" and "true ROAS" side-by-side
4. **Context**: Use color sparingly (green = good, red = concerning, gray = neutral)
5. **Responsive**: Desktop-first (1440px optimal), tablet-friendly (768px+)

---

## 6. Technical Architecture & Stack

### Recommended Stack

```
Frontend:  Next.js 14 (App Router)
UI:        shadcn/ui + Tailwind CSS
Auth:      Clerk (with organizations)
Database:  PostgreSQL via Supabase
ORM:       Drizzle ORM (or Prisma)
Charts:    Recharts
Hosting:   Vercel
APIs:      Next.js API routes / Server Actions
```

### Stack Justification

#### **Database: Supabase (PostgreSQL)**

**Why:**

- Managed Postgres (simple setup, good performance)
- Built-in auth support (can work alongside Clerk)
- Realtime subscriptions (future collaboration features)
- Generous free tier, scales well
- Direct SQL access for complex queries

**Alternative:** Neon (serverless Postgres, better cold starts)

#### **Auth: Clerk**

**Why:**

- Organizations/teams built-in
- Role-based access control
- Beautiful pre-built UI components
- SSR-friendly with Next.js
- Easy user management

#### **Next.js 14 App Router**

**Why:**

- Server Components = faster initial loads
- Server Actions = simpler data mutations
- Built-in API routes
- SEO-friendly (if ever needed)
- Great DX with TypeScript

#### **Drizzle ORM** (preferred) or Prisma

**Why Drizzle:**

- Lighter than Prisma
- Better TypeScript inference
- SQL-like syntax (easier to optimize)
- Smaller bundle size

**Why Prisma (alternative):**

- More mature ecosystem
- Better migration tooling
- Prisma Studio for DB browsing

### Architecture: Modular Design

```
/app
  /(dashboard)           # Main analytics dashboard
    /layout.tsx
    /page.tsx            # Main dashboard
    /campaigns/
      /[id]/page.tsx     # Campaign detail
    /ads/
      /[id]/page.tsx     # Ad detail
  /(modules)             # Future modules
    /creative-testing/   # Module 2 (future)
    /content-calendar/   # Module 3 (future)
  /api/
    /webhooks/           # Clerk, Meta webhooks
    /integrations/       # External API calls

/lib
  /db/                   # Database client, schema
  /services/             # Business logic layer
    /metrics.ts          # ROAS, projections, bonus calc
    /campaigns.ts        # Campaign CRUD
    /ads.ts              # Ad CRUD
    /performance.ts      # Data entry, imports
    /integrations/       # Meta API, future integrations
  /auth/                 # Clerk helpers, permissions
  /utils/                # Shared utilities

/components
  /dashboard/            # Dashboard-specific components
  /ui/                   # shadcn components
  /shared/               # Reusable cross-module components

/hooks                   # Custom React hooks
  /useMetrics.ts
  /usePermissions.ts

/types                   # TypeScript types
  /database.ts
  /metrics.ts
```

### Key Architectural Decisions

#### 1. **Separation of Concerns**

```
UI Layer (components)
    ↓
API Layer (Server Actions / API routes)
    ↓
Service Layer (business logic)
    ↓
Data Layer (ORM / raw SQL)
```

**Example:**

```typescript
// Service layer - /lib/services/metrics.ts
export async function calculateTrueROAS(
  organizationId: string,
  startDate: Date,
  endDate: Date
): Promise<ROASResult> {
  // 1. Fetch ad spend & revenue
  const performance = await db.query.dailyPerformance...

  // 2. Fetch shipped revenue & bonus structure
  const shipped = await db.query.shippedRevenue...
  const bonuses = await db.query.bonusStructures...

  // 3. Calculate bonus earned
  const bonusEarned = calculateBonusFromTiers(shipped, bonuses)

  // 4. Return computed metrics
  return {
    spend: sum(performance.spend),
    revenue: sum(performance.affiliate_revenue),
    bonusEarned,
    basicROAS: revenue / spend,
    trueROAS: (revenue + bonusEarned) / spend
  }
}
```

#### 2. **Data Access Patterns**

**For Dashboards (read-heavy):**

- Use Server Components with direct DB queries
- Cache with `unstable_cache` for expensive calculations
- Revalidate on data mutations

**For Data Entry (write-heavy):**

- Use Server Actions for mutations
- Optimistic UI updates
- Validate on server, show errors inline

#### 3. **Extensibility Strategy**

**Module Registration Pattern:**

```typescript
// /lib/modules/registry.ts
export const modules = [
  {
    id: "ad-analytics",
    name: "Ad Analytics",
    path: "/dashboard",
    icon: ChartBarIcon,
    permissions: ["view_analytics"],
  },
  // Future modules plugged in here
  {
    id: "creative-testing",
    name: "Creative Testing",
    path: "/creative-testing",
    icon: BeakerIcon,
    permissions: ["view_creatives"],
  },
];
```

**Benefits:**

- Easy to add new tools without touching core
- Consistent navigation
- Per-module permissions

#### 4. **Permission System**

```typescript
// /lib/auth/permissions.ts
export enum Permission {
  VIEW_DASHBOARD = "view_dashboard",
  EDIT_CAMPAIGNS = "edit_campaigns",
  ENTER_DATA = "enter_data",
  MANAGE_BONUSES = "manage_bonuses",
  MANAGE_ORG = "manage_org",
  MANAGE_INTEGRATIONS = "manage_integrations",
}

export const rolePermissions: Record<Role, Permission[]> = {
  owner: Object.values(Permission), // All permissions
  editor: [VIEW_DASHBOARD, EDIT_CAMPAIGNS, ENTER_DATA],
  ad_manager: [VIEW_DASHBOARD, ENTER_DATA],
  finance: [VIEW_DASHBOARD, MANAGE_BONUSES],
  viewer: [VIEW_DASHBOARD],
};

export async function hasPermission(
  userId: string,
  orgId: string,
  permission: Permission
): Promise<boolean> {
  const role = await getUserRole(userId, orgId);
  return rolePermissions[role].includes(permission);
}
```

#### 5. **Integration Architecture** (Phase 2)

```typescript
// /lib/integrations/base.ts
interface DataProvider {
  authenticate(): Promise<void>;
  fetchDailyPerformance(
    startDate: Date,
    endDate: Date
  ): Promise<PerformanceData[]>;
  sync(): Promise<SyncResult>;
}

// /lib/integrations/meta.ts
class MetaAdsProvider implements DataProvider {
  async authenticate() {
    /* OAuth flow */
  }
  async fetchDailyPerformance() {
    /* Meta API */
  }
  async sync() {
    /* Handle conflicts */
  }
}

// Easy to add Google, TikTok, etc.
```

---

## 7. Implementation Plan

### Pre-Development (Week 0)

- [ ] Finalize assumptions (bonus calc, attribution)
- [ ] Set up project repo
- [ ] Configure Vercel project
- [ ] Create Supabase project
- [ ] Set up Clerk application
- [ ] Document API keys needed

### Week 1-2: Foundation

**Goal:** Auth + basic CRUD working

- [ ] Initialize Next.js project with TypeScript
- [ ] Install and configure shadcn/ui
- [ ] Set up Clerk auth with organizations
- [ ] Design database schema (Drizzle)
- [ ] Run initial migrations
- [ ] Create base layout with navigation
- [ ] Implement organization switching
- [ ] Build campaign CRUD pages
- [ ] Build ad CRUD pages (linked to campaigns)

**Deliverable:** Can create orgs, campaigns, ads (no data yet)

### Week 3-4: Data Layer

**Goal:** Data entry + storage working

- [ ] Build daily performance entry form
- [ ] Implement bulk data entry modal
- [ ] Create CSV import for historical data
- [ ] Build shipped revenue entry page
- [ ] Implement bonus structure configuration
- [ ] Write metric calculation functions
- [ ] Add data validation logic
- [ ] Create test data seed script

**Deliverable:** Can enter and store all required data

### Week 5-6: Dashboard UI

**Goal:** Main dashboard functional

- [ ] Build summary cards with live metrics
- [ ] Implement monthly trend chart (Recharts)
- [ ] Create campaign performance table
- [ ] Add filters (date, campaign, status)
- [ ] Build campaign detail page
- [ ] Implement ROAS calculations (basic + true)
- [ ] Add month-to-date vs projected display
- [ ] Create bonus progress indicator

**Deliverable:** Full dashboard answering key questions

### Week 7-8: Polish & Launch Prep

**Goal:** Production-ready MVP

- [ ] Implement role-based permissions
- [ ] Add loading states and error handling
- [ ] Build responsive layouts (tablet)
- [ ] Write user documentation
- [ ] Conduct internal testing
- [ ] Fix bugs from testing
- [ ] Set up error monitoring (Sentry)
- [ ] Configure production environment
- [ ] Deploy to production
- [ ] Onboard first users

**Deliverable:** MVP launched ✅

### Week 9-10: Meta Integration (Phase 2)

- [ ] Research Meta Marketing API requirements
- [ ] Implement OAuth connection flow
- [ ] Build data sync service
- [ ] Create conflict resolution UI
- [ ] Add sync status indicators
- [ ] Test with real Meta account
- [ ] Document setup process

### Week 11-12: Enhanced Analytics (Phase 2)

- [ ] Build ad detail pages with deep metrics
- [ ] Implement campaign comparison mode
- [ ] Add stacked chart views
- [ ] Create scenario modeling tool
- [ ] Build PDF export feature
- [ ] Implement deep links with filters
- [ ] Add scheduled email reports

---

## 8. Risk Assessment & Mitigation

### Technical Risks

| Risk                            | Probability | Impact | Mitigation                                     |
| ------------------------------- | ----------- | ------ | ---------------------------------------------- |
| Meta API complexity             | Medium      | High   | Start with manual CSV import, defer to Phase 2 |
| Bonus calculation edge cases    | Medium      | Medium | Thoroughly test with real historical data      |
| Performance with large datasets | Low         | Medium | Use proper indexes, pagination, caching        |
| Clerk org switching lag         | Low         | Low    | Use client-side caching                        |

### Product Risks

| Risk                                     | Probability | Impact | Mitigation                                          |
| ---------------------------------------- | ----------- | ------ | --------------------------------------------------- |
| Users find data entry too slow           | Medium      | High   | Extensive UX testing, keyboard shortcuts            |
| Projected metrics inaccurate             | Medium      | Medium | Show confidence intervals, allow manual override    |
| Bonus structure changes frequently       | Low         | Medium | Make config easy to update, support effective dates |
| Need multi-currency sooner than expected | Low         | Medium | Design schema to add currency field easily          |

---

## 9. Success Metrics

### MVP Success (Week 8)

- ✅ 1+ organization actively using daily
- ✅ 30-day retention > 80%
- ✅ Data entry time < 15 min/day
- ✅ Dashboard load time < 2 seconds
- ✅ Zero critical bugs in production
- ✅ User satisfaction score > 4/5

### Phase 2 Success (Week 12)

- ✅ Meta integration reduces data entry by 80%
- ✅ Advanced dashboards used weekly
- ✅ 3+ team members per org active
- ✅ Deep links shared regularly
- ✅ Month-over-month retention > 90%

### Long-term Success (6 months)

- ✅ 5+ organizations using platform
- ✅ 2+ additional affiliate networks integrated
- ✅ Collaboration features adopted
- ✅ Platform informs 90%+ of ad decisions
- ✅ Documented ROI improvement vs. previous process

---

## 10. Open Questions for Product Refinement

1. **Bonus Attribution**: Should bonuses be attributed back to specific campaigns/ads proportionally, or stay at org level?
2. **Historical Analysis**: How important is year-over-year comparison vs. month-over-month?
3. **Alerting Priority**: What triggers are most valuable? (e.g., "ROAS below 2x for 3 days")
4. **Mobile Usage**: Any scenarios where mobile view is critical?
5. **Data Retention**: Any compliance requirements for data storage duration?
6. **Multi-brand**: Will one org manage multiple brands/Amazon accounts?

---

## 11. Next Steps

### Immediate (This Week)

1. ✅ Review this plan and validate assumptions
2. Address open questions above
3. Set up development environment
4. Create initial database schema
5. Begin Week 1 tasks

### This Month

1. Complete foundation (Weeks 1-2)
2. Ship data layer (Weeks 3-4)
3. Review progress and adjust timeline

### This Quarter

1. Launch MVP to first users
2. Gather feedback and iterate
3. Begin Phase 2 development

---

## Appendix A: Database Indexes

```sql
-- Performance optimization indexes
CREATE INDEX idx_daily_perf_ad_date ON daily_performance(ad_id, date DESC);
CREATE INDEX idx_daily_perf_date_range ON daily_performance(date) WHERE date >= CURRENT_DATE - INTERVAL '24 months';
CREATE INDEX idx_ads_campaign_status ON ads(campaign_id, status);
CREATE INDEX idx_campaigns_org_status ON campaigns(organization_id, status);
CREATE INDEX idx_shipped_revenue_org_month ON shipped_revenue(organization_id, month DESC);
```

## Appendix B: Key SQL Queries

### Monthly Summary Query

```sql
WITH monthly_performance AS (
  SELECT
    DATE_TRUNC('month', dp.date) as month,
    SUM(dp.spend) as total_spend,
    SUM(dp.affiliate_revenue) as total_revenue
  FROM daily_performance dp
  JOIN ads a ON dp.ad_id = a.id
  JOIN campaigns c ON a.campaign_id = c.id
  WHERE c.organization_id = $1
    AND dp.date >= $2 AND dp.date <= $3
  GROUP BY DATE_TRUNC('month', dp.date)
),
bonuses AS (
  SELECT
    month,
    shipped_revenue,
    calculate_bonus(shipped_revenue, bonus_structure) as bonus_amount
  FROM shipped_revenue sr
  WHERE sr.organization_id = $1
    AND sr.month >= $2 AND sr.month <= $3
)
SELECT
  mp.month,
  mp.total_spend,
  mp.total_revenue,
  b.shipped_revenue,
  b.bonus_amount,
  mp.total_revenue / NULLIF(mp.total_spend, 0) as basic_roas,
  (mp.total_revenue + b.bonus_amount) / NULLIF(mp.total_spend, 0) as true_roas
FROM monthly_performance mp
LEFT JOIN bonuses b ON mp.month = b.month
ORDER BY mp.month DESC;
```

---

**End of Project Plan**

_Last Updated: 2025-11-24_
_Version: 1.0_
_Owner: Product Team_
