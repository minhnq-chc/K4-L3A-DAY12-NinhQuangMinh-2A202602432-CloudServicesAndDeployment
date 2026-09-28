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

> Nếu để mặc định là "changeme", ứng dụng vẫn khởi động thành công trên server. Tuy nhiên, khi có request thực tế gọi đến LLM, lỗi mới phát sinh do sai API key. Việc không set default value giúp hệ thống "chết ngay từ đầu" (fail-fast), buộc em phải cung cấp đúng biến môi trường trước khi quá trình deploy hoàn tất, ngăn chặn các lỗi ngầm khó phát hiện khi đưa lên production.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi /ask vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà print("đã trả lời xong")
không làm được.

> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T16:00:00Z", "user_id": "sv-test", "cost_usd": 0.0015}
> Hai việc làm được: 1. Có thể đưa log này vào các hệ thống như Elasticsearch/Kibana để tự động tổng hợp chi phí (sum cost_usd) theo từng user_id rất dễ dàng. 2. Lọc chính xác các sự kiện lỗi dựa trên trường "level" hoặc "event" thay vì phải phân tích chuỗi văn bản (regex) phức tạp.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

`ash
docker build -f <Dockerfile-1-stage> -t agent:single .
docker build -t agent:multi .
docker images | grep agent
`

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | ~1000 MB |
| Multi-stage | ~150 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Dung lượng chênh lệch (~850MB) là do các công cụ build như gcc, mã nguồn C, và bộ nhớ đệm (cache) của pip sinh ra trong quá trình cài đặt thư viện. Khi dùng multi-stage, em chỉ copy thư mục chứa app và các thư viện đã biên dịch sang một image mới siêu nhẹ (slim), loại bỏ hoàn toàn các file tạm không cần thiết cho lúc chạy thực tế.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong pp/main.py rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
COPY . . lên trước RUN pip install thì kết quả khác thế nào?

> Lớp cài đặt thư viện (RUN pip install) vẫn được tái sử dụng từ cache, chỉ có lớp COPY . . và các lệnh sau đó mới bị chạy lại. Nếu đặt COPY . . lên trước RUN pip install, mỗi khi sửa đổi dù chỉ một dòng code Python, Docker sẽ vô hiệu hóa cache của pip, khiến em phải đợi tải và cài đặt lại toàn bộ thư viện từ đầu, rất lãng phí thời gian.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh USER cắt đứt chuỗi đó ở chỗ nào.

> Nếu ứng dụng Python có lỗ hổng RCE, kẻ tấn công có thể thực thi mã độc dưới quyền của process hiện tại. Nếu chạy bằng root, chúng sẽ kiểm soát hoàn toàn container, từ đó có thể lợi dụng các kẽ hở (như privileged mode) để chiếm quyền điều khiển máy chủ host. Lệnh USER appuser giới hạn quyền này: kẻ tấn công chỉ có quyền của một user thông thường, không thể can thiệp vào hệ thống lõi.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Có thể gửi tối đa 20 requests. Ví dụ: tại thời điểm 00:59, người dùng gửi 10 requests. Vừa bước sang 01:00, bộ đếm bị reset về 0, họ lập tức gửi tiếp 10 requests nữa. Kết quả là hệ thống phải chịu 20 requests chỉ trong 2 giây. Cơ chế sliding window (cửa sổ trượt) giải quyết triệt để vấn đề này vì khung thời gian 60s liên tục dịch chuyển theo từng request, không bị ranh giới phút cứng nhắc làm sai lệch.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit kiểm soát tần suất gửi yêu cầu (số lần/phút) để chống spam, còn Cost guard kiểm soát ngân sách tiêu thụ (tiền/tháng) để tránh vượt chi phí. Tình huống Rate limit cho qua nhưng Cost guard chặn: Người dùng hỏi 1 prompt cực dài tốn 15 USD, dù chỉ gọi 1 lần (dưới 10 lần/phút) nhưng đã vượt ngân sách 10 USD. Ngược lại, nếu họ spam 100 từ "hello" rất rẻ, ngân sách vẫn còn nhưng thao tác quá nhanh nên Rate limit sẽ chặn.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp chung, khi Redis sập 30s, health check sẽ báo lỗi. Orchestrator (Docker Swarm/Kubernetes) sẽ coi container đó đã "chết" và tiến hành kill/restart cả 3 container. Toàn bộ hệ thống sẽ sập hoàn toàn. Khi tách riêng, readiness check thất bại chỉ báo cho Load Balancer ngừng gửi request mới vào container, ứng dụng vẫn sống và sẽ tự động phục hồi phục vụ ngay khi Redis kết nối lại.

---

### Câu 9 — Stateless (CP4)

Chạy docker compose up --scale agent=3 rồi gọi /ask nhiều lần với cùng một
X-User-Id. Quan sát history_length trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Em nhận thấy chỉ số history_length sẽ thay đổi không nhất quán (ví dụ: 1, 2, 1, 1, 3). Lý do là các request được phân bổ ngẫu nhiên vào 3 container khác nhau, mà biến dict trong RAM của container này lại không chia sẻ với container kia. Khi chuyển sang dùng Redis, trạng thái được lưu trữ tập trung bên ngoài (stateless app), giúp cả 3 container đều đọc chung một lịch sử, do đó số đếm mới tăng đều đặn.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc $PORT...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi deploy lên Render, em gặp lỗi HTTP 404 Not Found khi truy cập vào URL gốc. Qua việc kiểm tra log và mã nguồn, em nhận ra ứng dụng không định nghĩa endpoint cho route "/". Cách khắc phục rất đơn giản: chỉ cần thêm "/health" hoặc "/ready" vào URL trên trình duyệt thì hệ thống trả về đúng file JSON trạng thái như mong muốn.