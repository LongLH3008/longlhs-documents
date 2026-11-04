[← Mục lục](./index.md)

# Unpacking

## Định nghĩa

**Unpacking** (iterable unpacking, sequence unpacking) là cú pháp gán các phần tử của một **iterable** (list, tuple, string, generator...) vào nhiều biến cùng lúc. Kết hợp với toán tử `*` (starred expression, PEP 3132) để gom phần còn lại vào một list.

```python
a, b, c = iterable
first, *rest = iterable
```

Với dict, không có cú pháp unpacking theo tên key như JS; dùng `match` (Python 3.10+) hoặc lấy từng key.

## Giải thích đơn giản

Đây là **destructuring** của JS, nhưng Python không cần dấu `[]`: chỉ cần dấu phẩy. Giống như mở hộp quà có 3 món và đặt mỗi món vào một tay (một biến).

Khác JS: số biến phải **khớp chính xác** số phần tử (trừ khi dùng `*`), nếu không là `ValueError`. JS thì cho biến thừa thành `undefined`.

## Ví dụ

### Cơ bản

```python
point = (3, 4)
x, y = point
print(x, y)            # 3 4

a, b, c = "xyz"        # string cũng là iterable
print(a, b, c)         # x y z

a, b = [1, 2, 3]       # ValueError: too many values to unpack (expected 2)
```

### Starred `*`

```python
first, *rest = [1, 2, 3, 4]
print(first, rest)     # 1 [2, 3, 4]

*init, last = [1, 2, 3, 4]
print(init, last)      # [1, 2, 3] 4

head, *middle, tail = "python"
print(head, middle, tail)   # p ['y', 't', 'h', 'o'] n
```

### Bỏ qua giá trị bằng `_`

```python
_, second, _ = (10, 20, 30)
print(second)          # 20

name, *_ = ["An", 25, "HN", "dev"]
```

### Lồng nhau

```python
data = ("An", (25, "HN"))
name, (age, city) = data
print(name, age, city)  # An 25 HN
```

### Trong vòng lặp

```python
users = [("An", 25), ("Bình", 30)]
for name, age in users:
    print(name, age)

scores = {"an": 8, "binh": 6}
for key, value in scores.items():
    print(key, value)
```

### Swap

```python
a, b = 1, 2
a, b = b, a    # JS: [a, b] = [b, a]
```

### "Destructure" dict

```python
user = {"name": "An", "age": 25, "city": "HN"}

# Cách 1: lấy từng key
name, age = user["name"], user["age"]

# Cách 2: operator.itemgetter
from operator import itemgetter
name, age = itemgetter("name", "age")(user)

# Cách 3: structural pattern matching (3.10+)
match user:
    case {"name": str(name), "age": int(age)}:
        print(name, age)   # An 25
```

### Unpacking để tạo collection mới (spread)

```python
a = [1, 2]
b = [3, 4]
merged = [*a, *b, 5]           # [1, 2, 3, 4, 5]   (JS: [...a, ...b, 5])
s = {*a, *b}                   # {1, 2, 3, 4}
d = {**{"x": 1}, "y": 2}       # {'x': 1, 'y': 2}  (JS: { ...obj, y: 2 })
print(*a)                      # 1 2  -> truyền từng phần tử vào print
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `const [a, b] = arr` | `a, b = arr` |
| `const [first, ...rest] = arr` | `first, *rest = arr` |
| `const [, second] = arr` | `_, second, *_ = arr` |
| `const { name, age } = obj` | `name, age = d["name"], d["age"]` hoặc `match` |
| `const { a = 1 } = obj` | `a = d.get("a", 1)` |
| thiếu phần tử → `undefined` | thiếu/thừa phần tử → `ValueError` |
| `[...a, ...b]` | `[*a, *b]` |
| `{ ...a, ...b }` | `{**a, **b}` |

## Xem thêm

- [ES6: Destructuring (JS)](../js/24-ES6-Destructuring.md)
- [Spread, Rest, Destructuring (JS)](../js/30-Spread-Rest-Destructuring.md)
- [Tuple](./14-Tuple.md)
- [\*args & \*\*kwargs](./21-args-kwargs.md)
- [Control Flow & match](./25-Control-Flow-Match.md)
