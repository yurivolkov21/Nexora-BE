# Module: Users — API Endpoints

> **Owner:** Member 2  
> **Base path:** `/api/v1/users`  
> **Requires auth:** All endpoints require `Authorization: Bearer <accessToken>` unless stated otherwise.

---

## Endpoints Overview

| Method | Path | Description | Auth |
|--------|------|-------------|------|
| GET | `/users/me` | Get current user profile | Required |
| PATCH | `/users/me` | Update profile info | Required |
| POST | `/users/me/avatar` | Upload/change avatar | Required |
| PATCH | `/users/me/password` | Change password | Required |
| GET | `/users/me/stats` | Get personal stats | Required |
| GET | `/users/:id` | View another user's public profile | Required |

---

## 1. GET `/users/me` — Get Current User Profile

Returns the full profile of the currently logged-in user.

**Request:**
```http
GET /api/v1/users/me
Authorization: Bearer <accessToken>
```

**Response `200`:**
```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {
    "_id": "64f1a2b3c4d5e6f7a8b9c0d1",
    "email": "user@example.com",
    "displayName": "Nguyen Van A",
    "avatarUrl": "https://res.cloudinary.com/nexora/image/upload/v1/avatars/user_id.jpg",
    "bio": "Hello, I am a student at ...",
    "provider": "local",
    "isVerified": true,
    "createdAt": "2025-01-15T08:00:00.000Z",
    "updatedAt": "2025-03-01T10:30:00.000Z"
  }
}
```

---

## 2. PATCH `/users/me` — Update Profile

Update `displayName` and/or `bio`. Avatar is handled by a separate endpoint.

**Request:**
```http
PATCH /api/v1/users/me
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "displayName": "Nguyen Van A (updated)",
  "bio": "Full-stack developer, coffee lover ☕"
}
```

**Validation rules:**
- `displayName`: string, min 2, max 50 chars (optional)
- `bio`: string, max 200 chars (optional)

**Response `200`:**
```json
{
  "statusCode": 200,
  "message": "Profile updated successfully",
  "data": {
    "_id": "64f1a2b3c4d5e6f7a8b9c0d1",
    "email": "user@example.com",
    "displayName": "Nguyen Van A (updated)",
    "bio": "Full-stack developer, coffee lover ☕",
    "avatarUrl": "https://res.cloudinary.com/nexora/image/upload/...",
    "updatedAt": "2025-03-10T12:00:00.000Z"
  }
}
```

---

## 3. POST `/users/me/avatar` — Upload Avatar

Upload a new profile picture via `multipart/form-data`. File is stored on **Cloudinary**; old avatar is deleted after upload.

**Request:**
```http
POST /api/v1/users/me/avatar
Authorization: Bearer <accessToken>
Content-Type: multipart/form-data

Body:
  file: <image file>
```

**Constraints:**
- Accepted MIME types: `image/jpeg`, `image/png`, `image/webp`
- Max file size: **2 MB**
- Recommended dimensions: at least 200×200 px

**Response `200`:**
```json
{
  "statusCode": 200,
  "message": "Avatar updated successfully",
  "data": {
    "avatarUrl": "https://res.cloudinary.com/nexora/image/upload/v1/avatars/64f1a2b3.jpg"
  }
}
```

**Response `400` — Invalid file:**
```json
{
  "statusCode": 400,
  "message": "File must be an image (jpeg, png, webp) and under 2MB"
}
```

> **NestJS Note:** Use `@UseInterceptors(FileInterceptor('file'))` + `multer` + `cloudinary` SDK. Delete old `avatarUrl` from Cloudinary before uploading new one.

---

## 4. PATCH `/users/me/password` — Change Password

Requires the user to confirm their current password before setting a new one. Not available for OAuth users (`provider !== 'local'`).

**Request:**
```http
PATCH /api/v1/users/me/password
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "currentPassword": "OldPass@123",
  "newPassword": "NewSecurePass#456",
  "confirmNewPassword": "NewSecurePass#456"
}
```

**Validation rules:**
- `currentPassword`: required
- `newPassword`: min 8 chars, at least 1 uppercase, 1 number, 1 special char
- `confirmNewPassword`: must match `newPassword`

**Response `200`:**
```json
{
  "statusCode": 200,
  "message": "Password changed successfully"
}
```

