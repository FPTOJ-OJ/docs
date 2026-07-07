# Trình chấm Tùy chỉnh (Custom Checkers)

Đối với các bài tập có nhiều kết quả đúng khác nhau (ví dụ: bài tập yêu cầu tìm đường đi ngắn nhất bất kỳ, hoặc bài toán hình học có sai số), so khớp văn bản thô (Standard) sẽ không đáp ứng được.

FPTOJ hỗ trợ thuộc tính `checker` trong `init.yml` để cấu hình trình chấm tùy chỉnh. Trình chấm này có thể viết bằng **Python** hoặc **C++ (sử dụng thư viện `testlib.h`)** nhằm kiểm tra tính đúng đắn của lời giải học sinh sau khi chạy xong.

---

## 1. Các Trình chấm mặc định có sẵn (Built-in Checkers)

Bạn có thể sử dụng ngay các bộ checker có sẵn của hệ thống bằng cách khai báo trường `checker` trong `init.yml`:

```yaml
checker: <tên_checker>
```
Hoặc truyền thêm tham số cấu hình:
```yaml
checker:
  name: <tên_checker>
  args:
    precision: 6
```

### 1.1 So khớp chuẩn - `standard`
Đây là bộ chấm mặc định nếu không khai báo `checker`.
- Tách đầu ra của thí sinh và đáp án theo từng dòng và từng từ (token), bỏ qua các khoảng trắng thừa ở cuối dòng. 
- So khớp chính xác từng từ trên mỗi dòng.

### 1.2 So khớp dễ dãi - `easy`
- Bỏ qua toàn bộ khoảng trắng, ký tự xuống dòng và không phân biệt chữ hoa/thường. Sau đó đếm số lần xuất hiện của mỗi ký tự để so sánh.

### 1.3 Chấm số thực có sai số - `floats`
Sử dụng khi đáp án là các số thực và chấp nhận sai số nhỏ.
- **precision**: Sai số cho phép (mặc định là `6`, tương ứng sai số $\le 10^{-6}$).
- **error_mode**: 
  - `default`: Chấp nhận sai số tuyệt đối hoặc tương đối trong khoảng cho phép.
  - `absolute`: Chỉ kiểm tra sai số tuyệt đối.
  - `relative`: Chỉ kiểm tra sai số tương đối.

### 1.4 So khớp tuyệt đối - `identical`
- So sánh chính xác từng ký tự bao gồm cả khoảng trắng và ký tự xuống dòng.
- Nếu sai khác khoảng trắng, hệ thống sẽ báo lỗi `Presentation Error` (Lỗi trình bày).

---

## 2. Viết Checker tự chế bằng Python (Custom Python Checker)

Bạn viết một đoạn script Python lưu trong thư mục bài tập (ví dụ: `check.py`) và cấu hình:
```yaml
checker: check.py
```

File `check.py` phải định nghĩa hàm `check` để máy chấm gọi chạy:

```python
from dmoj.result import CheckerResult

def check(process_output, judge_output, **kwargs):
    # process_output: bytearray chứa kết quả bài làm của học sinh
    # judge_output: bytearray chứa đáp án mẫu của giáo viên
    # kwargs: các thông tin phụ trợ (source code học sinh, input của test...)
    
    student_ans = process_output.decode('utf-8').strip()
    correct_ans = judge_output.decode('utf-8').strip()
    
    if student_ans == correct_ans:
        return True # Trả về True nếu đúng hoàn toàn (nhận 100% điểm của test)
        
    # Hoặc trả về điểm thành phần:
    # CheckerResult(đúng/sai, số_điểm_đạt_được, 'thông_tin_phản_hồi_cho_học_sinh')
    return CheckerResult(False, 5.0, feedback='Sai số nhỏ!')
```

---

## 3. Viết Checker bằng C++ và Testlib (Native Checkers - Khuyên dùng)

Với các bài tập lớn, chạy checker bằng Python sẽ rất chậm. Bạn nên viết checker bằng C++ sử dụng thư viện chuẩn của Codeforces là `testlib.h` (được đóng gói sẵn trên máy chấm).

Cấu hình trong `init.yml` sử dụng công cụ chuyển tiếp `bridged`:

```yaml
checker:
  name: bridged
  args:
    files:
      - checker.cpp
      - testlib.h
    lang: c++17
    type: testlib
```

### Viết file `checker.cpp`:
```cpp
#include "testlib.h"
#include <iostream>

using namespace std;

int main(int argc, char* argv[]) {
    // Khởi tạo testlib
    registerTestlibCmd(argc, argv);
    
    // Đọc dữ liệu từ file test đầu vào
    int n = inf.readInt();
    
    // Đọc đáp án mẫu của giáo viên
    int ans = ans.readInt();
    
    // Đọc kết quả của học sinh
    int ouf_ans = ouf.readInt();
    
    if (ans == ouf_ans) {
        quitf(_ok, "Kết quả chính xác!"); // Trả về AC
    } else {
        quitf(_wa, "Kết quả sai. Mong đợi %d, nhận được %d", ans, ouf_ans); // Trả về WA
    }
}
```
Trình chấm sẽ tự động biên dịch `checker.cpp` và chạy kiểm tra nhanh chóng với hiệu năng cực cao.
