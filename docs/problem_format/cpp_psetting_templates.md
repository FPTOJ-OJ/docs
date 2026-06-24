# Mẫu thiết lập bài tập C++ (Validator & Checkers)

Khi thiết lập các bài tập chuyên nghiệp, bạn cần viết mã nguồn C++ để xác thực tính hợp lệ của dữ liệu đầu vào (Validator) hoặc kiểm tra tính đúng đắn của lời giải (Checkers/Interactors). 

FPTOJ cung cấp sẵn 3 mẫu cấu hình đầu vào chuẩn viết bằng C++ trong thư mục `sample_files`:

- **[Mẫu bộ xác thực Validator](../../sample_files/problem_setting/validator.cpp)**: Dùng để kiểm tra bộ test đầu vào của bài tập có đúng định dạng và giới hạn đề bài đưa ra hay không.
- **[Mẫu Checker/Interactor Tuyệt đối (Identical)](../../sample_files/problem_setting/identical_checker_interactor.cpp)**: Kiểm tra đáp án yêu cầu khớp chính xác từng khoảng trắng và ký tự xuống dòng.
- **[Mẫu Checker/Interactor Tiêu chuẩn (Standard)](../../sample_files/problem_setting/standard_checker_interactor.cpp)**: Tự động bỏ qua khoảng trắng thừa tương tự bộ chấm mặc định của máy.

---

## 1. Trình xác thực dữ liệu (Validator)

Validator được dùng để chạy thử trên toàn bộ file test đầu vào (`.in`/`.inp`) nhằm đảm bảo dữ liệu test sạch, không bị thừa khoảng trắng, xuống dòng sai quy cách, hoặc giá trị biến vượt quá giới hạn mô tả trong đề.

Các hàm xác thực cơ bản được định nghĩa sẵn trong mẫu:

### Hàm khoảng trắng:
- `readSpace()`: Bắt buộc vị trí hiện tại phải là một ký tự khoảng trắng (Space). Nếu không sẽ báo lỗi bộ test.
- `readNewLine()`: Bắt buộc vị trí hiện tại phải là ký tự xuống dòng (`\n`).
- `readEOF()`: Bắt buộc tệp dữ liệu phải kết thúc ngay tại vị trí hiện tại.

### Hàm đọc dữ liệu và kiểm tra giới hạn:
- `readToken('min_char', 'max_char')`: Đọc từ tiếp theo và kiểm tra xem các ký tự của từ đó có nằm trong khoảng từ `min_char` đến `max_char` hay không (ví dụ: `readToken('a', 'z')` kiểm tra xâu chỉ gồm ký tự viết thường).
- `readLine('min_char', 'max_char')`: Đọc toàn bộ dòng tiếp theo (không bao gồm ký tự xuống dòng).
- `readInt(lo, hi)`: Đọc số nguyên tiếp theo và kiểm tra xem nó có nằm trong đoạn từ `lo` đến `hi` hay không. Tự động báo lỗi nếu số nguyên bị tràn, viết sai định dạng (ví dụ viết thừa số 0 ở đầu xâu).
- `readFloat(lo, hi, eps)`: Đọc số thực tiếp theo và kiểm tra giới hạn sai số.
- `readIntArray<Type>(N, lo, hi)`: Đọc mảng gồm `N` số nguyên cách nhau bởi dấu cách và kết thúc bằng ký tự xuống dòng.

---

## 2. Trình chấm Checker / Interactor

Các mẫu này được thiết kế để viết bộ chấm tùy biến (checker) hoặc tương tác (interactor) chạy trực tiếp trên máy chấm thông qua cổng `bridged`.

### 2.1 Checker Tuyệt đối (Identical Checker)
Yêu cầu học sinh định dạng kết quả chính xác tuyệt đối. Các hàm kiểm tra khoảng trắng (`readSpace`, `readNewLine`, `readEOF`) sẽ tự động trả về kết quả lỗi trình bày `Presentation Error` (PE) nếu thí sinh viết sai khoảng trắng, và `Wrong Answer` (WA) nếu kết quả tính toán sai.

Các hàm bổ sung:
- `exitWA()`, `exitPE()`: Dừng ngay chương trình và trả về lỗi WA/PE tương ứng.
- `assertWA(condition)`: Dừng chương trình trả về WA nếu điều kiện `condition` là `false`.
- Namespace `CheckerCodes`: Chứa các hằng số kết quả `AC` (0), `WA` (1), `PE` (2), `PARTIAL` (7). Sử dụng trong hàm `main` để trả về kết quả chấm: `return CheckerCodes::AC;`.

### 2.2 Checker Tiêu chuẩn (Standard Checker)
Checker này hoạt động linh hoạt và dễ chịu hơn cho thí sinh, bỏ qua các lỗi khoảng trắng thừa ở cuối dòng hoặc xuống dòng thừa.

- Tất cả các hàm kiểm tra lỗi định dạng sẽ trả về `WA` thay vì `PE`.
- Tự động tiêu thụ (consume) khoảng trắng khi gọi các hàm đọc dữ liệu như `readToken()`, `readInt()`.
