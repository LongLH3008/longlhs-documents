[← Mục lục](./index.md)

# Pattern Matching (match / case)

## Định nghĩa

`match / case` là câu lệnh **structural pattern matching** (PEP 634, Python 3.10+). Nó so khớp một giá trị (subject) lần lượt với các **pattern** trong từng `case`, chạy khối của `case` **đầu tiên khớp**, đồng thời có thể **tách (capture) dữ liệu** bên trong ra biến.

Các loại pattern chính:

| Pattern | Ví dụ | Khớp khi |
|---|---|---|
| Literal | `case 200:` | bằng giá trị đó |
| OR | `case 200 \| 201:` | khớp một trong các pattern |
| Wildcard | `case _:` | luôn khớp (giống `default`) |
| Capture | `case x:` | luôn khớp, gán vào `x` |
| Sequence | `case [a, b, *rest]:` | là sequence có hình dạng đó |
| Mapping | `case {"type": t}:` | là mapping có các key đó |
| Class | `case Point(x=0, y=y):` | là instance của class, thuộc tính khớp |
| Guard | `case x if x > 0:` | pattern khớp **và** điều kiện đúng |

## Giải thích đơn giản

Python không có `switch`. Thay vào đó là `match`, mạnh hơn `switch` rất nhiều: nó không chỉ so sánh giá trị mà còn **"nhìn hình dạng" dữ liệu** và tách biến ra cùng lúc, giống destructuring + switch gộp lại.

Ví von: `switch` là **máy soi vé chỉ đọc số vé**, còn `match` là **nhân viên kiểm tra hành lý**: nhìn xem vali có hình dạng gì, bên trong có gì, rồi phân loại.

Rẽ nhánh đơn giản theo điều kiện vẫn dùng `if / elif / else` (xem [If-Else](./25-If-Else.md)). `match` hợp khi phải phân loại dữ liệu có cấu trúc.

## Ví dụ

### match: thay switch

```python
def http_status(code):
    match code:
        case 200 | 201:
            return "OK"
        case 404:
            return "Not Found"
        case 500:
            return "Server Error"
        case _:                 # giống default
            return "Unknown"

print(http_status(201))   # OK
```

### match: theo hình dạng dữ liệu

```python
def handle(command):
    match command.split():
        case ["go", direction]:
            return f"Đi về {direction}"
        case ["take", *items]:
            return f"Nhặt {', '.join(items)}"
        case ["quit"]:
            return "Thoát"
        case _:
            return "Không hiểu"

print(handle("go bắc"))           # Đi về bắc
print(handle("take kiếm khiên"))  # Nhặt kiếm, khiên

def process(event):
    match event:
        case {"type": "click", "x": x, "y": y}:
            return f"Click tại ({x}, {y})"
        case {"type": "key", "key": k} if k == "Enter":   # guard
            return "Nhấn Enter"
        case _:
            return "Bỏ qua"

print(process({"type": "click", "x": 10, "y": 20}))   # Click tại (10, 20)
```

### match: class pattern

```python
from dataclasses import dataclass

@dataclass
class Point:
    x: int
    y: int

def where(p):
    match p:
        case Point(x=0, y=0):
            return "Gốc tọa độ"
        case Point(x=0, y=y):
            return f"Trên trục Y, y={y}"
        case Point(x=x, y=0):
            return f"Trên trục X, x={x}"
        case Point():
            return "Chỗ khác"

print(where(Point(0, 5)))   # Trên trục Y, y=5
```

### Bẫy: tên trần là capture, không phải so sánh

```python
RED = "red"
color = "blue"

match color:
    case RED:          # SAI: đây là capture pattern, gán color vào biến RED, luôn khớp!
        print("đỏ")

# Đúng: dùng tên có dấu chấm (Enum, module, class) để so sánh giá trị
from enum import Enum
class Color(Enum):
    RED = "red"
    BLUE = "blue"

match Color(color):
    case Color.RED:
        print("đỏ")
    case Color.BLUE:
        print("xanh")   # xanh
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `switch (x) { case 1: ... }` | `match x: case 1: ...` |
| `default:` | `case _:` |
| `case 1: case 2:` (fall-through) | `case 1 \| 2:` (không có fall-through, không cần `break`) |
| `const { type, x } = obj` rồi `if` | `case {"type": t, "x": x}:` |
| `if (x instanceof Point)` | `case Point():` |

## Xem thêm

- [If-Else & Ternary](./25-If-Else.md)
- [Vòng lặp (Loop)](./26-Loop.md)
- [Unpacking](./32-Unpacking.md)
- [Enum](./15-Enum.md)
- [Class](./39-Class.md)
