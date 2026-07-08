# Hướng dẫn thiết lập Máy chấm (Judge Server)

Tài liệu này hướng dẫn bạn quy trình cài đặt và kết nối một máy chấm bài (Judge Server) độc lập đến trang web FPTOJ chính. Máy chấm có thể nằm cùng máy chủ với Site hoặc trên một máy chủ riêng biệt.

Hệ thống sử dụng dự án máy chấm hiệu năng cao: **[FPTOJ Judge Server](https://github.com/FPTOJ-OJ/judge-server)**.

---

## 1. Đăng ký Máy chấm trên Admin Site

Trước khi kết nối máy chấm, bạn cần đăng ký máy chấm đó trên giao diện quản trị Admin của trang Web:

1. Đăng nhập trang Admin (`/admin/`).
2. Tìm tới mục **Judges** (Trình chấm) và bấm **Add judge** (Thêm trình chấm).
3. Nhập **Name** (Tên máy chấm, ví dụ: `judge-01`) và tạo một khóa xác thực bí mật bằng cách bấm nút **Regenerate** cạnh trường **Auth key**. Lưu lại tên và Auth key này để khai báo trên máy chấm.
4. Kiểm tra cổng kết nối: Trong file `dmoj/local_settings.py` phía Web Site, kiểm tra cấu hình `BRIDGED_JUDGE_ADDRESS`. Mặc định là `0.0.0.0:9999` để lắng nghe mọi kết nối bên ngoài. Đảm bảo cổng này đã được mở trên tường lửa (Firewall) bằng lệnh:
   ```bash
   sudo ufw allow 9999/tcp
   ```

---

## 2. Thiết lập phía Máy chấm (Judge-side Setup)

Có hai phương pháp cài đặt máy chấm: sử dụng **Docker** (Khuyến khích - bảo mật & đa ngôn ngữ) hoặc cài đặt trực tiếp qua **PyPI/pip** (Thủ công).

---

### Phương pháp A: Cài đặt qua Docker (Khuyến khích & Nhanh gọn)

Sử dụng Docker giúp cô lập tốt các tiến trình của thí sinh, đồng thời đóng gói đầy đủ các trình biên dịch (C++, Python, Java, Pascal, Go,...) mà không sợ xung đột hay tốn công cấu hình.

#### Bước 1: Tải mã nguồn máy chấm
```bash
git clone --recursive https://github.com/FPTOJ-OJ/judge-server.git /home/kien/judge
cd /home/kien/judge
```

#### Bước 2: Tạo cấu hình `judge.yml`
Tạo file cấu hình máy chấm và lưu tại `/data/problems/judge.yml`:
```yaml
id: judge-01                # Khớp chính xác với Name đã tạo trên Admin Site
key: "your_auth_key_here"  # Khớp chính xác với Auth key đã tạo trên Admin Site
problem_storage_globs:
  - /problems/*
```

#### Bước 3: Biên dịch Docker Image
Chúng ta biên dịch gói `judge-tier1` (bao gồm các ngôn ngữ cơ bản: C/C++, Python, Pascal, Java):
```bash
cd /home/kien/judge/.docker
docker build --build-arg TAG="master" -t dmoj/judge-tier1 -t dmoj/judge-tier1:latest ./tier1
```

#### Bước 4: Khởi chạy Máy chấm Docker
Chạy lệnh sau để khởi chạy máy chấm chạy ngầm và tự động tải lại nếu máy chủ reset:
```bash
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
* **Lưu ý:**
  * Thay `127.0.0.1` bằng IP của Web Site nếu máy chấm nằm ở Server khác.
  * `-p 9999` là cổng kết nối của Bridge trên Site.
  * Thư mục test case `/data/problems` trên Host được mount vào `/problems` trong Docker.

---

### Phương pháp B: Cài đặt thủ công bằng Python Pip (Tùy biến cao)

Nếu không sử dụng Docker, bạn có thể tự cài đặt trên hệ điều hành Ubuntu/Linux:

#### Bước 1: Cài đặt các gói phụ thuộc hệ thống
```bash
sudo apt update
sudo apt install -y python3-dev python3-pip build-essential libseccomp-dev
```

#### Bước 2: Cài đặt gói `dmoj` máy chấm
```bash
sudo pip3 install dmoj
```

#### Bước 3: Tự động nhận diện ngôn ngữ trên hệ thống
```bash
dmoj-autoconf
```
Lệnh trên sẽ quét máy chủ và in ra cấu hình các trình biên dịch hiện có dưới dạng YAML. Copy khối cấu hình đó và tạo file `judge.yml` tại `/data/problems/judge.yml`:
```yaml
id: judge-01
key: "your_auth_key_here"
problem_storage_globs:
  - /data/problems/*
runtime:
  # Dán toàn bộ block ngôn ngữ đã copy từ lệnh dmoj-autoconf vào đây
```

Bạn có thể xem file cấu hình mẫu đầy đủ tại đây: [Mẫu cấu hình judge_conf.yml](../sample_files/judge_conf.yml)

#### Bước 4: Khởi chạy kiểm tra máy chấm
```bash
dmoj -c /data/problems/judge.yml 127.0.0.1
```
Nếu màn hình hiện thông báo kết nối thành công và không báo lỗi, máy chấm đã sẵn sàng.

---

## 3. Quản lý Nhiều máy chấm (Multi-Judges)

Nếu muốn chạy nhiều máy chấm trên cùng một máy chủ để tăng tốc độ chấm (ví dụ: máy chấm tốc độ cao, máy chấm dự phòng), bạn chỉ cần:
1. Đăng ký các tên máy chấm mới trên Admin Site (ví dụ: `judge-02`, `judge-03`).
2. Tạo các tệp `judge.yml` hoặc chạy các Docker container khác nhau bằng cách thay đổi tham số tên và key:
   ```bash
   docker run --name fptoj-judge-speedup ... dmoj/judge-tier1:latest run -p 9999 -c /problems/judge-speedup.yml 127.0.0.1 judge-speedup key_secret_here
   ```
Các máy chấm sẽ tự động nhận bài và chia sẻ tải trọng chấm bài của học sinh một cách mượt mà.
