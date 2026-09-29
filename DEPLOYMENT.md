# Thông Tin Deploy — Checkpoint 5

## Thông Tin Học Viên

| Mục | Nội dung |
|-----|----------|
| Họ và tên | Phạm Xuân Quý |
| Mã học viên | 2A202602745 |
| Repo | https://github.com/quycute2003/K4-L3B-DAY12-PhamXuanQuy-2A202602745-CloudServicesAndDeployment |

## Service

| Mục | Nội dung |
|-----|----------|
| Public URL | https://day12-agent-nqwq.onrender.com |
| Platform | Render Blueprint |
| Ngày deploy | 2026-09-29 |

## Biến Môi Trường Đã Set Trên Cloud

Chỉ liệt kê tên biến và nguồn giá trị; không lưu giá trị secret trong repository.

| Biến | Đã set | Ghi chú |
|------|--------|---------|
| `PORT` | ✅ | Render tự gán |
| `AGENT_API_KEY` | ✅ | Secret nhập trong Render dashboard |
| `REDIS_URL` | ✅ | Kết nối từ Render Key Value `day12-redis` qua Blueprint |
| `RATE_LIMIT_PER_MINUTE` | ✅ | Cấu hình trong `render.yaml` |
| `MONTHLY_BUDGET_USD` | ✅ | Cấu hình trong `render.yaml` |
| `LOG_LEVEL` | ✅ | Cấu hình trong `render.yaml` |

## Kết Quả Chạy Thật

Kiểm tra public service ngày 2026-09-29:

```text
GET /health
HTTP 200
{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready
HTTP 200
{"status":"ready","redis":true}

POST /ask (không có X-API-Key)
HTTP 401
{"detail":"invalid or missing API key"}

POST /ask (có X-API-Key và X-User-Id: sv-test)
HTTP 200
{"user_id":"sv-test","history_length":0,"cost_usd":2.145e-05,"tokens":{"in":3,"out":35}}
```

## Lệnh Kiểm Tra

```bash
URL=https://day12-agent-nqwq.onrender.com

curl -i "$URL/health"
curl -i "$URL/ready"
curl -i -X POST "$URL/ask" \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# Dùng API key của service từ môi trường cục bộ; không ghi key vào repo.
curl -i -X POST "$URL/ask" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $DEPLOY_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'
```

## Ảnh Chụp Màn Hình

- `screenshots/dashboard.png` — trang quản lý service trên Render.
- `screenshots/health.png` — kết quả gọi `/health` trên public URL.

Không dùng phương án local fallback; service đã được deploy công khai trên Render.
