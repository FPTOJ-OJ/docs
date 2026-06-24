# Các loại Thể lệ Kỳ thi (Contest Formats)

FPTOJ hỗ trợ nhiều thể lệ tính điểm thi đấu khác nhau, phù hợp cho các kỳ thi tin học phổ biến trong nước và quốc tế.

---

## 1. Thể lệ Themis (Độc quyền FPTOJ)
- **Mô tả**: Được thiết kế riêng để giả lập hoàn toàn các kỳ thi chấm bằng công cụ **Themis** ngoại tuyến tại Việt Nam.
- **Tính điểm**: Hỗ trợ nộp bài thi bằng file ZIP chứa nhiều bài làm của học sinh. Có các tùy chọn khống chế chỉ cho phép nộp 1 lần duy nhất, tự động đăng ký dự thi, tài khoản dự phòng khi upload và chế độ thi thật ẩn điểm số.
- **Chi tiết cấu hình**: Xem thêm tại [Tài liệu Kỳ thi Themis](site/themis_contest.md).

---

## 2. Thể lệ Mặc định (Default)
- **Tính điểm**: Tổng điểm của thí sinh bằng tổng điểm lớn nhất đạt được của mỗi bài tập.
- **Phân định hòa (Tiebreak)**: Nếu bằng điểm, hệ thống tính tổng thời gian nộp bài của các bài có điểm lớn hơn 0.
- **Lưu ý**: *Mọi* lượt nộp bài (kể cả nộp sai hay nộp không thay đổi điểm) đều cộng thêm thời gian phạt (penalty time).

---

## 3. Thể lệ IOI (Olympic Tin học Quốc tế)
- **Tính điểm**: Tổng điểm là tổng điểm các bài tập. Với mỗi bài tập, điểm số được tính bằng tổng điểm lớn nhất của từng **Subtask (Bài tập con)** đạt được qua toàn bộ các lần nộp.
- **Ví dụ**: Bài tập có 2 Subtask. Lần nộp 1 đạt Subtask 1 (30đ), trượt Subtask 2 (10đ). Lần nộp 2 trượt Subtask 1 (0đ), đạt Subtask 2 (40đ). Điểm chung cuộc bài đó là 30 + 40 = 70 điểm.
- **Phân định hòa**: Mặc định thể lệ IOI không phân định hòa. Tuy nhiên, nếu cấu hình JSON `cumtime` là `true`, hệ thống sẽ tính tổng thời gian của lượt nộp đầu tiên vượt qua mỗi Subtask để phân định khi bằng điểm.

---

## 4. Thể lệ Legacy IOI (IOI kiểu cũ)
- **Tính điểm**: Điểm của bài là điểm cao nhất trong số các lần nộp của bài đó.
- **Phân định hòa**: Mặc định không phân định hòa. Nếu cấu hình `cumtime` là `true`, hệ thống tính tổng thời gian của lần nộp gần nhất làm thay đổi điểm số.

---

## 5. Thể lệ ICPC (Kỳ thi Lập trình Sinh viên Quốc tế)
- **Tính điểm**: Xếp hạng dựa trên **số lượng bài làm đúng (AC)** nhiều nhất.
- **Phân định hòa**: Bằng số bài thì tính tổng thời gian phạt (penalty). Thời gian phạt của một bài bằng thời gian từ lúc bắt đầu contest đến lúc nộp AC bài đó, cộng thêm 20 phút phạt cho mỗi lần nộp sai trước đó.
- **Cấu hình**: Có thể thay đổi số phút phạt thông qua trường `penalty` trong cấu hình JSON (mặc định là 20).

---

## 6. Thể lệ AtCoder
- **Tính điểm**: Tổng điểm của các bài nộp đạt điểm cao nhất.
- **Phân định hòa**: Tính thời gian nộp bài cuối cùng có điểm cao nhất cộng thêm thời gian phạt cho các lần nộp sai trước đó (mỗi lần phạt 5 phút).
- **Cấu hình**: Thay đổi thời gian phạt qua trường `penalty` trong cấu hình JSON (mặc định là 5 phút).

---

## 7. Thể lệ ECOO
- **Tính điểm**: Điểm số bằng điểm của lần nộp bài **cuối cùng** cho mỗi bài tập.
- **Cấu hình đặc biệt**:
  - `first_ac_bonus`: Điểm thưởng nếu giải đúng bài ngay từ lần nộp đầu tiên (mặc định 10 điểm).
  - `time_bonus`: Điểm thưởng khi nộp bài sớm trước khi kết thúc contest.
