# Technical Specifications - Ad Analytics Dashboard

## Quick Reference

**Stack:** Next.js 14 + Clerk + Supabase (Postgres) + shadcn/ui + Drizzle ORM  
**Timeline:** 8 weeks to MVP, 12 weeks to Phase 2  
**Primary Goal:** Calculate true ROAS including Amazon stepped bonuses

---

## Environment Setup

### Prerequisites
```bash
Node.js: v18+
pnpm: v8+ (preferred) or npm
Git
```

### Required Accounts
- [ ] Vercel account (hosting)
- [ ] Clerk account (auth)
- [ ] Supabase project (database)
- [ ] Meta Developer account (Phase 2)

### Environment Variables
```env
# .env.local
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_xxx
CLERK_SECRET_KEY=sk_xxx
NEXT_PUBLIC_CLERK_SIGN_IN_URL=/sign-in
NEXT_PUBLIC_CLERK_SIGN_UP_URL=/sign-up
NEXT_PUBLIC_CLERK_AFTER_SIGN_IN_URL=/dashboard
NEXT_PUBLIC_CLERK_AFTER_SIGN_UP_URL=/dashboard

DATABASE_URL=postgresql://xxx
DIRECT_URL=postgresql://xxx

# Phase 2
META_APP_ID=xxx
META_APP_SECRET=xxx
META_REDIRECT_URI=https://yourapp.com/api/integrations/meta/callback
```

---

## Project Initialization

```bash
# Create Next.js project
npx create-next-app@latest ad-analytics-dashboard --typescript --tailwind --app

# Install core dependencies
pnpm add @clerk/nextjs drizzle-orm postgres recharts
pnpm add -D drizzle-kit

# Install shadcn/ui
npx shadcn-ui@latest init

# Install specific shadcn components
npx shadcn-ui@latest add card table button dialog select calendar form input toast badge dropdown-menu navigation-menu sheet
```

---

## Database Schema (Drizzle)

### File: `/lib/db/schema.ts`

