# Ad Analytics Dashboard - Project Documentation

> A comprehensive web dashboard for analyzing ad spend vs. revenue/bonuses for social-media-driven affiliate businesses.

## 📋 Documentation Overview

This project contains complete planning documentation organized into three main documents:

### 1. **[PROJECT_PLAN.md](./PROJECT_PLAN.md)** - Strategic Overview
**Read this first** for the big picture.

Contains:
- Executive summary and assumptions
- User personas and core requirements
- 3-phase development roadmap (MVP → Enhanced → Scale)
- Complete data model and metrics definitions
- UX/UI design specifications
- Technical architecture and stack decisions
- Success metrics and risk assessment

**Best for:** Understanding the "what" and "why" of the project.

---

### 2. **[TECHNICAL_SPECS.md](./TECHNICAL_SPECS.md)** - Implementation Details
**Read this for hands-on development.**

Contains:
- Exact tech stack with justifications
- Complete database schema (Drizzle ORM)
- Code examples for services, actions, and components
- Permission system implementation
- API patterns and Server Actions
- Performance optimization strategies
- Deployment checklist

**Best for:** Developers building the actual features.

---

### 3. **[IMPLEMENTATION_ROADMAP.md](./IMPLEMENTATION_ROADMAP.md)** - Week-by-Week Tasks
**Read this for project management.**

Contains:
- Detailed week-by-week breakdown (12 weeks)
- Time estimates for each task (320-400 hours total)
- Specific files to create
- Acceptance criteria for each phase
- Risk mitigation strategies
- Developer workflow guidelines

**Best for:** Tracking progress and managing timelines.

---

## 🚀 Quick Start Guide

### For Project Stakeholders
1. Read **Executive Summary** in [PROJECT_PLAN.md](./PROJECT_PLAN.md)
2. Review **User Personas** (section 2)
3. Understand **MVP Success Metrics** (section 9)
4. Check **Timeline**: 8 weeks to MVP, 12 weeks to Phase 2

### For Product Managers
1. Study **Phased Roadmap** (PROJECT_PLAN.md, section 3)
2. Review **UX/UI Design** (PROJECT_PLAN.md, section 5)
3. Understand **Open Questions** (PROJECT_PLAN.md, section 10)
4. Use **Implementation Roadmap** for sprint planning

### For Developers
1. Review **Tech Stack** (TECHNICAL_SPECS.md)
2. Study **Database Schema** (TECHNICAL_SPECS.md)
3. Examine **Code Examples** (TECHNICAL_SPECS.md)
4. Follow **Week 1 Tasks** (IMPLEMENTATION_ROADMAP.md)

### For Designers
1. Review **UI Component Mapping** (PROJECT_PLAN.md, section 5)
2. Study **Dashboard Layout** (PROJECT_PLAN.md, section 5)
3. Understand **Key Flows** (PROJECT_PLAN.md, section 5)
4. Reference **shadcn/ui** components list

---

## 🎯 Project Goals

### Primary Objectives
1. ✅ Calculate **true ROAS** including Amazon stepped bonuses
2. ✅ Enable **daily ad decisions** (scale/cut/test)
3. ✅ Fast **data entry** (<15 min/day)
4. ✅ **Extensible** architecture for future tools

### Success Criteria (MVP)
- Dashboard answers "are ads profitable?" in <30 seconds
- Data entry for 30 days takes <15 minutes
- Accurate bonus calculations with real-time progress tracking
- 3+ team members actively using the platform

---

## 📊 Project Scope

### In Scope (MVP - 8 weeks)
- ✅ Campaign & ad management
- ✅ Manual daily data entry
- ✅ Bonus threshold configuration
- ✅ ROAS calculations (basic + true)
- ✅ Monthly projections
- ✅ Role-based permissions
- ✅ Main dashboard with charts

### In Scope (Phase 2 - 12 weeks)
- ✅ Meta Ads API integration
- ✅ Advanced analytics & comparisons
- ✅ Scenario modeling
- ✅ PDF reports & exports
- ✅ Deep links & sharing
- ✅ Enhanced collaboration

### Future (Phase 3+)
- Google Ads, TikTok Ads integrations
- Additional affiliate networks
- ML-powered predictions
- Creative testing module
- Content calendar module

---

## 🛠️ Tech Stack

