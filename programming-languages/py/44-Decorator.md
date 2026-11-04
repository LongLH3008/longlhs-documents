[← Mục lục](./index.md)

# Decorator

## Định nghĩa

**Decorator** là một callable (thường là hàm) **nhận một hàm/class và trả về một hàm/class khác**, dùng để bổ sung hành vi mà không sửa code gốc. Cú pháp `@decorator` đặt trên `def` là **syntactic sugar**:

```python
@decorator
def f(): ...

# tương đương
def f(): ...
f = decorator(f)
```

Decorator được xây dựng dựa trên **first-class function** và **closure**. `functools.wraps` giữ lại `__name__`, `__doc__` của hàm gốc.

## Giải thích đơn giản

Decorator là **higher-order function bọc quanh hàm khác**, giống bạn từng viết trong JS: `const loggedAdd = withLog(add)`. Python chỉ thêm cú pháp `@` cho đẹp. (JS cũng có decorator nhưng chủ yếu cho class, qua TypeScript hoặc proposal TC39.)

Ví von: decorator là **lớp giấy gói quà**. Món quà (hàm gốc) không đổi, nhưng bên ngoài có thêm thiệp, nơ (log, đo thời gian, kiểm tra quyền...). Bạn có thể gói nhiều lớp.

Bạn gặp decorator khắp nơi: `@property`, `@staticmethod`, `@dataclass`, `@app.get("/")` của FastAPI, `@pytest.fixture`.

## Ví dụ

### Decorator đơn giản

```python
import functools

def log_call(fn):
    @functools.wraps(fn)             # giữ tên & docstring của fn
    def wrapper(*args, **kwargs):
        print(f"-> {fn.__name__}{args}")
        result = fn(*args, **kwargs)
        print(f"<- {fn.__name__} = {result}")
        return result
    return wrapper

@log_call
def add(a, b):
    return a + b

add(2, 3)
# -> add(2, 3)
# <- add = 5
print(add.__name__)   # add  (không có wraps sẽ là "wrapper")
```

### Đo thời gian chạy

```python
import time, functools

def timer(fn):
    @functools.wraps(fn)
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        try:
            return fn(*args, **kwargs)
        finally:
            print(f"{fn.__name__} mất {time.perf_counter() - start:.3f}s")
    return wrapper

@timer
def slow():
    time.sleep(0.5)

slow()   # slow mất 0.500s
```

### Decorator có tham số (3 tầng hàm)

```python
import functools

def retry(times=3):
    def decorator(fn):
        @functools.wraps(fn)
        def wrapper(*args, **kwargs):
            for attempt in range(1, times + 1):
                try:
                    return fn(*args, **kwargs)
                except Exception as e:
                    print(f"Lần {attempt} lỗi: {e}")
            raise RuntimeError(f"Thất bại sau {times} lần")
        return wrapper
    return decorator

@retry(times=2)            # retry(times=2) trả về decorator, rồi decorator(call_api)
def call_api():
    raise ConnectionError("timeout")

call_api()
# Lần 1 lỗi: timeout
# Lần 2 lỗi: timeout
# RuntimeError: Thất bại sau 2 lần
```

### Cache có sẵn

```python
from functools import lru_cache, cache

@cache                     # memoize, giống viết closure lưu Map trong JS
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)

print(fib(100))            # 354224848179261915075, chạy tức thì
```

### Xếp chồng nhiều decorator

```python
@timer
@log_call
def mul(a, b):
    return a * b
# tương đương: mul = timer(log_call(mul))  -> decorator gần def nhất chạy bọc trước
```

### Decorator trong framework (minh họa)

```python
from fastapi import FastAPI
app = FastAPI()

@app.get("/users/{user_id}")   # đăng ký hàm làm handler cho route
def get_user(user_id: int):
    return {"id": user_id}
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `const f2 = withLog(f)` | `@log_call` trên `def f` |
| HOF bọc hàm | decorator = HOF + cú pháp `@` |
| `memoize(fn)` tự viết / lodash | `@functools.cache`, `@lru_cache` |
| decorator TS (`@Component`) chủ yếu cho class | decorator dùng cho cả function, method, class |
| `debounce(fn, 300)` (lodash) | `@retry(times=3)` (decorator có tham số) |

## Xem thêm

- [First-class & Higher-order Function (JS)](../js/09-First-class-Higher-order-Function.md)
- [Closure (JS)](../js/38-Closure.md)
- [Closure](./50-Closure.md)
- [\*args & \*\*kwargs](./33-args-kwargs.md)
- [Debounce & Throttle](../../fe/debounce-throttle.md)
