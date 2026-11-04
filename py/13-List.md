[← Mục lục](./index.md)

# List

## Định nghĩa

`list` là kiểu **sequence mutable** (có thể thay đổi), có thứ tự, chứa phần tử thuộc bất kỳ kiểu nào. Bên trong CPython, list là **dynamic array** chứa các con trỏ tới object, nên truy cập theo index là `O(1)`, thêm vào cuối là `O(1)` khấu hao, chèn/xóa ở đầu là `O(n)`.

## Giải thích đơn giản

`list` chính là `Array` của JS. Khác biệt chính:

- Tên method khác (`append` thay `push`, `pop(0)` thay `shift`...).
- Có **slicing** `a[1:3]` và index âm `a[-1]`.
- Nhiều method thay đổi list tại chỗ (`sort`, `reverse`, `append`) và **trả về `None`**, không trả về list. Đây là lỗi kinh điển: `a = a.sort()` làm `a` thành `None`.
- Python list **không có "lỗ hổng"** (sparse) như JS: không thể gán `a[100] = 1` khi list chỉ có 3 phần tử, sẽ lỗi `IndexError`.

## Ví dụ

### Thêm / xóa

```python
fruits = ["apple", "banana"]

fruits.append("orange")        # thêm cuối          (JS: push)
fruits.insert(0, "kiwi")       # chèn vào vị trí 0  (JS: unshift)
fruits.extend(["mango", "fig"])# nối list khác      (JS: push(...arr))
print(fruits)
# ['kiwi', 'apple', 'banana', 'orange', 'mango', 'fig']

last = fruits.pop()            # xóa & trả về phần tử cuối (JS: pop)
first = fruits.pop(0)          # xóa & trả về phần tử đầu  (JS: shift)
fruits.remove("banana")        # xóa theo giá trị (phần tử đầu tiên khớp)
del fruits[0]                  # xóa theo index
fruits.clear()                 # xóa hết
```

### Truy cập & slicing

```python
nums = [10, 20, 30, 40, 50]
print(nums[0], nums[-1])   # 10 50
print(nums[1:4])           # [20, 30, 40]
print(nums[::-1])          # [50, 40, 30, 20, 10]

nums[1:3] = [21, 31]       # gán lại một đoạn
print(nums)                # [10, 21, 31, 40, 50]

copy = nums[:]             # shallow copy (JS: [...nums])
```

### Tìm kiếm

```python
nums = [3, 1, 4, 1, 5]
print(4 in nums)          # True   (JS: includes)
print(nums.index(1))      # 1      (JS: indexOf, nhưng không thấy thì ValueError)
print(nums.count(1))      # 2
print(len(nums))          # 5
print(min(nums), max(nums), sum(nums))  # 1 5 14
```

### Sắp xếp

```python
nums = [3, 1, 4, 1, 5]

nums.sort()                     # sắp tại chỗ, trả None
print(nums)                     # [1, 1, 3, 4, 5]

nums.sort(reverse=True)
new_list = sorted(nums)         # trả list mới, không đổi nums

words = ["banana", "Apple", "cherry"]
print(sorted(words, key=str.lower))  # ['Apple', 'banana', 'cherry']
print(sorted(words, key=len))        # ['Apple', 'banana', 'cherry']
```

Python `sort()` mặc định so sánh **đúng kiểu** (số theo giá trị số), không như JS `[10, 9, 1].sort()` ra `[1, 10, 9]` vì JS so sánh theo chuỗi.

### Biến đổi (map / filter)

```python
nums = [1, 2, 3, 4]
print([n * 2 for n in nums])            # [2, 4, 6, 8]
print([n for n in nums if n % 2 == 0])  # [2, 4]
print(any(n > 3 for n in nums))         # True  (JS: some)
print(all(n > 0 for n in nums))         # True  (JS: every)
```

### Duyệt kèm index

```python
for i, fruit in enumerate(["a", "b", "c"]):
    print(i, fruit)

for x, y in zip([1, 2, 3], ["a", "b", "c"]):
    print(x, y)   # 1 a / 2 b / 3 c
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `arr.push(x)` | `lst.append(x)` |
| `arr.pop()` | `lst.pop()` |
| `arr.shift()` | `lst.pop(0)` (chậm, dùng `collections.deque` nếu cần) |
| `arr.unshift(x)` | `lst.insert(0, x)` |
| `arr.concat(b)` | `lst + b` |
| `arr.slice(1, 3)` | `lst[1:3]` |
| `arr.splice(1, 2)` | `del lst[1:3]` |
| `arr.includes(x)` | `x in lst` |
| `arr.length` | `len(lst)` |
| `arr.forEach((x, i) => ...)` | `for i, x in enumerate(lst):` |
| `arr.find(fn)` | `next((x for x in lst if fn(x)), None)` |
| `arr.flat()` | `[y for x in lst for y in x]` |

## Xem thêm

- [Array (JS)](../js/13-Array.md)
- [Sparse Array (JS)](../js/14-Sparse-Array.md)
- [Tuple](./14-Tuple.md)
- [Comprehension](./22-Comprehension.md)
- [Mutable & Immutable](./35-Mutable-Immutable.md)
