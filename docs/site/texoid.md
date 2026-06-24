# Hiển thị Hình vẽ & Sơ đồ bằng LaTeX (Texoid)

FPTOJ có khả năng biên dịch và hiển thị trực tiếp các hình vẽ, đồ thị, sơ đồ được vẽ bằng mã LaTeX (ví dụ: TikZ, standalone) lên trang mô tả đề bài. Điều này giúp giáo viên dễ dàng tạo ra các sơ đồ mạng, hình vẽ hình học trực quan mà không cần xuất thành ảnh thủ công.

Hệ thống hỗ trợ tính năng này thông qua dự án **[Texoid](https://github.com/DMOJ/texoid)** kết nối trực tiếp với công cụ cài đặt LaTeX của Server.

---

## 1. Cài đặt các công cụ phụ thuộc trên máy chủ

Texoid sử dụng bộ thư viện biên dịch LaTeX thành định dạng DVI, chuyển đổi sang SVG (`dvisvgm`) và nén thành PNG (`ImageMagick`).

```bash
sudo apt update
sudo apt install -y texlive-latex-base texlive-binaries imagemagick
```

---

## 2. Cài đặt và Chạy dịch vụ Texoid

1. Clone mã nguồn Texoid và cài đặt trong môi trường ảo:
   ```bash
   git clone https://github.com/DMOJ/texoid.git /home/<username>/texoid
   cd /home/<username>/texoid
   python3 -m venv env
   source env/bin/activate
   pip install -e .
   ```

2. Kiểm tra chạy dịch vụ trên cổng `8886`:
   ```bash
   env/bin/texoid --port=8886
   ```

---

## 3. Cấu hình chạy ngầm bằng Supervisor

Tạo file cấu hình `/etc/supervisor/conf.d/texoid.conf`:

```ini
[program:texoid]
command=/home/<username>/texoid/env/bin/texoid --port=8886
directory=/home/<username>/texoid
user=<username>
autostart=true
autorestart=true
stdout_logfile=/home/<username>/texoid/stdout.log
stderr_logfile=/home/<username>/texoid/stderr.log
```

Nạp cấu hình cho Supervisor:
```bash
sudo supervisorctl update
sudo supervisorctl status texoid
```

---

## 4. Cấu hình FPTOJ kết nối tới Texoid

Mở file `dmoj/local_settings.py` và cấu hình các trường sau:

```python
# Đường dẫn API của dịch vụ Texoid
TEXOID_URL = 'http://127.0.0.1:8886'

# Thư mục lưu trữ hình ảnh sơ đồ đã render (nên chọn thư mục cố định)
TEXOID_CACHE_ROOT = '/home/<username>/site/userdata/texoid_cache'

# URL tương ứng để truy cập thư mục cache từ trình duyệt
TEXOID_CACHE_URL = '/texoid/'
```

Tạo thư mục đệm và phân quyền:
```bash
mkdir -p /home/<username>/site/userdata/texoid_cache
chmod -R 775 /home/<username>/site/userdata/texoid_cache
```

Cấu hình thêm alias trong file cấu hình Nginx để phục vụ file ảnh tĩnh:
```nginx
location /texoid/ {
    alias /home/<username>/site/userdata/texoid_cache/;
}
```

Khởi động lại Nginx và FPTOJ:
```bash
sudo systemctl reload nginx
sudo supervisorctl restart site
```

---

## 5. Cách sử dụng sơ đồ LaTeX trong đề bài

Để tạo hình vẽ LaTeX, hãy bọc mã nguồn LaTeX của bạn bên trong thẻ `<latex>` trong phần mô tả đề bài:

```markdown
### Ví dụ về sơ đồ hình vẽ LaTeX:

Hình vẽ dưới đây được vẽ trực tiếp bằng mã LaTeX:

<latex>
\documentclass{standalone}
\begin{document}
$E=mc^2$
\end{document}
</latex>
```

Khuyên dùng lớp tài liệu `\documentclass{standalone}` để sơ đồ tự động co giãn ôm khít hình vẽ, hiển thị đẹp nhất trên giao diện web.
