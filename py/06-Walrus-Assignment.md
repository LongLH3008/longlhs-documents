[← Mục lục](./index.md)

# Walrus Operator & Assignment

## Định nghĩa

- **Assignment statement** (`=`) gán giá trị cho tên biến. Đây là **câu lệnh** (statement), không trả về giá trị, nên không thể đặt trong `if` hay `while`.
- **Augmented assignment** (`+=`, `-=`, `*=`, `//=`, `**=`...) là viết tắt của `x = x op y`.
- **Walrus operator** (`:=`, assignment expression, PEP 572, Python 3.8+) gán giá trị **và trả về giá trị đó** trong cùng một biểu thức.
- **Multiple / chained assignment**: `a, b = 1, 2` và `a = b = 0`.

## Giải thích đơn giản

Trong JS, `if ((m = re.exec(s)))` là hợp lệ vì phép gán cũng là biểu thức. Python cố tình cấm `=` trong điều kiện để tránh lỗi gõ nhầm `=` thay `==`. Khi thật sự cần "vừa gán vừa dùng", Python cho bạn `:=`, nhìn giống mắt và ngà con hải mã (walrus) nên có tên như vậy.

Python **không có** `&&=`, `||=`, `??=` như JS. Muốn đặt giá trị mặc định thì dùng `x = x or default` hoặc kiểm tra `is None`.

## Ví dụ

### Gán nhiều biến, hoán đổi

```python
a, b = 1, 2
a, b = b, a        # swap không cần biến tạm
print(a, b)        # 2 1

x = y = 0          # chained assignment
count = 10
count += 5         # 15
count //= 2        # 7
```

### Walrus trong while

```python
# Không có walrus
line = input()
while line != "quit":
    print("Bạn gõ:", line)
    line = input()

# Có walrus: gọn hơn, không lặp code
while (line := input()) != "quit":
    print("Bạn gõ:", line)
```

### Walrus trong if và comprehension

```python
import re

if (m := re.search(r"\d+", "order 123")):
    print(m.group())   # 123

data = [1, 5, 12, 30]
# Tính f(x) một lần, vừa lọc vừa dùng kết quả
results = [y for x in data if (y := x * x) > 20]
print(results)   # [25, 144, 900]
```

### Thay cho logical assignment của JS

```python
config = {}

# JS: config.port ??= 3000
if config.get("port") is None:
    config["port"] = 3000

# Hoặc với dict
config.setdefault("host", "localhost")
print(config)  # {'port': 3000, 'host': 'localhost'}

name = ""
name = name or "Ẩn danh"   # JS: name ||= "Ẩn danh"
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `if ((x = f()))` | `if (x := f()):` |
| `[a, b] = [b, a]` | `a, b = b, a` |
| `a \|\|= b` | `a = a or b` |
| `a &&= b` | `a = a and b` |
| `a ??= b` | `if a is None: a = b` |
| `obj.k ??= v` | `d.setdefault("k", v)` |

## Xem thêm

- [Logical Assignment Operators (JS)](../js/05-Logical-Assignment-Operators.md)
- [Toán tử (Operators)](./05-Operators.md)
- [Unpacking](./32-Unpacking.md)
