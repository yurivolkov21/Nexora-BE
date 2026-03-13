# Backend Setup Guide — Nexora

> Dành cho: **Member 1 (Leader)** và **Member 2**
> Đọc [`guides/00-getting-started.md`](../00-getting-started.md) trước khi bắt đầu.

---

## 1. Khởi tạo project NestJS

```bash
# Scaffold project mới
nest new nexora-be

# Chọn package manager: pnpm
# Chờ cài đặt xong

cd nexora-be
```

### Cấu trúc thư mục sau khi scaffold

```
nexora-be/
├── src/
│   ├── app.controller.ts
│   ├── app.module.ts
│   ├── app.service.ts
│   └── main.ts
├── test/
├── .eslintrc.js
├── .prettierrc
├── nest-cli.json
├── tsconfig.json
└── package.json
```

---

## 2. Cài đặt dependencies

```bash
# Core dependencies
pnpm add @nestjs/mongoose mongoose
pnpm add @nestjs/jwt @nestjs/passport passport passport-jwt passport-google-oauth20
pnpm add @nestjs/config joi
pnpm add @nestjs/swagger swagger-ui-express
pnpm add class-validator class-transformer
pnpm add bcryptjs
pnpm add @nestjs/websockets @nestjs/platform-socket.io socket.io
pnpm add bullmq @nestjs/bullmq
pnpm add nodemailer
pnpm add cloudinary multer @nestjs/platform-express
pnpm add openai
pnpm add firebase-admin

# Dev dependencies
pnpm add -D @types/passport-jwt @types/passport-google-oauth20
pnpm add -D @types/bcryptjs @types/multer
pnpm add -D @types/jest jest ts-jest supertest @types/supertest
```

---

## 3. Cấu hình cấu trúc thư mục chuẩn

Tạo lại cấu trúc thư mục theo kiến trúc đã định:

```bash
# Tạo thư mục modules
mkdir -p src/modules/auth/strategies
mkdir -p src/modules/auth/guards
mkdir -p src/modules/users
mkdir -p src/modules/tasks
mkdir -p src/modules/groups
mkdir -p src/modules/schedule
mkdir -p src/modules/forum
mkdir -p src/modules/notifications
mkdir -p src/modules/ai

# Tạo thư mục common và config
mkdir -p src/common/decorators
mkdir -p src/common/filters
mkdir -p src/common/interceptors
mkdir -p src/common/pipes
mkdir -p src/common/guards
mkdir -p src/config
mkdir -p src/shared/schemas
```

---

## 4. Cấu hình `.env`

Tạo file `.env.example` tại root project:

```env
# App
NODE_ENV=development
PORT=3000
FRONTEND_URL=http://localhost:3001

# MongoDB
MONGODB_URI=mongodb+srv://<user>:<password>@cluster.mongodb.net/nexora

# JWT
JWT_ACCESS_SECRET=your_access_secret_here
JWT_ACCESS_EXPIRES_IN=15m
JWT_REFRESH_SECRET=your_refresh_secret_here
JWT_REFRESH_EXPIRES_IN=7d

# Google OAuth
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_CALLBACK_URL=http://localhost:3000/auth/google/callback

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# OpenAI
OPENAI_API_KEY=sk-...

# Redis (Upstash)
REDIS_URL=redis://localhost:6379

# Email (Gmail SMTP)
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USER=your_email@gmail.com
MAIL_PASS=your_app_password

# Firebase FCM
FIREBASE_PROJECT_ID=your_project_id
FIREBASE_PRIVATE_KEY=your_private_key
FIREBASE_CLIENT_EMAIL=your_client_email
```

```bash
# Tạo .env thực từ template
cp .env.example .env
```

---

## 5. Cấu hình `main.ts`

