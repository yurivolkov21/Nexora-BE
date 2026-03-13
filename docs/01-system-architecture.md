# Kiến trúc hệ thống — Nexora

## Tên dự án

| | |
|---|---|
| **Tên** | Nexora |
| **Phát âm** | /nek-zɔː-ra/ — *"nek-ZOH-ra"* |
| **Slogan** | *"Where collaboration meets intelligence"* |
| **Domain dự kiến** | `nexora.app` |
| **Repo BE** | `nexora-be` |
| **Repo FE** | `nexora-fe` |
| **Repo Mobile** | `nexora-mobile` |

### Ý nghĩa tên

**Nexora** được ghép từ 2 nguồn gốc:

- **Nex-** — từ *"Next"* (tiếp theo, thế hệ mới) và *"Nexus"* (điểm kết nối, mối liên kết)
- **-ora** — từ tiếng Latin *"orare"* (nói, chia sẻ, giao tiếp)

> Nexora = **Nền tảng kết nối thế hệ mới, nơi mọi người giao tiếp và cộng tác thông minh hơn.**

### Lý do chọn tên này

1. **Không bị giới hạn bởi tính năng** — Không mang nghĩa cụ thể như "StudyApp" hay "TaskManager", dễ dàng mở rộng sang các domain khác khi scale
2. **Âm thanh hiện đại** — Suffix *-ora* tạo cảm giác tech nhưng gần gũi (tương tự Canva, Figma, Vercel)
3. **Dễ nhớ và dễ gõ** — 3 âm tiết, không có ký tự khó, phát âm được bằng cả tiếng Anh lẫn tiếng Việt
4. **Tiềm năng domain** — Tên đủ độc đáo để có khả năng đăng ký `.app` hoặc `.io`

---

## Tổng quan

**Nexora** là nền tảng cộng tác thông minh dành cho nhóm sinh viên, hỗ trợ quản lý công việc, lên kế hoạch học tập và thảo luận cộng đồng, được tích hợp AI.

---

## Sơ đồ kiến trúc tổng thể

