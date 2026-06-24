# Các mã trạng thái chấm bài (Verdicts)

Trang này liệt kê tất cả các mã trạng thái (kết quả chấm) bạn có thể gặp trên FPTOJ và ý nghĩa của chúng. 

Khi một bài nộp có nhiều trạng thái khác nhau trên các test case khác nhau, hệ thống sẽ ưu tiên hiển thị lỗi có thứ tự ưu tiên cao nhất làm kết quả chung cho toàn bộ bài nộp. Dưới đây là danh sách các trạng thái được xếp theo thứ tự ưu tiên tăng dần:

---

## 1. AC - Accepted (Chấp nhận bài làm)
Chúc mừng! Chương trình của bạn đã chạy chính xác trên tất cả các test case và trả về kết quả đúng trong giới hạn thời gian và bộ nhớ cho phép.

---

## 2. WA - Wrong Answer (Kết quả sai)
Chương trình của bạn đã chạy xong mà không bị sập (crash), nhưng kết quả xuất ra (output) không khớp với kết quả mẫu (key) của đề bài.

---

## 3. IR - Invalid Return (Trả về giá trị không hợp lệ)
Chương trình kết thúc với mã lỗi khác 0 (exit code != 0).
- Đối với các ngôn ngữ như C/C++, lỗi này thường xảy ra khi bạn quên `return 0;` trong hàm `main`, hoặc chương trình bị sập.
- Đối với Python hoặc Java, lỗi này thường đi kèm với tên ngoại lệ (Exception) cụ thể gây sập chương trình như `NameError`, `IndexError`, hoặc `java.lang.NullPointerException`.

---

## 4. RTE - Runtime Error (Lỗi thời gian chạy)
Chương trình của bạn bị dừng đột ngột hoặc vi phạm các quy tắc an toàn hệ thống (chủ yếu xảy ra với các ngôn ngữ biên dịch như C/C++). FPTOJ phân tích chi tiết lỗi thời gian chạy thành các mô tả sau:

| Thông báo hiển thị | Ý nghĩa chi tiết |
| :--- | :--- |
| `segmentation fault`, `bus error` | Chương trình truy cập vào vùng nhớ không được phép (ví dụ: chỉ số mảng vượt quá giới hạn, sử dụng con trỏ NULL) hoặc bị hệ thống tắt do tràn bộ nhớ. |
| `floating point exception` | Chương trình thực hiện phép toán không hợp lệ, phổ biến nhất là **chia cho số 0**. |
| `killed` | Chương trình bị tắt bởi hệ thống giám sát thời gian chạy (chưa rõ nguyên nhân). |
| `{} syscall disallowed` | Lỗi vi phạm bảo mật hệ thống. Chương trình của bạn đã cố gắng gọi một hàm hệ thống (system call) bị cấm (ví dụ: cố gắng mở file trên ổ đĩa, chạy lệnh hệ thống shell, hoặc kết nối mạng). |
| `std::bad_alloc` | Chương trình sử dụng lệnh `new` để cấp phát động nhưng bộ nhớ RAM còn lại không đủ đáp ứng. |
| `failed initializing` | Chương trình khai báo các biến hoặc mảng tĩnh toàn cục có kích thước quá lớn vượt quá giới hạn bộ nhớ của bài tập ngay từ khi khởi động (Ví dụ: khai báo `int a[10000][10000]` chiếm tới 381MB bộ nhớ trong khi giới hạn của bài chỉ là 64MB). |

---

## 5. OLE - Output Limit Exceeded (Xuất quá nhiều dữ liệu)
Chương trình ghi quá nhiều dữ liệu ra màn hình (`stdout`), thông thường giới hạn tối đa là 256MB để ngăn chặn các chương trình bị lặp vô hạn ghi file làm đầy ổ cứng của Server.

---

## 6. MLE - Memory Limit Exceeded (Quá giới hạn bộ nhớ)
Chương trình của bạn sử dụng lượng bộ nhớ RAM vượt quá giới hạn cho phép của bài tập. Lỗi này cũng có thể gián tiếp kích hoạt lỗi RTE `segmentation fault` hoặc `std::bad_alloc`.

---

## 7. TLE - Time Limit Exceeded (Quá giới hạn thời gian)
Chương trình chạy lâu hơn thời gian tối đa cho phép của bài tập (ví dụ: bị lặp vô hạn hoặc thuật toán có độ phức tạp quá lớn).

---

## 8. IE - Internal Error (Lỗi hệ thống)
Mã lỗi này xảy ra khi máy chấm gặp lỗi nội bộ hoặc cấu hình bộ test case của bài tập bị sai (ví dụ: thiếu file test đầu ra `.ans` / `.out`). Nếu gặp lỗi này, giáo viên hoặc quản trị viên hệ thống cần kiểm tra lại cấu hình bài tập.
