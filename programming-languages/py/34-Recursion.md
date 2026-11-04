[← Mục lục](./index.md)

# Đệ quy (Recursion)

## Định nghĩa

**Đệ quy (recursion)** là kỹ thuật trong đó một hàm **gọi lại chính nó** để giải bài toán bằng cách chia thành bài toán con nhỏ hơn cùng dạng. Một hàm đệ quy đúng luôn có:

1. **Base case** (điều kiện dừng): trường hợp đơn giản nhất, trả kết quả trực tiếp, không gọi lại.
2. **Recursive case**: gọi lại chính nó với đầu vào **tiến gần base case** hơn.

Mỗi lần gọi tạo một **stack frame** mới trên call stack. CPython giới hạn độ sâu đệ quy mặc định khoảng **1000** (`sys.getrecursionlimit()`), vượt quá sẽ ném `RecursionError`. CPython **không** tối ưu **tail call** (tail call optimization), nên đệ quy sâu luôn tốn stack.

## Giải thích đơn giản

Giống **búp bê Nga Matryoshka**: muốn biết có bao nhiêu con búp bê, bạn mở con ngoài cùng, rồi hỏi "con bên trong có bao nhiêu?" và lặp lại. Tới con nhỏ nhất không mở được nữa (base case) thì trả lời "1", rồi cộng dần ngược ra ngoài.

Quên base case giống **hai tấm gương đặt đối diện**: hình phản chiếu vô tận, tới khi hết chỗ (stack overflow).

## Ví dụ

### Giai thừa

```python
def factorial(n: int) -> int:
    if n <= 1:                     # base case
        return 1
    return n * factorial(n - 1)    # recursive case

print(factorial(5))   # 120 = 5 * 4 * 3 * 2 * 1
```

Call stack khi gọi `factorial(3)`:

```text
factorial(3) = 3 * factorial(2)
                   factorial(2) = 2 * factorial(1)
                                      factorial(1) = 1   <- base case
                   factorial(2) = 2 * 1 = 2
factorial(3) = 3 * 2 = 6
```

### Duyệt cấu trúc lồng nhau (ứng dụng thực tế)

Đệ quy rất hợp với dữ liệu dạng cây: thư mục, JSON lồng, comment trả lời comment.

```python
def flatten(items):
    result = []
    for item in items:
        if isinstance(item, list):
            result.extend(flatten(item))   # đi sâu vào list con
        else:
            result.append(item)
    return result

print(flatten([1, [2, [3, [4]], 5]]))   # [1, 2, 3, 4, 5]


def total_size(node: dict) -> int:
    """node = {"size": int, "children": [node, ...]}"""
    return node["size"] + sum(total_size(c) for c in node.get("children", []))

tree = {"size": 10, "children": [{"size": 5}, {"size": 3, "children": [{"size": 2}]}]}
print(total_size(tree))   # 20
```

### Fibonacci: đệ quy ngây thơ vs memoization

```python
def fib(n):
    if n < 2:
        return n
    return fib(n - 1) + fib(n - 2)   # tính lặp lại rất nhiều lần: O(2^n)

# fib(35) mất vài giây

from functools import lru_cache

@lru_cache(maxsize=None)            # nhớ kết quả đã tính: O(n)
def fib_fast(n):
    if n < 2:
        return n
    return fib_fast(n - 1) + fib_fast(n - 2)

print(fib_fast(100))   # 354224848179261915075 (tức thì)
```

### Giới hạn độ sâu

```python
import sys

def count_down(n):
    if n == 0:
        return 0
    return count_down(n - 1)

print(sys.getrecursionlimit())   # 1000
count_down(5000)                 # RecursionError: maximum recursion depth exceeded

# Có thể nâng giới hạn, nhưng dễ làm crash interpreter nếu quá lớn
sys.setrecursionlimit(10_000)
```

### Chuyển đệ quy thành vòng lặp

Khi độ sâu có thể lớn, viết lại bằng vòng lặp hoặc stack tường minh:

```python
def factorial_iter(n):
    result = 1
    for i in range(2, n + 1):
        result *= i
    return result

def flatten_iter(items):
    stack, result = [iter(items)], []
    while stack:
        for item in stack[-1]:
            if isinstance(item, list):
                stack.append(iter(item))
                break
            result.append(item)
        else:
            stack.pop()
    return result

print(flatten_iter([1, [2, [3, [4]], 5]]))   # [1, 2, 3, 4, 5]
```

## So sánh với JavaScript

| Khía cạnh | JavaScript | Python |
|---|---|---|
| Giới hạn độ sâu | phụ thuộc engine (~10.000 frame ở V8) | ~1000, chỉnh bằng `sys.setrecursionlimit` |
| Lỗi khi tràn | `RangeError: Maximum call stack size exceeded` | `RecursionError` |
| Tail call optimization | có trong spec ES6 (chỉ Safari làm) | không có |
| Memoization có sẵn | tự viết | `functools.lru_cache` / `functools.cache` |

## Xem thêm

- [Function](./28-Function.md)
- [Vòng lặp (Loop)](./26-Loop.md)
- [Decorator](./44-Decorator.md)
- [itertools & functools](./43-Itertools-Functools.md)
- [Memory Management & Garbage Collection](./53-Memory-Management-GC.md)
