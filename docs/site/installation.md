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

> [!WARNING]
> Script chỉ mới được kiểm tra và hoạt động ổn định trên **Ubuntu 20.04/22.04/24.04**.
> Nếu bạn sử dụng hệ điều hành khác (CentOS, Debian, macOS...), hãy cân nhắc sử dụng
> phương pháp **Docker Compose** (CÁCH 3) để triển khai thay vì chạy script trực tiếp.

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

Nếu bạn muốn tự quản lý cấu trúc hoặc tùy biến sâu, hãy làm theo các bước dưới đây:

### 1. Cài đặt các thư viện hệ thống cần thiết

Chạy lệnh sau để cài đặt các công cụ biên dịch, thư viện hệ thống cho Python, công cụ biên dịch giao diện và các dịch vụ nền:

```bash
sudo apt update
sudo apt install -y git gcc g++ make python3-dev python3-pip python3-venv libxml2-dev libxslt1-dev zlib1g-dev gettext curl redis-server supervisor nginx wkhtmltopdf lsof
```

**Cài đặt Node.js và các công cụ CSS/Sass:**
FPTOJ sử dụng Sass và PostCSS để biên dịch giao diện.
```bash
# Cài đặt NodeJS 18.x
curl -sL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Cài đặt các công cụ đóng gói CSS toàn cục
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

### 3. Cài đặt mã nguồn và môi trường ảo Python

Tạo môi trường ảo `dmojsite` để cài đặt các thư viện Python nhằm tránh xung đột hệ thống:

```bash
# Di chuyển tới thư mục dự án
cd ~/site

# Khởi tạo submodules
git submodule init
git submodule update

# Tạo môi trường ảo
python3 -m venv dmojsite
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
* **DATABASES:** Điền thông tin kết nối MariaDB (User: `dmoj`, Password đã tạo ở bước 2).
* **CELERY_BROKER_URL:** URL Redis cho Celery broker (`redis://127.0.0.1:6379/0`). `result_backend` được suy ra tự động từ cùng URL này trong `dmoj/celery.py` (dòng `app.conf.result_backend = app.conf.broker_url`). **Không** cần `CELERY_RESULT_BACKEND`. Secret override dùng `CELERY_BROKER_URL_SECRET` / `CELERY_RESULT_BACKEND_SECRET`.
* **SITE_NAME:** Tên thương hiệu trang web của bạn (ví dụ: `FPTOJ`).
* **DMOJ_PDF_PDFOID_URL:** Trỏ về dịch vụ PDF độc quyền `http://localhost:8888`.

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

Sao chép hoặc ghi cấu hình dịch vụ vào thư mục `/etc/supervisor/conf.d/`:

* **Web Site (`/etc/supervisor/conf.d/site.conf`):**
```ini
[program:site]
command=~/site/dmojsite/bin/uwsgi --ini uwsgi.ini
directory=~/site
stopsignal=QUIT
stdout_logfile=/tmp/site.stdout.log
stderr_logfile=/tmp/site.stderr.log
autorestart=true
```

* **Bridge kết nối Judge (`/etc/supervisor/conf.d/bridged.conf`):**
```ini
[program:bridged]
command=~/site/dmojsite/bin/python manage.py runbridged
directory=~/site
stopsignal=INT
stdout_logfile=/tmp/bridge.stdout.log
stderr_logfile=/tmp/bridge.stderr.log
autorestart=true
```

* **Event server cập nhật real-time (`/etc/supervisor/conf.d/wsevent.conf`):**
```ini
[program:wsevent]
command=/usr/bin/node ~/site/websocket/daemon.js
environment=NODE_PATH="~/site/node_modules"
stdout_logfile=/tmp/wsevent.stdout.log
stderr_logfile=/tmp/wsevent.stderr.log
autorestart=true
```

Nạp cấu hình và khởi chạy các dịch vụ:
```bash
sudo supervisorctl update
sudo supervisorctl status
```

---

### 8. Cấu hình Nginx và Khởi động

Ghi file cấu hình `/etc/nginx/sites-available/fptoj` và tạo liên kết:
```bash
sudo ln -sf /etc/nginx/sites-available/fptoj /etc/nginx/sites-enabled/fptoj
sudo rm -f /etc/nginx/sites-enabled/default # Xóa site mặc định
sudo systemctl restart nginx

---

## 🐳 CÁCH 3: Triển khai toàn bộ hệ thống bằng Docker Compose

Phương pháp này phù hợp khi bạn muốn triển khai nhanh mà không cần cài đặt từng
dịch vụ thủ công, hoặc khi bạn dùng hệ điều hành khác ngoài Ubuntu.

> [!NOTE]
> Các file cấu hình Docker nằm tại thư mục `docker/` trong mã nguồn Site
> (`~/site/docker/`).

### Cấu trúc dịch vụ

Hệ thống gồm 8 container chạy đồng thời:

| Container | Chức năng |
|---|---|
| `fptoj-db` | MariaDB 10.11 - Cơ sở dữ liệu |
| `fptoj-redis` | Redis 7 - Cache & hàng đợi Celery |
| `fptoj-web` | uWSGI Django - Web application |
| `fptoj-celery` | Celery worker - Xử lý tác vụ nền |
| `fptoj-wsevent` | Node.js - WebSocket event server |
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

> [!TIP]
> Thư mục dữ liệu (CSDL, problems, pdfcache) được mount từ host (`DATA_DIR`)
> nên dữ liệu sẽ được giữ nguyên khi chuyển đổi giữa các Tier.
```
