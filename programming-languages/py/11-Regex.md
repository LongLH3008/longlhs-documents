[← Mục lục](./index.md)

# Regular Expression (module re)

## Định nghĩa

**Regular expression (regex)** là một ngôn ngữ mô tả **mẫu (pattern) chuỗi ký tự**, dùng để tìm kiếm, kiểm tra, tách và thay thế văn bản. Trong Python, regex được cung cấp bởi module chuẩn `re`.

Các hàm chính:

| Hàm | Trả về | Ý nghĩa |
|---|---|---|
| `re.match(p, s)` | `Match` / `None` | khớp **ở đầu** chuỗi |
| `re.fullmatch(p, s)` | `Match` / `None` | khớp **toàn bộ** chuỗi |
| `re.search(p, s)` | `Match` / `None` | tìm vị trí khớp **đầu tiên** ở bất kỳ đâu |
| `re.findall(p, s)` | `list` | mọi kết quả khớp (hoặc các group) |
| `re.finditer(p, s)` | iterator `Match` | như `findall` nhưng lazy, có vị trí |
| `re.sub(p, repl, s)` | `str` | thay thế |
| `re.split(p, s)` | `list` | tách chuỗi theo pattern |
| `re.compile(p, flags)` | `Pattern` | biên dịch sẵn để dùng lại |

## Giải thích đơn giản

Regex giống **khuôn cắt bánh**: bạn mô tả hình dạng ("bắt đầu bằng chữ, theo sau là 3 chữ số, rồi dấu gạch") và Python tìm mọi chỗ trong văn bản vừa khuôn.

Luôn viết pattern bằng **raw string** `r"..."` để `\d`, `\b` không bị Python hiểu nhầm thành escape của chuỗi thường.

Cú pháp hay dùng:

| Ký hiệu | Ý nghĩa |
|---|---|
| `.` | ký tự bất kỳ (trừ xuống dòng) |
| `\d` `\w` `\s` | chữ số / chữ-số-gạch dưới / khoảng trắng |
| `\D` `\W` `\S` | phủ định của các loại trên |
| `[abc]` `[^abc]` `[a-z]` | tập ký tự / ngoài tập / khoảng |
| `*` `+` `?` | 0+ / 1+ / 0 hoặc 1 lần |
| `{n}` `{n,m}` | đúng n / từ n tới m lần |
| `^` `$` `\b` | đầu chuỗi / cuối chuỗi / ranh giới từ |
| `(...)` | group (bắt giá trị) |
| `(?P<name>...)` | named group |
| `(?:...)` | group không bắt |
| `a\|b` | a hoặc b |
| `*?` `+?` | non-greedy (lấy ít nhất có thể) |

## Ví dụ

### search, match, fullmatch

```python
import re

text = "Đơn hàng #A123 giao ngày 2026-11-04"

m = re.search(r"#(\w+)", text)
if m:
    print(m.group(0))   # #A123   (toàn bộ khớp)
    print(m.group(1))   # A123    (group 1)
    print(m.span())     # (9, 14)

print(re.match(r"\d+", text))         # None, vì đầu chuỗi không phải số
print(bool(re.fullmatch(r"\d{4}", "2026")))   # True
```

### Validate dữ liệu

```python
PHONE = re.compile(r"(0|\+84)\d{9}")

for p in ["0912345678", "+84912345678", "12345"]:
    print(p, bool(PHONE.fullmatch(p)))
# 0912345678 True / +84912345678 True / 12345 False
```

### findall & named group

```python
log = "user=an id=7; user=binh id=12"

print(re.findall(r"id=(\d+)", log))              # ['7', '12']
print(re.findall(r"user=(\w+) id=(\d+)", log))   # [('an', '7'), ('binh', '12')]

for m in re.finditer(r"user=(?P<name>\w+) id=(?P<id>\d+)", log):
    print(m["name"], int(m["id"]))               # an 7 / binh 12
    print(m.groupdict())                         # {'name': 'an', 'id': '7'}
```

### sub & split

```python
print(re.sub(r"\s+", " ", "nhiều    khoảng   trắng"))   # nhiều khoảng trắng

# Che số điện thoại, giữ 3 số cuối
print(re.sub(r"\d(?=\d{3})", "*", "0912345678"))         # *******678

# repl là hàm
print(re.sub(r"\d+", lambda m: str(int(m[0]) * 2), "a1 b20"))   # a2 b40

# Đổi định dạng ngày bằng backreference
print(re.sub(r"(\d{4})-(\d{2})-(\d{2})", r"\3/\2/\1", "2026-11-04"))  # 04/11/2026

print(re.split(r"[,;]\s*", "a, b;c,  d"))   # ['a', 'b', 'c', 'd']
```

### Flags & greedy

```python
print(re.findall(r"python", "Python PYTHON", re.IGNORECASE))   # ['Python', 'PYTHON']

html = "<b>đậm</b> và <i>nghiêng</i>"
print(re.findall(r"<.+>", html))    # ['<b>đậm</b> và <i>nghiêng</i>']  greedy
print(re.findall(r"<.+?>", html))   # ['<b>', '</b>', '<i>', '</i>']   non-greedy

# re.VERBOSE: viết pattern nhiều dòng có comment
DATE = re.compile(r"""
    (?P<year>\d{4}) -   # năm
    (?P<month>\d{2}) -  # tháng
    (?P<day>\d{2})      # ngày
""", re.VERBOSE)
print(DATE.search("hôm nay 2026-11-04")["month"])   # 11
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `/\d+/g` (literal) | `re.compile(r"\d+")` |
| `/x/i` | `re.compile("x", re.IGNORECASE)` |
| `re.test(s)` | `bool(re.search(p, s))` |
| `s.match(/x/)` | `re.search(p, s)` |
| `s.matchAll(/x/g)` | `re.finditer(p, s)` |
| `s.replace(/x/g, "y")` | `re.sub(p, "y", s)` (mặc định thay **tất cả**) |
| `s.split(/,\s*/)` | `re.split(r",\s*", s)` |
| `(?<name>...)`, `$1` | `(?P<name>...)`, `\1` hoặc `\g<name>` |

## Xem thêm

- [String](./19-String.md)
- [Walrus Operator & Assignment](./06-Walrus-Assignment.md)
- [File I/O & pathlib](./46-File-IO-pathlib.md)
