[← Mục lục](./index.md)

# Number

## Định nghĩa

Python có ba kiểu số built-in và hai kiểu số chính xác trong thư viện chuẩn:

| Kiểu | Mô tả |
|---|---|
| `int` | số nguyên, **độ chính xác tùy ý** (arbitrary precision), không tràn số |
| `float` | số thực dấu phẩy động 64-bit (IEEE 754), giống `number` của JS |
| `complex` | số phức, ví dụ `3+4j` |
| `decimal.Decimal` | số thập phân chính xác, dùng cho tiền tệ |
| `fractions.Fraction` | phân số chính xác, ví dụ `1/3` |

## Giải thích đơn giản

JS chỉ có một kiểu `number` (luôn là float), nên số nguyên lớn hơn `2^53` bị mất chính xác và phải dùng `BigInt`. Python `int` giống BigInt **mặc định**: muốn lớn bao nhiêu cũng được, chỉ tốn thêm bộ nhớ.

`float` của Python vẫn mắc lỗi làm tròn kinh điển `0.1 + 0.2 != 0.3` giống hệt JS, vì cả hai cùng dùng chuẩn IEEE 754. Khi làm việc với **tiền**, hãy dùng `Decimal`, giống như dùng thước kẹp thay vì ước lượng bằng mắt.

## Ví dụ

### int không giới hạn

```python
print(2 ** 100)
# 1267650600228229401496703205376

big = 10 ** 30 + 1
print(big - 10 ** 30)   # 1, không mất chính xác
```

### Viết số dễ đọc, hệ cơ số

```python
price = 1_000_000          # dấu _ chỉ để dễ đọc
print(0b1010, 0o17, 0xFF)  # 10 15 255
print(bin(10), hex(255))   # 0b1010 0xff
print(int("ff", 16))       # 255
```

### float và lỗi làm tròn

```python
print(0.1 + 0.2)               # 0.30000000000000004
print(0.1 + 0.2 == 0.3)        # False

import math
print(math.isclose(0.1 + 0.2, 0.3))  # True, so sánh có sai số

print(float("inf"), float("nan"))    # inf nan
print(math.isnan(float("nan")))      # True
```

### Decimal cho tiền tệ

```python
from decimal import Decimal, ROUND_HALF_UP

a = Decimal("0.1") + Decimal("0.2")
print(a)                       # 0.3
print(a == Decimal("0.3"))     # True

price = Decimal("19.995")
print(price.quantize(Decimal("0.01"), rounding=ROUND_HALF_UP))  # 20.00
```

Truyền **chuỗi** vào `Decimal`, không truyền float: `Decimal(0.1)` sẽ mang theo sai số của float.

### Phép chia

```python
print(10 / 4)          # 2.5  luôn ra float
print(10 // 4)         # 2    chia lấy nguyên
print(divmod(10, 4))   # (2, 2) -> (thương, dư)
```

### Chuyển đổi và format

```python
print(int("42"), float("3.5"))   # 42 3.5
print(round(3.14159, 2))         # 3.14
print(f"{1234567.891:,.2f}")     # 1,234,567.89
print(f"{0.256:.1%}")            # 25.6%
print((3.0).is_integer())        # True
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `number` (float 64-bit) | `float` |
| `BigInt` (`10n`) | `int` (mặc định đã không giới hạn) |
| `Number.MAX_SAFE_INTEGER` | không có giới hạn với `int` |
| `parseInt("42")` | `int("42")` |
| `parseFloat("3.5")` | `float("3.5")` |
| `Number.isNaN(x)` | `math.isnan(x)` |
| `x.toFixed(2)` | `f"{x:.2f}"` |
| `Infinity` | `float("inf")` / `math.inf` |

## Xem thêm

- [Number (JS)](../js/11-Number.md)
- [datetime & math](./03-Datetime-Math.md)
- [f-string](./18-f-string.md)
