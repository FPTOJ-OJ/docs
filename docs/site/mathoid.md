# Hiển thị Công thức Toán học (LaTeX & Mathoid)

FPTOJ hỗ trợ hiển thị các công thức toán học và biểu thức phức tạp trong đề bài (ví dụ: căn bậc hai, phân số, ma trận) thông qua định dạng LaTeX tiêu chuẩn.

Hệ thống sử dụng dự án **[Wikimedia Mathoid](https://github.com/wikimedia/mathoid)** để biên dịch các biểu thức LaTeX này thành hình ảnh SVG hoặc MathML tương thích tốt với mọi trình duyệt.

---

## 1. Cài đặt Mathoid

Hãy thực hiện cài đặt Mathoid theo [tài liệu hướng dẫn của Mathoid](https://github.com/wikimedia/mathoid). Sau khi cài đặt xong, giả định dịch vụ Mathoid đang lắng nghe cục bộ tại cổng `8888` (`http://localhost:8888`).

---

## 2. Cấu hình FPTOJ kết nối tới Mathoid

Mở file `dmoj/local_settings.py` và thêm các dòng cấu hình sau:

```python
# Đường dẫn API của dịch vụ Mathoid
MATHOID_URL = 'http://127.0.0.1:8888'

# Thư mục lưu trữ hình ảnh toán học đã biên dịch (SVG/PNG)
# Nên chọn thư mục tồn tại lâu dài để tránh tải lại làm chậm hệ thống
MATHOID_CACHE_ROOT = '/home/<username>/site/userdata/mathoid_cache'

# URL tương ứng để truy cập thư mục cache từ trình duyệt
MATHOID_CACHE_URL = '/mathoid/'
```

Tạo thư mục cache và cấp quyền ghi cho Web server:
```bash
mkdir -p /home/<username>/site/userdata/mathoid_cache
chmod -R 775 /home/<username>/site/userdata/mathoid_cache
```

Cấu hình thêm định tuyến Nginx để phục vụ trực tiếp các file ảnh trong thư mục cache:
```nginx
location /mathoid/ {
    alias /home/<username>/site/userdata/mathoid_cache/;
}
```

Khởi động lại Nginx và FPTOJ:
```bash
sudo systemctl reload nginx
sudo supervisorctl restart site
```

---

## 3. Cách viết công thức Toán LaTeX trong đề bài

Để viết công thức toán học, bạn viết cú pháp LaTeX giữa các ký hiệu đặc biệt:

### Công thức cùng dòng (Inline math):
Kẹp biểu thức trong ký tự ngã `~` (Ví dụ: `~1 \le N \le 10^9~` sẽ hiển thị thành $1 \le N \le 10^9$ nhỏ gọn cùng dòng).

### Công thức khối riêng biệt (Block math):
Kẹp biểu thức giữa cặp ký tự `$$` ở đầu dòng và cuối dòng:

```markdown
Dãy Fibonacci được định nghĩa theo công thức:

$$F(n) = \begin{cases} 0, & \text{nếu } n = 0 \\ 1, & \text{nếu } n = 1 \\ F(n-2) + F(n-1), & \text{nếu } n \ge 2 \end{cases}$$

Cho số nguyên dương ~N~ (~1 \le N \le 10^{19}~), hãy tính số Fibonacci thứ ~N~.
```
