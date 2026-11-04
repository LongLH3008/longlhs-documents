[← Mục lục](./index.md)

# Comprehension

## Định nghĩa

**Comprehension** là cú pháp ngắn gọn để tạo collection mới từ một iterable, kết hợp **biến đổi** (map) và **lọc** (filter) trong một biểu thức.

| Loại | Cú pháp | Kết quả |
|---|---|---|
| List comprehension | `[expr for x in it if cond]` | `list` |
| Dict comprehension | `{k: v for x in it if cond}` | `dict` |
| Set comprehension | `{expr for x in it if cond}` | `set` |
| Generator expression | `(expr for x in it if cond)` | generator (lazy) |

Comprehension có **scope riêng**: biến vòng lặp `x` không rò ra ngoài.

## Giải thích đơn giản

Trong JS bạn viết `arr.filter(x => x > 0).map(x => x * 2)`. Python gom hai bước vào một câu đọc như tiếng Anh:

```python
[x * 2 for x in arr if x > 0]
# "lấy x * 2, với mỗi x trong arr, nếu x > 0"
```

Ví von: comprehension giống **dây chuyền nhà máy**: nguyên liệu (`for x in it`) đi qua máy kiểm tra chất lượng (`if`), rồi qua máy gia công (`expr`), ra thành phẩm.

Generator expression `( ... )` thì **lười**: không tạo cả danh sách trong bộ nhớ, chỉ sinh từng phần tử khi cần.

## Ví dụ

### List comprehension

```python
nums = [1, -2, 3, -4, 5]

print([n * 2 for n in nums])           # [2, -4, 6, -8, 10]   (map)
print([n for n in nums if n > 0])      # [1, 3, 5]            (filter)
print([n * 2 for n in nums if n > 0])  # [2, 6, 10]           (filter + map)

# if-else ĐỨNG TRƯỚC for (đây là biểu thức điều kiện, không phải bộ lọc)
print(["dương" if n > 0 else "âm" for n in nums])
# ['dương', 'âm', 'dương', 'âm', 'dương']
```

### Dict & set comprehension

```python
words = ["apple", "banana", "kiwi"]

print({w: len(w) for w in words})      # {'apple': 5, 'banana': 6, 'kiwi': 4}
print({len(w) for w in words})         # {4, 5, 6}

prices = {"a": 10, "b": 50, "c": 5}
print({k: v for k, v in prices.items() if v >= 10})  # {'a': 10, 'b': 50}
```

### Lồng nhiều for (flatten, tích Descartes)

```python
matrix = [[1, 2], [3, 4], [5, 6]]

# Thứ tự for giống như viết vòng lặp lồng nhau từ ngoài vào trong
flat = [x for row in matrix for x in row]
print(flat)   # [1, 2, 3, 4, 5, 6]   (JS: matrix.flat())

pairs = [(c, s) for c in "AB" for s in [1, 2]]
print(pairs)  # [('A', 1), ('A', 2), ('B', 1), ('B', 2)]

# Chuyển vị ma trận
print([[row[i] for row in matrix] for i in range(2)])  # [[1, 3, 5], [2, 4, 6]]
```

### Generator expression

```python
# Tính tổng bình phương 10 triệu số mà không tạo list 10 triệu phần tử
total = sum(n * n for n in range(10_000_000))

print(any(n > 3 for n in [1, 5, 2]))   # True, dừng ngay khi gặp 5
```

### Khi nào KHÔNG dùng comprehension

```python
# Quá phức tạp, khó đọc -> viết vòng lặp thường
result = [f(x) for x in data if x if g(x) for y in x if y > 0]

# Chỉ để chạy side effect -> dùng for
[print(x) for x in data]   # tạo list None vô ích
for x in data:
    print(x)               # đúng
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `arr.map(x => x * 2)` | `[x * 2 for x in arr]` |
| `arr.filter(x => x > 0)` | `[x for x in arr if x > 0]` |
| `arr.filter(...).map(...)` | `[f(x) for x in arr if cond(x)]` |
| `arr.flatMap(r => r)` | `[x for r in arr for x in r]` |
| `Object.fromEntries(arr.map(...))` | `{k: v for ...}` |
| `new Set(arr.map(...))` | `{f(x) for x in arr}` |
| iterator helpers (lazy) | generator expression `( ... )` |

## Xem thêm

- [Array (JS)](../js/13-Array.md)
- [List](./20-List.md)
- [First-class & Higher-order Function](./10-First-class-Higher-order-Function.md)
- [Iterator & Generator](./42-Iterator-Generator.md)
