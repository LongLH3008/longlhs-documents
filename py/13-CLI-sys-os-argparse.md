[← Mục lục](./index.md)

# CLI: sys, os & argparse

## Định nghĩa

Ba module chuẩn dùng để viết chương trình dòng lệnh (CLI) và tương tác với hệ điều hành:

| Module | Dùng để |
|---|---|
| `sys` | thông tin và điều khiển **interpreter**: `sys.argv` (tham số dòng lệnh), `sys.exit()` (mã thoát), `sys.stdin/stdout/stderr`, `sys.path`, `sys.version` |
| `os` | giao tiếp **hệ điều hành**: biến môi trường `os.environ`, thư mục làm việc `os.getcwd()`, tiến trình `os.getpid()`... |
| `argparse` | **phân tích tham số dòng lệnh** theo khai báo: tham số vị trí, option `--flag`, kiểu, giá trị mặc định, tự sinh `--help` |

Ngoài ra `subprocess` dùng để chạy chương trình khác.

## Giải thích đơn giản

Một chương trình CLI giống **quầy gọi món**:

- `sys.argv` là **tờ giấy khách viết vội**: chỉ là danh sách chuỗi, bạn tự đọc, tự hiểu.
- `argparse` là **thực đơn có ô đánh dấu**: khai báo trước có những món nào, món nào bắt buộc, khách ghi sai thì báo lỗi rõ ràng, khách hỏi `--help` thì đưa thực đơn.
- `os.environ` là **thông tin dán sẵn trên quầy** (biến môi trường): mật khẩu DB, chế độ chạy... không cần khách nói.
- `sys.exit(code)` là **trả lời "xong" hay "lỗi"** cho người gọi: `0` là thành công, khác `0` là thất bại. Script shell và CI dựa vào mã này.

## Ví dụ

### sys.argv

```python
# greet.py
import sys

print(sys.argv)              # ['greet.py', 'An', '3']
if len(sys.argv) < 2:
    print("Cách dùng: python greet.py <tên>", file=sys.stderr)
    sys.exit(1)

print(f"Chào {sys.argv[1]}")
```

```bash
python greet.py An 3
```

### Biến môi trường với os.environ

```python
import os

db_url = os.environ.get("DATABASE_URL", "sqlite:///dev.db")   # có mặc định
debug = os.getenv("DEBUG", "0") == "1"
secret = os.environ["SECRET_KEY"]     # không có -> KeyError, fail sớm là tốt

print(os.getcwd())                    # thư mục đang đứng
```

```bash
DEBUG=1 SECRET_KEY=abc python app.py      # Linux/macOS
# PowerShell: $env:DEBUG="1"; python app.py
```

### argparse

```python
# resize.py
import argparse
from pathlib import Path

def main() -> int:
    parser = argparse.ArgumentParser(description="Đổi kích thước ảnh")
    parser.add_argument("src", type=Path, help="file ảnh nguồn")
    parser.add_argument("-w", "--width", type=int, required=True, help="chiều rộng (px)")
    parser.add_argument("-q", "--quality", type=int, default=85, choices=range(1, 101),
                        metavar="1-100")
    parser.add_argument("-v", "--verbose", action="store_true", help="in chi tiết")
    parser.add_argument("--format", choices=["jpg", "png", "webp"], default="jpg")
    args = parser.parse_args()

    if not args.src.exists():
        parser.error(f"không thấy file {args.src}")   # in lỗi + thoát mã 2

    if args.verbose:
        print(f"Resize {args.src} -> {args.width}px, q={args.quality}, {args.format}")
    return 0

if __name__ == "__main__":
    raise SystemExit(main())
```

```bash
python resize.py cat.png -w 800 -v --format webp
python resize.py --help
# usage: resize.py [-h] -w WIDTH [-q 1-100] [-v] [--format {jpg,png,webp}] src
```

### Subcommand (kiểu git commit / git push)

```python
import argparse

parser = argparse.ArgumentParser(prog="todo")
sub = parser.add_subparsers(dest="cmd", required=True)

add = sub.add_parser("add", help="thêm việc")
add.add_argument("title")

ls = sub.add_parser("list", help="liệt kê")
ls.add_argument("--done", action="store_true")

args = parser.parse_args(["add", "Học Python"])   # truyền list để test
print(args)   # Namespace(cmd='add', title='Học Python')
```

### Chạy lệnh khác bằng subprocess

```python
import subprocess

result = subprocess.run(["git", "status", "--short"], capture_output=True, text=True, check=True)
print(result.stdout)
```

Truyền **list** tham số thay vì chuỗi với `shell=True` để tránh command injection.

## So sánh với JavaScript (Node.js)

| Node.js | Python |
|---|---|
| `process.argv` | `sys.argv` (không có phần tử `node` ở đầu) |
| `process.env.X` | `os.environ["X"]` / `os.getenv("X")` |
| `process.exit(1)` | `sys.exit(1)` |
| `process.cwd()` | `os.getcwd()` / `Path.cwd()` |
| thư viện `commander`, `yargs` | có sẵn `argparse` |
| `child_process.execFile` | `subprocess.run([...])` |

## Xem thêm

- [if \_\_name\_\_ == "\_\_main\_\_"](./08-Main-Guard.md)
- [Cài đặt, venv & pip](./01-Setup-venv-pip.md)
- [File I/O & pathlib](./46-File-IO-pathlib.md)
- [Logging](./12-Logging.md)
- [Command Injection](../security/command-injection.md)
