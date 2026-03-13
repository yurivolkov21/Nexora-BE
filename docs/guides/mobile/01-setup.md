# Mobile Setup Guide — Nexora

> Dành cho: **Member 4**
> Đọc [`guides/00-getting-started.md`](../00-getting-started.md) trước khi bắt đầu.

---

## 1. Khởi tạo project Expo

```bash
npx create-expo-app@latest nexora-mobile

cd nexora-mobile
```

Kiểm tra Expo SDK version:

```bash
npx expo --version   # Nên là SDK 53
```

---

## 2. Cài đặt dependencies

```bash
# Navigation (Expo Router đã có sẵn từ create-expo-app)
npx expo install expo-router

# State management & data fetching (tái sử dụng logic từ web)
pnpm add @tanstack/react-query zustand axios

# Form & validation
pnpm add react-hook-form zod @hookform/resolvers

# Expo packages
npx expo install expo-notifications
npx expo install expo-secure-store          # Lưu token an toàn
npx expo install expo-image-picker
npx expo install expo-document-picker
npx expo install @react-native-async-storage/async-storage
npx expo install expo-constants
npx expo install expo-linking

# UI & Animations
npx expo install react-native-reanimated
npx expo install react-native-gesture-handler
npx expo install react-native-safe-area-context
npx expo install react-native-screens

# Styling
pnpm add nativewind
pnpm add -D tailwindcss@3   # NativeWind v4 yêu cầu Tailwind v3 — KHÔNG dùng v4

# Date
pnpm add date-fns
```

---

## 3. Cấu hình NativeWind (Tailwind cho React Native)

```bash
# Khởi tạo Tailwind config
npx tailwindcss init
```

Cập nhật `tailwind.config.js`:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./app/**/*.{js,jsx,ts,tsx}', './components/**/*.{js,jsx,ts,tsx}'],
  presets: [require('nativewind/preset')],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

Tạo `global.css` tại root:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Cập nhật `babel.config.js`:

```javascript
module.exports = function (api) {
  api.cache(true);
  return {
    presets: [
      ['babel-preset-expo', { jsxImportSource: 'nativewind' }],
      'nativewind/babel',
    ],
  };
};
```

Tạo (hoặc cập nhật) `metro.config.js` tại root:

```javascript
// metro.config.js
const { getDefaultConfig } = require('expo/metro-config');
const { withNativeWind } = require('nativewind/metro');

const config = getDefaultConfig(__dirname);

module.exports = withNativeWind(config, { input: './global.css' });
```

> **Lưu ý:** File này bắt buộc phải có — NativeWind v4 dùng Metro bundler để xử lý CSS. Thiếu file này thì `className` sẽ không render style nào.

---

## 4. Cấu hình Expo Router

Cập nhật `app.json`:

```json
{
  "expo": {
    "scheme": "nexora",
    "web": {
      "bundler": "metro"
    }
  }
}
```

Cấu trúc thư mục `app/` theo Expo Router:

```
app/
├── _layout.tsx              # Root layout
├── index.tsx                # Redirect → (auth)/login hoặc (tabs)/
├── (auth)/
│   ├── _layout.tsx
│   ├── login.tsx
│   └── register.tsx
└── (tabs)/
    ├── _layout.tsx          # Bottom tab navigator
    ├── index.tsx            # HomeScreen
    ├── tasks.tsx            # TasksScreen
    ├── schedule.tsx         # ScheduleScreen
    ├── forum.tsx            # ForumScreen
    └── profile.tsx          # ProfileScreen
```

---

## 5. Root Layout với Providers

```typescript
// app/_layout.tsx
import { Stack } from 'expo-router';
import { QueryClient, QueryClientProvider } from '@tanstack/react-query';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import '../global.css';

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000,
      retry: 1,
    },
  },
});

export default function RootLayout() {
  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <SafeAreaProvider>
        <QueryClientProvider client={queryClient}>
          <Stack screenOptions={{ headerShown: false }} />
        </QueryClientProvider>
      </SafeAreaProvider>
    </GestureHandlerRootView>
  );
}
```

---

## 6. Cấu hình Axios instance