```
┌──────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER                            │
│                                                                  │
│   ┌─────────────────────┐        ┌─────────────────────────┐     │
│   │    Next.js (Web)    │        │  React Native (Mobile)  │     │
│   │                     │        │                         │     │
│   │  - Dashboard        │        │  - Notifications        │     │
│   │  - Kanban Board     │        │  - Quick task update    │     │
│   │  - Forum            │        │  - Schedule viewer      │     │
│   │  - Schedule         │        │  - AI chat              │     │
│   │  - AI Assistant     │        │  - Pomodoro timer       │     │
│   └──────────┬──────────┘        └────────────┬────────────┘     │
└──────────────┼────────────────────────────────┼──────────────────┘
               │           REST API / WS        │
               ▼                                ▼
┌──────────────────────────────────────────────────────────────────┐
│                         API LAYER (NestJS)                       │
│                                                                  │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐  ┌────────────┐     │
│  │   Auth    │  │   Tasks   │  │ Schedule  │  │   Forum    │     │
│  │  Module   │  │  Module   │  │  Module   │  │   Module   │     │
│  └───────────┘  └───────────┘  └───────────┘  └────────────┘     │
│                                                                  │
│  ┌───────────┐  ┌───────────┐  ┌────────────────────────────┐    │
│  │  Groups   │  │    AI     │  │  Notification Module       │    │
│  │  Module   │  │  Module   │  │  (WebSocket + FCM)         │    │
│  └───────────┘  └───────────┘  └────────────────────────────┘    │
└──────────────────────────────┬───────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────┐
│                          DATA LAYER                              │
│                                                                  │
│   ┌──────────────┐    ┌──────────────┐    ┌──────────────────┐   │
│   │ MongoDB Atlas│    │    Redis     │    │   Cloudinary     │   │
│   │              │    │              │    │                  │   │
│   │ Main database│    │ Cache +      │    │ File / Image     │   │
│   │ Collections: │    │ Session +    │    │ Storage          │   │
│   │ - users      │    │ Rate Limit   │    │ (avatar, attach) │   │
│   │ - tasks      │    └──────────────┘    └──────────────────┘   │
│   │ - groups     │                                               │
│   │ - schedules  │                                               │
│   │ - posts      │                                               │
│   │ - comments   │                                               │
│   │ - notifs     │                                               │
│   └──────────────┘                                               │
└──────────────────────────────┬───────────────────────────────────┘
                               │
┌──────────────────────────────▼───────────────────────────────────┐
│                       EXTERNAL SERVICES                          │
│                                                                  │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────────┐  │
│  │  OpenAI API  │   │ Firebase FCM │   │    NodeMailer        │  │
│  │              │   │              │   │    (+ SMTP Gmail)    │  │
│  │ - GPT-4o     │   │ Push notif   │   │                      │  │
│  │ - Embeddings │   │ cho mobile   │   │ - Deadline reminder  │  │
│  │ - AI Chat    │   └──────────────┘   │ - Welcome email      │  │
│  └──────────────┘                      └──────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

---

## Tech Stack chi tiết

### Frontend — Web

| Công nghệ | Phiên bản | Mục đích |
|---|---|---|
| Next.js | 15.x | Framework chính, App Router, SSR/SSG |
| TypeScript | 5.x | Type safety |
| Tailwind CSS | **4.x** ⚠️ | Styling (breaking change so với v3, xem ghi chú bên dưới) |
| shadcn/ui | Latest | UI Components (đã hỗ trợ Tailwind v4) |
| TanStack Query | 5.x | Server state management, caching |
| Zustand | **5.x** | Client state management |
| Socket.io-client | 4.x | Real-time updates |
| React Hook Form + Zod | Latest | Form validation |

### Frontend — Mobile

| Công nghệ | Phiên bản | Mục đích |
|---|---|---|
| React Native | **0.77.x** | Framework mobile (New Architecture bật mặc định) |
| Expo | **SDK 53** | Toolchain, build, OTA update |
| Expo Router | 4.x | File-based navigation |
| TanStack Query | 5.x | API state (tái sử dụng từ web) |
| Zustand | **5.x** | Client state (tái sử dụng từ web) |
| Expo Notifications | Latest | Push notification handler |

### Backend — API

| Công nghệ | Phiên bản | Mục đích |
|---|---|---|
| NestJS | 11.x | Framework chính |
| TypeScript | 5.x | Type safety |
| Mongoose | 8.x | MongoDB ODM |
| Passport.js | Latest | Authentication strategies |
| Socket.io | 4.x | WebSocket real-time |
| Bull (BullMQ) | 5.x | Job queue (email, notification) |
| class-validator | Latest | DTO validation |
| Swagger (OpenAPI) | Latest | API documentation |

### Database & Infrastructure

| Công nghệ | Mục đích |
|---|---|
| MongoDB Atlas | Primary database (free tier M0) |
| Redis (Upstash) | Session cache, rate limiting, Bull queue |
| Cloudinary | Image & file storage |
| Firebase FCM | Mobile push notifications |
| Vercel | Deploy Next.js |
| Railway / Render | Deploy NestJS |

---

## Lưu ý về phiên bản khi cài đặt

> Khi chạy lệnh cài đặt, mặc định sẽ lấy phiên bản mới nhất (latest) — điều này là bình thường và khuyến khích. Tuy nhiên, có một số package cần chú ý:

### ⚠️ Tailwind CSS v4 — Thay đổi quan trọng

Tailwind v4 (ra đầu 2025) có nhiều thay đổi breaking so với v3:

```bash
# Cài Tailwind v4 cho Next.js
pnpm add tailwindcss @tailwindcss/postcss
```

- **Không còn file `tailwind.config.js`** — config chuyển vào `globals.css` bằng `@theme`
- **PostCSS plugin đổi tên** từ `tailwindcss` sang `@tailwindcss/postcss`
- **shadcn/ui** đã cập nhật hỗ trợ Tailwind v4, dùng lệnh init mới nhất
- Xem migration guide: https://tailwindcss.com/docs/v4-beta

### ⚠️ React Native 0.77 — New Architecture mặc định

- New Architecture (Fabric + TurboModules) được bật mặc định từ 0.76+
- Một số thư viện cũ chưa tương thích — kiểm tra trước khi cài
- Dùng Expo SDK 53 sẽ tự xử lý phần lớn compatibility này

### Cách cài đặt an toàn nhất

```bash
# Backend (NestJS) — không cần chỉ định version
npx @nestjs/cli new nexora-be

