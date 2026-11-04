[← Mục lục](./index.md)

# If-Else & Ternary (Câu lệnh điều kiện)

## Định nghĩa

Câu lệnh điều kiện là cấu trúc **rẽ nhánh** (branching) trong control flow: chọn khối lệnh nào được chạy dựa trên giá trị **truthy / falsy** của một biểu thức.

| Cú pháp | Loại | Ý nghĩa |
|---|---|---|
| `if cond:` | statement | chạy khối khi `cond` truthy |
| `elif cond:` | statement | "else if", kiểm tra tiếp khi các nhánh trên sai |
| `else:` | statement | chạy khi mọi nhánh trên đều sai |
| `a if cond else b` | **expression** (conditional expression / ternary) | trả về `a` hoặc `b`, dùng được trong biểu thức |

Python dùng **thụt lề** (indentation, chuẩn 4 dấu cách) thay cho `{}` để xác định khối lệnh, và dấu `:` ở cuối dòng điều kiện.

## Giải thích đơn giản

Giống **ngã ba đường có biển chỉ dẫn**: xe đi tới, đọc biển (điều kiện) rồi rẽ vào **đúng một** nhánh. Các biển được đọc từ trên xuống, gặp biển đúng đầu tiên là rẽ luôn, không đọc các biển sau.

- Điều kiện không cần là `True/False` thật sự: list rỗng, `0`, `""`, `None` đều được coi là **sai** (falsy). Xem [Truthy, Falsy & Short Circuiting](./36-Truthy-Short-Circuiting.md).
- Ternary viết **ngược** so với JS: giá trị đúng đứng trước. Đọc như câu tiếng Anh: "*adult* **if** age >= 18 **else** *kid*".

## Ví dụ

### if / elif / else

```python
score = 7.5

if score >= 8:
    grade = "Giỏi"
elif score >= 6.5:
    grade = "Khá"
elif score >= 5:
    grade = "Trung bình"
else:
    grade = "Yếu"

print(grade)   # Khá
```

### Ternary (conditional expression)

```python
age = 20
status = "adult" if age >= 18 else "kid"   # JS: age >= 18 ? "adult" : "kid"

# Dùng trong f-string, tham số, return...
print(f"Bạn {'đủ' if age >= 18 else 'chưa đủ'} tuổi")

def abs_value(x):
    return x if x >= 0 else -x
```

Không nên lồng nhiều ternary, khó đọc. Dùng `if/elif` hoặc dict tra cứu thay thế.

### So sánh nối chuỗi (chained comparison)

```python
x = 15
if 10 <= x < 20:        # JS phải viết: x >= 10 && x < 20
    print("Trong khoảng [10, 20)")
```

### Kết hợp and / or / not / in

```python
user = {"role": "admin", "active": True}

if user["active"] and user["role"] in ("admin", "mod"):
    print("Được vào trang quản trị")

if not user.get("banned"):
    print("Không bị cấm")
```

### Kiểm tra None và rỗng

```python
items = []
name = None

if not items:              # Pythonic, thay vì len(items) == 0
    print("Giỏ hàng trống")

if name is None:           # so sánh với None luôn dùng `is`, không dùng ==
    name = "Khách"
```

### Guard clause: thoát sớm thay vì lồng sâu

```python
# Lồng sâu, khó đọc
def checkout(cart, user):
    if user:
        if cart:
            if user.balance >= cart.total:
                return "OK"
            else:
                return "Không đủ tiền"
        else:
            return "Giỏ trống"
    else:
        return "Chưa đăng nhập"

# Guard clause: phẳng, dễ đọc
def checkout(cart, user):
    if not user:
        return "Chưa đăng nhập"
    if not cart:
        return "Giỏ trống"
    if user.balance < cart.total:
        return "Không đủ tiền"
    return "OK"
```

### Dict thay chuỗi if/elif dài

```python
def shipping_fee(region):
    fees = {"HN": 20_000, "HCM": 20_000, "DN": 30_000}
    return fees.get(region, 50_000)    # mặc định 50k

print(shipping_fee("DN"))      # 30000
print(shipping_fee("CM"))      # 50000
```

Với dữ liệu có cấu trúc phức tạp, cân nhắc [match / case](./27-Match-Case.md).

### pass: khối rỗng

```python
if debug:
    pass        # TODO: thêm log sau. Python không cho khối rỗng, phải có pass
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `if (c) { } else if (c2) { } else { }` | `if c: / elif c2: / else:` |
| `c ? a : b` | `a if c else b` |
| `x >= 1 && x <= 9` | `1 <= x <= 9` |
| `&&`, `\|\|`, `!` | `and`, `or`, `not` |
| `x === null` | `x is None` |
| `arr.includes(x)` | `x in arr` |
| `{}` rỗng là truthy | `{}` / `[]` rỗng là **falsy** |
| khối rỗng `{}` | `pass` |

## Xem thêm

- [Truthy, Falsy & Short Circuiting](./36-Truthy-Short-Circuiting.md)
- [Toán tử (Operators)](./05-Operators.md)
- [Vòng lặp (Loop)](./26-Loop.md)
- [Pattern Matching (match / case)](./27-Match-Case.md)
- [Walrus Operator & Assignment](./06-Walrus-Assignment.md)
