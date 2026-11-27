# Hướng dẫn cài đặt FPTOJ (Phần Site)

Tài liệu này hướng dẫn cách cài đặt giao diện và hệ thống quản lý trung tâm (Site) của FPTOJ. Phù hợp cho cả quản trị viên mới bắt đầu (admin non-tech) và các kỹ sư hệ thống muốn tùy biến thủ công.

---

### Trước khi bắt đầu: Clone mã nguồn

Tất cả các cách cài đặt bên dưới đều yêu cầu bạn phải có mã nguồn của FPTOJ Site trên máy chủ:

```bash
git clone --recursive https://github.com/FPTOJ-OJ/online-judge.git ~/site
cd ~/site
git submodule init
git submodule update
```

---

## ⚡ CÁCH 1: Cài đặt tự động bằng Script (Khuyến khích - Dễ như ăn bánh)

Hệ thống cung cấp một công cụ cài đặt tự động cực kỳ thông minh nằm ngay tại
`~/site/setup_fptoj.sh`. Script này sẽ:

1. **Kiểm tra phần cứng:** Tự động tối ưu số lượng tiến trình và tài nguyên
   theo CPU/RAM của máy chủ.
2. **Quét dịch vụ hiện có:** Tự động phát hiện xem MySQL, Redis, Nginx, hoặc
   Supervisor đã được cài đặt và đang chạy trên máy Host chưa. Nếu có, hệ thống
   sẽ sử dụng trực tiếp để tránh xung đột; nếu chưa, hệ thống sẽ đề xuất tự
   động cấu hình qua Docker hoặc cài đặt trực tiếp.
3. **Hướng dẫn trực quan:** Tương tác với người dùng qua giao diện dòng lệnh
   để điền các biến cấu hình quan trọng (Site Name, Domain, Email SMTP, Admin
   account) và tự động nạp cấu hình cũ làm mặc định.
4. **Tự động cấu hình & Khởi chạy:** Tự động sinh `local_settings.py`, thiết
   lập Virtualenv, cài đặt dependencies, biên dịch Styles/CSS, dịch ngôn ngữ,
   tạo cấu hình Supervisor & Nginx, và khởi chạy toàn bộ hệ thống.
5. **Cài đặt các dịch vụ độc quyền:** Tự động tải, cấu hình và chạy dịch vụ
   xuất PDF (`html-to-pdf-flask`) và máy chấm (`judge-server` qua Docker).

> **Lưu ý:** Script chỉ mới được kiểm tra và hoạt động ổn định trên **Ubuntu 22.04 / 24.04 / 26.04**. Nếu bạn sử dụng hệ điều hành khác (CentOS, Debian, macOS...), hãy sử dụng phương pháp **Docker Compose (Cách 3)** để triển khai thay vì chạy script trực tiếp.

### Các bước thực hiện:
```bash
# Di chuyển tới thư mục site
cd ~/site

# Chạy script cài đặt tự động (script sẽ tự nâng cấp quyền sudo)
./setup_fptoj.sh
```
Làm theo các hướng dẫn hiện trên màn hình, điền mật khẩu và tài khoản mong muốn. Khi hoàn tất, trang web của bạn sẽ hoạt động ngay lập tức tại cổng 80 (HTTP).

---

## 🛠 CÁCH 2: Cài đặt thủ công từng bước (Dành cho kỹ sư hệ thống)

Nếu bạn muốn tự quản lý cấu trúc hoặc tùy biến sâu, hãy làm theo các bước dưới đây. Hệ thống hiện chạy trên **Python 3.12** và **Django 5.2 LTS**.

### 1. Cài đặt các thư viện hệ thống cần thiết

Chạy lệnh sau để cài đặt các công cụ biên dịch, thư viện hệ thống cho Python và các dịch vụ nền:

```bash
sudo apt update
sudo apt install -y git gcc g++ make python3.12-dev python3-pip python3.12-venv libxml2-dev libxslt1-dev zlib1g-dev gettext curl redis-server supervisor nginx lsof pkg-config default-libmysqlclient-dev
```

**Lưu ý về Node.js & Công thức Toán:**

- **Không cần Node.js** để chạy runtime site hay WebSocket (`wsevent` đã chuyển sang Python thuần `aiohttp` trong `websocket/daemon.py`).
- **Không cần `mathoid` hay `texoid`**: FPTOJ hiển thị công thức toán trực tiếp ở trình duyệt bằng **MathJax / KaTeX** và hình minh họa SVG tĩnh.
- Bạn **chỉ cần Node.js (20 LTS)** nếu muốn tự biên dịch lại file Sass/CSS từ mã nguồn bằng `./make_style.sh`:

