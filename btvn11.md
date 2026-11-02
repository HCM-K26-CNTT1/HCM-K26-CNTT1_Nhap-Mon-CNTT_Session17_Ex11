# BÀI 11: THIẾT KẾ KIẾN TRÚC TRÌNH DUYỆT VÀ QUY TRÌNH GỠ LỖI F12 CHO PHÒNG KHÁM ẢO

## Phần 1. Thiết kế môi trường trình duyệt

### 1. Tiêu chuẩn trình duyệt cho bác sĩ

**Trình duyệt đề xuất: Google Chrome (Blink Engine).**

Lý do:
- Tương thích tốt với các ứng dụng web hiện đại.
- Hỗ trợ video call và dữ liệu realtime.
- Có công cụ DevTools để kiểm tra và sửa lỗi.

**Thiết lập Profile riêng:**

Tạo một Profile tên `CarePulse_Doctor` để bác sĩ sử dụng riêng cho công việc.

Các bước:
1. Mở Chrome, chọn biểu tượng Profile.
2. Chọn Add Profile.
3. Đặt tên `CarePulse_Doctor`.
4. Chỉ đăng nhập tài khoản bệnh viện trên Profile này.
5. Không dùng Profile này để truy cập các trang cá nhân.

**Lợi ích:** Giúp tách biệt tài khoản, cookie và lịch sử duyệt web, hạn chế rò rỉ dữ liệu phiên khám.

### 2. Chiến lược Cache và Cookie

**a. Browser Cache**

Cache giúp lưu tạm các tài nguyên để trang web tải nhanh hơn.

Các tài nguyên nên lưu Cache:

| Tài nguyên | Lý do |
|---|---|
| Video demo | Ít thay đổi, không cần tải lại nhiều lần |
| Icon nhịp tim | Giúp giao diện tải nhanh |
| Âm thanh còi cấp cứu | Có thể phát nhanh khi cần |
| File CSS | Giúp tải giao diện nhanh |
| File JavaScript | Giảm thời gian tải ứng dụng |

Không nên lưu lâu dữ liệu nhạy cảm như hồ sơ bệnh án, ECG và chỉ số sinh hiệu của bệnh nhân.

**b. Cookie `carepulse_virtual_room`**

Cookie này dùng để quản lý phiên khám. Cần có các cờ bảo mật:

- `Secure`: Chỉ gửi cookie qua HTTPS.
- `HttpOnly`: Ngăn JavaScript đọc cookie.
- `SameSite=Strict`: Hạn chế gửi cookie từ trang web khác.
- `Path=/`: Quy định phạm vi sử dụng cookie.

Ví dụ:

```http
Set-Cookie: carepulse_virtual_room=SESSION_ID; Secure; HttpOnly; SameSite=Strict; Path=/
```

Các cờ này giúp bảo vệ cookie phiên khám. Hệ thống vẫn cần HTTPS và cơ chế mã hóa dữ liệu cuộc gọi.

---

## Phần 2. Quy trình 4 bước gỡ lỗi bằng DevTools (F12)

### 3. Bước 1 — Sửa lỗi nút bấm bằng Inspect Element

**Mục tiêu:** Tìm nguyên nhân nút cấp cứu bị lệch hoặc bị che khuất.

Các bước thực hiện:

1. Mở trang CarePulse trên Chrome.
2. Nhấn `F12` để mở DevTools.
3. Nhấn `Ctrl + Shift + C` để chọn nút cấp cứu.
4. Chọn tab **Computed** để kiểm tra CSS Box Model.

Kiểm tra 4 thành phần:

- `Content`: Phần nội dung bên trong nút.
- `Padding`: Khoảng cách giữa nội dung và đường viền.
- `Border`: Đường viền của nút.
- `Margin`: Khoảng cách bên ngoài nút.

Sau đó, chuyển sang tab **Styles**, tìm `background-color` và sửa thành `#C00000` để kiểm tra màu đỏ của nút.

Nếu nút vẫn bị lệch, kiểm tra thêm chiều rộng, chiều cao và vị trí của nút.

**Kết quả:** Nút cấp cứu hiển thị đúng màu, đúng vị trí và không bị che khuất.

