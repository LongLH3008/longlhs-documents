[← Mục lục](./index.md)

# JSON

## Định nghĩa

Module chuẩn `json` dùng để **serialize** (chuyển object Python thành chuỗi JSON) và **deserialize** (chuyển chuỗi JSON thành object Python).

| Hàm | Ý nghĩa |
|---|---|
| `json.dumps(obj)` | object → chuỗi JSON (dump **s**tring) |
| `json.loads(s)` | chuỗi JSON → object (load **s**tring) |
| `json.dump(obj, file)` | ghi object ra file |
| `json.load(file)` | đọc object từ file |

Bảng ánh xạ kiểu:

| JSON | Python |
|---|---|
| object | `dict` |
| array | `list` |
| string | `str` |
| number | `int` / `float` |
| `true` / `false` | `True` / `False` |
| `null` | `None` |

## Giải thích đơn giản

`json.dumps` giống `JSON.stringify`, `json.loads` giống `JSON.parse`. Chữ **s** cuối tên hàm nghĩa là làm việc với **string**; không có **s** là làm việc với **file**.

Ví von: JSON là **thùng hàng tiêu chuẩn** để gửi dữ liệu giữa các ngôn ngữ. `dumps` là đóng thùng, `loads` là mở thùng.

## Ví dụ

```python
import json

user = {"name": "Long", "age": 25, "active": True, "tags": ["dev", "py"], "note": None}

# dict -> chuỗi JSON
s = json.dumps(user)
print(s)
# {"name": "Long", "age": 25, "active": true, "tags": ["dev", "py"], "note": null}

# In đẹp + giữ tiếng Việt
print(json.dumps({"city": "Hà Nội"}, indent=2, ensure_ascii=False))
# {
#   "city": "Hà Nội"
# }

# chuỗi JSON -> dict
data = json.loads('{"x": 1, "y": [1, 2]}')
print(data["y"][1])  # 2
```

### Đọc / ghi file

```python
import json

with open("config.json", "w", encoding="utf-8") as f:
    json.dump({"debug": True}, f, indent=2)

with open("config.json", encoding="utf-8") as f:
    config = json.load(f)

print(config["debug"])  # True
```

### Serialize kiểu không hỗ trợ sẵn

```python
import json
from datetime import datetime

def default(o):
    if isinstance(o, datetime):
        return o.isoformat()
    raise TypeError(f"Không serialize được {type(o)}")

print(json.dumps({"at": datetime(2026, 1, 1)}, default=default))
# {"at": "2026-01-01T00:00:00"}
```

### Bắt lỗi JSON sai

```python
try:
    json.loads("{name: 'x'}")   # key không có nháy kép -> sai chuẩn
except json.JSONDecodeError as e:
    print("JSON lỗi:", e)
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `JSON.stringify(obj, null, 2)` | `json.dumps(obj, indent=2)` |
| `JSON.parse(str)` | `json.loads(str)` |
| `replacer` | tham số `default=` |
| `reviver` | tham số `object_hook=` |
| `SyntaxError` | `json.JSONDecodeError` |

## Xem thêm

- [JSON (JS)](../js/02-JSON.md)
- [Dict](./15-Dict.md)
- [File I/O & pathlib](./34-File-IO-pathlib.md)
