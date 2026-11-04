[← Mục lục](./index.md)

# datetime & math

## Định nghĩa

- `datetime` là module chuẩn xử lý ngày giờ, gồm các class chính: `date` (ngày), `time` (giờ), `datetime` (ngày + giờ), `timedelta` (khoảng thời gian), `timezone`.
- `math` là module chuẩn chứa hằng số và hàm toán học (`pi`, `sqrt`, `floor`, `ceil`...).
- `random` là module sinh số ngẫu nhiên (giả ngẫu nhiên).

## Giải thích đơn giản

Trong JS, `Date` gom hết mọi thứ vào một object. Python tách nhỏ: muốn ngày thì dùng `date`, muốn cộng trừ thời gian thì dùng `timedelta`. Giống như thay vì một con dao đa năng, Python đưa bạn một bộ dao chuyên dụng.

Điểm dễ nhầm nhất: trong JS tháng bắt đầu từ `0` (tháng 1 là `0`), còn Python tháng bắt đầu từ `1` như cách con người đếm.

`Math` của JS là object global, còn Python phải `import math`.

## Ví dụ

### datetime

```python
from datetime import datetime, date, timedelta, timezone

now = datetime.now()                 # giờ địa phương
utc_now = datetime.now(timezone.utc) # giờ UTC, có timezone

d = date(2026, 1, 15)                # tháng 1 là 1 (JS là 0)
print(d.year, d.month, d.day)        # 2026 1 15
print(d.weekday())                   # 3 (thứ Năm, Thứ Hai = 0)

# Cộng trừ thời gian
tomorrow = d + timedelta(days=1)
print(tomorrow)                      # 2026-01-16

# Khoảng cách giữa hai ngày
diff = date(2026, 12, 31) - d
print(diff.days)                     # 350

# Format và parse
dt = datetime(2026, 1, 15, 9, 30)
print(dt.strftime("%d/%m/%Y %H:%M")) # 15/01/2026 09:30
print(datetime.strptime("15/01/2026", "%d/%m/%Y"))  # 2026-01-15 00:00:00

# ISO 8601
print(dt.isoformat())                # 2026-01-15T09:30:00
print(datetime.fromisoformat("2026-01-15T09:30:00"))

# Timestamp (giây, không phải mili giây như JS)
print(dt.timestamp())
```

### math & random

```python
import math
import random

print(math.pi)          # 3.141592653589793
print(math.sqrt(16))    # 4.0
print(math.floor(4.7))  # 4
print(math.ceil(4.1))   # 5
print(math.trunc(-4.7)) # -4
print(round(2.5))       # 2  (làm tròn kiểu banker: về số chẵn gần nhất)
print(round(3.14159, 2))# 3.14
print(abs(-5), max(1, 9, 3), min([4, 2, 8]))  # 5 9 2  (hàm built-in, không cần import)

print(random.random())        # float trong [0, 1)
print(random.randint(1, 6))   # int trong [1, 6], gồm cả 6
print(random.choice(["a", "b", "c"]))
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `new Date()` | `datetime.now()` |
| `new Date(2026, 0, 15)` | `date(2026, 1, 15)` |
| `Date.now()` (ms) | `time.time()` (giây, float) |
| `date.toISOString()` | `dt.isoformat()` |
| `Math.floor(x)` | `math.floor(x)` |
| `Math.round(2.5)` → `3` | `round(2.5)` → `2` |
| `Math.random()` | `random.random()` |
| `Math.max(...arr)` | `max(arr)` |

## Lưu ý

- Ưu tiên datetime **có timezone** (aware) khi lưu DB hoặc gửi API, tránh lẫn giờ địa phương.
- Không dùng `random` cho mục đích bảo mật (token, mật khẩu), hãy dùng module `secrets`.

## Xem thêm

- [Date & Math (JS)](../js/03-Date-Math.md)
- [Number](./11-Number.md)
