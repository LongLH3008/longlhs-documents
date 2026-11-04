[← Mục lục](./index.md)

# Name Binding (không có Hoisting)

## Định nghĩa

**Name binding** là việc gắn một **tên** với một **object** trong một namespace. Các thao tác tạo binding trong Python:

- Phép gán: `x = 1`, `x += 1`, `a, b = ...`, `:=`
- `def f` (gắn tên `f` với function object), `class C`
- `import m`, `from m import x`
- Biến vòng lặp `for x in ...`, `with ... as x`, `except E as x`
- Tham số hàm

Python thực thi code **từ trên xuống, từng câu lệnh**. `def` và `class` là **câu lệnh thực thi**: chỉ khi chạy tới dòng đó, tên mới được tạo. Vì vậy Python **không có hoisting** như JavaScript.

Tuy nhiên, khi **compile một hàm**, Python quét toàn bộ thân hàm để xác định tên nào là local (tên có phép gán). Đây là điểm duy nhất "giống hoisting": tên được **xác định là local từ đầu hàm**, nhưng **chưa có giá trị** cho đến dòng gán.

## Giải thích đơn giản

Trong JS, trước khi chạy, engine "kéo" khai báo `function` và `var` lên đầu scope, nên gọi hàm trước khi khai báo vẫn được. Python thì như **đọc sách từ trang đầu**: nhân vật chưa xuất hiện mà đã nhắc tới thì Python báo "không biết người này là ai" (`NameError`).

Phần "giống TDZ của `let`" là `UnboundLocalError`: Python biết tên đó thuộc hàm này (vì thấy phép gán ở dưới), nhưng bạn dùng nó trước khi gán.

## Ví dụ

### Gọi hàm trước khi def

```python
greet()           # NameError: name 'greet' is not defined

def greet():
    print("hi")
```

```js
// JS: chạy bình thường nhờ hoisting
greet();
function greet() { console.log("hi"); }
```

### Nhưng hàm A gọi hàm B định nghĩa sau thì OK

```python
def main():
    helper()      # chỉ tra tên helper khi main() được GỌI

def helper():
    print("helper")

main()            # helper  -> lúc này helper đã tồn tại
```

Đây là lý do code Python thường đặt `main()` ở cuối file (trong main guard).

### "TDZ" kiểu Python

```python
x = "global"

def show():
    print(x)      # UnboundLocalError
    x = "local"   # phép gán này làm x thành local cho TOÀN BỘ hàm

show()
```

```js
// JS tương tự với let
let x = "global";
function show() {
  console.log(x); // ReferenceError: Cannot access 'x' before initialization
  let x = "local";
}
```

### def là câu lệnh: có thể định nghĩa có điều kiện

```python
import sys

if sys.platform == "win32":
    def clear():
        print("cls")
else:
    def clear():
        print("clear")

clear()
```

### Gán lại tên hàm

```python
def f():
    return 1

g = f             # hai tên cùng trỏ tới một function object
def f():          # tạo function object MỚI, gắn với tên f
    return 2

print(f(), g())   # 2 1
```

### class cũng là câu lệnh

```python
class A:
    print("Thân class chạy ngay khi định nghĩa")
    x = 10
# In ra: Thân class chạy ngay khi định nghĩa
```

### Xóa binding với del

```python
x = 5
del x            # xóa tên, không xóa object (object bị thu hồi nếu không còn ai tham chiếu)
print(x)         # NameError
```

## So sánh với JavaScript

| Tình huống | JavaScript | Python |
|---|---|---|
| Gọi function trước khai báo | chạy (hoisting) | `NameError` |
| Dùng `var` trước khai báo | `undefined` | không có `var` |
| Dùng `let` trước khai báo trong scope | `ReferenceError` (TDZ) | `UnboundLocalError` |
| `class` trước khai báo | `ReferenceError` (TDZ) | `NameError` |
| Khai báo hàm trong `if` | hành vi phức tạp tùy mode | rõ ràng, chỉ tạo khi nhánh chạy |

## Xem thêm

- [Hoisting (JS)](../js/37-Hoisting.md)
- [Scope (LEGB)](./48-Scope-LEGB.md)
- [Mutable & Immutable](./47-Mutable-Immutable.md)
- [if \_\_name\_\_ == "\_\_main\_\_"](./08-Main-Guard.md)
