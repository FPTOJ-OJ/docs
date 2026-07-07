# Cấu hình tính năng xuất đề bài sang PDF (html-to-pdf-flask)

FPTOJ hỗ trợ xuất nội dung đề bài (bao gồm đầy đủ định dạng văn bản và công thức Toán LaTeX) ra định dạng PDF. Điều này cực kỳ hữu ích cho các kỳ thi offline để in ấn đề bài cho học sinh.

Thay vì sử dụng dự án phức tạp `pdfoid` của DMOJ gốc (yêu cầu Chromium driver và Selenium rất nặng nề và dễ lỗi), FPTOJ sử dụng dự án độc quyền gọn nhẹ **[html-to-pdf-flask](https://github.com/FPTOJ-OJ/html-to-pdf-flask)** dựa trên Python Flask và `pdfkit` (sử dụng công cụ `wkhtmltopdf` làm nhân chuyển đổi) để xuất PDF nhanh chóng và chuẩn xác.

---

## 1. Cài đặt và cấu hình dịch vụ trên máy chủ

Thực hiện cài đặt dịch vụ chuyển đổi PDF trên máy chủ:

### Cài đặt công cụ chuyển đổi wkhtmltopdf trên hệ thống:
```bash
sudo apt update
sudo apt install -y wkhtmltopdf
```

### Clone và thiết lập mã nguồn dịch vụ:
```bash
# Clone mã nguồn dịch vụ
git clone https://github.com/FPTOJ-OJ/html-to-pdf-flask.git /home/kien/html-to-pdf-flask
cd /home/kien/html-to-pdf-flask

# Tạo môi trường ảo riêng cho dịch vụ PDF
python3 -m venv env
source env/bin/activate

# Cài đặt các thư viện Python cần thiết và uwsgi
pip install --upgrade pip
pip install -r requirements.txt
pip install uwsgi
```

---

## 2. Kiểm tra dịch vụ hoạt động

Chạy thử ứng dụng trực tiếp từ dòng lệnh:
```bash
python main.py 127.0.0.1 8888
```

Bạn có thể chạy thử lệnh `curl` sau ở một cửa sổ dòng lệnh khác để kiểm tra xem dịch vụ có trả về file PDF đã được mã hóa Base64 hay không:

```bash
curl -d "title=Test&html=<h1>Hello World</h1>" -X POST http://127.0.0.1:8888
```

Dịch vụ hoạt động đúng sẽ trả về phản hồi JSON chứa file PDF dạng base64:
```json
{
    "success": true,
    "pdf": "JVBERi0xLjQKJdPr6eEKMSAwIG9iago8PC9DcmVhdG9yIChDaHJvbWl1bSkK..."
}
```

---

## 3. Cấu hình chạy ngầm tự động bằng Supervisor

Để dịch vụ PDF luôn chạy ngầm và tự khởi động lại cùng hệ thống, tạo file cấu hình cho Supervisor:

Tạo file `/etc/supervisor/conf.d/html-to-pdf-flask.conf` với nội dung sau:

```ini
[program:html-to-pdf-flask]
command=/home/kien/html-to-pdf-flask/env/bin/uwsgi --ini uwsgi.ini
directory=/home/kien/html-to-pdf-flask
stopsignal=QUIT
stdout_logfile=/tmp/html-to-pdf-flask_log.log
stderr_logfile=/tmp/html-to-pdf-flask.log
autorestart=true
stopasgroup=true
killasgroup=true
environment=BASE_PATH="/home/kien/site"
```

Sau đó nạp cấu hình và khởi động dịch vụ:
```bash
sudo supervisorctl update
sudo supervisorctl status html-to-pdf-flask
```

---

## 4. Cấu hình FPTOJ kết nối tới dịch vụ PDF

Để bật tính năng "Tải PDF" trên trang đề bài của FPTOJ, hãy mở file `dmoj/local_settings.py` và thêm hoặc sửa cấu hình sau:

```python
# Đường dẫn API của dịch vụ html-to-pdf-flask
DMOJ_PDF_PDFOID_URL = 'http://localhost:8888'

# Thư mục lưu trữ bộ nhớ đệm (cache) PDF để tối ưu hiệu năng
# Nên chọn thư mục tồn tại lâu dài để tránh render lại nhiều lần
DMOJ_PDF_PROBLEM_CACHE = '/data/pdfcache'

# Path to use for nginx's X-Accel-Redirect feature.
DMOJ_PDF_PROBLEM_INTERNAL = '/pdfcache'
```

Tạo các thư mục bộ nhớ đệm và phân quyền cho web server:
```bash
sudo mkdir -p /data/pdfcache
sudo chmod -R 775 /data/pdfcache
sudo chown -R kien:kien /data/pdfcache
```

Khởi động lại FPTOJ để cấu hình có hiệu lực:
```bash
sudo supervisorctl restart site
```

Bây giờ khi bạn vào trang chi tiết một bài tập bất kỳ, nút **"Xem bản PDF"** sẽ xuất hiện ở thanh bên phải của bài viết.
