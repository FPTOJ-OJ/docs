# Cập nhật hệ thống FPTOJ

FPTOJ được cập nhật liên tục để sửa các lỗi bảo mật, tối ưu hiệu năng và thêm
các tính năng mới. Quá trình cập nhật hệ thống rất đơn giản và an toàn nếu bạn
tuân thủ các bước dưới đây.

---

### Sao lưu trước khi cập nhật

Các bản cập nhật lớn có thể thay đổi cấu trúc cơ sở dữ liệu. Luôn sao lưu
trước khi update:

```bash
# Sao lưu database
mysqldump -u dmoj -p dmoj > ~/fptoj_backup_$(date +%Y%m%d).sql

# Sao lưu thư mục cấu hình (nếu có tùy chỉnh)
cp ~/site/dmoj/local_settings.py ~/local_settings.py.backup
```

---

## ⚡ Cập nhật bằng Setup Script

Phù hợp nếu bạn đã cài đặt lần đầu bằng `setup_fptoj.sh`:

```bash
cd ~/site

# Kéo mã nguồn mới nhất
git pull origin main
git submodule update --init --recursive

# Chạy lại script (tự động nâng cấp pip, migrate DB, biên dịch lại)
./setup_fptoj.sh
```

Script sẽ giữ nguyên `local_settings.py` và tự động thực hiện mọi bước nâng cấp.

---

## 🛠 Cập nhật thủ công

Phù hợp nếu bạn đã cài đặt thủ công (CÁCH 2):

```bash
cd ~/site
source dmojsite/bin/activate

# 1. Kéo mã nguồn mới
git pull origin main
git submodule update --init --recursive

# 2. Cập nhật thư viện Python
pip install -r requirements.txt

# 3. Cập nhật cấu trúc database
python manage.py migrate
python manage.py check

# 4. Biên dịch lại giao diện
./make_style.sh
python manage.py collectstatic --no-input
python manage.py compilemessages
python manage.py compilejsi18n
```

Sau đó khởi động lại dịch vụ:

```bash
sudo supervisorctl restart site
sudo supervisorctl restart bridged
sudo supervisorctl restart celery
sudo supervisorctl restart wsevent
```

---

## 🐳 Cập nhật bằng Docker Compose

Phù hợp nếu bạn đã triển khai bằng Docker Compose (CÁCH 3):

### 1. Pull mã nguồn mới và rebuild image

```bash
cd ~/site
git pull origin main
git submodule update --init --recursive
docker compose -f docker/docker-compose.tier1.yml build --pull
```

> `--pull` đảm bảo luôn lấy base image mới nhất (MariaDB, Redis, Nginx, ...).

### 2. Khởi chạy lại container

```bash
cd ~/site/docker
docker compose -f docker-compose.tier1.yml up -d
```

Docker sẽ tự động restart từng container với image mới. Dữ liệu (DB, problems, PDF cache) được mount từ `DATA_DIR` nên không bị mất.

### 3. Kiểm tra container đã cập nhật

```bash
docker compose -f docker-compose.tier1.yml ps
docker compose -f docker-compose.tier1.yml logs --tail=20 web
```

### 4. Nếu có lỗi migrate database

Trong một số bản cập nhật có thay đổi DB schema, container `web` có thể tự động
chạy migrate. Nếu không, hãy chạy thủ công:

```bash
docker exec -it fptoj-web-tier1 python manage.py migrate
docker exec -it fptoj-web-tier1 python manage.py check
```

### Nâng cấp Judge image

Sau khi cập nhật site, cũng nên rebuild lại judge image nếu có phiên bản judge mới:

```bash
cd ~/judge
git pull origin main
cd .docker
docker build --build-arg TAG="master" -t dmoj/judge-tier1:latest ./tier1

# Restart container judge
docker stop fptoj-judge-tier1-1
docker rm fptoj-judge-tier1-1
cd ~/site/docker
docker compose -f docker-compose.tier1.yml up -d judge-1
```

---

## 🔁 Cập nhật máy chấm (Judge Server độc lập)

Nếu máy chấm chạy riêng (không trong Docker Compose), cập nhật như sau:

### Docker judge

```bash
cd ~/judge
git pull origin main
cd .docker
docker build --build-arg TAG="master" -t dmoj/judge-tier1:latest ./tier1

# Xóa container cũ và chạy lại
docker rm -f fptoj-judge
docker run \
    --name fptoj-judge \
    -v /data/problems:/problems \
    --cap-add=SYS_PTRACE \
    -d \
    --restart=always \
    dmoj/judge-tier1:latest \
    run -p 9999 -c /problems/judge.yml \
    127.0.0.1 judge-01 your_auth_key_here
```

### PyPI judge (pip)

```bash
sudo pip3 install -U dmoj
```

Sau đó khởi động lại tiến trình judge.
