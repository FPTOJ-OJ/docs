# Hướng dẫn thiết lập Máy chấm (Judge Server)

Tài liệu này hướng dẫn bạn quy trình cài đặt và kết nối một máy chấm bài (Judge Server) độc lập đến trang web FPTOJ chính. Máy chấm có thể nằm cùng máy chủ với Site hoặc trên một máy chủ riêng biệt.

Hệ thống sử dụng dự án máy chấm độc quyền: **[FPTOJ Judge Server](https://github.com/FPTOJ-OJ/judge-server)**.

---

## 1. Chuẩn bị phía Giao diện Web (Site)

Trước khi kết nối máy chấm, bạn cần đăng ký máy chấm đó trên giao diện quản trị Admin của trang Web:

1. Đăng nhập trang Admin (`/admin/`).
2. Tìm tới mục **Judges** (Trình chấm) và bấm **Add judge** (Thêm trình chấm).
3. Nhập **Name** (Tên máy chấm, ví dụ: `judge-01`) và tạo một khóa xác thực bí mật bằng cách bấm nút **Regenerate** cạnh trường **Auth key**. Lưu lại tên và Auth key này để khai báo trên máy chấm.
4. Kiểm tra cổng kết nối: Trong file `dmoj/local_settings.py` phía Web Site, kiểm tra cấu hình `BRIDGED_JUDGE_ADDRESS`. Mặc định cổng kết nối là `localhost:9999` (nếu chạy cùng Server) hoặc có thể đổi IP máy để nhận kết nối từ Server máy chấm bên ngoài. Đảm bảo cổng này đã được mở trên tường lửa (Firewall).

---

## 2. Thiết lập phía Máy chấm (Judge-side Setup)

Có hai phương pháp cài đặt máy chấm: sử dụng **Docker** (Khuyến khích) hoặc cài đặt trực tiếp qua **PyPI/pip** (Thủ công).

---

### Phương pháp A: Cài đặt qua Docker (Khuyến khích & Nhanh gọn)

Sử dụng Docker giúp bạn có ngay toàn bộ các trình biên dịch (C++, Python, Java, Pascal, Go,...) đã được đóng gói sẵn và bảo mật cao mà không lo xung đột môi trường.

1. Tải mã nguồn của máy chấm:
   ```bash
   git clone --recursive https://github.com/FPTOJ-OJ/judge-server.git /home/<username>/judge-server
   cd /home/<username>/judge-server/.docker
   ```

2. Tạo file cấu hình máy chấm `judge.yml` và lưu tại `/home/<username>/problems/judge.yml` (Tạo thư mục `/home/<username>/problems` nếu chưa có). Mẫu file cấu hình:
   ```yaml
   id: judge-01                # Trùng với Name đã khai báo trên Admin Site
   key: "your_auth_key_here"  # Trùng với Auth key đã tạo trên Admin Site
   problem_storage_globs:
     - /problems/*
   ```

3. Biên dịch và khởi chạy Docker máy chấm (sử dụng gói cơ bản `judge-tier1` chứa C/C++, Python, Pascal, Java):
   ```bash
   make judge-tier1
   ```

4. Khởi chạy Docker Container chạy ngầm:
   ```bash
   docker run \
       --name judge-server \
       -v /home/<username>/problems:/problems \
       --cap-add=SYS_PTRACE \
       -d \
       --restart=always \
       dmoj/judge-tier1:latest \
       run -p 9999 -c /problems/judge.yml \
       127.0.0.1 judge-01 your_auth_key_here
   ```
   *(Lưu ý: Thay `127.0.0.1` bằng IP của Web Site nếu máy chấm nằm ở Server khác và `9999` bằng cổng kết nối của Bridge).*

---

### Phương pháp B: Cài đặt thủ công bằng Python Pip (Tùy biến cao)

Nếu không sử dụng Docker, bạn có thể tự cài đặt trên hệ điều hành Ubuntu/Linux:

1. Cài đặt các gói phụ thuộc hệ thống:
   ```bash
   sudo apt update
   sudo apt install -y python3-dev python3-pip build-essential libseccomp-dev
   ```

2. Cài đặt gói `dmoj` máy chấm:
   ```bash
   sudo pip3 install dmoj
   ```

3. Cấu hình tự động nhận diện ngôn ngữ lập trình trên máy:
   ```bash
   dmoj-autoconf
   ```
   Lệnh trên sẽ in ra danh sách các ngôn ngữ có trên hệ thống của bạn dạng YAML block. Hãy copy nội dung đó và tạo file `judge.yml` như sau:
   ```yaml
   id: judge-01
   key: "your_auth_key_here"
   problem_storage_globs:
     - /path/to/problems/*
   runtime:
     # Dán toàn bộ block ngôn ngữ đã copy từ lệnh dmoj-autoconf vào đây
   ```

   Bạn có thể xem file cấu hình mẫu đầy đủ tại đây: [Mẫu cấu hình judge_conf.yml](../../sample_files/judge_conf.yml)

4. Chạy kiểm tra máy chấm bằng dòng lệnh:
   ```bash
   dmoj -c judge.yml localhost
   ```
   Nếu màn hình hiển thị trạng thái kết nối thành công và không báo lỗi, máy chấm đã sẵn sàng nhận bài chấm. Nhấn `Ctrl + C` để thoát và cấu hình chạy ngầm bằng Supervisord hoặc systemd.