```
Frontend:  Next.js 14 (App Router)
UI:        shadcn/ui + Tailwind CSS
Auth:      Clerk (with Organizations)
Database:  PostgreSQL (via Supabase)
ORM:       Drizzle ORM
Charts:    Recharts
Hosting:   Vercel
```

**Why this stack?**
- **Next.js 14:** Server Components for performance, App Router for modern patterns
- **Clerk:** Built-in organizations, roles, and beautiful auth UI
- **Supabase:** Managed Postgres with generous free tier and great DX
- **Drizzle:** Lightweight ORM with excellent TypeScript support
- **shadcn/ui:** Beautiful, customizable components that we own

See [TECHNICAL_SPECS.md](./TECHNICAL_SPECS.md) for detailed justifications.

---

## 📅 Timeline Summary

| Phase | Duration | Deliverable |
|-------|----------|-------------|
| **Week 0** | 5-8 hours | Environment setup |
| **Weeks 1-2** | 52-68 hours | Foundation (auth, DB, CRUD) |
| **Weeks 3-4** | 56-72 hours | Data entry & metrics engine |
| **Weeks 5-6** | 54-70 hours | Dashboard UI & deep dives |
| **Weeks 7-8** | 50-66 hours | Permissions, testing, launch |
| **MVP Complete** | **~320 hours** | **Production-ready v1.0** |
| **Weeks 9-12** | 100-128 hours | Meta integration, advanced features |
| **Phase 2 Complete** | **~420 hours** | **Enhanced analytics v1.1** |

---

## 🗺️ Key Features Breakdown

### Core Features (MVP)
1. **Campaign & Ad Management**
   - Create, edit, delete campaigns
   - Manage ads within campaigns
   - Track status (active/paused/archived)

2. **Data Entry**
   - Fast daily performance input
   - CSV import for historical data
   - Bulk editing capabilities

3. **Metrics & Analytics**
   - Basic ROAS (revenue / spend)
   - True ROAS (including bonuses)
   - Month-to-date vs. projected
   - Campaign performance comparison

4. **Bonus Modeling**
   - Configure stepped thresholds
   - Track progress to next tier
   - Preview bonus calculations

5. **Dashboard**
   - 6 key metric cards
   - Monthly trend chart
   - Campaign performance table
   - Filters by date, campaign, status

6. **Permissions**
   - 5 roles (Owner, Editor, Ad Manager, Finance, Viewer)
   - Permission-based UI rendering
   - Protected actions via Server Actions

### Enhanced Features (Phase 2)
7. **Meta Ads Integration**
   - OAuth connection
   - Automated daily sync
   - Conflict resolution

8. **Advanced Analytics**
   - Stacked charts
   - Campaign comparisons
   - Scenario modeling
   - PDF reports

9. **Collaboration**
   - Deep links with filters
   - Saved views
   - Comments & notes (v2.1)

---

## 📐 Data Model (Simplified)

```
Organizations
├── Users (via Clerk)
├── Campaigns
│   └── Ads
│       └── DailyPerformance (fact table)
├── ShippedRevenue (monthly)
├── BonusStructures (tiers)
└── UserRoles (permissions)
```

**Key Relationships:**
- Organization has many Campaigns
- Campaign has many Ads
- Ad has many DailyPerformance records (grain: ad × date)
- Organization has ShippedRevenue per month per affiliate network
- BonusStructures define stepped bonus tiers

See [TECHNICAL_SPECS.md](./TECHNICAL_SPECS.md) for complete schema.

---

## 🎨 Design Principles

1. **Speed of Understanding**: Big numbers first, details on demand
2. **Speed of Entry**: Optimize for keyboard users, minimize clicks
3. **Clarity**: Always show both "basic ROAS" and "true ROAS"
4. **Context**: Use color sparingly (green = good, red = concerning)
5. **Responsive**: Desktop-first (1440px optimal), tablet-friendly

### Key UI Components
- Summary cards (6 metrics at top)
- Time-series chart (12-month trend)
- Data table (sortable, filterable)
- Quick entry modal (spreadsheet-like)
- Filter sidebar (date, campaign, status)

---

## 📈 Success Metrics

### Week 8 (MVP Launch)
- ⏱️ Dashboard load: <2 seconds
- 👥 1+ organization using daily
- ⌨️ Data entry: <15 min/day
- 🎯 User satisfaction: >4/5 stars
- 🐛 Critical bugs: 0