**Response `400` — Wrong current password:**
```json
{
  "statusCode": 400,
  "message": "Current password is incorrect"
}
```

**Response `400` — OAuth user:**
```json
{
  "statusCode": 400,
  "message": "Cannot change password for accounts linked with Google"
}
```

---

## 5. GET `/users/me/stats` — Personal Stats

Returns aggregated stats for the current user's activity. Used in profile page and mobile dashboard.

**Request:**
```http
GET /api/v1/users/me/stats
Authorization: Bearer <accessToken>
```

**Response `200`:**
```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {
    "tasksCompleted": 42,
    "tasksInProgress": 5,
    "studyHoursThisWeek": 12.5,
    "studyStreakDays": 7,
    "forumPostsCount": 15,
    "groupsJoined": 3,
    "aiRequestsThisMonth": 18
  }
}
```

> **Implementation Note:** Stats are computed via MongoDB aggregation pipelines across `tasks`, `scheduleblocks`, and `forumposts` collections. Cache result in Redis with TTL 5 minutes to avoid heavy repeated queries.

---

## 6. GET `/users/:id` — View Public Profile

View another user's public information (used for profile pages in groups or forum author cards).

**Request:**
```http
GET /api/v1/users/64f1a2b3c4d5e6f7a8b9c0d2
Authorization: Bearer <accessToken>
```

**Response `200`:**
```json
{
  "statusCode": 200,
  "message": "Success",
  "data": {
    "_id": "64f1a2b3c4d5e6f7a8b9c0d2",
    "displayName": "Tran Thi B",
    "avatarUrl": "https://res.cloudinary.com/nexora/image/upload/...",
    "bio": "Design thinking enthusiast",
    "createdAt": "2025-02-01T00:00:00.000Z"
  }
}
```

> **Note:** This endpoint does NOT expose email, password hash, or refresh token. Only public fields are returned.

**Response `404`:**
```json
{
  "statusCode": 404,
  "message": "User not found"
}
```

---

## MongoDB User Schema

> Full schema is defined in `modules/auth/api-endpoints.md`. Summary of fields relevant to this module:

```typescript
// Relevant fields managed by Users module
{
  displayName: string;      // Editable via PATCH /users/me
  avatarUrl: string;        // Updated via POST /users/me/avatar (Cloudinary URL)
  bio: string;              // Editable via PATCH /users/me
  password: string;         // Hashed — changed via PATCH /users/me/password
  provider: 'local' | 'google'; // If 'google', password change is blocked
}
```

See [`modules/auth/api-endpoints.md`](../auth/api-endpoints.md) for the complete User schema with all fields.

---

## DTO Examples (NestJS)

```typescript
// update-profile.dto.ts
import { IsOptional, IsString, MinLength, MaxLength } from 'class-validator';

export class UpdateProfileDto {
  @IsOptional()
  @IsString()
  @MinLength(2)
  @MaxLength(50)
  displayName?: string;

  @IsOptional()
  @IsString()
  @MaxLength(200)
  bio?: string;
}
```

```typescript
// change-password.dto.ts
import { IsString, MinLength, Matches } from 'class-validator';

export class ChangePasswordDto {
  @IsString()
  currentPassword: string;

  @IsString()
  @MinLength(8)
  @Matches(/^(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&#])/, {
    message: 'Password must contain at least 1 uppercase, 1 number, 1 special character',
  })
  newPassword: string;

  @IsString()
  confirmNewPassword: string;
}
```

---

## Notes for Frontend (Member 3)

- **Avatar upload**: Use `FormData` + `axios.post('/users/me/avatar', formData, { headers: { 'Content-Type': 'multipart/form-data' } })`. Show preview before upload using `URL.createObjectURL`.
- **Profile form**: Use `react-hook-form` + Zod schema for `displayName` + `bio` fields. Auto-populate from `useAuthStore` state.
- **After updating profile**: Call `queryClient.invalidateQueries(['user', 'me'])` to refresh stale data.
- **Stats page**: Display with `Recharts` bar/line charts for study hours by week.

## Notes for Mobile (Member 4)

- **Avatar upload on mobile**: Use `expo-image-picker` to select image → convert to `FormData` → POST to `/users/me/avatar`.
- **Token storage**: Access token in memory (Zustand), profile data cached via TanStack Query.
- **Stats**: Show summary cards on the Home screen (streak, tasks done this week).
