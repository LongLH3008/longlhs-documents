[← Mục lục](./index.md)

# Toán tử (Operators)

## Định nghĩa

Toán tử là ký hiệu hoặc từ khóa thực hiện phép tính trên toán hạng. Python có các nhóm:

| Nhóm | Toán tử |
|---|---|
| Số học | `+ - * / // % **` |
| So sánh | `== != < > <= >=` |
| Logic | `and or not` |
| Identity | `is`, `is not` |
| Membership | `in`, `not in` |
| Bitwise | `& \| ^ ~ << >>` |
| Gán | `= += -= *= /= //= %= **= :=` |
| Điều kiện (ternary) | `a if cond else b` |

## Giải thích đơn giản

Python dùng **chữ tiếng Anh** cho logic (`and`, `or`, `not`) thay vì ký hiệu (`&&`, `||`, `!`), đọc lên như một câu văn.

Hai thứ người từ JS hay nhầm:

- `==` trong Python **không ép kiểu**, nên `1 == "1"` là `False`. Python không có `===` vì không cần.
- `is` so sánh **có phải cùng một object trong bộ nhớ không** (giống `===` với object trong JS), còn `==` so sánh **giá trị**. Như hai tờ tiền 100k: `==` là cùng mệnh giá, `is` là đúng cùng một tờ.

## Ví dụ

### Số học

```python
print(7 / 2)    # 3.5  -> chia luôn ra float
print(7 // 2)   # 3    -> floor division (chia lấy phần nguyên, làm tròn xuống)
print(-7 // 2)  # -4   -> làm tròn xuống, không phải về 0
print(7 % 3)    # 1
print(2 ** 10)  # 1024 -> lũy thừa
print("ab" * 3) # ababab
print([0] * 3)  # [0, 0, 0]
```

### So sánh, chaining

```python
print(1 == 1.0)    # True
print(1 == "1")    # False, không ép kiểu
x = 5
print(1 < x < 10)  # True, so sánh nối chuỗi (JS không làm được)
```

### is vs ==

```python
a = [1, 2]
b = [1, 2]
c = a
print(a == b)   # True  -> cùng giá trị
print(a is b)   # False -> khác object
print(a is c)   # True  -> cùng object

value = None
print(value is None)  # cách chuẩn để kiểm tra None
```

### in / not in

```python
print("py" in "python")        # True
print(3 in [1, 2, 3])          # True
print("name" in {"name": "A"}) # True, kiểm tra key của dict
```

### Logic & ternary

```python
age = 20
print(age >= 18 and age < 60)  # True
print(not True)                # False

status = "adult" if age >= 18 else "kid"  # JS: age >= 18 ? "adult" : "kid"
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `&&`, `\|\|`, `!` | `and`, `or`, `not` |
| `===` | `==` (vì không ép kiểu) |
| `Object.is` / `===` với object | `is` |
| `Math.floor(a / b)` | `a // b` |
| `a ** b` | `a ** b` |
| `cond ? a : b` | `a if cond else b` |
| `arr.includes(x)` | `x in arr` |
| `x++` | không có, dùng `x += 1` |

## Xem thêm

- [Toán tử (JS)](../js/04-Operators.md)
- [Walrus Operator & Assignment](./05-Walrus-Assignment.md)
- [Truthy, Falsy & Short Circuiting](./23-Truthy-Short-Circuiting.md)
