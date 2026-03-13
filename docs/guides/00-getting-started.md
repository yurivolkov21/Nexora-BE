# Getting Started — Nexora

> Tài liệu này dành cho **tất cả thành viên** đọc trước khi bắt đầu. Hoàn thành các bước dưới đây trước khi chuyển sang guide riêng của từng role.

---

## 1. Yêu cầu môi trường

Kiểm tra các công cụ sau đã được cài đặt:

```bash
node --version     # Cần >= 20.x (LTS)
npm --version      # Đi kèm với Node.js
git --version      # Cần >= 2.x
```

### Cài đặt công cụ bổ sung

```bash
# pnpm — package manager nhanh hơn npm (dùng thống nhất toàn project)
npm install -g pnpm

# NestJS CLI (Member 1 & 2 cần)
npm install -g @nestjs/cli

# Expo CLI (Member 4 cần)
npm install -g expo-cli

# Kiểm tra lại
pnpm --version
nest --version
```

### Công cụ khuyên dùng

| Công cụ | Mục đích | Link |
|---|---|---|
| VS Code | Editor chính | https://code.visualstudio.com |
| MongoDB Compass | GUI quản lý MongoDB | https://www.mongodb.com/products/compass |
| Postman | Test API | https://www.postman.com |
| TablePlus (optional) | GUI cho Redis | https://tableplus.com |

### VS Code Extensions khuyên dùng

```
ESLint
Prettier - Code formatter
TypeScript Importer
Tailwind CSS IntelliSense
Prisma (nếu dùng sau này)
GitLens
Thunder Client (test API ngay trong VS Code)
```

---

## 2. Clone repos

```bash
# Backend
git clone https://github.com/[org]/nexora-be.git
cd nexora-be
pnpm install

# Frontend Web
git clone https://github.com/[org]/nexora-fe.git
cd nexora-fe
pnpm install

# Mobile
git clone https://github.com/[org]/nexora-mobile.git
cd nexora-mobile
pnpm install
```

> Thay `[org]` bằng GitHub organization hoặc username của nhóm.

---

## 3. Setup môi trường (`.env`)

Mỗi repo có file `.env.example`. Tạo file `.env` từ template:

```bash
# Chạy lệnh này trong thư mục của từng repo
cp .env.example .env
```

Sau đó điền các giá trị vào `.env`. Hỏi Member 1 (Leader) để lấy các thông tin sau:
- `MONGODB_URI` — connection string MongoDB Atlas
- `JWT_SECRET` — secret key cho JWT
- `OPENAI_API_KEY` — key OpenAI (dùng chung)
- `CLOUDINARY_*` — credentials Cloudinary

> ⚠️ **Không commit file `.env` lên Git.** File này đã có trong `.gitignore`.

---

## 4. Setup MongoDB Atlas (local dev)

> Nếu muốn dùng MongoDB local thay vì Atlas:

```bash
# Cài MongoDB Community Server
# https://www.mongodb.com/try/download/community

# Hoặc dùng Docker (khuyến nghị)
docker run -d -p 27017:27017 --name mongodb mongo:latest
```

Connection string local: `mongodb://localhost:27017/nexora`

---

## 5. Quy ước Git — Bắt buộc tuân thủ

### Branch naming

```
main                        # Production, không push trực tiếp
develop                     # Branch tích hợp chính
feature/[tên-tính-năng]     # Tính năng mới
fix/[mô-tả-bug]             # Sửa bug
chore/[mô-tả]               # Cấu hình, deps, refactor nhỏ
```

Ví dụ:
```
feature/auth-login
feature/kanban-drag-and-drop
fix/jwt-refresh-token
chore/setup-swagger
```

### Commit message — Conventional Commits

```
feat: thêm tính năng đăng nhập Google OAuth
fix: sửa lỗi refresh token hết hạn sớm
chore: cập nhật dependencies NestJS
docs: thêm hướng dẫn setup BE
refactor: tách notification service thành worker riêng
test: thêm unit test cho auth service
```

### Quy trình làm việc

```
1. Checkout từ develop
   git checkout develop && git pull
   git checkout -b feature/tên-tính-năng

2. Code + commit thường xuyên (không để 1 commit quá lớn)

3. Push branch và tạo Pull Request → develop
   git push origin feature/tên-tính-năng

4. Yêu cầu ít nhất 1 người review trước khi merge

5. Sau khi merge → xóa branch đã dùng
```

> Xem chi tiết hơn tại: [`conventions/git-workflow.md`](../conventions/git-workflow.md)

---

## 6. Hướng dẫn tiếp theo theo role

Sau khi hoàn thành các bước trên, chuyển sang guide riêng:

| Role | File tiếp theo |
|---|---|
| Member 1 & 2 — Backend | [`guides/backend/01-setup.md`](./backend/01-setup.md) |
| Member 3 — Frontend Web | [`guides/frontend/01-setup.md`](./frontend/01-setup.md) |
| Member 4 — Mobile | [`guides/mobile/01-setup.md`](./mobile/01-setup.md) |

---

## 7. Kênh liên lạc & công cụ nhóm

| Mục đích | Công cụ |
|---|---|
| Chat hàng ngày | Discord / Zalo nhóm |
| Quản lý task | GitHub Projects |
| Tài liệu chung | Thư mục `docs/` trong repo |
| API testing | Swagger UI tại `http://localhost:3000/api` |
| Postman collection | Hỏi Member 1 để lấy file export |

### Lịch sync nhóm

- **Thứ 2:** Planning — phân công task tuần
- **Thứ 6:** Review — demo tiến độ, báo blocker
- **Hàng ngày:** Update ngắn trên Discord (đang làm gì, bị block không)
