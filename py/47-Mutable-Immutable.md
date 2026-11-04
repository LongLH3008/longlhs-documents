[← Mục lục](./index.md)

# Mutable & Immutable

## Định nghĩa

Trong Python, **mọi giá trị đều là object** nằm trên heap, và **mọi biến đều là tên (name) tham chiếu tới object**. Mỗi object có ba thuộc tính: **identity** (`id()`), **type**, **value**.

- **Immutable**: giá trị của object **không thể thay đổi** sau khi tạo. Mọi "thay đổi" thực chất tạo object mới. Gồm: `int`, `float`, `bool`, `str`, `tuple`, `frozenset`, `bytes`, `None`.
- **Mutable**: giá trị có thể thay đổi tại chỗ (in-place), identity giữ nguyên. Gồm: `list`, `dict`, `set`, `bytearray`, và hầu hết object do người dùng định nghĩa.

Cơ chế truyền tham số của Python là **pass by object reference** (còn gọi là *call by sharing*): hàm nhận **cùng object** mà người gọi truyền vào, không phải bản sao.

## Giải thích đơn giản

Trong JS, primitive (number, string) được "copy giá trị", còn object/array được "copy tham chiếu". Python thống nhất hơn: **luôn luôn là tham chiếu**. Sự khác biệt bạn thấy chỉ đến từ việc object đó **có sửa được hay không**.

Ví von: biến là **nhãn dán**, object là **chiếc hộp**.

- `a = [1, 2]` là dán nhãn `a` lên hộp chứa `[1, 2]`.
- `b = a` là dán thêm nhãn `b` lên **cùng chiếc hộp** đó. Sửa đồ trong hộp qua `b` thì nhìn qua `a` cũng thấy.
- Với số (immutable), `x += 1` không sửa hộp cũ mà **gỡ nhãn ra dán sang hộp mới**.

## Ví dụ

### Immutable: "sửa" là tạo object mới

```python
x = 10
print(id(x))
x += 1
print(id(x))    # id khác -> x giờ trỏ tới object mới

s = "hi"
t = s
s += "!"
print(s, t)     # hi! hi  -> t vẫn trỏ tới chuỗi cũ
```

### Mutable: hai tên, một object

```python
a = [1, 2]
b = a           # không copy
b.append(3)
print(a)        # [1, 2, 3]
print(a is b)   # True
```

### Truyền vào hàm

```python
def add_item(lst):
    lst.append("mới")      # sửa object gốc -> người gọi thấy thay đổi

def reassign(lst):
    lst = ["khác"]         # chỉ gắn tên cục bộ sang object khác -> người gọi không thấy

items = ["cũ"]
add_item(items)
print(items)   # ['cũ', 'mới']
reassign(items)
print(items)   # ['cũ', 'mới']
```

### Shallow copy vs deep copy

```python
import copy

original = {"name": "An", "skills": ["py", "js"]}

shallow = original.copy()          # hoặc dict(original), {**original}
shallow["skills"].append("go")
print(original["skills"])          # ['py', 'js', 'go']  -> list bên trong vẫn dùng chung!

deep = copy.deepcopy(original)
deep["skills"].append("rust")
print(original["skills"])          # ['py', 'js', 'go']  -> không bị ảnh hưởng
```

### Bẫy: nhân list lồng nhau

```python
grid = [[0] * 3] * 3       # 3 tham chiếu tới CÙNG một list con
grid[0][0] = 1
print(grid)                # [[1, 0, 0], [1, 0, 0], [1, 0, 0]]

grid = [[0] * 3 for _ in range(3)]   # đúng: tạo 3 list riêng
grid[0][0] = 1
print(grid)                # [[1, 0, 0], [0, 0, 0], [0, 0, 0]]
```

### Hashable: chỉ immutable mới làm key dict

```python
d = {(1, 2): "ok"}      # tuple làm key được
d = {[1, 2]: "x"}       # TypeError: unhashable type: 'list'
```

### `+=` khác nhau giữa list và tuple

```python
a = [1]
b = a
a += [2]        # list: sửa tại chỗ (extend)
print(b)        # [1, 2]

t = (1,)
u = t
t += (2,)       # tuple: tạo tuple mới
print(u)        # (1,)
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| primitive copy theo giá trị | mọi thứ là tham chiếu; immutable nên "trông như" copy |
| object/array copy tham chiếu | list/dict/set copy tham chiếu |
| `{ ...obj }`, `[...arr]` (shallow) | `d.copy()`, `lst[:]`, `copy.copy()` |
| `structuredClone(obj)` (deep) | `copy.deepcopy(obj)` |
| `Object.freeze()` | `tuple`, `frozenset`, `@dataclass(frozen=True)` |
| `===` so sánh tham chiếu với object | `is` |

## Xem thêm

- [Primitive & Reference Types (JS)](../js/35-Primitive-Reference.md)
- [Default Parameter](./31-Default-Parameter.md)
- [Tuple](./21-Tuple.md)
- [Name Binding (không có Hoisting)](./49-Name-Binding.md)
