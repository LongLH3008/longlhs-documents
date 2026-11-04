[← Mục lục](./index.md)

# Default Parameter

## Định nghĩa

**Default parameter** là tham số có giá trị mặc định, dùng khi người gọi không truyền tham số đó: `def f(a, b=10)`.

Quy tắc quan trọng: giá trị mặc định được **đánh giá một lần duy nhất, tại thời điểm `def` được thực thi** (lúc định nghĩa hàm), và được lưu vào `f.__defaults__`. Mọi lần gọi hàm sau đó **dùng chung** object mặc định ấy.

Tham số có default phải đứng **sau** tham số không có default.

## Giải thích đơn giản

Trong JS, `function f(arr = [])` tạo một mảng **mới** mỗi lần gọi. Python thì không: default giống **chiếc cốc dùng chung đặt sẵn trên bàn**, ai gọi hàm mà không mang cốc theo thì dùng chung chiếc cốc đó. Nếu người trước đổ nước vào (mutate list), người sau sẽ thấy nước còn lại.

Với giá trị immutable (số, chuỗi, `None`, tuple) thì không sao. Rắc rối chỉ xảy ra với **mutable default** như `list`, `dict`, `set`. Đây là một trong những lỗi Python nổi tiếng nhất.

## Ví dụ

### Cơ bản

```python
def greet(name, greeting="Xin chào"):
    return f"{greeting}, {name}"

print(greet("An"))           # Xin chào, An
print(greet("An", "Hello"))  # Hello, An

def f(a=1, b):   # SyntaxError: tham số không default không được đứng sau tham số có default
    ...
```

### Bẫy mutable default

```python
def add_item(item, cart=[]):     # SAI
    cart.append(item)
    return cart

print(add_item("táo"))    # ['táo']
print(add_item("cam"))    # ['táo', 'cam']   -> bất ngờ!
print(add_item.__defaults__)  # (['táo', 'cam'],)
```

### Cách đúng: dùng None làm sentinel

```python
def add_item(item, cart=None):   # ĐÚNG
    if cart is None:
        cart = []                # tạo list mới mỗi lần gọi
    cart.append(item)
    return cart

print(add_item("táo"))    # ['táo']
print(add_item("cam"))    # ['cam']
```

### Default được tính lúc định nghĩa

```python
from datetime import datetime

def log(msg, at=datetime.now()):   # SAI: thời gian bị "đóng băng"
    print(at, msg)

def log(msg, at=None):             # ĐÚNG
    at = at or datetime.now()
    print(at, msg)
```

### Phân biệt "không truyền" và "truyền None"

```python
_MISSING = object()   # sentinel riêng

def update(value=_MISSING):
    if value is _MISSING:
        print("không truyền gì")
    elif value is None:
        print("truyền None, muốn xóa giá trị")
    else:
        print("giá trị:", value)

update()        # không truyền gì
update(None)    # truyền None, muốn xóa giá trị
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `function f(b = 10)` | `def f(b=10):` |
| default **tính lại mỗi lần gọi** | default **tính một lần lúc định nghĩa** |
| `f(arr = [])` an toàn | `f(arr=[])` là bug, dùng `arr=None` |
| truyền `undefined` → dùng default | truyền `None` → **không** dùng default, `None` là giá trị thật |
| `f(a = 1, b)` hợp lệ | `SyntaxError` |

## Xem thêm

- [ES6: Default Parameter (JS)](../js/23-ES6-Default-Parameter.md)
- [Function](./16-Function.md)
- [Mutable & Immutable](./35-Mutable-Immutable.md)
