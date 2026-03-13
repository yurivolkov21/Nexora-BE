# Tasks Module — API Endpoints

> Tài liệu tham chiếu API cho module Task Management.
> Dùng để FE và Mobile biết endpoint nào cần gọi.

**Base path:** `/api/v1/tasks`
**Owner:** Member 1
**Auth required:** Tất cả endpoints đều cần `Authorization: Bearer <token>`

---

## Endpoints

### GET `/tasks`

Lấy danh sách task (của cá nhân hoặc theo board).

**Query Params:**

| Param | Type | Required | Mô tả |
|---|---|---|---|
| `boardId` | string | No | Lọc theo board |
| `status` | string | No | `todo` \| `in_progress` \| `done` |
| `priority` | string | No | `low` \| `medium` \| `high` \| `urgent` |
| `assigneeId` | string | No | ID người được gán |
| `search` | string | No | Tìm theo tiêu đề |
| `page` | number | No | Default: 1 |
| `limit` | number | No | Default: 20 |

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Lấy danh sách task thành công",
  "data": {
    "items": [
      {
        "id": "65f1a2b3c4d5e6f7a8b9c0d1",
        "title": "Implement login API",
        "description": "Bao gồm email/password và Google OAuth",
        "status": "in_progress",
        "priority": "high",
        "labels": ["backend", "auth"],
        "deadline": "2026-03-15T00:00:00.000Z",
        "assignees": [
          { "id": "user_id", "displayName": "Member 1", "avatar": null }
        ],
        "boardId": "board_id",
        "order": 1,
        "createdAt": "2026-03-07T08:00:00.000Z",
        "updatedAt": "2026-03-07T09:00:00.000Z"
      }
    ],
    "total": 50,
    "page": 1,
    "limit": 20,
    "totalPages": 3
  }
}
```

---

### POST `/tasks`

Tạo task mới.

**Request Body:**

```json
{
  "title": "Implement login API",
  "description": "Bao gồm email/password và Google OAuth",
  "boardId": "65f1a2b3c4d5e6f7a8b9c0d1",
  "columnId": "65f1a2b3c4d5e6f7a8b9c0d2",
  "priority": "high",
  "deadline": "2026-03-15T00:00:00.000Z",
  "assigneeIds": ["user_id_1", "user_id_2"],
  "labels": ["backend", "auth"]
}
```

**Response `201`:**

```json
{
  "statusCode": 201,
  "message": "Tạo task thành công",
  "data": { /* Task object */ }
}
```

---

### GET `/tasks/:id`

Lấy chi tiết một task.

**Response `200`:**

```json
{
  "statusCode": 200,
  "data": {
    "id": "...",
    "title": "...",
    "checklist": [
      { "id": "c1", "text": "Viết DTO", "isDone": true },
      { "id": "c2", "text": "Viết service", "isDone": false }
    ],
    "comments": [
      {
        "id": "comment_id",
        "content": "Đã xong phần JWT",
        "author": { "id": "user_id", "displayName": "Member 1", "avatar": null },
        "createdAt": "2026-03-07T08:00:00.000Z"
      }
    ],
    "activityLog": [
      {
        "action": "status_changed",
        "from": "todo",
        "to": "in_progress",
        "by": { "id": "user_id", "displayName": "Member 1" },
        "at": "2026-03-07T09:00:00.000Z"
      }
    ]
  }
}
```

---

### PATCH `/tasks/:id`

Cập nhật task (partial update).

**Request Body** (tất cả field đều optional):

```json
{
  "title": "Implement login API v2",
  "status": "done",
  "priority": "medium",
  "deadline": "2026-03-20T00:00:00.000Z",
  "assigneeIds": ["user_id_1"]
}
```

**Response `200`:** Task object đã cập nhật.

---

### DELETE `/tasks/:id`

Xóa task (soft delete).

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Xóa task thành công",
  "data": null
}
```

---

### PATCH `/tasks/:id/move`

Di chuyển task sang cột khác (dùng cho drag & drop).

**Request Body:**

```json
{
  "columnId": "new_column_id",
  "order": 2
}
```

**Response `200`:** Task object đã cập nhật vị trí.

---

### POST `/tasks/:id/comments`

Thêm comment vào task.

**Request Body:**

```json
{
  "content": "Đã hoàn thành phần auth middleware"
}
```

**Response `201`:** Comment object mới.

---

### PATCH `/tasks/:id/checklist`

Cập nhật checklist của task.

**Request Body:**

```json
{
  "checklist": [
    { "id": "c1", "text": "Viết DTO", "isDone": true },
    { "id": "c2", "text": "Viết service", "isDone": true },
    { "text": "Viết test", "isDone": false }
  ]
}
```

---

## Board Endpoints

### GET `/boards`

Lấy danh sách board của user (cá nhân và nhóm).

**Response `200`:**

```json
{
  "data": {
    "items": [
      {
        "id": "board_id",
        "name": "Sprint 1",
        "type": "personal",
        "columns": [
          { "id": "col_1", "name": "To Do", "order": 0 },
          { "id": "col_2", "name": "In Progress", "order": 1 },
          { "id": "col_3", "name": "Done", "order": 2 }
        ]
      }
    ]
  }
}
```

### POST `/boards`

Tạo board mới.

```json
{
  "name": "Sprint 1",
  "groupId": "optional_group_id"
}
```

### PATCH `/boards/:id/columns`

Thêm/đổi tên/xóa/sắp xếp cột trên board.

```json
{
  "columns": [
    { "id": "col_1", "name": "Backlog", "order": 0 },
    { "id": "col_2", "name": "In Progress", "order": 1 },
    { "name": "Testing", "order": 2 },
    { "id": "col_3", "name": "Done", "order": 3 }
  ]
}
```

---

## MongoDB Schema — Task

```typescript
@Schema({ timestamps: true, versionKey: false })
export class Task {
  @Prop({ required: true, trim: true })
  title: string;

  @Prop({ default: null })
  description: string;

  @Prop({ type: Types.ObjectId, ref: 'Board', required: true })
  boardId: Types.ObjectId;

  @Prop({ type: Types.ObjectId, required: true })
  columnId: Types.ObjectId;

  @Prop({ enum: ['todo', 'in_progress', 'done'], default: 'todo' })
  status: string;

  @Prop({ enum: ['low', 'medium', 'high', 'urgent'], default: 'medium' })
  priority: string;

  @Prop({ type: [{ type: Types.ObjectId, ref: 'User' }], default: [] })
  assignees: Types.ObjectId[];

  @Prop({ type: [String], default: [] })
  labels: string[];

  @Prop({ default: null })
  deadline: Date;

  @Prop({ default: 0 })
  order: number;

  @Prop({
    type: [{ id: String, text: String, isDone: Boolean }],
    default: [],
  })
  checklist: { id: string; text: string; isDone: boolean }[];

  @Prop({ type: Types.ObjectId, ref: 'User', required: true })
  createdBy: Types.ObjectId;

  @Prop({ default: null })
  deletedAt: Date;
}
```
