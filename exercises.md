# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Các câu trả lời dưới đây dựa trên code và những lần kiểm tra thực tế của tôi.

Họ và tên: Phạm Thị Thùy Linh
Mã học viên: 2A202602909

---

### Câu 1 — Fail fast (CP1)

Nếu deploy lên Railway mà quên đặt `AGENT_API_KEY`, ứng dụng sẽ fail fast ngay khi tạo `Settings`. Nhờ vậy tôi biết cấu hình bắt buộc đang thiếu trước khi service public. Nếu để mặc định `changeme`, app vẫn chạy và người khác có thể đoán được khóa để gọi API.

### Câu 2 — Log cho máy đọc (CP1)

Một dòng log tôi kiểm tra được có dạng:

```json
{"event":"ask_completed","level":"info","timestamp":"2026-09-29T05:50:44.399163+00:00","user_id":"sv-test","tokens_in":12,"tokens_out":35,"cost_usd":0.0002}
```

Từ dòng này, tôi có thể lọc request theo `user_id` để điều tra một user và cộng `tokens_in`, `tokens_out` hoặc `cost_usd` để theo dõi chi phí. `timestamp` còn giúp dựng timeline. Một câu `print` không có cấu trúc ổn định để hệ thống log tự động lọc và thống kê.

### Câu 3 — Kích thước image (CP2)

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | Chưa đo được: Docker Engine trên máy chưa truy cập được |
| Multi-stage | Chưa đo được: Docker Engine trên máy chưa truy cập được |

Tôi không ghi bịa số MB vì Docker Engine hiện trả lỗi không có quyền kết nối. Về nguyên tắc, multi-stage nhỏ hơn vì stage runtime chỉ nhận package cần thiết từ builder, không mang theo cache cài đặt, công cụ build và file trung gian.

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Khi chỉ sửa `app/main.py`, các layer base image, `COPY requirements.txt` và `RUN pip install` được dùng lại từ cache; layer copy source và các layer sau đó phải chạy lại. Nếu đặt `COPY . .` trước `RUN pip install`, mọi thay đổi source đều làm layer copy thay đổi và khiến cài dependency chạy lại, dù `requirements.txt` không đổi.

### Câu 5 — Vì sao không chạy bằng root (CP2)

Nếu app có lỗ hổng cho phép thực thi lệnh, kẻ tấn công có thể chạy lệnh trong container. Khi container chạy bằng root, tiến trình đó có nhiều quyền đọc/ghi và có thể tìm cách khai thác tiếp để thoát container. `USER appuser` làm process chạy bằng user không đặc quyền, nên giới hạn quyền ban đầu và giảm tác động nếu code bị khai thác.

### Câu 6 — Cửa sổ trượt (CP3)

Tối đa là 20 request trong 2 giây: gửi 10 request ở giây 59 của một phút, sau đó bộ đếm reset ở giây 00 và gửi thêm 10 request ở giây 01 của phút kế tiếp. Sliding window 60 giây xóa request cũ theo timestamp nên không có khe hở reset này.

### Câu 7 — Rate limit và cost guard (CP3)

Rate limit giới hạn tốc độ hoặc số request trong một khoảng thời gian và trả 429. Cost guard giới hạn tổng chi phí mỗi user trong tháng và trả 402. Rate limit có thể cho qua một request đơn lẻ nhưng cost guard chặn nếu user đã tiêu gần hết ngân sách. Ngược lại, cost guard có thể cho qua request rẻ khi ngân sách còn, nhưng rate limit chặn nếu user vừa gửi đủ số request trong 60 giây.

### Câu 8 — /health khác /ready (CP4)

Nếu `/health` cũng kiểm tra Redis, khi Redis mất kết nối thì cả ba container đều bị báo unhealthy. Orchestrator có thể restart cả ba cùng lúc, làm cụm không còn instance phục vụ dù lỗi chỉ nằm ở Redis. Vì vậy `/health` chỉ kiểm tra process còn sống, còn `/ready` kiểm tra Redis để load balancer tạm ngừng gửi traffic.

### Câu 9 — Stateless (CP4)

Với dict Python, mỗi container có một bản sao lịch sử riêng. Khi load balancer chuyển request giữa A, B và C, `history_length` sẽ không tăng đều; nó có thể quay về 0 hoặc chỉ phản ánh lịch sử của container nhận request. Với Redis, cả ba container dùng chung key `history:<user_id>`, nên lịch sử được dùng chung.

### Câu 10 — Deploy thật (CP5)

Khi deploy Railway, tôi gặp lỗi Redis:

```text
exec: docker-entrypoint.sh: not found
```

Redis container restart liên tục nên `/health` vẫn hoạt động nhưng `/ready` trả 503 với `{"status":"not ready","redis":false}`. Tôi xác định nguyên nhân bằng deployment log của Redis và bằng cách gọi trực tiếp `/ready`. Cách sửa là dùng Redis service/image chính thức, xóa custom start command sai, reference `REDIS_URL` tới Redis service trong cùng Railway environment, rồi redeploy Redis và agent.
