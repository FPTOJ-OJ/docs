# Định dạng Bài tập và File cấu hình init.yml

Trong FPTOJ, mỗi bài tập được lưu trữ độc lập trong một thư mục riêng. Thư mục này bắt buộc phải chứa một file cấu hình tên là `init.yml`.

---

## 1. File cấu hình `init.yml`

File này sử dụng cú pháp YAML và phải chứa khóa bắt buộc: `test_cases`.

### Các trường cơ bản

| Trường | Bắt buộc | Mô tả |
|---|---|---|
| `test_cases` | **Có** | Danh sách các test case hoặc định nghĩa biểu thức chính quy (regex) để tự động quét |
| `archive` | Không (khuyên dùng) | Nén toàn bộ dữ liệu test thành file `.zip` gọn nhẹ |
| `io_mode` | Không (mặc định `std`) | Chế độ nhập/xuất dữ liệu: `std` (stdin/stdout) hoặc `file` (đọc/ghi file) |
| `input_file` | Khi `io_mode: file` | Tên file đầu vào mà bài làm của thí sinh sẽ đọc |
| `output_file` | Khi `io_mode: file` | Tên file đầu ra mà bài làm của thí sinh sẽ ghi |

### Chế độ I/O (io_mode)

FPTOJ hỗ trợ hai chế độ nhập/xuất dữ liệu:

#### `io_mode: std` (Mặc định)

Bài làm của thí sinh đọc dữ liệu từ **stdin** và ghi kết quả ra **stdout**.
Đây là chế độ phổ biến nhất, tương thích với mọi ngôn ngữ lập trình.

```yaml
archive: aplusb.zip
io_mode: std
test_cases:
- {in: aplusb.1.in, out: aplusb.1.out, points: 10}
```

#### `io_mode: file`

Bài làm của thí sinh đọc dữ liệu từ một file cụ thể (vd: `BAI1.INP`) và ghi kết
quả ra một file khác (vd: `BAI1.OUT`). Thường dùng trong các kỳ thi Olympic
Tin học để tương thích với bộ test có sẵn.

Khi dùng `io_mode: file`, bạn **bắt buộc** phải khai báo thêm hai trường:

- **`input_file`**: Tên file đầu vào (vd: `BAI1.INP`).
- **`output_file`**: Tên file đầu ra (vd: `BAI1.OUT`).

```yaml
archive: bai1.zip
io_mode: file
input_file: BAI1.INP
output_file: BAI1.OUT
test_cases:
- {in: test1/bai1.inp, out: test1/bai1.out, points: 10}
- {in: test2/bai1.inp, out: test2/bai1.out, points: 20}
```

> [!TIP]
> Với `io_mode: file`, tên file trong các trường `in`/`out` của `test_cases`
> là đường dẫn bên trong file ZIP (dùng để so sánh kết quả), khác với tên file
> `input_file`/`output_file` mà thí sinh dùng trong code.

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
