# Tải xuống dữ liệu người dùng (GDPR Compliance)

FPTOJ cung cấp tính năng cho phép người dùng tải xuống toàn bộ dữ liệu cá nhân của họ (bao gồm lịch sử bình luận, mã nguồn các bài nộp cũ) để tuân thủ quyền riêng tư dữ liệu.

Mặc định tính năng này bị tắt. Để kích hoạt, bạn thực hiện cấu hình theo các bước sau:

---

## 1. Cấu hình local_settings.py

Mở file `dmoj/local_settings.py` và cấu hình các thông số sau:

```python
import datetime

# Kích hoạt tính năng tải dữ liệu
DMOJ_USER_DATA_DOWNLOAD = True

# Thư mục lưu trữ tạm thời các file zip dữ liệu tải về
# (Quản trị viên cần tự cấu hình dọn dẹp các file cũ)
DMOJ_USER_DATA_CACHE = '/home/<username>/site/userdata/datacache'

# Đường dẫn nội bộ cho tính năng X-Accel-Redirect của Nginx
DMOJ_USER_DATA_INTERNAL = '/datacache'

# Giới hạn tần suất tải dữ liệu của mỗi người dùng (ví dụ: tối đa 1 lần/ngày)
DMOJ_USER_DATA_DOWNLOAD_RATELIMIT = datetime.timedelta(days=1)
```

Tạo thư mục đệm và phân quyền:
```bash
mkdir -p /home/<username>/site/userdata/datacache
chmod -R 775 /home/<username>/site/userdata/datacache
```

---

## 2. Cấu hình Nginx để phục vụ file nhanh hơn

Để Nginx trực tiếp gửi file nén cho người dùng (thay vì chạy qua Django uWSGI chậm chạp), thêm cấu hình sau vào block `server` trong cấu hình Nginx của bạn:

```nginx
# Cấu hình cache phục vụ tải dữ liệu người dùng
location /datacache {
    internal;
    # Trỏ đến thư mục cha của thư mục datacache (không ghi /datacache ở cuối)
    root /home/<username>/site/userdata;
}
```

Kiểm tra và tải lại Nginx:
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 3. Cấu hình tự động dọn dẹp dữ liệu cũ (Cron job)

Hệ thống không tự động xóa các file dữ liệu đã nén sau khi người dùng tải xong. Hãy thiết lập một tác vụ tự động (Cron job) để dọn dẹp các file cũ hơn 2 ngày:

```bash
# Mở bảng cấu hình cron job
crontab -e
```

Thêm dòng sau vào cuối file cấu hình:
```cron
0 */4 * * * find /home/<username>/site/userdata/datacache/ -type f -mtime 2 -delete
```
*(Tác vụ này sẽ quét thư mục cache mỗi 4 giờ một lần và xóa các file có tuổi thọ trên 2 ngày).*

Sau khi hoàn tất cài đặt, người dùng sẽ thấy nút **"Tải dữ liệu của tôi"** xuất hiện trong phần **Chỉnh sửa hồ sơ** cá nhân trên trang web.
