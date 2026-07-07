# Quản lý bài tập trên giao diện FPTOJ

FPTOJ đi kèm với một giao diện web trực quan giúp giáo viên dễ dàng tạo, chỉnh sửa nội dung đề bài cũng như tải lên bộ test case chấm điểm.

---

## 1. Cấu hình thư mục chứa dữ liệu test của Web

Trước tiên, hãy đảm bảo biến cấu hình `DMOJ_PROBLEM_DATA_ROOT` trong file `dmoj/local_settings.py` đã được thiết lập đến một thư mục trên máy chủ của bạn (ví dụ: `/home/<username>/site/userdata/problems`). Đây là nơi lưu trữ toàn bộ file ZIP test case được tải lên từ web.

---

## 2. Thêm một bài tập mới

1. Đăng nhập trang Admin (`/admin/`).
2. Tìm mục **Problems** (Bài tập) và bấm nút **Add** (Thêm).
3. Nhập các thông tin cơ bản:
   - **Problem code (Mã bài tập)**: Định danh duy nhất trên toàn hệ thống (ví dụ: `aplusb`). Chỉ chứa ký tự chữ và số viết thường, không dấu cách.
   - **Problem title (Tên bài tập)**: Tên hiển thị của bài tập (ví dụ: `Tính tổng A + B`).
   - **Authors (Tác giả)**: Hãy chọn tài khoản của chính bạn làm tác giả, nếu không bạn có thể bị khóa quyền chỉnh sửa bài tập này về sau.
4. **Soạn thảo đề bài (Description)**:
   - Sử dụng cú pháp **Markdown** kết hợp với công thức Toán **LaTeX** để viết đề bài sinh động.
   - Bạn có thể xem và tham khảo mẫu đề bài chuẩn đầy đủ tính năng tại đây: [Mẫu Markdown đề bài](../../sample_files/problem_markdown_example.md.txt) (Hãy copy và dán thử vào trình soạn thảo để xem cách hoạt động).
5. Cuối cùng, kéo xuống cuối trang và bấm **Save** (Lưu). Nhấp vào nút **View on site** ở góc trên để xem giao diện đề bài hiển thị cho học sinh.

---

## 3. Tải lên và cấu hình dữ liệu chấm bài (Test Data)

FPTOJ lưu trữ cấu hình chấm điểm dưới dạng file YAML (`init.yml`), tuy nhiên bạn không cần chỉnh sửa file này thủ công mà có thể làm trực tiếp qua giao diện web:

1. Tại trang chi tiết bài tập trên giao diện người dùng, bấm nút **"Edit test data"** (Chỉnh sửa dữ liệu test).
2. Chuẩn bị file ZIP chứa bộ test case:
   - File test đầu vào thường có đuôi `.in` hoặc `.inp` (ví dụ: `aplusb.1.in`, `aplusb.2.in`).
   - File test đầu ra mẫu tương ứng có đuôi `.out` hoặc `.ans` (ví dụ: `aplusb.1.out`, `aplusb.2.out`).
   - Nén trực tiếp các file này vào một file `.zip`.
3. Bấm **Chọn tệp** và upload file ZIP đó lên. Hệ thống sẽ tự động giải nén và liệt kê danh sách các test case.
4. **Cấu hình điểm số cho từng test case**:
   - Nhập số điểm tương ứng cho từng test case (ví dụ: mỗi test 10 điểm).
   - Nếu bạn bật tùy chọn **Partial points** (Điểm thành phần) trong trang soạn thảo bài tập ở bước 2, học sinh sẽ nhận được số điểm tương ứng với các test case chạy đúng, thay vì phải đúng toàn bộ mới được điểm.

---

## 4. Kiểm tra nộp bài

Sau khi hoàn tất cấu hình dữ liệu test, hãy quay lại trang bài tập và bấm nút **Submit solution** (Nộp bài giải) để nộp thử code mẫu của bạn xem trình chấm hoạt động chính xác hay không. Nếu có lỗi, bạn có thể quay lại giao diện chỉnh sửa test case bất cứ lúc nào.