```bash
# Chỉ cài nếu cần tự build Sass/CSS bằng make_style.sh
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs
sudo npm install -g sass postcss-cli postcss autoprefixer
```

---

### 2. Thiết lập Cơ sở dữ liệu (MariaDB / MySQL)

FPTOJ hoạt động tối ưu nhất trên MariaDB hoặc MySQL 8.0.

```bash
# Cài đặt MariaDB Server (nếu chưa có trên Host)
sudo apt install -y mariadb-server libmysqlclient-dev

# Khởi động dịch vụ MariaDB
sudo systemctl enable mariadb
sudo systemctl start mariadb

# Cấu hình bảo mật và truy cập CSDL
sudo mariadb
```

Trong dấu nhắc lệnh của MariaDB, nhập các lệnh sau (thay thế `<password>` bằng mật khẩu của bạn):

```sql
CREATE DATABASE dmoj DEFAULT CHARACTER SET utf8mb4 DEFAULT COLLATE utf8mb4_general_ci;
GRANT ALL PRIVILEGES ON dmoj.* TO 'dmoj'@'127.0.0.1' IDENTIFIED BY '<password>';
FLUSH PRIVILEGES;
EXIT;
```

**Quan trọng:** Cập nhật múi giờ cho database để các truy vấn thời gian hoạt động đúng:
```bash
mariadb-tzinfo-to-sql /usr/share/zoneinfo | sudo mariadb -u root mysql
```

---

### 3. Cài đặt mã nguồn và môi trường ảo Python (Python 3.12)

Tạo môi trường ảo `dmojsite` bằng **Python 3.12** để cài đặt các thư viện cho **Django 5.2 LTS** nhằm tránh xung đột hệ thống (`requirements.txt` đã tích hợp `aiohttp` cho WebSocket và loại bỏ `django-fernet-fields`, `jsonfield`):

```bash
# Di chuyển tới thư mục dự án
cd ~/site

# Khởi tạo submodules
git submodule init
git submodule update

# Tạo môi trường ảo với Python 3.12
python3.12 -m venv dmojsite
source dmojsite/bin/activate
```

Khi môi trường ảo được kích hoạt `(dmojsite)`, cài đặt các thư viện Python:

```bash
(dmojsite) pip install --upgrade pip
(dmojsite) pip install -r requirements.txt
(dmojsite) pip install mysqlclient uwsgi pymysql
```

---

### 4. Cấu hình file `local_settings.py`

Sao chép file cấu hình mẫu để chỉnh sửa hoặc tự viết:
```bash
(dmojsite) cp dmoj/local_settings.py.example dmoj/local_settings.py
```

Các trường thông tin quan trọng cần cập nhật trong `dmoj/local_settings.py`:

* **DATABASES:** Điền thông tin kết nối MariaDB (User: `dmoj`, Password đã tạo ở bước 2). Nên giữ `'CONN_MAX_AGE': 60` và `'CONN_HEALTH_CHECKS': True` để tái sử dụng kết nối DB an toàn trên Django 5.2.
* **CELERY_BROKER_URL:** URL Redis cho Celery broker (`redis://127.0.0.1:6379/0`). `result_backend` được suy ra tự động từ cùng URL này trong `dmoj/celery.py` (dòng `app.conf.result_backend = app.conf.broker_url`). **Không** cần `CELERY_RESULT_BACKEND`. Secret override dùng `CELERY_BROKER_URL_SECRET` / `CELERY_RESULT_BACKEND_SECRET`.
* **SITE_NAME:** Tên thương hiệu trang web của bạn (ví dụ: `FPTOJ`).
* **DMOJ_PDF_PDFOID_URL:** Trỏ về dịch vụ PDF độc quyền `http://localhost:8888`.
* **LOGGING:** Trong môi trường production, cấu hình các logger (`console`, `bridge`, `judge.bridge`, `django.request`) ở mức `WARNING`/`ERROR` để tránh ghi log debug quá nhiều.

---

### 5. Dịch giao diện và biên dịch Assets

```bash
# Biên dịch giao diện CSS (Sass)
(dmojsite) ./make_style.sh

# Thu thập các file tĩnh (CSS/JS/Hình ảnh) vào thư mục static
(dmojsite) python manage.py collectstatic --no-input

# Tạo file ngôn ngữ (tiếng Việt / đa ngôn ngữ)
(dmojsite) python manage.py compilemessages
(dmojsite) python manage.py compilejsi18n
```

---

