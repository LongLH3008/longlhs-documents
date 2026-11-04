[← Mục lục](./index.md)

# if \_\_name\_\_ == "\_\_main\_\_"

## Định nghĩa

Mỗi module Python có biến đặc biệt `__name__`:

- Khi file được **chạy trực tiếp** (`python app.py`), `__name__ == "__main__"`.
- Khi file được **import** từ file khác, `__name__` là tên module (ví dụ `"app"`).

Khối `if __name__ == "__main__":` (gọi là **main guard**) chứa code chỉ chạy khi file là điểm khởi động, không chạy khi bị import.

## Giải thích đơn giản

Khi Python `import` một file, nó **chạy toàn bộ code ở top-level** của file đó. Nếu file có `print()` hay gọi hàm ở ngoài cùng, chỉ cần import là chúng chạy, rất phiền.

Main guard giống tấm biển **"Chỉ dành cho chủ nhà"**: khi bạn (chủ nhà) chạy file trực tiếp thì cửa mở, còn khách (file khác import vào) chỉ được dùng đồ trong nhà (hàm, class) chứ không kích hoạt "tiệc" bên trong.

Trong JS, IIFE dùng để tạo scope riêng và chạy ngay. Python không cần IIFE vì mỗi file đã là một scope riêng (module scope). Thứ gần nhất với IIFE là main guard (chạy ngay khi là entry point) và khai báo hàm `main()`.

## Ví dụ

```python
# math_utils.py
def add(a, b):
    return a + b

def main():
    # Code thử nghiệm / chạy CLI
    print("2 + 3 =", add(2, 3))

if __name__ == "__main__":
    main()
```

```bash
python math_utils.py
# 2 + 3 = 5
```

```python
# app.py
from math_utils import add   # KHÔNG in "2 + 3 = 5"

print(add(10, 20))  # 30
```

### Không có main guard thì sao

```python
# bad_utils.py
def add(a, b):
    return a + b

print("Đang test:", add(1, 1))   # chạy mỗi lần ai đó import file này
```

```python
import bad_utils   # in ra "Đang test: 2" dù bạn không muốn
```

### Bắt buộc với multiprocessing trên Windows/macOS

```python
from multiprocessing import Pool

def square(x):
    return x * x

if __name__ == "__main__":       # thiếu dòng này sẽ lỗi hoặc tạo process vô hạn
    with Pool(4) as p:
        print(p.map(square, [1, 2, 3]))  # [1, 4, 9]
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| IIFE `(() => { ... })()` để tạo scope riêng | không cần, mỗi file đã là module scope |
| `require.main === module` (CommonJS) | `__name__ == "__main__"` |
| `import.meta.main` (Deno/Bun) | `__name__ == "__main__"` |

## Xem thêm

- [IIFE (JS)](../js/07-IIFE.md)
- [Modules & Packages](./27-Modules-Packages.md)
- [GIL, Threading & Multiprocessing](./43-GIL-Concurrency.md)
