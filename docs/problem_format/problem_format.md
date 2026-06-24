# Định dạng Bài tập và File cấu hình init.yml

Trong FPTOJ, mỗi bài tập được lưu trữ độc lập trong một thư mục riêng. Thư mục này bắt buộc phải chứa một file cấu hình tên là `init.yml`.

---

## 1. File cấu hình `init.yml`

File này sử dụng cú pháp YAML và phải chứa khóa bắt buộc: `test_cases`.
- **test_cases**: Khai báo danh sách các test case cụ thể, hoặc định nghĩa biểu thức chính quy (regex) để hệ thống tự động quét và phân loại file test.
- **archive**: Khóa tùy chọn (khuyên dùng). Cho phép nén toàn bộ thư mục dữ liệu test thành một file `.zip` (ví dụ: `aplusb.zip`) để lưu trữ gọn nhẹ, thay vì để các file text phẳng trực tiếp trong thư mục.

---

## 2. Cách cấu hình `test_cases`

Có hai cách để khai báo danh sách test case:

---

### Cách 1: Khai báo thủ công từng test case (List method)

Chúng ta liệt kê chi tiết từng file dữ liệu đầu vào (`in`), đầu ra (`out`) và điểm số (`points`) tương ứng.

```yaml
archive: aplusb.zip
test_cases:
- {points: 0, in: aplusb.p0.in, out: aplusb.p0.out}  # Test ví dụ (0 điểm)
- {points: 10, in: aplusb.1.in, out: aplusb.1.out}
- {points: 10, in: aplusb.2.in, out: aplusb.2.out}
```

> [!TIP]  
> Các test case có `points: 0` thường được dùng làm **Pretest (hoặc test ví dụ)**. Nếu bài làm chạy sai ở test 0 điểm này, máy chấm sẽ dừng ngay lập tức (Short-circuit) và không chấm tiếp các test sau để tiết kiệm thời gian. Do đó, hãy xếp các test 0 điểm lên đầu tiên.

#### Test theo nhóm (Batched cases):
Cho phép gộp nhiều test case thành một nhóm (Subtask). Thí sinh phải trả lời đúng **tất cả** các test trong nhóm mới nhận được điểm của nhóm đó.

```yaml
test_cases:
- points: 20
  batched:
  - {in: sub1.1.in, out: sub1.1.out}
  - {in: sub1.2.in, out: sub1.2.out}
- points: 30
  batched:
  - {in: sub2.1.in, out: sub2.1.out}
  - {in: sub2.2.in, out: sub2.2.out}
  dependencies: [1]  # Nhóm này chỉ chấm nếu Nhóm 1 đã đạt điểm tối đa
```
*Trường `dependencies` định nghĩa các nhóm phụ thuộc. Nhóm sau chỉ chấm khi các nhóm đứng trước (khai báo dạng index 1, 2,...) đạt điểm tối đa.*

---

### Cách 2: Tự động quét bằng Biểu thức chính quy (Regex method)

Nếu bộ test case của bạn được đặt tên theo quy luật chuẩn, bạn có thể dùng regex để máy chấm tự động nhận diện, giảm công sức viết file `init.yml`.

Mặc định máy chấm quét các file đầu vào có đuôi `.in` hoặc `.inp` và đầu ra là `.out` hoặc `.ans`.

Ví dụ cấu hình tự động quét:
```yaml
archive: data.zip
test_cases:
  # Điểm số cho mỗi test case quét được
  case_points: 10
```

Các định dạng tên file tương thích tốt bao gồm:
- `test.1.in` / `test.1.out`
- `test-case-1.in` / `test-case-1.out`
- `test-batch-1-case-2.in` (Chấm theo nhóm: Nhóm 1 - Test 2)