```typescript
// src/main.ts
import { NestFactory } from '@nestjs/core';
import { ValidationPipe } from '@nestjs/common';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  // Global prefix
  app.setGlobalPrefix('api/v1');

  // CORS
  app.enableCors({
    origin: process.env.FRONTEND_URL,
    credentials: true,
  });

  // Global validation pipe
  app.useGlobalPipes(
    new ValidationPipe({
      whitelist: true,         // Loại bỏ field không có trong DTO
      forbidNonWhitelisted: true,
      transform: true,         // Tự động transform type
    }),
  );

  // Swagger
  const config = new DocumentBuilder()
    .setTitle('Nexora API')
    .setDescription('API documentation cho Nexora')
    .setVersion('1.0')
    .addBearerAuth()
    .build();
  const document = SwaggerModule.createDocument(app, config);
  SwaggerModule.setup('api', app, document);

  await app.listen(process.env.PORT ?? 3000);
  console.log(`🚀 Server running at http://localhost:${process.env.PORT ?? 3000}`);
  console.log(`📖 Swagger docs at http://localhost:${process.env.PORT ?? 3000}/api`);
}
bootstrap();
```

---

## 6. Cấu hình `app.module.ts`

```typescript
// src/app.module.ts
import { Module } from '@nestjs/common';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { MongooseModule } from '@nestjs/mongoose';
import * as Joi from 'joi';

@Module({
  imports: [
    // Config module — load .env và validate
    ConfigModule.forRoot({
      isGlobal: true,
      validationSchema: Joi.object({
        NODE_ENV: Joi.string().valid('development', 'production', 'test').default('development'),
        PORT: Joi.number().default(3000),
        MONGODB_URI: Joi.string().required(),
        JWT_ACCESS_SECRET: Joi.string().required(),
        JWT_REFRESH_SECRET: Joi.string().required(),
        OPENAI_API_KEY: Joi.string().required(),
      }),
    }),

    // MongoDB
    MongooseModule.forRootAsync({
      imports: [ConfigModule],
      useFactory: (configService: ConfigService) => ({
        uri: configService.get<string>('MONGODB_URI'),
      }),
      inject: [ConfigService],
    }),

    // Feature modules (thêm dần khi implement)
    // AuthModule,
    // UsersModule,
    // TasksModule,
    // ...
  ],
})
export class AppModule {}
```

---

## 7. Kết nối MongoDB Atlas

1. Truy cập [MongoDB Atlas](https://cloud.mongodb.com) → tạo tài khoản
2. Tạo cluster M0 (free tier)
3. Tạo database user (username + password)
4. Whitelist IP: `0.0.0.0/0` (cho phép tất cả — phù hợp cho dev)
5. Lấy connection string → paste vào `MONGODB_URI` trong `.env`

```bash
# Test kết nối bằng cách chạy app
pnpm run start:dev

# Nếu thấy log "Connected to MongoDB" là thành công
```

---

## 8. Docker Compose cho local dev (Member 1 setup)

Tạo `docker-compose.yml` tại root:

```yaml
version: '3.8'
services:
  mongodb:
    image: mongo:7
    container_name: nexora_mongo
    ports:
      - '27017:27017'
    volumes:
      - mongo_data:/data/db

  redis:
    image: redis:7-alpine
    container_name: nexora_redis
    ports:
      - '6379:6379'

volumes:
  mongo_data:
```

```bash
# Khởi động MongoDB + Redis local
docker-compose up -d

# Kiểm tra
docker ps
```

---

## 9. Swagger UI

Sau khi app đang chạy, truy cập:
```
http://localhost:3000/api
```

Mỗi khi thêm endpoint mới, đảm bảo có decorator Swagger:

```typescript
@ApiTags('auth')
@ApiOperation({ summary: 'Đăng nhập' })
@ApiResponse({ status: 200, description: 'Thành công' })
@ApiResponse({ status: 401, description: 'Sai credentials' })
```

---

## 10. Kiểm tra setup thành công

```bash
pnpm run start:dev
```

Checklist:
- [ ] App chạy không có lỗi tại `http://localhost:3000`
- [ ] Swagger UI hiển thị tại `http://localhost:3000/api`
- [ ] Kết nối MongoDB thành công (không có lỗi connection trong terminal)
- [ ] `.env` đã được điền đủ các giá trị bắt buộc

---

## Bước tiếp theo

- Member 1 → [`guides/backend/02-auth-module.md`](./02-auth-module.md)
- Member 2 → [`guides/backend/04-schedule-module.md`](./04-schedule-module.md) (sau khi Member 1 xong Auth)
- Tham khảo API contracts tại [`modules/`](../../modules/)
