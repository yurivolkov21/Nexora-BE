# Auth Module — API Endpoints

> Tài liệu tham chiếu API cho module Authentication.
> Dùng để FE và Mobile biết endpoint nào cần gọi mà không cần hỏi BE.

**Base path:** `/api/v1/auth`
**Owner:** Member 1

---

## Endpoints

### POST `/auth/register`

Đăng ký tài khoản mới.

**Request Body:**

```json
{
  "email": "user@example.com",
  "password": "StrongPass123!",
  "displayName": "Nguyen Van A"
}
```

**Response `201`:**

```json
{
  "statusCode": 201,
  "message": "Đăng ký thành công",
  "data": {
    "id": "65f1a2b3c4d5e6f7a8b9c0d1",
    "email": "user@example.com",
    "displayName": "Nguyen Van A",
    "role": "member",
    "createdAt": "2026-03-07T08:00:00.000Z"
  }
}
```

**Errors:**
- `400` — Thiếu field bắt buộc hoặc validation thất bại
- `409` — Email đã tồn tại

---

### POST `/auth/login`

Đăng nhập bằng email + password.

**Request Body:**

```json
{
  "email": "user@example.com",
  "password": "StrongPass123!"
}
```

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Đăng nhập thành công",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "user": {
      "id": "65f1a2b3c4d5e6f7a8b9c0d1",
      "email": "user@example.com",
      "displayName": "Nguyen Van A",
      "avatar": null,
      "role": "member"
    }
  }
}
```

> Refresh token được set tự động trong **HTTP-only cookie** (`refresh_token`).

**Errors:**
- `401` — Email hoặc password sai

---

### POST `/auth/refresh`

Lấy Access Token mới bằng Refresh Token trong cookie.

**Headers:** Không cần Authorization header.
**Cookie:** `refresh_token` (tự động gửi nếu `withCredentials: true`)

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Token mới đã được tạo",
  "data": {
    "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

**Errors:**
- `401` — Refresh token không hợp lệ hoặc đã hết hạn → redirect về login

---

### POST `/auth/logout`

Đăng xuất, revoke refresh token.

**Headers:** `Authorization: Bearer <access_token>`

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Đăng xuất thành công",
  "data": null
}
```

> Cookie `refresh_token` sẽ bị xóa tự động.

---

### GET `/auth/google`

Redirect đến Google OAuth consent screen.

> Gọi bằng cách redirect browser, không phải Axios call.

```typescript
// FE
window.location.href = `${API_URL}/auth/google`;
```

---

### GET `/auth/google/callback`

Google OAuth callback. Được gọi tự động bởi Google sau khi user authorize.

> Sau khi xử lý xong → redirect về FE với access token trong query param (hoặc cookie).

---

### POST `/auth/forgot-password`

Gửi OTP qua email để reset password.

**Request Body:**

```json
{
  "email": "user@example.com"
}
```

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "OTP đã được gửi đến email của bạn",
  "data": null
}
```

---

### POST `/auth/reset-password`

Đặt lại password bằng OTP.

**Request Body:**

```json
{
  "email": "user@example.com",
  "otp": "123456",
  "newPassword": "NewStrongPass123!"
}
```

**Response `200`:**

```json
{
  "statusCode": 200,
  "message": "Mật khẩu đã được đặt lại thành công",
  "data": null
}
```

**Errors:**
- `400` — OTP sai hoặc đã hết hạn (10 phút)

---

## MongoDB Schema — User

```typescript
// src/shared/schemas/user.schema.ts

@Schema({ timestamps: true, versionKey: false })
export class User {
  @Prop({ required: true, unique: true, lowercase: true, trim: true })
  email: string;

  @Prop({ required: true, select: false }) // select: false → không trả về trong query mặc định
  password: string;

  @Prop({ required: true, trim: true })
  displayName: string;

  @Prop({ default: null })
  avatar: string;

  @Prop({ enum: ['admin', 'member'], default: 'member' })
  role: string;

  @Prop({ default: null })
  googleId: string;

  @Prop({ default: null, select: false })
  refreshToken: string;          // Lưu hashed refresh token

  @Prop({ default: null, select: false })
  resetPasswordOtp: string;

  @Prop({ default: null, select: false })
  resetPasswordOtpExpiry: Date;

  @Prop({ default: null })
  deletedAt: Date;               // Soft delete
}
```

---

## Lưu ý tích hợp

### FE Web (Next.js)

```typescript
// Lưu accessToken trong memory (Zustand store), KHÔNG lưu localStorage
useAuthStore.getState().setAccessToken(data.accessToken);

// Axios interceptor tự động gọi /auth/refresh khi nhận 401
// Xem: guides/frontend/01-setup.md
```

### Mobile (React Native)

```typescript
// Lưu accessToken trong expo-secure-store (không phải AsyncStorage)
await SecureStore.setItemAsync('accessToken', data.accessToken);

// Refresh token: mobile cần gửi trong body (không hỗ trợ HTTP-only cookie tốt trên mobile)
// BE cần có endpoint POST /auth/refresh hỗ trợ cả 2 cách: cookie VÀ body
```