```typescript
import { pgTable, uuid, varchar, text, timestamp, decimal, integer, date, pgEnum, jsonb, unique } from 'drizzle-orm/pg-core'
import { sql } from 'drizzle-orm'

// Enums
export const platformEnum = pgEnum('platform', ['meta', 'google', 'tiktok', 'other'])
export const statusEnum = pgEnum('status', ['active', 'paused', 'archived'])
export const dataSourceEnum = pgEnum('data_source', ['manual', 'meta_api', 'google_api'])
export const roleEnum = pgEnum('role', ['owner', 'editor', 'ad_manager', 'finance', 'viewer'])
export const calculationTypeEnum = pgEnum('calculation_type', ['incremental', 'replacement'])

// Organizations
export const organizations = pgTable('organizations', {
  id: uuid('id').primaryKey().defaultRandom(),
  name: varchar('name', { length: 255 }).notNull(),
  settings: jsonb('settings').default({}),
  createdAt: timestamp('created_at').defaultNow().notNull(),
})

// Campaigns
export const campaigns = pgTable('campaigns', {
  id: uuid('id').primaryKey().defaultRandom(),
  organizationId: uuid('organization_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  name: varchar('name', { length: 255 }).notNull(),
  platform: platformEnum('platform').notNull(),
  status: statusEnum('status').default('active').notNull(),
  startDate: date('start_date').notNull(),
  endDate: date('end_date'),
  createdBy: varchar('created_by', { length: 255 }), // Clerk user ID
  metadata: jsonb('metadata').default({}),
  createdAt: timestamp('created_at').defaultNow().notNull(),
})

// Ads
export const ads = pgTable('ads', {
  id: uuid('id').primaryKey().defaultRandom(),
  campaignId: uuid('campaign_id').references(() => campaigns.id, { onDelete: 'cascade' }).notNull(),
  externalId: varchar('external_id', { length: 255 }), // For API integrations
  name: varchar('name', { length: 255 }).notNull(),
  platformAdType: varchar('platform_ad_type', { length: 100 }),
  status: statusEnum('status').default('active').notNull(),
  metadata: jsonb('metadata').default({}),
  createdAt: timestamp('created_at').defaultNow().notNull(),
})

// Daily Performance (Fact Table)
export const dailyPerformance = pgTable('daily_performance', {
  id: uuid('id').primaryKey().defaultRandom(),
  adId: uuid('ad_id').references(() => ads.id, { onDelete: 'cascade' }).notNull(),
  date: date('date').notNull(),
  spend: decimal('spend', { precision: 12, scale: 2 }).notNull().default('0'),
  impressions: integer('impressions').default(0),
  clicks: integer('clicks').default(0),
  conversions: integer('conversions').default(0),
  affiliateRevenue: decimal('affiliate_revenue', { precision: 12, scale: 2 }).notNull().default('0'),
  dataSource: dataSourceEnum('data_source').default('manual').notNull(),
  notes: text('notes'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, (table) => ({
  uniqueAdDateSource: unique().on(table.adId, table.date, table.dataSource),
}))

// Shipped Revenue
export const shippedRevenue = pgTable('shipped_revenue', {
  id: uuid('id').primaryKey().defaultRandom(),
  organizationId: uuid('organization_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  affiliateNetwork: varchar('affiliate_network', { length: 100 }).notNull(),
  month: date('month').notNull(), // First day of month
  shippedRevenue: decimal('shipped_revenue', { precision: 12, scale: 2 }).notNull().default('0'),
  commissionRevenue: decimal('commission_revenue', { precision: 12, scale: 2 }).notNull().default('0'),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
}, (table) => ({
  uniqueOrgNetworkMonth: unique().on(table.organizationId, table.affiliateNetwork, table.month),
}))

// Bonus Structures
export const bonusStructures = pgTable('bonus_structures', {
  id: uuid('id').primaryKey().defaultRandom(),
  organizationId: uuid('organization_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  affiliateNetwork: varchar('affiliate_network', { length: 100 }).notNull(),
  effectiveFrom: date('effective_from').notNull(),
  effectiveTo: date('effective_to'),
  tiers: jsonb('tiers').notNull(), // [{"threshold": 1000, "bonus": 100}, ...]
  calculationType: calculationTypeEnum('calculation_type').default('incremental').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
})

// User Roles (augments Clerk)
export const userRoles = pgTable('user_roles', {
  userId: varchar('user_id', { length: 255 }).notNull(), // Clerk user ID
  organizationId: uuid('organization_id').references(() => organizations.id, { onDelete: 'cascade' }).notNull(),
  role: roleEnum('role').default('viewer').notNull(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
}, (table) => ({
  uniqueUserOrg: unique().on(table.userId, table.organizationId),
}))
```

### Migrations

```bash
# Generate migration
pnpm drizzle-kit generate:pg

# Apply migration
pnpm drizzle-kit push:pg

# Open Drizzle Studio
pnpm drizzle-kit studio
```

---

## Core Service Functions

### File: `/lib/services/metrics.ts`

