[← Mục lục](./index.md)

# self & Bound Method

## Định nghĩa

- **`self`** là tên quy ước cho **tham số đầu tiên** của instance method, nhận chính instance đang gọi method. Đây là tham số **tường minh**, không phải từ khóa.
- Khi truy cập `obj.method`, Python (thông qua **descriptor protocol**, method `__get__` của function) tạo ra một **bound method**: một object gói function cùng instance, lưu trong `__func__` và `__self__`. Gọi `obj.method(x)` tương đương `Class.method(obj, x)`.
- Truy cập qua class (`Class.method`) trả về **function thường**, phải tự truyền instance.

## Giải thích đơn giản

Trong JS, `this` được quyết định **lúc gọi** (ai đứng trước dấu chấm), nên khi tách method ra (`const f = obj.method`) là mất `this`, phải dùng `bind`, `call`, `apply` hoặc arrow function để sửa. Đây là nguồn bug rất lớn trong JS.

Python giải quyết bằng cách:

1. Bắt bạn viết `self` rõ ràng, nhìn là biết method dùng instance nào.
2. Khi lấy `obj.method`, Python **tự "bind" sẵn** instance vào. Bạn tách method ra, truyền làm callback, lưu vào biến... `self` **vẫn đúng**. Không cần `bind`.

Ví von: JS `this` giống **người phát ngôn đọc thông báo**: ai cầm micro thì "tôi" là người đó. Python bound method giống **thư có chữ ký sẵn**: chuyển tay bao nhiêu lần thì người gửi vẫn là người đã ký.

## Ví dụ

### self là tham số bình thường

```python
class User:
    def __init__(self, name):
        self.name = name

    def greet(self, greeting):
        return f"{greeting}, {self.name}"

u = User("An")
print(u.greet("Hi"))            # Hi, An
print(User.greet(u, "Hi"))      # Hi, An  -> hai cách tương đương
```

### Bound method giữ instance

```python
u = User("An")
f = u.greet                     # tách method ra
print(f("Hello"))               # Hello, An  -> self vẫn đúng

print(f.__self__ is u)          # True
print(f.__func__ is User.greet) # True
```

So sánh với JS:

```js
class User {
  constructor(name) { this.name = name; }
  greet(g) { return `${g}, ${this.name}`; }
}
const u = new User("An");
const f = u.greet;
f("Hello");                 // TypeError: Cannot read properties of undefined
const g = u.greet.bind(u);  // phải bind
g("Hello");                 // "Hello, An"
```

### Truyền method làm callback

```python
class Button:
    def __init__(self, label):
        self.label = label
        self.clicks = 0

    def on_click(self):
        self.clicks += 1
        print(f"{self.label} clicked {self.clicks} lần")

btn = Button("OK")
callbacks = [btn.on_click, btn.on_click]
for cb in callbacks:
    cb()
# OK clicked 1 lần
# OK clicked 2 lần
```

### Tương đương call / apply / bind

```python
from functools import partial

def introduce(self, city):
    return f"{self.name} ở {city}"

u = User("An")

print(introduce(u, "HN"))                # call: truyền "this" như tham số đầu
print(introduce(*[u, "HN"]))             # apply: rải list tham số
bound = partial(introduce, u)            # bind: cố định tham số đầu
print(bound("HCM"))                      # An ở HCM

import types
m = types.MethodType(introduce, u)       # gắn function thành bound method
print(m("DN"))                           # An ở DN
```

### classmethod nhận cls, staticmethod không nhận gì

```python
class Config:
    env = "dev"

    @classmethod
    def current(cls):          # cls là class (hoặc subclass) đang gọi
        return cls.env

    @staticmethod
    def version():             # không có self/cls
        return "1.0"

class ProdConfig(Config):
    env = "prod"

print(ProdConfig.current())    # prod
print(Config.version())        # 1.0
```

### Quên self

```python
class Bad:
    def hello():               # thiếu self
        return "hi"

Bad().hello()   # TypeError: Bad.hello() takes 0 positional arguments but 1 was given
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `this` ngầm, quyết định lúc gọi | `self` tường minh, bind khi truy cập `obj.method` |
| `const f = obj.m` → mất `this` | `f = obj.m` → vẫn giữ `self` |
| `fn.call(obj, a)` | `fn(obj, a)` / `Class.m(obj, a)` |
| `fn.apply(obj, [a])` | `fn(obj, *[a])` |
| `fn.bind(obj)` | `functools.partial(fn, obj)` / `types.MethodType(fn, obj)` |
| arrow function giữ `this` ngoài | không cần, vì không có `this` ngầm |
| `this` trong static | `cls` trong `@classmethod` |

## Xem thêm

- [this, bind, call, apply (JS)](../js/39-this-bind-call-apply.md)
- [Class](./28-Class.md)
- [Data Model, Dunder Methods & MRO](./40-Data-Model-MRO.md)
