# Cấu hình Máy chấm (judge.yml)

Máy chấm của FPTOJ sử dụng một file định dạng YAML (thường đặt tên là `judge.yml`) để lưu trữ cấu hình về các trình biên dịch ngôn ngữ, thư mục chứa dữ liệu test của bài tập và các thông số xác thực khác.

Xem file mẫu cấu hình đầy đủ: [Mẫu cấu hình judge_conf.yml](../sample_files/judge_conf.yml)

---

## 1. Các trường cấu hình cơ bản

* **id**: Tên định danh duy nhất của máy chấm. Trường này phải khớp chính xác với trường **Name** được tạo trong mục **Judges** ở trang Admin.
* **key**: Khóa xác thực bí mật để kết nối. Phải khớp chính xác với **Auth key** trong trang Admin.
* **problem_storage_globs**: Định nghĩa đường dẫn chứa bộ test đề bài của hệ thống. Máy chấm sẽ dựa vào đây để định vị thư mục bài tập và đọc dữ liệu test.
  
Ví dụ:
```yaml
id: judge-01
key: "your_secret_auth_key"
problem_storage_globs:
  # Quét toàn bộ thư mục con cấp 1 trong /problems
  - /problems/*
  # Quét đệ quy mọi thư mục con ở tất cả các cấp
  - /problems/**/
```

---

## 2. Cấu hình các trình biên dịch (runtime)

Khối `runtime` định nghĩa đường dẫn đến trình biên dịch hoặc trình thực thi của các ngôn ngữ lập trình được hỗ trợ trên máy chấm.

Hầu hết các ngôn ngữ sẽ được tự động điền bởi lệnh `dmoj-autoconf`. Tuy nhiên, nếu bạn cài đặt trình biên dịch tùy biến hoặc chạy trong Docker, bạn có thể chỉnh sửa cấu hình này để tối ưu hóa cờ biên dịch (compiler flags).

Ví dụ cấu hình cờ tối ưu hóa cho C++20 và Python 3:
```yaml
runtime:
  c++20:
    - /usr/bin/g++
    - -O3
    - -std=c++20
  py3:
    - /usr/bin/python3
```

---

## 3. Khắc phục lỗi kết nối thường gặp (Troubleshooting)

Khi khởi chạy máy chấm, nếu gặp các lỗi sau, hãy kiểm tra các nguyên nhân tương ứng:

### Lỗi 1: `Handshake failed: Invalid key`
* **Nguyên nhân:** Khóa `key` khai báo trong `judge.yml` không trùng khớp với **Auth key** được tạo trên Admin Site.
* **Khắc phục:** Copy lại chính xác Auth key từ trang Admin Site và dán vào file `judge.yml`.

### Lỗi 2: `Connection refused` hoặc `Connection timeout`
* **Nguyên nhân:** Máy chấm không thể kết nối tới cổng của Bridge trên Web Site (mặc định là `9999`).
* **Khắc phục:** 
  1. Đảm bảo cổng `9999` đã được mở trên tường lửa của máy chủ Web Site.
  2. Kiểm tra xem tiến trình `bridged` trên Web Site đang chạy hay đã bị dừng (`sudo supervisorctl status bridged`).
  3. Khai báo chính xác IP công khai hoặc IP mạng nội bộ của Web Site thay vì `localhost` nếu máy chấm nằm ở một máy chủ riêng biệt.

### Lỗi 3: `Syscall disallowed` hoặc lỗi liên quan đến Sandbox
* **Nguyên nhân:** Khi chạy không có quyền root hoặc thiếu cấu hình nhân Linux, cơ chế Sandbox bảo mật của DMOJ không thể kích hoạt seccomp filter.
* **Khắc phục:** Nếu chạy trong Docker, hãy đảm bảo bạn đã thêm cờ `--cap-add=SYS_PTRACE` khi chạy lệnh `docker run`. Cờ này cho phép container thực hiện các cuộc gọi syscall giám sát bảo mật.
