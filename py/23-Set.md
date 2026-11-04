[← Mục lục](./index.md)

# Set

## Định nghĩa

`set` là collection **không có thứ tự**, **không trùng lặp**, **mutable**, cài đặt bằng hash table. Kiểm tra phần tử (`in`), thêm, xóa có độ phức tạp trung bình `O(1)`. Phần tử phải **hashable**.

`frozenset` là phiên bản **immutable** (và hashable) của set, có thể làm key của dict hoặc phần tử của set khác.

Set hỗ trợ đầy đủ các phép toán tập hợp: hợp `|`, giao `&`, hiệu `-`, hiệu đối xứng `^`, tập con `<=`, tập cha `>=`.

## Giải thích đơn giản

Giống `Set` của JS, nhưng Python có sẵn **toán tử tập hợp** như trong sách Toán, không cần viết `filter` thủ công.

Ví von: set là **danh sách khách mời của một bữa tiệc**: mỗi người chỉ ghi một lần, và câu hỏi "người này có trong danh sách không?" được trả lời ngay lập tức.

Lưu ý cú pháp: `{}` là **dict rỗng**, muốn set rỗng phải viết `set()`.

## Ví dụ

### Tạo & thao tác

```python
s = {1, 2, 3}
empty = set()            # KHÔNG phải {}
print(type({}))          # <class 'dict'>

s.add(4)
s.add(2)                 # trùng, không đổi
s.update([5, 6])         # thêm nhiều phần tử
s.remove(6)              # không có thì KeyError
s.discard(100)           # không có cũng không lỗi
print(3 in s, len(s))    # True 5
```

### Loại bỏ trùng lặp

```python
emails = ["a@x.com", "b@x.com", "a@x.com"]
print(set(emails))                      # {'a@x.com', 'b@x.com'} (thứ tự không đảm bảo)

# Giữ thứ tự xuất hiện ban đầu
print(list(dict.fromkeys(emails)))      # ['a@x.com', 'b@x.com']
```

### Phép toán tập hợp

```python
frontend = {"An", "Bình", "Chi"}
backend = {"Chi", "Dũng"}

print(frontend | backend)   # hợp:  {'An', 'Bình', 'Chi', 'Dũng'}
print(frontend & backend)   # giao: {'Chi'}             -> biết cả FE và BE
print(frontend - backend)   # hiệu: {'An', 'Bình'}      -> chỉ biết FE
print(frontend ^ backend)   # đối xứng: {'An', 'Bình', 'Dũng'}

print({"Chi"} <= frontend)          # True, tập con
print(frontend.isdisjoint({"Em"}))  # True, không có phần tử chung
```

### Tra cứu nhanh hơn list

```python
banned = {"spam", "scam", "hack"}   # set: O(1)

def is_clean(word):
    return word not in banned

# Với list, `in` phải duyệt từng phần tử: O(n)
```

### Set comprehension & frozenset

```python
print({c.lower() for c in "Hello"})   # {'h', 'e', 'l', 'o'}

key = frozenset({"read", "write"})
permissions = {key: "editor"}          # frozenset làm key được
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `new Set([1, 2])` | `{1, 2}` / `set([1, 2])` |
| `new Set()` | `set()` |
| `s.add(x)` | `s.add(x)` |
| `s.delete(x)` | `s.discard(x)` / `s.remove(x)` |
| `s.has(x)` | `x in s` |
| `s.size` | `len(s)` |
| `[...new Set(arr)]` | `list(set(arr))` / `list(dict.fromkeys(arr))` |
| `a.union(b)` (ES2025) | `a \| b` |
| `a.intersection(b)` (ES2025) | `a & b` |
| `a.difference(b)` (ES2025) | `a - b` |
| Set giữ thứ tự chèn | set **không** đảm bảo thứ tự |

## Xem thêm

- [Set (JS)](../js/33-Set.md)
- [Dict](./22-Dict.md)
- [Mutable & Immutable](./47-Mutable-Immutable.md)