```typescript
import { db } from '@/lib/db'
import { dailyPerformance, shippedRevenue, bonusStructures, ads, campaigns } from '@/lib/db/schema'
import { eq, and, gte, lte, sql } from 'drizzle-orm'

export interface ROASMetrics {
  spend: number
  revenue: number
  shippedRevenue: number
  bonusEarned: number
  basicROAS: number
  trueROAS: number
  projectedSpend?: number
  projectedRevenue?: number
  projectedBonus?: number
  nextThreshold?: number
  progressToNextThreshold?: number
}

export async function calculateMonthMetrics(
  organizationId: string,
  month: Date,
  includeProjections = false
): Promise<ROASMetrics> {
  
  const startOfMonth = new Date(month.getFullYear(), month.getMonth(), 1)
  const endOfMonth = new Date(month.getFullYear(), month.getMonth() + 1, 0)
  const today = new Date()
  
  // Get performance data for the month
  const perfData = await db
    .select({
      totalSpend: sql<number>`COALESCE(SUM(${dailyPerformance.spend}), 0)`,
      totalRevenue: sql<number>`COALESCE(SUM(${dailyPerformance.affiliateRevenue}), 0)`,
    })
    .from(dailyPerformance)
    .innerJoin(ads, eq(dailyPerformance.adId, ads.id))
    .innerJoin(campaigns, eq(ads.campaignId, campaigns.id))
    .where(
      and(
        eq(campaigns.organizationId, organizationId),
        gte(dailyPerformance.date, startOfMonth),
        lte(dailyPerformance.date, endOfMonth)
      )
    )
  
  // Get shipped revenue for the month
  const shipped = await db
    .select()
    .from(shippedRevenue)
    .where(
      and(
        eq(shippedRevenue.organizationId, organizationId),
        eq(shippedRevenue.month, startOfMonth)
      )
    )
  
  const totalShipped = shipped.reduce((sum, row) => sum + parseFloat(row.shippedRevenue.toString()), 0)
  
  // Get applicable bonus structure
  const bonusStructure = await db
    .select()
    .from(bonusStructures)
    .where(
      and(
        eq(bonusStructures.organizationId, organizationId),
        lte(bonusStructures.effectiveFrom, startOfMonth),
        // effectiveTo is null or >= startOfMonth
      )
    )
    .orderBy(bonusStructures.effectiveFrom)
    .limit(1)
  
  const bonusEarned = bonusStructure[0] 
    ? calculateBonusFromTiers(totalShipped, bonusStructure[0].tiers as any, bonusStructure[0].calculationType)
    : 0
  
  const spend = parseFloat(perfData[0].totalSpend.toString())
  const revenue = parseFloat(perfData[0].totalRevenue.toString())
  
  let result: ROASMetrics = {
    spend,
    revenue,
    shippedRevenue: totalShipped,
    bonusEarned,
    basicROAS: spend > 0 ? revenue / spend : 0,
    trueROAS: spend > 0 ? (revenue + bonusEarned) / spend : 0,
  }
  
  // Add projections if current month and requested
  if (includeProjections && month.getMonth() === today.getMonth()) {
    const daysInMonth = endOfMonth.getDate()
    const daysPassed = today.getDate()
    const daysRemaining = daysInMonth - daysPassed
    
    const avgDailySpend = spend / daysPassed
    const avgDailyRevenue = revenue / daysPassed
    const avgDailyShipped = totalShipped / daysPassed
    
    result.projectedSpend = spend + (avgDailySpend * daysRemaining)
    result.projectedRevenue = revenue + (avgDailyRevenue * daysRemaining)
    
    const projectedShipped = totalShipped + (avgDailyShipped * daysRemaining)
    result.projectedBonus = bonusStructure[0]
      ? calculateBonusFromTiers(projectedShipped, bonusStructure[0].tiers as any, bonusStructure[0].calculationType)
      : 0
    
    // Calculate next threshold
    if (bonusStructure[0]) {
      const tiers = bonusStructure[0].tiers as Array<{threshold: number, bonus: number}>
      const nextTier = tiers.find(t => t.threshold > totalShipped)
      if (nextTier) {
        result.nextThreshold = nextTier.threshold
        result.progressToNextThreshold = (totalShipped / nextTier.threshold) * 100
      }
    }
  }
  
  return result
}

function calculateBonusFromTiers(
  shippedAmount: number,
  tiers: Array<{threshold: number, bonus: number}>,
  type: 'incremental' | 'replacement'
): number {
  const sortedTiers = [...tiers].sort((a, b) => a.threshold - b.threshold)
  
  if (type === 'replacement') {
    // Find highest tier we've reached
    const applicableTier = sortedTiers.reverse().find(t => shippedAmount >= t.threshold)
    return applicableTier?.bonus || 0
  } else {
    // Incremental: sum all tiers we've passed
    return sortedTiers
      .filter(t => shippedAmount >= t.threshold)
      .reduce((sum, t) => sum + t.bonus, 0)
  }
}

export async function getCampaignPerformance(
  organizationId: string,
  startDate: Date,
  endDate: Date
) {
  return await db
    .select({
      campaignId: campaigns.id,
      campaignName: campaigns.name,
      platform: campaigns.platform,
      status: campaigns.status,
      totalSpend: sql<number>`COALESCE(SUM(${dailyPerformance.spend}), 0)`,
      totalRevenue: sql<number>`COALESCE(SUM(${dailyPerformance.affiliateRevenue}), 0)`,
      totalImpressions: sql<number>`COALESCE(SUM(${dailyPerformance.impressions}), 0)`,
      totalClicks: sql<number>`COALESCE(SUM(${dailyPerformance.clicks}), 0)`,
    })
    .from(campaigns)
    .leftJoin(ads, eq(ads.campaignId, campaigns.id))
    .leftJoin(dailyPerformance, eq(dailyPerformance.adId, ads.id))
    .where(
      and(
        eq(campaigns.organizationId, organizationId),
        gte(dailyPerformance.date, startDate),
        lte(dailyPerformance.date, endDate)
      )
    )
    .groupBy(campaigns.id, campaigns.name, campaigns.platform, campaigns.status)
    .orderBy(sql`SUM(${dailyPerformance.spend}) DESC`)
}
```

