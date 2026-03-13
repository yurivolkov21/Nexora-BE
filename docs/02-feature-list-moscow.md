# Danh sách tính năng — Nexora (MoSCoW)

## Giải thích MoSCoW

| Nhãn | Ý nghĩa |
|---|---|
| **Must Have** | Bắt buộc phải có, không có thì sản phẩm không dùng được |
| **Should Have** | Quan trọng nhưng có thể delay sang sprint sau |
| **Could Have** | Tốt nếu có thời gian, không ảnh hưởng core |
| **Won't Have** | Không làm trong đợt này, để lại v2 |

---

## Module 1 — Authentication & User

### Must Have

- [ ] Đăng ký tài khoản bằng email + password
- [ ] Đăng nhập bằng email + password
- [ ] Đăng nhập bằng Google OAuth
- [ ] JWT Authentication (Access Token + Refresh Token)
- [ ] Đăng xuất (revoke Refresh Token)
- [ ] Phân quyền cơ bản: `admin` | `member`
- [ ] Xem và cập nhật hồ sơ cá nhân (avatar, display name, bio)

### Should Have

- [ ] Đổi mật khẩu
- [ ] Quên mật khẩu (gửi OTP qua email)
- [ ] Xác thực email sau khi đăng ký

### Could Have

- [ ] Đăng nhập bằng GitHub OAuth
- [ ] Bật / tắt 2FA (Two-Factor Authentication)
- [ ] Xem lịch sử hoạt động tài khoản

---

## Module 2 — Groups (Không gian nhóm)

### Must Have

- [ ] Tạo nhóm mới (tên, mô tả, ảnh đại diện nhóm)
- [ ] Mời thành viên qua email
- [ ] Role trong nhóm: `owner` | `admin` | `member`
- [ ] Xem danh sách thành viên nhóm
- [ ] Rời nhóm / Xóa thành viên (dành cho owner/admin)
- [ ] Xem danh sách nhóm của bản thân

### Should Have

- [ ] Mời thành viên qua invite link (có thời hạn)
- [ ] Chuyển quyền owner cho thành viên khác
- [ ] Nhóm có thể là `public` (ai cũng tìm được) hoặc `private`

### Could Have

- [ ] Tìm kiếm & tham gia nhóm public
- [ ] Group activity feed (log các hoạt động trong nhóm)
- [ ] Ghim thông báo quan trọng trong nhóm

---

## Module 3 — Task Management

### Must Have

- [ ] Tạo workspace (cá nhân hoặc gắn với nhóm)
- [ ] Kanban Board với 3 cột mặc định: `To Do` | `In Progress` | `Done`
- [ ] Thêm / đổi tên / xóa cột trên board
- [ ] Tạo, sửa, xóa Task
- [ ] Task bao gồm: tiêu đề, mô tả, deadline, priority (Low/Medium/High/Urgent)
- [ ] Gán task cho 1 hoặc nhiều thành viên
- [ ] Label / Tag màu sắc cho task
- [ ] Di chuyển task giữa các cột (drag & drop)
- [ ] Filter task theo: assignee, priority, label, deadline
- [ ] Tìm kiếm task theo tên

### Should Have

- [ ] Checklist con trong task (subtask)
- [ ] Đính kèm file / ảnh vào task
- [ ] Comment trong task
- [ ] Lịch sử thay đổi task (activity log)
- [ ] Task view: Board | List | Calendar
- [ ] Duplicate task
- [ ] Định kỳ lặp lại task (recurring)

### Could Have

- [ ] Export danh sách task ra CSV / Excel
- [ ] Gantt chart view
- [ ] Dependency giữa các task (task A phụ thuộc task B)
- [ ] Template board có sẵn

### Won't Have (v1)

- [ ] Time tracking tích hợp
- [ ] Billing / budget per project

---

## Module 4 — Study Schedule

### Must Have

- [ ] Xem lịch học dạng Calendar (tháng / tuần / ngày)
- [ ] Tạo, sửa, xóa sự kiện / lịch học
- [ ] Sự kiện bao gồm: tiêu đề, mô tả, thời gian bắt đầu/kết thúc, màu sắc
- [ ] Gắn deadline của task lên calendar tự động
- [ ] Pomodoro timer tích hợp (25/5 phút, tùy chỉnh được)
- [ ] Reminder thông báo trước sự kiện (15 phút, 1 giờ, 1 ngày)

