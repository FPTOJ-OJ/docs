# Trình chấm Tương tác & Gọi hàm (Custom Graders)

Đôi khi, việc tương tác tĩnh giữa máy chấm và bài nộp (Input/Output thô qua tệp tin hoặc luồng vào ra chuẩn) là không đủ. Ví dụ:
- **Bài toán tương tác (Interactive)**: Thí sinh cần giao tiếp với máy chấm qua cơ chế hỏi - đáp (ví dụ: máy sinh số bí ẩn, thí sinh đoán số và máy phản hồi lớn hơn/nhỏ hơn).
- **Chấm hàm/Thư viện (Signature/Grader)**: Thí sinh không viết hàm `main` đọc dữ liệu, mà chỉ viết một hàm chức năng (ví dụ: `bool is_valid(int n)`) và liên kết vào chương trình chấm chính của giáo viên.

FPTOJ hỗ trợ cấu hình các dạng chấm nâng cao này trực tiếp qua khóa `custom_judge`, `interactive`, hoặc `signature_grader` trong `init.yml`.

---

## 1. Trình chấm Tương tác (Interactive Grading)

Học sinh cần giao tiếp thời gian thực với máy chấm. Chúng ta sử dụng khóa `interactive` để khai báo tệp tin tương tác (thường viết bằng C++ kết hợp `testlib.h`).

```yaml
unbuffered: true
interactive: {files: interactor.cpp, type: testlib}
test_cases:
- {in: data.1.in, points: 20}
- {in: data.2.in, points: 80}
```

> [!IMPORTANT]  
> Nên thiết lập `unbuffered: true` để tránh bộ nhớ đệm (buffer) của luồng xuất làm nghẽn tiến trình hỏi - đáp thời gian thực, học sinh sẽ không cần gọi lệnh `fflush(stdout)` thủ công trong code của họ.

### Viết file `interactor.cpp` (sử dụng C++ và testlib):
```cpp
#include "testlib.h"
#include <iostream>

using namespace std;

int main(int argc, char* argv[]) {
    // Khởi tạo interactor
    registerInteraction(argc, argv);
    
    // Đọc số bí mật từ file test đầu vào (argv[1])
    int secret = inf.readInt();
    
    int guesses = 0;
    while (guesses < 30) {
        // Đọc dự đoán của học sinh từ luồng xuất của họ (ouf)
        int guess = ouf.readInt();
        guesses++;
        
        if (guess == secret) {
            // Ghi kết quả về cho học sinh qua luồng nhập của họ (stdout)
            cout << "OK" << endl;
            quitf(_ok, "Đoán đúng sau %d lần!", guesses);
        } else if (guess > secret) {
            cout << "FLOATS" << endl;
        } else {
            cout << "SINKS" << endl;
        }
    }
    quitf(_wa, "Quá số lần đoán cho phép!");
}
```

---

## 2. Chấm hàm/Thư viện (Function Signature Grading)

Dành cho các bài toán kiểu IOI, học sinh chỉ cần thực hiện viết một hàm giải thuật theo đặc tả.

```yaml
signature_grader: {entry: handler.cpp, header: solution.h}
test_cases:
- {in: test.1.in, out: test.1.out, points: 100}
```

- **entry (handler.cpp)**: Chương trình chấm chính của giáo viên chứa hàm `main()`. Hàm `main` này sẽ đọc test case từ `stdin`, gọi hàm của học sinh và in kết quả ra `stdout`.
- **header (solution.h)**: File chứa nguyên mẫu hàm (prototype) định nghĩa các hàm mà học sinh cần viết.

### Ví dụ file `solution.h`:
```cpp
#ifndef SOLUTION_H
#define SOLUTION_H
bool is_valid(int n);
#endif
```

### Ví dụ file `handler.cpp` (chương trình của giáo viên):
```cpp
#include "solution.h"
#include <iostream>

using namespace std;

int main() {
    int n;
    if (cin >> n) {
        bool res = is_valid(n); // Gọi hàm của học sinh
        cout << (res ? "CORRECT" : "WRONG") << endl;
    }
    return 0;
}
```

Học sinh nộp bài chỉ cần viết file chứa hàm `is_valid`:
```cpp
#include "solution.h"
bool is_valid(int n) {
    return n % 2 == 0;
}
```
Máy chấm sẽ tự động ghép code của học sinh với file chấm của giáo viên và biên dịch thành một file chạy duy nhất.