### 6. Khởi tạo Cơ sở dữ liệu và Tài khoản Admin

```bash
# Tạo các bảng cơ sở dữ liệu
(dmojsite) python manage.py migrate

# Nạp dữ liệu cấu hình ban đầu
(dmojsite) python manage.py loaddata navbar
(dmojsite) python manage.py loaddata language_small
(dmojsite) python manage.py loaddata demo

# Tạo tài khoản quản trị tối cao (Superuser)
(dmojsite) python manage.py createsuperuser
```

---

### 7. Cấu hình và chạy các tiến trình nền bằng Supervisord

Đảm bảo file `uwsgi.ini` trong thư mục `~/site/` đã bật các cờ an toàn cho Django & Supervisor (`lazy-apps = true`, `need-app = true`, `die-on-term = true`).

Sao chép hoặc ghi cấu hình dịch vụ vào thư mục `/etc/supervisor/conf.d/`:

* **Web Site (`/etc/supervisor/conf.d/site.conf`):** (thay `/home/kien/site` bằng đường dẫn tuyệt đối thực tế vì `supervisord` không tự mở rộng `~`):

```ini
[program:site]
command=/home/kien/site/dmojsite/bin/uwsgi --ini uwsgi.ini
directory=/home/kien/site
user=kien
group=kien
stopsignal=TERM
stopwaitsecs=30
stopasgroup=true
killasgroup=true
stdout_logfile=/home/kien/site/logs/site.stdout.log
stderr_logfile=/home/kien/site/logs/site.stderr.log
autorestart=true
```

* **Bridge kết nối Judge (`/etc/supervisor/conf.d/bridged.conf`):**

```ini
[program:bridged]
command=/home/kien/site/dmojsite/bin/python manage.py runbridged
directory=/home/kien/site
user=kien
group=kien
stopsignal=INT
stopasgroup=true
killasgroup=true
stdout_logfile=/home/kien/site/logs/bridge.stdout.log
stderr_logfile=/home/kien/site/logs/bridge.stderr.log
autorestart=true
```

* **Event server cập nhật real-time (`/etc/supervisor/conf.d/wsevent.conf`):**

Chạy bằng Python thuần (`aiohttp`) thông qua `websocket/daemon.py`, không cần Node.js:

```ini
[program:wsevent]
command=/home/kien/site/dmojsite/bin/python /home/kien/site/websocket/daemon.py
directory=/home/kien/site
user=kien
group=kien
environment=WS_BIND_HOST="127.0.0.1",WS_GET_PORT="15100",WS_POST_PORT="15101",WS_HTTP_PORT="15102"
stopsignal=TERM
stopasgroup=true
killasgroup=true
stdout_logfile=/home/kien/site/logs/wsevent.stdout.log
stderr_logfile=/home/kien/site/logs/wsevent.stderr.log
autorestart=true
```

Nạp cấu hình và khởi chạy các dịch vụ:
```bash
sudo supervisorctl update
sudo supervisorctl status
```

---

### 8. Cấu hình Nginx và Khởi động

Ghi file cấu hình `/etc/nginx/sites-available/fptoj` (tham khảo mẫu `sample_files/nginx.conf`) và tạo liên kết:
```bash
sudo ln -sf /etc/nginx/sites-available/fptoj /etc/nginx/sites-enabled/fptoj
sudo rm -f /etc/nginx/sites-enabled/default # Xóa site mặc định
sudo systemctl restart nginx
```

#### Tối ưu Nginx cho Cloudflare Origin (Khuyến nghị)
Khi đặt FPTOJ phía sau Cloudflare Proxy, cấu hình Nginx nên bổ sung các tham số sau trong khối `server` (đã có sẵn trong mẫu `nginx.conf`):

* `keepalive_timeout 900s;` và `keepalive_requests 100000;`: Giữ kết nối keep-alive lâu và phục vụ nhiều request trên cùng kết nối với Cloudflare Edge, giảm độ trễ bắt tay TCP/TLS.
* `large_client_header_buffers 4 16k;`: Tránh lỗi `400 Request Header Or Cookie Too Large` khi Cloudflare gửi kèm nhiều header và cookie.
* **Tách `/static/cache/` và `/static/`:**
  * `location /static/cache/`: Chứa file CSS/JS đã được `django-compressor` nén và gắn hash, dùng `expires max;` và `add_header Cache-Control "public, immutable";` để trình duyệt và Cloudflare cache dài hạn.
  * `location /static`: Chứa các file tĩnh thông thường (ảnh, font, file chưa hash), dùng `expires 7d;` và `add_header Cache-Control "public";`.

