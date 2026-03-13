# Groups Module — API Endpoints

> Tài liệu tham chiếu API cho module Groups (Không gian nhóm).
> Dùng để FE và Mobile biết endpoint nào cần gọi mà không cần hỏi BE.

**Base path:** `/api/v1/groups`
**Owner:** Member 1
**Auth required:** Tất cả endpoints đều cần `Authorization: Bearer <token>`

---

## Endpoints

### GET `/groups`

Lấy danh sách nhóm mà user hiện tại đang tham gia.

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Lấy danh sách nhóm thành công",
  "data": {
    "items": [
      {
        "id": "65f1a2b3c4d5e6f7a8b9c0d1",
        "name": "Nhóm Đồ Án",
        "description": "Nhóm làm đồ án tốt nghiệp",
        "avatar": "https://cloudinary.com/...",
        "type": "private",
        "memberCount": 4,
        "myRole": "owner",
        "createdAt": "2026-03-07T08:00:00.000Z"
      }
    ],
    "total": 3
  }
}
```

---

### POST `/groups`

Tạo nhóm mới. Người tạo tự động trở thành `owner`.

**Request Body:**

```json
{
  "name": "Nhóm Đồ Án",
  "description": "Nhóm làm đồ án tốt nghiệp",
  "type": "private"
}
```

**Response `201`:**

```json
{
  "statusCode": 201,
  "message": "Tạo nhóm thành công",
  "data": {
    "id": "65f1a2b3c4d5e6f7a8b9c0d1",
    "name": "Nhóm Đồ Án",
    "description": "Nhóm làm đồ án tốt nghiệp",
    "avatar": null,
    "type": "private",
    "inviteCode": "NEXORA-ABC123",
    "members": [
      {
        "userId": "user_id",
        "displayName": "Nguyen Van A",
        "avatar": null,
        "role": "owner",
        "joinedAt": "2026-03-07T08:00:00.000Z"
      }
    ],
    "createdAt": "2026-03-07T08:00:00.000Z"
  }
}
```

---

### GET `/groups/:id`

Lấy thông tin chi tiết một nhóm.

**Response `200`:**

```json
{
  "statusCode": 200,
  "data": {
    "id": "...",
    "name": "Nhóm Đồ Án",
    "description": "...",
    "avatar": null,
    "type": "private",
    "inviteCode": "NEXORA-ABC123",
    "members": [
      {
        "userId": "user_id_1",
        "displayName": "Member 1",
        "avatar": null,
        "role": "owner",
        "joinedAt": "2026-03-07T08:00:00.000Z"
      },
      {
        "userId": "user_id_2",
        "displayName": "Member 2",
        "avatar": null,
        "role": "member",
        "joinedAt": "2026-03-08T08:00:00.000Z"
      }
    ],
    "boards": [
      { "id": "board_id", "name": "Sprint 1" }
    ],
    "createdAt": "2026-03-07T08:00:00.000Z"
  }
}
```

**Errors:**
- `403` — Không phải thành viên của nhóm
- `404` — Nhóm không tồn tại

---

### PATCH `/groups/:id`

Cập nhật thông tin nhóm. Chỉ `owner` hoặc `admin` mới có quyền.

**Request Body** (tất cả optional):

```json
{
  "name": "Nhóm Đồ Án CNTT",
  "description": "Mô tả mới",
  "type": "public"
}
```

---

### DELETE `/groups/:id`

Xóa nhóm. Chỉ `owner` mới có quyền.

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Nhóm đã được xóa",
  "data": null
}
```

---

### POST `/groups/:id/invite`

Mời thành viên qua email.

**Request Body:**

```json
{
  "emails": ["member2@example.com", "member3@example.com"]
}
```

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Đã gửi lời mời đến 2 email",
  "data": {
    "sent": ["member2@example.com", "member3@example.com"],
    "failed": []
  }
}
```

---

### GET `/groups/:id/invite-link`

Lấy hoặc tạo mới invite link cho nhóm. Chỉ `owner` / `admin`.

**Response `200`:**

```json
{
  "statusCode": 200,
  "data": {
    "inviteLink": "https://nexora.app/join/NEXORA-ABC123",
    "inviteCode": "NEXORA-ABC123",
    "expiresAt": "2026-03-14T08:00:00.000Z"
  }
}
```

---

### POST `/groups/join/:inviteCode`

Tham gia nhóm qua invite code.

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Tham gia nhóm thành công",
  "data": { /* Group object */ }
}
```

**Errors:**
- `400` — Invite code không hợp lệ hoặc đã hết hạn
- `409` — Đã là thành viên của nhóm

---

### PATCH `/groups/:id/members/:userId/role`

Thay đổi role của thành viên. Chỉ `owner` / `admin`.

**Request Body:**

```json
{
  "role": "admin"
}
```

> `role` chấp nhận: `admin` | `member`. Không thể đổi role của `owner` qua endpoint này.

---

### DELETE `/groups/:id/members/:userId`

Xóa thành viên khỏi nhóm. `Owner`/`admin` xóa người khác, hoặc user tự rời nhóm.

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Đã xóa thành viên khỏi nhóm",
  "data": null
}
```

**Errors:**
- `403` — Không có quyền xóa thành viên này
- `422` — Owner không thể tự rời nhóm, phải chuyển quyền trước

---

### PATCH `/groups/:id/transfer-ownership`

Chuyển quyền owner. Chỉ `owner` hiện tại.

**Request Body:**

```json
{
  "newOwnerId": "user_id_of_new_owner"
}
```

---

## MongoDB Schema — Group

```typescript
@Schema({ timestamps: true, versionKey: false })
export class Group {
  @Prop({ required: true, trim: true })
  name: string;

  @Prop({ default: null })
  description: string;

  @Prop({ default: null })
  avatar: string;

  @Prop({ enum: ['public', 'private'], default: 'private' })
  type: string;

  @Prop({ unique: true })
  inviteCode: string;       // VD: "NEXORA-ABC123"

  @Prop({ default: null })
  inviteCodeExpiresAt: Date;

  @Prop({
    type: [{
      userId: { type: Types.ObjectId, ref: 'User' },
      role: { type: String, enum: ['owner', 'admin', 'member'], default: 'member' },
      joinedAt: { type: Date, default: Date.now },
    }],
    default: [],
  })
  members: { userId: Types.ObjectId; role: string; joinedAt: Date }[];

  @Prop({ default: null })
  deletedAt: Date;
}
```

---

## Lưu ý tích hợp

- Khi tạo board, cần có `groupId` (optional) để gắn board vào nhóm — xem `modules/tasks/api-endpoints.md`
- Thông báo mời nhóm dùng `notification type: 'group_invite'` — xem `modules/notifications/api-endpoints.md`
