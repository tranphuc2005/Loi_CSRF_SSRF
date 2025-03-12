# CSRF & SSRF Training

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

## Cách sử dụng
### Yêu cầu hệ thống
- Python 3.x
- Flask
- Docker (nếu muốn chạy môi trường cách ly)

### Cài đặt
```bash
# Clone repository
git clone https://github.com/your-username/csrf-ssrf-training.git
cd csrf-ssrf-training

# Cài đặt thư viện cần thiết
pip install -r requirements.txt
```

### Chạy chương trình
```bash
python app.py
```
Ứng dụng sẽ chạy tại `http://localhost:5000`

## Thực hành
### 1. Kiểm tra CSRF
- Mở trình duyệt và truy cập `http://localhost:5000/csrf`
- Thử gửi form từ trang độc hại để kiểm tra lỗ hổng.
- Kích hoạt CSRF Token và thử lại.

### 2. Kiểm tra SSRF
- Mở trình duyệt và truy cập `http://localhost:5000/ssrf?url=http://example.com`
- Thử gửi yêu cầu đến IP nội bộ để kiểm tra mức độ bảo vệ của ứng dụng.

## Đóng góp
- Nếu bạn muốn đóng góp, vui lòng tạo một pull request hoặc mở một issue.

## Giấy phép
Dự án này được phát hành theo giấy phép MIT.

