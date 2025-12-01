# Quick Reference - Ad Analytics Dashboard

> One-page cheat sheet for quick lookups during development

---

## 📋 Project at a Glance

| Item             | Detail                                  |
| ---------------- | --------------------------------------- |
| **Project Name** | Ad Analytics Dashboard                  |
| **Primary Goal** | Calculate true ROAS with Amazon bonuses |
| **Timeline**     | 8 weeks to MVP, 12 weeks to Phase 2     |
| **Team Size**    | 1-2 developers                          |
| **Total Effort** | 320-400 hours                           |

---

## 🛠️ Tech Stack (Copy-Paste Ready)

```bash
# Core Dependencies
pnpm add next@latest react react-dom
pnpm add @clerk/nextjs
pnpm add drizzle-orm postgres
pnpm add recharts
pnpm add -D drizzle-kit typescript @types/node @types/react

# shadcn/ui Setup
npx shadcn-ui@latest init
npx shadcn-ui@latest add card table button dialog select calendar form input toast badge dropdown-menu navigation-menu sheet
```

---

## 🔑 Environment Variables Template

```env
# .env.local
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxx
CLERK_SECRET_KEY=sk_test_xxx
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard

DATABASE_URL=postgresql://user:pass@host:5432/db
DIRECT_URL=postgresql://user:pass@host:5432/db

# Phase 2
META_APP_ID=xxx
META_APP_SECRET=xxx
```

---

## 📊 Core Metrics Formulas

### Basic ROAS

```typescript
basicROAS = totalAffiliateRevenue / totalAdSpend;
```

### True ROAS (with bonuses)

```typescript
trueROAS = (totalAffiliateRevenue + bonusEarned) / totalAdSpend;
```

### Projected End-of-Month

```typescript
const daysInMonth = 30;
const daysPassed = 15;
const daysRemaining = daysInMonth - daysPassed;

projectedSpend = currentSpend + (currentSpend / daysPassed) * daysRemaining;
projectedRevenue =
  currentRevenue + (currentRevenue / daysPassed) * daysRemaining;
```

### Bonus Calculation (Incremental)

```typescript
function calculateBonus(shipped: number, tiers: Tier[]): number {
  return tiers
    .filter((t) => shipped >= t.threshold)
    .reduce((sum, t) => sum + t.bonus, 0);
}
```

---

## 🗂️ File Structure

```
/app
  /(dashboard)
    /page.tsx                  # Main dashboard
    /campaigns/
      /page.tsx                # Campaign list
      /[id]/page.tsx          # Campaign detail
    /ads/[id]/page.tsx        # Ad detail
    /settings/
      /team/page.tsx          # User management
      /bonuses/page.tsx       # Bonus config
  /actions/
    /campaigns.ts             # Campaign mutations
    /ads.ts                   # Ad mutations
    /performance.ts           # Data entry
  /api/
    /webhooks/                # Clerk, Meta webhooks

/lib
  /db/
    /schema.ts                # Drizzle schema
    /index.ts                 # DB client
  /services/
    /metrics.ts               # ROAS, projections
    /campaigns.ts             # Campaign logic
    /performance.ts           # Data queries
  /auth/
    /permissions.ts           # Permission system
  /utils/
    /dates.ts                 # Date helpers
    /currency.ts              # Formatting

/components
  /dashboard/
    /summary-cards.tsx        # Metric cards
    /trend-chart.tsx          # Monthly chart
    /campaign-table.tsx       # Performance table
  /performance/
    /data-entry-modal.tsx     # Daily entry form
    /csv-import.tsx           # Historical import
  /ui/                        # shadcn components
```

---

## 🔐 Permission Matrix

| Role           | View | Edit Campaigns | Enter Data | Manage Bonuses | Manage Org |
| -------------- | ---- | -------------- | ---------- | -------------- | ---------- |
| **Owner**      | ✅   | ✅             | ✅         | ✅             | ✅         |
| **Editor**     | ✅   | ✅             | ✅         | ❌             | ❌         |
| **Ad Manager** | ✅   | ❌             | ✅         | ❌             | ❌         |
| **Finance**    | ✅   | ❌             | ❌         | ✅             | ❌         |
| **Viewer**     | ✅   | ❌             | ❌         | ❌             | ❌         |

---

## 🎯 Key User Flows (Time Targets)

| Flow                           | Target Time | Steps                                              |
| ------------------------------ | ----------- | -------------------------------------------------- |
| **Create Campaign + 5 Ads**    | <2 minutes  | Click + New → Fill form → Add ads → Save           |
| **Enter 30 Days of Data**      | <15 minutes | Open modal → Select date → Fill spreadsheet → Save |
| **View Dashboard**             | <30 seconds | Open dashboard → See all metrics → Make decision   |
| **Check Campaign Performance** | <1 minute   | Click campaign → Review metrics → Identify issues  |