### Should Have

- [ ] Theo dõi tiến độ học tập (Progress bar theo ngày/tuần)
- [ ] Thống kê thời gian học (chart theo tuần / tháng)
- [ ] Chia sẻ lịch học với thành viên nhóm
- [ ] Phân loại sự kiện theo môn học / dự án

### Could Have

- [ ] Đồng bộ với Google Calendar (import/export)
- [ ] Study streak (chuỗi ngày học liên tiếp)
- [ ] Đề xuất thời gian học dựa trên lịch trống (AI)

### Won't Have (v1)

- [ ] Tích hợp LMS (Moodle, Canvas)
- [ ] Video học tích hợp

---

## Module 5 — Forum / Discussion

### Must Have

- [ ] Tạo bài đăng (title + nội dung rich text)
- [ ] Comment và trả lời comment (threaded)
- [ ] Upvote / Downvote bài đăng
- [ ] Tag / Category bài đăng
- [ ] Tìm kiếm bài đăng theo từ khóa
- [ ] Phân loại: `General` | `Study` | `Project` | `Q&A`
- [ ] Ghim bài đăng (dành cho admin)

### Should Have

- [ ] Rich text editor (bold, italic, code block, image, link)
- [ ] Đánh dấu câu trả lời là "đã giải quyết" (cho loại Q&A)
- [ ] Báo cáo bài đăng / comment vi phạm
- [ ] Thông báo khi có người reply bài đăng của mình

### Could Have

- [ ] Lọc bài theo: mới nhất / hot nhất / chưa có trả lời
- [ ] Follow tag để nhận thông báo bài mới
- [ ] Bookmarks (lưu bài đăng yêu thích)

### Won't Have (v1)

- [ ] Hệ thống reputation / điểm uy tín
- [ ] Editor markdown nâng cao (LaTeX, diagram)

---

## Module 6 — AI Features

### Must Have

- [ ] AI Chatbot trong app (hỏi đáp chung, hỗ trợ sử dụng)
- [ ] AI gợi ý phân chia task khi tạo project mới (nhập mô tả → AI tạo task list)
- [ ] AI tóm tắt nội dung bài đăng dài trong Forum

### Should Have

- [ ] Smart deadline suggestion: AI gợi ý deadline hợp lý dựa trên workload hiện tại
- [ ] AI phân tích năng suất cá nhân và đưa ra gợi ý cải thiện
- [ ] Lưu lịch sử hội thoại AI (conversation history)

### Could Have

- [ ] AI tạo mô tả task tự động từ tiêu đề ngắn
- [ ] AI gợi ý tag/category cho bài đăng Forum
- [ ] AI tổng hợp báo cáo tuần (weekly summary)

### Won't Have (v1)

- [ ] Fine-tuned model riêng
- [ ] AI voice assistant
- [ ] Tích hợp AI vào video call

---

## Module 7 — Notifications

### Must Have

- [ ] Thông báo real-time trong app (WebSocket)
- [ ] Thông báo khi: task được assign, deadline sắp đến (24h trước), có người mention `@you`
- [ ] Trang xem tất cả thông báo (có phân trang)
- [ ] Đánh dấu đã đọc (từng thông báo hoặc tất cả)

### Should Have

- [ ] Push notification trên mobile (Firebase FCM)
- [ ] Email notification cho deadline quan trọng
- [ ] Cài đặt tùy chỉnh: bật/tắt từng loại thông báo
- [ ] Badge số lượng thông báo chưa đọc

### Could Have

- [ ] Digest email tóm tắt hoạt động theo ngày/tuần
- [ ] Quiet hours (không nhận thông báo trong khoảng giờ nhất định)

---

## Tổng quan MVP (Minimum Viable Product)

> MVP là phiên bản tối thiểu cần hoàn thiện trước khi demo & bảo vệ đồ án.

```
✅ Auth (Must Have hoàn chỉnh)
✅ Groups (Must Have)
✅ Task Management (Must Have + Should Have cơ bản)
✅ Schedule (Must Have)
✅ Forum (Must Have)
✅ AI Chatbot + Task suggestion (Must Have)
✅ Notification real-time (Must Have)
✅ Mobile app: Task + Schedule + Notification
```

**Tổng số tính năng Must Have:** ~45 items
**Tổng số màn hình ước tính:**
- Web: ~20–25 màn hình
- Mobile: ~12–15 màn hình
