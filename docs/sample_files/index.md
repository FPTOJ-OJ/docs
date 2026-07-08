# Mẫu cấu hình tham khảo

Dưới đây là các file cấu hình mẫu cho hệ thống FPTOJ. Bạn có thể xem nội dung
trực tiếp hoặc tải về để sử dụng làm template.

## Cấu hình Site

| File | Mô tả |
|---|---|
| [local_settings.py](local_settings.py) | Cấu hình Django chính (database, email, pdf, event, ...) |
| [site.conf](site.conf) | Supervisor config cho uWSGI site |
| [bridged.conf](bridged.conf) | Supervisor config cho Bridge server |
| [celery.conf](celery.conf) | Supervisor config cho Celery worker |
| [wsevent.conf](wsevent.conf) | Supervisor config cho WebSocket event server |
| [html-to-pdf-flask.conf](html-to-pdf-flask.conf) | Supervisor config cho dịch vụ xuất PDF |
| [uwsgi.ini](uwsgi.ini) | Cấu hình uWSGI workers |
| [nginx.conf](nginx.conf) | Cấu hình Nginx reverse proxy |

## Cấu hình Judge

| File | Mô tả |
|---|---|
| [judge_conf.yml](judge_conf.yml) | Cấu hình máy chấm (runtimes, problem paths) |

## Mẫu đề bài

| File | Mô tả |
|---|---|
| [problem_markdown_example.md.txt](problem_markdown_example.md.txt) | Mẫu Markdown đề bài đầy đủ tính năng (LaTeX, bảng, code) |

## Mẫu C++ (Validator, Checker, Interactor)

| File | Mô tả |
|---|---|
| [problem_setting/validator.cpp](problem_setting/validator.cpp) | Bộ xác thực dữ liệu đầu vào (Validator) |
| [problem_setting/identical_checker_interactor.cpp](problem_setting/identical_checker_interactor.cpp) | Checker so khớp tuyệt đối |
| [problem_setting/standard_checker_interactor.cpp](problem_setting/standard_checker_interactor.cpp) | Checker tiêu chuẩn (bỏ qua khoảng trắng thừa) |
