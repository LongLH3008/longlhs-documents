[← Mục lục](./index.md)

# Dict

## Định nghĩa

`dict` là kiểu **mapping mutable** lưu các cặp `key: value`, cài đặt bằng **hash table**: tra cứu, thêm, xóa trung bình `O(1)`. Key phải **hashable** (str, int, tuple chứa phần tử bất biến...). Từ Python 3.7, dict **giữ thứ tự chèn**.

## Giải thích đơn giản

`dict` là sự kết hợp của **object literal** và **Map** trong JS:

- Cú pháp giống object: `{"name": "An"}`, nhưng key **phải có nháy** nếu là chuỗi.
- Hành vi giống `Map`: key có thể là số, tuple..., không bị ép thành chuỗi.
- Truy cập bằng `d["key"]`, **không** dùng được `d.key`. Truy cập key không tồn tại là lỗi `KeyError`, không ra `undefined`.

Ví von: dict là **danh bạ điện thoại**: tra tên (key) ra số (value) ngay lập tức, không cần lật từng trang.

## Ví dụ

### Tạo & truy cập

```python
user = {"name": "An", "age": 25}
user2 = dict(name="Bình", age=30)   # cách khác

print(user["name"])          # An
print(user["email"])         # KeyError: 'email'
print(user.get("email"))     # None, an toàn
print(user.get("email", "chưa có"))  # giá trị mặc định

print("age" in user)         # True  (JS: "age" in obj / obj.hasOwnProperty)
print(len(user))             # 2
```

### Thêm / sửa / xóa

```python
user["email"] = "an@mail.com"   # thêm hoặc sửa
user.update({"age": 26, "city": "HN"})

del user["city"]                # xóa, key không có thì KeyError
age = user.pop("age")           # xóa & trả về value
user.pop("missing", None)       # không lỗi nếu thiếu key
user.setdefault("role", "user") # chỉ gán nếu chưa có key
```

### Merge dict

```python
a = {"x": 1, "y": 2}
b = {"y": 20, "z": 3}

merged = {**a, **b}      # JS: { ...a, ...b }
merged = a | b           # Python 3.9+
print(merged)            # {'x': 1, 'y': 20, 'z': 3}

a |= b                   # merge tại chỗ
```

### Duyệt

```python
prices = {"apple": 10, "banana": 5}
for name, price in prices.items():
    print(name, price)
```

### Dict lồng nhau & truy cập an toàn

```python
data = {"user": {"profile": {"city": "HN"}}}

# JS: data?.user?.profile?.city
city = data.get("user", {}).get("profile", {}).get("city")
print(city)   # HN
```

### collections: defaultdict & Counter

```python
from collections import defaultdict, Counter

# Nhóm theo chữ cái đầu
groups = defaultdict(list)
for w in ["apple", "avocado", "banana"]:
    groups[w[0]].append(w)     # không cần kiểm tra key tồn tại
print(dict(groups))            # {'a': ['apple', 'avocado'], 'b': ['banana']}

# Đếm tần suất
print(Counter("banana"))             # Counter({'a': 3, 'n': 2, 'b': 1})
print(Counter("banana").most_common(1))  # [('a', 3)]
```

### Dict comprehension

```python
squares = {n: n * n for n in range(4)}
print(squares)   # {0: 0, 1: 1, 2: 4, 3: 9}
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `obj.name` / `obj["name"]` | `d["name"]` |
| `obj.missing` → `undefined` | `d["missing"]` → `KeyError`; `d.get("missing")` → `None` |
| `"k" in obj` | `"k" in d` |
| `delete obj.k` | `del d["k"]` / `d.pop("k")` |
| `{ ...a, ...b }` | `{**a, **b}` / `a \| b` |
| `Object.keys(obj).length` | `len(d)` |
| `new Map([[1, "a"]])` | `{1: "a"}` |
| `structuredClone(obj)` | `copy.deepcopy(d)` |

## Xem thêm

- [Object (JS)](../js/15-Object.md)
- [Map (JS)](../js/32-Map.md)
- [dict: keys, values, items](./08-Keys-Values-Items.md)
- [Comprehension](./22-Comprehension.md)
- [JSON](./02-JSON.md)
