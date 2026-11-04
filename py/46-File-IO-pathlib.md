[← Mục lục](./index.md)

# File I/O & pathlib

## Định nghĩa

- **`open(path, mode, encoding)`** là hàm built-in mở file, trả về **file object**. Các mode chính:

| Mode | Ý nghĩa |
|---|---|
| `"r"` | đọc (mặc định), file phải tồn tại |
| `"w"` | ghi, **xóa nội dung cũ** hoặc tạo mới |
| `"a"` | ghi nối vào cuối |
| `"x"` | tạo mới, lỗi nếu đã tồn tại |
| `"b"` | chế độ nhị phân, kết hợp: `"rb"`, `"wb"` |
| `"+"` | đọc và ghi, ví dụ `"r+"` |

- **`pathlib.Path`** là API hướng đối tượng để thao tác đường dẫn và file, thay thế cho `os.path`.

## Giải thích đơn giản

Giống module `fs` và `path` của Node.js, nhưng:

- Python mặc định là **đồng bộ** (blocking), không cần callback hay `await`. Muốn async thì dùng `asyncio` + thư viện như `aiofiles`.
- `pathlib` dùng toán tử `/` để nối đường dẫn, đọc rất tự nhiên: `Path("data") / "users.json"`.

Luôn chỉ định `encoding="utf-8"` khi làm việc với text, vì trên Windows encoding mặc định có thể không phải UTF-8, làm tiếng Việt bị lỗi font.

## Ví dụ

### Đọc & ghi text

```python
# Ghi
with open("note.txt", "w", encoding="utf-8") as f:
    f.write("Dòng 1\n")
    f.writelines(["Dòng 2\n", "Dòng 3\n"])

# Đọc toàn bộ
with open("note.txt", encoding="utf-8") as f:
    content = f.read()

# Đọc từng dòng (tiết kiệm RAM với file lớn)
with open("note.txt", encoding="utf-8") as f:
    for line in f:
        print(line.rstrip())

# Ghi nối
with open("note.txt", "a", encoding="utf-8") as f:
    print("Dòng 4", file=f)     # print cũng ghi được vào file
```

### pathlib

```python
from pathlib import Path

base = Path("data")
file = base / "users" / "list.json"      # JS: path.join("data", "users", "list.json")

print(file.name)        # list.json
print(file.stem)        # list
print(file.suffix)      # .json
print(file.parent)      # data/users
print(file.exists())

file.parent.mkdir(parents=True, exist_ok=True)   # mkdir -p
file.write_text('{"ok": true}', encoding="utf-8")
print(file.read_text(encoding="utf-8"))

print(Path.cwd(), Path.home())
print(Path(__file__).resolve().parent)  # thư mục chứa file code hiện tại (JS: __dirname)
```

### Liệt kê & tìm file

```python
from pathlib import Path

for p in Path(".").iterdir():           # con trực tiếp
    print(p, "DIR" if p.is_dir() else "FILE")

for p in Path(".").glob("*.py"):        # khớp mẫu ở thư mục hiện tại
    print(p)

for p in Path(".").rglob("*.md"):       # đệ quy
    print(p)
```

### Đổi tên, xóa, copy

```python
from pathlib import Path
import shutil

p = Path("old.txt")
p.write_text("x")
p.rename("new.txt")
Path("new.txt").unlink(missing_ok=True)     # xóa file

shutil.copy("a.txt", "b.txt")
shutil.rmtree("tmp_dir", ignore_errors=True) # xóa thư mục và nội dung (rm -rf)
```

### File nhị phân & CSV

```python
with open("image.png", "rb") as f:
    header = f.read(8)
print(header)   # b'\x89PNG\r\n\x1a\n'

import csv
with open("users.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.DictWriter(f, fieldnames=["name", "age"])
    writer.writeheader()
    writer.writerow({"name": "An", "age": 25})

with open("users.csv", encoding="utf-8") as f:
    for row in csv.DictReader(f):
        print(row["name"], row["age"])
```

## So sánh với Node.js

| Node.js | Python |
|---|---|
| `fs.readFileSync(p, "utf8")` | `Path(p).read_text(encoding="utf-8")` |
| `fs.writeFileSync(p, s)` | `Path(p).write_text(s, encoding="utf-8")` |
| `fs.appendFileSync(p, s)` | `open(p, "a")` |
| `fs.existsSync(p)` | `Path(p).exists()` |
| `fs.mkdirSync(p, { recursive: true })` | `Path(p).mkdir(parents=True, exist_ok=True)` |
| `fs.readdirSync(p)` | `Path(p).iterdir()` |
| `fs.rmSync(p, { recursive: true })` | `shutil.rmtree(p)` |
| `path.join(a, b)` | `Path(a) / b` |
| `path.extname(p)` | `Path(p).suffix` |
| `__dirname` | `Path(__file__).parent` |
| `readline` / stream | `for line in f:` |

## Xem thêm

- [Context Manager (with)](./45-Context-Manager.md)
- [JSON](./03-JSON.md)
- [Iterator & Generator](./42-Iterator-Generator.md)
- [Upload file lớn](../be/upload-heavy-file.md)