---

## Permission System

### File: `/lib/auth/permissions.ts`

```typescript
import { auth } from '@clerk/nextjs'
import { db } from '@/lib/db'
import { userRoles } from '@/lib/db/schema'
import { eq, and } from 'drizzle-orm'

export enum Permission {
  VIEW_DASHBOARD = 'view_dashboard',
  EDIT_CAMPAIGNS = 'edit_campaigns',
  ENTER_DATA = 'enter_data',
  MANAGE_BONUSES = 'manage_bonuses',
  MANAGE_ORG = 'manage_org',
  MANAGE_INTEGRATIONS = 'manage_integrations',
  MANAGE_ROLES = 'manage_roles',
}

export type Role = 'owner' | 'editor' | 'ad_manager' | 'finance' | 'viewer'

const rolePermissions: Record<Role, Permission[]> = {
  owner: Object.values(Permission),
  editor: [
    Permission.VIEW_DASHBOARD,
    Permission.EDIT_CAMPAIGNS,
    Permission.ENTER_DATA,
  ],
  ad_manager: [
    Permission.VIEW_DASHBOARD,
    Permission.ENTER_DATA,
  ],
  finance: [
    Permission.VIEW_DASHBOARD,
    Permission.MANAGE_BONUSES,
  ],
  viewer: [
    Permission.VIEW_DASHBOARD,
  ],
}

export async function getUserRole(userId: string, organizationId: string): Promise<Role | null> {
  const role = await db
    .select()
    .from(userRoles)
    .where(and(
      eq(userRoles.userId, userId),
      eq(userRoles.organizationId, organizationId)
    ))
    .limit(1)
  
  return role[0]?.role || null
}

export async function hasPermission(
  userId: string,
  organizationId: string,
  permission: Permission
): Promise<boolean> {
  const role = await getUserRole(userId, organizationId)
  if (!role) return false
  
  return rolePermissions[role].includes(permission)
}

export async function requirePermission(
  organizationId: string,
  permission: Permission
): Promise<void> {
  const { userId } = auth()
  if (!userId) throw new Error('Unauthorized')
  
  const allowed = await hasPermission(userId, organizationId, permission)
  if (!allowed) throw new Error('Forbidden')
}

// Server Component helper
export async function checkPermission(organizationId: string, permission: Permission) {
  const { userId } = auth()
  if (!userId) return false
  return hasPermission(userId, organizationId, permission)
}
```

---

## API Routes & Server Actions

### File: `/app/actions/campaigns.ts`

