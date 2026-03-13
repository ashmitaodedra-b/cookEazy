# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Status

cookEazy is a **Recipe Discovery web app** currently in pre-development. The full requirements are in `recipe-app-prd.md`. No application code exists yet — the Next.js project has not been scaffolded.

## Planned Stack

- **Framework:** Next.js 14+ with App Router and React Server Components
- **Styling:** Tailwind CSS v3
- **Auth + Database:** Supabase (PostgreSQL, RLS, Storage)
- **Deployment:** Vercel (production) + GitHub Actions (CI/CD)
- **Language:** TypeScript throughout

## Expected Commands (once scaffolded)

```bash
npm run dev          # Start local dev server (http://localhost:3000)
npm run build        # Production build
npm run lint         # ESLint
npm run type-check   # tsc --noEmit

# Supabase
npx supabase start           # Start local Supabase stack (Docker required)
npx supabase db push         # Apply migrations to remote
npx supabase db reset        # Reset local DB and re-run migrations + seed
npm run db:seed              # Run seed script (non-production only)

# Single test (once tests are set up)
npx jest path/to/test.ts
```

## Planned App Router Structure

```
app/
  (auth)/login/        # Login page — no layout chrome
  (auth)/signup/       # Signup page — no layout chrome
  (main)/              # Main layout with nav
    page.tsx           # Homepage feed (Server Component, fetches recipes)
    recipes/[id]/      # Recipe detail page
    recipes/new/       # Multi-step recipe creation form (Client Component)
    profile/[username]/
    collections/
  admin/               # Role-gated; middleware redirects non-admins
  api/recipes/         # Route handlers for mutations
  api/ratings/
  layout.tsx           # Root layout — sets up Supabase session provider
```

## Supabase Client Pattern

Two clients must be used correctly — mixing them causes auth bugs:

- `lib/supabase/server.ts` — Server Components, Route Handlers, middleware. Uses `cookies()` from `next/headers`.
- `lib/supabase/client.ts` — Client Components only. Uses `createBrowserClient`.

## Database Schema (key relationships)

```
auth.users (Supabase managed)
  └─► profiles (extended user data, role: 'user' | 'admin')
        └─► recipes (status: 'draft' | 'published' | 'archived')
              ├─► ingredients (ordered by sort_order)
              ├─► steps (ordered by step_number)
              ├─► recipe_tags ──► tags
              └─► ratings (unique per user per recipe, score 1–5)
        └─► collections
              └─► collection_items ──► recipes
```

All tables use Row-Level Security. Users may only mutate their own rows. Admin role is enforced at both the RLS level and in Next.js middleware.

## Migrations

SQL migrations live in `supabase/migrations/` and are applied with `supabase db push`. Never write raw SQL from the client — all DB access goes through the Supabase JS client with JWT verification.

## Environment Variables

```bash
NEXT_PUBLIC_SUPABASE_URL=       # safe to expose
NEXT_PUBLIC_SUPABASE_ANON_KEY=  # safe to expose
SUPABASE_SERVICE_ROLE_KEY=      # server-only, never pass to client
```

Set these in `.env.local` locally and in Vercel project settings for production. A `.env.local.example` file should be kept in the repo as a reference.

## Seed Data

The seed script (`supabase/seed.ts`) creates 3 demo accounts (viewer, contributor, admin), 30 recipes across 6 cuisine categories, and sample ratings/bookmarks. It is idempotent — safe to re-run. Never run it against production.
