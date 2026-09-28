# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng > *Câu trả lời của bạn* bằng câu trả lời.
> grade.py đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Ninh Quang Minh  Mã học viên: 2A202602432

---

### Câu 1 — Fail fast (CP1)

Trong Settings, gent_api_key không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định "changeme".

> Nếu để mặc định 'changeme', app vẫn chạy bình thường nhưng khi user gọi API, nó sẽ thất bại do sai key LLM. Điều này làm ta tưởng app đã deploy thành công nhưng thực ra đang lỗi ngầm. Fail-fast giúp phát hiện lỗi cấu hình ngay từ vòng build/deploy CI/CD, không cho phép bản lỗi lên production.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi /ask vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà print("đã trả lời xong")
không làm được.

> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T09:00:00Z", "user_id": "sv-test", "cost_usd": 0.001}
> Hai việc làm được: 1. Có thể dùng tool (Datadog/Elasticsearch) để query và tính tổng tiền (sum cost_usd) theo user_id. 2. Có thể lọc theo event name hoặc mốc thời gian một cách chính xác qua key-value thay vì dùng Regex quét text.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ~1000 MB |
| Multi-stage | ~150 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần chênh lệch là các công cụ build (như gcc), mã nguồn thư viện C gốc, cache của pip và các dependencies trung gian không cần thiết lúc chạy app thực tế.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong pp/main.py rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
COPY . . lên trước RUN pip install thì kết quả khác thế nào?

> Lớp cài đặt 'pip install' được lấy từ cache, chỉ lớp 'COPY . .' và sau đó bị chạy lại. Nếu đảo ngược, mỗi lần sửa code python, Docker sẽ vứt bỏ cache của pip install và tải lại toàn bộ thư viện từ đầu, làm thời gian build tốn thêm rất nhiều phút.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh USER cắt đứt chuỗi đó ở chỗ nào.

> Nếu app bị dính lỗi RCE (chạy mã độc từ xa), kẻ tấn công chiếm được quyền root trong container. Từ đó, nếu container cấu hình sai (được mount volume nhạy cảm hoặc chạy privileged), hacker có thể thoát ra ngoài và kiểm soát host. Lệnh USER appuser giới hạn quyền kẻ tấn công chỉ ở mức user thường, không thể chỉnh sửa file hệ thống hay tận dụng lỗ hổng leo thang đặc quyền.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi tối đa 20 requests. Bằng cách gửi 10 request lúc 00:59, hệ thống cho qua. Qua 01:00, biến đếm reset về 0, họ gửi tiếp 10 request nữa. Tổng cộng spam 20 requests chỉ trong 2 giây. Sliding window ngăn chặn triệt để điều này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit đếm số lần gọi (tần suất), cost guard đếm số tiền (chi phí). Tình huống cost chặn nhưng rate qua: User gọi 1 request siêu dài tốn nhiều tiền, chưa quá 10 lần/phút (rate limit ok) nhưng vượt tổng tiền ngân sách 10 USD (cost guard chặn). Tình huống rate chặn nhưng cost qua: User spam 20 câu hỏi cực ngắn, tiền còn nhiều (cost ok) nhưng gọi quá nhanh (rate chặn).

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thực tế sự kiện.

> Khi Redis sập 30s, health probe của cả 3 container báo lỗi. Orchestrator (K8s/Docker) tưởng 3 container này bị treo nên sẽ giết (kill/restart) cả cụm. Hậu quả là app sập toàn tập và phải khởi động lại. Nếu tách riêng, ready báo lỗi chỉ làm Load balancer ngừng gửi request vào, container vẫn sống chờ Redis hồi phục.

---

### Câu 9 — Stateless (CP4)

Chạy docker compose up --scale agent=3 rồi gọi /ask nhiều lần với cùng một
X-User-Id. Quan sát history_length. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Con số sẽ nhảy loạn xạ (ví dụ 1, 2, 1, 1, 3) vì request phân tán ngẫu nhiên vào 3 container khác nhau. Container A không biết dữ liệu dict trên RAM của Container B. Dùng Redis giúp dữ liệu tập trung, history_length sẽ tăng đều đặn 1, 2, 3.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc $PORT...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi HTTP 404 khi vào trang chủ / trên Render. Tìm ra nguyên nhân do code không định nghĩa endpoint root @app.get('/'). Khắc phục bằng cách truy cập đúng đường dẫn /health hoặc /ready để xem kết quả.
