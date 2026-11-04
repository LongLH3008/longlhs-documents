[← Mục lục](./index.md)

# Testing với pytest

## Định nghĩa

**Unit test** là đoạn code tự động kiểm tra một đơn vị nhỏ (hàm, class) cho ra kết quả đúng với đầu vào cho trước. Python có sẵn module `unittest` (kiểu xUnit, viết test bằng class) và `doctest`, nhưng thư viện bên thứ ba **`pytest`** là chuẩn de facto vì:

- Test chỉ là **hàm** bắt đầu bằng `test_`, đặt trong file `test_*.py` hoặc `*_test.py`.
- Dùng câu lệnh `assert` thường, pytest tự hiển thị chi tiết khi sai (assertion rewriting).
- **Fixture**: cơ chế chuẩn bị/dọn dẹp dữ liệu, tiêm vào test qua tên tham số.
- **Parametrize**: chạy một test với nhiều bộ dữ liệu.
- Hệ sinh thái plugin: `pytest-cov` (coverage), `pytest-asyncio`, `pytest-mock`...

## Giải thích đơn giản

Test giống **bộ đề kiểm tra kèm đáp án** cho code. Mỗi lần sửa code, bạn bấm một nút (`pytest`) và máy chấm toàn bộ đề trong vài giây. Câu nào sai, máy chỉ ra đúng câu đó, đáp án mong đợi và đáp án thực tế.

Fixture giống **người chuẩn bị phòng thi**: trước mỗi bài thi bày bàn ghế, giấy bút (tạo DB tạm, file tạm), thi xong dọn sạch, để bài thi sau không bị ảnh hưởng.

## Ví dụ

### Cài đặt & chạy

```bash
pip install pytest
pytest              # tự tìm mọi file test_*.py
pytest -v           # chi tiết từng test
pytest -k discount  # chỉ chạy test có tên chứa "discount"
pytest -x           # dừng ở lỗi đầu tiên
```

### Test đầu tiên

```python
# shop.py
def apply_discount(price: float, percent: float) -> float:
    if not 0 <= percent <= 100:
        raise ValueError("percent phải trong [0, 100]")
    return round(price * (1 - percent / 100), 2)
```

```python
# test_shop.py
import pytest
from shop import apply_discount

def test_discount_basic():
    assert apply_discount(100, 20) == 80

def test_discount_zero():
    assert apply_discount(50, 0) == 50

def test_discount_invalid():
    with pytest.raises(ValueError, match="percent"):
        apply_discount(100, 150)

def test_float():
    assert 0.1 + 0.2 == pytest.approx(0.3)   # so sánh số thực an toàn
```

Khi test sai, pytest in rõ:

```text
>       assert apply_discount(100, 20) == 70
E       assert 80.0 == 70
```

### Parametrize

```python
@pytest.mark.parametrize(
    ("price", "percent", "expected"),
    [
        (100, 10, 90),
        (100, 50, 50),
        (19.99, 15, 16.99),
        (0, 30, 0),
    ],
)
def test_discount_cases(price, percent, expected):
    assert apply_discount(price, percent) == expected
```

### Fixture

```python
import pytest

class Cart:
    def __init__(self):
        self.items = []
    def add(self, name, price):
        self.items.append((name, price))
    def total(self):
        return sum(p for _, p in self.items)

@pytest.fixture
def cart():
    c = Cart()
    c.add("bút", 10)
    yield c              # trước yield: setup, sau yield: teardown
    c.items.clear()

def test_total(cart):    # pytest tiêm fixture theo tên tham số
    assert cart.total() == 10

def test_add(cart):      # mỗi test nhận một Cart mới
    cart.add("vở", 20)
    assert cart.total() == 30
```

Fixture có sẵn hay dùng: `tmp_path` (thư mục tạm), `monkeypatch` (thay biến môi trường, hàm), `capsys` (bắt output `print`).

```python
def test_write_file(tmp_path):
    f = tmp_path / "a.txt"
    f.write_text("hi", encoding="utf-8")
    assert f.read_text(encoding="utf-8") == "hi"

def test_env(monkeypatch):
    monkeypatch.setenv("DEBUG", "1")
    import os
    assert os.environ["DEBUG"] == "1"
```

### Mock một phụ thuộc bên ngoài

```python
# weather.py
import requests

def is_hot(city: str) -> bool:
    data = requests.get(f"https://api.example.com/{city}").json()
    return data["temp"] > 30
```

```python
# test_weather.py
from unittest.mock import patch
import weather

def test_is_hot():
    with patch("weather.requests.get") as fake_get:
        fake_get.return_value.json.return_value = {"temp": 35}
        assert weather.is_hot("HCM") is True
        fake_get.assert_called_once()
```

### Đo coverage

```bash
pip install pytest-cov
pytest --cov=shop --cov-report=term-missing
```

## So sánh với JavaScript

| JavaScript (Jest / Vitest) | Python (pytest) |
|---|---|
| `test("...", () => {})` | `def test_...():` |
| `expect(x).toBe(1)` | `assert x == 1` |
| `expect(fn).toThrow("msg")` | `with pytest.raises(Err, match="msg"):` |
| `test.each([...])` | `@pytest.mark.parametrize` |
| `beforeEach` / `afterEach` | `@pytest.fixture` với `yield` |
| `jest.fn()`, `jest.mock()` | `unittest.mock.Mock`, `patch` |
| `toBeCloseTo` | `pytest.approx` |
| `--coverage` | `pytest-cov` |

## Xem thêm

- [Exception](./37-Exception.md)
- [Context Manager (with)](./45-Context-Manager.md)
- [Decorator](./44-Decorator.md)
- [Iterator & Generator](./42-Iterator-Generator.md)
- [Cài đặt, venv & pip](./01-Setup-venv-pip.md)
