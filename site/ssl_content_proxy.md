# SSL Proxy cho Nội dung Người dùng (Github Camo)

Nội dung do người dùng tự tải lên (ví dụ: liên kết ảnh trong bình luận) có thể gây ra cảnh báo bảo mật "Mixed Content" (nội dung hỗn hợp) nếu trang web của bạn chạy HTTPS nhưng ảnh của người dùng lại trỏ tới một liên kết HTTP không bảo mật.

Để khắc phục điều này, FPTOJ hỗ trợ định tuyến và proxy lại toàn bộ hình ảnh của người dùng thông qua dự án **[Github Camo](https://github.com/atmos/camo)** chạy bằng Node.js / CoffeeScript.

> [!WARNING]  
> Việc cài đặt Camo trên cùng một máy chủ vật lý với Site chính có thể làm lộ địa chỉ IP thật của máy chủ (nếu bạn sử dụng dịch vụ ẩn IP như Cloudflare). Đối với các hệ thống lớn, hãy cân nhắc chạy Camo trên một VPS/Server riêng biệt.

---

## 1. Cài đặt Camo

Cài đặt NodeJS/CoffeeScript và tải mã nguồn Camo về máy chủ:

```bash
sudo apt install -y coffeescript
git clone https://github.com/atmos/camo.git /home/<username>/camo
cd /home/<username>/camo
npm install
```

Kiểm tra khởi chạy Camo bằng lệnh:

```bash
PORT="8081" CAMO_KEY="your_camo_secret_key" coffee server.coffee
```
- **PORT**: Cổng dịch vụ Camo sẽ lắng nghe (ví dụ: `8081`).
- **CAMO_KEY**: Khóa HMAC bí mật tự chọn dùng để mã hóa URL ảnh.

---

## 2. Cấu hình chạy ngầm bằng Supervisor

Tạo file cấu hình `/etc/supervisor/conf.d/camo.conf` để tự động chạy Camo:

```ini
[program:camo]
command=coffee server.coffee
directory=/home/<username>/camo
environment=PORT="8081",CAMO_KEY="your_camo_secret_key"
user=<username>
autostart=true
autorestart=true
stdout_logfile=/home/<username>/camo/stdout.log
stderr_logfile=/home/<username>/camo/stderr.log
```

Nạp cấu hình cho Supervisor:
```bash
sudo supervisorctl update
sudo supervisorctl status camo
```

---

## 3. Cấu hình FPTOJ sử dụng Camo

Mở file `dmoj/local_settings.py` trên thư mục Web và cấu hình các biến sau:

```python
# Đường dẫn URL chạy dịch vụ Camo của bạn
DMOJ_CAMO_URL = "https://yourdomain.com:8081"

# Khóa bí mật đã cấu hình ở bước trên
DMOJ_CAMO_KEY = "your_camo_secret_key"

# Các tên miền loại trừ (không cần qua proxy vì đã có HTTPS bảo mật sẵn)
DMOJ_CAMO_EXCLUDE = ("https://fptoj.com", "https://yourdomain.com")

# Bắt buộc Camo dùng giao thức HTTPS
DMOJ_CAMO_HTTPS = True
```

Khởi động lại FPTOJ:
```bash
sudo supervisorctl restart site
```

*(Lưu ý: Sau khi bật, bạn có thể cần xóa bộ nhớ đệm Cache của Django để hệ thống cập nhật lại các ảnh cũ sang link proxy Camo).*