# Frontend (Next.js)
pnpm create next-app@latest nexora-fe
# → Tự chọn: TypeScript ✓, Tailwind ✓, App Router ✓

# Mobile (Expo)
npx create-expo-app@latest nexora-mobile
```

### Packages nên dùng `@latest` khi cài thêm

```bash
# BE — cài thêm packages
pnpm add @nestjs/mongoose mongoose @nestjs/jwt passport-jwt
pnpm add @nestjs/swagger class-validator class-transformer
pnpm add bullmq @nestjs/bull socket.io

# FE — cài thêm packages
pnpm add @tanstack/react-query zustand axios zod
pnpm add react-hook-form socket.io-client
pnpm add recharts date-fns
```

---

## Luồng dữ liệu chính

### Authentication Flow

```
Client → POST /auth/login → Validate credentials → Issue JWT (AT + RT)
       → Store RT in HTTP-only cookie
       → Return AT in response body
       → Client lưu AT trong memory (không localStorage)
```

### Real-time Notification Flow

```
Server event (deadline sắp đến / task được assign)
  → BullMQ Job Queue
  → Notification Worker
  ├── → Socket.io emit (web client)
  ├── → Firebase FCM (mobile push)
  └── → NodeMailer (email, nếu quan trọng)
```

### AI Feature Flow

```
User request (gợi ý phân chia task / tóm tắt / chatbot)
  → NestJS AI Module
  → OpenAI API (GPT-4o)
  → Stream response về client
  → Lưu history vào MongoDB
```

---

## Cấu trúc thư mục

### Backend (NestJS)

```
nexora-be/
├── src/
│   ├── modules/
│   │   ├── auth/
│   │   │   ├── auth.controller.ts
│   │   │   ├── auth.service.ts
│   │   │   ├── auth.module.ts
│   │   │   ├── strategies/         # JWT, Google OAuth
│   │   │   └── guards/
│   │   ├── users/
│   │   ├── tasks/
│   │   ├── groups/
│   │   ├── schedule/
│   │   ├── forum/
│   │   ├── notifications/
│   │   └── ai/
│   ├── common/
│   │   ├── decorators/
│   │   ├── filters/                # Exception filters
│   │   ├── interceptors/
│   │   ├── pipes/
│   │   └── guards/
│   ├── config/
│   │   ├── database.config.ts
│   │   ├── jwt.config.ts
│   │   └── app.config.ts
│   ├── shared/
│   │   └── schemas/                # Mongoose schemas
│   └── main.ts
├── test/
├── .env
├── .env.example
├── docker-compose.yml
└── package.json
```

### Frontend Web (Next.js)

```
nexora-fe/
├── src/
│   ├── app/
│   │   ├── (auth)/
│   │   │   ├── login/
│   │   │   └── register/
│   │   ├── (dashboard)/
│   │   │   ├── dashboard/
│   │   │   ├── tasks/
│   │   │   ├── schedule/
│   │   │   ├── forum/
│   │   │   └── groups/
│   │   └── layout.tsx
│   ├── components/
│   │   ├── ui/                     # shadcn components
│   │   ├── task/
│   │   ├── schedule/
│   │   ├── forum/
│   │   └── ai/
│   ├── hooks/                      # Custom hooks (tái dùng cho mobile)
│   ├── lib/
│   │   ├── api.ts                  # Axios instance
│   │   └── utils.ts
│   ├── stores/                     # Zustand stores
│   └── types/                      # Shared TypeScript types
├── public/
└── package.json
```

---

## Môi trường (Environments)

| Env | BE | FE Web | Database |
|---|---|---|---|
| **Development** | localhost:3000 | localhost:3001 | MongoDB local / Atlas |
| **Staging** | Railway (branch staging) | Vercel (preview) | Atlas M0 staging |
| **Production** | Railway (main) | Vercel (main) | Atlas M0 production |

---

## Quy ước Git

### Branch naming

```
main                    # Production-ready
develop                 # Integration branch
feature/[tên-tính-năng] # Feature mới
fix/[mô-tả-bug]         # Bugfix
chore/[mô-tả]           # Config, deps, setup
```

### Commit message (Conventional Commits)

```
feat: add kanban board drag-and-drop
fix: resolve JWT refresh token expiry bug
chore: update NestJS dependencies
docs: add API documentation for tasks module
refactor: extract notification service
test: add unit tests for auth service
```
