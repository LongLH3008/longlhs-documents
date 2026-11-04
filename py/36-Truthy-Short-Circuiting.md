[← Mục lục](./index.md)

# Truthy, Falsy & Short Circuiting

## Định nghĩa

- **Truth value testing**: mọi object đều có giá trị đúng/sai khi đặt trong ngữ cảnh boolean (`if`, `while`, `and`, `or`, `not`, `bool()`).
- Các giá trị **falsy**: `False`, `None`, `0`, `0.0`, `0j`, `Decimal(0)`, `""`, `[]`, `()`, `{}`, `set()`, `range(0)`, và object có `__bool__()` trả `False` hoặc `__len__()` trả `0`. Mọi thứ khác là **truthy**.
- **Short-circuit evaluation**: `and` và `or` dừng đánh giá ngay khi biết kết quả và **trả về một trong hai toán hạng** (không ép thành `bool`).
  - `a or b`: trả `a` nếu `a` truthy, ngược lại trả `b`.
  - `a and b`: trả `a` nếu `a` falsy, ngược lại trả `b`.

## Giải thích đơn giản

Cơ chế giống hệt `||` và `&&` của JS. Khác biệt cần nhớ:

- Trong Python, **list rỗng `[]` và dict rỗng `{}` là falsy**. Trong JS, `[]` và `{}` là truthy. Vì vậy Python viết `if items:` để kiểm tra "có phần tử không".
- Python **không có `??`** (nullish coalescing). `x or default` sẽ thay cả `0` và `""` bằng default, giống `||` của JS. Muốn chỉ thay `None`, phải viết tường minh.

Ví von: `or` giống **người bảo vệ tìm người đầu tiên có vé**: thấy ai có vé (truthy) là cho vào luôn, không xét người sau.

## Ví dụ

### Truthy / falsy

```python
for v in [0, 1, "", "a", [], [0], {}, None, 0.0]:
    print(repr(v), bool(v))
# 0 False / 1 True / '' False / 'a' True / [] False / [0] True / {} False / None False / 0.0 False

items = []
if not items:            # Pythonic, không cần len(items) == 0
    print("Giỏ hàng trống")
```

### or / and trả về toán hạng

```python
print(0 or "mặc định")      # mặc định
print("An" or "mặc định")   # An
print(None or [] or "x")    # x
print(1 and "ok")           # ok
print(0 and "ok")           # 0
```

### Giá trị mặc định

```python
def greet(name=None):
    name = name or "bạn"         # thay None, "" bằng "bạn"
    return f"Xin chào {name}"

# Nếu 0 là giá trị hợp lệ, KHÔNG dùng or
def set_volume(level=None):
    level = 50 if level is None else level   # tương đương JS: level ?? 50
    return level

print(set_volume(0))   # 0, đúng
```

### Short circuit để tránh lỗi

```python
user = None
# JS: user && user.name
name = user and user["name"]     # không lỗi, vì dừng ở user (None)
print(name)                      # None

# Gọi hàm chỉ khi điều kiện đúng
debug = True
debug and print("debug mode")
```

### `not` luôn trả bool

```python
print(not [])     # True
print(not "abc")  # False
```

### Class tự định nghĩa truthiness

```python
class Cart:
    def __init__(self):
        self.items = []
    def __len__(self):
        return len(self.items)

cart = Cart()
print(bool(cart))   # False vì len = 0
```

## So sánh với JavaScript

| Giá trị | JavaScript | Python |
|---|---|---|
| `[]` | truthy | **falsy** |
| `{}` | truthy | **falsy** |
| `""`, `0`, `null`/`None` | falsy | falsy |
| `NaN` | falsy | **truthy** |
| `"0"` | truthy | truthy |

| JavaScript | Python |
|---|---|
| `a \|\| b` | `a or b` |
| `a && b` | `a and b` |
| `a ?? b` | `a if a is not None else b` |
| `!!x` | `bool(x)` |
| `arr.length === 0` | `not arr` |

## Xem thêm

- [Short Circuiting (JS)](../js/31-Short-Circuiting.md)
- [Toán tử (Operators)](./05-Operators.md)
- [Walrus Operator & Assignment](./06-Walrus-Assignment.md)
- [Data Model, Dunder Methods & MRO](./52-Data-Model-MRO.md)
