# CLAUDE.md — DadTracker

This file is loaded automatically by Claude Code sessions working in this directory. Keep it concise — it's a primer, not documentation.

## What this project is

**DadTracker** — a multi-user productivity web app being built as a personal Father's Day gift. Features: tasks (Kanban + list with hybrid urgency), calendar, workout log, food/water tracking with personalized goals, secure auth.

**Audience:** the user's dad (single primary user, but built multi-user for proper auth practice).
**Repo:** public on GitHub — security and secrets handling are first-class.

## Where the plan lives

`BUILD_PLAN.md` (project root) is the source of truth for scope, phases, tech decisions, and verification criteria. **Always read it before starting work in a new session.**

## Locked-in stack

- **Next.js 15 (App Router) + TypeScript**
- **PostgreSQL + Prisma**
- **Auth.js (NextAuth v5)** — Credentials provider + bcrypt, httpOnly cookies
- **Tailwind CSS**, **Recharts**, **@dnd-kit**, **FullCalendar**, **Zod**
- **Docker** + **docker-compose** (app + postgres + named volume)

## Project conventions (don't drift from these)

- **Units:** internal storage is **always metric** (kg, cm, mL). Imperial is a display/input toggle only. Imperial is the default for the UI since this is a US user.
- **Urgency thresholds:** green > 3 days out, yellow ≤ 3 days, red ≤ 24h or overdue. Per-task `urgencyPinned` boolean bypasses auto-escalation.
- **IDOR:** every API route that touches user-owned data **must** re-check `session.user.id === record.userId` before read/write. No exceptions.
- **Secrets:** `.env*` is git-ignored except `.env.example` (variable names only, empty values). Never commit real secrets. Audit `git diff --cached` before each push.
- **Disclaimer:** any computed health goal (calories, water) shows a "not medical advice" note.
- **Commits:** small and logical. One commit per meaningful sub-step within a phase. Phase boundaries are good commit boundaries.
- **README:** update at the end of every phase so it never goes stale.

## How to work in this repo

The user wants **incremental, phase-by-phase delivery with verification before moving on**. After each phase:
1. Tell the user how to test it locally.
2. Wait for confirmation before starting the next phase.
3. Update README.

Ask before destructive ops (force push, reset --hard, deleting files you didn't create). Ask before scope expansion.

## Current state

Check `git log` and `BUILD_PLAN.md` to see which phase is active. As of initial setup: project not yet scaffolded; GitHub repo URL pending from user.
