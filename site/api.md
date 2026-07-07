# Hướng dẫn sử dụng API (v2)

FPTOJ hỗ trợ giao tiếp và truy xuất dữ liệu thông qua API định dạng JSON đơn giản, phục vụ cho việc tích hợp với các hệ thống bên ngoài hoặc ứng dụng di động.

---

## 1. Xác thực bằng API Token

Mỗi người dùng có thể tạo một API Token ngẫu nhiên trên trang **Chỉnh sửa hồ sơ** cá nhân. Các token này cho phép truy cập hầu hết dữ liệu công khai trên site dưới danh nghĩa tài khoản của bạn.

> [!CAUTION]  
> Trang quản trị `/admin` bị chặn hoàn toàn, không thể truy cập bằng API Token để đảm bảo an toàn.

Để sử dụng, hãy đính kèm Header HTTP sau trong mọi request (thay thế `<API_Token>` bằng mã token của bạn):

```http
Authorization: Bearer <API_Token>
```

### Các mã lỗi xác thực:
- `400 Invalid authorization header`: Header không đúng định dạng. Đảm bảo khớp regex: `Authorization: Bearer ([a-zA-Z0-9_-]{48})`
- `401 Invalid token`: Token không hợp lệ hoặc đã bị thu hồi.
- `403 Admin inaccessible`: Cố gắng truy cập vào các URL thuộc khu vực Admin.

---

## 2. Giới hạn tần suất gọi API (Rate Limiting)

Hệ thống giới hạn tối đa **90 yêu cầu mỗi phút** trên mỗi IP/Tài khoản. Nếu vượt quá giới hạn này, bạn sẽ nhận được thông báo lỗi hoặc yêu cầu xác thực Captcha.

---

## 3. Định dạng phản hồi chung (Response Format)

Tất cả phản hồi từ API đều có chung cấu trúc như sau:

```json
{
    "api_version": "2.0",
    "method": "GET",
    "fetched": "2026-06-24T22:00:00Z",
    "data": {},
    "error": null
}
```
*Lưu ý: Chỉ một trong hai trường `data` hoặc `error` xuất hiện trong phản hồi.*

### Định dạng lỗi (`error`):
```json
{
    "error": {
        "code": 404,
        "message": "Không tìm thấy tài nguyên yêu cầu"
    }
}
```

### Định dạng danh sách dữ liệu:
```json
{
    "data": {
        "current_object_count": 2,
        "objects_per_page": 50,
        "total_objects": 120,
        "page_index": 1,
        "total_pages": 3,
        "objects": [
            { "id": 1, "name": "..." }
        ]
    }
}
```

---

## 4. Các Endpoint chính

### 4.1 `/api/v2/contests` (Danh sách kỳ thi)
- **Bộ lọc cơ bản**: `tag`, `is_rated` (boolean)
- **Dữ liệu trả về**:
```json
{
    "key": "th1234",
    "name": "Kỳ thi thử THPT",
    "start_time": "2026-06-24T08:00:00Z",
    "end_time": "2026-06-24T11:00:00Z",
    "time_limit": 10800.0,
    "is_rated": false,
    "format": "themis"
}
```

### 4.2 `/api/v2/users` (Thông tin thành viên)
- **Bộ lọc**: `organization` (id), `role`
- **Dữ liệu trả về**:
```json
{
    "username": "hocsinh01",
    "display_rank": "Thí sinh",
    "rating": 1500,
    "points": 450.5
}
```

### 4.3 `/api/v2/problems` (Danh sách bài tập)
- **Bộ lọc**: `partial` (boolean), `points` (số điểm)
- **Dữ liệu trả về**:
```json
{
    "code": "aplusb",
    "name": "Tính tổng A + B",
    "points": 100,
    "time_limit": 1.0,
    "memory_limit": 65536,
    "is_public": true
}
```

### 4.4 `/api/v2/submissions` (Danh sách bài nộp)
- **Bộ lọc**: `user` (username), `problem` (code), `result` (AC/WA/TLE)
- **Dữ liệu trả về**:
```json
{
    "id": 12053,
    "user": "hocsinh01",
    "problem": "aplusb",
    "date": "2026-06-24T08:15:32Z",
    "time": 0.052,
    "memory": 4096,
    "points": 100.0,
    "language": "c++17",
    "status": "SC",
    "result": "AC"
}
```
