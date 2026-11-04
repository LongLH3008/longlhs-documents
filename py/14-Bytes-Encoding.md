[← Mục lục](./index.md)

# bytes, Unicode & Encoding

## Định nghĩa

Python 3 tách rõ hai kiểu dữ liệu văn bản và nhị phân:

| Kiểu | Chứa | Mutable |
|---|---|---|
| `str` | dãy **Unicode code point** (ký tự trừu tượng, ví dụ `'ệ'` = U+1EC7) | không |
| `bytes` | dãy **byte** (số nguyên 0–255) | không |
| `bytearray` | như `bytes` | có |

**Encoding** là quy tắc chuyển `str` thành `bytes` (`str.encode()`), **decoding** là chiều ngược lại (`bytes.decode()`). Encoding phổ biến nhất là **UTF-8**: mã hóa mỗi code point thành 1 đến 4 byte, tương thích ASCII. Đọc/ghi file, mạng, socket, hash, mã hóa đều làm việc với `bytes`.

## Giải thích đơn giản

`str` là **lời nói**, `bytes` là **băng ghi âm**. Muốn gửi lời nói qua dây điện (file, mạng) phải **ghi âm** theo một định dạng (encode). Bên nhận phải **phát lại đúng định dạng đó** (decode). Ghi bằng UTF-8 mà phát bằng định dạng khác thì nghe ra tiếng lạ: đó là lỗi chữ "Ä‘áº¹p" (mojibake) quen thuộc.

Chữ tiếng Việt có dấu chiếm **nhiều byte** trong UTF-8, nên `len("ệ") == 1` nhưng `len("ệ".encode()) == 3`.

## Ví dụ

### encode & decode

```python
s = "Việt"
b = s.encode("utf-8")
print(b)             # b'Vi\xe1\xbb\x87t'
print(len(s), len(b))   # 4 6
print(b.decode("utf-8"))   # Việt

print(b[0])          # 86  -> index bytes trả về int
print(b[:2])         # b'Vi' -> slice trả về bytes
```

### Literal bytes

```python
header = b"GIF89a"            # chỉ chứa ký tự ASCII
raw = bytes([0x48, 0x69])     # b'Hi'
print(raw.hex())              # '4869'
print(bytes.fromhex("4869"))  # b'Hi'

# b"Việt"  -> SyntaxError: bytes literal chỉ cho ASCII
```

### Lỗi giải mã & cách xử lý

```python
data = "Đẹp".encode("utf-8")

print(data.decode("latin-1"))               # 'Ä\x90áº¹p'  mojibake
data.decode("ascii")                        # UnicodeDecodeError
print(data.decode("ascii", errors="replace"))   # '�����p'
print(data.decode("ascii", errors="ignore"))    # 'p'
```

### File: luôn chỉ định encoding

```python
from pathlib import Path

Path("note.txt").write_text("Xin chào", encoding="utf-8")   # mode text
print(Path("note.txt").read_bytes())        # b'Xin ch\xc3\xa0o'

with open("logo.png", "rb") as f:           # mode nhị phân -> bytes
    signature = f.read(8)
```

Trên Windows, encoding mặc định có thể không phải UTF-8 (tùy locale), nên luôn ghi rõ `encoding="utf-8"` khi mở file text.

### Unicode normalization

Cùng một chữ "ệ" có thể biểu diễn bằng 1 code point (dựng sẵn) hoặc nhiều code point (chữ gốc + dấu). Nhìn giống nhau nhưng `==` trả `False`:

```python
import unicodedata

a = "ệ"                 # 'ệ' dựng sẵn
b = "ệ"          # 'e' + dấu nặng + dấu mũ
print(a, b, a == b)          # ệ ệ False
print(unicodedata.normalize("NFC", a) == unicodedata.normalize("NFC", b))   # True

# Bỏ dấu tiếng Việt (làm slug)
def strip_accents(s):
    s = s.replace("đ", "d").replace("Đ", "D")
    return "".join(c for c in unicodedata.normalize("NFD", s) if unicodedata.category(c) != "Mn")

print(strip_accents("Tiếng Việt"))   # Tieng Viet
```

### Ứng dụng: hash & base64

```python
import hashlib, base64

print(hashlib.sha256("mật khẩu".encode()).hexdigest()[:16])
print(base64.b64encode(b"hello").decode())   # 'aGVsbG8='
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `string` (UTF-16 bên trong) | `str` (code point) |
| `"😀".length === 2` | `len("😀") == 1` |
| `Uint8Array`, `Buffer` (Node) | `bytes` / `bytearray` |
| `new TextEncoder().encode(s)` | `s.encode("utf-8")` |
| `new TextDecoder().decode(b)` | `b.decode("utf-8")` |
| `s.normalize("NFC")` | `unicodedata.normalize("NFC", s)` |
| `btoa` / `Buffer.from(x).toString("base64")` | `base64.b64encode` |

## Xem thêm

- [String](./19-String.md)
- [Data Type](./17-Data-Type.md)
- [File I/O & pathlib](./46-File-IO-pathlib.md)
- [JSON](./03-JSON.md)
