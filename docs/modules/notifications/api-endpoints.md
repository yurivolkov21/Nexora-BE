# Notifications Module — API Endpoints

> **Owner:** Member 2
> **Base path:** `/api/v1/notifications`
> **Status:** 🔄 Sẽ được cập nhật khi module được implement (Phase 1 — Tháng 4)

---

## REST Endpoints (Dự kiến)

| Method | Endpoint | Mô tả |
|---|---|---|
| `GET` | `/notifications` | Danh sách thông báo của user (có pagination) |
| `PATCH` | `/notifications/:id/read` | Đánh dấu đã đọc một thông báo |
| `PATCH` | `/notifications/read-all` | Đánh dấu tất cả đã đọc |
| `DELETE` | `/notifications/:id` | Xóa thông báo |
| `GET` | `/notifications/unread-count` | Số thông báo chưa đọc (dùng cho badge) |

---

## WebSocket Events

### Client lắng nghe (Server → Client)

| Event | Khi nào trigger | Payload |
|---|---|---|
| `notification:new` | Có thông báo mới | `{ id, type, title, body, data }` |
| `notification:badge` | Số unread thay đổi | `{ count: number }` |

### Client gửi (Client → Server)

| Event | Mô tả |
|---|---|
| `join` | User kết nối, join vào room của mình |

---

## Notification Types

```typescript
type NotificationType =
  | 'task_assigned'       // Được gán task
  | 'task_deadline'       // Task sắp đến deadline (24h)
  | 'task_status_changed' // Trạng thái task thay đổi
  | 'mention'             // Được mention @username
  | 'comment_reply'       // Có reply vào comment của mình
  | 'group_invite'        // Được mời vào nhóm
  | 'post_reply';         // Có reply vào bài đăng Forum
```

---

## Schema dự kiến — Notification

```typescript
{
  id: string;
  userId: string;           // Người nhận
  type: NotificationType;
  title: string;
  body: string;
  data: {                   // Dữ liệu để điều hướng khi click
    entityType: 'task' | 'post' | 'group';
    entityId: string;
  };
  isRead: boolean;
  createdAt: Date;
}
```

---

## Tích hợp phía FE/Mobile

### Web — Socket.io

```typescript
// Kết nối WebSocket sau khi đăng nhập
const socket = io(SOCKET_URL, {
  auth: { token: accessToken },
});

socket.on('notification:new', (notification) => {
  // Hiển thị toast + tăng badge count
});
```

### Mobile — Push Notification

```typescript
// Đăng ký FCM token sau khi đăng nhập
const token = await Notifications.getExpoPushTokenAsync();
await api.post('/users/fcm-token', { token: token.data });
```

---

> 📝 **TODO:** Member 2 cập nhật file này sau khi implement xong module.