Lưu ý: Sửa trong DevTools chỉ là tạm thời. Muốn áp dụng chính thức phải sửa trong mã nguồn.

### 4. Bước 2 — Kiểm tra tốc độ mạng bằng Network

**Mục tiêu:** Tìm nguyên nhân API lấy dữ liệu sinh hiệu mất 8 giây để phản hồi.

Các bước thực hiện:

1. Nhấn `F12`, chọn tab **Network**.
2. Tích chọn **Disable Cache**.
3. Chọn bộ lọc **Fetch/XHR** để xem các yêu cầu API.
4. Tải lại trang bằng `Ctrl + R`.
5. Tìm API lấy dữ liệu nhịp tim và xem cột **Time**, biểu đồ **Waterfall**.

Cách xác định lỗi:

| Trường hợp | Dấu hiệu | Cách xử lý |
|---|---|---|
| Máy chủ phản hồi chậm | TTFB cao, phải chờ lâu mới nhận được phản hồi | Kiểm tra máy chủ và API |
| Ảnh hồ sơ quá nặng | Thời gian tải ảnh lâu, dung lượng lớn | Nén ảnh và giảm kích thước |

**Kết quả:** Xác định được nguyên nhân khiến dữ liệu sinh hiệu tải chậm.

### 5. Bước 3 — Xử lý mã trạng thái HTTP

Khi gọi API, hệ thống cần kiểm tra mã HTTP để biết yêu cầu thành công hay thất bại.

| Mã HTTP | Ý nghĩa | Cách xử lý |
|---|---|---|
| `200 OK` | Yêu cầu thành công | Hiển thị dữ liệu sinh hiệu |
| `404 Not Found` | Không tìm thấy dữ liệu | Kiểm tra đường dẫn API và thông báo cho người dùng |
| `500 Server Error` | Máy chủ gặp lỗi | Thông báo hệ thống đang gặp sự cố và cho phép thử lại |

**Nguyên tắc:** Không để lỗi API làm ứng dụng bị crash hoặc hiển thị dữ liệu cũ như dữ liệu mới.

### 6. Bước 4 — Xử lý khi mất kết nối mạng

Khi Wi-Fi bị ngắt, ứng dụng cần thông báo rõ ràng thay vì để màn hình trắng.

**a. Kiểm tra trạng thái mạng**

Sử dụng JavaScript để phát hiện khi mất hoặc khôi phục kết nối:

```javascript
window.addEventListener("offline", () => {
    alert("Mất kết nối mạng. Vui lòng kiểm tra Wi-Fi.");
});

window.addEventListener("online", () => {
    alert("Kết nối mạng đã được khôi phục.");
});
```

**b. Thông báo cho bệnh nhân**

Khi mất mạng, hiển thị thông báo:

> Kết nối mạng của bạn đang bị gián đoạn. Vui lòng kiểm tra Wi-Fi hoặc dữ liệu di động. Hệ thống sẽ thử kết nối lại khi mạng được khôi phục. Nếu đang trong tình huống cấp cứu, vui lòng gọi số cấp cứu địa phương hoặc đến cơ sở y tế gần nhất.

**c. Cách xử lý của hệ thống**

1. Phát hiện mất mạng và hiển thị thông báo.
2. Tạm dừng cập nhật dữ liệu realtime.
3. Đánh dấu dữ liệu cũ là không còn cập nhật.
4. Khi có mạng trở lại, kiểm tra kết nối và đồng bộ dữ liệu.
5. Nếu vẫn lỗi, tiếp tục thông báo để người dùng biết tình trạng.

**Kết quả:** Ứng dụng không bị treo hoặc hiện màn hình trắng, bệnh nhân biết cách xử lý khi mất kết nối.

---

## Phần 3. Tổng kết

Qua bài tập, em đã biết cách thiết lập Chrome Profile riêng cho bác sĩ, sử dụng Cache và Cookie để tăng tốc độ và bảo vệ phiên khám.

Em cũng biết sử dụng DevTools để kiểm tra lỗi giao diện, phân tích tốc độ mạng, xử lý mã HTTP và phát hiện tình trạng mất kết nối.

Những kiến thức này giúp ứng dụng CarePulse hoạt động ổn định hơn và bảo vệ dữ liệu bệnh nhân tốt hơn.