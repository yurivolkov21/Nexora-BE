# Timeline dự án — Nexora

## Tổng quan

| Thông tin | Chi tiết |
|---|---|
| **Bắt đầu** | Tháng 3/2026 |
| **Deadline mục tiêu** | Cuối tháng 8/2026 |
| **Tổng thời gian** | ~6 tháng |
| **Nhóm** | 4 thành viên |
| **Buffer time** | Đã tính vào mỗi giai đoạn (~20% thời gian dự phòng) |

> **Lưu ý:** Timeline này đã tính thêm thời gian học công nghệ mới, viết test, fix bug và deploy. Không phải toàn bộ thời gian là code tính năng mới.

---

## Sơ đồ timeline tổng quan

```
THÁNG 3          THÁNG 4          THÁNG 5          THÁNG 6          THÁNG 7          THÁNG 8
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│   PHASE 0    │ │   PHASE 1    │ │   PHASE 2    │ │   PHASE 3    │ │   PHASE 4    │ │   PHASE 5    │
│   Setup &    │ │   Core BE    │ │   FE Web +   │ │   Mobile +   │ │   AI + Tích  │ │   Polish +   │
│   Learning   │ │   APIs       │ │   Core UI    │ │   Forum      │ │   hợp hoàn   │ │   Deploy +   │
│              │ │              │ │              │ │              │ │   chỉnh      │ │   Bảo vệ     │
│ 2–3 tuần     │ │ 4 tuần       │ │ 4 tuần       │ │ 4 tuần       │ │ 3 tuần       │ │ 3 tuần       │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
```

---

## Phase 0 — Setup & Learning (Tháng 3, tuần 1–2)

**Mục tiêu:** Cả nhóm đồng bộ kiến thức, setup môi trường, thống nhất quy ước.

### Tuần 1 (7–14/3)

#### Cả nhóm
- [ ] Brief ý tưởng dự án cho toàn team
- [ ] Thống nhất tech stack, quy ước Git, commit message
- [ ] Tạo 2 repo GitHub: `nexora-be` và `nexora-fe`
- [ ] Tạo không gian cộng tác: Discord/Zalo nhóm, Notion workspace
- [ ] Setup môi trường dev cá nhân (Node.js 20+, pnpm, MongoDB Compass)

#### Member 1 (Leader)
- [ ] Scaffold NestJS project (`nest new nexora-be`)
- [ ] Setup `.env`, Joi validation, Swagger
- [ ] Connect MongoDB Atlas, setup Mongoose
- [ ] Tạo Docker Compose cho local dev
- [ ] Setup GitHub Actions CI cơ bản

#### Member 3 (FE Web)
- [ ] Scaffold Next.js project (`pnpm create next-app nexora-fe`)
- [ ] Cấu hình Tailwind CSS, shadcn/ui
- [ ] Setup folder structure, Axios instance, TanStack Query

#### Member 4 (Mobile)
- [ ] Setup Expo project (`npx create-expo-app nexora-mobile`)
- [ ] Cài Expo Router, NativeWind
- [ ] Setup Axios instance, TanStack Query

### Tuần 2 (15–21/3) — Learning Sprint

> Mỗi member tự học công nghệ mình phụ trách trước khi vào code thật.

| Member | Nội dung học |
|---|---|
| Member 1 | TypeScript nâng cao, NestJS core (Module/Controller/Service/Guard), Mongoose |
| Member 2 | NestJS WebSocket (Socket.io), BullMQ, Firebase FCM, NodeMailer |
| Member 3 | Next.js App Router, TanStack Query, Zustand, shadcn/ui |
| Member 4 | Expo Router, React Native basics, TanStack Query trên mobile, Expo Notifications |

### Deliverables Phase 0
```
✅ 2 repo đã được khởi tạo, có cấu trúc thư mục chuẩn
✅ Môi trường dev của tất cả member đã chạy được
✅ Swagger UI accessible tại localhost:3000/api
✅ Hello World API đã hoạt động
✅ Tài liệu docs/ đã được sync lên repo
```

---

## Phase 1 — Core Backend APIs (Tháng 4, tuần 3–6)

**Mục tiêu:** Hoàn thiện các API nền tảng để FE và Mobile bắt đầu tích hợp được.

### Tuần 3–4 (22/3 – 4/4): Auth + Task API

#### Member 1
- [ ] Auth Module: đăng ký, đăng nhập, JWT, Refresh Token
- [ ] Google OAuth (Passport.js)
- [ ] Role Guard (admin/member)
- [ ] Forgot password flow (OTP email)
- [ ] Task Module: CRUD task đầy đủ
- [ ] Kanban logic: thứ tự cột, kéo thả

#### Member 2
- [ ] User Module: CRUD profile, upload avatar (Cloudinary)
- [ ] Group Module: tạo nhóm, invite via email/link, role management
- [ ] Schedule Module: CRUD sự kiện, recurring events, gắn task deadline

### Tuần 5–6 (5–18/4): Forum + Notification API

