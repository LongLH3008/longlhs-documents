[← Mục lục](./index.md)

# dict: keys, values, items

## Định nghĩa

`dict` có ba method trả về **view object** (dictionary view):

| Method | Trả về |
|---|---|
| `d.keys()` | view các key |
| `d.values()` | view các value |
| `d.items()` | view các cặp `(key, value)` dạng tuple |

View object là **động**: khi dict thay đổi, view phản ánh ngay, không phải bản sao. Nó iterable, hỗ trợ `len()`, `in`; riêng `keys()` và `items()` còn hỗ trợ phép toán tập hợp (`&`, `|`, `-`).

## Giải thích đơn giản

Giống `Object.keys / values / entries` của JS, nhưng:

- Gọi như method của dict (`d.keys()`) chứ không phải hàm static (`Object.keys(d)`).
- Kết quả không phải list mà là "cửa sổ nhìn vào dict". Dict đổi thì cửa sổ thấy ngay. Muốn có list cố định thì bọc `list(...)`.

## Ví dụ

```python
scores = {"an": 8, "binh": 6, "chi": 9}

print(scores.keys())    # dict_keys(['an', 'binh', 'chi'])
print(scores.values())  # dict_values([8, 6, 9])
print(scores.items())   # dict_items([('an', 8), ('binh', 6), ('chi', 9)])

# Duyệt (cách dùng phổ biến nhất)
for name, score in scores.items():
    print(f"{name}: {score}")

# Duyệt dict trực tiếp = duyệt keys
for name in scores:
    print(name)
```

### View là động

```python
keys = scores.keys()
scores["dung"] = 7
print(keys)   # dict_keys(['an', 'binh', 'chi', 'dung'])  -> tự cập nhật

frozen = list(scores.keys())   # bản sao cố định
```

### Biến đổi dict (giống Object.fromEntries(Object.entries(...).map(...)))

```python
# Cộng 1 điểm cho mọi người
bonus = {k: v + 1 for k, v in scores.items()}

# Lọc người đạt >= 8
good = {k: v for k, v in scores.items() if v >= 8}
print(good)   # {'an': 8, 'chi': 9}

# Đảo key <-> value
inverted = {v: k for k, v in scores.items()}

# Tạo dict từ list các cặp
pairs = [("x", 1), ("y", 2)]
print(dict(pairs))   # {'x': 1, 'y': 2}
```

### Phép toán tập hợp trên keys

```python
a = {"x": 1, "y": 2}
b = {"y": 3, "z": 4}
print(a.keys() & b.keys())   # {'y'}       key chung
print(a.keys() - b.keys())   # {'x'}       chỉ có ở a
```

### Thống kê nhanh

```python
print(sum(scores.values()))              # tổng điểm
print(max(scores, key=scores.get))       # người điểm cao nhất
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `Object.keys(o)` | `d.keys()` / `list(d)` |
| `Object.values(o)` | `d.values()` |
| `Object.entries(o)` | `d.items()` |
| `Object.fromEntries(arr)` | `dict(arr)` |
| `for (const [k, v] of Object.entries(o))` | `for k, v in d.items():` |
| trả về Array mới (bản sao) | trả về view động |

## Lưu ý

Không được thêm/xóa key trong khi đang duyệt view, sẽ lỗi `RuntimeError: dictionary changed size during iteration`. Hãy duyệt trên `list(d.items())`.

## Xem thêm

- [Keys, Values, Entries (JS)](../js/08-Keys-Values-Entries.md)
- [Dict](./22-Dict.md)
- [Comprehension](./35-Comprehension.md)
