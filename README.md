# CSRF & SSRF

## Giới thiệu
Đây là khoá học thực hành về **Cross-Site Request Forgery (CSRF)** và **Server-Side Request Forgery (SSRF)**. Mục tiêu của khoá học là giúp người học hiểu rõ về cách thức hoạt động, cách khai thác cũng như cách phòng chống hai loại tấn công này.

## Nội dung
### 1. CSRF (Cross-Site Request Forgery)
- **Mô tả:** CSRF là một loại tấn công ép người dùng thực hiện hành động ngoài ý muốn trên một website mà họ đã đăng nhập.
- **Mô hình tấn công:**
  - Nạn nhân đăng nhập vào trang web hợp lệ.
  - Kẻ tấn công gửi một yêu cầu HTTP độc hại đến trang web từ một trang khác.
  - Trình duyệt của nạn nhân tự động gửi yêu cầu đó với phiên xác thực hợp lệ.
- **Phòng chống:**
  - Sử dụng **CSRF Token**.
  - Kiểm tra **Referer Header**.
  - Sử dụng phương thức **SameSite Cookies**.

### 2. SSRF (Server-Side Request Forgery)
- **Mô tả:** SSRF là một cuộc tấn công mà kẻ tấn công lợi dụng server để gửi yêu cầu đến các tài nguyên nội bộ.
- **Mô hình tấn công:**
  - Kẻ tấn công nhập một URL độc hại vào ứng dụng web.
  - Máy chủ thay mặt kẻ tấn công gửi yêu cầu đến địa chỉ được chỉ định.
  - Kẻ tấn công có thể truy cập tài nguyên nội bộ hoặc thực hiện khai thác khác.
- **Phòng chống:**
  - Xác thực và lọc dữ liệu đầu vào.
  - Chặn truy cập đến IP nội bộ (127.0.0.1, 169.254.x.x, v.v.).
  - Sử dụng danh sách trắng (whitelist) cho các URL bên ngoài.
