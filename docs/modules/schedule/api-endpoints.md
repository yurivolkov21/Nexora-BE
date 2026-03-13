# Schedule Module — API Endpoints

> **Owner:** Member 2
> **Base path:** `/api/v1/schedule`
> **Status:** 🔄 Sẽ được cập nhật khi module được implement (Phase 1 — Tháng 4)

---

## Endpoints (Dự kiến)

| Method | Endpoint | Mô tả |
|---|---|---|
| `GET` | `/schedule` | Lấy danh sách sự kiện theo khoảng thời gian |
| `POST` | `/schedule` | Tạo sự kiện mới |
| `GET` | `/schedule/:id` | Chi tiết sự kiện |
| `PATCH` | `/schedule/:id` | Cập nhật sự kiện |
| `DELETE` | `/schedule/:id` | Xóa sự kiện |
| `GET` | `/schedule/summary` | Tóm tắt tiến độ (giờ học, task hoàn thành) theo tuần |

---

## Query Params dự kiến

```
GET /schedule?startDate=2026-03-01&endDate=2026-03-31&type=study
```

| Param | Mô tả |
|---|---|
| `startDate` | Ngày bắt đầu lọc (ISO 8601) |
| `endDate` | Ngày kết thúc lọc (ISO 8601) |
| `type` | `study` \| `personal` \| `group` |

---

## Schema dự kiến — Event

```typescript
{
  id: string;
  title: string;
  description?: string;
  startTime: Date;
  endTime: Date;
  color: string;          // Hex color, VD: "#6366f1"
  type: 'study' | 'personal' | 'group';
  isRecurring: boolean;
  recurrenceRule?: string; // iCal RRULE format
  linkedTaskId?: string;   // Nếu sự kiện gắn với task deadline
  createdBy: string;
  groupId?: string;
  createdAt: Date;
  updatedAt: Date;
}
```

---

> 📝 **TODO:** Member 2 cập nhật file này sau khi implement xong module.
