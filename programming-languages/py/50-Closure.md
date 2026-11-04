[← Mục lục](./index.md)

# Closure

## Định nghĩa

**Closure** là một **function lồng bên trong function khác**, có tham chiếu tới biến của **enclosing scope**, và **vẫn giữ được các biến đó** sau khi hàm bao ngoài đã kết thúc.

Ba điều kiện để có closure trong Python:

1. Có hàm lồng nhau (nested function).
2. Hàm bên trong tham chiếu tới biến của hàm bên ngoài (gọi là **free variable**).
3. Hàm bên ngoài trả về (hoặc truyền đi) hàm bên trong.

Python lưu các biến được giữ lại trong **cell object**, xem được qua thuộc tính `fn.__closure__`. Muốn **gán lại** free variable thì phải khai báo `nonlocal`.

## Giải thích đơn giản

Closure trong Python **giống hệt JS** về bản chất: hàm con mang theo một **"ba lô"** chứa các biến của nơi nó được sinh ra. Hàm cha kết thúc rồi nhưng ba lô vẫn còn.

Khác biệt duy nhất cần nhớ: JS cho hàm con **gán lại** biến ngoài tự do (`count++`), còn Python thì **đọc** tự do nhưng **gán lại** phải có `nonlocal`, nếu không Python tưởng bạn tạo biến local mới.

## Ví dụ

### #1 Bộ đếm duy trì giá trị

```python
def create_counter():
    count = 0

    def increase():
        nonlocal count      # JS không cần dòng này
        count += 1
        return count

    return increase

counter1 = create_counter()
print(counter1())   # 1
print(counter1())   # 2

counter2 = create_counter()
print(counter2())   # 1  -> độc lập với counter1
print(counter1())   # 3

print(counter1.__closure__[0].cell_contents)   # 3, xem "ba lô"
```

So với bản JS:

```js
function createCounter() {
  let counter = 0;
  return () => ++counter;
}
```

### #2 Logger theo namespace

```python
def create_logger(namespace):
    def logger(message):
        print(f"[{namespace}] {message}")
    return logger

info = create_logger("INFO")
error = create_logger("ERROR")

info("Khởi động server")   # [INFO] Khởi động server
error("Mất kết nối DB")    # [ERROR] Mất kết nối DB
```

### #3 Đóng gói dữ liệu (private state)

```python
def create_storage():
    store = {}              # không ai bên ngoài truy cập trực tiếp được

    def get(key):
        return store.get(key)

    def set_(key, value):
        store[key] = value  # sửa object, không gán lại tên store -> không cần nonlocal

    def remove(key):
        store.pop(key, None)

    return {"get": get, "set": set_, "remove": remove}

user = create_storage()
user["set"]("name", "An")
print(user["get"]("name"))   # An
user["remove"]("name")
print(user["get"]("name"))   # None
```

### #4 Bẫy late binding trong vòng lặp

```python
handlers = []
for i in range(3):
    def handler():
        return i
    handlers.append(handler)

print([h() for h in handlers])   # [2, 2, 2]
```

Closure giữ **biến** `i`, không giữ **giá trị** của `i` lúc tạo hàm. Khi gọi, `i` đã là `2`. Đây chính là lỗi kinh điển `var` trong vòng `for` của JS, nhưng Python **không có `let` để sửa tự động**, vì không có block scope.

Cách sửa:

```python
# Cách 1: default parameter "chốt" giá trị lúc định nghĩa
handlers = [lambda i=i: i for i in range(3)]

# Cách 2: factory function tạo scope mới cho mỗi lần lặp
def make_handler(n):
    return lambda: n
handlers = [make_handler(i) for i in range(3)]

# Cách 3: functools.partial
from functools import partial
handlers = [partial(lambda n: n, i) for i in range(3)]

print([h() for h in handlers])   # [0, 1, 2]
```

### #5 Closure là nền tảng của decorator

```python
def require_role(role):
    def decorator(fn):
        def wrapper(user, *args, **kwargs):
            if user.get("role") != role:    # role được closure giữ lại
                raise PermissionError("Không đủ quyền")
            return fn(user, *args, **kwargs)
        return wrapper
    return decorator

@require_role("admin")
def delete_user(user, user_id):
    return f"Đã xóa {user_id}"

print(delete_user({"role": "admin"}, 42))   # Đã xóa 42
```

## Ứng dụng

- Tạo hàm cấu hình sẵn (factory): logger, validator, multiplier.
- Giữ trạng thái riêng tư mà không cần class.
- Decorator, callback, memoization.

Khi trạng thái phức tạp (nhiều method, nhiều dữ liệu), **class** thường dễ đọc hơn closure.

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `let count = 0; return () => ++count` | cần `nonlocal count` để gán lại |
| sửa vòng lặp bằng `let` (block scope) | không có block scope, dùng default param / factory |
| không xem được biến closure trực tiếp | `fn.__closure__[i].cell_contents` |
| closure tạo private state | giống, hoặc dùng class với `_attr` |

## Xem thêm

- [Closure (JS)](../js/38-Closure.md)
- [Scope (LEGB)](./48-Scope-LEGB.md)
- [Decorator](./44-Decorator.md)
- [Lambda](./29-Lambda.md)
