[← Mục lục](./index.md)

# Vòng lặp (Loop): for & while

## Định nghĩa

Vòng lặp là cấu trúc điều khiển (control flow) cho phép **thực thi lặp lại** một khối lệnh. Python có đúng hai loại:

| Câu lệnh | Cơ chế | Dùng khi |
|---|---|---|
| `for x in iterable:` | lấy lần lượt từng phần tử của một **iterable** (list, str, dict, range, file, generator...) cho tới khi hết | biết trước tập dữ liệu cần duyệt |
| `while cond:` | kiểm tra `cond` trước mỗi lượt, lặp khi còn **truthy** | không biết trước số lượt, lặp theo điều kiện |

Các từ khóa đi kèm:

| Từ khóa | Ý nghĩa |
|---|---|
| `break` | thoát ngay khỏi vòng lặp **gần nhất** |
| `continue` | bỏ phần còn lại của lượt hiện tại, sang lượt kế |
| `else` (sau `for` / `while`) | chạy khi vòng lặp **kết thúc bình thường**, tức là **không** gặp `break` |
| `pass` | câu lệnh rỗng, giữ chỗ cho thân vòng lặp |

Về bản chất, `for` trong Python luôn là **for-each**: nó gọi `iter(obj)` để lấy iterator, rồi gọi `next()` liên tục cho tới khi gặp `StopIteration` (xem [Iterator & Generator](./42-Iterator-Generator.md)).

## Giải thích đơn giản

- `for` giống **người phát kẹo đi dọc hàng ghế**: mỗi ghế (phần tử) nhận một lần, hết ghế thì dừng. Không cần biết có bao nhiêu ghế, không cần tự đếm chỉ số.
- `while` giống **"còn đói thì còn ăn"**: trước mỗi miếng lại tự hỏi "còn đói không?". Nếu quên làm cho điều kiện sai đi thì ăn mãi không dừng (vòng lặp vô hạn).
- `break` là **đứng dậy bỏ về giữa chừng**, `continue` là **bỏ qua món này, gọi món tiếp**.
- `for ... else` hơi lạ: hiểu `else` là **"không bị break"**. Ví dụ đi tìm chìa khóa trong từng phòng: tìm thấy thì `break`; đi hết mọi phòng mà không thấy thì khối `else` chạy ("báo mất chìa").

Khác JS: Python **không có** `for (let i = 0; i < n; i++)` và **không có** `do...while`. Muốn đếm số thì dùng `range()`.

## Ví dụ

### for cơ bản & range

```python
for fruit in ["táo", "cam", "xoài"]:
    print(fruit)

for ch in "abc":           # string cũng là iterable
    print(ch)              # a b c

for i in range(3):         # 0 1 2          (stop không bao gồm)
    print(i)

for i in range(2, 10, 3):  # 2 5 8          (start, stop, step)
    print(i)

for i in range(5, 0, -1):  # 5 4 3 2 1      (đếm ngược)
    print(i)

print(list(range(0, 10, 2)))   # [0, 2, 4, 6, 8]
```

`range` là object **lazy**: `range(10**9)` không tạo list 1 tỷ phần tử, chỉ sinh số khi cần.

### Lấy cả index: enumerate

```python
names = ["An", "Bình", "Chi"]

# Không nên (kiểu C/JS)
for i in range(len(names)):
    print(i, names[i])

# Nên (Pythonic)
for i, name in enumerate(names, start=1):
    print(i, name)          # 1 An / 2 Bình / 3 Chi
```

### Duyệt song song: zip

```python
names = ["An", "Bình", "Chi"]
scores = [8, 9, 7]

for name, score in zip(names, scores):
    print(f"{name}: {score}")

# zip dừng ở list ngắn nhất; muốn báo lỗi nếu lệch độ dài (3.10+):
# zip(names, scores, strict=True)
```

### Duyệt dict

```python
prices = {"cafe": 25_000, "trà": 15_000}

for key in prices:                  # mặc định duyệt key
    print(key)

for key, value in prices.items():   # key và value
    print(f"{key}: {value:,}đ")
```

### Duyệt ngược & đã sắp xếp

```python
nums = [3, 1, 2]
for n in reversed(nums):            # 2 1 3   (không đổi list gốc)
    print(n)
for n in sorted(nums, reverse=True):# 3 2 1   (tạo list mới)
    print(n)
```

### while

