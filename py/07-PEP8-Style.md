[← Mục lục](./index.md)

# PEP 8 & Coding Style

## Định nghĩa

- **PEP** (Python Enhancement Proposal) là tài liệu đề xuất/chuẩn hóa của cộng đồng Python.
- **PEP 8** là chuẩn coding style chính thức: cách đặt tên, thụt lề, độ dài dòng, khoảng trắng, thứ tự import.
- **PEP 20 (The Zen of Python)** là 19 nguyên tắc triết lý, xem bằng `import this`.
- Trong Python, **thụt lề (indentation) là cú pháp**: nó xác định block code thay cho `{}`.

## Giải thích đơn giản

JS có `"use strict"` để bật chế độ khắt khe khi chạy. Python không cần thứ đó vì bản thân ngôn ngữ đã khá "nghiêm": không ép kiểu ngầm, dùng biến chưa gán là lỗi ngay. Thứ Python "bắt buộc" là **thụt lề**: thụt sai là code chạy sai hoặc lỗi `IndentationError`.

PEP 8 giống **luật giao thông chung**: không tuân thủ thì code vẫn chạy, nhưng người khác đọc rất mệt. Tool như **black** / **ruff** tự format giống Prettier / ESLint.

## Ví dụ

### Indentation là cú pháp

```python
def greet(name):
    if name:
        print(f"Xin chào {name}")   # 4 dấu cách, không dùng tab
    else:
        print("Xin chào bạn")
    print("Luôn chạy")              # cùng cấp với if -> nằm ngoài if
```

### Quy tắc đặt tên

```python
MAX_RETRY = 3                 # hằng số: UPPER_SNAKE_CASE

user_name = "long"            # biến, hàm: snake_case (JS dùng camelCase)

def get_user_by_id(user_id):  # hàm: snake_case
    ...

class UserService:            # class: PascalCase
    def _internal(self):      # 1 dấu _ đầu: "private" theo quy ước
        ...

    def __mangled(self):      # 2 dấu _ đầu: name mangling
        ...
```

### Thứ tự import

```python
# 1. Thư viện chuẩn
import os
import sys

# 2. Thư viện bên thứ ba
import requests

# 3. Module trong project
from app.services import user_service
```

### Zen of Python (trích)

```python
import this
# Beautiful is better than ugly.
# Explicit is better than implicit.
# Simple is better than complex.
# Readability counts.
# There should be one-- and preferably only one --obvious way to do it.
```

### Tool tự động

```bash
pip install ruff black
ruff check .      # lint, giống eslint
black .           # format, giống prettier
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `{ ... }` bao block | thụt lề 4 space + `:` |
| `;` cuối câu (tùy chọn) | không dùng |
| `camelCase` cho biến/hàm | `snake_case` |
| `"use strict"` | không cần, ngôn ngữ đã nghiêm |
| `#private` field | `_name` (quy ước), `__name` (mangling) |
| ESLint / Prettier | ruff / black |
| `// comment` | `# comment` |

## Xem thêm

- [Strict Mode (JS)](../js/06-Strict-Mode.md)
- [Type Hints](./41-Type-Hints.md)
- [Class](./39-Class.md)