### Week 12 (Phase 2)
- 🔄 Meta sync reduces manual entry by 80%
- 👥 3+ active users per org
- 🔗 Deep links shared regularly
- 📊 Advanced features adopted
- 📈 90%+ month-over-month retention

### 6 Months (Long-term)
- 🏢 5+ organizations active
- 🌐 2+ affiliate networks integrated
- 💬 Collaboration features in use
- 📊 Platform informs 90%+ of ad decisions
- 💰 Documented ROI vs. previous process

---

## 🚧 Current Status

**Phase:** Planning Complete ✅  
**Next Steps:**
1. Review and validate assumptions
2. Answer open questions (see PROJECT_PLAN.md, section 10)
3. Set up development environment
4. Begin Week 1 implementation

---

## 🤝 Team Roles

### Required Roles
- **Full-Stack Developer** (1-2): Next.js, TypeScript, SQL
- **Designer** (0.5): UI/UX design and feedback (can use shadcn defaults)
- **Product Owner** (0.25): Requirements, testing, feedback

### Optional Roles
- **DevOps** (0.1): CI/CD setup (Vercel handles most)
- **Data Analyst** (0.1): Validate metric calculations

---

## 📚 Additional Resources

### External Documentation
- [Next.js 14 Docs](https://nextjs.org/docs)
- [Clerk Organizations Guide](https://clerk.com/docs/organizations/overview)
- [Drizzle ORM Docs](https://orm.drizzle.team/docs/overview)
- [shadcn/ui Components](https://ui.shadcn.com/)
- [Supabase Docs](https://supabase.com/docs)
- [Meta Marketing API](https://developers.facebook.com/docs/marketing-apis)

### Internal Files
- [Plan Prompt.md](./Plan%20Prompt.md) - Original requirements
- [PROJECT_PLAN.md](./PROJECT_PLAN.md) - Strategic overview
- [TECHNICAL_SPECS.md](./TECHNICAL_SPECS.md) - Implementation details
- [IMPLEMENTATION_ROADMAP.md](./IMPLEMENTATION_ROADMAP.md) - Week-by-week tasks

---

## ❓ FAQ

### Q: Why Next.js over Remix/SvelteKit?
**A:** Next.js 14 with Server Components offers the best balance of performance, DX, and ecosystem maturity. Vercel deployment is seamless.

### Q: Why Supabase over plain Postgres?
**A:** Supabase provides managed Postgres with automatic backups, connection pooling, and a generous free tier. Easy to migrate away if needed.

### Q: Can we use Prisma instead of Drizzle?
**A:** Yes, but Drizzle is lighter and has better TS inference. See TECHNICAL_SPECS.md for details.

### Q: Mobile app needed?
**A:** Not for MVP. Web-first approach is sufficient. Responsive design handles tablet usage.

### Q: How extensible is this for future tools?
**A:** Very. Module registry pattern allows bolt-on tools without touching core code. See TECHNICAL_SPECS.md section 6.

### Q: What if bonus structure changes mid-month?
**A:** Bonus structures have `effective_from` and `effective_to` dates. System will use the correct structure for each date range.

---

## 📞 Contact & Support

**Project Owner:** [To be assigned]  
**Tech Lead:** [To be assigned]  
**Repository:** [GitHub link]  
**Slack Channel:** [#ad-analytics]

---

## 📝 Document Changelog

| Date | Version | Changes |
|------|---------|---------|
| 2025-11-24 | 1.0 | Initial comprehensive project plan |

---

## ✅ Next Actions

### Immediate (This Week)
1. [ ] Review all three planning documents
2. [ ] Answer open questions in PROJECT_PLAN.md section 10
3. [ ] Validate assumptions with stakeholders
4. [ ] Set up GitHub repository
5. [ ] Create Vercel, Clerk, and Supabase accounts

### This Month (Week 1-4)
1. [ ] Complete environment setup (Week 0)
2. [ ] Build foundation (Weeks 1-2)
3. [ ] Implement data layer (Weeks 3-4)
4. [ ] Review progress and adjust if needed

### This Quarter (Months 1-3)
1. [ ] Ship MVP to first users (Week 8)
2. [ ] Gather feedback and iterate
3. [ ] Begin Phase 2 development (Weeks 9-12)
4. [ ] Plan Phase 3 features

---

**Ready to build?** Start with [IMPLEMENTATION_ROADMAP.md](./IMPLEMENTATION_ROADMAP.md) Week 1 tasks! 🚀


