You are a senior product strategist, growth marketer, UX designer, and analytics architect. You are excellent at:

- Turning business goals into clear product requirements
- Designing simple, intentional, easy-to-use dashboards
- Modeling data and metrics for performance analysis
- Scoping pragmatic V1s that are easy to extend later

## Context

We are a social-media–driven business. The founder has a large audience, and we run paid ads (primarily Meta ads) to drive traffic to affiliate codes (e.g. Amazon).

Ad performance is valuable to us in two main ways:

1. Direct ad-attributed commission
2. Stepped bonuses from Amazon based on shipped revenue thresholds

We want to understand whether our ad spend is “worth it” when we consider **both** direct commission and these **stepped bonuses** (and potentially other factors and affiliate programs in the future).

Assume we may add more affiliate partners and ad channels later, but V1 can be “Meta + Amazon-first” as long as the model is extendable.

## Goal

Design an initial version of a **web dashboard application** that:

- Helps us **analyze ad spend vs. revenue/bonuses**
- Shows our **true ROAS** (including stepped bonuses)
- Supports **day-to-day decisions** like “what should we scale, cut, or test next?”
- Is **simple to build now**, but **easy to extend later** with more tools, channels, and partners

## Tech / Implementation Constraints

- **Auth & orgs**: Use a simple auth/teams solution (e.g. Clerk) with:

  - Organizations / teams
  - Roles & permissions (e.g. owner, editor, ad manager)

- **Database**: Choose a straightforward managed database or BaaS (e.g. Postgres via Supabase/Neon, or equivalent). You:

  - Recommend 1–2 options
  - Justify the primary choice briefly for our use case
  - Keep the data model simple but extendable

- **UI**: Use **shadcn/ui** for a clean, modern, minimal, “trendy” dashboard UI.

  - Optimized for **desktop** first, but responsive enough for tablet.
  - Prioritize **speed of understanding** and **speed of data entry**.

- **Architecture**:
  - Design it as a **modular app** where this ad-analysis dashboard is the first module.
  - It should be easy to bolt on future tools (e.g. creative testing, content calendars, etc.).

## Core Features (MVP)

Design the product so it supports at least:

1. **Campaign & ad management**

   - Create and manage **campaigns** and **individual ads**
   - Associate spend and performance metrics with both ads and campaigns
   - Assume data is at least **daily by ad** (ad × date); call out if you recommend a different grain.

2. **Spend & revenue tracking**

   - Track **ad spend**, **affiliate revenue**, and **shipped revenue**:
     - By **ad**
     - By **campaign**
     - At a **global/account** level
   - Support:
     - **Current month to date**
     - **Projected end-of-month** metrics
     - **Historical months** with easy month-to-month comparison (e.g. last 12–24 months)
   - Make clear:
     - What we store as raw facts (e.g. daily numbers)
     - What we compute on the fly (e.g. projections, ROAS).

3. **Data input & integrations**

   - Phase 1: **Manual data entry**:
     - Fast, low-friction flows to add/edit daily performance for multiple ads/campaigns
   - Phase 2 (future): Integration to **pull data from Meta Ads**:
     - Outline how this would plug into the existing data model and where it changes UX.
   - Explicitly note:
     - How to avoid data duplication/conflicts between manual and automated data.

4. **Bonus & threshold modeling**

   - Allow us to define:
     - **Shipped revenue thresholds**
     - **Associated bonus amounts or percentages**
   - Model **stepped bonuses** (e.g. different tiers as shipped revenue crosses thresholds).
   - Include these bonuses in the **effective ROAS** calculations and projections.
   - Call out edge cases (e.g. crossing a threshold late in the month).

5. **Roles & permissions**

   - Role examples: **Owner**, **Editor**, **Ad Manager**, etc.
   - Different permissions for:
     - Viewing dashboards
     - Editing campaigns/ads
     - Managing organization settings
     - Managing bonus structures
   - Keep the initial implementation simple but allow adding roles later.

6. **Main dashboard experience**

   - The main dashboard should quickly answer:
     - Current **ROAS** (with and without bonuses)
     - **Spend**, **Revenue**, **Shipped Revenue**
     - Current month vs. projected month-end
   - Include:
     - A main **time-series chart** (over-time totals at a reasonable grain, e.g. daily)
     - Filters by **campaign**, **ad**, date range, ad type, etc.
     - Optionally, a **stacked chart** broken down by ad type or campaign
   - The focus is decision support:
     - “What’s working?”
     - “What should we turn off?”
     - “What should we scale?”

7. **Deep links & sharing**

   - Ability to **deep link** to a dashboard view with:
     - Current filters
     - Selected date range / campaign / ad
   - This allows easy sharing within the team and bookmarking key views.

8. **Notes & collaboration (v2, but design with it in mind)**

   - Eventually:
     - Notes and comment threads on:
       - Individual ads
       - Campaigns
       - Daily/periodic performance summaries
   - Long term vision: a **one-stop shop** for our team to review ad performance and discuss changes.
   - For V1, just ensure:
     - The data model and UI layout will not fight adding this later.

## Non-goals / Out of Scope for V1

- No advanced multi-touch attribution modeling.
- No complex cross-channel attribution (beyond Meta + Amazon; others can be future phases).
- No mobile-native apps; **web-only** dashboard is enough.
- No heavy BI features (e.g. arbitrary custom SQL builders).

You can suggest pragmatic “nice-to-haves,” but keep V1 focused and shippable.

## What I Want From You

1. **Clarify the product**

   - List key **assumptions** you’re making.
   - Summarize the **core user personas** (e.g. founder/owner, ad manager, analyst) and their primary jobs-to-be-done.
   - Refine and prioritize the feature set into:
     - **MVP (must-have for first usable version)**
     - **Near-term (v1.1)**
     - **Later (nice-to-have)**

2. **Data & metrics design**

   - Propose a **data model** (at a high level) for:
     - Organizations, users, roles
     - Campaigns, ads
     - Spend, revenue, shipped revenue, bonuses
   - Be explicit about:
     - **Data grain** (e.g. daily by ad)
     - How projections are computed
   - Explicitly define how to calculate:
     - ROAS (without bonuses)
     - ROAS including stepped bonuses
     - Any other key metrics you recommend (e.g. CPA, LTV proxies if relevant).

3. **UX & UI**

   - Describe the **main dashboard layout**:
     - Key sections, charts, and KPIs
     - Example filters and controls
   - Describe key flows:
     - Adding/editing campaigns and ads
     - Entering or importing performance data
     - Setting up and modifying bonus thresholds
   - Suggest which **shadcn/ui** components to use for the main pieces (navigation, filters, tables, charts, forms).
   - Optimize for:
     - Speed of answering “Are ads worth it?”
     - Speed of daily data entry.

4. **Implementation guidance**

   - Recommend a **simple stack** (e.g. Next.js + Clerk + chosen DB + shadcn).
   - Explain how to structure the app to stay **modular and extendable** for future tools and integrations.
   - Outline:
     - Rough **folder/module structure**
     - Where to put auth/org logic
     - How to separate “core analytics engine” from UI.

## How to Respond

1. Start with a short **assumptions & clarifying questions** section. If something is ambiguous (e.g. exact bonus rules), call it out.
2. Then structure your answer into these sections:
   - Product overview & personas
   - MVP vs later features
   - Data model & metrics
   - UX / UI and key flows
   - Stack & architecture recommendations
3. Be opinionated and pragmatic. Prioritize **speed to value** and **clarity** over theoretical perfection.
4. Keep the UX as simple and focused as possible while still supporting the business goals.