---

## 🐳 CÁCH 3: Triển khai toàn bộ hệ thống bằng Docker Compose

Phương pháp này phù hợp khi bạn muốn triển khai nhanh mà không cần cài đặt từng
dịch vụ thủ công, hoặc khi bạn dùng hệ điều hành khác ngoài Ubuntu.

Các file cấu hình Docker nằm tại thư mục `docker/` trong mã nguồn Site (`~/site/docker/`).

### Cấu trúc dịch vụ

Hệ thống gồm 8 container chạy đồng thời:

| Container | Chức năng |
|---|---|
| `fptoj-db` | MariaDB 10.11 - Cơ sở dữ liệu |
| `fptoj-redis` | Redis 7 - Cache & hàng đợi Celery |
| `fptoj-web` | uWSGI Django - Web application |
| `fptoj-celery` | Celery worker - Xử lý tác vụ nền |
| `fptoj-wsevent` | Python (aiohttp) - WebSocket event server |
| `fptoj-bridged` | Bridge server - Kết nối máy chấm |
| `fptoj-nginx` | Nginx - Reverse proxy (cổng 80) |
| `fptoj-judge` | Judge server - Máy chấm bài |

### Hồ sơ tài nguyên (Tiers)

Tuỳ theo cấu hình máy chủ, chọn một trong ba hồ sơ sau:

| Hồ sơ | Tài nguyên | uWSGI workers | Celery | Số máy chấm |
|---|---|---|---|---|
| **Tier 1** | Tối thiểu (1-2 CPU, 2-4GB RAM) | 2 | 1 | 1 |
| **Tier 2** | Trung bình (4-8 CPU, 8-16GB RAM) | 4 | 2 | 2 |
| **Tier 3** | Cao cấp (8-16 CPU, 16-32GB RAM) | 8 | 4 | 4 |

### Các bước triển khai

#### Bước 1: Cài đặt Docker & Docker Compose

```bash
sudo apt update
sudo apt install -y docker.io docker-compose-v2
sudo systemctl enable docker
sudo systemctl start docker
```

#### Bước 2: Tạo cấu hình môi trường

```bash
cd ~/site/docker
cp .env.example .env
# Sửa file .env với các thông tin của bạn
nano .env
```

Các biến quan trọng cần cập nhật trong `.env`:

| Biến | Mô tả |
|---|---|
| `SECRET_KEY` | Khóa bảo mật Django (tạo bằng `python3 -c 'from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())'`) |
| `ALLOWED_HOSTS` | Danh sách domain/IP được phép truy cập |
| `MYSQL_ROOT_PASSWORD` | Mật khẩu root MariaDB |
| `MYSQL_PASSWORD` | Mật khẩu user `dmoj` của MariaDB |
| `JUDGE_ID` | Tên máy chấm (mặc định `judge-01`) |
| `JUDGE_KEY` | Khóa bí mật của máy chấm (tạo ngẫu nhiên) |
| `DATA_DIR` | Thư mục dữ liệu trên host (mặc định `/data`) |

#### Bước 3: Xây dựng Docker image cho Site

```bash
cd ~/site
docker compose -f docker/docker-compose.tier1.yml build
```

#### Bước 4: Khởi chạy toàn bộ hệ thống

```bash
cd ~/site/docker
docker compose -f docker-compose.tier1.yml up -d
```

Kiểm tra các container đã chạy:

```bash
docker compose -f docker-compose.tier1.yml ps
```

#### Bước 5: Tạo tài khoản Admin

```bash
docker exec -it fptoj-web-tier1 python manage.py createsuperuser
```

#### Bước 6: Cập nhật cấu hình Judge trên Admin Site

1. Vào trang `/admin/`, mục **Judges**.
2. Bấm **Add judge**, nhập:
   - **Name**: `judge-01` (khớp với `JUDGE_ID` trong `.env`).
   - **Auth key**: Dán khóa đã tạo trong `.env`.
3. Bấm **Save**.

#### Nâng cấp lên hồ sơ cao hơn

Khi máy chủ có nhiều tài nguyên hơn, bạn có thể chuyển sang Tier 2 hoặc Tier 3:

```bash
cd ~/site/docker
docker compose -f docker-compose.tier1.yml down
docker compose -f docker-compose.tier2.yml up -d
```

**Lưu ý:** Thư mục dữ liệu (CSDL, problems, pdfcache) được mount từ host (`DATA_DIR`) nên dữ liệu sẽ được giữ nguyên khi chuyển đổi giữa các Tier.
