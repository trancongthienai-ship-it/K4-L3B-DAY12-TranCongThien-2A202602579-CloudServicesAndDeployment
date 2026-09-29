# Phiếu Phản Ánh — K4 Level 3B, Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng `> *Câu trả lời của bạn*` bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: ..........................  Mã học viên: ..........................

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `agent_api_key` không có giá trị mặc định nên app chết ngay
khi khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà
việc "chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu để mặc định là `"changeme"`, ứng dụng sẽ âm thầm khởi động và chạy bình thường mà bạn không hề hay biết mình đang dùng một API key yếu. Kẻ xấu có thể dễ dàng đoán ra `"changeme"`, gọi API của bạn và đốt sạch hạn mức tiền của bạn ở OpenAI. Việc "chết sớm" (fail-fast) ngay lập tức ép bạn phải khai báo key đàng hoàng trước khi ứng dụng kịp lên sóng, ngăn chặn nguy cơ bảo mật từ trong trứng nước.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/ask` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Dòng log JSON: `{"level": "INFO", "event": "ask_completed", "user_id": "sv-test", "tokens_in": 3, "tokens_out": 35, "cost_usd": 2.145e-05, "timestamp": "2026-09-29T04:11:18Z"}`
> Hai việc làm được:
> 1. Đẩy log vào các hệ thống quản lý như ELK, Datadog để tự động vẽ biểu đồ thống kê tổng số tiền (cost_usd) đã tiêu trong ngày.
> 2. Lọc và tìm kiếm dễ dàng theo cấu trúc (ví dụ: truy vấn tất cả các request có `user_id` là "sv-test" để phân tích).

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
| 1 stage (bản đầu) | ~1000 MB |
| Multi-stage | 271 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần dung lượng chênh lệch (giảm đi hơn 700MB) bao gồm các công cụ biên dịch (C++ compiler, các gói build-essential), các file bộ nhớ đệm (cache/tạm) sinh ra trong quá trình chạy `pip install`, và nhân hệ điều hành cồng kềnh của base image `python:3.11`. 
> Bằng cách sử dụng Multi-stage, chúng ta chỉ lấy đúng KẾT QUẢ sau khi cài đặt (các thư viện đã build xong) copy sang một image hoàn toàn mới và cực kỳ tinh gọn (`python:3.11-slim`), nên vứt bỏ được toàn bộ phần rác và các công cụ thừa thải không cần thiết cho lúc chạy thực tế (runtime).

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> - Khi sửa một ký tự trong `app/main.py` rồi build lại, các layer ở trên (cài hệ điều hành, copy requirements.txt, chạy `pip install`) sẽ **được dùng lại từ cache**. Lệnh `COPY . .` và các lệnh theo sau (như `CMD`) sẽ **phải chạy lại**.
> - Nếu đặt `COPY . .` lên trước `RUN pip install`, chỉ cần sửa một dòng code nhỏ trong `main.py`, lệnh `COPY . .` sẽ làm vỡ cache ở đó, kéo theo lệnh `RUN pip install` tốn thời gian cực kỳ lãng phí cũng phải chạy lại toàn bộ dù danh sách thư viện không hề thay đổi.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> - Lỗ hổng trong code Python (ví dụ: lỗi cho phép người dùng truyền tham số độc để thực thi shell) -> Kẻ tấn công chạy được lệnh bash bằng quyền của tiến trình Python (quyền root) -> Chúng dùng quyền root đó chỉnh sửa file hệ thống trong container, tải mã độc, cài backdoor, thậm chí leo thang khai thác ra ngoài hệ điều hành máy host.
> - Lệnh `USER appuser` cắt đứt chuỗi này ngay tại lúc tiến trình Python chạy: dù kẻ tấn công có chiếm được tiến trình, chúng cũng chỉ có quyền của một user cấp thấp, không thể động vào các file hệ thống hay điều khiển server sâu hơn.

---

### Câu 6 — Cửa sổ trượt (CP3)

Rate limit của bạn dùng sliding window 60 giây. Nếu thay bằng cách đếm theo
phút đồng hồ (reset lúc giây 00), một người dùng có thể gửi tối đa bao nhiêu
request trong 2 giây liên tiếp khi hạn mức là 10/phút? Giải thích cách đạt được
con số đó.

