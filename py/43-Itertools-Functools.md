[← Mục lục](./index.md)

# itertools & functools

## Định nghĩa

Hai module chuẩn hỗ trợ lập trình theo phong cách hàm (functional) và xử lý iterator hiệu quả:

- **`itertools`**: tập hàm tạo và kết hợp **iterator lazy** (sinh phần tử khi cần, không tạo list trung gian), viết bằng C nên nhanh. Gồm 3 nhóm: iterator vô hạn (`count`, `cycle`, `repeat`), iterator kết thúc theo đầu vào (`chain`, `islice`, `groupby`, `accumulate`, `batched`, `pairwise`, `takewhile`, `zip_longest`...), và tổ hợp (`product`, `permutations`, `combinations`).
- **`functools`**: công cụ làm việc với **hàm như dữ liệu** (higher-order function): `partial`, `reduce`, `lru_cache` / `cache`, `cached_property`, `wraps`, `total_ordering`, `singledispatch`.

## Giải thích đơn giản

- `itertools` là **băng chuyền trong nhà máy**: các công đoạn nối với nhau (lọc, cắt, ghép, nhóm), mỗi sản phẩm đi qua từng khâu **một lúc một cái**, không cần kho chứa hết hàng ở giữa. Nhờ vậy xử lý file log 10 GB vẫn không tràn RAM.
- `functools` là **bộ phụ kiện gắn vào hàm**: `partial` là **cài sẵn vài nút bấm** để gọi gọn hơn, `lru_cache` là **cuốn sổ ghi nhớ đáp án** để lần sau hỏi lại thì trả ngay.

## Ví dụ: itertools

### Nối, cắt, đếm

```python
from itertools import chain, islice, count, cycle, repeat

print(list(chain([1, 2], (3,), "ab")))      # [1, 2, 3, 'a', 'b']
print(list(chain.from_iterable([[1], [2, 3]])))   # [1, 2, 3]  làm phẳng 1 tầng

print(list(islice(count(10, 5), 4)))         # [10, 15, 20, 25]  cắt iterator vô hạn
print(list(islice(cycle("AB"), 5)))          # ['A', 'B', 'A', 'B', 'A']
print(list(repeat("x", 3)))                  # ['x', 'x', 'x']
```

### Gom nhóm: groupby

`groupby` chỉ gom các phần tử **liền kề** cùng key, nên thường phải `sorted` trước:

```python
from itertools import groupby

orders = [("An", 50), ("Bình", 20), ("An", 30), ("Chi", 10)]
orders.sort(key=lambda o: o[0])

for name, group in groupby(orders, key=lambda o: o[0]):
    print(name, sum(amount for _, amount in group))
# An 80 / Bình 20 / Chi 10
```

### Chia lô, cặp liền kề, cộng dồn

```python
from itertools import batched, pairwise, accumulate   # batched: 3.12+
import operator

ids = range(1, 8)
for batch in batched(ids, 3):
    print(batch)            # (1, 2, 3) / (4, 5, 6) / (7,)

prices = [100, 105, 98, 110]
print([b - a for a, b in pairwise(prices)])        # [5, -7, 12]

print(list(accumulate([1, 2, 3, 4])))              # [1, 3, 6, 10]  tổng dồn
print(list(accumulate([3, 1, 4, 1, 5], max)))      # [3, 3, 4, 4, 5]  max dồn
print(list(accumulate([1, 2, 3, 4], operator.mul)))# [1, 2, 6, 24]
```

### takewhile, dropwhile, zip_longest

```python
from itertools import takewhile, dropwhile, zip_longest

nums = [1, 3, 5, 6, 7]
print(list(takewhile(lambda n: n % 2, nums)))   # [1, 3, 5]  lấy tới khi sai
print(list(dropwhile(lambda n: n % 2, nums)))   # [6, 7]     bỏ tới khi sai

print(list(zip_longest("abc", [1], fillvalue=0)))   # [('a', 1), ('b', 0), ('c', 0)]
```

### Tổ hợp

