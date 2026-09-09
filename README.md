# MapRouter

> View bus routes, schedules and fares on an interactive map.

## Overview

MapRouter is a read-only public web application that lets riders browse bus routes, inspect stops on an interactive map, check timetables and see ticket prices. All data is published by the transport operator through an admin panel — there is no crowdsourcing. The MVP targets a single city or region. Real-time vehicle tracking is out of scope for the initial release.

## Tech Stack

| Layer | Choice |
|-------|--------|
| Database | Supabase (Postgres + PostGIS) |
| Backend | Next.js + TypeScript |
| Frontend | Next.js (App Router) + Tailwind CSS + MapLibre GL JS |
| Auth | Supabase Auth (Google provider) |
| Hosting | Vercel (app) + Supabase (data) |

## Architecture

The application is a single Next.js monolith using the App Router. Public pages are server-rendered reads from Supabase. Route Handlers under `app/api` expose a small REST surface. PostGIS powers spatial queries — bounding-box viewport filtering for stops and route polyline rendering. Row-Level Security policies enforce public read access and admin-only writes. The `/admin` section is protected by Next.js middleware and Supabase Auth (Google), with a role column on the `profiles` table controlling access.

## Data Model

```text
agencies
  id, name, url, tz

routes
  id, agency_id FK, short_name, long_name, description, color, type

stops
  id, code, name, description, location geography(POINT,4326), wheelchair_boarding

trips
  id, route_id FK, direction, day_type (weekday|saturday|sunday), headsign

schedule_items
  id, trip_id FK, stop_id FK, stop_sequence, arrival_time, departure_time

prices
  id, route_id FK, label, amount, currency, valid_from, valid_to
```

Indexes: GIST on `stops.location`; btree on `trips(route_id, day_type)` and `schedule_items(trip_id, stop_sequence)`. Tables are kept GTFS-compatible so a GTFS import can be added later without schema rewrites.

## API

- `GET /api/routes` — list routes (optional `?agency_id=`)
- `GET /api/routes/[id]` — route detail
- `GET /api/routes/[id]/schedule?day_type=` — timetable
- `GET /api/routes/[id]/prices` — fares
- `GET /api/stops?bbox=` — stops in map viewport (PostGIS envelope query)
- Admin CRUD endpoints (`POST` / `PUT` / `DELETE`), auth-gated

## Roadmap

**Phase 0 — Scaffold.** `create-next-app` (TypeScript, App Router, Tailwind), Supabase project with PostGIS enabled, environment variables and Supabase client setup, lint and type-check CI.

**Phase 1 — Schema + seed.** Migrations for the tables above, seed data (1 agency, 2 routes, ~15 stops, schedules, prices), generated TypeScript types.

**Phase 2 — Public UI.** MapLibre map with stops and route polylines, route list and detail pages with tabs (Overview / Map / Timetable / Prices), responsive layout.

**Phase 3 — Admin.** Google OAuth, `/admin` CRUD for all entities, RLS policies.

**Phase 4 — Deploy.** Vercel + Supabase production, environment variables, smoke tests.

**Post-MVP backlog.** GTFS import, real-time vehicle positions, A-to-B trip planning, favorites, multi-city support.

## Getting Started

**Prerequisites:** Node 20+, a Supabase project with PostGIS enabled, Vercel account (optional, for deploy).

1. Clone the repo.
2. `pnpm install` (or `npm install`).
3. Copy `.env.example` to `.env.local` and set `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, and `SUPABASE_SERVICE_ROLE_KEY` (server only).
4. Run migrations: `supabase db push` (or apply via the Supabase dashboard).
5. `npm run dev`.

Note: `.env.example` does not exist yet — it will be created as part of Phase 0.

## Open Questions

1. **Scope:** one city for MVP or multi-city from day one?
2. **Data source:** manual admin entry, or an existing GTFS feed to import?
3. **Fare model:** flat fare per route, zone-based, or distance-based?
4. **Map tiles:** free (OpenFreeMap / OSM raster) or paid provider?
5. **Admin model:** single super-admin or roles (editor / viewer)?