```typescript
'use server'

import { auth } from '@clerk/nextjs'
import { revalidatePath } from 'next/cache'
import { db } from '@/lib/db'
import { campaigns, ads } from '@/lib/db/schema'
import { requirePermission, Permission } from '@/lib/auth/permissions'
import { eq } from 'drizzle-orm'

export async function createCampaign(data: {
  organizationId: string
  name: string
  platform: 'meta' | 'google' | 'tiktok' | 'other'
  startDate: Date
}) {
  const { userId } = auth()
  if (!userId) throw new Error('Unauthorized')
  
  await requirePermission(data.organizationId, Permission.EDIT_CAMPAIGNS)
  
  const [campaign] = await db.insert(campaigns).values({
    ...data,
    createdBy: userId,
  }).returning()
  
  revalidatePath('/dashboard')
  return campaign
}

export async function updateCampaign(
  campaignId: string,
  data: Partial<{
    name: string
    status: 'active' | 'paused' | 'archived'
    endDate: Date | null
  }>
) {
  const { userId } = auth()
  if (!userId) throw new Error('Unauthorized')
  
  // Get campaign to check org
  const [campaign] = await db.select().from(campaigns).where(eq(campaigns.id, campaignId))
  if (!campaign) throw new Error('Campaign not found')
  
  await requirePermission(campaign.organizationId, Permission.EDIT_CAMPAIGNS)
  
  const [updated] = await db
    .update(campaigns)
    .set(data)
    .where(eq(campaigns.id, campaignId))
    .returning()
  
  revalidatePath('/dashboard')
  revalidatePath(`/campaigns/${campaignId}`)
  return updated
}

export async function deleteCampaign(campaignId: string) {
  const { userId } = auth()
  if (!userId) throw new Error('Unauthorized')
  
  const [campaign] = await db.select().from(campaigns).where(eq(campaigns.id, campaignId))
  if (!campaign) throw new Error('Campaign not found')
  
  await requirePermission(campaign.organizationId, Permission.EDIT_CAMPAIGNS)
  
  await db.delete(campaigns).where(eq(campaigns.id, campaignId))
  
  revalidatePath('/dashboard')
}
```

### File: `/app/actions/performance.ts`

```typescript
'use server'

import { auth } from '@clerk/nextjs'
import { revalidatePath } from 'next/cache'
import { db } from '@/lib/db'
import { dailyPerformance, ads, campaigns } from '@/lib/db/schema'
import { requirePermission, Permission } from '@/lib/auth/permissions'
import { eq, and } from 'drizzle-orm'

export async function saveDailyPerformance(data: Array<{
  adId: string
  date: Date
  spend: number
  impressions?: number
  clicks?: number
  conversions?: number
  affiliateRevenue: number
  notes?: string
}>) {
  const { userId } = auth()
  if (!userId) throw new Error('Unauthorized')
  
  // Check permission for first ad's org
  const [ad] = await db.select().from(ads).where(eq(ads.id, data[0].adId))
  if (!ad) throw new Error('Ad not found')
  
  const [campaign] = await db.select().from(campaigns).where(eq(campaigns.id, ad.campaignId))
  if (!campaign) throw new Error('Campaign not found')
  
  await requirePermission(campaign.organizationId, Permission.ENTER_DATA)
  
  // Upsert performance data
  for (const row of data) {
    await db
      .insert(dailyPerformance)
      .values({
        ...row,
        dataSource: 'manual',
      })
      .onConflictDoUpdate({
        target: [dailyPerformance.adId, dailyPerformance.date, dailyPerformance.dataSource],
        set: {
          spend: row.spend,
          impressions: row.impressions || 0,
          clicks: row.clicks || 0,
          conversions: row.conversions || 0,
          affiliateRevenue: row.affiliateRevenue,
          notes: row.notes,
          updatedAt: new Date(),
        },
      })
  }
  
  revalidatePath('/dashboard')
  return { success: true }
}
```

---

## UI Component Examples

### File: `/components/dashboard/summary-cards.tsx`