---

## 📐 Database Schema (Quick Reference)

### Main Tables

- `organizations` - Top-level entity
- `campaigns` - Marketing campaigns
- `ads` - Individual ads within campaigns
- `daily_performance` - **Fact table** (ad × date grain)
- `shipped_revenue` - Monthly shipped revenue by affiliate
- `bonus_structures` - Bonus tier definitions
- `user_roles` - Permission assignments

### Critical Indexes

```sql
CREATE INDEX idx_daily_perf_ad_date ON daily_performance(ad_id, date DESC);
CREATE INDEX idx_ads_campaign ON ads(campaign_id);
CREATE INDEX idx_campaigns_org ON campaigns(organization_id);
```

---

## 🎨 shadcn Components Used

| Component        | Use Case                      |
| ---------------- | ----------------------------- |
| `Card`           | Summary metric cards          |
| `Table`          | Campaign/ad lists             |
| `Dialog`         | Modals (data entry)           |
| `Sheet`          | Slide-over panels (filters)   |
| `Select`         | Dropdowns (campaign selector) |
| `Calendar`       | Date pickers                  |
| `Form`           | All forms                     |
| `Button`         | Actions                       |
| `Toast`          | Success/error notifications   |
| `Badge`          | Status indicators             |
| `DropdownMenu`   | User menu, actions            |
| `NavigationMenu` | Top nav                       |

---

## 🚀 Common Commands

### Development

```bash
# Start dev server
pnpm dev

# Run type check
pnpm tsc --noEmit

# Generate DB migration
pnpm drizzle-kit generate:pg

# Push schema to DB
pnpm drizzle-kit push:pg

# Open DB Studio
pnpm drizzle-kit studio
```

### Database

```bash
# Connect to DB
psql $DATABASE_URL

# Run migration
pnpm drizzle-kit migrate

# Seed data
pnpm tsx scripts/seed.ts
```

### Deployment

```bash
# Deploy to Vercel
git push origin main  # Auto-deploys

# Manual deploy
vercel --prod
```

---

## 📊 Dashboard Layout (ASCII)

```
┌────────────────────────────────────────────────┐
│  [Logo]   Dashboard   Campaigns   [User Menu] │
├────────────────────────────────────────────────┤
│  ┌─────────┐ ┌─────────┐ ┌─────────┐         │
│  │ ROAS    │ │ Spend   │ │ Revenue │         │
│  │  2.4x   │ │ $12.5K  │ │ $29.8K  │         │
│  └─────────┘ └─────────┘ └─────────┘         │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐         │
│  │True ROAS│ │ Shipped │ │ Bonus   │         │
│  │  3.1x   │ │ $8.2K   │ │ $10K    │         │
│  └─────────┘ └─────────┘ └─────────┘         │
│                                                │
│  Filters: [Month ▼] [Campaign ▼] [Status ▼]  │
│                                                │
│  ┌──────────────────────────────────────────┐ │
│  │        Monthly Trend Chart               │ │
│  │  [Line chart: Spend vs Revenue]          │ │
│  └──────────────────────────────────────────┘ │
│                                                │
│  ┌──────────────────────────────────────────┐ │
│  │     Campaign Performance Table           │ │
│  │  Campaign | Spend | Revenue | ROAS       │ │
│  │  --------|-------|---------|-------       │ │
│  │  Prod A  | $8.2K | $24.6K  | 3.0x        │ │
│  │  Prod B  | $3.1K | $6.5K   | 2.1x        │ │
│  └──────────────────────────────────────────┘ │
│                                                │
│  [+ New Campaign]  [Enter Daily Data]         │
└────────────────────────────────────────────────┘
```

---

## ⚡ Performance Targets

| Metric           | Target | Notes                      |
| ---------------- | ------ | -------------------------- |
| Dashboard Load   | <2s    | Initial paint with data    |
| Query Execution  | <500ms | With 1 year of data        |
| Data Entry Save  | <1s    | Batch insert 30 records    |
| Chart Render     | <300ms | 12 months of data          |
| Bundle Size      | <500KB | Initial JS payload         |
| Lighthouse Score | >90    | Performance, Accessibility |

---

## 🐛 Common Issues & Fixes

### Issue: Clerk org switching lag

**Fix:** Use client-side caching with `useOrganization()` hook

### Issue: Slow dashboard with large datasets

**Fix:** Add database indexes, implement pagination, cache metrics

