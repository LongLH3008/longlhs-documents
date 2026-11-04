[← Mục lục](./index.md)

# OOP

## Định nghĩa

**Object-Oriented Programming** (lập trình hướng đối tượng) tổ chức chương trình thành các **object** gồm dữ liệu và hành vi. Bốn tính chất:

| Tính chất | Ý nghĩa | Trong Python |
|---|---|---|
| **Encapsulation** (đóng gói) | gom dữ liệu + hành vi, che giấu chi tiết bên trong | `_attr`, `__attr`, `@property` |
| **Inheritance** (kế thừa) | class con dùng lại và mở rộng class cha | `class Dog(Animal):`, `super()`, đa kế thừa |
| **Polymorphism** (đa hình) | cùng một lời gọi, mỗi object xử lý theo cách riêng | method overriding, **duck typing** |
| **Abstraction** (trừu tượng) | chỉ lộ ra giao diện cần thiết | `abc.ABC`, `@abstractmethod`, `typing.Protocol` |

Python hỗ trợ **multiple inheritance** (đa kế thừa), điều JS không có.

## Giải thích đơn giản

Ý tưởng OOP giống JS, nhưng Python nổi tiếng với **duck typing**: "Nếu nó đi như vịt và kêu như vịt, thì nó là vịt". Python không quan tâm object thuộc class nào, chỉ quan tâm object **có method cần dùng không**. Vì vậy đa hình trong Python thường không cần kế thừa.

Ví von: tuyển tài xế, công ty duck typing không hỏi "bạn tốt nghiệp trường lái nào" (class), mà chỉ hỏi "bạn có lái được không" (có method `drive()` không).

## Ví dụ

### Kế thừa & super()

```python
class Animal:
    def __init__(self, name):
        self.name = name

    def speak(self):
        return "..."

    def intro(self):
        return f"{self.name} nói: {self.speak()}"

class Dog(Animal):
    def __init__(self, name, breed):
        super().__init__(name)        # gọi constructor cha (JS: super(name))
        self.breed = breed

    def speak(self):                  # override
        return "Gâu gâu"

class Cat(Animal):
    def speak(self):
        return "Meo meo"

for a in [Dog("Lu", "corgi"), Cat("Mimi")]:
    print(a.intro())
# Lu nói: Gâu gâu
# Mimi nói: Meo meo

print(isinstance(Dog("x", "y"), Animal))   # True
print(issubclass(Cat, Animal))             # True
```

### Abstraction với ABC

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self) -> float: ...

class Circle(Shape):
    def __init__(self, r):
        self.r = r
    def area(self):
        return 3.14159 * self.r ** 2

Shape()          # TypeError: Can't instantiate abstract class Shape
print(Circle(2).area())   # 12.56636
```

### Duck typing

```python
class FileLogger:
    def write(self, msg):
        print(f"[file] {msg}")

class ConsoleLogger:              # không kế thừa gì cả
    def write(self, msg):
        print(f"[console] {msg}")

def log_all(logger, messages):
    for m in messages:
        logger.write(m)           # chỉ cần có method write

log_all(FileLogger(), ["a"])
log_all(ConsoleLogger(), ["b"])
```

### Protocol: duck typing có kiểm tra kiểu

```python
from typing import Protocol

class Writer(Protocol):
    def write(self, msg: str) -> None: ...

def log_all(logger: Writer, messages: list[str]) -> None:
    for m in messages:
        logger.write(m)
# mypy/pyright chấp nhận mọi class có write(msg: str), không cần kế thừa Writer
```

### Đa kế thừa & mixin

```python
class JsonMixin:
    def to_json(self):
        import json
        return json.dumps(self.__dict__, ensure_ascii=False)

class TimestampMixin:
    def touch(self):
        from datetime import datetime
        self.updated_at = datetime.now().isoformat()

class Article(JsonMixin, TimestampMixin):
    def __init__(self, title):
        self.title = title

a = Article("Python OOP")
a.touch()
print(a.to_json())   # {"title": "Python OOP", "updated_at": "..."}
```

### Composition thay vì kế thừa

```python
class Engine:
    def start(self):
        return "Động cơ chạy"

class Car:
    def __init__(self):
        self.engine = Engine()     # Car "có một" Engine, không phải "là một" Engine
    def start(self):
        return self.engine.start()
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `class Dog extends Animal` | `class Dog(Animal):` |
| `super(name)` trong constructor | `super().__init__(name)` |
| `super.speak()` | `super().speak()` |
| chỉ đơn kế thừa (dùng mixin bằng `Object.assign`) | **đa kế thừa** `class A(B, C):` |
| `interface` (TS) | `ABC` / `Protocol` |
| `abstract class` (TS) | `ABC` + `@abstractmethod` |
| kế thừa qua prototype chain | kế thừa qua **MRO** |

## Xem thêm

- [OOP (JS)](../js/34-OOP.md)
- [Class](./39-Class.md)
- [Data Model, Dunder Methods & MRO](./52-Data-Model-MRO.md)
- [Dependency Injection](../be/dependency-injection.md)
