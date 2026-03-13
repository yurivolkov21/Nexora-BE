# Code Style — Nexora

> Quy ước viết code cho toàn bộ thành viên. Áp dụng cho cả BE (NestJS) và FE (Next.js, React Native).

---

## TypeScript

### Luôn khai báo kiểu rõ ràng

```typescript
// ✅ Đúng
const userId: string = '123';
function getUser(id: string): Promise<User> { ... }

// ❌ Sai
const userId = '123' as any;
function getUser(id) { ... }
```

### Interface vs Type

- Dùng `interface` cho object shape (có thể extend)
- Dùng `type` cho union, intersection, primitive alias

```typescript
// ✅
interface User {
  id: string;
  email: string;
  role: UserRole;
}

type UserRole = 'admin' | 'member';
type ApiResponse<T> = { data: T; message: string };
```

### Enum

Dùng kiểu `const` enum hoặc union type, tránh dùng numeric enum:

```typescript
// ✅
const TaskStatus = {
  TODO: 'todo',
  IN_PROGRESS: 'in_progress',
  DONE: 'done',
} as const;

type TaskStatus = typeof TaskStatus[keyof typeof TaskStatus];

// ❌
enum TaskStatus {
  TODO = 0,
  IN_PROGRESS = 1,
}
```

---

## Naming Conventions

| Loại | Convention | Ví dụ |
|---|---|---|
| Variable, function | camelCase | `getTaskById`, `userId` |
| Class, Interface | PascalCase | `TaskService`, `UserDto` |
| Constant | UPPER_SNAKE_CASE | `JWT_SECRET`, `MAX_RETRY` |
| File (BE) | kebab-case | `task.service.ts`, `auth.module.ts` |
| File (FE) | kebab-case hoặc PascalCase cho component | `TaskCard.tsx`, `use-auth.ts` |
| CSS class | kebab-case (Tailwind) | `bg-primary`, `flex-col` |
| Env variable | UPPER_SNAKE_CASE | `DATABASE_URL`, `JWT_SECRET` |

---

## Backend (NestJS) Conventions

### Cấu trúc Module

Mỗi module gồm đủ 4 file cơ bản:

```
tasks/
├── tasks.module.ts
├── tasks.controller.ts
├── tasks.service.ts
└── dto/
    ├── create-task.dto.ts
    └── update-task.dto.ts
```

### DTO Validation

```typescript
// ✅ Luôn dùng class-validator + class-transformer
import { IsString, IsNotEmpty, IsOptional, IsEnum, IsDateString } from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class CreateTaskDto {
  @ApiProperty({ description: 'Tiêu đề task' })
  @IsString()
  @IsNotEmpty()
  title: string;

  @ApiPropertyOptional({ description: 'Mô tả task' })
  @IsOptional()
  @IsString()
  description?: string;

  @ApiPropertyOptional({ enum: TaskPriority })
  @IsOptional()
  @IsEnum(TaskPriority)
  priority?: TaskPriority;
}
```

### Service — Business Logic

```typescript
// ✅ Service chứa toàn bộ business logic, Controller chỉ gọi service
@Injectable()
export class TasksService {
  constructor(
    @InjectModel(Task.name) private taskModel: Model<Task>,
  ) {}

  async findAll(userId: string, query: QueryTaskDto): Promise<PaginatedResult<Task>> {
    // Logic ở đây
  }
}

// ✅ Controller chỉ nhận request, gọi service, trả response
@Controller('tasks')
export class TasksController {
  constructor(private readonly tasksService: TasksService) {}

  @Get()
  findAll(@CurrentUser() user: User, @Query() query: QueryTaskDto) {
    return this.tasksService.findAll(user.id, query);
  }
}
```

### Error Handling

```typescript
// ✅ Dùng NestJS built-in exceptions, không throw Error trực tiếp
import { NotFoundException, ForbiddenException } from '@nestjs/common';

async findById(id: string): Promise<Task> {
  const task = await this.taskModel.findById(id);
  if (!task) throw new NotFoundException('Task không tồn tại');
  return task;
}
```

---

## Frontend (Next.js) Conventions

### Component Structure

```typescript
// ✅ Mỗi component trong file riêng, export default
// src/components/task/TaskCard.tsx

interface TaskCardProps {
  task: Task;
  onUpdate?: (id: string) => void;
}

export default function TaskCard({ task, onUpdate }: TaskCardProps) {
  return (
    <div className="rounded-lg border p-4">
      <h3 className="font-medium">{task.title}</h3>
    </div>
  );
}
```

### Custom Hooks

```typescript
// ✅ Tách data fetching vào custom hook, tái sử dụng được
// src/hooks/useTasks.ts

export function useTasks(boardId: string) {
  return useQuery({
    queryKey: ['tasks', boardId],
    queryFn: () => api.get(`/tasks?boardId=${boardId}`).then(r => r.data),
  });
}

export function useCreateTask() {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: CreateTaskDto) => api.post('/tasks', data),
    onSuccess: () => queryClient.invalidateQueries({ queryKey: ['tasks'] }),
  });
}
```

### Server vs Client Components (Next.js)

```typescript
// ✅ Mặc định là Server Component — không cần 'use client'
// Dùng cho: fetch data, SEO, layout tĩnh
export default async function TasksPage() {
  return <TaskBoard />;
}

// ✅ Thêm 'use client' chỉ khi cần: useState, useEffect, event handlers, browser API
'use client';
export default function TaskCard({ task }: { task: Task }) {
  const [isOpen, setIsOpen] = useState(false);
  return <div onClick={() => setIsOpen(true)}>...</div>;
}
```

---

## Mobile (React Native) Conventions

### StyleSheet vs NativeWind

```typescript
// ✅ Dùng NativeWind (className) cho styling thông thường
<View className="flex-1 bg-white p-4">
  <Text className="text-lg font-semibold text-gray-900">Title</Text>
</View>

// ✅ Dùng StyleSheet chỉ khi cần dynamic style hoặc animation
const styles = StyleSheet.create({
  animated: { transform: [{ translateX: animatedValue }] }
});
```

### Tái sử dụng hooks từ Web

```typescript
// ✅ Hook viết không phụ thuộc vào DOM/browser API — dùng được cho cả web và mobile
// src/hooks/useTasks.ts (shared)
export function useTasks(boardId: string) {
  return useQuery({
    queryKey: ['tasks', boardId],
    queryFn: () => api.get(`/tasks?boardId=${boardId}`).then(r => r.data),
  });
}
```

---

## Chung — Những điều tuyệt đối không làm

```typescript
// ❌ Không dùng any
const data: any = response.data;

// ❌ Không bỏ qua lỗi TypeScript bằng @ts-ignore mà không có comment giải thích
// @ts-ignore
doSomething();

// ❌ Không hardcode URL, credential, secret vào code
const API_URL = 'http://localhost:3000'; // Hardcode
const SECRET = 'abc123';               // Hardcode secret

// ❌ Không commit file .env
git add .env

// ❌ Không để console.log thừa trong code trước khi merge
console.log('debug:', data);
```

---

## Formatter & Linter

Cả 3 repo đều dùng:

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": true,
  "printWidth": 100,
  "trailingComma": "all",
  "tabWidth": 2
}
```

Chạy trước khi commit:

```bash
# Format code
pnpm run format

# Lint check
pnpm run lint

# Hoặc tự động với lint-staged (cấu hình trong package.json)
```

> VS Code với extension **Prettier** và **ESLint** sẽ tự format khi lưu file nếu cấu hình `editor.formatOnSave: true`.
