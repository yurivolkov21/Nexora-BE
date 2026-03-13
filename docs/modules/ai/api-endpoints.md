# AI Module — API Endpoints & Prompts

> **Owner:** Member 1
> **Base path:** `/api/v1/ai`
> **Status:** 🔄 Sẽ được implement ở Phase 4 (Tháng 6)

---

## Endpoints (Dự kiến)

| Method | Endpoint | Mô tả |
|---|---|---|
| `POST` | `/ai/chat` | Gửi tin nhắn đến AI Chatbot (có streaming) |
| `GET` | `/ai/chat/history` | Lịch sử hội thoại |
| `DELETE` | `/ai/chat/history` | Xóa lịch sử hội thoại |
| `POST` | `/ai/suggest-tasks` | AI gợi ý danh sách task từ mô tả project |
| `POST` | `/ai/summarize-post` | AI tóm tắt bài đăng Forum |
| `POST` | `/ai/suggest-deadline` | AI gợi ý deadline dựa trên workload hiện tại |

---

## POST `/ai/chat` — Streaming

```typescript
// Request
{
  "message": "Giải thích JWT refresh token hoạt động như thế nào?"
}

// Response — Server-Sent Events (SSE) stream
// Content-Type: text/event-stream
data: {"delta": "JWT "}
data: {"delta": "refresh "}
data: {"delta": "token "}
data: {"delta": "là..."}
data: [DONE]
```

**FE — Nhận streaming response:**

```typescript
const response = await fetch(`${API_URL}/ai/chat`, {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${accessToken}`,
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({ message }),
});

const reader = response.body?.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader!.read();
  if (done) break;
  const chunk = decoder.decode(value);
  // Append chunk vào UI
}
```

---

## POST `/ai/suggest-tasks`

```typescript
// Request
{
  "projectDescription": "Xây dựng hệ thống quản lý task cho nhóm sinh viên, bao gồm kanban board, real-time notification và tích hợp AI",
  "deadline": "2026-08-31T00:00:00.000Z",
  "memberCount": 4
}

// Response
{
  "data": {
    "tasks": [
      {
        "title": "Setup NestJS project và cấu hình cơ bản",
        "description": "Khởi tạo project, setup Docker, MongoDB, Swagger",
        "estimatedDays": 3,
        "priority": "high",
        "suggestedAssignee": "backend"
      },
      // ...
    ]
  }
}
```

---

## Prompt Templates

> Lưu các prompt đã được tối ưu để tránh mỗi lần phải viết lại.

### System Prompt — AI Chatbot

```
Bạn là trợ lý AI của Nexora, một nền tảng cộng tác dành cho sinh viên.
Nhiệm vụ của bạn là hỗ trợ người dùng:
- Giải đáp các câu hỏi về lập trình (JavaScript, TypeScript, React, NestJS...)
- Hỗ trợ quản lý task và lên kế hoạch học tập
- Tóm tắt tài liệu và bài đăng

Hãy trả lời ngắn gọn, rõ ràng bằng tiếng Việt. 
Nếu câu hỏi liên quan đến code, hãy cung cấp ví dụ cụ thể.
```

### Prompt — Suggest Tasks

```
Dựa trên mô tả dự án sau, hãy tạo danh sách task cụ thể và có thể thực hiện được:

Mô tả: {projectDescription}
Deadline: {deadline}
Số thành viên: {memberCount}

Yêu cầu:
- Mỗi task phải có tiêu đề rõ ràng (không quá 10 từ)
- Có mô tả ngắn gọn
- Ước tính số ngày thực hiện
- Mức độ ưu tiên (low/medium/high/urgent)
- Vai trò phù hợp (backend/frontend/mobile/design)

Trả về JSON array với format đã cho.
```

### Prompt — Summarize Post

```
Tóm tắt bài đăng sau thành tối đa 3 câu ngắn gọn bằng tiếng Việt:

{postContent}
```

---

## Model được dùng

| Feature | Model | Lý do |
|---|---|---|
| Chatbot | `gpt-4o-mini` | Đủ thông minh, chi phí thấp |
| Suggest tasks | `gpt-4o` | Cần reasoning tốt hơn |
| Summarize | `gpt-4o-mini` | Task đơn giản, tiết kiệm token |
| Suggest deadline | `gpt-4o-mini` | Phân tích workload cơ bản |

> Giới hạn rate: Mỗi user tối đa 20 AI requests / giờ để kiểm soát chi phí API.

---

> 📝 **TODO:** Member 1 cập nhật file này sau khi implement xong AI module.
