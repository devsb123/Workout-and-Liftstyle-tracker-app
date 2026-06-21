# DadTracker — Build Plan

## Context

Personal Father's Day gift: a multi-user productivity web app called **DadTracker** with tasks (Kanban + list, hybrid urgency), calendar, workout log, food/water tracking with personalized goals, and secure auth. Repo is **public on GitHub** — so secrets handling and IDOR protection are first-class concerns, not afterthoughts.

The user wants this built **incrementally, one phase at a time, with verification between phases**, and small/logical commits so the GitHub history reads well.

## Decisions Locked In

| Topic | Choice |
|---|---|
| Framework | **Next.js 15 (App Router) + TypeScript** |
| DB / ORM | **PostgreSQL + Prisma** |
| Auth | **Auth.js (NextAuth v5) — Credentials provider + bcrypt**, httpOnly session cookies |
| Styling | **Tailwind CSS** |
| Charts | **Recharts** |
| Drag & drop | **@dnd-kit** |
| Calendar UI | **FullCalendar** (month/week/day views, free tier) |
| Validation | **Zod** on every form & API route |
| Units default | **Imperial** (lb / ft-in / oz), with metric toggle in Settings; **internal storage always metric** so BMR math stays clean |
| Urgency thresholds | Green when due > 3 days out, yellow at ≤ 3 days, red at ≤ 24h or overdue. Per-task "pin manual urgency" override flag bypasses auto-escalation. |
| Repo | User pastes GitHub URL at start of Phase 0; I `git clone` into the current folder |
| Deploy | Docker + docker-compose (app + postgres + named volume); README documents the flow. No production hosting in scope. |

## Critical Files (to be created)

Project is greenfield — empty directory. Key files the plan will produce:

- `package.json`, `tsconfig.json`, `next.config.ts`, `tailwind.config.ts`, `postcss.config.mjs`
- `prisma/schema.prisma` — all models (User, Account, Session, Task, Tag, Event, Workout, ExerciseLog, Meal, WaterLog, ProfileSettings)
- `src/lib/auth.ts` — NextAuth config (Credentials + bcrypt + Prisma adapter)
- `src/lib/db.ts` — Prisma client singleton
- `src/lib/urgency.ts` — pure function computing effective urgency from `{manualUrgency, dueAt, pinned, now}` — unit-tested
- `src/lib/units.ts` — imperial↔metric conversions; storage stays metric
- `src/lib/goals.ts` — Mifflin-St Jeor BMR/TDEE + 35 mL/kg water default
- `src/middleware.ts` — route protection
- `src/app/api/**/route.ts` — REST endpoints, **every handler re-checks `session.user.id` ownership** (no IDOR)
- `src/app/(auth)/{login,signup}/page.tsx`
- `src/app/(app)/{dashboard,tasks,calendar,workouts,nutrition,settings}/page.tsx`
- `Dockerfile` (multi-stage: deps → build → slim runtime)
- `docker-compose.yml` (app + postgres + named volume)
- `.env.example` (variable names only, no values)
- `.gitignore` (must include `.env`, `.env.local`, `node_modules`, `.next`)
- `README.md` (updated at end of each phase)

## Build Phases

Each phase ends with a **manual verification checklist** before moving on. Commits stay small (typically 1 commit per logical sub-step within a phase).

### Phase 0 — Repo bootstrap
1. Clone user-provided GitHub URL into current dir.
2. `create-next-app` with TS + Tailwind + App Router + ESLint, accept npm.
3. Add Prettier, basic ESLint config, `.gitignore` audit (confirm `.env*` ignored).
4. First commit: scaffold.
- **Verify:** `npm run dev` shows Next welcome page at `localhost:3000`.

### Phase 1 — DB + Prisma + Docker baseline
1. Install Prisma, init schema with **User** + NextAuth tables only.
2. `Dockerfile` (multi-stage) + `docker-compose.yml` (app + postgres + named volume `dadtracker_pgdata`).
3. `.env.example` with `DATABASE_URL`, `NEXTAUTH_SECRET`, `NEXTAUTH_URL`.
4. README: "Run via Docker" section + "Run migrations" section.
- **Verify:** `docker compose up` boots, `npx prisma migrate dev` creates tables, `npx prisma studio` shows empty User table.

### Phase 2 — Auth (NextAuth + Credentials + bcrypt)
1. Install `next-auth@beta`, `@auth/prisma-adapter`, `bcryptjs`, `zod`.
2. `src/lib/auth.ts` — Credentials provider, Prisma adapter, JWT session strategy, bcrypt verify.
3. Signup API route — Zod-validate email/password, bcrypt hash (cost 12), create user.
4. Login + Signup pages, basic Tailwind styling.
5. Middleware redirects unauth users to `/login`.
6. **Security:** rate-limit signup + login (in-memory bucket fine for personal app; document upgrade path), security headers via `next.config.ts`, CSRF handled by NextAuth.
- **Verify:** signup → auto-login → protected dashboard. Logout works. Direct hit on `/dashboard` while logged out redirects to login.

### Phase 3 — Settings/Profile + units + goals math
1. Extend Prisma: `ProfileSettings` model (age, sex, weightKg, heightCm, activityLevel, unitsPreference, calorieGoalOverride, waterGoalMlOverride).
2. `src/lib/units.ts` (imperial↔metric, storage = metric).
3. `src/lib/goals.ts` — Mifflin-St Jeor + activity multipliers + 35 mL/kg water.
4. Settings page form (Zod-validated), shows computed goals live, manual override fields.
5. "Not medical advice" disclaimer banner on goals.
- **Verify:** enter age/weight/height as imperial, see metric stored in DB (Prisma Studio), goals computed correctly against a hand calculation.

