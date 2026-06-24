# Phòng chống Spam bằng Cloudflare Turnstile

Khi vận hành hệ thống FPTOJ công khai trên Internet, các tài khoản ảo (spambot) có thể tự động đăng ký số lượng lớn làm quá tải hệ thống.

FPTOJ hỗ trợ tích hợp **Cloudflare Turnstile** - giải pháp xác thực người dùng thông minh, bảo mật và thân thiện (thay thế cho Google reCAPTCHA cũ vốn thường gây phiền toái cho người dùng).

---

## 1. Đăng ký nhận Khóa API Cloudflare Turnstile

1. Đăng nhập vào bảng quản trị **Cloudflare Dashboard**.
2. Tìm đến mục **Turnstile** ở menu bên trái.
3. Bấm **Add Site**, nhập tên miền của bạn và chọn loại widget (thường dùng Managed hoặc Non-interactive).
4. Nhận cặp mã khóa:
   - **Sitekey** (Khóa công khai)
   - **Secret key** (Khóa bí mật)

---

## 2. Cài đặt thư viện hỗ trợ Turnstile

Kích hoạt môi trường ảo Python của FPTOJ và chạy lệnh cài đặt:

```bash
cd <thư mục_fptoj_site>
source dmojsite/bin/activate
(dmojsite) pip install django-turnstile
```

---

## 3. Cấu hình hệ thống

Mở file `dmoj/local_settings.py` trong thư mục cấu hình Django của bạn và thêm hoặc cập nhật các dòng sau ở phần cấu hình tùy chỉnh:

```python
# Cấu hình Cloudflare Turnstile
TURNSTILE_SITEKEY = 'your_sitekey_here'
TURNSTILE_SECRET = 'your_secret_key_here'
```

*Lưu ý:* Hãy đảm bảo `'turnstile'` đã được khai báo trong danh sách `INSTALLED_APPS` trong `dmoj/settings.py` (mặc định FPTOJ đã tích hợp sẵn).

---

## 4. Áp dụng thay đổi

Khởi động lại dịch vụ FPTOJ để cấu hình mới có hiệu lực:

```bash
sudo supervisorctl restart site
```

Bây giờ, tại trang đăng ký tài khoản mới của FPTOJ sẽ xuất hiện ô xác thực bảo mật của Cloudflare Turnstile giúp bảo vệ trang web khỏi spambot.
