# Hướng dẫn sử dụng & Cài đặt FPTOJ (FPT Online Judge)

Chào mừng bạn đến với tài liệu hướng dẫn chính thức của **FPTOJ (FPT Online Judge)** - Hệ thống chấm bài lập trình tự động hiện đại, tối ưu cho giảng dạy và tổ chức các kỳ thi tin học tại Việt Nam.

Tài liệu này được biên soạn bằng tiếng Việt nhằm giúp các giáo viên, quản trị viên (kể cả các admin không chuyên về kỹ thuật - admin non) dễ dàng tự cài đặt, vận hành và quản lý hệ thống.

---

## 🎯 Các tính năng nổi bật của FPTOJ

1. **Giao diện hiện đại & cá nhân hóa**: Dễ dàng thay đổi logo, favicon, tên miền, cấu hình SEO và lời chào mừng trực tiếp từ bảng Admin mà không cần sửa code.
2. **Kỳ thi kiểu Themis độc quyền**: Hỗ trợ chấm offline/online kết hợp qua file zip nộp bài hàng loạt, ẩn điểm số khi thi (giống thi thật trên giấy), tự động đăng ký thí sinh và giới hạn nộp bài.
3. **Trình soạn thảo tối giản**: CKEditor được tinh giản, gọn gàng, giảm thiểu các nút bấm phức tạp gây rối giao diện.
4. **Xuất PDF độc quyền**: Sử dụng giải pháp `html-to-pdf-flask` nhẹ nhàng, hoạt động ổn định và chính xác hơn nhiều so với hệ thống cũ.

---

## 🛠️ Hướng dẫn nhanh cho người mới bắt đầu

Dưới đây là thứ tự các bước để xây dựng một hệ thống FPTOJ hoàn chỉnh:

1. **[Cài đặt hệ thống Site](site/installation.md)**: Thiết lập giao diện web, cơ sở dữ liệu MariaDB, Redis và các tiến trình nền.
2. **[Cài đặt Trình chấm (Judge)](judge/setting_up_a_judge.md)**: Thiết lập một hoặc nhiều máy chấm bài độc lập kết nối đến trang web chính.
3. **[Cấu hình html-to-pdf-flask](site/html-to-pdf-flask.md)**: Cài đặt dịch vụ xuất đề thi sang file PDF chất lượng cao.
4. **[Định dạng Contest Themis](site/themis_contest.md)**: Tìm hiểu cách giáo viên tạo và quản trị kỳ thi theo phong cách chấm Themis.
