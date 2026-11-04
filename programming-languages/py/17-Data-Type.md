[← Mục lục](./index.md)

# Data Type

## Định nghĩa

Python là ngôn ngữ **dynamic typing** (kiểu gắn với giá trị, không gắn với biến) và **strong typing** (không tự ép kiểu ngầm giữa các kiểu không liên quan). Mọi thứ trong Python đều là **object**, kể cả số, hàm, class.

Các kiểu built-in chính:

| Nhóm | Kiểu | Ví dụ | Mutable? |
|---|---|---|---|
| Số | `int`, `float`, `complex` | `42`, `3.14`, `2+3j` | Không |
| Logic | `bool` | `True`, `False` | Không |
| Chuỗi | `str` | `"hello"` | Không |
| Rỗng | `NoneType` | `None` | Không |
| Dãy | `list` | `[1, 2]` | Có |
| Dãy | `tuple` | `(1, 2)` | Không |
| Dãy | `range` | `range(5)` | Không |
| Ánh xạ | `dict` | `{"a": 1}` | Có |
| Tập hợp | `set`, `frozenset` | `{1, 2}` | `set` có, `frozenset` không |
| Nhị phân | `bytes`, `bytearray` | `b"abc"` | `bytes` không, `bytearray` có |

## Giải thích đơn giản

So với JS:

- JS có `undefined` **và** `null`; Python chỉ có một: `None`. Truy cập biến chưa gán trong Python là **lỗi** (`NameError`), không ra `undefined`.
- JS có một kiểu `number`; Python tách `int` (số nguyên, **không giới hạn độ lớn**) và `float`.
- JS Array ≈ Python `list`. JS Object (dùng làm map) ≈ Python `dict`.
- JS ép kiểu ngầm (`"5" + 1 = "51"`), Python báo lỗi `TypeError`. Python giống một người thủ thư khó tính: sách toán không được xếp lẫn sách văn.

## Ví dụ

### Kiểm tra kiểu

```python
print(type(42))         # <class 'int'>
print(type(3.0))        # <class 'float'>
print(type("hi"))       # <class 'str'>
print(type(None))       # <class 'NoneType'>
print(type([1, 2]))     # <class 'list'>

print(isinstance(42, int))            # True
print(isinstance(True, int))          # True  (bool là subclass của int)
print(isinstance(3.0, (int, float)))  # True, kiểm tra nhiều kiểu
```

### Strong typing: không ép kiểu ngầm

```python
"5" + 1          # TypeError: can only concatenate str (not "int") to str
"5" + str(1)     # '51'
int("5") + 1     # 6
```

### Ép kiểu tường minh

```python
int("42")       # 42
int(3.99)       # 3 (cắt phần thập phân)
float("1.5")    # 1.5
str(100)        # '100'
bool("")        # False
list("abc")     # ['a', 'b', 'c']
tuple([1, 2])   # (1, 2)
set([1, 1, 2])  # {1, 2}
int("abc")      # ValueError
```

### Dynamic typing

```python
x = 10        # x trỏ tới int
x = "mười"    # giờ trỏ tới str, hợp lệ
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `typeof x` | `type(x)` / `isinstance(x, T)` |
| `undefined`, `null` | `None` |
| `number`, `bigint` | `int` (không giới hạn), `float` |
| `boolean` (`true`) | `bool` (`True`, viết hoa) |
| `Array` | `list` |
| `Object` / `Map` | `dict` |
| `Set` | `set` |
| `Symbol` | không có tương đương trực tiếp |
| ép kiểu ngầm | báo `TypeError` |

## Xem thêm

- [Data Type (JS)](../js/10-Data-Type.md)
- [Number](./18-Number.md)
- [Mutable & Immutable](./47-Mutable-Immutable.md)
- [Type Hints](./41-Type-Hints.md)
