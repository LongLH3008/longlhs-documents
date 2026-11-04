[← Mục lục](./index.md)

# Tuple

## Định nghĩa

`tuple` là kiểu **sequence immutable** (bất biến), có thứ tự. Sau khi tạo, không thể thêm, xóa, hay thay phần tử. Vì bất biến (và nếu mọi phần tử bên trong cũng hashable), tuple là **hashable**, nên có thể làm **key của dict** hoặc phần tử của **set**.

`collections.namedtuple` và `typing.NamedTuple` là biến thể có tên cho từng vị trí.

## Giải thích đơn giản

Nếu `list` là **tờ giấy viết bút chì** (tẩy, sửa thoải mái) thì `tuple` là **tờ giấy đã ép plastic**: đọc được, nhưng không sửa được.

JS không có tuple thật. Thứ gần nhất là `Object.freeze([1, 2])` hoặc kiểu `[string, number]` trong TypeScript. Trong Python, tuple dùng khắp nơi: hàm trả về nhiều giá trị, `dict.items()`, tọa độ `(x, y)`...

Dấu `,` mới tạo ra tuple, không phải dấu `()`.

## Ví dụ

### Tạo tuple

```python
point = (3, 4)
also_tuple = 3, 4       # không cần ngoặc
single = (5,)           # tuple 1 phần tử BẮT BUỘC có dấu phẩy
not_tuple = (5)         # đây chỉ là số 5
empty = ()

print(point[0], point[-1])   # 3 4
print(len(point))            # 2
point[0] = 10                # TypeError: 'tuple' object does not support item assignment
```

### Hàm trả về nhiều giá trị

```python
def min_max(nums):
    return min(nums), max(nums)   # thực chất trả về một tuple

lo, hi = min_max([4, 1, 9])       # unpacking
print(lo, hi)                     # 1 9
```

### Làm key của dict

```python
distances = {
    ("HN", "HCM"): 1700,
    ("HN", "DN"): 760,
}
print(distances[("HN", "DN")])   # 760

# list không làm key được
{[1, 2]: "x"}   # TypeError: unhashable type: 'list'
```

### NamedTuple: tuple có tên trường

```python
from typing import NamedTuple

class Point(NamedTuple):
    x: float
    y: float

p = Point(3, 4)
print(p.x, p.y)        # 3 4
print(p[0])            # 3, vẫn truy cập theo index được
x, y = p               # vẫn unpack được
p2 = p._replace(x=10)  # tạo bản mới, bản cũ không đổi
```

### Bất biến "nông"

```python
t = (1, [2, 3])
t[1].append(4)   # được! list bên trong vẫn mutable
print(t)         # (1, [2, 3, 4])
```

## Khi nào dùng tuple thay list

- Dữ liệu cố định, có ý nghĩa theo vị trí: tọa độ, bản ghi `(id, name)`, RGB `(255, 0, 0)`.
- Cần làm key dict hoặc phần tử set.
- Muốn báo cho người đọc rằng "dữ liệu này không nên bị sửa".

## So sánh với JavaScript

| JavaScript / TypeScript | Python |
|---|---|
| `Object.freeze([1, 2])` | `(1, 2)` |
| type `[number, number]` (TS) | `tuple[int, int]` |
| `return [a, b]` rồi destructure | `return a, b` |
| `Map` với key là chuỗi `"HN-HCM"` | `dict` với key là tuple `("HN", "HCM")` |

## Xem thêm

- [Array (JS)](../js/13-Array.md)
- [List](./13-List.md)
- [Unpacking](./20-Unpacking.md)
- [Mutable & Immutable](./35-Mutable-Immutable.md)
