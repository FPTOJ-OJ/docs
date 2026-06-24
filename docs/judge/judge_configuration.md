# Cấu hình Máy chấm (judge.yml)

Máy chấm của FPTOJ sử dụng một file định dạng YAML (thường đặt tên là `judge.yml`) để lưu trữ cấu hình về các trình biên dịch ngôn ngữ, thư mục chứa dữ liệu test của bài tập và các thông số xác thực khác.

Xem file mẫu cấu hình đầy đủ: [Mẫu cấu hình judge_conf.yml](../../sample_files/judge_conf.yml)

---

## 1. Cấu hình định danh (id) và khóa bí mật (key)

Đây là hai trường quan trọng nhất để xác thực và kết nối máy chấm tới trang web chính của bạn.

- **id**: Tên định danh của máy chấm. Trường này phải khớp chính xác với trường **Name** được tạo trong mục **Judges** ở trang Admin.
- **key**: Khóa xác thực bí mật. Phải khớp chính xác với **Auth key** trong trang Admin.

Ví dụ:
```yaml
id: judge-01
key: "your_secret_auth_key"
```

---

## 2. Cấu hình thư mục chứa dữ liệu bài tập (problem_storage_globs)

Trình chấm cần biết các bài tập được lưu trữ ở đâu trên đĩa cứng để tải dữ liệu test case khi chấm bài. Cấu hình này sử dụng cú pháp glob (đường dẫn đại diện) của Python.

Bất kỳ thư mục con nào khớp với quy luật glob này và chứa file cấu hình `init.yml` đều được coi là một thư mục bài tập hợp lệ.

Ví dụ:
```yaml
problem_storage_globs:
  # Khớp với toàn bộ các thư mục con cấp 1 nằm trong /home/<username>/problems/
  - /home/<username>/problems/*
  
  # Khớp đệ quy mọi thư mục con ở tất cả các cấp trong /home/<username>/problems/
  - /home/<username>/problems/**/
```

---

## 3. Cấu hình các trình biên dịch (runtime)

Khối `runtime` định nghĩa đường dẫn đến trình biên dịch/thực thi của các ngôn ngữ trên máy chấm.

Hầu hết các ngôn ngữ sẽ được tự động cấu hình và điền bởi lệnh `dmoj-autoconf`. Tuy nhiên, nếu bạn cài đặt trình biên dịch ở một thư mục không nằm trong biến môi trường `$PATH` của hệ thống, bạn cần cấu hình thủ công trong file `judge.yml`.

Ví dụ cấu hình cho Python 3 và GCC C++:
```yaml
runtime:
  py3:
    - /usr/bin/python3
  c++20:
    - /usr/bin/g++
    - -O3
    - -std=c++20
```
