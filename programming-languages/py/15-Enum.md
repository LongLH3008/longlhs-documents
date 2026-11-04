[← Mục lục](./index.md)

# Enum

## Định nghĩa

`Enum` (module chuẩn `enum`) là kiểu dữ liệu định nghĩa một **tập hữu hạn các hằng có tên** (members). Mỗi member là một **singleton** có `name` và `value`, so sánh bằng `is` được, duyệt được, và không thể gán lại.

Các biến thể:

| Class | Đặc điểm |
|---|---|
| `Enum` | cơ bản, member **không** bằng giá trị thô (`Color.RED != "red"`) |
| `StrEnum` (3.11+) | member là `str`, so sánh trực tiếp với chuỗi |
| `IntEnum` | member là `int`, so sánh/tính toán như số |
| `Flag` / `IntFlag` | kết hợp nhiều giá trị bằng `\|` (bitmask), ví dụ quyền đọc/ghi |
| `auto()` | tự sinh giá trị |

## Giải thích đơn giản

Enum giống **menu chọn trạng thái đơn hàng** trong app: chỉ có "Chờ xử lý", "Đang giao", "Đã giao", "Đã hủy". Không ai gõ tay được "Đang gaio" (sai chính tả) vì chỉ được chọn trong menu.

So với dùng chuỗi trần `"shipping"`: gõ sai chính tả thì code vẫn chạy và bug âm thầm. Gõ sai `Status.SHIPING` thì báo `AttributeError` ngay, editor còn gợi ý tên.

## Ví dụ

### Enum cơ bản

```python
from enum import Enum, auto

class Status(Enum):
    PENDING = "pending"
    SHIPPING = "shipping"
    DONE = "done"
    CANCELLED = "cancelled"

s = Status.SHIPPING
print(s)              # Status.SHIPPING
print(s.name, s.value)    # SHIPPING shipping
print(s is Status.SHIPPING)   # True

print(Status("done"))         # Status.DONE  (tra theo value, ví dụ từ DB/JSON)
print(Status["PENDING"])      # Status.PENDING (tra theo name)
Status("lost")                # ValueError

for st in Status:
    print(st.name)            # PENDING SHIPPING DONE CANCELLED

print(Status.DONE == "done")  # False  -> Enum thường không bằng giá trị thô
```

### auto & method

```python
class Priority(Enum):
    LOW = auto()      # 1
    MEDIUM = auto()   # 2
    HIGH = auto()     # 3

    def label(self) -> str:
        return {"LOW": "Thấp", "MEDIUM": "Vừa", "HIGH": "Cao"}[self.name]

    @property
    def is_urgent(self) -> bool:
        return self is Priority.HIGH

print(Priority.HIGH.value, Priority.HIGH.label(), Priority.HIGH.is_urgent)   # 3 Cao True
```

### StrEnum: tiện khi làm việc với JSON / API

```python
from enum import StrEnum
import json

class Role(StrEnum):
    ADMIN = "admin"
    USER = "user"

print(Role.ADMIN == "admin")                  # True
print(json.dumps({"role": Role.ADMIN}))       # {"role": "admin"}
print(f"Quyền: {Role.USER}")                  # Quyền: user
```

### Flag: kết hợp nhiều giá trị

```python
from enum import Flag, auto

class Perm(Flag):
    READ = auto()
    WRITE = auto()
    DELETE = auto()

editor = Perm.READ | Perm.WRITE
print(Perm.WRITE in editor)     # True
print(Perm.DELETE in editor)    # False
print(editor)                   # Perm.READ|WRITE
```

### Dùng với match

```python
def next_step(s: Status) -> str:
    match s:
        case Status.PENDING:
            return "Đóng gói"
        case Status.SHIPPING:
            return "Chờ giao"
        case Status.DONE | Status.CANCELLED:
            return "Kết thúc"
```

## So sánh với JavaScript / TypeScript

| JS / TS | Python |
|---|---|
| `Object.freeze({ RED: "red" })` | `class Color(Enum): RED = "red"` |
| TS `enum Color { Red = "red" }` | `class Color(StrEnum): RED = "red"` |
| TS union `"a" \| "b"` | `Literal["a", "b"]` (type hint) hoặc Enum |
| `Object.values(Color)` | `list(Color)` |
| bitmask thủ công `READ \| WRITE` | `Flag` |

## Xem thêm

- [Pattern Matching (match / case)](./27-Match-Case.md)
- [Class](./39-Class.md)
- [Type Hints](./41-Type-Hints.md)
- [JSON](./03-JSON.md)
