# Git Workflow — Nexora

> Quy ước Git bắt buộc cho toàn bộ thành viên. Đọc kỹ trước khi bắt đầu làm việc.

---

## Mô hình branch

```
main
  └── develop
        ├── feature/[tên-tính-năng]
        ├── fix/[mô-tả-bug]
        └── chore/[mô-tả]
```

| Branch | Mục đích | Ai được merge vào |
|---|---|---|
| `main` | Production-ready code | Chỉ Leader sau khi test kỹ |
| `develop` | Integration — nơi tích hợp các feature | Qua Pull Request, cần 1 reviewer |
| `feature/*` | Tính năng mới | Cá nhân tự tạo từ `develop` |
| `fix/*` | Sửa bug | Cá nhân tự tạo từ `develop` |
| `chore/*` | Config, deps, setup, refactor nhỏ | Cá nhân tự tạo từ `develop` |

---

## Quy ước đặt tên branch

### Format

```
[type]/[mô-tả-ngắn-gọn-dùng-dấu-gạch-ngang]
```

### Ví dụ đúng

```bash
feature/auth-login
feature/auth-google-oauth
feature/kanban-drag-and-drop
feature/task-filter-by-label
fix/jwt-refresh-token-expiry
fix/notification-not-sent
chore/setup-swagger-docs
chore/update-nestjs-dependencies
```

### Ví dụ sai

```bash
feature/AuthLogin          # Không dùng CamelCase
feature/auth login         # Không có dấu cách
new-feature                # Thiếu type prefix
my-branch                  # Không mô tả nội dung
```

---

## Commit Message — Conventional Commits

### Format

```
[type]: [mô tả ngắn, viết thường, không chấm cuối]
```

### Các type được dùng

| Type | Khi nào dùng |
|---|---|
| `feat` | Thêm tính năng mới |
| `fix` | Sửa bug |
| `chore` | Cập nhật deps, config, tooling |
| `docs` | Thêm/sửa tài liệu |
| `refactor` | Refactor code không thêm feature, không fix bug |
| `test` | Thêm hoặc sửa test |
| `style` | Format code, không thay đổi logic |
| `perf` | Cải thiện hiệu năng |

### Ví dụ đúng

```bash
feat: add google oauth login
feat: implement kanban drag and drop
fix: resolve jwt refresh token not rotating
fix: correct task filter by multiple labels
chore: update nestjs to v11.1
docs: add auth module api documentation
refactor: extract notification worker to separate service
test: add unit tests for auth service
```

### Ví dụ sai

```bash
Fix bug                    # Không có type, viết hoa
feat: Add Login Feature.   # Viết hoa, có dấu chấm
update code                # Quá chung chung
wip                        # Không mô tả gì
```

---

## Quy trình làm việc hàng ngày

### Bắt đầu tính năng mới

```bash
# 1. Luôn cập nhật develop trước
git checkout develop
git pull origin develop

# 2. Tạo branch mới từ develop
git checkout -b feature/tên-tính-năng

# 3. Code...

# 4. Commit thường xuyên (không để 1 commit quá lớn)
git add .
git commit -m "feat: add task creation form"

# 5. Push branch lên remote
git push origin feature/tên-tính-năng

# 6. Tạo Pull Request trên GitHub → base: develop
```

### Trong lúc làm việc — cập nhật code mới nhất từ develop

```bash
# Khi develop có thay đổi, cần sync vào branch đang làm
git fetch origin
git rebase origin/develop

# Nếu có conflict, giải quyết conflict rồi:
git rebase --continue
```

> Dùng `rebase` thay vì `merge` để giữ lịch sử commit sạch.

### Hoàn thành và merge

```bash
# Đảm bảo branch đã up-to-date với develop
git fetch origin
git rebase origin/develop

# Push lần cuối
git push origin feature/tên-tính-năng --force-with-lease

# Vào GitHub tạo Pull Request (nếu chưa có)
# Chờ reviewer approve → merge
# Sau khi merge, xóa branch
git checkout develop
git pull origin develop
git branch -d feature/tên-tính-năng
```

---

## Pull Request

### Tiêu đề PR

Dùng cùng format với commit message:

```
feat: add Google OAuth login
fix: resolve notification not delivered to mobile
```

### Mô tả PR nên có

```markdown
## Mô tả
[Tóm tắt ngắn gọn tính năng/fix này làm gì]

## Thay đổi chính
- [Thay đổi 1]
- [Thay đổi 2]

## Cách test
1. [Bước 1]
2. [Bước 2]

## Screenshots (nếu có UI)
[Paste ảnh vào đây]
```

### Quy tắc review

- PR phải được **ít nhất 1 người khác review** trước khi merge
- Reviewer nên comment trong vòng **24 giờ**
- Không self-merge (tự approve PR của bản thân) trừ khi Leader quyết định
- Nếu PR quá lớn (>500 lines), nên tách nhỏ ra

---

## Các lệnh Git hay dùng

```bash
# Xem trạng thái hiện tại
git status
git log --oneline --graph --all

# Undo commit cuối (giữ nguyên code, chỉ unstage)
git reset --soft HEAD~1

# Discard thay đổi chưa commit
git restore .

# Stash code đang làm dở khi cần đổi sang branch khác
git stash
git stash pop

# Xem diff trước khi commit
git diff --staged
```

---

## Những điều tuyệt đối không làm

```bash
# ❌ Push thẳng lên main hoặc develop
git push origin main

# ❌ Force push lên develop hoặc main
git push origin develop --force

# ❌ Commit trực tiếp lên develop
git checkout develop && git commit ...

# ❌ Merge mà không có reviewer
# → Luôn tạo PR và chờ approve
```
