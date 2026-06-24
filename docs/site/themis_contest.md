# Định dạng Contest kiểu Themis (Độc quyền FPTOJ)

FPTOJ được tích hợp tính năng quản lý và tổ chức kỳ thi theo phong cách **Themis** - một công cụ chấm thi cực kỳ phổ biến tại các trường THPT và kỳ thi Học sinh giỏi ở Việt Nam. Tính năng này giúp giáo viên dễ dàng chuyển đổi từ hình thức chấm thi offline bằng Themis sang chấm thi online tự động mà không thay đổi thói quen tổ chức thi.

---

## 1. Cách tạo kỳ thi (Contest) kiểu Themis nhanh chóng

Thay vì tạo thủ công từng bài tập và liên kết vào contest rất mất thời gian, FPTOJ cung cấp giao diện **Tạo Contest Themis tự động** từ File Zip dữ liệu đề.

### Các bước thực hiện:
1. Đăng nhập trang quản trị Admin (`/admin/`).
2. Vào mục **Contests (Kỳ thi)**.
3. Bấm nút **"Create Themis Contest"** ở góc trên bên phải trang danh sách.
4. Điền các thông tin:
   - **Tên kỳ thi**: Tên cuộc thi hiển thị trên trang chủ.
   - **Thời gian bắt đầu / Kết thúc**: Cấu hình thời gian chạy contest.
   - **Vô hạn thời gian (Infinite Time)**: Đánh dấu nếu muốn đây là một contest luyện tập không giới hạn thời gian.
   - **Chế độ I/O đầy đủ (Full I/O Mode)**: Tự động cấu hình đọc ghi file cho toàn bộ bài tập (ví dụ: file nguồn C++ phải đọc từ `<PROBLEM>.INP` và xuất ra `<PROBLEM>.OUT`).
   - **Chỉ nộp 1 lần duy nhất (One-time Submission)**: Học sinh chỉ được upload bài làm duy nhất 1 lần (không được nộp đè nhiều lần).
   - **Ẩn điểm số với học sinh (Hide Scores)**: Kỳ thi thật, học sinh nộp bài chỉ biết trạng thái đã nộp thành công, không thấy điểm số hay chi tiết các test case. Điểm số chỉ có Admin/Giáo viên nhìn thấy.
   - **File ZIP dữ liệu test**: Nén thư mục chứa các bài thi dưới cấu trúc chuẩn Themis và upload lên.

---

## 2. Cấu trúc chuẩn của File ZIP dữ liệu test

File ZIP upload lên trang tạo contest phải tuân thủ cấu trúc thư mục sau:

```
[Ten_Contest].zip
├── Bài 1 (Ví dụ: SUM)
│   ├── Test01
│   │   ├── SUM.INP
│   │   └── SUM.OUT (hoặc SUM.ANS)
│   ├── Test02
│   │   ├── SUM.INP
│   │   └── SUM.OUT
│   ...
└── Bài 2 (Ví dụ: SEQUENCE)
    ├── Test01
    │   ├── SEQUENCE.INP
    │   └── SEQUENCE.OUT (hoặc SEQUENCE.ANS)
    ...
```

*Lưu ý:* Hệ thống hỗ trợ tự động tìm kiếm và nhận diện cấu trúc thư mục test, hỗ trợ cả định dạng đuôi viết thường `.inp`, `.out`, `.in`, `.ans`.

---

## 3. Các tính năng độc quyền dành cho giáo viên và học sinh

### 3.1. Chức năng nộp bài hộ / Chấm hàng loạt (Bulk Upload)
Trong giao diện contest phía người dùng, Giáo viên/Quản trị viên có chức năng **Upload zip bài làm của học sinh**. Cấu trúc file zip nộp bài:
```
Submissions.zip
├── [Username_HocSinh_1]
│   ├── SUM.cpp
│   └── SEQUENCE.pas
├── [Username_HocSinh_2]
│   ├── SUM.py
│   └── SEQUENCE.cpp
```
Khi giáo viên upload file zip này lên, hệ thống sẽ tự động bóc tách và chấm điểm cho từng học sinh tương ứng.

### 3.2. Cơ chế tài khoản dự phòng tự động (Student Fallback)
Khi giáo viên nộp bài hộ học sinh thông qua file zip chấm hàng loạt:
- Nếu thư mục mang tên học sinh `hocsinh_01` nhưng tài khoản này **không tồn tại** trên hệ thống FPTOJ: Hệ thống sẽ **không báo lỗi**, thay vào đó sẽ tự động đứng tên chạy dưới tài khoản của giáo viên (Admin) đang thực hiện upload để tránh việc dừng tiến trình chấm.
- Hệ thống sẽ hiển thị cảnh báo trên màn hình console logs:
  `!! Student [hocsinh_01] not found in database. Submitted under admin [kien] instead.`

### 3.3. Tự động đăng ký tham gia (Auto-Registration)
Khi nộp bài hộ, học sinh chưa đăng ký tham gia Contest đó sẽ được hệ thống tự động đăng ký tham gia (Contest Participation) mà không cần giáo viên phải duyệt hoặc học sinh phải bấm nút đăng ký trước đó.

### 3.4. Giới hạn nộp bài 1 lần (One-time Submission Limit)
Nếu bật cấu hình này, khi học sinh thực hiện nộp bài (upload zip), hệ thống sẽ lưu vết thời gian. Kể từ lần nộp đầu tiên quá 120 giây (thời gian bù trừ độ trễ mạng), mọi lượt nộp bài sau đó đều bị chặn nhằm đảm bảo tính trung thực như thi thật trên giấy.

### 3.5. Thi thật ẩn điểm số (Real Exam / Hidden Scores)
Để tái lập kỳ thi quốc gia hoặc kỳ thi học sinh giỏi thực tế:
- **Bảng xếp hạng (Scoreboard)**: Hiển thị ký tự `?` ở cột điểm và tổng điểm của các thí sinh khác đối với học sinh. Giáo viên và Admin vẫn xem được điểm đầy đủ theo thời gian thực.
- **Danh sách bài nộp**: Ẩn các tag trạng thái (AC, WA, TLE, RTE), ẩn số điểm, CPU time, bộ nhớ sử dụng. Chỉ hiển thị trạng thái đơn giản là `Đã chấm` (Graded) hoặc `Đang chấm` (Grading).
- **Chi tiết bài nộp (Submission Detail)**: Khi học sinh nhấn xem chi tiết bài nộp của mình, hệ thống sẽ ẩn toàn bộ danh sách test case và hiển thị thông báo:
  `"Bài nộp của bạn đã được chấm xong. Kết quả chi tiết được ẩn đi cho đến khi kỳ thi kết thúc."`
- **Màn hình Console logs**: Ẩn điểm số cập nhật thời gian thực khi đang chấm bài.