```python
# Đếm ngược
n = 3
while n > 0:
    print(n)
    n -= 1          # quên dòng này -> vòng lặp vô hạn
print("Bắn!")

# Lặp tới khi người dùng nhập đúng
password = ""
while password != "secret":
    password = input("Mật khẩu: ")
```

### Giả lập do...while

```python
# JS: do { x = read() } while (x < 0)
while True:
    x = int(input("Nhập số dương: "))
    if x >= 0:
        break
```

### break & continue

```python
for n in range(1, 10):
    if n % 2 == 0:
        continue        # bỏ qua số chẵn
    if n > 7:
        break           # dừng hẳn khi vượt 7
    print(n)            # 1 3 5 7
```

### for ... else: tìm kiếm

```python
def find_user(users, target):
    for user in users:
        if user["name"] == target:
            print("Tìm thấy:", user)
            break
    else:                           # chỉ chạy khi KHÔNG break
        print("Không tìm thấy", target)

users = [{"name": "An"}, {"name": "Bình"}]
find_user(users, "Bình")   # Tìm thấy: {'name': 'Bình'}
find_user(users, "Chi")    # Không tìm thấy Chi
```

### Vòng lặp lồng nhau & thoát nhiều tầng

`break` chỉ thoát vòng **trong cùng**. Python không có `break label` như JS. Cách sạch nhất là tách thành hàm rồi `return`:

```python
def find_pair(nums, target):
    for i, a in enumerate(nums):
        for b in nums[i + 1:]:
            if a + b == target:
                return a, b        # thoát cả hai vòng
    return None

print(find_pair([2, 7, 11, 15], 9))   # (2, 7)
```

Hoặc dùng `itertools.product` để làm phẳng hai vòng thành một:

```python
from itertools import product

for x, y in product(range(3), range(3)):
    if x + y == 3:
        print(x, y)     # 1 2
        break
```

### Bẫy 1: sửa list trong lúc duyệt

```python
nums = [1, 2, 2, 3]
for n in nums:
    if n == 2:
        nums.remove(n)      # SAI: bỏ sót phần tử vì index bị dịch
print(nums)                 # [1, 2, 3]  (vẫn còn một số 2!)

# Đúng: tạo list mới
nums = [1, 2, 2, 3]
nums = [n for n in nums if n != 2]
print(nums)                 # [1, 3]

# Hoặc duyệt trên bản sao
for n in nums[:]:
    ...
```

Với dict, thêm/xóa key trong lúc duyệt sẽ ném `RuntimeError: dictionary changed size during iteration`. Duyệt trên `list(d)` nếu cần xóa.

### Bẫy 2: biến vòng lặp "rò" ra ngoài

```python
for i in range(3):
    pass
print(i)        # 2  -> biến i vẫn tồn tại sau vòng lặp (Python không có block scope)
```

Xem thêm [Scope (LEGB)](./48-Scope-LEGB.md).

### Khi nào không cần viết vòng lặp

```python
nums = [1, 2, 3, 4]

total = sum(nums)                          # thay vì cộng dồn
squares = [n * n for n in nums]            # comprehension
has_big = any(n > 3 for n in nums)         # True
all_pos = all(n > 0 for n in nums)         # True
biggest = max(nums)
```

Các hàm built-in và comprehension vừa ngắn hơn vừa thường nhanh hơn vòng lặp viết tay, vì phần lặp chạy bằng C.

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `for (let i = 0; i < n; i++)` | `for i in range(n):` |
| `for (let i = n - 1; i >= 0; i--)` | `for i in range(n - 1, -1, -1):` hoặc `reversed(range(n))` |
| `for (const x of arr)` | `for x in arr:` |
| `for (const k in obj)` | `for k in d:` |
| `for (const [k, v] of Object.entries(obj))` | `for k, v in d.items():` |
| `arr.forEach((x, i) => ...)` | `for i, x in enumerate(arr):` |
| `while (c) {}` | `while c:` |
| `do { } while (c)` | `while True: ... if not c: break` |
| `outer: for ... break outer` | tách hàm + `return` |
| không có | `for ... else` |

## Xem thêm

- [If-Else & Ternary](./25-If-Else.md)
- [Pattern Matching (match / case)](./27-Match-Case.md)
- [Comprehension](./35-Comprehension.md)
- [Iterator & Generator](./42-Iterator-Generator.md)
- [itertools & functools](./43-Itertools-Functools.md)
- [dict: keys, values, items](./09-Keys-Values-Items.md)
- [Đệ quy (Recursion)](./34-Recursion.md)
