# Nexora — Project Documentation

> **Nexora** (/nek-zɔː-ra/) — *"Điểm kết nối tri thức"*  
> A collaborative platform for university students: Task Management + Study Planner + Forum/Discussion + AI.

---

## Quick Start

**New to the project? Start here:**

1. Read [`docs/01-system-architecture.md`](01-system-architecture.md) — understand the big picture
2. Follow [`docs/guides/00-getting-started.md`](guides/00-getting-started.md) — set up your local environment
3. Go to your **role-specific guide** (table below)
4. Reference [`docs/02-feature-list-moscow.md`](02-feature-list-moscow.md) for what to build

---

## Role-Specific Entry Points

| Member | Role | First Guide |
|--------|------|-------------|
| **Member 1** (Leader) | Backend Lead + AI | [`guides/backend/01-setup.md`](guides/backend/01-setup.md) |
| **Member 2** | Backend: Schedule + Forum + Notif | [`guides/backend/01-setup.md`](guides/backend/01-setup.md) |
| **Member 3** | Frontend Web (Next.js) | [`guides/frontend/01-setup.md`](guides/frontend/01-setup.md) |
| **Member 4** | Mobile (React Native/Expo) | [`guides/mobile/01-setup.md`](guides/mobile/01-setup.md) |

---

## Document Map

### Overview Docs

| File | Purpose |
|------|---------|
| [`01-system-architecture.md`](01-system-architecture.md) | Brand identity, system diagram, tech stack, folder structures, deployment |
| [`02-feature-list-moscow.md`](02-feature-list-moscow.md) | All features with MoSCoW priority, MVP definition |
| [`03-team-assignment.md`](03-team-assignment.md) | Team of 4, roles, module ownership, deliverables |
| [`04-timeline.md`](04-timeline.md) | 6 phases March–August 2026, milestones, risks |

### Setup Guides (`guides/`)

| File | Purpose |
|------|---------|
| [`guides/00-getting-started.md`](guides/00-getting-started.md) | Env setup for all members (Node, pnpm, Docker, git) |
| [`guides/backend/01-setup.md`](guides/backend/01-setup.md) | NestJS scaffold, deps, env, Swagger, Docker Compose |
| [`guides/frontend/01-setup.md`](guides/frontend/01-setup.md) | Next.js 15, Tailwind v4, shadcn/ui, Axios, Zustand |
| [`guides/mobile/01-setup.md`](guides/mobile/01-setup.md) | Expo SDK 53, NativeWind, Expo Router, notifications |

### Team Conventions (`conventions/`)

| File | Purpose |
|------|---------|
| [`conventions/git-workflow.md`](conventions/git-workflow.md) | Branch naming, commit messages, PR rules |
| [`conventions/api-design.md`](conventions/api-design.md) | REST standards, response format, status codes |
| [`conventions/code-style.md`](conventions/code-style.md) | TypeScript, naming conventions, Prettier config |

### Module API References (`modules/`)

| Module | File | Status |
|--------|------|--------|
| **Auth** | [`modules/auth/api-endpoints.md`](modules/auth/api-endpoints.md) | ✅ Full (8 endpoints + User schema) |
| **Users** | [`modules/users/api-endpoints.md`](modules/users/api-endpoints.md) | ✅ Full (6 endpoints, avatar, stats) |
| **Groups** | [`modules/groups/api-endpoints.md`](modules/groups/api-endpoints.md) | ✅ Full (10 endpoints + Group schema) |
| **Tasks** | [`modules/tasks/api-endpoints.md`](modules/tasks/api-endpoints.md) | ✅ Full (Task CRUD, Board, Comments) |
| **Schedule** | [`modules/schedule/api-endpoints.md`](modules/schedule/api-endpoints.md) | 🔧 Placeholder — Member 2 fills in |
| **Forum** | [`modules/forum/api-endpoints.md`](modules/forum/api-endpoints.md) | 🔧 Placeholder — Member 2 fills in |
| **Notifications** | [`modules/notifications/api-endpoints.md`](modules/notifications/api-endpoints.md) | ✅ WebSocket events + schema |
| **AI** | [`modules/ai/api-endpoints.md`](modules/ai/api-endpoints.md) | ✅ Full (SSE stream, prompt templates) |

---

## Tech Stack at a Glance

| Layer | Technology |
|-------|-----------|
| **Backend** | NestJS 11 · TypeScript 5 · Mongoose 8 · Passport.js · Socket.io 4 · BullMQ 5 |
| **Frontend** | Next.js 15 (App Router) · Tailwind CSS 4 · shadcn/ui · TanStack Query 5 · Zustand 5 |
| **Mobile** | React Native 0.77 · Expo SDK 53 · Expo Router 4 · NativeWind |
| **Database** | MongoDB Atlas · Redis (Upstash) · Cloudinary |
| **AI** | OpenAI GPT-4o / GPT-4o-mini |
| **Deploy** | Vercel (FE) · Railway/Render (BE) · MongoDB Atlas |

---

## Repository Structure

```
GitHub Organization: nexora-app
├── nexora-be        → NestJS backend
├── nexora-fe        → Next.js frontend web
└── nexora-mobile    → Expo React Native app
```

---

## Build Order (Phase 1 Priority)

Follow this order when implementing to unblock other members:

```
1. Auth module (register, login, JWT)     ← unblocks everyone
2. Users module (profile CRUD)            ← needed by FE/Mobile
3. Groups module (create, invite, join)   ← unblocks task boards
4. Tasks module (CRUD + board)            ← core feature
5. Schedule module                        ← Member 2
6. Forum module                           ← Member 2
7. Notifications + AI                     ← final integration
```

---

## Useful Commands

```bash
# Backend
pnpm run start:dev          # Start NestJS dev server (port 3000)
pnpm run swagger            # View API docs at localhost:3000/api

# Frontend
pnpm run dev                # Start Next.js dev server (port 3001)

# Mobile
npx expo start              # Start Expo dev server
npx expo start --android    # Start on Android emulator

# Docker (local DB)
docker compose up -d        # Start MongoDB + Redis containers
docker compose down         # Stop containers
```

---

## Key Contacts / Ownership

| Module | Owner |
|--------|-------|
| Auth, Tasks, AI, Groups | Member 1 |
| Schedule, Forum, Notifications, Users | Member 2 |
| Web UI (all modules) | Member 3 |
| Mobile app (all modules) | Member 4 |

---

*Last updated: March 2026 · Nexora v1.0 documentation*
