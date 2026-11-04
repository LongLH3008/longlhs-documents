[← Mục lục](./index.md)

# Scope (LEGB)

## Định nghĩa

**Scope** là vùng code mà một tên (biến, hàm, class) có thể truy cập. Python tra cứu tên theo quy tắc **LEGB**, từ trong ra ngoài:

| Cấp | Tên | Ý nghĩa |
|---|---|---|
| **L** | Local | bên trong hàm hiện tại |
| **E** | Enclosing | trong các hàm bao ngoài (hàm lồng nhau) |
| **G** | Global | top-level của module (file) |
| **B** | Built-in | tên có sẵn: `print`, `len`, `range`... |

Chỉ có **module, function, class** (và comprehension) tạo scope mới. **`if`, `for`, `while`, `with`, `try` KHÔNG tạo scope**.

Từ khóa thay đổi phạm vi gán:

- `global x`: gán vào biến `x` ở global scope.
- `nonlocal x`: gán vào biến `x` ở enclosing scope gần nhất.

## Giải thích đơn giản

Khác biệt lớn nhất với JS hiện đại: JS có **block scope** (`let`, `const` sống trong `{}`), còn Python **không có block scope**. Biến tạo trong `if` hay `for` vẫn dùng được sau khối đó, giống `var` của JS ngày xưa.

Điểm thứ hai: trong Python, **gán giá trị cho một tên bên trong hàm sẽ biến tên đó thành local** của hàm, trừ khi bạn khai báo `global`/`nonlocal`. Đọc thì được tra ra ngoài, nhưng gán thì không.

Ví von: tìm chìa khóa theo thứ tự **túi áo (Local) → ba lô (Enclosing) → phòng (Global) → kho chung của tòa nhà (Built-in)**. Tìm thấy ở đâu trước thì dùng ở đó.

## Ví dụ

### LEGB

```python
x = "global"

def outer():
    x = "enclosing"
    def inner():
        x = "local"
        print(x)       # local
    inner()
    print(x)           # enclosing

outer()
print(x)               # global
print(len("abc"))      # len lấy từ built-in
```

### Không có block scope

```python
if True:
    message = "hi"
print(message)         # hi (JS với let sẽ ReferenceError)

for i in range(3):
    pass
print(i)               # 2  -> biến vòng lặp còn sống sau vòng lặp

# Ngoại lệ: comprehension có scope riêng
[j for j in range(3)]
print(j)               # NameError
```

### Gán trong hàm làm biến thành local

```python
count = 0

def increment():
    count += 1         # UnboundLocalError: cannot access local variable 'count'

def increment_ok():
    global count
    count += 1

increment_ok()
print(count)           # 1
```

Python quyết định một tên là local **lúc compile hàm**: chỉ cần có phép gán `count = ...` ở bất kỳ đâu trong hàm, `count` là local trong **toàn bộ** hàm.

### nonlocal

```python
def make_counter():
    count = 0
    def increment():
        nonlocal count     # dùng count của make_counter
        count += 1
        return count
    return increment

c = make_counter()
print(c(), c(), c())   # 1 2 3
```

### Đọc biến ngoài thì không cần khai báo

```python
config = {"debug": False}

def enable_debug():
    config["debug"] = True    # không gán tên config, chỉ sửa object -> không cần global

enable_debug()
print(config)                 # {'debug': True}
```

### Che khuất built-in (shadowing)

```python
list = [1, 2, 3]       # che mất hàm built-in list
list("abc")            # TypeError: 'list' object is not callable
del list               # xóa tên global, built-in hiện lại
```

### Xem các tên trong scope

```python
def f():
    a = 1
    print(locals())    # {'a': 1}

print(globals().keys())
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| block scope với `let`/`const` | **không có** block scope |
| `var` function scope | mọi biến đều function scope |
| closure đọc/ghi biến ngoài tự do | ghi biến ngoài cần `nonlocal`/`global` |
| global object `window`/`globalThis` | `globals()`, module scope |
| ReferenceError (TDZ) | `UnboundLocalError` / `NameError` |
| scope chain | LEGB |

## Xem thêm

- [Scope (JS)](../js/36-Scope.md)
- [ES6: let & const (JS)](../js/19-ES6-let-const.md)
- [Name Binding (không có Hoisting)](./37-Name-Binding.md)
- [Closure](./38-Closure.md)