#### Member 1
- [ ] Code review và hỗ trợ Member 2
- [ ] Viết unit test cho Auth Module (Jest)
- [ ] Tối ưu API (indexing MongoDB, response time)

#### Member 2
- [ ] Forum Module: CRUD post, comment, vote, tag, search
- [ ] Notification Module: WebSocket Gateway (Socket.io)
- [ ] BullMQ job queue cho notification bất đồng bộ
- [ ] Firebase FCM setup

### Deliverables Phase 1
```
✅ Tất cả Must Have API đã có endpoint, documented trên Swagger
✅ Postman collection export và share cho Member 3, 4
✅ Auth flow hoạt động end-to-end (register → login → protected route)
✅ Task CRUD hoạt động
✅ WebSocket notification hoạt động (test bằng Postman WS)
```

---

## Phase 2 — Frontend Web Core UI (Tháng 4 cuối – Tháng 5, tuần 7–10)

**Mục tiêu:** Hoàn thiện giao diện web cho các tính năng lõi.

### Tuần 7–8 (19/4 – 2/5): Auth + Dashboard + Task Board

#### Member 3
- [ ] Auth pages: Login, Register, Forgot Password
- [ ] Layout: Sidebar, Header, Notification bell
- [ ] Dashboard: tổng quan task, upcoming deadlines, activity feed
- [ ] Kanban Board: kéo thả task giữa các cột (`@hello-pangea/dnd`)
- [ ] Modal: tạo/sửa task, assign, label, deadline picker
- [ ] Tích hợp TanStack Query với Task API + Auth API

#### Member 1 (hỗ trợ)
- [ ] Fix bug API được Member 3 báo cáo

### Tuần 9–10 (3–16/5): Schedule + Groups

#### Member 3
- [ ] Schedule page: Calendar view (month/week/day)
- [ ] Pomodoro timer component
- [ ] Progress tracking chart (Recharts)
- [ ] Groups page: tạo nhóm, danh sách members, board của nhóm
- [ ] Invite member flow (email + link)

### Deliverables Phase 2
```
✅ Web app có thể đăng ký, đăng nhập, xem dashboard
✅ Tạo/sửa/xóa task, kéo thả trên Kanban Board
✅ Xem lịch học, tạo sự kiện trên calendar
✅ Tạo nhóm, mời thành viên
✅ Notification real-time hoạt động trên web (Socket.io)
```

---

## Phase 3 — Mobile App + Forum (Tháng 5 cuối – Tháng 6, tuần 11–14)

**Mục tiêu:** Hoàn thiện mobile app và module Forum.

### Tuần 11–12 (17–30/5): Mobile Core Screens

#### Member 4
- [ ] Auth screens: Login, Register
- [ ] HomeScreen: dashboard tóm tắt
- [ ] TasksScreen: list view tasks, filter
- [ ] TaskDetailScreen: xem chi tiết, cập nhật trạng thái
- [ ] ScheduleScreen: week calendar view
- [ ] Pomodoro timer screen
- [ ] Tích hợp với API (Auth + Task + Schedule)

#### Member 2 (hỗ trợ)
- [ ] Hoàn thiện Firebase FCM, test push notification trên Expo Go

### Tuần 13–14 (31/5 – 13/6): Forum + Notification trên Mobile + Web

#### Member 3
- [ ] Forum pages trên web: danh sách post, chi tiết, tạo bài mới (Tiptap editor)
- [ ] Notification page: lịch sử thông báo, mark as read

#### Member 4
- [ ] ForumScreen trên mobile: xem danh sách, chi tiết, comment
- [ ] NotificationScreen: danh sách thông báo
- [ ] ProfileScreen + Settings

### Deliverables Phase 3
```
✅ Mobile app chạy được trên Expo Go (Android + iOS)
✅ Auth, Task, Schedule hoạt động đầy đủ trên mobile
✅ Push notification nhận được trên mobile
✅ Forum hoạt động trên cả web và mobile
✅ Notification system hoàn chỉnh (web + mobile + email)
```

---

## Phase 4 — AI Features + Integration (Tháng 6 cuối – Tháng 7, tuần 15–17)

**Mục tiêu:** Tích hợp AI, hoàn thiện các tính năng Should Have, đảm bảo hệ thống nhất quán.

### Tuần 15 (14–20/6): AI Module

#### Member 1
- [ ] AI Chatbot (streaming response với OpenAI GPT-4o)
- [ ] AI gợi ý phân chia task từ mô tả project
- [ ] Lưu conversation history

#### Member 3
- [ ] AI floating chat panel trên web
- [ ] Streaming UI (hiển thị từng token response)

#### Member 4
- [ ] AIScreen trên mobile

### Tuần 16 (21–27/6): AI nâng cao + Should Have features

#### Member 1
- [ ] AI tóm tắt bài đăng Forum
- [ ] Smart deadline suggestion
- [ ] Weekly summary AI

