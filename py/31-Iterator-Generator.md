[← Mục lục](./index.md)

# Iterator & Generator

## Định nghĩa

- **Iterable**: object có thể duyệt bằng `for`, tức là có method `__iter__()` trả về một iterator. Ví dụ: list, str, dict, set, file.
- **Iterator**: object có `__next__()` trả về phần tử kế tiếp, và raise `StopIteration` khi hết. Iterator cũng là iterable (`__iter__` trả về chính nó). Iterator chỉ duyệt được **một lần**.
- **Generator function**: hàm có từ khóa `yield`. Gọi hàm không chạy thân hàm ngay mà trả về một **generator object** (một loại iterator). Mỗi lần `next()`, hàm chạy tới `yield` kế tiếp rồi **tạm dừng**, giữ nguyên trạng thái.
- **`yield from`**: ủy quyền cho một iterable/generator khác.

## Giải thích đơn giản

Cùng ý tưởng với **iteration protocol** (`Symbol.iterator`, `next()`) và **generator** (`function*`, `yield`) của JS. Python chỉ khác tên: `__iter__`/`__next__` thay cho `[Symbol.iterator]`/`next`, và kết thúc bằng exception `StopIteration` thay cho `{ done: true }`.

Ví von: list là **cả nồi cơm đã nấu sẵn** (tốn chỗ, có ngay mọi phần), generator là **máy làm bánh mì**: bấm một lần ra một ổ, không cần làm sẵn một nghìn ổ để trong kho. Rất hợp khi xử lý file lớn hoặc dữ liệu vô hạn.

`for` thực chất làm việc sau:

```python
it = iter(iterable)
while True:
    try:
        x = next(it)
    except StopIteration:
        break
    # thân vòng lặp
```

## Ví dụ

### Iterator thủ công

```python
nums = [1, 2, 3]
it = iter(nums)
print(next(it))   # 1
print(next(it))   # 2
print(next(it))   # 3
print(next(it))   # StopIteration
```

### Generator function

```python
def countdown(n):
    print("Bắt đầu")
    while n > 0:
        yield n          # tạm dừng, trả n ra ngoài
        n -= 1
    print("Hết")

gen = countdown(3)       # chưa in gì cả
print(next(gen))         # Bắt đầu / 3
print(list(gen))         # [2, 1] / Hết
print(list(gen))         # [] -> generator chỉ duyệt được một lần
```

### Dữ liệu vô hạn, lấy dần

```python
from itertools import islice

def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

print(list(islice(fibonacci(), 10)))
# [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

### Đọc file lớn theo từng dòng (pipeline)

```python
def read_lines(path):
    with open(path, encoding="utf-8") as f:
        for line in f:          # file object cũng là iterator, đọc từng dòng
            yield line.rstrip("\n")

def only_errors(lines):
    for line in lines:
        if "ERROR" in line:
            yield line

# Không bao giờ nạp cả file vào RAM
for line in only_errors(read_lines("app.log")):
    print(line)
```

### yield from

```python
def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)     # ủy quyền đệ quy
        else:
            yield item

print(list(flatten([1, [2, [3, 4]], 5])))   # [1, 2, 3, 4, 5]
```

### Class iterable tự định nghĩa

```python
class Range:
    def __init__(self, start, stop):
        self.start, self.stop = start, stop

    def __iter__(self):          # dùng generator cho gọn
        current = self.start
        while current < self.stop:
            yield current
            current += 1

for x in Range(1, 4):
    print(x)   # 1 2 3
```

### itertools hữu ích

```python
from itertools import chain, count, groupby

print(list(chain([1, 2], [3])))     # [1, 2, 3]
for i in count(10):                 # 10, 11, 12, ... vô hạn
    if i > 12:
        break
data = ["a1", "a2", "b1"]
for key, group in groupby(data, key=lambda s: s[0]):
    print(key, list(group))         # a ['a1', 'a2'] / b ['b1']
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `obj[Symbol.iterator]()` | `obj.__iter__()` / `iter(obj)` |
| `it.next()` → `{ value, done }` | `next(it)` → giá trị, hết thì `StopIteration` |
| `function* gen() { yield 1 }` | `def gen(): yield 1` |
| `yield* other()` | `yield from other()` |
| `for (const x of it)` | `for x in it:` |
| `Array.from(gen())` | `list(gen())` |
| `async function*` | `async def` + `yield` (async generator) |

## Xem thêm

- [ES6: Tổng quan (JS)](../js/18-ES6-Overview.md)
- [Comprehension](./22-Comprehension.md)
- [Control Flow & match](./25-Control-Flow-Match.md)
- [Coroutine & async/await](./41-async-await.md)