```python
from itertools import product, permutations, combinations

sizes, colors = ["S", "M"], ["đỏ", "xanh"]
print(list(product(sizes, colors)))
# [('S', 'đỏ'), ('S', 'xanh'), ('M', 'đỏ'), ('M', 'xanh')]

print(list(permutations("ABC", 2)))   # 6 cặp có thứ tự: AB AC BA BC CA CB
print(list(combinations("ABC", 2)))   # 3 cặp không thứ tự: AB AC BC
print(list(product([0, 1], repeat=3))[:3])   # [(0,0,0), (0,0,1), (0,1,0)]
```

### Pipeline lazy đọc file lớn

```python
from itertools import islice

def errors(path):
    with open(path, encoding="utf-8") as f:
        lines = (line.rstrip() for line in f)          # generator
        yield from (l for l in lines if "ERROR" in l)

for line in islice(errors("app.log"), 10):   # chỉ đọc tới khi đủ 10 dòng lỗi
    print(line)
```

## Ví dụ: functools

### partial: cố định bớt tham số

```python
from functools import partial

def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
cube = partial(power, exp=3)
print(square(5), cube(2))       # 25 8

to_int_from_bin = partial(int, base=2)
print(to_int_from_bin("1010"))  # 10
```

### reduce: gộp về một giá trị

```python
from functools import reduce

print(reduce(lambda acc, x: acc * x, [1, 2, 3, 4], 1))   # 24

# Thường có cách dễ đọc hơn: sum, max, math.prod, "".join
import math
print(math.prod([1, 2, 3, 4]))   # 24
```

### cache & lru_cache: memoization

```python
from functools import cache, lru_cache

@cache                     # không giới hạn (3.9+)
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)

print(fib(80))             # 23416728348467685

@lru_cache(maxsize=128)    # giữ 128 kết quả gần nhất
def get_user(user_id: int):
    print("query DB", user_id)
    return {"id": user_id}

get_user(1); get_user(1)   # chỉ in "query DB 1" một lần
print(get_user.cache_info())   # CacheInfo(hits=1, misses=1, maxsize=128, currsize=1)
```

Tham số phải **hashable** (không truyền list/dict được).

### cached_property

```python
from functools import cached_property

class Report:
    def __init__(self, rows):
        self.rows = rows

    @cached_property
    def total(self):          # tính lần đầu, các lần sau trả giá trị đã lưu
        print("đang tính...")
        return sum(self.rows)

r = Report([1, 2, 3])
print(r.total, r.total)       # đang tính... / 6 6
```

### total_ordering & singledispatch

```python
from functools import total_ordering, singledispatch

@total_ordering               # chỉ cần __eq__ và __lt__, tự sinh <=, >, >=
class Version:
    def __init__(self, s):
        self.parts = tuple(map(int, s.split(".")))
    def __eq__(self, o):
        return self.parts == o.parts
    def __lt__(self, o):
        return self.parts < o.parts

print(Version("1.10.0") > Version("1.9.3"))   # True

@singledispatch               # overload theo kiểu tham số đầu
def to_text(x):
    return str(x)

@to_text.register
def _(x: list):
    return ", ".join(map(to_text, x))

@to_text.register
def _(x: float):
    return f"{x:.2f}"

print(to_text([1, 2.5, "a"]))  # 1, 2.50, a
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `[...a, ...b]` | `chain(a, b)` (lazy) |
| `arr.flat()` | `chain.from_iterable(arr)` |
| `arr.slice(0, n)` trên generator | `islice(it, n)` |
| `Object.groupBy` | `groupby` (cần sort) hoặc `defaultdict(list)` |
| `arr.reduce(fn, init)` | `functools.reduce(fn, it, init)` |
| `fn.bind(null, a)` | `partial(fn, a)` |
| lodash `memoize` | `@cache` / `@lru_cache` |
| lodash `chunk` | `itertools.batched` |
| không có sẵn | `product`, `permutations`, `combinations` |

## Xem thêm

- [Iterator & Generator](./42-Iterator-Generator.md)
- [First-class & Higher-order Function](./10-First-class-Higher-order-Function.md)
- [Decorator](./44-Decorator.md)
- [Lambda](./29-Lambda.md)
- [Module collections](./24-Collections.md)
- [Đệ quy (Recursion)](./34-Recursion.md)