```typescript
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card'
import { TrendingUp, TrendingDown } from 'lucide-react'
import { ROASMetrics } from '@/lib/services/metrics'

export function SummaryCards({ metrics }: { metrics: ROASMetrics }) {
  return (
    <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
      <Card>
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium">Current ROAS</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="text-2xl font-bold">{metrics.basicROAS.toFixed(2)}x</div>
          <p className="text-xs text-muted-foreground">Without bonuses</p>
        </CardContent>
      </Card>
      
      <Card>
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium">True ROAS</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="text-2xl font-bold text-green-600">{metrics.trueROAS.toFixed(2)}x</div>
          <p className="text-xs text-muted-foreground">
            +${metrics.bonusEarned.toLocaleString()} bonus
          </p>
        </CardContent>
      </Card>
      
      <Card>
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium">Spend MTD</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="text-2xl font-bold">${metrics.spend.toLocaleString()}</div>
          {metrics.projectedSpend && (
            <p className="text-xs text-muted-foreground">
              Proj: ${metrics.projectedSpend.toLocaleString()}
            </p>
          )}
        </CardContent>
      </Card>
      
      <Card>
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium">Revenue MTD</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="text-2xl font-bold">${metrics.revenue.toLocaleString()}</div>
          {metrics.projectedRevenue && (
            <p className="text-xs text-muted-foreground">
              Proj: ${metrics.projectedRevenue.toLocaleString()}
            </p>
          )}
        </CardContent>
      </Card>
      
      <Card>
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium">Shipped Revenue</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="text-2xl font-bold">${metrics.shippedRevenue.toLocaleString()}</div>
          {metrics.progressToNextThreshold && (
            <p className="text-xs text-muted-foreground">
              {metrics.progressToNextThreshold.toFixed(0)}% to next threshold
            </p>
          )}
        </CardContent>
      </Card>
      
      <Card>
        <CardHeader className="flex flex-row items-center justify-between space-y-0 pb-2">
          <CardTitle className="text-sm font-medium">Next Bonus</CardTitle>
        </CardHeader>
        <CardContent>
          <div className="text-2xl font-bold">
            {metrics.nextThreshold ? `$${metrics.nextThreshold.toLocaleString()}` : 'Max reached'}
          </div>
          <p className="text-xs text-muted-foreground">
            {metrics.nextThreshold 
              ? `$${(metrics.nextThreshold - metrics.shippedRevenue).toLocaleString()} to go`
              : 'Highest tier achieved'}
          </p>
        </CardContent>
      </Card>
    </div>
  )
}
```

### File: `/components/dashboard/data-entry-modal.tsx`

