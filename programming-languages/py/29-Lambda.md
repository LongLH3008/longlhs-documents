[← Mục lục](./index.md)

# Lambda

## Định nghĩa

`lambda` tạo một **anonymous function** (hàm không tên) gồm **duy nhất một biểu thức**. Giá trị của biểu thức đó được tự động return.

```python
lambda tham_so_1, tham_so_2: bieu_thuc
```

Lambda không chứa được câu lệnh (`if` dạng khối, `for`, `return`, gán `=`, `try`...). Nó hoàn toàn tương đương một `def` một dòng.

## Giải thích đơn giản

Lambda giống **arrow function rút gọn** của JS (`x => x * 2`), nhưng bị giới hạn hơn: chỉ được một biểu thức, không có `{ ... }` nhiều dòng.

Python muốn bạn dùng lambda cho các hàm **nhỏ, dùng một lần** (như `key=` của `sorted`). Hàm cần nhiều logic thì viết `def` có tên rõ ràng. Giống như ghi chú trên giấy nhớ (lambda) so với viết vào sổ (def).

Khác biệt quan trọng: arrow function của JS **không có `this` riêng**. Python không có khái niệm `this` ngầm (dùng `self` tường minh), nên lambda và def không khác nhau về điểm này.

## Ví dụ

### Cơ bản

```python
square = lambda x: x * x
print(square(5))          # 25

add = lambda a, b=1: a + b
print(add(2), add(2, 3))  # 3 5

no_arg = lambda: "hi"
print(no_arg())           # hi
```

PEP 8 khuyên **không gán lambda vào biến**; nếu cần tên thì dùng `def`:

```python
def square(x):
    return x * x
```

### Dùng đúng chỗ: làm tham số

```python
users = [("An", 30), ("Bình", 22), ("Chi", 27)]

print(sorted(users, key=lambda u: u[1]))
# [('Bình', 22), ('Chi', 27), ('An', 30)]

print(max(users, key=lambda u: u[1]))   # ('An', 30)

nums = [1, 2, 3, 4]
print(list(filter(lambda n: n % 2 == 0, nums)))  # [2, 4]
```

### Biểu thức điều kiện trong lambda

```python
grade = lambda score: "Đạt" if score >= 5 else "Trượt"
print(grade(7))   # Đạt
```

### Bẫy: lambda trong vòng lặp (late binding)

```python
funcs = [lambda: i for i in range(3)]
print([f() for f in funcs])   # [2, 2, 2]  -> không phải [0, 1, 2]!

# Cách sửa: "chốt" giá trị bằng default parameter
funcs = [lambda i=i: i for i in range(3)]
print([f() for f in funcs])   # [0, 1, 2]
```

Lambda chỉ nhớ **biến** `i`, không nhớ giá trị tại lúc tạo. Đây là cùng bản chất với lỗi `var` trong vòng lặp của JS (xem bài Closure).

### Thay lambda bằng operator

```python
from operator import itemgetter, attrgetter

sorted(users, key=itemgetter(1))   # tương đương lambda u: u[1]
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `x => x * 2` | `lambda x: x * 2` |
| `(a, b) => a + b` | `lambda a, b: a + b` |
| `() => "hi"` | `lambda: "hi"` |
| `x => { ...nhiều dòng... }` | không có, dùng `def` |
| arrow function không có `this` riêng | không áp dụng (Python không có `this` ngầm) |

## Xem thêm

- [ES6: Arrow Function (JS)](../js/20-ES6-Arrow-Function.md)
- [First-class & Higher-order Function](./10-First-class-Higher-order-Function.md)
- [Function](./28-Function.md)
- [Closure](./50-Closure.md)
