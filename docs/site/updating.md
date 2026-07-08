# Cập nhật hệ thống FPTOJ

FPTOJ được cập nhật liên tục để sửa các lỗi bảo mật, tối ưu hiệu năng và thêm các tính năng mới. Quá trình cập nhật hệ thống rất đơn giản và an toàn nếu bạn tuân thủ các bước dưới đây.

> [!IMPORTANT]  
> Các bản cập nhật lớn có thể thay đổi cấu trúc cơ sở dữ liệu. **Hãy luôn sao lưu (backup) cơ sở dữ liệu MariaDB trước khi thực hiện cập nhật.**

---

## ⚡ Cập nhật nhanh bằng Setup Script (Khuyên dùng)

Nếu bạn đã cài đặt bằng script `setup_fptoj.sh`, bạn có thể chạy lại script này bất kỳ lúc nào để tự động cập nhật hệ thống:

```bash
cd ~/site

# Cập nhật mã nguồn mới nhất từ kho lưu trữ
git pull origin main

# Chạy lại setup script để tự động nâng cấp pip, cơ sở dữ liệu và biên dịch lại giao diện
./setup_fptoj.sh
```
Script sẽ giữ nguyên các cấu hình trước đó của bạn (nạp từ `local_settings.py`) và thực hiện toàn bộ các bước nâng cấp một cách an toàn.

---

## 🛠 Cập nhật thủ công từng bước

Nếu bạn tự quản trị thủ công, hãy kích hoạt môi trường ảo `dmojsite` và thực hiện tuần tự:

### 1. Tải mã nguồn mới nhất
```bash
cd ~/site
source dmojsite/bin/activate

# Lấy mã nguồn mới nhất từ nhánh main
git pull origin main

# Cập nhật các submodules
git submodule update --init --recursive
```

### 2. Cập nhật các thư viện Python
Có thể bản cập nhật mới yêu cầu thêm một số thư viện Python, chạy lệnh sau để cập nhật:

```bash
(dmojsite) pip install -r requirements.txt
```

### 3. Cập nhật cấu trúc Cơ sở dữ liệu (Database Migrations)
Cập nhật cấu trúc bảng dữ liệu để tương thích với mã nguồn mới:

```bash
(dmojsite) python manage.py migrate
(dmojsite) python manage.py check
```

### 4. Biên dịch lại giao diện và file tĩnh (Static Files)
Biên dịch lại mã CSS/JS và nén các file tĩnh:

```bash
# Biên dịch giao diện CSS (Sass)
(dmojsite) ./make_style.sh

# Thu thập file tĩnh mới
(dmojsite) python manage.py collectstatic --no-input

# Biên dịch lại các file ngôn ngữ và JS đa ngôn ngữ
(dmojsite) python manage.py compilemessages
(dmojsite) python manage.py compilejsi18n
```

### 5. Khởi động lại dịch vụ
Khởi động lại các tiến trình thông qua Supervisor để áp dụng mã nguồn mới:

```bash
sudo supervisorctl restart site
sudo supervisorctl restart bridged
sudo supervisorctl restart celery
sudo supervisorctl restart wsevent
```

Hoàn thành! Bạn đã cập nhật FPTOJ lên phiên bản mới nhất thành công.
