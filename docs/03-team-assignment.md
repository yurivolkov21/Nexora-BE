# Phân công nhóm — Nexora

## Tổng quan nhóm

| Thành viên | Vai trò | Tech Stack chính |
|---|---|---|
| **Member 1 (Leader)** | Backend Lead + AI Module | NestJS, MongoDB, OpenAI API, CI/CD |
| **Member 2** | Backend Support | NestJS, Redis, WebSocket, Firebase FCM |
| **Member 3** | Frontend Web | Next.js, Tailwind CSS, TanStack Query |
| **Member 4** | Mobile | React Native, Expo, Expo Router |

> **Lưu ý:** Member 3 và Member 4 bắt đầu sau khi BE đã có API Auth + Task CRUD cơ bản để không bị block.

---

## Member 1 — Backend Lead + AI Module

### Vai trò
- Thiết kế kiến trúc hệ thống tổng thể
- Setup project ban đầu, CI/CD pipeline
- Chịu trách nhiệm review code toàn team

### Phụ trách modules

#### Auth Module
- Đăng ký / Đăng nhập email + password
- Google OAuth (Passport.js)
- JWT (Access Token + Refresh Token, HTTP-only cookie)
- Middleware phân quyền (Role Guard)
- Forgot password (OTP email)

#### Task Module
- CRUD Task đầy đủ
- Kanban logic (cột, thứ tự drag & drop)
- Assign task, label, priority, filter, search
- Task activity log

#### Groups Module
- CRUD Group
- Invite link + email invite
- Role management (owner/admin/member)

#### AI Module
- Tích hợp OpenAI API (GPT-4o)
- AI Chatbot (streaming response)
- AI gợi ý phân chia task từ mô tả project
- AI tóm tắt bài Forum
- Smart deadline suggestion
- Lưu conversation history vào MongoDB

#### DevOps / Setup
- Docker Compose cho local dev (NestJS + MongoDB + Redis)
- Setup Swagger (OpenAPI docs đầy đủ)
- Environment config (.env, validation với Joi)
- Deploy NestJS lên Railway
- GitHub Actions CI (lint + test on PR)

### Deliverables chính

```
Week 1–2  : Project setup, Auth API hoàn chỉnh
Week 3–4  : Task API + Groups API
Week 5–6  : AI Module
Week 7+   : Review code, fix bug, support members
```

---

## Member 2 — Backend Support

### Vai trò
- Xây dựng các module hỗ trợ phía BE
- Đảm bảo hệ thống real-time và thông báo hoạt động ổn định

### Phụ trách modules

#### Schedule Module
- CRUD sự kiện / lịch học
- Logic recurring events
- Gắn task deadline lên calendar
- Phân loại sự kiện theo môn/dự án

#### Forum Module
- CRUD Post (title, rich text content, category, tags)
- Threaded comments (reply to comment)
- Upvote / Downvote logic
- Ghim bài, báo cáo vi phạm
- Full-text search (MongoDB text index)

#### Notification Module
- WebSocket Gateway (Socket.io với NestJS)
- Emit event khi: task assigned, deadline gần, mention, reply
- BullMQ Job Queue để xử lý notification bất đồng bộ
- Firebase FCM integration (push notification mobile)
- NodeMailer + Gmail SMTP (email notification)
- Notification CRUD (mark read, delete, paginate)

#### User Module (hỗ trợ Member 1)
- Cập nhật profile (avatar upload qua Cloudinary)
- Đổi mật khẩu
- Thống kê cá nhân (số task hoàn thành, giờ học...)

### Deliverables chính

```
Week 1–2  : Schedule API + Forum API cơ bản
Week 3–4  : Notification system (WebSocket + FCM)
Week 5–6  : Email notification + BullMQ queue
Week 7+   : Optimize, fix bug, viết unit test
```

---

## Member 3 — Frontend Web (Next.js)

### Vai trò
- Xây dựng toàn bộ giao diện web
- Đảm bảo UI/UX nhất quán, responsive

### Phụ trách pages & components

#### Auth Pages
- `/login` — Form đăng nhập + Google OAuth button
- `/register` — Form đăng ký
- `/forgot-password` — Quên mật khẩu

#### Dashboard
- `/dashboard` — Tổng quan: task hôm nay, sự kiện sắp đến, activity feed
- Widget: biểu đồ task theo trạng thái, upcoming deadlines

#### Task Management
- `/tasks` — Danh sách tất cả workspace
- `/tasks/[boardId]` — Kanban Board (drag & drop với `@hello-pangea/dnd`)
- Modal: tạo/sửa task, assign, label, deadline picker

#### Schedule
- `/schedule` — Calendar view (month/week/day) với `react-big-calendar` hoặc `FullCalendar`
- Pomodoro timer component
- Progress tracking chart (Recharts)

#### Forum
- `/forum` — Danh sách bài đăng, filter theo category/tag
- `/forum/[postId]` — Chi tiết bài + threaded comments
- `/forum/new` — Tạo bài mới (rich text editor: Tiptap hoặc Quill)

#### Groups
- `/groups` — Danh sách nhóm
- `/groups/[groupId]` — Trang nhóm: members, boards, discussion

