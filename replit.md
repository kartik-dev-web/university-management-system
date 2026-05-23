# University Management System (UMS)

A full-stack academic management portal for three roles — Admin, Faculty, and Student — with dashboards, attendance tracking, marks/grades, notices, timetable, leave requests, department and course management, and JWT-based authentication.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 8080)
- `pnpm --filter @workspace/university-ms run dev` — run the React frontend (port 25717)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required env: `DATABASE_URL` — Postgres connection string, `SESSION_SECRET` — JWT signing key

## Demo Credentials

| Role    | Email                        | Password      |
|---------|------------------------------|---------------|
| Admin   | admin@university.edu         | Admin@123     |
| Faculty | faculty@university.edu       | Faculty@123   |
| Student | student@university.edu       | Student@123   |

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- Frontend: React + Vite, shadcn/ui, Recharts, Wouter (routing), TanStack Query
- API: Express 5, JWT auth (jsonwebtoken + bcryptjs)
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec at `lib/api-spec/openapi.yaml`)
- Build: esbuild (CJS bundle)

## Where things live

- `lib/db/src/schema/` — Drizzle ORM schema (users, departments, semesters, courses, enrollments, attendance, marks, notices, leave_requests, timetable)
- `lib/api-spec/openapi.yaml` — OpenAPI 3.0 contract (source of truth for API)
- `lib/api-client-react/src/generated/` — Generated React Query hooks and Zod schemas (do not edit manually)
- `artifacts/api-server/src/routes/` — Express route handlers (auth, dashboard, users, departments, courses, semesters, attendance, marks, notices, leaves, timetable)
- `artifacts/api-server/src/middlewares/auth.ts` — JWT `authenticate` middleware + `requireRole` guard
- `artifacts/university-ms/src/pages/` — React pages (admin/, faculty/, student/)
- `artifacts/university-ms/src/hooks/use-auth.tsx` — Auth context + `setAuthTokenGetter` global wiring

## Architecture decisions

- **Contract-first API**: OpenAPI spec is the single source of truth; Orval generates fully typed React Query hooks used throughout the frontend. Never write fetch calls by hand.
- **JWT in localStorage**: Token stored in localStorage; `setAuthTokenGetter` from the API client automatically attaches `Authorization: Bearer` to every request.
- **Role-based routing**: `ProtectedRoute` component checks `user.role` against the required role; unauthenticated users hit an "Unauthorized" state, not a redirect, because the token is checked client-side before navigation.
- **Flat route structure**: All API routes are prefixed `/api/*` and served by the API server; the React app is served at `/`. The shared Replit reverse proxy handles path-based routing with no cross-origin issues.
- **Seeded demo data**: 1 admin, 10 faculty, 20 students across 5 departments; 12 courses; 238 attendance records; 48 mark entries; 8 notices; 10 leave requests; 22 timetable slots.

## Product

- **Admin**: Full CRUD over departments, courses, semesters, users (students + faculty); approve/reject leave requests; post notices; view attendance and marks across the institution; manage timetable.
- **Faculty**: Mark student attendance per course and date; view marks register by subject; see personal timetable; read notices; submit leave requests.
- **Student**: View attendance summary with per-subject breakdown and shortage warnings; view marks by course and exam type with CGPA; see personal timetable; read notices; submit leave requests.

## User preferences

- Professional academic design, dark-navy primary color scheme
- shadcn/ui component library throughout
- All pages fully connected to real backend data (no mocks or placeholders)

## Gotchas

- Run `pnpm --filter @workspace/api-spec run codegen` after any OpenAPI spec changes before touching frontend code.
- Do not run `pnpm dev` at workspace root — start workflows individually via Replit.
- Timetable filtering for students is done client-side: fetch all entries, then filter by enrolled course IDs.
- The `query: { enabled: ... }` pattern on generated hooks requires `as any` cast because Orval types `UseQueryOptions` with `queryKey` required (RQ v5), but the hook fills it internally.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
