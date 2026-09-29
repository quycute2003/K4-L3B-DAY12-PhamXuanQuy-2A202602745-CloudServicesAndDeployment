# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay từng dòng giữ chỗ bên dưới bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Phạm Xuân Quý  Mã học viên: 2A202602745

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Khi deploy lên Render, nếu tôi quên khai báo `AGENT_API_KEY` thì ứng dụng dừng ngay và log chỉ rõ biến cấu hình bắt buộc đang thiếu. Nhờ vậy tôi sửa cấu hình trước khi public URL nhận traffic. Nếu dùng mặc định `"changeme"`, service vẫn báo khỏe nhưng bot có thể đoán khóa này, gọi `/ask` trái phép và làm tiêu ngân sách mà tôi khó phát hiện nguyên nhân.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng tôi thu được khi gọi `/ask` là: `{"event": "ask_completed", "level": "info", "timestamp": "2026-09-29T03:30:39.259403+00:00", "user_id": "exercise-log", "tokens_in": 5, "tokens_out": 37, "cost_usd": 2.295e-05}`. Vì đây là JSON, hệ thống log có thể lọc chính xác theo `event`, `level` hoặc `user_id`; đồng thời có thể cộng `cost_usd`, thống kê token và vẽ biểu đồ theo `timestamp`. Dòng `print("đã trả lời xong")` không cung cấp trường có cấu trúc để tìm kiếm hay tính toán như vậy.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | Quá trình đo lại bị nghẽn ở Wi-Fi trường; riêng các layer nén của base `python:3.11` đã phải tải hơn 400 MB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Tôi đo được image production multi-stage là 271 MB. Khi dựng lại bản đầu, Docker phải tải nhiều layer lớn của image `python:3.11` đầy đủ; riêng các layer nén hiển thị trong log đã vượt 400 MB trước khi cài dependency. Phần chênh lệch chủ yếu là hệ điều hành và công cụ đi kèm base image đầy đủ, cùng các tệp trung gian/cache cài đặt. Bản runtime dùng `python:3.11-slim` và chỉ nhận dependency đã cài từ builder nên không mang các phần không cần để chạy ứng dụng.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Dockerfile của tôi copy `requirements.txt` rồi chạy `pip install` trước khi copy `app/` và `utils/`. Vì vậy khi chỉ sửa `app/main.py`, layer base, tạo thư mục làm việc, copy requirements và cài dependency đều lấy lại từ cache; chỉ layer copy source và các layer sau nó phải tạo lại. Nếu đặt `COPY . .` trước `RUN pip install`, bất kỳ thay đổi source nào cũng làm layer copy đổi checksum và buộc `pip install` chạy lại dù `requirements.txt` không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Một lỗ hổng như command injection có thể cho kẻ tấn công thực thi lệnh trong container. Nếu process đang chạy bằng root, mã độc có toàn quyền trong filesystem và namespace của container; nếu runtime hoặc cấu hình mount có thêm lỗ hổng, nó có thể sửa file được mount, truy cập socket Docker hoặc khai thác container escape để giành quyền cao trên host. Lệnh `USER agent` làm payload chỉ có quyền của user thường, nên chặn bước “chiếm ứng dụng” tự động biến thành “có root trong container”. Nó không thay thế việc vá lỗi nhưng giảm đáng kể phạm vi thiệt hại.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi tối đa 20 request trong 2 giây: gửi 10 request ở cuối phút cũ, ví dụ 12:00:59.x, rồi gửi tiếp 10 request ngay đầu phút mới, 12:01:00.x. Bộ đếm theo phút đã reset ở giây 00 nên cả hai nhóm đều hợp lệ. Sliding window nhìn lại đúng 60 giây nên nhóm thứ hai vẫn thấy 10 request trước đó và bị chặn.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit bảo vệ tốc độ gọi trong cửa sổ ngắn, còn cost guard bảo vệ tổng tiền đã dùng theo user trong cả tháng UTC. Rate limit có thể cho qua request đầu tiên trong phút nhưng cost guard vẫn chặn 402 nếu user đã tiêu gần hết ngân sách và chi phí ước tính làm vượt trần. Ngược lại, user có thể mới tiêu rất ít tiền nhưng gửi request thứ 11 trong 60 giây; khi đó cost guard vẫn cho phép về ngân sách nhưng rate limiter chặn 429.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu endpoint duy nhất kiểm tra Redis, khi Redis mất kết nối thì cả ba container cùng trả 503 cho probe. Orchestrator hiểu nhầm rằng cả ba process đều chết, lần lượt restart chúng; container mới lên vẫn không kết nối được Redis nên lại fail probe và tiếp tục restart, tạo vòng lặp và làm mất toàn bộ capacity trong 30 giây đó. Với hai endpoint, `/health` vẫn 200 vì process còn sống, còn `/ready` trả 503 để load balancer tạm ngừng gửi request. Khi Redis phục hồi, `/ready` tự trở lại 200 mà không cần restart container.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Mỗi lần `/ask` thành công ghi hai message vào Redis nên với cùng `X-User-Id`, `history_length` tăng theo 0, 2, 4, 6... bất kể request rơi vào replica nào. Khi thử scale bằng Compose, tôi còn gặp xung đột vì cấu hình host port `8000:8000` không thể bind cho ba replica; trên cloud/load balancer thì các instance không cần cùng bind một host port như vậy. Nếu dùng dict Python, mỗi replica có lịch sử riêng nên số sẽ tăng thất thường theo replica, ví dụ 0, 0, 2, 0, 2, 4, và restart một replica sẽ làm phần lịch sử của nó mất hẳn.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi kiểm tra deployment Render, lần chạy CP5 đầu tiên báo `httpx.ConnectError: [WinError 10054] An existing connection was forcibly closed by the remote host` tại `/health`. Tôi mở Render Dashboard thấy service vẫn `Live`, sau đó gọi lại `/health` bằng curl và nhận 200; lần chạy test kế tiếp cũng pass. Nguyên nhân là kết nối tạm thời bị reset trong lúc free instance vừa thức dậy hoặc vừa redeploy, không phải lỗi endpoint. Tôi xử lý bằng cách chờ service warm up rồi retry và giữ timeout request đầu tiên dài hơn; kết quả cuối là `/health` 200, `/ready` 200 và `/ask` thiếu key trả 401.
