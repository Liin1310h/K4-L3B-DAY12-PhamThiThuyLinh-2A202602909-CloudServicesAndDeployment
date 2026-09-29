# Thông Tin Deploy — Checkpoint 5

> Điền file này sau khi deploy xong. `pytest tests/test_cp5.py` đọc file này
> để tìm địa chỉ service của bạn và gọi thử.
>
> **Chỉ ghi TÊN biến môi trường, tuyệt đối không dán giá trị API key vào đây.**
> Repo này công khai — dán khóa vào là mất khóa.

## Thông Tin Học Viên

| Mục         | Nội dung                                                                                         |
| ----------- | ------------------------------------------------------------------------------------------------ |
| Họ và tên   | Phạm Thị Thùy Linh                                                                               |
| Mã học viên | 2A202602909                                                                                      |
| Repo        | https://github.com/Liin1310h/K4-L3B-DAY12-PhamThiThuyLinh-2A202602909-CloudServicesAndDeployment |

## Service

| Mục         | Nội dung                                                                           |
| ----------- | ---------------------------------------------------------------------------------- |
| Public URL  | https://k4-l3b-day12-phamthithuylinh-2a202602909-cloudse-production.up.railway.app |
| Platform    | Railway                                                                            |
| Ngày deploy | 29/09/2026                                                                         |

## Biến Môi Trường Đã Set Trên Cloud

Ghi tên biến và **nguồn giá trị**, không ghi giá trị:

| Biến                    | Đã set | Ghi chú                                                                           |
| ----------------------- | ------ | --------------------------------------------------------------------------------- |
| `PORT`                  | ✅     | platform tự gán                                                                   |
| `AGENT_API_KEY`         | ✅     | đặt trong dashboard, không nằm trong repo                                         |
| `REDIS_URL`             | ⚠️     | Đã cấu hình Reference Variable tới Railway Redis nhưng `/ready` chưa kết nối được |
| `RATE_LIMIT_PER_MINUTE` | ✅     | 10                                                                                |
| `MONTHLY_BUDGET_USD`    | ✅     | 10.0                                                                              |
| `LOG_LEVEL`             | ✅     | INFO                                                                              |

## Lệnh Kiểm Tra

Public URL sử dụng trong các lệnh dưới đây:

```bash
# 1. Liveness — mong đợi 200 {"status":"ok"}
curl -i https://k4-l3b-day12-phamthithuylinh-2a202602909-cloudse-production.up.railway.app/health

# 2. Readiness — mong đợi 200 {"status":"ready"} (đã nối được Redis)
curl -i https://k4-l3b-day12-phamthithuylinh-2a202602909-cloudse-production.up.railway.app/ready

# 3. Không có API key — mong đợi 401
curl -i -X POST https://k4-l3b-day12-phamthithuylinh-2a202602909-cloudse-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -d '{"question":"Hello"}'

# 4. Có API key — mong đợi 200 kèm câu trả lời
curl -i -X POST https://k4-l3b-day12-phamthithuylinh-2a202602909-cloudse-production.up.railway.app/ask \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $AGENT_API_KEY" \
  -H "X-User-Id: sv-test" \
  -d '{"question":"Deploy là gì?"}'

# 5. Rate limit — gọi 15 lần, những lần cuối phải trả 429
for i in $(seq 1 15); do
  curl -s -o /dev/null -w "%{http_code} " -X POST https://k4-l3b-day12-phamthithuylinh-2a202602909-cloudse-production.up.railway.app/ask \
    -H "Content-Type: application/json" \
    -H "X-API-Key: $AGENT_API_KEY" \
    -H "X-User-Id: sv-test" \
    -d '{"question":"test"}'
done; echo
```

## Kết Quả Chạy Thật

Dán output của các lệnh trên vào đây:

```
(.venv) PS E:\AI_VIN\Lab12\K4-L3B-DAY12-PhamThiThuyLinh-2A202602909-CloudServicesAndDeployment> curl.exe -i https://k4-l3b-day12-phamthithuylinh-2a202602909-cloudse-production.up.railway.app/health
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 29 Sep 2026 04:47:29 GMT
Server: railway-hikari
x-railway-request-id: dksw8lPzQcGWgOX3npoFkQ
Content-Length: 57
x-hikari-trace: hkg1.aebn,hnd1.4cm0
x-railway-edge: hkg1
Connection: keep-alive

{"status":"ok","service":"day12-agent","version":"1.0.0"}

GET /ready — đã kiểm tra: HTTP/1.1 503 Service Unavailable
{"status":"not ready","redis":false}

Nguyên nhân: Railway agent chưa kết nối được Redis. Cần sửa Reference Variable
REDIS_URL trên Railway rồi deploy lại trước khi chạy test CP5 cuối cùng.
(.venv) PS E:\AI_VIN\Lab12\K4-L3B-DAY12-PhamThiThuyLinh-2A202602909-CloudServicesAndDeployment> curl.exe -i https://k4-l3b-day12-phamthithuylinh-2a202602909-cloudse-production.up.railway.app/ready
HTTP/1.1 200 OK
Content-Type: application/json
Date: Tue, 29 Sep 2026 05:47:03 GMT
Server: railway-hikari
x-railway-request-id: kHzRozWNShG2T0lZCYBc-A
Content-Length: 31
x-hikari-trace: hkg1.aebn
x-railway-edge: hkg1
Connection: keep-alive

{"status":"ready","redis":true}
```

## Ảnh Chụp Màn Hình

Đặt ảnh trong thư mục `screenshots/`:

- `screenshots/dashboard.png` — trang quản lý service trên platform
- `screenshots/health.png` — kết quả gọi `/health` từ trình duyệt hoặc curl

---

## Trạng thái CP5

Service đã có public HTTPS URL và `/health` hoạt động. CP5 chưa hoàn tất vì
`/ready` đang trả `503` với `{"redis":false}`; cần sửa kết nối Railway Redis,
kiểm tra lại `/ready`, sau đó cập nhật phần “Kết Quả Chạy Thật” và chạy lại
`pytest tests/test_cp5.py -v`.
