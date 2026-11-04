[← Mục lục](./index.md)

# \*args & \*\*kwargs

## Định nghĩa

- **`*args`** trong khai báo hàm gom mọi **positional argument** thừa thành một **tuple**.
- **`**kwargs`** trong khai báo hàm gom mọi **keyword argument** thừa thành một **dict**.
- Khi **gọi** hàm, `*iterable` rải (unpack) các phần tử thành positional argument, `**dict` rải các cặp key-value thành keyword argument.

Tên `args`, `kwargs` chỉ là quy ước; thứ có ý nghĩa là dấu `*` và `**`.

Thứ tự tham số hợp lệ:

```python
def f(pos_only, /, normal, *args, kw_only, **kwargs): ...
```

## Giải thích đơn giản

- `*args` = **rest parameter** `...args` của JS.
- `**kwargs` = JS không có tương đương trực tiếp (gần nhất là nhận một object `options` rồi `...rest` nó).
- `f(*lst)` = **spread** `f(...arr)` của JS; `f(**d)` = rải một object ra thành named argument.

Ví von: `*` là **túi đựng đồ không tên** (cứ bỏ theo thứ tự), `**` là **tủ có ngăn dán nhãn** (mỗi đồ có tên riêng).

## Ví dụ

### \*args

```python
def total(*nums):
    print(type(nums))   # <class 'tuple'>
    return sum(nums)

print(total())            # 0
print(total(1, 2, 3))     # 6
```

### \*\*kwargs

```python
def build_user(name, **info):
    print(type(info))     # <class 'dict'>
    return {"name": name, **info}

print(build_user("An", age=25, city="HN"))
# {'name': 'An', 'age': 25, 'city': 'HN'}
```

### Kết hợp tất cả

```python
def demo(a, b=2, *args, flag=False, **kwargs):
    print(f"{a=} {b=} {args=} {flag=} {kwargs=}")

demo(1)
# a=1 b=2 args=() flag=False kwargs={}
demo(1, 3, 4, 5, flag=True, x=9)
# a=1 b=3 args=(4, 5) flag=True kwargs={'x': 9}
```

### Unpack khi gọi hàm

```python
def point(x, y, z):
    return f"({x}, {y}, {z})"

coords = [1, 2, 3]
print(point(*coords))                 # (1, 2, 3)   (JS: point(...coords))

opts = {"x": 1, "y": 2, "z": 3}
print(point(**opts))                  # (1, 2, 3)

print(max(*[4, 9, 2]))                # 9
```

### Wrapper chuyển tiếp mọi tham số (dùng trong decorator)

```python
def logged(fn):
    def wrapper(*args, **kwargs):
        print(f"Gọi {fn.__name__} với {args} {kwargs}")
        return fn(*args, **kwargs)
    return wrapper

@logged
def add(a, b):
    return a + b

add(2, b=3)
# Gọi add với (2,) {'b': 3}
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `function f(...args)` | `def f(*args):` (tuple) |
| `function f({ a, ...rest })` | `def f(a, **kwargs):` (dict) |
| `f(...arr)` | `f(*lst)` |
| `f({ ...obj })` | `f(**d)` |
| `arguments` | `*args`, `**kwargs` |
| `Math.max(...arr)` | `max(lst)` (không cần rải) |

## Xem thêm

- [ES6: Rest & Spread (Parameters) (JS)](../js/25-ES6-Rest-Spread-Parameters.md)
- [Function](./28-Function.md)
- [Unpacking](./32-Unpacking.md)
- [Decorator](./44-Decorator.md)
