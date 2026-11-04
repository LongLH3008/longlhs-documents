[← Mục lục](./index.md)

# Control Flow & match

## Định nghĩa

Các câu lệnh điều khiển luồng trong Python:

| Câu lệnh | Ý nghĩa |
|---|---|
| `if / elif / else` | rẽ nhánh theo điều kiện |
| `for x in iterable` | duyệt từng phần tử của iterable |
| `while cond` | lặp khi điều kiện còn đúng |
| `break`, `continue` | thoát vòng lặp / bỏ qua lượt hiện tại |
| `for ... else`, `while ... else` | khối `else` chạy khi vòng lặp **kết thúc mà không gặp `break`** |
| `match / case` | **structural pattern matching** (PEP 634, Python 3.10+) |
| `pass` | câu lệnh rỗng, giữ chỗ |

## Giải thích đơn giản

- Python **không có** `for (let i = 0; i < n; i++)`. Mọi vòng `for` đều giống `for...of` của JS. Muốn đếm số thì dùng `range()`.
- Không có `do...while`, không có `switch`. Thay vào đó là `match`, mạnh hơn `switch` rất nhiều: nó không chỉ so sánh giá trị mà còn **"nhìn hình dạng" dữ liệu** và tách biến ra cùng lúc, giống destructuring + switch gộp lại.
- `elif` là viết tắt của `else if`.

Ví von: `switch` là **máy soi vé chỉ đọc số vé**, còn `match` là **nhân viên kiểm tra hành lý**: nhìn xem vali có hình dạng gì, bên trong có gì, rồi phân loại.

## Ví dụ

### if / elif / else

```python
score = 7.5
if score >= 8:
    grade = "Giỏi"
elif score >= 6.5:
    grade = "Khá"
else:
    grade = "Trung bình"
print(grade)   # Khá
```

### for với range, enumerate, zip

```python
for i in range(3):           # 0 1 2      (JS: for (let i = 0; i < 3; i++))
    print(i)

for i in range(10, 0, -3):   # 10 7 4 1   (start, stop, step)
    print(i)

for i, name in enumerate(["An", "Bình"], start=1):
    print(i, name)           # 1 An / 2 Bình

for name, age in zip(["An", "Bình"], [25, 30]):
    print(name, age)
```

### while, break, continue

```python
n = 0
while True:          # giả lập do...while
    n += 1
    if n % 2 == 0:
        continue
    if n > 7:
        break
    print(n)         # 1 3 5 7
```

### for ... else

```python
def find_prime_divisor(n):
    for d in range(2, n):
        if n % d == 0:
            print(f"{n} chia hết cho {d}")
            break
    else:                      # không gặp break
        print(f"{n} là số nguyên tố")

find_prime_divisor(9)    # 9 chia hết cho 3
find_prime_divisor(7)    # 7 là số nguyên tố
```

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

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `if {} else if {} else {}` | `if: / elif: / else:` |
| `for (let i = 0; i < n; i++)` | `for i in range(n):` |
| `for (const x of arr)` | `for x in arr:` |
| `for (const k in obj)` | `for k in d:` |
| `arr.forEach((x, i) => ...)` | `for i, x in enumerate(arr):` |
| `do { } while (c)` | `while True: ... if not c: break` |
| `switch (x) { case 1: ... }` | `match x: case 1: ...` |
| `default:` | `case _:` |
| `case 1: case 2:` (fall-through) | `case 1 \| 2:` (không có fall-through) |

## Xem thêm

- [Toán tử (JS)](../js/04-Operators.md)
- [Unpacking](./20-Unpacking.md)
- [Iterator & Generator](./31-Iterator-Generator.md)
- [Exception](./26-Exception.md)