### Issue: Bonus calculation edge cases

**Fix:** Thoroughly test with `bonus.test.ts`, handle mid-month changes

### Issue: CSV import failures

**Fix:** Validate data before insert, show clear error messages

---

## 📝 Git Commit Convention

```
feat: Add campaign creation form
fix: Correct ROAS calculation for zero spend
refactor: Extract metrics service
docs: Update API documentation
test: Add bonus calculation tests
chore: Update dependencies
```

---

## 🔗 Quick Links

| Resource               | URL                                                |
| ---------------------- | -------------------------------------------------- |
| **Vercel Dashboard**   | [vercel.com/dashboard](https://vercel.com)         |
| **Clerk Dashboard**    | [dashboard.clerk.com](https://dashboard.clerk.com) |
| **Supabase Dashboard** | [app.supabase.com](https://app.supabase.com)       |
| **shadcn/ui Docs**     | [ui.shadcn.com](https://ui.shadcn.com)             |
| **Next.js Docs**       | [nextjs.org/docs](https://nextjs.org/docs)         |
| **Drizzle ORM Docs**   | [orm.drizzle.team](https://orm.drizzle.team)       |

---

## 📞 Emergency Contacts

```
Production Down:    [Check Vercel status]
Database Issues:    [Check Supabase status]
Auth Problems:      [Check Clerk status]
General Blockers:   [Slack #ad-analytics]
```

---

## ✅ Pre-Launch Checklist

```
Infrastructure
[ ] Vercel project created
[ ] Environment variables set
[ ] Database migrations applied
[ ] Clerk webhooks configured

Code Quality
[ ] All tests passing
[ ] No TypeScript errors
[ ] Linter passing
[ ] Bundle size < 500KB

Testing
[ ] Manual smoke test complete
[ ] All critical flows tested
[ ] Cross-browser tested
[ ] Responsive design verified

Documentation
[ ] README updated
[ ] API documented
[ ] User guide created

Monitoring
[ ] Sentry configured
[ ] Error alerts set up
[ ] Uptime monitoring active
```

---

## 📊 Sprint Goals Summary

| Week | Goal        | Key Deliverable          |
| ---- | ----------- | ------------------------ |
| 1    | Foundation  | Auth + DB setup          |
| 2    | Core Models | Campaign/Ad CRUD         |
| 3    | Data Entry  | Performance input + CSV  |
| 4    | Metrics     | ROAS calculation engine  |
| 5    | Dashboard   | Main UI with charts      |
| 6    | Deep Dive   | Campaign/Ad detail pages |
| 7    | Permissions | Role system complete     |
| 8    | Launch      | MVP in production        |
| 9    | Integration | Meta Ads connected       |
| 10   | Analytics   | Advanced features        |
| 11   | Collab      | Deep links + sharing     |
| 12   | Polish      | Phase 2 complete         |

---

## 🎓 Key Learnings / Best Practices

1. **Always validate on server** - Never trust client input
2. **Index early** - Add DB indexes before performance becomes an issue
3. **Cache aggressively** - Especially for historical data
4. **Test with real data** - Use actual Meta/Amazon data for validation
5. **Mobile-first forms** - Even though desktop-first overall
6. **Keyboard shortcuts** - Power users will thank you
7. **Loading states everywhere** - Better UX than waiting
8. **Error messages matter** - Be specific and helpful

---

## 💬 Anchored Comments Feature (Phase 3)

### Concept

Google Sheets/Figma-style comment pins anchored to dashboard elements via right-click.

### Quick Implementation

```typescript
// Wrap any dashboard element
<CommentPinsOverlay
  anchorType="chart"
  anchorTargetId="monthly-trend-chart"
  organizationId={orgId}
>
  <YourChartComponent />
</CommentPinsOverlay>
```

### Anchor Types

- `widget` - Dashboard cards/widgets
- `chart` - Charts with data point anchoring
- `tableCell` - Specific table cells (row + column)
- `metricCard` - Summary metric cards

### Data Model

```typescript
type Comment = {
  id: string;
  anchor: {
    kind: "widget" | "chart" | "tableCell" | "metricCard";
    targetId: string;
    metadata: {
      xPct?: number;
      yPct?: number;
      rowKey?: string;
      columnKey?: string;
    };
  };
  text: string;
  resolved: boolean;
  parentId?: string; // for replies
  mentions?: string[];
};
```

### Key Features

- Right-click to add comment
- Visual pins/hotspots with badges
- Threading with replies
- Resolve/unresolve
- @mentions with notifications
- Real-time updates (optional WebSocket)

---

**Print this page for quick reference during development!** 🚀
