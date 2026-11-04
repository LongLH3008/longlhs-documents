[← Mục lục](./index.md)

# String

## Định nghĩa

`str` là kiểu dữ liệu **immutable** (bất biến) biểu diễn chuỗi ký tự **Unicode**. Chuỗi là một **sequence**, nên hỗ trợ index, slicing, `len()`, `in` và duyệt bằng `for`.

Cách viết:

| Cú pháp | Ý nghĩa |
|---|---|
| `'abc'` hoặc `"abc"` | chuỗi thường, hai loại nháy như nhau |
| `"""..."""` | chuỗi nhiều dòng, cũng dùng làm docstring |
| `f"..."` | f-string, nhúng biểu thức |
| `r"..."` | raw string, không xử lý `\` (hay dùng cho regex, đường dẫn Windows) |
| `b"..."` | bytes, không phải str |

## Giải thích đơn giản

Gần như giống JS string: bất biến, mọi method đều trả về chuỗi **mới**. Khác biệt lớn nhất là **slicing** `s[start:stop:step]`, giống `slice()` của JS nhưng mạnh hơn và hỗ trợ **index âm** (`s[-1]` là ký tự cuối).

Ví von: chuỗi là chuỗi hạt cố định đã gắn keo. Bạn không thay được một hạt, nhưng có thể cắt ra một đoạn hoặc làm một chuỗi mới.

## Ví dụ

### Index và slicing

```python
s = "Python"
print(s[0])      # P
print(s[-1])     # n   (JS: s.at(-1))
print(s[0:3])    # Pyt (từ 0 đến trước 3)
print(s[2:])     # thon
print(s[:-2])    # Pyth
print(s[::2])    # Pto (bước 2)
print(s[::-1])   # nohtyP (đảo chuỗi)
print(len(s))    # 6

s[0] = "J"       # TypeError: 'str' object does not support item assignment
```

### Method thường dùng

```python
t = "  Hello, World  "

print(t.strip())              # "Hello, World"     (JS: trim)
print(t.lower(), t.upper())
print("hello world".title())  # Hello World
print("a,b,c".split(","))     # ['a', 'b', 'c']
print("-".join(["a", "b"]))   # a-b  (JS: arr.join("-"), Python ngược lại)
print("abc".replace("b", "x"))# axc  (thay TẤT CẢ, giống replaceAll)
print("python".startswith("py"), "file.py".endswith(".py"))  # True True
print("banana".find("na"))    # 2  (không thấy trả -1, giống indexOf)
print("banana".count("a"))    # 3
print("42".isdigit())         # True
print("7".zfill(3))           # 007  (JS: padStart(3, "0"))
print("hi".center(6, "*"))    # **hi**
```

### Kiểm tra chuỗi con, duyệt

```python
print("World" in "Hello World")  # True  (JS: includes)

for ch in "abc":
    print(ch)
```

### Chuỗi nhiều dòng & raw string

```python
sql = """
SELECT *
FROM users
WHERE id = 1
"""

path = r"C:\new\table"   # \n và \t không bị hiểu là xuống dòng/tab
```

### Nối chuỗi hiệu quả

```python
# Chậm khi nối nhiều lần trong vòng lặp (tạo chuỗi mới mỗi lần)
result = ""
for i in range(3):
    result += str(i)

# Nhanh và Pythonic
result = "".join(str(i) for i in range(3))  # "012"
```

### str và bytes

```python
text = "Xin chào"
data = text.encode("utf-8")   # bytes
print(data)                   # b'Xin ch\xc3\xa0o'
print(data.decode("utf-8"))   # Xin chào
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `s.length` | `len(s)` |
| `s.at(-1)` | `s[-1]` |
| `s.slice(1, 3)` | `s[1:3]` |
| `s.trim()` | `s.strip()` |
| `s.includes(x)` | `x in s` |
| `s.indexOf(x)` | `s.find(x)` |
| `s.replaceAll(a, b)` | `s.replace(a, b)` |
| `arr.join(",")` | `",".join(arr)` |
| `s.padStart(3, "0")` | `s.zfill(3)` / `s.rjust(3, "0")` |
| `` `Hi ${name}` `` | `f"Hi {name}"` |
| `s.split("").reverse().join("")` | `s[::-1]` |

## Xem thêm

- [String (JS)](../js/12-String.md)
- [f-string](./30-f-string.md)
- [List](./20-List.md)
- [Mutable & Immutable](./47-Mutable-Immutable.md)
