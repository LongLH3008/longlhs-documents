[← Mục lục](./index.md)

# Module collections

## Định nghĩa

`collections` là module chuẩn cung cấp các **container chuyên dụng**, bổ sung cho `list`, `tuple`, `dict`, `set`:

| Kiểu | Bản chất | Dùng khi |
|---|---|---|
| `Counter` | subclass của `dict`, value là số đếm | đếm tần suất |
| `defaultdict` | subclass của `dict`, tự tạo value mặc định khi thiếu key | gom nhóm, đếm, không muốn kiểm tra key |
| `deque` | **double-ended queue** (danh sách liên kết các block), thêm/bớt hai đầu `O(1)` | hàng đợi, stack, cửa sổ trượt |
| `namedtuple` | tuple có **tên trường** | bản ghi nhẹ, bất biến |
| `OrderedDict` | dict nhớ thứ tự, có `move_to_end`, `popitem(last=False)` | LRU cache tự viết |
| `ChainMap` | gộp nhiều dict thành một view, tra theo thứ tự | cấu hình nhiều tầng (mặc định < file < env) |

## Giải thích đơn giản

Nếu `list`, `dict` là **hộp đựng đồ đa năng**, thì `collections` là **bộ hộp chuyên dụng**:

- `Counter` là **máy đếm phiếu bầu**: đổ phiếu vào, nó tự đếm mỗi tên bao nhiêu phiếu.
- `defaultdict` là **tủ hồ sơ tự tạo ngăn**: mở ngăn chưa có thì ngăn mới tự xuất hiện, không báo lỗi.
- `deque` là **hàng người xếp ở quầy có hai cửa**: vào ra ở đầu nào cũng nhanh. `list.pop(0)` thì giống bắt cả hàng dịch lên một bước.
- `namedtuple` là **phiếu thông tin in sẵn tiêu đề cột**: gọi `p.x` thay vì nhớ "vị trí 0 là x".

## Ví dụ

### Counter

```python
from collections import Counter

words = "táo cam táo xoài cam táo".split()
c = Counter(words)
print(c)                    # Counter({'táo': 3, 'cam': 2, 'xoài': 1})
print(c["táo"], c["mít"])   # 3 0   (key thiếu trả 0, không KeyError)
print(c.most_common(2))     # [('táo', 3), ('cam', 2)]

c.update(["mít", "mít"])
print(c["mít"])             # 2

# Phép toán giữa các Counter
a = Counter(a=3, b=1)
b = Counter(a=1, b=2)
print(a + b)    # Counter({'a': 4, 'b': 3})
print(a - b)    # Counter({'a': 2})   (bỏ số <= 0)

# Kiểm tra anagram
print(Counter("listen") == Counter("silent"))   # True
```

### defaultdict

```python
from collections import defaultdict

orders = [("An", "cafe"), ("Bình", "trà"), ("An", "bánh")]

by_user = defaultdict(list)          # thiếu key -> tạo list()
for user, item in orders:
    by_user[user].append(item)       # không cần if user not in ...
print(dict(by_user))                 # {'An': ['cafe', 'bánh'], 'Bình': ['trà']}

count = defaultdict(int)             # thiếu key -> 0
for ch in "hello":
    count[ch] += 1
print(count["l"])                    # 2

# Cây lồng nhiều tầng
tree = lambda: defaultdict(tree)
cfg = tree()
cfg["db"]["primary"]["host"] = "localhost"
```

### deque

```python
from collections import deque

q = deque()
q.append("A")          # vào cuối
q.append("B")
q.appendleft("VIP")    # chen lên đầu
print(q.popleft())     # VIP   O(1), list.pop(0) là O(n)
print(q)               # deque(['A', 'B'])

# Giữ N phần tử gần nhất (cửa sổ trượt)
recent = deque(maxlen=3)
for page in ["home", "cart", "pay", "done"]:
    recent.append(page)
print(list(recent))    # ['cart', 'pay', 'done']

d = deque([1, 2, 3, 4])
d.rotate(1)
print(d)               # deque([4, 1, 2, 3])
```

BFS (duyệt theo chiều rộng) là ứng dụng kinh điển của `deque`:

```python
def bfs(graph, start):
    seen, queue, order = {start}, deque([start]), []
    while queue:
        node = queue.popleft()
        order.append(node)
        for nxt in graph[node]:
            if nxt not in seen:
                seen.add(nxt)
                queue.append(nxt)
    return order

g = {"A": ["B", "C"], "B": ["D"], "C": ["D"], "D": []}
print(bfs(g, "A"))     # ['A', 'B', 'C', 'D']
```

### namedtuple

```python
from collections import namedtuple

Point = namedtuple("Point", ["x", "y"])
p = Point(3, 4)
print(p.x, p[1])          # 3 4
x, y = p                  # vẫn unpack như tuple
print(p._replace(x=0))    # Point(x=0, y=4)  (tạo mới, vì bất biến)
print(p._asdict())        # {'x': 3, 'y': 4}

# Cách hiện đại có type hint
from typing import NamedTuple

class User(NamedTuple):
    name: str
    age: int = 0

print(User("An"))         # User(name='An', age=0)
```

### OrderedDict: LRU cache đơn giản

```python
from collections import OrderedDict

class LRU:
    def __init__(self, capacity):
        self.cap, self.data = capacity, OrderedDict()

    def get(self, key):
        if key not in self.data:
            return None
        self.data.move_to_end(key)          # vừa dùng -> đưa về cuối
        return self.data[key]

    def put(self, key, value):
        self.data[key] = value
        self.data.move_to_end(key)
        if len(self.data) > self.cap:
            self.data.popitem(last=False)   # bỏ phần tử lâu nhất

cache = LRU(2)
cache.put("a", 1); cache.put("b", 2); cache.get("a"); cache.put("c", 3)
print(list(cache.data))   # ['a', 'c']  ('b' bị loại)
```

### ChainMap: cấu hình nhiều tầng

```python
from collections import ChainMap
import os

defaults = {"port": 8000, "debug": False}
file_cfg = {"port": 9000}
env_cfg = {k.lower(): v for k, v in os.environ.items() if k in ("PORT", "DEBUG")}

cfg = ChainMap(env_cfg, file_cfg, defaults)   # tra từ trái sang phải
print(cfg["port"], cfg["debug"])               # 9000 False (nếu không có env)
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `arr.reduce((m, x) => (m[x] = (m[x] ?? 0) + 1, m), {})` | `Counter(arr)` |
| `Object.groupBy(arr, fn)` / `map[k] ??= []` | `defaultdict(list)` |
| `arr.shift()` (O(n)) | `deque.popleft()` (O(1)) |
| object `{x, y}` | `namedtuple` / `NamedTuple` |
| `Map` (giữ thứ tự chèn) | `dict` / `OrderedDict` |
| `{...defaults, ...file, ...env}` | `ChainMap(env, file, defaults)` (không copy) |

## Xem thêm

- [Dict](./22-Dict.md)
- [List](./20-List.md)
- [Tuple](./21-Tuple.md)
- [Set](./23-Set.md)
- [itertools & functools](./43-Itertools-Functools.md)
- [Map (JS)](../js/32-Map.md)
