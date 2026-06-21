# DadTracker

A personal productivity web app built as a Father's Day gift. Features task management (Kanban + list with smart urgency), calendar, workout logging, food & water tracking with personalized goals, and secure multi-user auth.

> This is a personal/family project, not intended for production-scale public deployment.

## Tech Stack

- **Framework:** Next.js 15 (App Router) + TypeScript
- **Database:** PostgreSQL + Prisma ORM
- **Auth:** Auth.js v5 (Credentials provider + bcrypt)
- **Styling:** Tailwind CSS
- **Charts:** Recharts
- **Drag & Drop:** @dnd-kit
- **Calendar:** FullCalendar
- **Validation:** Zod

## Features

- Secure login / signup (bcrypt hashed passwords, httpOnly session cookies)
- Task tracker with Kanban + list views, smart urgency escalation, drag-and-drop
- Calendar with month/week/day views, linked to tasks
- Workout log with exercise history charts
- Food & water tracker with personalized calorie/water goals (Mifflin-St Jeor)
- Settings/profile with imperial ↔ metric unit toggle

## Environment Variables

Copy `.env.example` to `.env` and fill in the values:

```bash
cp .env.example .env
```

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `NEXTAUTH_SECRET` | Random secret for Auth.js (generate with `openssl rand -base64 32`) |
| `NEXTAUTH_URL` | Base URL of the app (e.g. `http://localhost:3000`) |
| `POSTGRES_PASSWORD` | Postgres password for Docker (local dev only) |

## Setup — Via Docker (recommended)

```bash
# 1. Copy and fill env file
cp .env.example .env

# 2. Start app + postgres
docker compose up

# 3. In a separate terminal, run migrations
docker compose exec app npx prisma migrate deploy
```

App will be available at `http://localhost:3000`.

## Setup — Without Docker

Requires Node 20+ and a running PostgreSQL instance.

```bash
# 1. Install dependencies
npm install

# 2. Copy and fill env file
cp .env.example .env

# 3. Run migrations
npx prisma migrate dev

# 4. Start dev server
npm run dev
```

## Database Migrations

```bash
# Create and apply a new migration (dev only)
npx prisma migrate dev --name <migration-name>

# Apply existing migrations (production / Docker)
npx prisma migrate deploy

# Open Prisma Studio (visual DB browser)
npx prisma studio
```

## Screenshots

_Coming soon._
