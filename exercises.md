# Phiếu Phản Ánh — K4 Ngày 12

> **Bài làm cá nhân.** Trả lời bằng lời của chính bạn, dựa trên những gì bạn
> quan sát được khi chạy code — không sao chép đáp án của người khác.
>
> Cách trả lời: thay dòng trả lời mẫu bên dưới bằng câu trả lời.
> `grade.py` đếm số câu đã trả lời (15 điểm cho 10 câu).
>
> Họ và tên: Phan Đức Anh  Mã học viên: 2A202601554

---

### Câu 1 — Fail fast (CP1)

Trong `Settings`, `api_token` không có giá trị mặc định nên app chết ngay khi
khởi động nếu thiếu biến môi trường. Hãy mô tả một tình huống cụ thể mà việc
"chết sớm" này cứu bạn, so với việc để mặc định `"changeme"`.

> Nếu quên đặt `API_TOKEN` trên cloud, app sẽ dừng ngay và báo lỗi cấu hình. Nếu dùng token mặc định `"changeme"`, app vẫn public và người lạ có thể đoán token để gọi API.

---

### Câu 2 — Log cho máy đọc (CP1)

Chạy service và gọi `/chat` vài lần. Dán một dòng log JSON bạn thu được, rồi
nêu **hai** việc bạn làm được với dòng log đó mà `print("đã trả lời xong")`
không làm được.

> Ví dụ: `{"event":"chat_completed","severity":"INFO","client_id":"sv01","usd_cost":0.00002}`. Log JSON giúp lọc theo client hoặc mức lỗi và cộng chi phí theo từng request; câu `print` thông thường không có các trường dữ liệu này.

---

### Câu 3 — Kích thước image (CP2)

Build cả hai phiên bản và ghi lại số đo thật:

```bash
docker build -f <Dockerfile-1-stage> -t chat:single .
docker build -t chat:multi .
docker images | grep chat
```

| Bản | Dung lượng |
|-----|-----------|
| 1 stage (bản đầu) | khoảng 1800 MB theo tài liệu lab |
| Multi-stage | 270 MB |

Giải thích: phần dung lượng chênh lệch đó là những gì?

> Phần chênh lệch chủ yếu là base image đầy đủ, công cụ build, cache và file không cần lúc chạy. Multi-stage chỉ đưa dependency và source cần thiết sang runtime image.

---

### Câu 4 — Thứ tự lệnh trong Dockerfile (CP2)

Sửa một ký tự trong `app/main.py` rồi build lại. Với Dockerfile của bạn, những
layer nào được dùng lại từ cache, layer nào phải chạy lại? Nếu bạn đặt
`COPY . .` lên trước `RUN pip install` thì kết quả khác thế nào?

> Khi chỉ sửa source, layer base, `requirements.txt` và `pip install` được dùng lại; layer copy source trở đi phải chạy lại. Nếu `COPY . .` đứng trước `pip install`, mỗi lần sửa code đều làm pip cài lại thư viện.

---

### Câu 5 — Vì sao không chạy bằng root (CP2)

Container mặc định chạy bằng root. Mô tả chuỗi sự kiện dẫn từ "một lỗ hổng
trong code Python của bạn" tới "kẻ tấn công có quyền cao trên máy host", và
lệnh `USER` cắt đứt chuỗi đó ở chỗ nào.

> Lỗ hổng Python có thể cho kẻ tấn công chạy lệnh trong container. Nếu process chạy root, họ có quyền rất cao và hậu quả sẽ lớn hơn nếu tiếp tục khai thác container runtime; `USER appuser` giới hạn họ ở quyền user thường.

---

### Câu 6 — Bearer token (CP3)

Vì sao 401 phải kèm header `WWW-Authenticate: Bearer`? Và vì sao ta trả **cùng
một** thông báo lỗi cho cả ba trường hợp (thiếu header, sai scheme, sai token)
thay vì nói rõ sai ở đâu cho người dùng dễ sửa?

> `WWW-Authenticate: Bearer` cho client biết cơ chế xác thực được yêu cầu theo chuẩn HTTP. Dùng cùng một thông báo cho mọi lỗi giúp tránh tiết lộ manh mối về scheme hoặc token cho người đang dò.

---

### Câu 7 — Token bucket (CP3)

Với `capacity=10`, `refill_per_minute=10`: một client im lặng 10 phút rồi gửi
liên tiếp. Nó gửi được bao nhiêu request trước khi bị 429? Nếu bỏ đoạn
`min(capacity, ...)` trong `available()` thì con số đó thành bao nhiêu, và tại sao?

> Xô bị giới hạn ở 10 token nên client chỉ gửi liên tiếp được 10 request, request thứ 11 nhận 429. Nếu bỏ `min`, sau 10 phút xô có thể tích khoảng 100–110 token tùy trạng thái ban đầu vì token không còn bị chặn ở capacity.

---

### Câu 8 — Ngân sách theo ngày (CP3)

So sánh hạn mức $30/tháng với hạn mức $1/ngày cho cùng một client. Giả sử có sự
cố khiến một client gọi liên tục từ 2h sáng. Với mỗi cách, thiệt hại tối đa là
bao nhiêu và service tự hồi phục khi nào?

> Hạn mức $30/tháng cho phép một sự cố đốt tối đa $30 và chỉ tự phục hồi ở tháng sau. Hạn mức $1/ngày giới hạn thiệt hại khoảng $1 và tự mở lại vào ngày UTC tiếp theo.

---

### Câu 9 — /healthz khác /readyz (CP4)

Nếu gộp hai endpoint làm một và cho nó kiểm tra Redis, chuyện gì xảy ra với cụm
3 container khi Redis mất kết nối 30 giây? Trả lời theo đúng thứ tự sự kiện.

> Redis mất kết nối làm endpoint chung trả 503 ở cả ba container, khiến orchestrator restart cả cụm. Container mới vẫn lỗi và tiếp tục bị restart; tách `/healthz` và `/readyz` giúp process sống nguyên, chỉ tạm ngừng nhận traffic đến khi Redis hồi phục.

---

### Câu 10 — Deploy thật (CP5)

Ghi lại **một** lỗi bạn gặp khi deploy lên cloud (build fail, health check
timeout, sai REDIS_URL, app không đọc `$PORT`...): thông báo lỗi là gì, bạn
tìm ra nguyên nhân bằng cách nào, và sửa ra sao?

> Khi mở URL gốc trên Render, tôi nhận `{"detail":"Not Found"}` và tưởng deploy lỗi. Tôi kiểm tra `/healthz` và `/readyz` đều trả 200 nên xác định app hoạt động, chỉ là chưa khai báo route `/`; tôi dùng đúng các endpoint của lab thay vì URL gốc.
