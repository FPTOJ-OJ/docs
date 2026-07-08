# Quản lý Đề Thi (Quiz / Exam)

Hệ thống hỗ trợ tạo và quản lý các đề thi trắc nghiệm với hai loại câu hỏi: **Trắc nghiệm (Multiple Choice)** và **Đúng/Sai (True/False Cluster)**. Có thể quản lý qua giao diện web hoặc CLI.

---

## Tổng quan

- **Exam (QuizSource)**: Một đề thi hoàn chỉnh với thời gian làm bài, câu hỏi, tag định hướng
- **Question (QuizQuestion)**: Câu hỏi thuộc đề thi, có thể loại choice hoặc tf
- **Option (QuizOption)**: Lựa chọn/câu trả lời của câu hỏi
- **Tag (QuizTag)**: Thẻ phân loại câu hỏi (ví dụ: KHMT, THUD)
- **Session (QuizSession)**: Phiên làm bài của người dùng

---

## 2. Quản lý qua CLI

Sử dụng lệnh `python manage.py quiz_cli` (xem chi tiết tại [CLI cho AI Agents](cli_for_agents.md)).

### Tạo exam

```bash
python manage.py quiz_cli exam create "Kiểm Tra 1 Tiết" \
    --description "Nội dung: Lập trình C++" \
    --duration 45 --visible --login
```

### Thêm câu hỏi trắc nghiệm

```bash
python manage.py quiz_cli exam question add 1 \
    --content "2 + 2 = ?" \
    --type choice --difficulty easy \
    --option 'A:"3":false' --option 'B:"4":true' \
    --option 'C:"5":false' --option 'D:"6":false' \
    --tag khmt
```

### Thêm câu hỏi Đúng/Sai

```bash
python manage.py quiz_cli exam question add 1 \
    --content "Các phát biểu sau đúng hay sai?" \
    --type tf --difficulty medium \
    --option 'a:"Phát biểu 1":true' \
    --option 'b:"Phát biểu 2":false' \
    --option 'c:"Phát biểu 3":true' \
    --option 'd:"Phát biểu 4":false' \
    --explanation "Giải thích chi tiết"
```

---

## 3. Hệ thống Định hướng (Orientation)

Hệ thống hỗ trợ **định hướng chuyên ngành** cho kỳ thi THPT:

| Tag slug | Tên | Nội dung |
|----------|-----|----------|
| `khmt` | Khoa học máy tính (CS) | Các câu hỏi về thuật toán, cấu trúc dữ liệu, lập trình |
| `thud` | Tin học ứng dụng (IT) | Các câu hỏi về mạng, CSDL, công nghệ thông tin |

Khi tham gia thi, thí sinh chọn 1 trong 2 định hướng và **không thể thay đổi** trong suốt bài thi. Chỉ các câu hỏi có tag phù hợp (hoặc tag `common`) mới được hiển thị và tính điểm.

---

## 4. Bulk Import từ JSON

### CLI

```bash
python manage.py quiz_cli bulk /path/to/exams.json
python manage.py quiz_cli bulk /path/to/exams.json --dry-run
```

### Cấu trúc JSON

```json
{
  "exams": [
    {
      "name": "Bài Kiểm Tra Giữa Kỳ",
      "description": "Kiến thức cơ bản về CS.",
      "duration": 60,
      "visible": false,
      "featured": false,
      "login": true,
      "tags": [
        {"name": "KHMT", "slug": "khmt"},
        {"name": "THUD", "slug": "thud"}
      ],
      "questions": [
        {
          "content": "Độ phức tạp của Quick Sort trong trường hợp xấu nhất là:",
          "type": "choice",
          "difficulty": "medium",
          "tags": ["khmt"],
          "options": [
            {"label": "A", "content": "O(n)", "is_correct": false},
            {"label": "B", "content": "O(n log n)", "is_correct": false},
            {"label": "C", "content": "O(n²)", "is_correct": true},
            {"label": "D", "content": "O(log n)", "is_correct": false}
          ]
        },
        {
          "content": "Xác định tính đúng/sai của các phát biểu:",
          "type": "tf",
          "difficulty": "easy",
          "tags": ["thud"],
          "options": [
            {"label": "a", "content": "TCP có kết nối.", "is_correct": true},
            {"label": "b", "content": "UDP đảm bảo thứ tự gói tin.", "is_correct": false}
          ]
        }
      ]
    }
  ]
}
```

---

## 5. Xem kết quả

- **Review**: Sau khi nộp bài, thí sinh xem lại kết quả từng câu
- **Điểm**: Câu đúng được điểm, câu sai 0 điểm (câu thuộc định hướng khác không tính)
- **Giải thích**: Nếu exam bật `show_explanations`, thí sinh thấy lời giải chi tiết

---

## 6. Giao diện Web

- `/quiz/` — Trang chủ quiz (luyện tập)
- `/quiz/manage/` — Quản lý câu hỏi
- `/quiz/manage/exams/` — Quản lý đề thi
- `/admin/judge/quizsource/` — Admin exam
- `/admin/judge/quizquestion/` — Admin questions
- `/admin/judge/quiztag/` — Admin tags
