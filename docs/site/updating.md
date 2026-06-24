# Cập nhật hệ thống FPTOJ

FPTOJ được phát triển và cập nhật liên tục các tính năng mới cũng như sửa các lỗi bảo mật. Quá trình cập nhật hệ thống rất đơn giản và an toàn nếu bạn làm theo các bước sau.

> [!IMPORTANT]  
> Các bản cập nhật lớn có thể thay đổi cấu trúc cơ sở dữ liệu. **Hãy luôn sao lưu (backup) cơ sở dữ liệu MariaDB trước khi thực hiện cập nhật.**

---

## Các bước cập nhật hệ thống

Thực hiện các lệnh sau dưới tài khoản quản trị viên và kích hoạt môi trường ảo `dmojsite`:

```bash
# Truy cập thư mục code và kích hoạt môi trường ảo
cd <thư mục_fptoj_site>
source dmojsite/bin/activate

# Tải mã nguồn mới nhất từ kho lưu trữ FPTOJ
# Thay bằng git URL của FPTOJ nếu bạn có cấu hình riêng
git pull origin master
```

### 1. Cập nhật các thư viện Python
Có thể bản cập nhật mới yêu cầu thêm một số thư viện Python, chạy lệnh sau để bổ sung:

```bash
(dmojsite) pip install -r requirements.txt
```

### 2. Cập nhật Cơ sở dữ liệu (Database Migrations)
Cập nhật cấu trúc bảng dữ liệu để tương thích với mã nguồn mới:

```bash
(dmojsite) python3 manage.py migrate
(dmojsite) python3 manage.py check
```

### 3. Biên dịch lại giao diện và file tĩnh (Static Files)
Biên dịch lại mã CSS/JS và nén các file tĩnh:

```bash
# Biên dịch giao diện CSS (Sass)
(dmojsite) ./make_style.sh

# Thu thập file tĩnh mới
(dmojsite) python3 manage.py collectstatic --no-input

# Biên dịch lại các file ngôn ngữ và JS đa ngôn ngữ
(dmojsite) python3 manage.py compilemessages
(dmojsite) python3 manage.py compilejsi18n
```

### 4. Khởi động lại dịch vụ
Khởi động lại các tiến trình thông qua Supervisor để áp dụng mã nguồn mới:

```bash
sudo supervisorctl restart site
sudo supervisorctl restart bridged
sudo supervisorctl restart celery
```

Hoàn thành! Bạn đã cập nhật FPTOJ lên phiên bản mới nhất thành công.
