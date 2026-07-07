# Trình sinh bộ Test tự động (Generators)

Khi bài tập có dữ liệu test rất lớn (ví dụ: test case lên tới vài chục MB), việc tải lên file nén ZIP chứa các file văn bản phẳng sẽ rất chậm và tốn tài nguyên đĩa cứng. 

FPTOJ hỗ trợ tính năng **Generator (Trình sinh test)**. Thay vì tải lên các file test tĩnh, giáo viên chỉ cần tải lên một file mã nguồn sinh test (ví dụ: viết bằng C++ sử dụng thư viện `testlib.h`). Trình chấm sẽ tự động biên dịch và chạy file này với các tham số khác nhau để sinh ra dữ liệu test case ngay tại thời điểm chấm bài.

---

## 1. Cấu hình `generator` trong init.yml

Khóa `generator` trong file `init.yml` có thể nhận các giá trị:

- **Tên file nguồn**: Ví dụ `gen.cpp`.
- **Mảng các file**: Nếu file sinh test cần thêm file header phụ trợ (ví dụ: `[gen.cpp, testlib.h]`).
- **Khối cấu hình chi tiết (YAML Map)**:
  - `source`: Tên file nguồn sinh test (hoặc mảng các file).
  - `language`: Ngôn ngữ lập trình của file sinh test (nếu trống, hệ thống tự nhận dạng qua đuôi file).
  - `compiler_time_limit`: Giới hạn thời gian biên dịch file sinh test (nên đặt là 60 giây nếu sử dụng `testlib.h`).
  - `time_limit`: Giới hạn thời gian chạy của trình sinh test để sinh ra 1 test case.
  - `memory_limit`: Giới hạn bộ nhớ RAM cấp cho trình sinh test.

---

## 2. Tham số sinh test (generator_args)

Trình sinh test nhận các tham số dòng lệnh (`arguments`) để sinh ra các test case khác nhau (ví dụ: sinh test nhỏ, sinh test lớn, sinh test ngẫu nhiên).

Cấu hình tham số được khai báo trong trường `generator_args` của mỗi test case:

```yaml
generator: [gen.cpp, testlib.h]
test_cases:
- {generator_args: [1, 100], points: 10}   # Sinh test 1: n=100
- {generator_args: [2, 10000], points: 20} # Sinh test 2: n=10000
- {generator_args: [3, 100000], points: 70} # Sinh test 3: n=100000 (test lớn)
```

### Nguyên lý hoạt động:
1. Máy chấm sẽ biên dịch `gen.cpp` thành file thực thi.
2. Với test case 1, máy chấm gọi chạy: `./gen 1 100`.
3. Trình sinh test phải ghi dữ liệu **đầu vào (Input)** ra luồng xuất chuẩn `stdout`.
4. Trình sinh test phải ghi dữ liệu **đầu ra mẫu (Output/Answer)** ra luồng lỗi tiêu chuẩn `stderr`.
5. Máy chấm thu thập `stdout` làm file đầu vào và `stderr` làm đáp án mẫu để đối chiếu với bài làm của học sinh.