#### Member 2
- [ ] Hoàn thiện email digest notification
- [ ] Tối ưu BullMQ queue

#### Member 3
- [ ] Dark mode
- [ ] Responsive hoàn chỉnh (mobile web)
- [ ] Task: Calendar view, List view

### Tuần 17 (28/6 – 4/7): Integration testing

- [ ] Cả nhóm: kiểm tra end-to-end toàn bộ luồng chính
- [ ] Fix các bug tích hợp giữa FE, Mobile, BE
- [ ] Performance check: API response time, UI lag

### Deliverables Phase 4
```
✅ AI Chatbot hoạt động trên web và mobile
✅ AI gợi ý task hoạt động
✅ Dark mode trên web
✅ Tất cả Must Have + phần lớn Should Have đã xong
✅ Không còn bug nghiêm trọng ảnh hưởng đến luồng chính
```

---

## Phase 5 — Polish, Deploy & Bảo vệ (Tháng 7 cuối – Tháng 8, tuần 18–20)

**Mục tiêu:** Hoàn thiện chất lượng, deploy production, chuẩn bị hồ sơ đồ án.

### Tuần 18 (5–11/7): Bug fixing + Testing

- [ ] Viết unit test cho các module quan trọng (Auth, Task, AI) — Member 1, 2
- [ ] E2E test cơ bản trên web (Playwright) — Member 3
- [ ] Test trên nhiều thiết bị Android/iOS — Member 4
- [ ] Fix toàn bộ bug được báo cáo trong Phase 4

### Tuần 19 (12–18/7): Deploy & Performance

#### Member 1
- [ ] Deploy NestJS lên Railway (production environment)
- [ ] Setup MongoDB Atlas production cluster
- [ ] Configure Redis (Upstash production)
- [ ] Environment variables, secrets management

#### Member 3
- [ ] Deploy Next.js lên Vercel
- [ ] Configure domain, SSL

#### Member 4
- [ ] Build APK với Expo EAS Build (Android)
- [ ] Build IPA nếu có Apple Developer Account (iOS)
- [ ] Test APK trên thiết bị thật

#### Cả nhóm
- [ ] Load testing API cơ bản
- [ ] Tối ưu: lazy loading, image optimization, API caching

### Tuần 20 (19–25/7): Tài liệu & Chuẩn bị bảo vệ

- [ ] Viết báo cáo đồ án (phân công theo chương)
- [ ] Cập nhật README.md cho cả 2 repo
- [ ] Quay video demo (3–5 phút)
- [ ] Chuẩn bị slide thuyết trình
- [ ] Chuẩn bị câu hỏi & trả lời thường gặp

### Tuần 21–22 (cuối tháng 7 – tháng 8): Buffer Time

> **Thời gian dự phòng** — dùng cho:
> - Fix bug phát sinh sau deploy
> - Bổ sung tính năng bị thiếu
> - Rehearsal demo và thuyết trình
> - Submit hồ sơ đồ án

### Deliverables Phase 5
```
✅ Web app live trên Vercel (production URL)
✅ API live trên Railway (production URL)
✅ APK Android có thể cài đặt và demo
✅ Báo cáo đồ án hoàn chỉnh
✅ Slide thuyết trình hoàn chỉnh
✅ Video demo sẵn sàng
✅ README documenting đầy đủ cả 2 repo
```

---

## Tóm tắt milestone theo tháng

| Tháng | Milestone chính | Người chịu trách nhiệm |
|---|---|---|
| **Tháng 3** | Setup xong, Auth API hoạt động | Member 1 + Cả nhóm |
| **Tháng 4** | Core BE APIs hoàn chỉnh (Task, Group, Schedule, Forum, Notification) | Member 1 + 2 |
| **Tháng 5** | Web app core UI hoàn chỉnh, Mobile app chạy được | Member 3 + 4 |
| **Tháng 6** | Mobile hoàn chỉnh, Forum xong, AI tích hợp | Cả nhóm |
| **Tháng 7** | Polish, Deploy production, tài liệu | Cả nhóm |
| **Tháng 8** | Buffer, fix bug, bảo vệ đồ án | Cả nhóm |

---

## Rủi ro & Kế hoạch dự phòng

| Rủi ro | Khả năng | Xử lý |
|---|---|---|
| Member bị bệnh / bận học | Cao | Buffer time 2 tuần đã tính sẵn |
| OpenAI API tốn chi phí cao | Trung bình | Dùng GPT-4o-mini thay GPT-4o, giới hạn request/user |
| Scope creep (thêm tính năng ngoài kế hoạch) | Cao | Tuân thủ MoSCoW, chỉ add Could Have khi đã xong Should Have |
| Bug khó tìm trước bảo vệ | Trung bình | Phase testing tuần 18, không thêm tính năng mới sau tuần 17 |
| React Native build APK thất bại | Thấp | Dùng Expo Go để demo nếu EAS Build có vấn đề |
| MongoDB Atlas free tier giới hạn | Thấp | M0 có 512MB storage, đủ cho demo |
