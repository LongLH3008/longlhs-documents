[← Mục lục](./index.md)

# Biến, Hằng & Input / Output

## Định nghĩa

- **Biến (variable)** trong Python là một **tên (name)** được gắn (bind) vào một **object** trong bộ nhớ bằng phép gán `=`. Python là ngôn ngữ **dynamically typed**: kiểu thuộc về object, không thuộc về tên, nên cùng một tên có thể lần lượt trỏ tới object khác kiểu. Không có từ khóa khai báo (`let`, `var`, `const`).
- **Hằng (constant)**: Python **không có** hằng thật sự. Quy ước (PEP 8) đặt tên `UPPER_SNAKE_CASE` để báo "đừng gán lại". Có thể dùng `typing.Final` để type checker cảnh báo.
- **Output**: hàm built-in `print(*objects, sep=" ", end="\n", file=sys.stdout, flush=False)` ghi ra standard output.
- **Input**: hàm built-in `input(prompt="")` đọc **một dòng** từ standard input và **luôn trả về `str`** (đã bỏ ký tự xuống dòng).

## Giải thích đơn giản

Biến Python giống **tờ giấy note dán lên một chiếc hộp**, không phải chiếc hộp. `x = 5` là dán nhãn "x" lên hộp chứa số 5. `x = "hi"` là gỡ nhãn ra dán sang hộp khác. Hai nhãn có thể dán lên cùng một hộp (xem [Name Binding](./49-Name-Binding.md)).

`input()` giống **nhân viên lễ tân chỉ biết ghi chữ**: bạn nói "25" thì họ ghi chuỗi `"25"`, muốn tính toán phải tự đổi sang số bằng `int()`.

## Ví dụ

### Gán biến

```python
name = "An"            # str
age = 25               # int
height = 1.72          # float
is_student = False     # bool
nothing = None         # "không có giá trị", giống null

age = "hai mươi lăm"   # hợp lệ: tên age giờ trỏ tới str

# Gán nhiều biến
x, y, z = 1, 2, 3
a = b = 0              # cùng trỏ tới object 0
x, y = y, x            # hoán đổi không cần biến tạm
```

### Quy tắc đặt tên

```python
user_name = "An"       # snake_case cho biến, hàm
MAX_RETRY = 3          # UPPER_SNAKE_CASE cho hằng
class UserProfile: ... # PascalCase cho class
_internal = 1          # tiền tố _: dùng nội bộ

# Không hợp lệ: 2name, user-name, class (từ khóa)
# Phân biệt hoa thường: Name và name là hai biến khác nhau
```

### Hằng với Final

```python
from typing import Final

API_URL: Final = "https://api.example.com"
API_URL = "x"     # chạy vẫn được, nhưng mypy/pyright báo lỗi
```

### Xóa biến & kiểm tra kiểu

```python
x = 10
print(type(x), id(x))     # <class 'int'> 140...  (id = định danh object)
del x                     # gỡ tên x
print(x)                  # NameError: name 'x' is not defined
```

### print

```python
print("Xin chào", "An")                 # Xin chào An
print("a", "b", "c", sep="-")           # a-b-c
print("Đang tải", end="...")            # không xuống dòng
print("xong")                           # Đang tải...xong

name, score = "An", 9.25
print(f"{name} được {score:.1f} điểm")  # An được 9.2 điểm

import sys
print("Lỗi!", file=sys.stderr)          # ghi ra stderr
```

### input

```python
name = input("Tên bạn: ")           # luôn là str
age = int(input("Tuổi: "))          # tự ép kiểu, nhập "abc" -> ValueError
print(f"Chào {name}, năm sau bạn {age + 1} tuổi")

# Đọc nhiều số trên một dòng: "3 5 7"
a, b, c = map(int, input().split())
```

### Nhập an toàn

```python
while True:
    raw = input("Nhập tuổi: ")
    try:
        age = int(raw)
        break
    except ValueError:
        print("Phải là số nguyên, nhập lại.")
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `let x = 1` | `x = 1` |
| `const PI = 3.14` | `PI = 3.14` (quy ước) hoặc `PI: Final = 3.14` |
| `null`, `undefined` | `None` (chỉ có một) |
| `typeof x` | `type(x)` |
| `console.log(a, b)` | `print(a, b)` |
| `console.error(x)` | `print(x, file=sys.stderr)` |
| `prompt("Tên?")` (browser) | `input("Tên? ")` |
| `[a, b] = [b, a]` | `a, b = b, a` |
| `camelCase` | `snake_case` |

## Xem thêm

- [Name Binding (không có Hoisting)](./49-Name-Binding.md)
- [Data Type](./17-Data-Type.md)
- [f-string](./30-f-string.md)
- [PEP 8 & Coding Style](./07-PEP8-Style.md)
- [Scope (LEGB)](./48-Scope-LEGB.md)
- [Exception](./37-Exception.md)
