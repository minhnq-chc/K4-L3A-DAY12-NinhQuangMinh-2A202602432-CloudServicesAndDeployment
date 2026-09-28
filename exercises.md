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

> Nếu mình để nguyên "changeme", lúc app chạy lên sẽ không báo lỗi gì cả, cứ tưởng ngon. Nhưng đến lúc user gọi API thật, nó mới quăng lỗi vì sai API key. Nhờ set không có default value, app sẽ chết ngay lúc vừa khởi động (fail-fast), giúp mình biết ngay là quên set biến môi trường, khỏi mất công loay hoay debug trên production.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi /ask vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà print("đã trả lời xong")
không làm được.

> {"event": "ask_completed", "level": "info", "timestamp": "2026-09-28T16:00:00Z", "user_id": "sv-test", "cost_usd": 0.0015}
> Hai việc làm được: 1. Có thể vứt đống log này vào Kibana/Datadog để tự động sum tiền (cost_usd) theo từng user_id cực nhanh. 2. Filter ra các event lỗi một cách chuẩn xác nhờ key-value thay vì ngồi viết Regex tìm text thủ công.

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

> Phần 850MB chênh lệch toàn là mấy cái tool build code, gcc, rồi đống cache pip sinh ra lúc tải thư viện. Khi dùng multi-stage, mình chỉ bốc đúng app và thư viện đã cài xong ném sang một cái image nhẹ (slim) để chạy thôi, vứt hết đống rác lúc build đi.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong pp/main.py rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
COPY . . lên trước RUN pip install thì kết quả khác thế nào?

> Layer cài thư viện (pip install) vẫn được dùng lại từ cache, chỉ có layer COPY . . và mấy cái lệnh sau nó bị chạy lại. Nếu lỡ dại đặt COPY . . lên trước RUN pip install, thì cứ mỗi lần sửa 1 dấu phẩy trong python, Docker sẽ vứt luôn cache pip, bắt ngồi chờ tải lại thư viện từ đầu rất ức chế.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh USER cắt đứt chuỗi đó ở chỗ nào.

> Nếu dính lỗi RCE trong python app, hacker sẽ chạy code dưới quyền của app. Nếu để mặc định là root, hacker sẽ thành root của container đó. Lỡ như container chạy kiểu privileged hoặc mount file hệ thống vào, nó có thể từ đó hack luôn máy chủ host. Đổi sang USER appuser chặn ngay từ đầu: hacker vô được cũng chỉ là một user cùi, không thể quậy phá OS.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Sẽ có lúc họ gửi được tối đa 20 requests. Ví dụ 00:59 phút họ bắn liền 10 request. Sang đúng 01:00 phút biến đếm đếm lại từ đầu, họ lập tức bắn tiếp 10 request nữa. Kết quả là 20 request đập vào hệ thống chỉ trong vỏn vẹn 2 giây. Dùng sliding window thì cửa sổ 60s trượt liên tục, không bao giờ có kẽ hở giao thời kiểu này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit là để chặn spam phá hoại (đếm số lần/phút), cost guard là để chặn phá sản (đếm tiền/tháng). Rate limit qua nhưng cost chặn: User hỏi 1 câu siêu dài tốn tận 15 USD, mới gọi 1 lần (rate limit ok) nhưng vượt ngân sách 10 USD (cost chặn). Ngược lại: User spam 100 câu chữ "hi" rẻ bèo, tiền chưa hết (cost ok) nhưng gọi quá nhanh (rate limit chặn ngay).

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu gộp chung, khi Redis sập 30s, health check sẽ báo lỗi (do check redis). Khi đó Orchestrator (Docker/K8s) tưởng app mình bị treo cứng nên sẽ rút ống thở, kill cả 3 container rồi khởi động lại. Cả hệ thống sập toàn tập. Tách ra thì khi Redis chết, readiness chỉ làm app ngừng nhận traffic mới chứ không giết app, khi Redis sống lại app tự phục hồi.

---

### Câu 9 — Stateless (CP4)

Chạy docker compose up --scale agent=3 rồi gọi /ask nhiều lần với cùng một
X-User-Id. Quan sát history_length trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Em test thử thấy số history_length nó cứ nhảy loạn cào cào (ví dụ 1, 2, 1, 1, 3) vì request bị ném ngẫu nhiên cho 3 container, mà con A thì mù tịt dict trên RAM của con B. Khi dùng Redis thì dữ liệu được đẩy ra ngoài lưu tập trung, nên cả 3 con đều đọc chung một chỗ, số đếm mới tăng đều đặn chuẩn xác 1, 2, 3 được.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc $PORT...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lúc vứt lên Render em bị lỗi HTTP 404 Not Found ngay khi mở cái URL trang chủ của nó. Sau đó check log và nhớ lại mới nhận ra là nãy giờ code mình làm gì có viết endpoint nào ở đường dẫn gốc. Sửa bằng cách đơn giản là gõ thêm /health hoặc /ready vào đuôi URL là nó trả về JSON ngon lành cành đào.