# Phiếu Phản Ánh — K4 Level 3A, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Trương Việt Anh        Mã học viên: 2A202602444

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Một tình huống cụ thể là lúc deploy lên Render nhưng tôi quên tạo biến
> `AGENT_API_KEY`. Nếu code có khóa mặc định `"changeme"`, service vẫn lên
> trạng thái Live và bất kỳ ai đoán được khóa mặc định đều có thể gọi `/ask`.
> Khi trường này không có giá trị mặc định, Pydantic báo lỗi ngay lúc khởi
> động. Tôi phát hiện cấu hình thiếu trước khi service nhận traffic, thay vì
> chỉ phát hiện sau khi API đã bị sử dụng trái phép.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Một dòng log tôi thu được là:
> `{"event":"ask_completed","level":"info","timestamp":"2026-09-28T09:23:15.151802+00:00","user_id":"sv-test","tokens_in":3,"tokens_out":35,"cost_usd":2.145e-05}`.
> Từ log này, tôi có thể lọc các sự kiện `ask_completed` theo `user_id` để
> điều tra hoạt động của một người dùng. Tôi cũng có thể cộng `cost_usd`,
> `tokens_in` và `tokens_out` để lập thống kê hoặc cảnh báo chi phí. Dòng
> `print("đã trả lời xong")` không có các trường có cấu trúc để máy thực hiện
> hai việc đó.

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
| 1 stage (bản đầu) | 1.73 GB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Kết quả đo thực tế cho thấy image 1-stage có dung lượng 1.73 GB, còn image
> multi-stage chỉ còn 271 MB, giảm khoảng 1.46 GB (xấp xỉ 84%). Phần chênh lệch
> chủ yếu đến từ base image `python:3.11` đầy đủ và các công cụ phục vụ build.
> Bản multi-stage dùng `python:3.11-slim`; stage runtime chỉ nhận dependency đã
> cài cùng mã nguồn cần chạy, không mang toàn bộ môi trường builder sang image
> cuối. Vì vậy image cuối nhỏ hơn đáng kể nhưng vẫn đủ thành phần để chạy ứng
> dụng.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Vì `requirements.txt` được copy và cài trước mã nguồn, sửa một ký tự trong
> `app/main.py` không làm thay đổi layer chứa requirements. Docker có thể dùng
> lại layer copy requirements và layer `pip install`; từ layer copy thư mục
> `app` trở đi phải tạo lại. Nếu đặt `COPY . .` trước `RUN pip install`, mọi
> thay đổi mã nguồn đều làm layer copy đổi, khiến layer cài thư viện phía sau
> cũng mất cache và phải chạy lại dù requirements không đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Một lỗ hổng trong API có thể cho kẻ tấn công thực thi lệnh bên trong
> container. Nếu tiến trình chạy bằng root, mã độc có toàn quyền trong
> container và có thể sửa file hệ thống, đọc secret hoặc lợi dụng thêm lỗi
> cấu hình/runtime để tác động tới host. Root trong container không tự động là
> root trên host, nhưng nó làm hậu quả và khả năng leo thang lớn hơn nhiều.
> Lệnh `USER appuser` cắt chuỗi ở bước thực thi sau khai thác: tiến trình bị
> giới hạn bởi quyền của user thường UID 10001, nên không thể tùy ý sửa các
> tài nguyên chỉ dành cho root.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> Người dùng có thể gửi 20 request trong khoảng 2 giây. Họ gửi 10 request ở
> cuối phút, ví dụ từ `10:00:59`, rồi gửi tiếp 10 request ngay sau khi bộ đếm
> reset ở `10:01:00`. Mỗi phút đồng hồ vẫn chỉ ghi nhận 10 request, nhưng tải
> thực tế là 20 request gần như liên tiếp. Sliding window 60 giây nhìn cả hai
> nhóm trong cùng một cửa sổ nên chặn được cách lách này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> Rate limit giới hạn số request trong một khoảng thời gian ngắn, còn cost
> guard giới hạn tổng tiền của từng người dùng trong cả tháng. Một người dùng
> chỉ gửi hai request nhưng prompt rất lớn hoặc tác vụ rất đắt vẫn còn dưới
> rate limit, trong khi cost guard phải chặn vì ngân sách tháng đã hết. Ngược
> lại, người dùng có thể gửi 11 request rất rẻ trong một phút: tổng chi phí
> vẫn thấp nên cost guard cho qua, nhưng rate limit 10/phút phải chặn request
> thứ 11.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Nếu `/health` cũng kiểm tra Redis, khi Redis mất kết nối thì cả ba container
> cùng trả health check lỗi. Orchestrator hiểu nhầm rằng ba process đều chết
> và lần lượt restart chúng. Container mới vẫn không kết nối được Redis nên
> lại fail health check, tạo thành vòng lặp restart và làm mất toàn bộ năng
> lực phục vụ, kể cả các endpoint không cần Redis. Với thiết kế tách riêng,
> `/health` vẫn trả 200 vì process còn sống, còn `/ready` trả 503 để load
> balancer tạm ngừng gửi request mới; khi Redis phục hồi, các container sẵn
> sàng lại mà không phải restart hàng loạt.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Với Redis dùng chung, mọi instance đọc cùng một key lịch sử nên
> `history_length` trước khi lưu lượt hiện tại tăng đều `0, 2, 4, 6, ...`, dù
> request được chuyển tới container nào. Nếu dùng một dict Python trong từng
> container, mỗi instance có lịch sử riêng. Khi load balancer phân phối
> request, tôi có thể thấy số bị lặp hoặc nhảy không đều như `0, 0, 2, 0, 2`
> tùy container nhận request; một container không biết những lượt đã được xử
> lý bởi container khác.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Lỗi thực tế tôi gặp là khi thử mở Railway, trình duyệt báo
> `ERR_CONNECTION_RESET` và không tải được trang đăng nhập. Tôi xác định đây
> là lỗi kết nối tới platform, không phải lỗi build của ứng dụng, vì service
> còn chưa được tạo và trang lỗi ghi rõ không thể tải `railway.com`. Tôi đổi
> sang Render, dùng Blueprint đọc `render.yaml`, nhập `AGENT_API_KEY` trong
> dashboard và để Blueprint tạo cả `day12-agent` lẫn `day12-redis`. Sau khi
> deploy, tôi kiểm tra URL công khai: `/health` trả 200, `/ready` trả 200 với
> `redis=true`, còn `/ask` không có key trả 401.
