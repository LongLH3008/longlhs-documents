[← Mục lục](./index.md)

# Modules & Packages

## Định nghĩa

- **Module** là một file `.py`. Tên module là tên file (bỏ `.py`).
- **Package** là một thư mục chứa module, thường có file `__init__.py` (chạy khi package được import, đánh dấu thư mục là package).
- **`import`** nạp module, chạy code top-level của nó **một lần duy nhất**, rồi cache trong `sys.modules`.
- Python tìm module theo thứ tự trong **`sys.path`**: thư mục của script đang chạy, `PYTHONPATH`, thư viện chuẩn, `site-packages` (nơi pip cài).

## Giải thích đơn giản

Giống ES Modules của JS ở chỗ mỗi file là một scope riêng và có thể export/import. Khác ở chỗ:

- Python **không có `export`**: mọi tên ở top-level đều import được. Quy ước tên bắt đầu bằng `_` là "nội bộ", và biến `__all__` quy định những gì được lấy khi `from x import *`.
- Không import theo đường dẫn file (`./utils.js`), mà theo **tên module có dấu chấm** (`app.utils`).

Ví von: package là **tủ sách**, module là **cuốn sách**, hàm/class là **chương sách**. `import` là mang cả cuốn về, `from ... import` là chỉ photo một chương.

## Ví dụ

### Các kiểu import

```python
import math                      # import cả module
print(math.sqrt(16))

import numpy as np               # đặt alias (JS: import * as np from "numpy")

from math import sqrt, pi        # lấy tên cụ thể (JS: import { sqrt, pi } from "math")
print(sqrt(9), pi)

from datetime import datetime as dt   # alias cho tên cụ thể

from math import *               # lấy tất cả, KHÔNG nên dùng (làm bẩn namespace)
```

### Cấu trúc project

```
my_app/
├── main.py
└── app/
    ├── __init__.py
    ├── config.py
    └── services/
        ├── __init__.py
        └── user_service.py
```

```python
# app/services/user_service.py
from app.config import DB_URL          # absolute import (khuyên dùng)
from ..config import DB_URL            # relative import (trong package)

def get_user(user_id):
    return {"id": user_id}

def _helper():                         # quy ước: nội bộ
    ...
```

```python
# main.py
from app.services.user_service import get_user
print(get_user(1))
```

### `__init__.py` gom API của package

```python
# app/services/__init__.py
from .user_service import get_user

__all__ = ["get_user"]
```

```python
# Người dùng chỉ cần
from app.services import get_user
```

### Module chỉ chạy một lần

```python
# counter.py
print("Đang nạp counter")
count = 0
```

```python
import counter     # in "Đang nạp counter"
import counter     # không in lại, lấy từ cache sys.modules
```

### Chạy module như script

```bash
python -m app.main        # chạy từ thư mục gốc, import theo package hoạt động đúng
python -m json.tool data.json
```

## Lỗi thường gặp

- **Đặt tên file trùng thư viện** (`json.py`, `random.py`, `requests.py`): `import json` sẽ nạp file của bạn thay vì thư viện chuẩn.
- **Circular import**: `a.py` import `b.py` và ngược lại → lỗi `ImportError` hoặc tên chưa được định nghĩa. Cách sửa: tách phần dùng chung ra module thứ ba, hoặc import bên trong hàm.
- Chạy `python app/services/user_service.py` trực tiếp làm relative import lỗi; hãy dùng `python -m`.

## So sánh với JavaScript

| JavaScript (ESM) | Python |
|---|---|
| `export function f()` | chỉ cần `def f():` |
| `import { f } from "./utils.js"` | `from utils import f` |
| `import * as u from "./utils.js"` | `import utils as u` |
| `export default` | không có |
| `index.js` trong thư mục | `__init__.py` |
| `node_modules` | `site-packages` |
| `import()` động | `importlib.import_module("name")` |
| module cache | `sys.modules` |

## Xem thêm

- [ES6: Modules (JS)](../js/29-ES6-Modules.md)
- [Circular Dependencies](../be/circular-dependencies.md)
- [Cài đặt, venv & pip](./01-Setup-venv-pip.md)
- [if \_\_name\_\_ == "\_\_main\_\_"](./07-Main-Guard.md)
