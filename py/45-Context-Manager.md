[← Mục lục](./index.md)

# Context Manager (with)

## Định nghĩa

**Context manager** là object định nghĩa hai method:

- `__enter__(self)`: chạy khi vào khối `with`, giá trị trả về gán cho biến sau `as`.
- `__exit__(self, exc_type, exc_value, traceback)`: chạy khi rời khối `with`, **kể cả khi có exception**. Trả về `True` nghĩa là "nuốt" exception.

Câu lệnh `with` đảm bảo tài nguyên (file, kết nối DB, lock, transaction...) được **thiết lập và dọn dẹp** đúng cách. Module `contextlib` cung cấp `@contextmanager` để viết context manager bằng generator.

## Giải thích đơn giản

`with` là cách viết gọn và an toàn của `try ... finally`. Giống tủ lạnh có **cửa tự đóng**: bạn mở cửa lấy đồ (enter), dù bạn làm rơi đồ hay không (exception), cửa vẫn tự đóng lại (exit).

Trong JS, bạn thường tự viết `try { ... } finally { conn.close() }`. JS gần đây có `using` (Explicit Resource Management, `Symbol.dispose`), chính là ý tưởng của `with`.

## Ví dụ

### Mở file

```python
# Không có with: phải tự đóng, quên là rò rỉ tài nguyên
f = open("data.txt", encoding="utf-8")
try:
    content = f.read()
finally:
    f.close()

# Có with: tự đóng, kể cả khi lỗi
with open("data.txt", encoding="utf-8") as f:
    content = f.read()
print(f.closed)   # True

# Nhiều context manager cùng lúc
with open("in.txt") as src, open("out.txt", "w") as dst:
    dst.write(src.read())
```

### Tự viết bằng class

```python
import time

class Timer:
    def __enter__(self):
        self.start = time.perf_counter()
        return self

    def __exit__(self, exc_type, exc, tb):
        self.elapsed = time.perf_counter() - self.start
        print(f"Mất {self.elapsed:.3f}s")
        return False          # không nuốt exception

with Timer() as t:
    sum(range(10_000_000))
# Mất 0.2xxs
```

### Tự viết bằng @contextmanager

```python
from contextlib import contextmanager

@contextmanager
def transaction(conn):
    print("BEGIN")
    try:
        yield conn            # phần trước yield = __enter__
        print("COMMIT")       # không lỗi
    except Exception:
        print("ROLLBACK")     # có lỗi
        raise
    # phần sau yield = __exit__

with transaction("db") as c:
    print("INSERT ...")
# BEGIN / INSERT ... / COMMIT
```

### Lock trong threading

```python
import threading

lock = threading.Lock()
counter = 0

def increment():
    global counter
    with lock:                # acquire khi vào, release khi ra
        counter += 1
```

### Các context manager có sẵn hữu ích

```python
from contextlib import suppress, redirect_stdout
import io

with suppress(FileNotFoundError):     # bỏ qua lỗi cụ thể
    open("khong-ton-tai.txt")

buf = io.StringIO()
with redirect_stdout(buf):            # gom print vào biến
    print("hello")
print(buf.getvalue())                 # hello
```

### async with

```python
import aiohttp

async def fetch(url):
    async with aiohttp.ClientSession() as session:     # __aenter__ / __aexit__
        async with session.get(url) as res:
            return await res.text()
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `try { ... } finally { r.close() }` | `with r: ...` |
| `using r = getResource()` (ES2026) | `with get_resource() as r:` |
| `[Symbol.dispose]()` | `__exit__()` |
| `await using` / `Symbol.asyncDispose` | `async with` / `__aexit__()` |

## Xem thêm

- [Exception](./37-Exception.md)
- [File I/O & pathlib](./46-File-IO-pathlib.md)
- [Iterator & Generator](./42-Iterator-Generator.md)
- [Race Condition](../be/race-condition.md)
