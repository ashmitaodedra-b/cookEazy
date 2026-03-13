# Recipe Discovery Web App — Product Requirements Document

**Project:** cookEazy
**Version:** 1.0
**Date:** 2026-03-13
**Status:** Draft

---

## Table of Contents

1. [Overview](#1-overview)
2. [Problem Statement](#2-problem-statement)
3. [Goals](#3-goals)
4. [Target Users](#4-target-users)
5. [Core Features](#5-core-features)
6. [User Flows](#6-user-flows)
7. [Data Model](#7-data-model)
8. [Non-Functional Requirements](#8-non-functional-requirements)
9. [Technical Reference](#9-technical-reference)

---

## 1. Overview

**cookEazy** is a web application that helps home cooks discover, save, and share recipes tailored to their tastes, dietary preferences, and available ingredients. Users can browse a curated recipe library, search by ingredient or cuisine, bookmark favorites, and contribute their own recipes — all within a clean, mobile-first interface.

The application is built with:

- **Next.js 14+ (App Router)** — Server Components, file-based routing, API routes
- **Supabase** — PostgreSQL database, Row-Level Security, Auth (email/password + OAuth)
- **Vercel** — CI/CD deployment pipeline with preview environments
- **Tailwind CSS** — Utility-first responsive styling

---

## 2. Problem Statement

Home cooks face several friction points when deciding what to cook:

| Pain Point | Impact |
|---|---|
| Too many scattered sources (YouTube, blogs, Pinterest) | Decision fatigue; time wasted switching tabs |
| No single place to save and organize personal recipes | Favorites get lost or buried |
| Search engines return SEO-bloated recipe pages | Hard to find genuinely good recipes quickly |
| No ingredient-based filtering | Users buy extra groceries instead of cooking with what they have |
| Poor mobile experience on recipe blogs | Hard to follow recipes on a phone while cooking |

cookEazy consolidates discovery, saving, and sharing into one frictionless experience.

---

## 3. Goals

### Business Goals

- Achieve **1,000 registered users** within 3 months of launch.
- Maintain a **recipe submission rate** of ≥ 10 new community recipes per week by month 2.
- Target a **Day-7 retention rate** of ≥ 40% through personalization and bookmarking.

### Product Goals

- Enable a user to **find a suitable recipe in under 60 seconds** from landing.
- Allow authenticated users to **save, organize, and rate** recipes with zero friction.
- Provide a **recipe creation flow** completable in under 5 minutes.

### Technical Goals

- Achieve **Lighthouse score ≥ 90** across Performance, Accessibility, and SEO.
- Cold-start page load (LCP) **≤ 2.5 s** on a 4G connection.
- Zero-downtime deployments via Vercel with **preview URLs per PR**.

---

## 4. Target Users

### Primary — The Busy Home Cook

- **Age:** 25–45
- **Context:** Cooking 3–5 times per week; limited time; shops weekly
- **Goals:** Find quick, reliable recipes using ingredients already on hand
- **Frustrations:** Ads, long blog intros, no easy way to filter by prep time

### Secondary — The Food Enthusiast

- **Age:** 20–35
- **Context:** Enjoys exploring new cuisines; loves sharing creations
- **Goals:** Discover trending and niche recipes; build a public recipe portfolio
- **Frustrations:** No single platform that rewards contribution

### Tertiary — The Dietary-Restricted User

- **Age:** Any
- **Context:** Vegetarian, vegan, gluten-free, or allergy-aware
- **Goals:** Filter recipes reliably by dietary tag without sifting through irrelevant results
- **Frustrations:** Inconsistent tagging on general recipe sites

---

## 5. Core Features

### 5.1 Authentication & Profiles

- Email/password sign-up and login via **Supabase Auth**
- OAuth providers: **Google**, **GitHub**
- User profile page: display name, avatar, bio, submitted recipes, bookmarks
- Password reset via email magic link

### 5.2 Recipe Discovery

- **Homepage feed** — curated cards (trending, newest, staff picks)
- **Search bar** — full-text search across title, ingredients, and tags
- **Filter panel** — cuisine, dietary tags, prep time, difficulty, rating
- **Ingredient-based search** — enter available ingredients; surface matching recipes
- **Recipe detail page** — hero image, metadata, ingredient list, step-by-step instructions, author card, rating, comments

### 5.3 Recipe Management (Authenticated)

- **Create recipe** — multi-step form: metadata → ingredients → steps → publish
- **Edit / delete** own recipes
- **Draft mode** — save incomplete recipes before publishing
- **Image upload** — stored in Supabase Storage with optimized Next.js `<Image>` delivery

### 5.4 Bookmarks & Collections

- One-click bookmark on any recipe card or detail page
- Organize bookmarks into named **collections** (e.g., "Weeknight Dinners", "Meal Prep")
- Collections visible on profile (public or private toggle)

### 5.5 Ratings & Reviews

- 1–5 star rating per recipe (one per user)
- Text comment (optional, ≤ 500 characters)
- Aggregate rating displayed on card and detail page
- Report / flag inappropriate content

### 5.6 Seed Demo Data

- On first deployment, a **seed script** populates:
  - 3 demo user accounts (viewer, contributor, admin)
  - 30 recipes spanning 6 cuisine categories
  - Sample bookmarks, ratings, and comments
- Seed is idempotent — safe to re-run

### 5.7 Admin Panel

- Simple `/admin` route (role-gated via Supabase RLS)
- Approve/reject flagged recipes and comments
- View aggregate stats: DAU, recipe count, top-rated recipes

---

## 6. User Flows

### 6.1 New User — First Discovery

```
Landing Page
  └─► Browse recipe feed (no login required)
        └─► Click recipe card
              └─► View Recipe Detail Page
                    ├─► [Not logged in] → CTA "Sign up to save this recipe"
                    └─► [Logged in] → Bookmark / Rate
```

### 6.2 Sign-Up & Onboarding

```
Sign Up Page
  └─► Enter email + password  OR  OAuth (Google / GitHub)
        └─► Email verification (if email/password)
              └─► Onboarding screen: select dietary preferences + cuisines of interest
                    └─► Personalized feed generated
                          └─► Redirected to Homepage
```

### 6.3 Recipe Creation

```
Dashboard → "Add Recipe" button
  └─► Step 1: Basic Info (title, description, cuisine, tags, difficulty, prep/cook time)
        └─► Step 2: Ingredients (dynamic list — add/remove rows)
              └─► Step 3: Instructions (ordered rich-text steps)
                    └─► Step 4: Upload cover image
                          └─► Preview → Publish
                                └─► Recipe Detail Page (live)
```

### 6.4 Ingredient-Based Search

```
Homepage → "What's in your fridge?" input
  └─► Enter ingredients (comma-separated or tag input)
        └─► Search → Results grid filtered by ingredient overlap score
              └─► Click recipe → Detail Page
```

### 6.5 Bookmark to Collection

```
Recipe Detail Page → Bookmark icon
  └─► [No collections] → Prompt: "Create your first collection"
        └─► Name collection → Save → Recipe added
  └─► [Has collections] → Dropdown picker → Select / create → Saved toast
```

---

## 7. Data Model

All tables live in **Supabase (PostgreSQL)**. Row-Level Security (RLS) policies enforce access control.

### 7.1 `users` (managed by Supabase Auth + extended profile)

```sql
-- Supabase Auth handles auth.users; this extends it:
CREATE TABLE public.profiles (
  id          UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  username    TEXT UNIQUE NOT NULL,
  display_name TEXT,
  avatar_url  TEXT,
  bio         TEXT,
  role        TEXT DEFAULT 'user' CHECK (role IN ('user', 'admin')),
  created_at  TIMESTAMPTZ DEFAULT NOW()
);
```

### 7.2 `recipes`

```sql
CREATE TABLE public.recipes (
  id           UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  author_id    UUID REFERENCES public.profiles(id) ON DELETE SET NULL,
  title        TEXT NOT NULL,
  description  TEXT,
  cuisine      TEXT,
  difficulty   TEXT CHECK (difficulty IN ('easy', 'medium', 'hard')),
  prep_time    INT,   -- minutes
  cook_time    INT,   -- minutes
  servings     INT,
  cover_image  TEXT,  -- Supabase Storage URL
  status       TEXT DEFAULT 'draft' CHECK (status IN ('draft', 'published', 'archived')),
  created_at   TIMESTAMPTZ DEFAULT NOW(),
  updated_at   TIMESTAMPTZ DEFAULT NOW()
);
```

### 7.3 `ingredients`

```sql
CREATE TABLE public.ingredients (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  recipe_id  UUID REFERENCES public.recipes(id) ON DELETE CASCADE,
  name       TEXT NOT NULL,
  quantity   TEXT,
  unit       TEXT,
  sort_order INT DEFAULT 0
);
```

### 7.4 `steps`

```sql
CREATE TABLE public.steps (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  recipe_id  UUID REFERENCES public.recipes(id) ON DELETE CASCADE,
  step_number INT NOT NULL,
  instruction TEXT NOT NULL
);
```

### 7.5 `tags`

```sql
CREATE TABLE public.tags (
  id   UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT UNIQUE NOT NULL  -- e.g. 'vegan', 'gluten-free', 'quick'
);

CREATE TABLE public.recipe_tags (
  recipe_id UUID REFERENCES public.recipes(id) ON DELETE CASCADE,
  tag_id    UUID REFERENCES public.tags(id) ON DELETE CASCADE,
  PRIMARY KEY (recipe_id, tag_id)
);
```

### 7.6 `ratings`

```sql
CREATE TABLE public.ratings (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  recipe_id  UUID REFERENCES public.recipes(id) ON DELETE CASCADE,
  user_id    UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
  score      INT CHECK (score BETWEEN 1 AND 5),
  comment    TEXT CHECK (char_length(comment) <= 500),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  UNIQUE (recipe_id, user_id)
);
```

### 7.7 `collections` & `collection_items`

```sql
CREATE TABLE public.collections (
  id         UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id    UUID REFERENCES public.profiles(id) ON DELETE CASCADE,
  name       TEXT NOT NULL,
  is_public  BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE public.collection_items (
  collection_id UUID REFERENCES public.collections(id) ON DELETE CASCADE,
  recipe_id     UUID REFERENCES public.recipes(id) ON DELETE CASCADE,
  added_at      TIMESTAMPTZ DEFAULT NOW(),
  PRIMARY KEY (collection_id, recipe_id)
);
```

### 7.8 Entity Relationship Summary

```
profiles ──< recipes ──< ingredients
                │      └─< steps
                │      └─< recipe_tags >── tags
                │      └─< ratings
                └──< collections ──< collection_items >── recipes
```

---

## 8. Non-Functional Requirements

### 8.1 Performance

| Metric | Target |
|---|---|
| Largest Contentful Paint (LCP) | ≤ 2.5 s (4G) |
| First Input Delay (FID) | ≤ 100 ms |
| Cumulative Layout Shift (CLS) | ≤ 0.1 |
| Lighthouse Score (Perf / A11y / SEO) | ≥ 90 each |
| API response time (p95) | ≤ 300 ms |

### 8.2 Scalability

- Stateless Next.js server functions scale horizontally on Vercel Edge Network.
- Supabase connection pooling via **PgBouncer** (built-in on Supabase).
- Images served via Supabase CDN + Next.js Image Optimization.
- Database indexes on `recipes.cuisine`, `recipes.status`, `recipes.created_at`, full-text search via `tsvector` column.

### 8.3 Security

- All mutations require authenticated session (JWT verified by Supabase).
- RLS policies on every table — users can only mutate their own rows.
- Admin role enforced at both RLS and middleware layer.
- Inputs sanitized server-side; no raw SQL from client.
- CORS, CSP, and `X-Frame-Options` headers configured in `next.config.js`.
- Secrets (Supabase URL, anon key, service role key) stored in Vercel environment variables — never committed.

### 8.4 Accessibility

- WCAG 2.1 AA compliance.
- Keyboard-navigable interface (focus rings, skip links).
- `aria-label` on icon-only buttons; `alt` text on all images.
- Color contrast ratio ≥ 4.5:1 for body text.

### 8.5 Responsiveness

- Mobile-first breakpoints: `sm` (640px), `md` (768px), `lg` (1024px), `xl` (1280px).
- Recipe grid: 1 col (mobile) → 2 col (tablet) → 3–4 col (desktop).
- Navigation collapses to bottom tab bar on mobile.

### 8.6 Reliability & Observability

- Vercel's built-in analytics for Web Vitals monitoring.
- Error boundaries on all route segments; graceful fallback UI.
- Structured logging via `console.error` captured by Vercel Log Drains.
- Uptime target: **99.9%** monthly (Vercel SLA + Supabase SLA).

### 8.7 Deployment Pipeline

```
Feature Branch
  └─► PR opened → Vercel Preview URL generated automatically
        └─► Code review + Supabase migration review
              └─► Merge to main → Production deploy (zero-downtime)
                    └─► Supabase migration applied via CLI in CI step
```

- Migrations tracked in `supabase/migrations/` directory.
- `supabase db push` run in GitHub Actions before Vercel build.
- Seed script: `npm run db:seed` — runs only in non-production environments.

### 8.8 Internationalisation (future)

- App scaffolded with `next-intl` in mind; all user-facing strings extracted to locale files.
- Initial launch: English only.

---

## 9. Technical Reference

### Stack Summary

| Layer | Technology |
|---|---|
| Framework | Next.js 14+ (App Router, Server Components) |
| Styling | Tailwind CSS v3 |
| Auth | Supabase Auth (email, Google, GitHub OAuth) |
| Database | Supabase (PostgreSQL 15) |
| Storage | Supabase Storage (recipe images) |
| ORM / Query | Supabase JS client (`@supabase/supabase-js`) |
| Deployment | Vercel (production + preview) |
| CI/CD | GitHub Actions + Vercel Git Integration |
| Migrations | Supabase CLI (`supabase db push`) |

### Key File Structure

```
cookEazy/
├── app/
│   ├── (auth)/
│   │   ├── login/page.tsx
│   │   └── signup/page.tsx
│   ├── (main)/
│   │   ├── page.tsx                  # Homepage feed
│   │   ├── recipes/
│   │   │   ├── [id]/page.tsx         # Recipe detail
│   │   │   └── new/page.tsx          # Create recipe
│   │   ├── profile/[username]/page.tsx
│   │   └── collections/page.tsx
│   ├── admin/page.tsx
│   ├── api/
│   │   ├── recipes/route.ts
│   │   └── ratings/route.ts
│   └── layout.tsx
├── components/
│   ├── RecipeCard.tsx
│   ├── RecipeGrid.tsx
│   ├── FilterPanel.tsx
│   ├── RatingStars.tsx
│   └── BookmarkButton.tsx
├── lib/
│   ├── supabase/
│   │   ├── client.ts                 # Browser client
│   │   └── server.ts                 # Server client (cookies)
│   └── utils.ts
├── supabase/
│   ├── migrations/
│   │   ├── 001_initial_schema.sql
│   │   └── 002_rls_policies.sql
│   └── seed.ts
├── public/
├── next.config.js
├── tailwind.config.ts
└── .env.local.example
```

### Environment Variables

```env
# .env.local.example
NEXT_PUBLIC_SUPABASE_URL=https://<project>.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=<anon-key>
SUPABASE_SERVICE_ROLE_KEY=<service-role-key>   # server-only
```

### Architecture Diagram

```
Browser
  │
  ▼
Vercel Edge (CDN / Middleware)
  │
  ├── Next.js Server Components (RSC) ──► Supabase DB (PostgreSQL)
  │                                              │
  ├── Next.js API Routes ────────────────────────┘
  │
  └── Supabase Auth (JWT) ──► Supabase Storage (Images)
```

---

*Document maintained by the cookEazy core team. Update this PRD when scope, stack, or requirements change.*
