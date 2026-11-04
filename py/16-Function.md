[← Mục lục](./index.md)

# Function

## Định nghĩa

Function được khai báo bằng từ khóa `def`, là object kiểu `function`. Python hỗ trợ nhiều loại tham số:

| Loại | Cú pháp | Ý nghĩa |
|---|---|---|
| Positional-or-keyword | `def f(a, b)` | truyền theo vị trí hoặc theo tên |
| Default | `def f(a, b=10)` | có giá trị mặc định |
| Positional-only | `def f(a, b, /)` | chỉ truyền theo vị trí |
| Keyword-only | `def f(*, a, b)` | bắt buộc truyền theo tên |
| Var-positional | `def f(*args)` | gom tham số vị trí thừa vào tuple |
| Var-keyword | `def f(**kwargs)` | gom tham số tên thừa vào dict |

Hàm không có `return` sẽ trả về `None`.

## Giải thích đơn giản

Giống `function` của JS, nhưng có hai điểm "xịn" hơn:

1. **Keyword argument**: gọi hàm bằng tên tham số `create_user(name="An", age=20)`, không cần nhớ thứ tự. Trong JS phải giả lập bằng cách truyền object `{ name, age }`.
2. **Kiểm tra số lượng tham số**: gọi thiếu hay thừa tham số là lỗi `TypeError` ngay, không âm thầm thành `undefined` như JS.

Python **không có hoisting** cho hàm: phải `def` trước khi gọi (ở top-level).

## Ví dụ

### Khai báo & gọi

```python
def greet(name, greeting="Xin chào"):
    """Trả về lời chào. Đây là docstring."""
    return f"{greeting}, {name}!"

print(greet("An"))                       # Xin chào, An!
print(greet("An", "Hello"))              # Hello, An!
print(greet(greeting="Hi", name="Bình")) # Hi, Bình!  (keyword args, đảo thứ tự được)
print(greet.__doc__)                     # Trả về lời chào. Đây là docstring.

greet()                  # TypeError: missing 1 required positional argument: 'name'
greet("A", "B", "C")     # TypeError: takes from 1 to 2 positional arguments but 3 were given
```

### Keyword-only & positional-only

```python
def connect(host, port, /, *, timeout=5, retry=False):
    print(host, port, timeout, retry)

connect("localhost", 5432, timeout=10)   # OK
connect("localhost", 5432, 10)           # TypeError: timeout phải truyền theo tên
connect(host="localhost", port=5432)     # TypeError: host, port chỉ truyền theo vị trí
```

Keyword-only rất hữu ích cho các cờ boolean: `send(msg, urgent=True)` dễ đọc hơn `send(msg, True)`.

### Trả về nhiều giá trị

```python
def divide(a, b):
    return a // b, a % b    # trả về tuple

q, r = divide(17, 5)
print(q, r)   # 3 2
```

### Hàm lồng nhau

```python
def outer():
    def inner():
        return "từ inner"
    return inner()

print(outer())   # từ inner
```

### Hàm không return

```python
def log(msg):
    print(msg)

result = log("hi")
print(result)    # None
```

### pass: thân hàm rỗng

```python
def todo():
    pass         # hoặc dùng ... (Ellipsis)
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `function f(a, b) {}` | `def f(a, b):` |
| `const f = (a) => a * 2` | `f = lambda a: a * 2` |
| `f({ name, age })` (giả lập named args) | `f(name=..., age=...)` |
| thiếu tham số → `undefined` | thiếu tham số → `TypeError` |
| `arguments` | `*args`, `**kwargs` |
| không return → `undefined` | không return → `None` |
| function declaration được hoisting | không hoisting |
| JSDoc `/** ... */` | docstring `"""..."""` |

## Xem thêm

- [Function (JS)](../js/16-Function.md)
- [Lambda](./17-Lambda.md)
- [Default Parameter](./19-Default-Parameter.md)
- [\*args & \*\*kwargs](./21-args-kwargs.md)
- [Name Binding (không có Hoisting)](./37-Name-Binding.md)
