[← Mục lục](./index.md)

# First-class & Higher-order Function

## Định nghĩa

- **First-class function**: function là một object như mọi giá trị khác, có thể gán vào biến, truyền làm tham số, trả về từ hàm khác, lưu vào list/dict.
- **Higher-order function** (HOF): hàm **nhận** một hàm làm tham số hoặc **trả về** một hàm.
- Các HOF built-in phổ biến: `map`, `filter`, `sorted(key=...)`, `min/max(key=...)`, `functools.reduce`.

## Giải thích đơn giản

Giống JS hoàn toàn về ý tưởng: hàm là "đồ vật" có thể cầm đi đưa người khác. HOF là "người quản lý" nhận nhân viên (hàm) rồi giao việc cho họ.

Khác biệt về cách viết: JS gọi `arr.map(fn)` (method của Array), còn Python gọi `map(fn, arr)` (hàm global) và trả về **iterator lười** (lazy), cần `list()` để xem kết quả. Trên thực tế, người viết Python hay dùng **comprehension** thay cho `map`/`filter`.

## Ví dụ

### Function là object

```python
def shout(text):
    return text.upper()

yell = shout                # gán vào biến, không có ()
print(yell("hi"))           # HI
print(shout.__name__)       # shout

actions = {"up": str.upper, "low": str.lower}
print(actions["up"]("abc")) # ABC
```

### Truyền hàm làm tham số

```python
def apply_twice(fn, x):
    return fn(fn(x))

print(apply_twice(lambda n: n * 2, 3))   # 12
```

### Trả về hàm

```python
def multiplier(factor):
    def mul(x):
        return x * factor
    return mul

double = multiplier(2)
print(double(5))   # 10
```

### map, filter, reduce, sorted

```python
from functools import reduce

nums = [3, 1, 4, 1, 5, 9]

print(list(map(lambda x: x * 2, nums)))         # [6, 2, 8, 2, 10, 18]
print(list(filter(lambda x: x % 2 == 1, nums))) # [3, 1, 1, 5, 9]
print(reduce(lambda acc, x: acc + x, nums, 0))  # 23

# Cách "Pythonic" hơn
print([x * 2 for x in nums])
print([x for x in nums if x % 2 == 1])
print(sum(nums))

users = [{"name": "An", "age": 30}, {"name": "Bình", "age": 22}]
print(sorted(users, key=lambda u: u["age"]))       # sắp theo tuổi
print(max(users, key=lambda u: u["age"])["name"])  # An
```

### functools.partial (gán trước tham số)

```python
from functools import partial

def power(base, exp):
    return base ** exp

square = partial(power, exp=2)
print(square(7))   # 49
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `arr.map(fn)` | `map(fn, arr)` hoặc `[fn(x) for x in arr]` |
| `arr.filter(fn)` | `filter(fn, arr)` hoặc `[x for x in arr if fn(x)]` |
| `arr.reduce(fn, init)` | `functools.reduce(fn, arr, init)` |
| `arr.sort((a, b) => a.age - b.age)` | `sorted(arr, key=lambda u: u["age"])` |
| `fn.bind(null, a)` | `functools.partial(fn, a)` |
| `arr.some(fn)` / `arr.every(fn)` | `any(...)` / `all(...)` |

## Xem thêm

- [First-class & Higher-order Function (JS)](../js/09-First-class-Higher-order-Function.md)
- [Lambda](./29-Lambda.md)
- [Comprehension](./35-Comprehension.md)
- [Decorator](./44-Decorator.md)
