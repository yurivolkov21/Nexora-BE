# Frontend Web Setup Guide — Nexora

> Dành cho: **Member 3**
> Đọc [`guides/00-getting-started.md`](../00-getting-started.md) trước khi bắt đầu.

---

## 1. Khởi tạo project Next.js

```bash
pnpm create next-app@latest nexora-fe
```

Chọn các option sau khi được hỏi:

```
✔ Would you like to use TypeScript?          › Yes
✔ Would you like to use ESLint?              › Yes
✔ Would you like to use Tailwind CSS?        › Yes
✔ Would you like your code inside a `src/`?  › Yes
✔ Would you like to use App Router?          › Yes
✔ Would you like to use Turbopack?           › Yes
✔ Customize the default import alias?        › No
```

```bash
cd nexora-fe
```

---

## 2. Cài đặt dependencies

```bash
# State management & data fetching
pnpm add @tanstack/react-query zustand axios

# Form & validation
pnpm add react-hook-form zod @hookform/resolvers

# Real-time
pnpm add socket.io-client

# UI & Styling
pnpm add recharts@^3 date-fns
pnpm add @hello-pangea/dnd

# Rich text editor
pnpm add @tiptap/react @tiptap/pm @tiptap/starter-kit @tiptap/extension-image @tiptap/extension-link

# Dev dependencies
pnpm add -D @tanstack/react-query-devtools
```

---

## 3. Cài đặt shadcn/ui

> shadcn/ui đã hỗ trợ Tailwind v4. Dùng lệnh init mới nhất.

```bash
pnpm dlx shadcn@latest init
```

Chọn các option:

```
✔ Which style would you like to use?  › Default
✔ Which color would you like to use?  › Slate
✔ Would you like to use CSS variables for theming?  › Yes
```

Cài các components cần dùng ngay:

```bash
pnpm dlx shadcn@latest add button input label card dialog
pnpm dlx shadcn@latest add dropdown-menu avatar badge toast
pnpm dlx shadcn@latest add form select textarea checkbox
pnpm dlx shadcn@latest add sheet sidebar separator
```

---

## 4. Cấu trúc thư mục

Tạo cấu trúc theo kiến trúc đã định:

```bash
mkdir -p src/app/\(auth\)/login
mkdir -p src/app/\(auth\)/register
mkdir -p src/app/\(auth\)/forgot-password
mkdir -p src/app/\(dashboard\)/dashboard
mkdir -p src/app/\(dashboard\)/tasks/\[boardId\]
mkdir -p src/app/\(dashboard\)/schedule
mkdir -p src/app/\(dashboard\)/forum/\[postId\]
mkdir -p src/app/\(dashboard\)/forum/new
mkdir -p src/app/\(dashboard\)/groups/\[groupId\]
mkdir -p src/components/ui
mkdir -p src/components/task
mkdir -p src/components/schedule
mkdir -p src/components/forum
mkdir -p src/components/ai
mkdir -p src/components/layout
mkdir -p src/hooks
mkdir -p src/lib
mkdir -p src/stores
mkdir -p src/types
```

---

## 5. Cấu hình Axios instance

```typescript
// src/lib/api.ts
import axios from 'axios';

const api = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_URL ?? 'http://localhost:3000/api/v1',
  withCredentials: true, // Gửi cookie (cho refresh token)
});

// Interceptor: tự động gắn Access Token vào header
api.interceptors.request.use((config) => {
  const token = useAuthStore.getState().accessToken;
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Interceptor: tự động refresh token khi nhận 401
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      try {
        const { data } = await api.post('/auth/refresh');
        useAuthStore.getState().setAccessToken(data.accessToken);
        return api(originalRequest);
      } catch {
        useAuthStore.getState().logout();
        window.location.href = '/login';
      }
    }
    return Promise.reject(error);
  },
);

export default api;
```

> ⚠️ Import `useAuthStore` sau khi tạo store ở bước 6.

---

## 6. Cấu hình Zustand Store

```typescript
// src/stores/authStore.ts
import { create } from 'zustand';

interface User {
  _id: string;
  email: string;
  displayName: string;
  avatar?: string;
  role: 'admin' | 'member';
}

interface AuthState {
  user: User | null;
  accessToken: string | null;
  setUser: (user: User) => void;
  setAccessToken: (token: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>((set) => ({
  user: null,
  accessToken: null,
  setUser: (user) => set({ user }),
  setAccessToken: (accessToken) => set({ accessToken }),
  logout: () => set({ user: null, accessToken: null }),
}));
```

---

## 7. Cấu hình TanStack Query

```typescript
// src/app/providers.tsx
'use client';

import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { ReactQueryDevtools } from '@tanstack/react-query-devtools';
import { useState } from 'react';

export function Providers({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            staleTime: 60 * 1000, // 1 phút
            retry: 1,
          },
        },
      }),
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  );
}
```

```typescript
// src/app/layout.tsx — thêm Providers vào root layout
import { Providers } from './providers';

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="vi">
      <body>
        <Providers>{children}</Providers>
      </body>
    </html>
  );
}
```

---

## 8. Cấu hình `.env`

Tạo `.env.local` tại root:

```env
NEXT_PUBLIC_API_URL=http://localhost:3000/api/v1
NEXT_PUBLIC_SOCKET_URL=http://localhost:3000
```

> `.env.local` đã có trong `.gitignore` của Next.js mặc định.

---

## 9. Cấu hình TypeScript paths (alias)

Kiểm tra `tsconfig.json` đã có:

```json
{
  "compilerOptions": {
    "paths": {
      "@/*": ["./src/*"]
    }
  }
}
```

Dùng alias trong code:
```typescript
import { useAuthStore } from '@/stores/authStore';
import api from '@/lib/api';
```

---

## 10. Kiểm tra setup thành công

```bash
pnpm dev
```

Truy cập `http://localhost:3001` (hoặc Next.js sẽ tự chọn port 3000 nếu chưa có gì chạy).

Checklist:
- [ ] App chạy không lỗi
- [ ] Trang mặc định Next.js hiển thị bình thường
- [ ] shadcn/ui đã có trong `src/components/ui/`
- [ ] Không có lỗi TypeScript khi chạy `pnpm build`

---

## Lưu ý về Tailwind CSS v4

shadcn/ui init đã tự cấu hình Tailwind v4 đúng cách. Nếu cần thêm custom theme:

```css
/* src/app/globals.css — thêm custom variables vào @theme */
@import "tailwindcss";

@theme {
  --color-primary: #6366f1;
  --color-primary-foreground: #ffffff;
  /* ... */
}
```

> Không còn file `tailwind.config.js` ở v4. Mọi config đều nằm trong CSS.

---

## Bước tiếp theo

Sau khi BE (Member 1) xong Auth API → bắt đầu implement:
- `src/app/(auth)/login/page.tsx` — Trang đăng nhập
- `src/app/(auth)/register/page.tsx` — Trang đăng ký

Tham khảo API contracts tại [`modules/auth/api-endpoints.md`](../../modules/auth/api-endpoints.md)
