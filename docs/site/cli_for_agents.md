# CLI cho AI Agents

Tài liệu này mô tả các lệnh CLI quản trị để quản lý bài tập, testcase, dữ liệu, đề thi và người dùng.

---

## Ưu tiên: Dùng Generator-Based Testcases

Khi tạo problem, ưu tiên dùng generator script (C++ hoặc Python) để sinh testcase thay vì tải file `.in`/`.out`.

```cpp
#include <bits/stdc++.h>
using namespace std;
int main(int argc, char* argv[]) {
    int n = atoi(argv[1]);
    int max_val = atoi(argv[2]);
    srand(time(0));
    cout << n << " " << rand() % max_val << endl;
    return 0;
}
```

---

## Problem Management CLI (`problem_cli`)

```bash
python manage.py problem_cli <subcommand> [options]
```

### List / View

```bash
python manage.py problem_cli list
python manage.py problem_cli list --public --private --type dp --group usaco
python manage.py problem_cli list --org my-school --search "hello" --json
python manage.py problem_cli detail <problem_code>
```

### CRUD

```bash
python manage.py problem_cli create <code> <name> --type <type_id> --group <group_id> \
    --description "..." --points 10 --time-limit 1.0 --memory-limit 65536 \
    --public --author "admin" --org-private

python manage.py problem_cli update <code> --name "New Title" --points 20
python manage.py problem_cli delete <code> --force
```

### Testcases (Generator-Based — Ưu tiên)

```bash
python manage.py problem_cli testcase add <code> --generator-args "5 10" --points 2
python manage.py problem_cli testcase add <code> --input test.in --output test.out --points 1
python manage.py problem_cli testcase add <code> --type S --points 10
python manage.py problem_cli testcase add <code> --type E
python manage.py problem_cli testcase delete <code> <testcase_id>
```

### Data Files & Compile

```bash
python manage.py problem_cli data show <code>
python manage.py problem_cli data upload-zip <code> /path/to/data.zip
python manage.py problem_cli data upload-generator <code> /path/to/gen.py
python manage.py problem_cli data compile <code>
```

### Types, Groups, Authors

```bash
python manage.py problem_cli type list --json
python manage.py problem_cli type add dp "Dynamic Programming"
python manage.py problem_cli group add usaco "USACO"
python manage.py problem_cli author add <code> <username>
python manage.py problem_cli author remove <code> <username>
```

---

## Quiz/Exam Management CLI (`quiz_cli`)

```bash
python manage.py quiz_cli <subcommand> [options]
```

### Exams

```bash
python manage.py quiz_cli exam list --visible --featured --locked --search "name" --json
python manage.py quiz_cli exam detail <exam_id>
python manage.py quiz_cli exam create "Exam Name" --description "..." --duration 60 \
    --visible --featured --login --locked --org-only --author admin
python manage.py quiz_cli exam update <id> --name "New Name" --duration 90
python manage.py quiz_cli exam delete <id> --force
```

### Questions

```bash
python manage.py quiz_cli exam question list <exam_id> --json
python manage.py quiz_cli exam question add <exam_id> --content "What is 2+2?" \
    --type choice --difficulty easy \
    --option 'A:"3":false' --option 'B:"4":true' \
    --option 'C:"5":false' --option 'D:"6":false' --tag khmt
python manage.py quiz_cli exam question add <exam_id> --content "Which are correct?" \
    --type tf --difficulty medium \
    --option 'a:"Statement 1":true' --option 'b:"Statement 2":false' --tag thud \
    --explanation "Explanation text"
python manage.py quiz_cli exam question update <qid> --content "New text"
python manage.py quiz_cli exam question remove <exam_id> <qid>
python manage.py quiz_cli exam question delete <qid> --force
```

### Tags

```bash
python manage.py quiz_cli tag list --json
python manage.py quiz_cli tag add "KHMT" khmt
python manage.py quiz_cli tag delete <slug>
```

### Bulk Import

```bash
python manage.py quiz_cli bulk <json_file> --dry-run --json
python manage.py problem_cli bulk <json_file> --dry-run --json
```

---

## User Management CLI (`user_cli`)

```bash
python manage.py user_cli <subcommand> [options]
```

### List / View

```bash
python manage.py user_cli list --staff --superuser --setter --admin
python manage.py user_cli list --active --search "kien" --json
python manage.py user_cli detail <username>
```

### Create / Promote / Demote

```bash
python manage.py user_cli create <username> --email user@example.com --password "secret123"
python manage.py user_cli create <username> --setter --staff
python manage.py user_cli promote <username> --staff --superuser --setter --admin-rank
python manage.py user_cli demote <username> --staff --superuser
```

### Activate / Deactivate

```bash
python manage.py user_cli activate <username>
python manage.py user_cli deactivate <username>
```

### Vai trò

| Role | is_staff | is_superuser | display_rank | Ý nghĩa |
|------|----------|-------------|--------------|---------|
| **Superuser** | ✓ | ✓ | admin | Toàn quyền |
| **Staff** | ✓ | ✗ | admin/setter | Vào admin, có quyền được cấp |
| **Problem Setter** | ✗ | ✗ | setter | Giáo viên, tạo/quản lý đề thi |
| **Normal User** | ✗ | ✗ | user | Người dùng thường |

---

## Seed & Translations

```bash
python manage.py seed_quiz_sample --force
python manage.py compilemessages -l vi
python manage.py makemessages -l vi
```

---

## Quick Reference

| Task | Command |
|------|---------|
| List problems | `python manage.py problem_cli list` |
| Create problem | `python manage.py problem_cli create <code> <name> --type X --group Y` |
| Add testcase (generator) | `python manage.py problem_cli testcase add <code> --generator-args "..." --points N` |
| List exams | `python manage.py quiz_cli exam list` |
| Add question (MC) | `python manage.py quiz_cli exam question add <id> --type choice --option 'A:"A":true'` |
| Add question (TF) | `python manage.py quiz_cli exam question add <id> --type tf --option 'a:"S":true'` |
| List users | `python manage.py user_cli list` |
| Create user | `python manage.py user_cli create <username> --email ...` |
| Promote (staff) | `python manage.py user_cli promote <username> --staff` |
| Bulk import problems | `python manage.py problem_cli bulk <json_file>` |
| Bulk import exams | `python manage.py quiz_cli bulk <json_file>` |
