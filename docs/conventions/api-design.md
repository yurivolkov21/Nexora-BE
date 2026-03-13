# API Design Conventions — Nexora

> Quy ước thiết kế REST API cho BE team. FE và Mobile team đọc để hiểu cách tích hợp.

---

## Base URL

```
Development : http://localhost:3000/api/v1
Production  : https://api.nexora.app/api/v1
```

---

## URL Structure

### Format

```
/api/v1/[resource]/[id]/[sub-resource]
```

### Quy tắc đặt tên

- Dùng **kebab-case**, **số nhiều** cho resource
- Không dùng động từ trong URL (động từ là HTTP method)

```
✅ GET  /api/v1/tasks
✅ GET  /api/v1/tasks/:id
✅ GET  /api/v1/tasks/:id/comments
✅ POST /api/v1/groups/:id/members

❌ GET  /api/v1/getTask
❌ POST /api/v1/createTask
❌ GET  /api/v1/task          # Số ít
❌ GET  /api/v1/Tasks         # PascalCase
```

---

## HTTP Methods

| Method | Mục đích | Ví dụ |
|---|---|---|
| `GET` | Lấy dữ liệu | `GET /tasks` |
| `POST` | Tạo mới | `POST /tasks` |
| `PATCH` | Cập nhật một phần | `PATCH /tasks/:id` |
| `PUT` | Thay thế hoàn toàn | Hạn chế dùng |
| `DELETE` | Xóa | `DELETE /tasks/:id` |

> Dùng `PATCH` thay vì `PUT` cho update vì thường chỉ cập nhật một phần field.

---

## Response Format

### Success Response

```json
{
  "statusCode": 200,
  "message": "Lấy danh sách task thành công",
  "data": { ... }
}
```

### Success với Pagination

```json
{
  "statusCode": 200,
  "message": "Lấy danh sách task thành công",
  "data": {
    "items": [ ... ],
    "total": 100,
    "page": 1,
    "limit": 20,
    "totalPages": 5
  }
}
```

### Error Response

```json
{
  "statusCode": 400,
  "message": "Dữ liệu không hợp lệ",
  "errors": [
    {
      "field": "email",
      "message": "Email không đúng định dạng"
    }
  ]
}
```

---

## HTTP Status Codes

| Code | Ý nghĩa | Khi nào dùng |
|---|---|---|
| `200` | OK | GET, PATCH, DELETE thành công |
| `201` | Created | POST tạo mới thành công |
| `204` | No Content | DELETE không trả về data |
| `400` | Bad Request | Validation lỗi, thiếu field bắt buộc |
| `401` | Unauthorized | Chưa đăng nhập, token hết hạn |
| `403` | Forbidden | Đã đăng nhập nhưng không có quyền |
| `404` | Not Found | Resource không tồn tại |
| `409` | Conflict | Trùng email khi đăng ký, duplicate data |
| `422` | Unprocessable Entity | Dữ liệu hợp lệ nhưng logic không thực hiện được |
| `500` | Internal Server Error | Lỗi server không mong muốn |

---

## Pagination

Dùng **query parameters** cho pagination:

```
GET /api/v1/tasks?page=1&limit=20&sortBy=createdAt&order=desc
```

| Param | Default | Mô tả |
|---|---|---|
| `page` | `1` | Trang hiện tại |
| `limit` | `20` | Số item mỗi trang (max: 100) |
| `sortBy` | `createdAt` | Field để sort |
| `order` | `desc` | `asc` hoặc `desc` |

---

## Filtering & Search

```
GET /api/v1/tasks?status=in_progress&priority=high&assigneeId=123
GET /api/v1/posts?search=nestjs&category=study&tag=backend
```

---

## Authentication

Tất cả endpoints cần auth phải có header:

```
Authorization: Bearer <access_token>
```

Refresh token được lưu trong **HTTP-only cookie**, tự động gửi qua `withCredentials: true`.

```
POST /api/v1/auth/refresh
Cookie: refresh_token=<refresh_token>
```

---

## ID Format

Dùng MongoDB ObjectId (`_id`). Khi trả về trong response, dùng tên field `id` (không phải `_id`) để thống nhất:

```json
{
  "id": "65f1a2b3c4d5e6f7a8b9c0d1",
  "title": "Task title"
}
```

> Trong NestJS, dùng `@Transform` trong DTO hoặc `toJSON()` trong Mongoose Schema để chuyển `_id` → `id`.

---

## Timestamps

Tất cả resource đều có:

```json
{
  "id": "...",
  "createdAt": "2026-03-07T08:00:00.000Z",
  "updatedAt": "2026-03-07T09:30:00.000Z"
}
```

Dùng format **ISO 8601 UTC** (`YYYY-MM-DDTHH:mm:ss.sssZ`).

---

## Soft Delete

Thay vì xóa cứng, các resource quan trọng (users, tasks, posts) dùng soft delete:

```json
{
  "deletedAt": "2026-03-07T10:00:00.000Z"
}
```

Khi query, mặc định filter `deletedAt: null`. Chỉ admin mới có thể xem các record đã xóa.

---

## Versioning

API version nằm trong URL path: `/api/v1/...`

Khi có breaking change trong tương lai → tạo `/api/v2/...` song song, không xóa v1 ngay.

---

## Swagger Documentation

Mỗi endpoint BE phải có đầy đủ decorator:

```typescript
@ApiTags('tasks')
@ApiBearerAuth()
@Controller('tasks')
export class TasksController {

  @Get()
  @ApiOperation({ summary: 'Lấy danh sách task' })
  @ApiQuery({ name: 'page', required: false, type: Number })
  @ApiQuery({ name: 'limit', required: false, type: Number })
  @ApiResponse({ status: 200, description: 'Thành công', type: TaskListResponseDto })
  @ApiResponse({ status: 401, description: 'Chưa xác thực' })
  findAll() { ... }
}
```

> Xem Swagger UI tại `http://localhost:3000/api` khi app đang chạy.
