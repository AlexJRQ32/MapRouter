# PROD-READINESS — MapRouter

> Auditoría de preparación para producción basada en evidencia real de los archivos del repo.
> Fecha de auditoría: 2026-09-10.

## 1. Estado actual

- **Qué es:** App web de solo lectura para que usuarios consulten rutas, horarios y tarifas de buses en un mapa interactivo (datos publicados por el operador vía panel admin). Fuente: `README.md`.
- **Stack real (verificado):**
  - **Backend/Frontend:** Next.js 16 (App Router) + TypeScript + Tailwind CSS 4 + MapLibre GL JS. Evidencia: `maprouter/package.json` (`next: 16.3.4`, `react: 19.2.8`), `next.config.ts`, `app/layout.tsx`, `app/globals.css`.
  - **Datos:** Supabase (Postgres + PostGIS) y Supabase Auth (Google) — definido en `README.md`, **no implementado**.
  - **Hosting:** Vercel + Supabase (planeado).
- **Gestor de paquetes:** ✅ **pnpm** correcto. `package.json` declara `"packageManager": "pnpm@11.21.0"` y existe `pnpm-lock.yaml`. `README.md` dice `pnpm install (or npm install)`.
- **Estado de implementación: SCAFFOLD PURO.** La carpeta `app/` contiene **solo** el boilerplate de `create-next-app`:
  - `app/page.tsx` es la pantalla de bienvenida por defecto de Next.js ("To get started, edit the page.tsx file…", con enlaces a Vercel/Next docs).
  - No hay `lib/`, `components/`, cliente de Supabase, route handlers (`app/api`), esquema de BD, seeds ni `/admin`.
  - No existe `.env.example` (el propio `README.md` admite: "`.env.example` does not exist yet").
- **Conclusión:** No apto para producción. Está en fase 0 del roadmap (`README.md` Phase 0–4). "Uppropped Sept 2026" = repo empujado pero sin features.

## 2. Tabla priorizada de pendientes

| Pri | Ítem | Evidencia / por qué |
|-----|------|---------------------|
| **P0** | Implementar esquema + migraciones Supabase (PostGIS) | `README.md` Data Model define `agencies/routes/stops/trips/schedule_items/prices`. Nada creado. |
| **P0** | Cliente Supabase + tipos generados | No existe `lib/supabase` ni imports en `app/`. |
| **P0** | UI pública (mapa MapLibre, lista/detalle de rutas, tabs) | `app/page.tsx` es boilerplate; faltan todas las páginas. |
| **P0** | Route handlers `app/api/*` (GET rutas/stops/schedule/prices) | No existen. |
| **P0** | Crear `.env.example` | README lo admite ausente. Requerido para deploy. |
| **P0** | Auth admin (Google OAuth) + middleware + RLS | `/admin` y políticas RLS no implementados. |
| **P1** | Seeds de datos (1 agencia, 2 rutas, ~15 stops…) | `README.md` Phase 1. |
| **P1** | Configurar map tiles (OpenFreeMap/OSM vs pago) | `README.md` Open Questions #4 sin resolver. |
| **P1** | `vercel.json` / config de build | No existe `vercel.json`; Vercel auto-detecta Next.js (suficiente para empezar). |
| **P2** | Tests / lint CI | `package.json` tiene `lint` (eslint) pero no scripts de test ni CI. |
| **P2** | Definir modelo de admin (super-admin vs roles) | `README.md` Open Questions #5. |

## 3. Variables de entorno requeridas

> Definidas en `README.md` (Getting Started) pero **no verificadas en código** porque no hay cliente Supabase todavía. `.env.example` no existe.

| Variable | Uso | Dónde hoy |
|----------|-----|-----------|
| `NEXT_PUBLIC_SUPABASE_URL` | URL del proyecto Supabase | Documentada en `README.md`. Sin `.env.example`. |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Anon key (lectura pública) | Documentada en `README.md`. |
| `SUPABASE_SERVICE_ROLE_KEY` | Server-only (escrituras admin) | Documentada en `README.md` ("server only"). |
| `GOOGLE_OAUTH_*` (implícitas) | Login Google para `/admin` | No definidas aún; dependerán de la librería Supabase Auth. |

> `.gitignore` ignora `.env*` ✅. Sin riesgo de commit de secretos hoy (no hay `.env`).

## 4. Plan de deployment paso a paso (cuando esté implementado)

```bash
cd maprouter
pnpm install
cp .env.example .env.local      # CREAR primero el .env.example (P0)
# Editar .env.local con las 3 vars de Supabase
supabase db push               # aplicar migraciones (o vía dashboard Supabase)
pnpm dev                       # local: http://localhost:3000
pnpm build && pnpm start       # smoke test de producción local
# Deploy: conectar repo a Vercel (auto-detecta Next.js). Setear las 3 env vars en Vercel.
```

> Vercel no requiere `vercel.json` para Next.js (lo detecta). Si se quiere SPA fallback o rewrites específicos, añadir más adelante.

## 5. Riesgos de seguridad

1. **N/A hoy** — no hay código de auth ni datos. Los riesgos aparecerán al implementar:
   - RLS debe permitir **solo lectura pública** y escritura **admin-only** (`README.md` lo describe; hay que implementarlo y auditarlo).
   - `SUPABASE_SERVICE_ROLE_KEY` es secreto server-side: nunca exponerla al cliente (uso de Route Handlers server-side).
   - Middleware de Next.js debe proteger `/admin` con la columna `role` de `profiles`.
2. **Buenas prácticas ya presentes:** pnpm como gestor ✅, `.env*` ignorado ✅.

---
*Evidencia: `README.md`, `maprouter/package.json`, `maprouter/next.config.ts`, `maprouter/app/page.tsx`, `maprouter/.gitignore`, `maprouter/pnpm-lock.yaml`. Todo lo relativo a features/auth/DB está marcado "no verificado / no implementado" porque el repo es scaffold.*