```typescript
'use client'

import { useState } from 'react'
import { Dialog, DialogContent, DialogHeader, DialogTitle } from '@/components/ui/dialog'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Calendar } from '@/components/ui/calendar'
import { saveDailyPerformance } from '@/app/actions/performance'
import { useToast } from '@/components/ui/use-toast'

interface Ad {
  id: string
  name: string
  campaignName: string
}

export function DataEntryModal({ 
  ads, 
  open, 
  onClose 
}: { 
  ads: Ad[], 
  open: boolean, 
  onClose: () => void 
}) {
  const [date, setDate] = useState<Date>(new Date())
  const [data, setData] = useState<Record<string, { spend: string, revenue: string }>>({})
  const { toast } = useToast()
  
  const handleSave = async () => {
    const entries = Object.entries(data).map(([adId, values]) => ({
      adId,
      date,
      spend: parseFloat(values.spend) || 0,
      affiliateRevenue: parseFloat(values.revenue) || 0,
    })).filter(e => e.spend > 0 || e.affiliateRevenue > 0)
    
    if (entries.length === 0) {
      toast({ title: 'No data entered', variant: 'destructive' })
      return
    }
    
    await saveDailyPerformance(entries)
    toast({ title: 'Performance data saved' })
    onClose()
  }
  
  return (
    <Dialog open={open} onOpenChange={onClose}>
      <DialogContent className="max-w-4xl max-h-[80vh] overflow-y-auto">
        <DialogHeader>
          <DialogTitle>Enter Daily Performance</DialogTitle>
        </DialogHeader>
        
        <div className="space-y-4">
          <div>
            <label className="text-sm font-medium">Date</label>
            <Calendar
              mode="single"
              selected={date}
              onSelect={(d) => d && setDate(d)}
              disabled={(date) => date > new Date()}
            />
          </div>
          
          <div className="border rounded-lg">
            <table className="w-full">
              <thead>
                <tr className="border-b">
                  <th className="text-left p-2">Ad</th>
                  <th className="text-left p-2">Campaign</th>
                  <th className="text-left p-2">Spend ($)</th>
                  <th className="text-left p-2">Revenue ($)</th>
                </tr>
              </thead>
              <tbody>
                {ads.map(ad => (
                  <tr key={ad.id} className="border-b">
                    <td className="p-2">{ad.name}</td>
                    <td className="p-2 text-sm text-muted-foreground">{ad.campaignName}</td>
                    <td className="p-2">
                      <Input
                        type="number"
                        step="0.01"
                        placeholder="0.00"
                        value={data[ad.id]?.spend || ''}
                        onChange={(e) => setData(prev => ({
                          ...prev,
                          [ad.id]: { ...prev[ad.id], spend: e.target.value }
                        }))}
                      />
                    </td>
                    <td className="p-2">
                      <Input
                        type="number"
                        step="0.01"
                        placeholder="0.00"
                        value={data[ad.id]?.revenue || ''}
                        onChange={(e) => setData(prev => ({
                          ...prev,
                          [ad.id]: { ...prev[ad.id], revenue: e.target.value }
                        }))}
                      />
                    </td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
          
          <div className="flex justify-end gap-2">
            <Button variant="outline" onClick={onClose}>Cancel</Button>
            <Button onClick={handleSave}>Save</Button>
          </div>
        </div>
      </DialogContent>
    </Dialog>
  )
}
```

---

## Testing Strategy

### Unit Tests
- Bonus calculation logic (all tier types)
- ROAS computation
- Projection algorithms
- Permission checks

### Integration Tests
- Campaign CRUD operations
- Data entry workflows
- Metric aggregation queries

### E2E Tests (Playwright)
- Full user journey: create campaign → add ads → enter data → view dashboard
- Permission enforcement
- Data entry form submission

---

## Performance Considerations

### Database Indexes (Critical)
```sql
CREATE INDEX idx_daily_perf_ad_date ON daily_performance(ad_id, date DESC);
CREATE INDEX idx_daily_perf_org_date ON daily_performance(date) 
  WHERE date >= CURRENT_DATE - INTERVAL '24 months';
CREATE INDEX idx_ads_campaign ON ads(campaign_id) WHERE status != 'archived';
CREATE INDEX idx_campaigns_org ON campaigns(organization_id) WHERE status != 'archived';
```

### Caching Strategy
- Dashboard metrics: Cache for 5 minutes
- Campaign lists: Cache until mutation
- Historical data (>30 days old): Cache aggressively

### Query Optimization
- Use materialized views for monthly aggregates
- Paginate campaign/ad lists (50 per page)
- Limit historical queries to 24 months by default

---

## Deployment Checklist

### Pre-Launch
- [ ] Set up Vercel project with env vars
- [ ] Run all migrations on production DB
- [ ] Seed initial bonus structure
- [ ] Test Clerk auth flow end-to-end
- [ ] Configure error monitoring (Sentry)
- [ ] Set up uptime monitoring
- [ ] Test on multiple browsers
- [ ] Verify mobile/tablet responsiveness

### Launch Day
- [ ] Deploy to production
- [ ] Smoke test all critical flows
- [ ] Monitor error rates
- [ ] Have rollback plan ready

### Post-Launch
- [ ] Collect user feedback
- [ ] Monitor performance metrics
- [ ] Iterate on data entry UX
- [ ] Plan Phase 2 features

---

**End of Technical Specifications**


