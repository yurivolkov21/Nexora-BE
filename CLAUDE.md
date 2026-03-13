# Nexora — Backend (NestJS)

## Project Context
Nexora là nền tảng cộng tác học tập cho sinh viên: Task Management (Kanban) + Study Schedule + Forum + AI Assistant.
Stack: **NestJS 11** · TypeScript 5 · Mongoose 8 · Passport.js · Socket.io 4 · BullMQ 5 · OpenAI GPT-4o

Docs đầy đủ: `docs/` (Nexora-Docs submodule)
- API conventions: `docs/conventions/api-design.md`
- Module API specs: `docs/be/modules/`
- Setup guide: `docs/be/01-setup.md`

---

## Project Structure
```
src/
├── modules/
│   ├── auth/           # JWT + Passport + Google OAuth
│   ├── users/          # User profile, avatar (Cloudinary)
│   ├── groups/         # Group management, invite system
│   ├── tasks/          # Kanban board, CRUD, assignments
│   ├── schedule/       # Calendar events, recurring, Pomodoro
│   ├── forum/          # Posts, comments, voting, search
│   ├── notifications/  # WebSocket + FCM + Email (BullMQ)
│   └── ai/             # GPT-4o chatbot (SSE), task suggestion
├── common/
│   ├── decorators/     # @GetUser(), @Roles(), @Public()
│   ├── guards/         # JwtAuthGuard, RolesGuard
│   ├── filters/        # GlobalExceptionFilter
│   ├── interceptors/   # ResponseTransformInterceptor
│   ├── pipes/          # ZodValidationPipe
│   └── types/          # Shared TypeScript types
├── config/             # ConfigModule, env validation (Joi)
└── shared/
    └── schemas/        # Reusable Mongoose schemas/subdocuments
```

---

## NestJS Patterns — ALWAYS FOLLOW

### Module Structure
Mỗi module phải có đủ: `module.ts`, `controller.ts`, `service.ts`
DTOs trong `dto/` folder, Mongoose schema trong `schemas/` folder.

```typescript
// ✅ Correct — inject service qua constructor
@Injectable()
export class TasksService {
  constructor(
    @InjectModel(Task.name) private taskModel: Model<Task>,
    private notificationsService: NotificationsService,
  ) {}
}

// ❌ Wrong — không dùng property injection
@Injectable()
export class TasksService {
  @Inject() private taskModel: Model<Task>  // sai
}
```

### Controller Rules
```typescript
// ✅ Đúng
@Controller('tasks')
@UseGuards(JwtAuthGuard)
@ApiBearerAuth()
@ApiTags('tasks')
export class TasksController {
  @Get()
  @ApiOperation({ summary: 'Lấy danh sách task' })
  @ApiResponse({ status: 200, type: TaskListResponseDto })
  findAll(@GetUser() user: UserDocument, @Query() query: FilterTaskDto) {
    return this.tasksService.findAll(user._id, query)
  }
}
```

### DTO & Validation
```typescript
// Dùng class-validator + class-transformer cho tất cả DTO
export class CreateTaskDto {
  @IsString()
  @IsNotEmpty()
  @MaxLength(200)
  title: string

  @IsEnum(TaskPriority)
  priority: TaskPriority

  @IsOptional()
  @IsDateString()
  deadline?: string
}
```

### Mongoose Schema
```typescript
@Schema({ timestamps: true })
export class Task {
  @Prop({ required: true, maxlength: 200 })
  title: string

  @Prop({ type: Types.ObjectId, ref: 'User', required: true })
  createdBy: Types.ObjectId

  @Prop({ default: null })
  deletedAt: Date | null  // soft delete pattern
}
export const TaskSchema = SchemaFactory.createForClass(Task)
```

### Soft Delete (CRITICAL)
Resource quan trọng (users, tasks, posts) dùng soft delete:
```typescript
// Delete
await this.taskModel.findByIdAndUpdate(id, { deletedAt: new Date() })

// Query — luôn filter deletedAt: null
await this.taskModel.find({ deletedAt: null, ...otherFilters })
```

