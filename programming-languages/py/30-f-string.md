[← Mục lục](./index.md)

# f-string

## Định nghĩa

**f-string** (formatted string literal, PEP 498, Python 3.6+) là chuỗi có tiền tố `f` hoặc `F`, cho phép nhúng **biểu thức Python** bên trong `{}`. Biểu thức được đánh giá lúc chạy và chuyển thành chuỗi, có thể kèm **format spec** sau dấu `:` theo cú pháp *Format Specification Mini-Language*.

```python
f"{bieu_thuc!conversion:format_spec}"
```

## Giải thích đơn giản

f-string chính là **template literal** của JS: `` `Xin chào ${name}` `` ↔ `f"Xin chào {name}"`. Điểm hơn của Python là phần **format sau dấu `:`**: làm tròn số, thêm dấu phẩy ngàn, căn lề, phần trăm... ngay trong chuỗi, không cần gọi `toFixed()` hay `toLocaleString()`.

Ví von: f-string là **mẫu đơn có ô trống**, bạn điền biến vào ô, còn format spec là **quy định cách viết** trong ô (viết hoa, 2 chữ số thập phân...).

## Ví dụ

### Cơ bản

```python
name = "An"
age = 25
print(f"Tôi là {name}, {age} tuổi")      # Tôi là An, 25 tuổi
print(f"Năm sau: {age + 1}")              # biểu thức bất kỳ
print(f"{name.upper()} - {len(name)}")    # gọi method

user = {"name": "Bình"}
print(f"Hi {user['name']}")               # nháy bên trong khác nháy bên ngoài
```

### Format số

```python
pi = 3.14159265
print(f"{pi:.2f}")          # 3.14          (2 chữ số thập phân)
print(f"{1234567:,}")       # 1,234,567     (phân cách ngàn)
print(f"{1234567.8:,.1f}")  # 1,234,567.8
print(f"{0.856:.1%}")       # 85.6%
print(f"{42:05d}")          # 00042         (đệm số 0)
print(f"{255:x} {255:b}")   # ff 11111111   (hex, binary)
print(f"{1e6:.2e}")         # 1.00e+06
```

### Căn lề (in bảng)

```python
for item, price in [("Táo", 10), ("Chuối", 5)]:
    print(f"{item:<10}|{price:>6}")
# Táo       |    10
# Chuối     |     5

print(f"{'title':^20}")     # căn giữa trong 20 ký tự
print(f"{'x':*^7}")         # ***x***
```

### Debug nhanh với `=` (Python 3.8+)

```python
x = 10
y = 3
print(f"{x=}, {y=}, {x * y=}")
# x=10, y=3, x * y=30
```

### Ngày giờ

```python
from datetime import datetime
now = datetime(2026, 1, 15, 9, 5)
print(f"{now:%d/%m/%Y %H:%M}")   # 15/01/2026 09:05
```

### Conversion `!r`, `!s`

```python
text = "hi"
print(f"{text!r}")   # 'hi'  -> dùng repr(), thấy rõ là chuỗi
print(f"{text}")     # hi
```

### Chuỗi nhiều dòng

```python
html = f"""
<div>
  <h1>{name}</h1>
</div>
"""
```

### Các cách format cũ (vẫn gặp trong code cũ)

```python
"Hi %s, %d tuổi" % (name, age)          # kiểu C, cũ
"Hi {}, {} tuổi".format(name, age)      # str.format
f"Hi {name}, {age} tuổi"                # khuyên dùng
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `` `Hi ${name}` `` | `f"Hi {name}"` |
| `x.toFixed(2)` | `f"{x:.2f}"` |
| `n.toLocaleString("en-US")` | `f"{n:,}"` |
| `String(n).padStart(5, "0")` | `f"{n:05d}"` |
| `s.padEnd(10)` | `f"{s:<10}"` |
| tagged template `` tag`...` `` | không có tương đương; Python 3.14 có **t-string** (`t"..."`) |
| `console.log({ x })` | `print(f"{x=}")` |

## Xem thêm

- [ES6: Template Literals (JS)](../js/21-ES6-Template-Literals.md)
- [ES6: Tagged Template Literals (JS)](../js/22-ES6-Tagged-Template-Literals.md)
- [String](./19-String.md)
- [Number](./18-Number.md)
