# Hướng dẫn cài đặt FPTOJ (Phần Site)

Tài liệu này hướng dẫn cách cài đặt giao diện và hệ thống quản lý trung tâm (Site) của FPTOJ. Phù hợp cho các quản trị viên mới bắt đầu (admin non-tech).

---

## 1. Cài đặt các thư viện hệ thống cần thiết

Chạy các lệnh sau để cài đặt công cụ biên dịch, cơ sở dữ liệu, NodeJS (để dịch giao diện) và Redis:

```bash
sudo apt update
sudo apt install -y git gcc g++ make python3-dev python3-pip libxml2-dev libxslt1-dev zlib1g-dev gettext curl redis-server supervisor nginx

# Cài đặt NodeJS 18.x
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Cài đặt công cụ biên dịch CSS/Sass
sudo npm install -g sass postcss-cli postcss autoprefixer
```

---

## 2. Tạo Cơ sở dữ liệu (MariaDB / MySQL)

FPTOJ hoạt động tối ưu nhất trên MariaDB hoặc MySQL.

```bash
# Cài đặt MariaDB Server
sudo apt install -y mariadb-server libmysqlclient-dev

# Khởi động dịch vụ MariaDB
sudo systemctl enable mariadb
sudo systemctl start mariadb

# Cấu hình CSDL
sudo mariadb
```

Trong dấu nhắc lệnh của MariaDB, nhập các lệnh sau (thay thế `<password>` bằng mật khẩu của bạn):

```sql
CREATE DATABASE dmoj DEFAULT CHARACTER SET utf8mb4 DEFAULT COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON dmoj.* TO 'dmoj'@'localhost' IDENTIFIED BY '<password>';
FLUSH PRIVILEGES;
EXIT;
```

Cập nhật múi giờ cho database để các truy vấn thời gian hoạt động đúng:
```bash
mariadb-tzinfo-to-sql /usr/share/zoneinfo | sudo mariadb -u root mysql
```

---

## 3. Cài đặt mã nguồn và môi trường ảo Python

Chúng ta sẽ tạo một môi trường ảo (virtual environment) tên là `dmojsite` để cài đặt các thư viện Python, tránh xung đột hệ thống.

```bash
# Tạo thư mục dự án và tải mã nguồn (ví dụ thư mục /home/<username>/site)
git clone https://github.com/FPTOJ-OJ/online-judge.git site
cd site
git submodule init
git submodule update

# Tạo môi trường ảo
python3 -m venv dmojsite
source dmojsite/bin/activate
```

Khi môi trường ảo hoạt động, bạn sẽ thấy `(dmojsite)` ở đầu dòng lệnh. Từ đây, cài đặt các thư viện Python:

```bash
(dmojsite) pip install --upgrade pip
(dmojsite) pip install -r requirements.txt
(dmojsite) pip install mysqlclient uwsgi
```

---

## 4. Cấu hình file `local_settings.py`

Sao chép file cấu hình mẫu để bắt đầu chỉnh sửa:
```bash
(dmojsite) cp dmoj/local_settings.py.example dmoj/local_settings.py
```
Hoặc bạn có thể sử dụng mẫu cấu hình tối giản đã chuẩn bị sẵn tại:
[Mẫu local_settings.py](../../sample_files/local_settings.py)

Các trường thông tin quan trọng cần cập nhật trong `dmoj/local_settings.py`:
- **DATABASES**: Nhập thông tin kết nối MariaDB (User: `dmoj`, Password đã tạo ở bước 2).
- **CELERY_BROKER_URL** & **CELERY_RESULT_BACKEND**: Bỏ dấu chú thích (`#`) và trỏ về Redis `redis://127.0.0.1:6379/0`.
- **DMOJ_SYSCON_TITLE**: Tên trang web Online Judge của bạn.

---

## 5. Dịch giao diện và biên dịch Assets

```bash
# Biên dịch giao diện CSS (Sass)
(dmojsite) ./make_style.sh

# Thu thập các file tĩnh (CSS/JS/Hình ảnh) vào thư mục static
(dmojsite) python3 manage.py collectstatic --no-input

# Tạo file ngôn ngữ (đa ngôn ngữ)
(dmojsite) python3 manage.py compilemessages
(dmojsite) python3 manage.py compilejsi18n
```

---

## 6. Khởi tạo Cơ sở dữ liệu và Tài khoản Admin

```bash
# Tạo các bảng cơ sở dữ liệu
(dmojsite) python3 manage.py migrate

# Nạp dữ liệu mẫu ban đầu (thanh menu, ngôn ngữ lập trình cơ bản)
(dmojsite) python3 manage.py loaddata navbar
(dmojsite) python3 manage.py loaddata language_small
(dmojsite) python3 manage.py loaddata demo

# Tạo tài khoản quản trị tối cao (Superuser)
(dmojsite) python3 manage.py createsuperuser
```
*(Lưu ý: Fixture `demo` sẽ tạo một tài khoản admin mặc định với thông tin đăng nhập là `admin` / `admin`. Hãy đổi mật khẩu này ngay lập tức trong bảng Admin để bảo mật).*

---

## 7. Cấu hình và chạy các tiến trình nền bằng Supervisord

Để chạy FPTOJ trên môi trường Production, chúng ta sử dụng `uWSGI` chạy web, `supervisord` để quản lý các tiến trình nền tự động khởi chạy khi Server reset.

Sao chép các file cấu hình mẫu từ thư mục `sample_files` vào thư mục cấu hình của hệ thống:

1. **uWSGI**: [uwsgi.ini](../../sample_files/uwsgi.ini) - cấu hình cách chạy Django qua socket.
2. **Supervisor**:
   - Web Site: [site.conf](../../sample_files/site.conf) chuyển vào `/etc/supervisor/conf.d/site.conf`
   - Bridge kết nối Judge: [bridged.conf](../../sample_files/bridged.conf) chuyển vào `/etc/supervisor/conf.d/bridged.conf`
   - Celery xử lý chấm điểm/tác vụ nền: [celery.conf](../../sample_files/celery.conf) chuyển vào `/etc/supervisor/conf.d/celery.conf`
   - Event server cập nhật trực tiếp: [wsevent.conf](../../sample_files/wsevent.conf) chuyển vào `/etc/supervisor/conf.d/wsevent.conf`

Cập nhật đường dẫn thư mục dự án của bạn (ví dụ `<thư mục_fptoj_site>`) trong các file cấu hình này trước khi lưu.

Chạy lệnh sau để tải cấu hình vào Supervisor:
```bash
sudo supervisorctl update
sudo supervisorctl status
```
Tất cả các tiến trình phải có trạng thái `RUNNING`.

---

## 8. Cấu hình Nginx và SSL (HTTPS)

Sao chép file cấu hình mẫu [nginx.conf](../../sample_files/nginx.conf) vào `/etc/nginx/sites-available/fptoj` và tạo link liên kết sang `/etc/nginx/sites-enabled/`.

Thay đổi `server_name` thành tên miền của bạn và cấu hình đường dẫn `root` chỉ tới thư mục chứa code tĩnh của bạn.

Kiểm tra cú pháp Nginx và khởi động lại:
```bash
sudo nginx -t
sudo systemctl restart nginx
```

Chúc mừng! Bạn đã hoàn thành cài đặt FPTOJ Site. Tiếp theo, hãy đọc tài liệu cài đặt Judge và cấu hình dịch vụ xuất PDF độc quyền.
