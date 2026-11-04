[← Mục lục](./index.md)

# Memory Management & Garbage Collection

## Định nghĩa

CPython quản lý bộ nhớ **tự động** bằng hai cơ chế kết hợp:

1. **Reference counting** (cơ chế chính): mỗi object có trường `ob_refcnt` đếm số tham chiếu trỏ tới nó. Tạo tham chiếu (gán tên, đưa vào list, truyền tham số) thì tăng, mất tham chiếu (`del`, gán lại, ra khỏi scope) thì giảm. Khi về **0**, object được giải phóng **ngay lập tức** và `__del__` (nếu có) được gọi.
2. **Cyclic garbage collector** (module `gc`): reference counting không xử lý được **vòng tham chiếu** (A trỏ B, B trỏ A). GC định kỳ quét các object container (list, dict, instance...) để tìm và thu hồi các nhóm vòng không còn ai bên ngoài trỏ tới. GC chia object thành các **thế hệ (generation)**: object mới ở thế hệ trẻ, được quét thường xuyên; sống sót qua nhiều lần quét thì được đẩy lên thế hệ già, ít quét hơn.

Bên dưới, CPython dùng bộ cấp phát riêng **pymalloc** cho object nhỏ (≤ 512 byte), chia bộ nhớ thành arena → pool → block để giảm chi phí gọi `malloc` của hệ điều hành.

## Giải thích đơn giản

Mỗi object giống **một quả bóng bay** có buộc nhiều sợi dây (tham chiếu). Còn ít nhất một người cầm dây thì bóng ở lại. Sợi dây cuối cùng được thả ra là bóng **bay mất ngay** (bộ nhớ được trả lại). Đó là reference counting.

Vấn đề: hai quả bóng **buộc dây vào nhau** nhưng không ai cầm cả. Mỗi quả vẫn có "một sợi dây" nên tưởng là còn dùng, nhưng thực ra đã bị bỏ rơi. Thỉnh thoảng có **nhân viên dọn dẹp** (cyclic GC) đi kiểm tra những cụm bóng như vậy và cắt bỏ.

Khác JS: V8 không dùng reference counting mà chỉ dùng **mark-and-sweep** theo thế hệ, nên thời điểm giải phóng không xác định. Trong CPython, đa số object được giải phóng **ngay khi hết tham chiếu**, có thể đoán trước được.

## Ví dụ

### Quan sát reference count

```python
import sys

a = []
print(sys.getrefcount(a))   # 2  (a + tham số tạm của chính getrefcount)

b = a                       # thêm một tên
container = [a]             # thêm một tham chiếu từ list
print(sys.getrefcount(a))   # 4

del b
container.clear()
print(sys.getrefcount(a))   # 2
```

### Giải phóng ngay khi hết tham chiếu

```python
class Resource:
    def __init__(self, name):
        self.name = name
        print("mở", name)
    def __del__(self):
        print("giải phóng", self.name)

r = Resource("A")    # mở A
r = Resource("B")    # mở B / giải phóng A   <- A mất tham chiếu cuối, bị hủy ngay
del r                # giải phóng B
```

Không nên dựa vào `__del__` để đóng file, kết nối: dùng `with` (xem [Context Manager](./45-Context-Manager.md)).

### Vòng tham chiếu & gc

```python
import gc

class Node:
    def __init__(self):
        self.other = None

a, b = Node(), Node()
a.other, b.other = b, a     # vòng: a -> b -> a
del a, b                    # refcount mỗi node vẫn là 1, chưa được giải phóng

print(gc.collect())         # số object không truy cập được đã thu hồi (gồm cả 2 Node)
print(gc.get_threshold())   # (2000, 10, 10) ở 3.13, bản cũ là (700, 10, 10): ngưỡng kích hoạt từng thế hệ
```

### weakref: tham chiếu không giữ object sống

```python
import weakref

class BigImage:
    pass

img = BigImage()
cache = weakref.WeakValueDictionary()
cache["logo"] = img          # không làm tăng refcount

print("logo" in cache)       # True
del img
print("logo" in cache)       # False  -> tự biến mất khi object bị hủy
```

Hữu ích cho cache, observer, quan hệ cha-con để tránh tạo vòng.

### Interning & object nhỏ dùng chung

```python
a = 256
b = 256
print(a is b)       # True: CPython cache sẵn int từ -5 tới 256

x = int("1000")
y = int("1000")
print(x is y)       # False: hai object khác nhau, dù x == y

s1 = "hello"
s2 = "hello"
print(s1 is s2)     # True (thường): chuỗi giống identifier được intern
```

Đây là chi tiết cài đặt, **không** dựa vào nó: so sánh giá trị luôn dùng `==`, chỉ dùng `is` cho `None`, `True`, `False`.

### Đo bộ nhớ & tìm rò rỉ

```python
import sys, tracemalloc

print(sys.getsizeof([]))                 # 56  (byte, chỉ tính vỏ list)
print(sys.getsizeof([0] * 1000))         # ~8056: mỗi phần tử là con trỏ 8 byte

tracemalloc.start()
data = [str(i) * 10 for i in range(100_000)]
current, peak = tracemalloc.get_traced_memory()
print(f"hiện tại {current / 1e6:.1f} MB, đỉnh {peak / 1e6:.1f} MB")

for stat in tracemalloc.take_snapshot().statistics("lineno")[:3]:
    print(stat)                          # dòng code cấp phát nhiều nhất
```

### Tiết kiệm bộ nhớ

```python
# 1. Generator thay vì list khi chỉ cần duyệt một lần
total = sum(x * x for x in range(10_000_000))   # gần như không tốn RAM

# 2. __slots__ bỏ __dict__ của từng instance
class Point:
    __slots__ = ("x", "y")
    def __init__(self, x, y):
        self.x, self.y = x, y
```

## So sánh với JavaScript

| Khía cạnh | JavaScript (V8) | Python (CPython) |
|---|---|---|
| Cơ chế chính | tracing GC (mark-sweep-compact), theo thế hệ | reference counting |
| Xử lý vòng tham chiếu | tự nhiên (tracing) | cyclic GC bổ sung |
| Thời điểm giải phóng | không xác định | đa số là ngay khi refcount = 0 |
| Gọi GC thủ công | không (chỉ có cờ `--expose-gc`) | `gc.collect()` |
| Tham chiếu yếu | `WeakMap`, `WeakRef` | `weakref` |
| Hàm hủy | `FinalizationRegistry` | `__del__`, `weakref.finalize` |

## Xem thêm

- [Mutable & Immutable](./47-Mutable-Immutable.md)
- [Name Binding (không có Hoisting)](./49-Name-Binding.md)
- [Context Manager (with)](./45-Context-Manager.md)
- [Data Model, Dunder Methods & MRO](./52-Data-Model-MRO.md)
- [Iterator & Generator](./42-Iterator-Generator.md)
- [GIL, Threading & Multiprocessing](./56-GIL-Concurrency.md)
