[← Mục lục](./index.md)

# Cài đặt, venv & pip

## Định nghĩa

- **Python interpreter** (thường là **CPython**) là chương trình đọc và thực thi file `.py`. Python là ngôn ngữ thông dịch, kiểu động (dynamic typing), kiểu mạnh (strong typing).
- **pip** là package manager chính thức, tải thư viện từ **PyPI** (Python Package Index).
- **venv** (virtual environment) là một thư mục chứa interpreter riêng và các package riêng cho từng project, giúp các project không đụng phiên bản thư viện của nhau.

## Giải thích đơn giản

Nếu bạn quen Node.js:

- `python` giống `node`.
- `pip` giống `npm`, PyPI giống npm registry.
- `requirements.txt` (hoặc `pyproject.toml`) giống `package.json`.
- `venv` giống thư mục `node_modules`, nhưng khác ở chỗ: Node tự tìm `node_modules` gần nhất, còn Python mặc định cài **global**. Vì vậy bạn phải tự tạo và **kích hoạt** venv, nếu không mọi project sẽ dùng chung một "kho" thư viện, rất dễ xung đột.

Ví von: venv là **căn bếp riêng của từng nhà hàng**. Nhà hàng A dùng dao phiên bản 1, nhà hàng B dùng dao phiên bản 2, không ai phải chia sẻ.

## Ví dụ

### Tạo và dùng venv

```bash
# Tạo venv trong thư mục .venv
python -m venv .venv

# Kích hoạt
source .venv/bin/activate        # macOS / Linux
.venv\Scripts\activate           # Windows (cmd / PowerShell)

# Cài package
pip install requests

# Lưu danh sách package (giống package-lock ở mức đơn giản)
pip freeze > requirements.txt

# Máy khác cài lại đúng phiên bản
pip install -r requirements.txt

# Thoát venv
deactivate
```

### Chạy code

```bash
python main.py          # chạy file
python                  # mở REPL (giống gõ `node`)
python -m http.server   # chạy một module như script
```

```python
# main.py
import requests

res = requests.get("https://api.github.com")
print(res.status_code)  # 200
```

### Bảng so sánh lệnh

| Việc cần làm | Node.js | Python |
|---|---|---|
| Chạy file | `node app.js` | `python app.py` |
| Cài package | `npm install axios` | `pip install requests` |
| File khai báo dependency | `package.json` | `requirements.txt` / `pyproject.toml` |
| Thư mục chứa package | `node_modules/` | `.venv/lib/.../site-packages/` |
| Chạy tool không cài global | `npx eslint` | `python -m pip`, `pipx run black` |

## Lưu ý

- Luôn thêm `.venv/` vào `.gitignore`, giống `node_modules/`.
- Dùng `python -m pip` thay vì `pip` để chắc chắn pip thuộc đúng interpreter đang dùng.
- Các tool hiện đại như **uv** hoặc **Poetry** gộp venv + pip + lock file, trải nghiệm gần giống npm/pnpm.

## Xem thêm

- [ES6 Modules (JS)](../js/29-ES6-Modules.md)
- [Modules & Packages](./38-Modules-Packages.md)
- [Dockerfile](../docker/dockerfile.md)
