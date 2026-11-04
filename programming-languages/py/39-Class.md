[← Mục lục](./index.md)

# Class

## Định nghĩa

`class` định nghĩa một **kiểu dữ liệu mới** gồm thuộc tính (attribute) và phương thức (method). Các thành phần chính:

| Thành phần | Cú pháp | Ý nghĩa |
|---|---|---|
| Constructor | `def __init__(self, ...)` | khởi tạo instance |
| Instance attribute | `self.x = ...` | dữ liệu riêng của từng object |
| Class attribute | khai báo trực tiếp trong thân class | dùng chung cho mọi instance |
| Instance method | `def m(self, ...)` | tham số đầu tiên là instance |
| Class method | `@classmethod def m(cls, ...)` | tham số đầu là class, hay dùng làm factory |
| Static method | `@staticmethod def m(...)` | không nhận `self` hay `cls` |
| Property | `@property` | getter/setter truy cập như thuộc tính |

**`@dataclass`** (module `dataclasses`) tự sinh `__init__`, `__repr__`, `__eq__` cho class chủ yếu chứa dữ liệu.

## Giải thích đơn giản

Giống `class` của JS ES6, với các khác biệt:

- Constructor tên là `__init__`, không phải `constructor`.
- Phải viết `self` **tường minh** làm tham số đầu tiên của method và khi truy cập thuộc tính (`self.name`). JS dùng `this` ngầm.
- Tạo object **không cần `new`**: `User("An")`.
- Không có `private` thật sự. Quy ước `_name` là "đừng đụng vào", `__name` thì Python đổi tên (name mangling) để khó đụng nhầm.

Ví von: class là **bản vẽ ngôi nhà**, instance là **từng ngôi nhà xây theo bản vẽ**. `self` là cách mỗi ngôi nhà nói "của tôi".

## Ví dụ

### Class cơ bản

```python
class User:
    species = "human"                 # class attribute (dùng chung)

    def __init__(self, name, age):
        self.name = name              # instance attribute
        self.age = age

    def greet(self):
        return f"Xin chào, tôi là {self.name}"

    def __repr__(self):               # chuỗi khi in object (giống toString)
        return f"User(name={self.name!r}, age={self.age})"

u = User("An", 25)                    # không có new
print(u.greet())                      # Xin chào, tôi là An
print(u)                              # User(name='An', age=25)
print(User.species, u.species)        # human human
```

### classmethod & staticmethod

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    @classmethod
    def from_fahrenheit(cls, f):      # factory: tạo instance theo cách khác
        return cls((f - 32) * 5 / 9)

    @staticmethod
    def is_valid(c):                  # hàm tiện ích, không cần self/cls
        return c >= -273.15

t = Temperature.from_fahrenheit(212)
print(t.celsius)                      # 100.0
print(Temperature.is_valid(-300))     # False
```

### property: getter / setter

```python
class Account:
    def __init__(self, balance):
        self._balance = balance       # _ : quy ước nội bộ

    @property
    def balance(self):                # JS: get balance()
        return self._balance

    @balance.setter
    def balance(self, value):         # JS: set balance(v)
        if value < 0:
            raise ValueError("Số dư không được âm")
        self._balance = value

acc = Account(100)
acc.balance = 50          # gọi setter
print(acc.balance)        # 50, gọi getter
acc.balance = -1          # ValueError
```

### Private & name mangling

```python
class Secret:
    def __init__(self):
        self.__key = "abc"     # đổi tên thành _Secret__key

s = Secret()
print(s.__key)                 # AttributeError
print(s._Secret__key)          # abc  -> vẫn truy cập được, chỉ là khó hơn
```

### dataclass: class chứa dữ liệu

```python
from dataclasses import dataclass, field

@dataclass
class Product:
    name: str
    price: float
    tags: list[str] = field(default_factory=list)   # tránh mutable default

p = Product("Bút", 5.0)
print(p)                         # Product(name='Bút', price=5.0, tags=[])
print(p == Product("Bút", 5.0))  # True, tự sinh __eq__

@dataclass(frozen=True)          # immutable
class Point:
    x: int
    y: int
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `constructor(name)` | `def __init__(self, name):` |
| `this.name` | `self.name` |
| `new User("An")` | `User("An")` |
| `static create()` | `@classmethod` / `@staticmethod` |
| `get x()` / `set x(v)` | `@property` / `@x.setter` |
| `#secret` (private thật) | `_secret` (quy ước) / `__secret` (mangling) |
| `toString()` | `__str__` / `__repr__` |
| `class A extends B` | `class A(B):` |
| `instanceof` | `isinstance(obj, A)` |

## Xem thêm

- [ES6: Classes (JS)](../js/28-ES6-Classes.md)
- [OOP](./40-OOP.md)
- [self & Bound Method](./51-self-Bound-Method.md)
- [Data Model, Dunder Methods & MRO](./52-Data-Model-MRO.md)
