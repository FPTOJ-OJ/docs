# Hệ thống Phân quyền (Permissions)

Hệ thống phân quyền của FPTOJ cực kỳ chi tiết, cho phép cấu hình linh hoạt quyền hạn của từng nhóm người dùng (Giáo viên, Admin, Học sinh).

Dưới đây là tài liệu chi tiết các quyền đặc biệt trên hệ thống. Các quyền cơ bản không được liệt kê ở đây sẽ tuân theo phân quyền mặc định của Django (`add`, `change`, `delete`, `view`).

---

## 1. Bài viết (Blog posts)

Theo mặc định của Django.

#### `edit_all_post` (Chỉnh sửa tất cả bài viết)
- **Quyền điều kiện**: `change_blogpost`
- Người dùng có thể xem và chỉnh sửa tất cả bài đăng blog trên trang Admin mà không cần phải là người viết bài đó.

---

## 2. Bình luận (Comments)

#### `override_comment_lock` (Bỏ qua khóa bình luận)
- Người dùng có thể đăng bình luận trong các bài viết hoặc bài tập đã bị khóa tính năng bình luận.

---

## 3. Kỳ thi (Contests)

#### `see_private_contest` (Xem kỳ thi riêng tư)
- Người dùng có thể xem các kỳ thi ẩn/riêng tư mà không cần có tên trong danh sách ban tổ chức.
- Họ cũng có thể xem bảng xếp hạng bị ẩn điểm số, nhưng **không có quyền** chỉnh sửa kỳ thi trong trang Admin.

#### `edit_own_contest` (Chỉnh sửa kỳ thi của tôi)
- Cho phép người dùng chỉnh sửa các kỳ thi trên trang Admin nếu họ được điền tên trong danh sách ban tổ chức (**authors** hoặc **curators**).

#### `edit_all_contest` (Chỉnh sửa tất cả kỳ thi)
- **Quyền điều kiện**: `edit_own_contest`
- **Quyền bao gồm**: `see_private_contest`
- Cho phép xem và chỉnh sửa toàn bộ các kỳ thi trên hệ thống mà không cần nằm trong ban tổ chức.

#### `clone_contest` (Sao chép kỳ thi)
- Cho phép nhân bản các kỳ thi có sẵn mà người đó có quyền chỉnh sửa.

#### `moss_contest` (Chạy phát hiện gian lận MOSS)
- Cho phép chạy công cụ phát hiện đạo code (MOSS) cho các kỳ thi được quyền quản lý.

#### `contest_rating` (Cập nhật Rating)
- Cho phép bật tính năng tính điểm Rating cho kỳ thi và thực hiện tính điểm/xóa điểm Rating sau khi kỳ thi kết thúc.

#### `contest_access_code` (Quản lý mã tham gia)
- Cho phép chỉnh sửa trường mã truy cập (`access_code`) của kỳ thi.

#### `create_private_contest` (Tạo kỳ thi riêng tư)
- Cho phép tạo các kỳ thi giới hạn cho lớp học, tổ chức hoặc tài khoản cá nhân cụ thể. Đồng thời cho phép thay đổi trạng thái ẩn/hiện của kỳ thi đó.

#### `change_contest_visibility` (Thay đổi ẩn/hiện kỳ thi)
- Cho phép thay đổi trạng thái ẩn/hiện công khai của tất cả các kỳ thi.

#### `contest_problem_label` (Sửa nhãn bài tập kỳ thi)
- Cho phép viết và sửa kịch bản LUA cấu hình nhãn cột bài tập trên bảng xếp hạng.

#### `lock_contest` (Khóa bài nộp kỳ thi)
- Cho phép khóa hoặc mở khóa tính năng chấm lại (rejudge) cho toàn bộ bài nộp trong kỳ thi.

---

## 4. Bài tập (Problems)

#### `see_organization_problem` (Xem bài tập lớp học)
- Cho phép xem các bài tập được cấu hình riêng tư cho tổ chức/lớp học.

#### `see_private_problem` (Xem bài tập ẩn)
- **Quyền bao gồm**: `see_organization_problem`
- Cho phép xem toàn bộ các bài tập bị ẩn trên hệ thống. Tuy nhiên, **không có quyền** chỉnh sửa hoặc xem mã nguồn bài nộp của học sinh khác.

#### `edit_own_problem` (Chỉnh sửa bài tập của tôi)
- Cho phép chỉnh sửa bài tập trong trang Admin nếu người đó là tác giả (**authors**) hoặc người kiểm duyệt (**curators**). Người dùng cũng xem được code bài nộp của học sinh ở các bài này.

#### `edit_public_problem` (Chỉnh sửa bài tập công khai)
- **Quyền điều kiện**: `edit_own_problem`
- Cho phép chỉnh sửa mọi bài tập đã được công khai trên hệ thống.

#### `edit_all_problem` (Chỉnh sửa tất cả bài tập)
- **Quyền điều kiện**: `edit_own_problem`
- **Quyền bao gồm**: `see_private_problem`, `edit_public_problem`, `view_all_submission`
- Cho phép xem và chỉnh sửa tất cả bài tập trên hệ thống bất kể tác giả là ai.

#### `problem_full_markup` (Sử dụng mã HTML đầy đủ)
- Cho phép sử dụng các thẻ HTML nâng cao bao gồm cả thẻ `<script>` và `<style>` trong mô tả đề bài. Người không có quyền này chỉ được soạn thảo bằng Markdown và các thẻ HTML an toàn bị giới hạn.

#### `clone_problem` (Sao chép bài tập)
- Cho phép sao chép/nhân bản bài tập đã có.

#### `change_public_visibility` (Thay đổi công khai bài tập)
- Cho phép tích chọn hoặc bỏ chọn trường `is_public` (công khai bài tập ra trang chủ).

---

## 5. Bài nộp (Submissions)

#### `abort_any_submission` (Hủy chấm bài bất kỳ)
- Cho phép dừng tiến trình chấm của bất kỳ bài nộp nào đang chạy. Nếu không có quyền này, học sinh chỉ có thể tự hủy bài nộp của chính mình khi máy chấm đang chờ.

#### `rejudge_submission` (Chấm lại bài nộp)
- **Quyền điều kiện**: `edit_own_problem`
- Cho phép yêu cầu máy chấm chấm lại các bài nộp thuộc bài tập mà họ có quyền quản lý.

#### `rejudge_submission_lot` (Chấm lại hàng loạt)
- **Quyền điều kiện**: `rejudge_submission`
- Cho phép gửi yêu cầu chấm lại số lượng lớn bài nộp cùng lúc mà không bị giới hạn bởi cấu hình tần suất tối đa.

#### `spam_submission` (Nộp bài không giới hạn)
- Cho phép nộp bài liên tục mà không bị giới hạn thời gian chờ giữa 2 lần nộp (Spam protection).

#### `view_all_submission` (Xem tất cả mã nguồn bài nộp)
- Cho phép xem mã nguồn bài làm của toàn bộ thí sinh trên toàn hệ thống nhưng không có quyền chỉnh sửa dữ liệu bài tập đó.

#### `resubmit_other` (Nộp lại bài của người khác)
- Cho phép lấy code của người khác và gửi yêu cầu nộp lại dưới tên họ.

#### `lock_submission` (Khóa bài nộp)
- Cho phép khóa kết quả chấm của một bài nộp. Bài nộp đã bị khóa sẽ không thể bị chấm lại bởi bất kỳ ai (kể cả admin tối cao) cho đến khi được mở khóa.
