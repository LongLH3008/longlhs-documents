[← Mục lục](./index.md)

# Exception

## Định nghĩa

**Exception** là object biểu diễn lỗi xảy ra lúc chạy. Khi một exception được **raise** mà không ai **xử lý**, chương trình dừng và in **traceback**.

Cú pháp xử lý:

```python
try:
    ...                 # code có thể lỗi
except SomeError as e:
    ...                 # xử lý loại lỗi cụ thể
else:
    ...                 # chạy khi try KHÔNG lỗi
finally:
    ...                 # luôn chạy (dọn dẹp)
```

Mọi exception kế thừa từ `BaseException`; exception thông thường kế thừa từ `Exception`. Một số loại phổ biến: `ValueError`, `TypeError`, `KeyError`, `IndexError`, `AttributeError`, `FileNotFoundError`, `ZeroDivisionError`.

## Giải thích đơn giản

Tương đương `try/catch/finally` của JS, với hai điểm khác:

1. Python bắt lỗi **theo loại** (`except KeyError`), JS phải `catch (e)` rồi `if (e instanceof ...)`.
2. Python dùng exception **nhiều hơn** JS. Triết lý **EAFP** ("Easier to Ask Forgiveness than Permission"): cứ làm, lỗi thì xử lý, thay vì kiểm tra trước mọi thứ (LBYL, "Look Before You Leap").

Ví von: EAFP giống **cứ mở cửa, cửa khóa thì tìm chìa**; LBYL giống **gõ cửa hỏi trước rồi mới mở**.

## Ví dụ

### try / except / else / finally

```python
def safe_divide(a, b):
    try:
        result = a / b
    except ZeroDivisionError:
        print("Không chia được cho 0")
        return None
    except TypeError as e:
        print("Sai kiểu:", e)
        return None
    else:
        print("Chia thành công")
        return result
    finally:
        print("Kết thúc")

safe_divide(10, 2)
# Chia thành công
# Kết thúc
safe_divide(1, 0)
# Không chia được cho 0
# Kết thúc
```

### Bắt nhiều loại cùng lúc

```python
try:
    value = int(input("Nhập số: "))
except (ValueError, KeyboardInterrupt):
    value = 0
```

### Raise exception

```python
def withdraw(balance, amount):
    if amount <= 0:
        raise ValueError("Số tiền phải lớn hơn 0")   # JS: throw new Error(...)
    if amount > balance:
        raise ValueError(f"Không đủ số dư: {balance}")
    return balance - amount
```

### Custom exception

```python
class InsufficientFundsError(Exception):
    def __init__(self, balance, amount):
        super().__init__(f"Cần {amount}, chỉ có {balance}")
        self.balance = balance
        self.amount = amount

try:
    raise InsufficientFundsError(100, 500)
except InsufficientFundsError as e:
    print(e)            # Cần 500, chỉ có 100
    print(e.amount)     # 500
```

### Exception chaining

```python
try:
    config = {}["db"]
except KeyError as e:
    raise RuntimeError("Thiếu cấu hình database") from e
# Traceback hiển thị cả lỗi gốc KeyError lẫn RuntimeError
```

### EAFP vs LBYL

```python
user = {"name": "An"}

# LBYL
if "email" in user:
    email = user["email"]
else:
    email = None

# EAFP
try:
    email = user["email"]
except KeyError:
    email = None
```

### Re-raise và bắt lỗi chung

```python
import logging

try:
    risky()
except Exception:
    logging.exception("Lỗi khi chạy risky")   # log kèm traceback
    raise                                       # ném lại lỗi gốc
```

Không dùng `except:` trống (bắt cả `KeyboardInterrupt`, `SystemExit`). Tối thiểu là `except Exception:`.

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `try { } catch (e) { } finally { }` | `try: / except E as e: / finally:` |
| `throw new Error("x")` | `raise ValueError("x")` |
| `class MyErr extends Error` | `class MyErr(Exception):` |
| `if (e instanceof TypeError)` | `except TypeError:` |
| `new Error("x", { cause: e })` | `raise X("x") from e` |
| không có | `else:` chạy khi không lỗi |
| `e.message` | `str(e)` / `e.args` |
| `AggregateError` | `ExceptionGroup` + `except*` (3.11+) |

## Xem thêm

- [Promise (JS)](../js/41-Promise.md)
- [Context Manager (with)](./33-Context-Manager.md)
- [Class](./28-Class.md)
- [Control Flow & match](./25-Control-Flow-Match.md)
