[← Mục lục](./index.md)

# Data Model, Dunder Methods & MRO

## Định nghĩa

- **Python Data Model** là bộ quy tắc mô tả cách object tương tác với cú pháp của ngôn ngữ (toán tử, `len()`, `for`, `in`, `[]`, `with`, gọi hàm...). Các hành vi này được định nghĩa qua **dunder methods** (special methods, "double underscore"), ví dụ `__init__`, `__len__`, `__add__`.
- **Attribute lookup**: khi truy cập `obj.x`, Python tìm trong `obj.__dict__` (dữ liệu của instance), rồi tới class, rồi tới các class cha theo **MRO**.
- **MRO** (Method Resolution Order) là thứ tự tìm kiếm thuộc tính trong cây kế thừa, tính bằng thuật toán **C3 linearization**, xem qua `Class.__mro__` hoặc `Class.mro()`. Mọi class đều kết thúc ở `object`.
- Class cũng là object, là instance của **metaclass** (mặc định là `type`).

## Giải thích đơn giản

Trong JS, kế thừa đi qua **prototype chain**: `obj → Class.prototype → Parent.prototype → Object.prototype → null`. Python có chuỗi tương đương là **MRO**: `obj.__dict__ → Class → Parent → object`.

Dunder methods là cách bạn "dạy" object của mình **nói chuyện bằng cú pháp Python**. Viết `__add__` thì object dùng được dấu `+`; viết `__len__` thì dùng được `len()`. JS chỉ có vài cái tương tự (`Symbol.iterator`, `toString`, `valueOf`, `Symbol.toPrimitive`), không cho overload toán tử.

Ví von: dunder methods là **ổ cắm tiêu chuẩn**. Thiết bị nào làm đúng chuẩn ổ cắm (`__len__`, `__iter__`...) thì cắm vào được mọi "tường điện" của Python (`len()`, `for`...).

## Ví dụ

### Dunder methods thường gặp

```python
class Vector:
    def __init__(self, x, y):           # khởi tạo
        self.x, self.y = x, y

    def __repr__(self):                 # repr(v), hiển thị trong REPL/debug
        return f"Vector({self.x}, {self.y})"

    def __str__(self):                  # str(v), print(v)
        return f"({self.x}, {self.y})"

    def __add__(self, other):           # v1 + v2
        return Vector(self.x + other.x, self.y + other.y)

    def __mul__(self, k):               # v * 3
        return Vector(self.x * k, self.y * k)

    def __eq__(self, other):            # v1 == v2
        return (self.x, self.y) == (other.x, other.y)

    def __hash__(self):                 # cần khi định nghĩa __eq__ mà muốn làm key dict
        return hash((self.x, self.y))

    def __abs__(self):                  # abs(v)
        return (self.x ** 2 + self.y ** 2) ** 0.5

    def __bool__(self):                 # if v:
        return bool(self.x or self.y)

v = Vector(3, 4)
print(v + Vector(1, 1))   # (4, 5)
print(v * 2)              # (6, 8)
print(abs(v))             # 5.0
print(v == Vector(3, 4))  # True
print([v])                # [Vector(3, 4)]  -> list dùng __repr__
```

### Container protocol

```python
class Playlist:
    def __init__(self, songs):
        self._songs = list(songs)

    def __len__(self):              # len(p)
        return len(self._songs)

    def __getitem__(self, i):       # p[0], p[1:3], và tự động hỗ trợ for
        return self._songs[i]

    def __contains__(self, song):   # "x" in p
        return song in self._songs

    def __call__(self):             # p() -> object gọi được như hàm
        return f"Đang phát {self._songs[0]}"

p = Playlist(["A", "B", "C"])
print(len(p), p[1], "B" in p)   # 3 B True
for s in p:
    print(s)
print(p())                       # Đang phát A
```

### Attribute lookup

```python
class Animal:
    sound = "..."

class Dog(Animal):
    pass

d = Dog()
print(d.__dict__)        # {}     -> instance chưa có gì
print(d.sound)           # ...    -> tìm thấy ở Animal
d.sound = "Gâu"          # tạo attribute trên instance, che class attribute
print(d.__dict__)        # {'sound': 'Gâu'}
print(Dog.sound)         # ...
```

### MRO & super() trong đa kế thừa (diamond)

```python
class A:
    def hello(self):
        return "A"

class B(A):
    def hello(self):
        return "B -> " + super().hello()

class C(A):
    def hello(self):
        return "C -> " + super().hello()

class D(B, C):
    def hello(self):
        return "D -> " + super().hello()

print([cls.__name__ for cls in D.__mro__])
# ['D', 'B', 'C', 'A', 'object']
print(D().hello())
# D -> B -> C -> A
```

`super()` **không** có nghĩa là "class cha", mà là "class **kế tiếp trong MRO** của object hiện tại". Vì vậy trong `B`, `super()` lại trỏ tới `C`, không phải `A`. Nhờ đó mỗi class trong diamond chỉ được gọi **một lần**.

### Class là object, type là metaclass

```python
print(type(5))          # <class 'int'>
print(type(int))        # <class 'type'>
print(type(type))       # <class 'type'>

# Tạo class động, tương đương "class Point: x = 0"
Point = type("Point", (object,), {"x": 0})
print(Point().x)        # 0
```

### Kiểm soát truy cập thuộc tính

```python
class Config:
    def __init__(self, data):
        self._data = data

    def __getattr__(self, name):    # chỉ gọi khi lookup thông thường thất bại
        try:
            return self._data[name]
        except KeyError:
            raise AttributeError(name)

c = Config({"debug": True})
print(c.debug)   # True  (giống Proxy get trap của JS)
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| prototype chain | MRO (`Class.__mro__`) |
| `Object.getPrototypeOf(obj)` | `type(obj)`, `Class.__bases__` |
| `obj.hasOwnProperty("x")` | `"x" in obj.__dict__` |
| `toString()` | `__str__` / `__repr__` |
| `[Symbol.iterator]` | `__iter__` |
| `valueOf` / `Symbol.toPrimitive` | `__int__`, `__float__`, `__add__`... |
| không overload toán tử | `__add__`, `__eq__`, `__lt__`... |
| `Proxy` với `get` trap | `__getattr__` / `__getattribute__` |
| đơn kế thừa | đa kế thừa + C3 MRO |

## Xem thêm

- [Prototype (JS)](../js/40-Prototype.md)
- [OOP](./29-OOP.md)
- [Class](./28-Class.md)
- [self & Bound Method](./39-self-Bound-Method.md)
- [Iterator & Generator](./31-Iterator-Generator.md)
