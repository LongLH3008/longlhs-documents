[← Mục lục](./index.md)

# Logging

## Định nghĩa

`logging` là module chuẩn của Python để **ghi nhận sự kiện** khi chương trình chạy, có phân cấp mức độ và cấu hình được nơi ghi (console, file, hệ thống tập trung). Gồm 4 thành phần:

| Thành phần | Vai trò |
|---|---|
| **Logger** | điểm vào để gọi `log.info(...)`, đặt tên theo module (`logging.getLogger(__name__)`), phân cấp theo dấu chấm |
| **Handler** | quyết định log đi đâu: `StreamHandler` (console), `FileHandler`, `RotatingFileHandler`... |
| **Formatter** | định dạng dòng log (thời gian, level, tên logger, message) |
| **Filter** | lọc bản ghi theo điều kiện tùy ý |

Các level theo thứ tự tăng dần: `DEBUG (10)` < `INFO (20)` < `WARNING (30)` < `ERROR (40)` < `CRITICAL (50)`. Mặc định root logger chỉ in từ `WARNING` trở lên.

## Giải thích đơn giản

`print` giống **hét to trong phòng**: ai cũng nghe, không biết ai hét, lúc nào, quan trọng cỡ nào, và không tắt được khi lên production.

`logging` giống **sổ nhật ký có phân loại**: mỗi dòng ghi rõ giờ, nơi phát sinh, mức độ ("thông tin", "cảnh báo", "lỗi"). Bạn có thể quyết định: ở máy dev đọc hết, ở production chỉ ghi lỗi ra file, mà **không cần sửa code**, chỉ đổi cấu hình.

## Ví dụ

### Cơ bản

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s %(levelname)-8s %(name)s: %(message)s",
)

log = logging.getLogger(__name__)

log.debug("Không hiện vì dưới level INFO")
log.info("Server khởi động ở cổng %d", 8000)   # dùng %s/%d, không dùng f-string
log.warning("Bộ nhớ còn %s%%", 15)
log.error("Không kết nối được DB")
```

```text
2026-11-04 09:00:00,123 INFO     __main__: Server khởi động ở cổng 8000
2026-11-04 09:00:00,124 WARNING  __main__: Bộ nhớ còn 15%
2026-11-04 09:00:00,124 ERROR    __main__: Không kết nối được DB
```

Truyền tham số kiểu `log.info("x=%s", x)` thay vì `log.info(f"x={x}")`: chuỗi chỉ được format **khi** bản ghi thực sự được in, và các công cụ gom log nhóm được các dòng cùng mẫu.

### Ghi lại exception kèm traceback

```python
try:
    1 / 0
except ZeroDivisionError:
    log.exception("Tính toán thất bại")   # level ERROR + traceback đầy đủ
```

### Logger theo module

```python
# app/db.py
import logging
log = logging.getLogger(__name__)    # tên "app.db", con của logger "app"

def query(sql):
    log.debug("SQL: %s", sql)

# main.py: chỉ bật DEBUG cho phần db
logging.basicConfig(level=logging.WARNING)
logging.getLogger("app.db").setLevel(logging.DEBUG)
```

Thư viện **không bao giờ** nên gọi `basicConfig`; chỉ tạo logger bằng `getLogger(__name__)`. Ứng dụng (file chạy chính) mới là nơi cấu hình.

### Console + file xoay vòng

```python
import logging
from logging.handlers import RotatingFileHandler

log = logging.getLogger("app")
log.setLevel(logging.DEBUG)

fmt = logging.Formatter("%(asctime)s [%(levelname)s] %(name)s: %(message)s")

console = logging.StreamHandler()
console.setLevel(logging.INFO)
console.setFormatter(fmt)

file = RotatingFileHandler("app.log", maxBytes=5_000_000, backupCount=3, encoding="utf-8")
file.setLevel(logging.DEBUG)
file.setFormatter(fmt)

log.addHandler(console)
log.addHandler(file)

log.debug("chỉ vào file")
log.info("vào cả console và file")
```

### Cấu hình bằng dictConfig

```python
import logging.config

logging.config.dictConfig({
    "version": 1,
    "formatters": {"simple": {"format": "%(levelname)s %(name)s: %(message)s"}},
    "handlers": {"console": {"class": "logging.StreamHandler", "formatter": "simple"}},
    "root": {"level": "INFO", "handlers": ["console"]},
})
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `console.log` / `console.info` | `log.info` |
| `console.debug` | `log.debug` |
| `console.warn` | `log.warning` |
| `console.error(err)` | `log.error(...)` / `log.exception(...)` |
| thư viện `winston`, `pino` | có sẵn `logging` trong stdlib |

## Xem thêm

- [Exception](./37-Exception.md)
- [Modules & Packages](./38-Modules-Packages.md)
- [CLI: sys, os & argparse](./13-CLI-sys-os-argparse.md)
- [Biến, Hằng & Input / Output](./02-Variable-Input-Output.md)