### Phase 4 — Tasks (the centerpiece)
1. Schema: `Task` (title, description, dueAt, manualUrgency enum, urgencyPinned bool, status enum [TODO/IN_PROGRESS/DONE], completedAt, createdAt, order int, userId, tagIds), `Tag` (name, userId, color).
2. `src/lib/urgency.ts` — pure fn `effectiveUrgency({manualUrgency, dueAt, pinned, now})` → `'green'|'yellow'|'red'`. Unit-tested with vitest (a couple of focused tests, not exhaustive).
3. CRUD API routes — **every route checks `task.userId === session.user.id` before read/write**.
4. Kanban view with `@dnd-kit` (3 columns, drag between columns updates `status`; intra-column drag updates `order`).
5. Flat list view with sort/filter (urgency, dueAt, tag).
6. View toggle (Kanban ↔ List) persisted in localStorage.
7. Auto-archive cron-light: on each Tasks page load, server filters out `status=DONE AND completedAt < now-30d` from the default view; toggle "show archived" reveals them. (Avoids needing a real cron for a personal app.)
8. Tag CRUD (inline create from task editor).
- **Verify:** create tasks with/without due dates, see colors shift correctly as you change due date (use a future date close to now), pin urgency and confirm it doesn't auto-escalate, drag between columns, archive behavior works.

### Phase 5 — Calendar
1. Schema: `Event` (title, startAt, endAt, allDay, userId) — separate from Task, but calendar view pulls both.
2. FullCalendar month/week/day views.
3. Click empty day → modal to create Task (with due date prefilled) or Event.
4. Click existing item → edit modal.
5. Color dot/border on items uses same `effectiveUrgency` for tasks; events get a neutral color.
- **Verify:** task created in Tasks tab shows up on its due date in Calendar; clicking a day adds an item; edit/delete works.

### Phase 6 — Dashboard
1. "Quick add task" input (calls same Task POST endpoint).
2. Today's tasks list (status != DONE, due today or overdue).
3. Today's water progress ring + meal calorie progress bar (both pull from goals).
4. Next 3 upcoming calendar items.
- **Verify:** quick-add from dashboard appears in Tasks tab. Progress widgets reflect today's logs.

### Phase 7 — Workout tracker
1. Schema: `Workout` (date, userId), `ExerciseLog` (workoutId, name, kind enum [STRENGTH/CARDIO], sets, reps, weightKg, durationSec, distanceM, notes).
2. Workout entry form with dynamic exercise rows.
3. "Repeat last workout" — clones the most recent workout's exercises with empty values to fill in.
4. Per-exercise history chart (Recharts line chart, weight or duration over time).
- **Verify:** log 3 workouts on different dates; chart shows progression; repeat-last prefills correctly.

### Phase 8 — Food & water
1. Schema: `Meal` (name, calories, proteinG, carbsG, fatG, mealType enum, eatenAt, userId), `WaterLog` (amountMl, loggedAt, userId).
2. Meal log form, daily meal list grouped by mealType.
3. Water quick-add buttons (8 oz / 12 oz / 16 oz in imperial mode; 250 / 500 mL in metric).
4. Daily totals vs goal (progress bar for calories, ring for water).
5. Disclaimer banner reused from Settings.
- **Verify:** log meals, see calorie total tick up against goal; water buttons increment ring; goals respect overrides from Settings.

### Phase 9 — Hardening + README polish
1. Audit every API route for ownership check (`session.user.id === record.userId`).
2. Confirm `.env` ignored, `.env.example` accurate, no secrets in git history.
3. Add `helmet`-equivalent headers config in `next.config.ts` (CSP, X-Frame-Options, Referrer-Policy).
4. README: full feature list, both Docker and non-Docker setup paths, env vars, migration commands, screenshots placeholder, "personal/family project" note.
5. Final commit cleanup.
- **Verify:** `git log` reads cleanly; README setup steps work from a fresh clone on another folder.

## Security Notes (Public Repo)

- **Never** commit `.env*` (except `.env.example` with empty values).
- `NEXTAUTH_SECRET` generated via `openssl rand -base64 32`, only in `.env`.
- Default Postgres password in `docker-compose.yml` is for local dev only — README must say so explicitly.
- No real user data in seed scripts; no fixtures with PII.
- Audit before each push: `git diff --cached` for accidental secrets.

## Verification (end-to-end)

After Phase 9, the success criterion is:
1. Fresh clone → `cp .env.example .env` → fill values → `docker compose up` → `npx prisma migrate deploy` → app boots.
2. Signup, fill profile in Settings, see correct calorie/water goals.
3. Create tasks with mixed due dates → urgency colors shift correctly; drag between Kanban columns; toggle to list view.
4. Calendar reflects tasks; clicking a day creates an item.
5. Dashboard quick-add works; today's widgets update.
6. Log workout twice, see chart line; "repeat last" prefills.
7. Log meals + water, see totals vs goals.
8. Log out → all `/app/*` routes redirect to login.
9. Logged in as User A, try to fetch `/api/tasks/<a-task-id-belonging-to-User-B>` → 404 or 403 (IDOR check).

## Open Items Before Phase 0

I need from you at the start of Phase 0:
- **GitHub repo URL** (e.g., `https://github.com/<you>/dadtracker.git`).

Everything else is decided.
