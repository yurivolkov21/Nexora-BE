# Forum Module — API Endpoints

> **Owner:** Member 2
> **Base path:** `/api/v1/forum`
> **Status:** 🔄 Sẽ được cập nhật khi module được implement (Phase 1 — Tháng 4)

---

## Endpoints (Dự kiến)

| Method | Endpoint | Mô tả |
|---|---|---|
| `GET` | `/forum/posts` | Danh sách bài đăng (có pagination, filter, search) |
| `POST` | `/forum/posts` | Tạo bài đăng mới |
| `GET` | `/forum/posts/:id` | Chi tiết bài đăng + comments |
| `PATCH` | `/forum/posts/:id` | Cập nhật bài đăng (chỉ author) |
| `DELETE` | `/forum/posts/:id` | Xóa bài đăng |
| `POST` | `/forum/posts/:id/vote` | Upvote / Downvote |
| `POST` | `/forum/posts/:id/comments` | Thêm comment |
| `PATCH` | `/forum/comments/:id` | Sửa comment |
| `DELETE` | `/forum/comments/:id` | Xóa comment |
| `POST` | `/forum/comments/:id/vote` | Vote comment |

---

## Query Params dự kiến

```
GET /forum/posts?category=study&tag=nestjs&search=authentication&sortBy=votes&page=1&limit=20
```

| Param | Mô tả |
|---|---|
| `category` | `general` \| `study` \| `project` \| `qa` |
| `tag` | Filter theo tag |
| `search` | Full-text search |
| `sortBy` | `newest` \| `votes` \| `unanswered` |

---

## Schema dự kiến — Post

```typescript
{
  id: string;
  title: string;
  content: string;         // HTML từ Tiptap editor
  category: 'general' | 'study' | 'project' | 'qa';
  tags: string[];
  author: { id: string; displayName: string; avatar: string };
  upvotes: number;
  downvotes: number;
  commentCount: number;
  isPinned: boolean;
  isResolved: boolean;     // Cho loại Q&A
  createdAt: Date;
  updatedAt: Date;
}
```

---

> 📝 **TODO:** Member 2 cập nhật file này sau khi implement xong module.
