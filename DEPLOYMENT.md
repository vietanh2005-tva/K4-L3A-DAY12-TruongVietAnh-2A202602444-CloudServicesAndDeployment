# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Trương Việt Anh |
| Mã học viên | 2A202602444 |
| Repo | https://github.com/vietanh2005-tva/K4-L3A-DAY12-TruongVietAnh-2A202602444-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-dn9e.onrender.com |
| Platform | Render |
| Ngày deploy | 2026-09-28 |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ ghi tên và nguồn của biến môi trường; giá trị secret không nằm trong repository.

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Render tự gán |
| `AGENT_API_KEY` | ✅ | Đặt trong Render Dashboard, không nằm trong repo |
| `REDIS_URL` | ✅ | Render Key Value `day12-redis` |
| `RATE_LIMIT_PER_MINUTE` | ✅ | Cấu hình bởi `render.yaml` |
| `MONTHLY_BUDGET_USD` | ✅ | Cấu hình bởi `render.yaml` |
| `LOG_LEVEL` | ✅ | Cấu hình bởi `render.yaml` |

## Kết Quả Kiểm Tra Thực Tế

Các endpoint được kiểm tra trực tiếp trên URL công khai sau khi Render báo trạng thái `Live`:

```text
GET /health
200 {"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready
200 {"status":"ready","redis":true}

POST /ask (không gửi X-API-Key)
401
```

Kết quả cho thấy process đang sống, kết nối Redis sẵn sàng và endpoint `/ask`
không cho phép truy cập khi thiếu API key.

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png` — Render Dashboard hiển thị web service `day12-agent` ở trạng thái `Live`.