```typescript
// lib/api.ts
import axios from 'axios';
import * as SecureStore from 'expo-secure-store';

const API_URL = process.env.EXPO_PUBLIC_API_URL ?? 'http://localhost:3000/api/v1';

const api = axios.create({
  baseURL: API_URL,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Tự động gắn Access Token vào header
api.interceptors.request.use(async (config) => {
  const token = await SecureStore.getItemAsync('accessToken');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

// Tự động refresh token khi nhận 401
api.interceptors.response.use(
  (response) => response,
  async (error) => {
    const originalRequest = error.config;
    if (error.response?.status === 401 && !originalRequest._retry) {
      originalRequest._retry = true;
      try {
        const refreshToken = await SecureStore.getItemAsync('refreshToken');
        const { data } = await axios.post(`${API_URL}/auth/refresh`, { refreshToken });
        await SecureStore.setItemAsync('accessToken', data.accessToken);
        return api(originalRequest);
      } catch {
        await SecureStore.deleteItemAsync('accessToken');
        await SecureStore.deleteItemAsync('refreshToken');
        // Redirect về login (xử lý trong auth store)
      }
    }
    return Promise.reject(error);
  },
);

export default api;
```

> Dùng `expo-secure-store` thay vì `AsyncStorage` để lưu token bảo mật hơn.

---

## 7. Cấu hình `.env`

Tạo `.env` tại root (Expo dùng prefix `EXPO_PUBLIC_`):

```env
EXPO_PUBLIC_API_URL=http://localhost:3000/api/v1
```

> Khi test trên thiết bị thật (không phải simulator), thay `localhost` bằng IP của máy tính trong cùng mạng LAN. Ví dụ: `http://192.168.1.100:3000/api/v1`

---

## 8. Setup Push Notification (Firebase FCM)

1. Tạo project trên [Firebase Console](https://console.firebase.google.com)
2. Thêm Android app → download `google-services.json` → đặt vào root
3. Thêm iOS app → download `GoogleService-Info.plist` → đặt vào root
4. Cập nhật `app.json`:

```json
{
  "expo": {
    "android": {
      "googleServicesFile": "./google-services.json"
    },
    "ios": {
      "googleServicesFile": "./GoogleService-Info.plist"
    },
    "plugins": [
      ["expo-notifications", {
        "icon": "./assets/notification-icon.png",
        "color": "#6366f1"
      }]
    ]
  }
}
```

---

## 9. Chạy app trên thiết bị

```bash
# Chạy trên Expo Go (development — không cần build)
npx expo start

# Chạy trên Android Emulator
npx expo start --android

# Chạy trên iOS Simulator (cần macOS)
npx expo start --ios
```

Scan QR code bằng app **Expo Go** trên điện thoại để xem live.

---

## 10. Kiểm tra setup thành công

Checklist:
- [ ] App chạy được trên Expo Go
- [ ] NativeWind hoạt động (thử `className="bg-blue-500 p-4"` trên một View)
- [ ] Expo Router navigation hoạt động (chuyển tab được)
- [ ] Axios call được API BE (thử gọi `GET /api/v1` hoặc health endpoint)
- [ ] Không còn warning về deprecated packages

---

## Lưu ý về New Architecture (RN 0.77)

React Native 0.77 bật New Architecture (Fabric) mặc định. Hầu hết các Expo packages đã hỗ trợ, nhưng nếu gặp lỗi với một thư viện bên thứ 3:

```javascript
// app.json — tắt New Architecture nếu cần (giải pháp tạm thời)
{
  "expo": {
    "newArchEnabled": false
  }
}
```

> Ưu tiên tìm phiên bản mới hơn của thư viện hỗ trợ New Architecture thay vì tắt đi.

---

## Bước tiếp theo

Sau khi BE (Member 1) xong Auth API → bắt đầu implement:
- `app/(auth)/login.tsx` — Màn hình đăng nhập
- `app/(auth)/register.tsx` — Màn hình đăng ký

Tham khảo API contracts tại [`modules/auth/api-endpoints.md`](../../modules/auth/api-endpoints.md)