---

## API Response Format (BẮTBUỘC)

Mọi response phải theo chuẩn này — dùng `ResponseTransformInterceptor`:

```typescript
// Success
{ "statusCode": 200, "message": "...", "data": { ... } }

// Success với pagination
{
  "statusCode": 200,
  "data": { "items": [...], "total": 100, "page": 1, "limit": 20, "totalPages": 5 }
}

// Error (GlobalExceptionFilter tự handle)
{ "statusCode": 400, "message": "...", "errors": [{ "field": "...", "message": "..." }] }
```

HTTP Status codes: `200` GET/PATCH, `201` POST, `204` DELETE, `400` validation, `401` unauth, `403` forbidden, `404` not found, `409` conflict.

---

## Authentication Flow

```typescript
// Access Token: 15 phút, trong Authorization header
// Refresh Token: 7 ngày, trong HTTP-only cookie

// Dùng @GetUser() decorator để lấy user trong controller
@Get('me')
getMe(@GetUser() user: UserDocument) {
  return user
}

// @Public() để skip auth guard
@Public()
@Post('auth/login')
login(@Body() dto: LoginDto) { ... }
```

---

## AI Module — Cost Control (QUAN TRỌNG)

```typescript
// GPT-4o-mini: chatbot, summarize (rẻ hơn)
// GPT-4o: suggest tasks (chỉ khi cần reasoning cao)

// Rate limit: 20 requests/user/hour
// SSE streaming cho chatbot responses
// Lưu conversation history trong MongoDB (tối đa 50 turns)
```

---

## WebSocket (Socket.io)

```typescript
// Events chuẩn cho Notifications module
'notification:new'        // gửi đến user cụ thể
'task:assigned'           // gửi đến assignee
'task:deadline_reminder'  // gửi trước 24h

// Join room theo userId khi connect
socket.join(`user:${userId}`)
```

---

## BullMQ Queue (Email & Push Notifications)

```typescript
// Queue names
'email-queue'        // email notifications
'push-queue'         // Firebase FCM push
'cleanup-queue'      // periodic cleanup jobs

// Luôn handle job failures
@Process()
async handleJob(job: Job<EmailJobData>) {
  try { ... }
  catch (error) {
    // log error với job context
    throw error  // BullMQ sẽ retry theo config
  }
}
```

---

## Testing

```typescript
// Unit test: mock dependencies
// Integration test: dùng @nestjs/testing + mongodb-memory-server
// E2E test: supertest với real HTTP calls

// Coverage target: 80% minimum, 100% cho auth/security logic
```

---

## Environment Variables (.env)
```
MONGODB_URI=
REDIS_URL=
JWT_ACCESS_SECRET=
JWT_REFRESH_SECRET=
OPENAI_API_KEY=
CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=
FIREBASE_SERVICE_ACCOUNT_KEY=
GMAIL_USER=
GMAIL_APP_PASSWORD=
GOOGLE_CLIENT_ID=
GOOGLE_CLIENT_SECRET=
PORT=3000
NODE_ENV=development
```

---

## DO NOT
- Đừng dùng `any` type — dùng proper TypeScript types
- Đừng hardcode port, secret, URL — luôn dùng `ConfigService`
- Đừng gọi trực tiếp `Model.find()` trong controller — phải qua service
- Đừng trả raw Mongoose document — transform qua DTO/serializer
- Đừng dùng `PUT` — dùng `PATCH` cho partial updates
- Đừng viết endpoint mới mà không có Swagger decorators

## Slash Commands Available
- `/plan` — Lên kế hoạch trước khi implement một module/feature
- `/tdd` — Implement theo TDD (RED → GREEN → REFACTOR)
- `/code-review` — Review toàn bộ thay đổi trước khi commit
- `/build-fix` — Sửa build error có hệ thống