> - Có thể gửi tối đa 20 request trong 2 giây liên tiếp.
> - Giải thích: Họ gửi 10 request lúc 00:59 (thuộc về phút trước). Vừa bước sang giây 00:00 của phút tiếp theo, bộ đếm (đếm theo phút đồng hồ) lập tức bị reset về 0. Họ liền gửi bồi thêm 10 request nữa lúc 00:00. Tổng cộng chỉ trong 2 giây (00:59 - 00:00) họ đã ném vào hệ thống 20 request, lách luật thành công (bursting). Dùng Sliding Window (cửa sổ trượt) sẽ dập tắt được mánh khóe này.

---

### Câu 7 — Rate limit và cost guard (CP3)

Hai cơ chế này khác nhau ở điểm nào? Cho một tình huống mà rate limit cho qua
nhưng cost guard phải chặn, và một tình huống ngược lại.

> - **Sự khác nhau:** Rate Limit giới hạn **tần suất (số lượng) request** trong một khoảng thời gian ngắn (ví dụ: mỗi phút) để chống spam, chống nghẽn server. Cost Guard giới hạn **tổng chi phí (số tiền/token)** trong một khoảng thời gian dài (theo tháng) để bảo vệ túi tiền của bạn.
> - **Tình huống Rate Limit cho qua nhưng Cost Guard chặn:** Bạn gửi duy nhất 1 request trong 1 phút (Rate limit cho qua), nhưng request đó chứa một câu hỏi siêu dài tốn tận 11 đô (vượt quá ngân sách 10 đô/tháng). Cost Guard sẽ chặn không cho gọi LLM.
> - **Tình huống ngược lại (Rate Limit chặn, Cost Guard cho qua):** Đầu tháng bạn chưa tiêu đồng nào (Cost Guard cho qua), nhưng bạn lỡ tay cho chạy vòng lặp gửi tới 100 request trong cùng 1 giây. Rate Limit sẽ chặn ngay lập tức để bảo vệ server khỏi bị quá tải.

---

### Câu 8 — /health khác /ready (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> - 1. Khi Redis mất kết nối 30 giây, hàm ping tới Redis sẽ báo lỗi.
> - 2. Do bị gộp chung, `/health` của cả 3 container sẽ đồng loạt trả về lỗi 503.
> - 3. Orchestrator (K8s, Railway...) gọi `/health` thấy lỗi liền nghĩ rằng tiến trình Python đã bị "treo/đơ", nên nó nhẫn tâm gửi tín hiệu SIGKILL để khởi động lại (restart) cả 3 container cùng lúc.
> - 4. Kết quả: Toàn bộ hệ thống bị sập (downtime). Người dùng đang sử dụng bị văng lỗi. Nếu tách riêng `/ready`, hệ thống chỉ tạm dừng nhận khách mới chứ không tự giết chết chính mình.

---

### Câu 9 — Stateless (CP4)

Chạy `docker compose up --scale agent=3` rồi gọi `/ask` nhiều lần với cùng một
`X-User-Id`. Quan sát `history_length` trong response. Nếu lịch sử được lưu
trong một dict Python thay vì Redis, bạn sẽ thấy con số đó thay đổi thế nào?

> Nếu lưu bằng dict Python (trên RAM), 3 container sẽ có 3 vùng nhớ riêng rẽ. Khi bạn gọi `/ask` liên tục, Load Balancer sẽ đẩy request xoay vòng ngẫu nhiên vào máy A, máy B, hoặc máy C.
> 
> Hậu quả là `history_length` sẽ nhảy lung tung rải rác (ví dụ: request 1 nhảy vào máy A trả về length 1, request 2 nhảy vào máy B trả về length 1, request 3 lại quay về máy A thì length mới lên 2). Kéo theo đó, AI sẽ bị "mất trí nhớ" hoặc bị chập cheng vì không đọc được toàn bộ ngữ cảnh trước đó. Lưu tập trung bằng Redis sẽ giải quyết hoàn toàn việc này.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Bị lỗi 500 Internal Server Error khi gọi endpoint `/ready` sau khi deploy lên Railway.
> - **Tìm ra nguyên nhân:** Mình quan sát thấy API trả về mã HTTP 500 thay vì 503 (báo Redis sập). Chớp lấy manh mối đó, mình nhận ra app đã bị crash ngay từ khâu cấu hình URL Redis ban đầu (hàm `redis.from_url` không đọc được chuỗi cấu hình). 
> - **Sửa lỗi:** Lỗi do khi thêm biến môi trường trên Railway, mình đã tạo nhầm một chuỗi cấu hình có dư tận 2 cặp ngoặc `${{${{day12-redis.REDIS_URL}}`. Mình đã lên trang Railway, sửa biến thành đúng `${{day12-redis.REDIS_URL}}`. Kết quả /ready lập tức trả về 200 {"status":"ready","redis":true}.
