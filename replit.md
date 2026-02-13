# replit.md

## Overview

"The Stargazer" is an interactive fiction / visual novel web application. Players progress through a branching narrative (chapters 1–4 with multiple story paths and outcomes) presented in a "book" layout with rich animations. The app tracks reader progress (current chapter, clues found, story path chosen, love meter, minigame attempts) via a backend API and PostgreSQL database. A "Snowflake Minigame" in Chapter 4 adds gameplay elements. The frontend uses a storybook aesthetic with night/day themes, star fields, and page-turn animations.

## User Preferences

Preferred communication style: Simple, everyday language.

## System Architecture

### Monorepo Structure
The project is a single repo with three top-level source directories:
- **`client/`** — React SPA (Vite-based)
- **`server/`** — Express API server
- **`shared/`** — Code shared between client and server (DB schema, API route definitions, Zod validation schemas)

### Frontend (`client/src/`)
- **Framework:** React 18 with TypeScript
- **Bundler:** Vite (config in `vite.config.ts`)
- **Routing:** `wouter` (lightweight router) — routes map to chapters and outcome pages
- **State/Data Fetching:** `@tanstack/react-query` for server state; local `localStorage` for session ID persistence
- **UI Components:** shadcn/ui (new-york style) built on Radix UI primitives + Tailwind CSS
- **Animations:** `framer-motion` for page transitions, element reveals, and the minigame
- **Icons:** `lucide-react`
- **Forms:** `react-hook-form` with `@hookform/resolvers` for Zod validation
- **Styling:** Tailwind CSS with CSS custom properties for theming (light "parchment" mode and dark "night sky" mode); custom fonts (DM Sans, Libre Baskerville, Playfair Display, JetBrains Mono)
- **Path aliases:** `@/` → `client/src/`, `@shared/` → `shared/`, `@assets/` → `attached_assets/`

### Backend (`server/`)
- **Framework:** Express 5 on Node.js, running via `tsx` in development
- **API Pattern:** RESTful JSON API under `/api/` prefix. Routes defined in `server/routes.ts` with corresponding Zod schemas from `shared/routes.ts`
- **Database:** PostgreSQL via `pg` Pool + Drizzle ORM. Schema defined in `shared/schema.ts`
- **Storage Layer:** `server/storage.ts` implements `IStorage` interface with `DatabaseStorage` class — abstracts all DB operations behind a clean interface
- **Dev Server:** Vite dev server middleware integrated via `server/vite.ts` for HMR in development
- **Production:** Static files served from `dist/public`; server bundled with esbuild to `dist/index.cjs`

### Shared Code (`shared/`)
- **`schema.ts`** — Drizzle table definitions, Zod insert/update schemas, TypeScript types
- **`routes.ts`** — API route path constants, input/output Zod schemas, URL builder utility. This acts as a contract between frontend and backend.

### Database Schema
Single table `story_progress`:
| Column | Type | Description |
|--------|------|-------------|
| id | serial PK | Auto-increment ID |
| sessionId | text | Frontend-generated UUID identifying a reader's session |
| currentChapter | integer (default 1) | Which chapter the reader is on |
| minigameAttempts | integer (default 0) | How many times the minigame was attempted |
| cluesFound | text[] (default []) | Array of clue IDs found |
| isComplete | boolean (default false) | Whether story is finished |
| hasFailedHeart | boolean (default false) | Minigame failure flag |
| loveMeter | integer (default 0) | Score/affection tracker |
| storyPath | text (default "standard") | Branch taken: standard, sadness, imprisoned, moon, escape |
| lastUpdated | timestamp | Auto-updated on changes |

### API Endpoints
- `GET /api/story/:sessionId` — Retrieve session progress
- `POST /api/story` — Create new session (body validated against `insertProgressSchema`)
- `PATCH /api/story/:sessionId` — Update session progress (body validated against `updateProgressSchema`)

### Build System
- **Dev:** `npm run dev` runs `tsx server/index.ts` with Vite middleware for HMR
- **Build:** `npm run build` runs `script/build.ts` — Vite builds the client to `dist/public`, esbuild bundles the server to `dist/index.cjs`
- **DB Push:** `npm run db:push` uses `drizzle-kit push` to sync schema to database
- **Type Check:** `npm run check` runs `tsc`

## External Dependencies

### Database
- **PostgreSQL** — Required. Connection via `DATABASE_URL` environment variable. Uses `pg` (node-postgres) Pool with Drizzle ORM.
- **Drizzle Kit** — For schema migrations/push (`drizzle.config.ts` configured for PostgreSQL dialect, migrations output to `./migrations`)

### Key NPM Packages
- **drizzle-orm** + **drizzle-zod** — ORM and schema-to-Zod generation
- **express** v5 — HTTP server
- **connect-pg-simple** — PostgreSQL session store (available but sessions not currently used for auth)
- **zod** — Runtime validation across frontend and backend
- **@tanstack/react-query** — Async state management on the client
- **framer-motion** — Animation library
- **shadcn/ui** ecosystem — Radix UI primitives, class-variance-authority, tailwind-merge, clsx

### Replit-Specific
- **@replit/vite-plugin-runtime-error-modal** — Error overlay in dev
- **@replit/vite-plugin-cartographer** and **@replit/vite-plugin-dev-banner** — Dev-only Replit integrations (conditionally loaded)

### Environment Variables Required
- `DATABASE_URL` — PostgreSQL connection string (required for both server startup and drizzle-kit)