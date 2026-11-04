[← Mục lục](./index.md)

# Type Hints

## Định nghĩa

**Type hints** (PEP 484 và các PEP sau) là cú pháp **chú thích kiểu** cho biến, tham số và giá trị trả về. Python runtime **không kiểm tra** type hints: chúng chỉ là metadata (lưu trong `__annotations__`) dùng cho:

- **Static type checker** như `mypy`, `pyright` (Pylance trong VS Code).
- IDE gợi ý code, autocomplete.
- Thư viện đọc annotation lúc chạy như **Pydantic**, **FastAPI**, `dataclasses`.

## Giải thích đơn giản

Type hints là **TypeScript của Python**, nhưng khác ở một điểm lớn: TypeScript phải **biên dịch** sang JS (và bị xóa kiểu), còn Python chạy thẳng file có type hint, chỉ **bỏ qua** chúng. Muốn bắt lỗi kiểu, bạn chạy `mypy` riêng, giống chạy `tsc --noEmit`.

Ví von: type hint là **nhãn ghi trên hộp** ("hộp này đựng sách"). Người vận chuyển (Python runtime) không mở hộp kiểm tra, nhưng thanh tra (mypy) sẽ đọc nhãn và báo nếu bạn bỏ nhầm đồ vào.

## Ví dụ

### Cơ bản

```python
name: str = "An"
age: int = 25
scores: list[int] = [8, 9]
user: dict[str, int] = {"age": 25}
point: tuple[int, int] = (1, 2)
tags: set[str] = {"py"}

def greet(name: str, times: int = 1) -> str:
    return f"Hi {name}! " * times

def log(msg: str) -> None:      # không trả về gì
    print(msg)
```

### Runtime không kiểm tra

```python
def add(a: int, b: int) -> int:
    return a + b

print(add("x", "y"))   # "xy", Python vẫn chạy!
# Nhưng mypy báo: Argument 1 to "add" has incompatible type "str"; expected "int"
```

```bash
pip install mypy
mypy app.py
```

### Optional, Union, Literal

```python
from typing import Literal

def find_user(user_id: int) -> dict | None:    # Python 3.10+, cũ hơn: Optional[dict]
    ...

def parse(value: int | str) -> int:            # Union
    return int(value)

def set_mode(mode: Literal["light", "dark"]) -> None:
    ...
```

### Callable, TypedDict, Any

```python
from typing import Callable, TypedDict, Any

def apply(fn: Callable[[int], int], x: int) -> int:
    return fn(x)

class UserDict(TypedDict):        # giống interface/type cho object của TS
    name: str
    age: int

u: UserDict = {"name": "An", "age": 25}

def debug(x: Any) -> None:        # tắt kiểm tra kiểu
    ...
```

### Generic

```python
def first[T](items: list[T]) -> T:     # cú pháp Python 3.12+
    return items[0]

from typing import TypeVar             # cú pháp cũ
T = TypeVar("T")
def last(items: list[T]) -> T:
    return items[-1]
```

### Pydantic: dùng type hint để validate thật lúc chạy

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

print(User(name="An", age="25"))   # name='An' age=25 (tự ép "25" thành 25)
User(name="An", age="abc")         # ValidationError
```

## So sánh với TypeScript

| TypeScript | Python |
|---|---|
| `let x: string` | `x: str` |
| `number[]` | `list[int]` / `list[float]` |
| `Record<string, number>` | `dict[str, int]` |
| `[number, string]` | `tuple[int, str]` |
| `string \| null` | `str \| None` |
| `"a" \| "b"` | `Literal["a", "b"]` |
| `(x: number) => number` | `Callable[[int], int]` |
| `interface User { ... }` | `TypedDict` / `Protocol` / `dataclass` |
| `any` | `Any` |
| `function f<T>(x: T): T` | `def f[T](x: T) -> T:` |
| `tsc --noEmit` | `mypy .` / `pyright` |
| zod | Pydantic |

## Xem thêm

- [Data Type (JS)](../js/10-Data-Type.md)
- [Data Type](./10-Data-Type.md)
- [Class](./28-Class.md)
- [PEP 8 & Coding Style](./06-PEP8-Style.md)