#### AI Assistant
- Floating AI chat button (có thể mở từ bất kỳ trang nào)
- AI chat panel với streaming response

#### Layout & Common
- Sidebar navigation
- Header với search, notification bell, avatar
- Notification dropdown (real-time với Socket.io)
- Dark mode toggle
- Toast notifications

### Tech stack sử dụng

```
Next.js 15 (App Router)
TypeScript
Tailwind CSS + shadcn/ui
TanStack Query v5       — API calls, caching
Zustand                 — Global state (auth, UI)
Axios                   — HTTP client
Socket.io-client        — Real-time
React Hook Form + Zod   — Form validation
Recharts                — Charts
Tiptap                  — Rich text editor
@hello-pangea/dnd       — Drag & drop
date-fns                — Date utilities
```

### Deliverables chính

```
Week 1    : Setup Next.js, layout, auth pages
Week 2–3  : Dashboard + Task Board (sau khi BE có API)
Week 4    : Schedule + AI chat
Week 5–6  : Forum + Groups
Week 7+   : Polish UI, responsive, dark mode, fix bug
```

---

## Member 4 — Mobile (React Native + Expo)

### Vai trò
- Xây dựng toàn bộ ứng dụng mobile
- Tái sử dụng logic/hooks từ FE Web khi có thể

### Phụ trách screens

#### Auth Screens
- `LoginScreen` — Đăng nhập email + Google
- `RegisterScreen` — Đăng ký

#### Main Tabs (Bottom Navigation)
- `HomeScreen` — Dashboard: task hôm nay, sự kiện sắp tới, notifications
- `TasksScreen` — Danh sách task (list view, không có Kanban vì màn hình nhỏ)
- `ScheduleScreen` — Lịch học (week view), Pomodoro timer
- `ForumScreen` — Danh sách bài đăng, xem chi tiết, comment
- `ProfileScreen` — Thông tin cá nhân, settings

#### Detail Screens
- `TaskDetailScreen` — Xem/sửa task, checklist, comment
- `PostDetailScreen` — Chi tiết bài Forum + comments
- `GroupScreen` — Thông tin nhóm, danh sách members
- `NotificationScreen` — Tất cả thông báo
- `AIScreen` — AI Chat assistant
- `PomodoroScreen` — Full-screen Pomodoro timer

#### Settings
- `SettingsScreen` — Thông báo, appearance, account

### Tech stack sử dụng

```
React Native 0.77   — New Architecture bật mặc định
Expo SDK 53
Expo Router v4      — File-based navigation
TanStack Query v5   — API state (tái dùng từ web hooks)
Zustand             — Client state
Axios               — HTTP client
Expo Notifications  — Push notification handler
AsyncStorage        — Local storage (token)
React Native Reanimated — Animations
NativeWind          — Tailwind cho React Native
date-fns            — Date utilities
```

### Tái sử dụng từ FE Web

```
/shared/
  ├── hooks/
  │   ├── useAuth.ts
  │   ├── useTasks.ts
  │   ├── useSchedule.ts
  │   └── useNotifications.ts
  ├── stores/
  │   └── authStore.ts
  ├── types/
  │   └── index.ts
  └── lib/
      ├── api.ts
      └── utils.ts
```
> Tạo package `@nexora/shared` hoặc dùng monorepo (Turborepo) để chia sẻ code.

### Deliverables chính

```
Week 1    : Setup Expo, navigation structure, auth screens
Week 2–3  : HomeScreen + TasksScreen (sau khi BE có API)
Week 4    : ScheduleScreen + Pomodoro
Week 5–6  : Forum + Profile + Notification handler
Week 7+   : AI Chat screen, push notification, build APK/IPA với EAS
```

---

## Quy trình làm việc nhóm

### Git Workflow

```
main
  └── develop
        ├── feature/m1-auth-module          (Member 1)
        ├── feature/m1-task-module          (Member 1)
        ├── feature/m2-schedule-module      (Member 2)
        ├── feature/m2-notification         (Member 2)
        ├── feature/m3-dashboard-ui         (Member 3)
        ├── feature/m3-task-board-ui        (Member 3)
        ├── feature/m4-auth-screens         (Member 4)
        └── feature/m4-task-screens         (Member 4)
```

### Quy tắc merge

1. Không push thẳng lên `main` hoặc `develop`
2. Tạo Pull Request → ít nhất 1 người review → mới merge
3. `develop` → `main` chỉ khi milestone hoàn thành

### Công cụ cộng tác nhóm

| Mục đích | Công cụ |
|---|---|
| Code repository | GitHub |
| Giao tiếp hàng ngày | Discord / Zalo nhóm |
| Quản lý task nhóm | GitHub Projects hoặc Notion |
| Tài liệu chung | Notion hoặc Google Docs |
| Thiết kế UI | Figma (optional) |
| API testing | Postman / Swagger UI |

### Lịch sync nhóm (đề xuất)

- **Thứ 2:** Weekly planning — phân công task tuần mới
- **Thứ 6:** Weekly review — demo những gì đã làm, blockers
- **Hàng ngày:** Update trạng thái ngắn trong Discord (đang làm gì, bị block không)
