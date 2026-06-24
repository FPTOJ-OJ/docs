# Cấu hình tính năng xuất đề bài sang PDF (html-to-pdf-flask)

FPTOJ hỗ trợ xuất nội dung đề bài (đầy đủ định dạng và công thức Toán LaTeX) ra định dạng PDF. Điều này cực kỳ hữu ích cho các kỳ thi offline để in ấn đề bài cho học sinh.

Thay vì sử dụng dự án phức tạp `pdfoid` của DMOJ gốc (yêu cầu Chromium driver và Selenium rất nặng nề và dễ lỗi), FPTOJ sử dụng dự án độc quyền gọn nhẹ **[html-to-pdf-flask](https://github.com/FPTOJ-OJ/html-to-pdf-flask)** dựa trên Python Flask và Playwright để chuyển đổi nhanh chóng và chính xác.

---

## 1. Cài đặt html-to-pdf-flask

Thực hiện cài đặt dịch vụ chuyển đổi PDF trên máy chủ:

```bash
# Clone mã nguồn dịch vụ
git clone https://github.com/FPTOJ-OJ/html-to-pdf-flask.git /home/<username>/html-to-pdf-flask
cd /home/<username>/html-to-pdf-flask

# Tạo môi trường ảo riêng cho dịch vụ PDF
python3 -m venv env
source env/bin/activate

# Cài đặt các thư viện Python cần thiết
pip install --upgrade pip
pip install -r requirements.txt

# Cài đặt trình duyệt headless Chromium của Playwright
playwright install chromium
playwright install-deps chromium
```

---

## 2. Kiểm tra dịch vụ hoạt động

Kích hoạt dịch vụ chạy thử trên cổng `8887`:

```bash
python app.py
```

Bạn có thể chạy thử lệnh curl sau ở một cửa sổ dòng lệnh khác để kiểm tra xem dịch vụ có trả về file PDF đã được mã hóa Base64 hay không:

```bash
curl -d "title=Test&html=<h1>Hello World</h1>" -X POST http://127.0.0.1:8887
```

Dịch vụ hoạt động đúng sẽ trả về phản hồi JSON có dạng:
```json
{
    "success": true,
    "pdf": "JVBERi0xLjQKJdPr6eEKMSAwIG9iago8PC9DcmVhdG9yIChDaHJvbWl1bSkK..."
}
```

---

## 3. Cấu hình chạy ngầm tự động bằng Supervisor

Để dịch vụ PDF luôn chạy ngầm và tự khởi động khi máy chủ reset, tạo file cấu hình cho Supervisor:

Tạo file `/etc/supervisor/conf.d/pdf_renderer.conf` với nội dung sau:

```ini
[program:pdf_renderer]
command=/home/<username>/html-to-pdf-flask/env/bin/gunicorn --bind 127.0.0.1:8887 --workers 2 --timeout 60 app:app
directory=/home/<username>/html-to-pdf-flask
user=<username>
autostart=true
autorestart=true
stdout_logfile=/home/<username>/html-to-pdf-flask/stdout.log
stderr_logfile=/home/<username>/html-to-pdf-flask/stderr.log
```

Sau đó nạp cấu hình và khởi động dịch vụ:
```bash
sudo supervisorctl update
sudo supervisorctl status pdf_renderer
```

---

## 4. Cấu hình FPTOJ kết nối tới dịch vụ PDF

Để bật tính năng "Tải PDF" trên trang đề bài của FPTOJ, hãy thêm hoặc mở khóa cấu hình sau trong file `dmoj/local_settings.py` của bạn:

```python
# Đường dẫn API của dịch vụ html-to-pdf-flask
DMOJ_PDF_PDFOID_URL = 'http://127.0.0.1:8887'

# Thư mục lưu trữ bộ nhớ đệm (cache) PDF để tối ưu hiệu năng
# Nên chọn thư mục tồn tại lâu dài thay vì /tmp để tránh render lại nhiều lần
DMOJ_PDF_PROBLEM_CACHE = '/home/<username>/site/userdata/pdf_cache'
```

Tạo thư mục bộ nhớ đệm và phân quyền cho web server:
```bash
mkdir -p /home/<username>/site/userdata/pdf_cache
chmod -R 775 /home/<username>/site/userdata/pdf_cache
```

Khởi động lại FPTOJ để cấu hình có hiệu lực:
```bash
sudo supervisorctl restart site
```

Bây giờ khi bạn vào trang chi tiết một bài tập bất kỳ, nút **"Xem bản PDF"** (hoặc **"Tải PDF"**) sẽ xuất hiện ở thanh bên phải của bài viết.
