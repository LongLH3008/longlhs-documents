[← Mục lục](./index.md)

# GIL, Threading & Multiprocessing

## Định nghĩa

- **GIL** (Global Interpreter Lock) là một mutex trong **CPython** chỉ cho phép **một thread thực thi Python bytecode tại một thời điểm** trong một process. GIL giúp việc quản lý bộ nhớ (reference counting) an toàn và đơn giản, nhưng làm các thread **không chạy song song thật sự** với code thuần Python nặng CPU.
- GIL **được nhả ra** khi thread chờ I/O (mạng, file, `time.sleep`) và trong nhiều thư viện C (NumPy, hashlib, zlib...).
- **Free-threaded CPython** (PEP 703, build `python3.13t` trở đi) là bản build tùy chọn **không có GIL**; vẫn đang trong giai đoạn chuyển tiếp, bản mặc định vẫn có GIL.

Ba mô hình đồng thời trong Python:

| Mô hình | Module | Song song thật? | Hợp với |
|---|---|---|---|
| **Async** | `asyncio` | Không (1 thread) | rất nhiều tác vụ I/O (hàng nghìn kết nối) |
| **Threading** | `threading`, `ThreadPoolExecutor` | Không với CPU (do GIL), có với I/O | I/O với thư viện đồng bộ (requests, DB driver) |
| **Multiprocessing** | `multiprocessing`, `ProcessPoolExecutor` | **Có** (mỗi process có GIL riêng) | tính toán nặng CPU |

## Giải thích đơn giản

Hãy tưởng tượng một **căn bếp chỉ có một con dao** (GIL). Bạn thuê bốn đầu bếp (thread), nhưng tại mỗi lúc chỉ một người cầm dao thái rau (chạy code Python). Nếu việc chính là **chờ nước sôi** (I/O), bốn người vẫn hiệu quả, vì người chờ nồi sẽ đưa dao cho người khác. Nhưng nếu việc chính là **thái** (CPU), thuê thêm người cũng không nhanh hơn.

Muốn thái nhanh hơn, bạn phải **mở thêm bếp** (process), mỗi bếp có dao riêng. Đổi lại, các bếp không dùng chung tủ lạnh: dữ liệu phải được gửi qua lại (serialize bằng pickle), tốn chi phí.

So với JS: Node.js chạy JavaScript trên một thread (giống asyncio). Muốn song song thì dùng **Worker Threads** (mỗi worker có isolate và event loop riêng, giao tiếp bằng message) hoặc **cluster** (nhiều process). Python multiprocessing gần với cluster/worker của Node.

## Ví dụ

### Threading: hiệu quả với I/O

```python
import time
from concurrent.futures import ThreadPoolExecutor

def download(url):
    time.sleep(1)                 # giả lập chờ mạng, GIL được nhả ra
    return f"xong {url}"

urls = [f"https://site/{i}" for i in range(5)]

start = time.perf_counter()
with ThreadPoolExecutor(max_workers=5) as pool:
    results = list(pool.map(download, urls))
print(f"{time.perf_counter() - start:.1f}s")   # ~1.0s thay vì 5s
```

### Threading: không giúp gì với CPU

```python
import time
from concurrent.futures import ThreadPoolExecutor

def cpu_task(n):
    return sum(i * i for i in range(n))

N = 10_000_000

start = time.perf_counter()
for _ in range(4):
    cpu_task(N)
print("tuần tự:", round(time.perf_counter() - start, 1))   # ví dụ ~3.2s

start = time.perf_counter()
with ThreadPoolExecutor(4) as pool:
    list(pool.map(cpu_task, [N] * 4))
print("4 thread:", round(time.perf_counter() - start, 1))  # vẫn ~3.2s, do GIL
```

### Multiprocessing: song song thật cho CPU

```python
import time
from concurrent.futures import ProcessPoolExecutor

def cpu_task(n):
    return sum(i * i for i in range(n))

if __name__ == "__main__":                     # bắt buộc trên Windows/macOS
    N = 10_000_000
    start = time.perf_counter()
    with ProcessPoolExecutor(4) as pool:
        list(pool.map(cpu_task, [N] * 4))
    print("4 process:", round(time.perf_counter() - start, 1))  # ~0.9s trên máy 4 nhân
```

### Race condition vẫn xảy ra dù có GIL

```python
import threading

counter = 0
lock = threading.Lock()

def unsafe():
    global counter
    for _ in range(100_000):
        counter += 1          # đọc - cộng - ghi: không atomic, thread có thể bị chen ngang

def safe():
    global counter
    for _ in range(100_000):
        with lock:
            counter += 1

threads = [threading.Thread(target=safe) for _ in range(4)]
for t in threads: t.start()
for t in threads: t.join()
print(counter)                # 400000 (với unsafe có thể ra số nhỏ hơn)
```

GIL chỉ đảm bảo **một bytecode** chạy nguyên vẹn, không đảm bảo **một dòng code** chạy nguyên vẹn.

### Kết hợp asyncio với thread/process

```python
import asyncio
from concurrent.futures import ProcessPoolExecutor

def cpu_task(n):
    return sum(i * i for i in range(n))

async def main():
    loop = asyncio.get_running_loop()
    with ProcessPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, cpu_task, 10_000_000)
    print(result)

if __name__ == "__main__":
    asyncio.run(main())
```

## Chọn mô hình nào

```
Tác vụ chủ yếu là gì?
├── Chờ I/O (HTTP, DB, file)
│   ├── Thư viện hỗ trợ async (httpx, asyncpg)  → asyncio
│   └── Thư viện đồng bộ (requests, psycopg2)    → ThreadPoolExecutor
└── Tính toán CPU (xử lý ảnh, số liệu)
    ├── Dùng NumPy/Pandas (C nhả GIL)           → thường đã đủ nhanh
    └── Code Python thuần                         → ProcessPoolExecutor
```

## So sánh với JavaScript

| JavaScript (Node.js) | Python |
|---|---|
| một thread JS + event loop | asyncio (một thread + event loop) |
| libuv thread pool cho fs/crypto | `ThreadPoolExecutor`, `asyncio.to_thread` |
| Worker Threads (song song thật, không chia sẻ object) | `multiprocessing` / free-threaded build |
| `cluster` module | `multiprocessing`, Gunicorn nhiều worker |
| `postMessage` / structured clone | `Queue`, `Pipe`, pickle |
| `SharedArrayBuffer` + `Atomics` | `multiprocessing.shared_memory`, `Value`, `Lock` |
| không có GIL nhưng mỗi isolate là đơn luồng | có GIL trong một process |

## Xem thêm

- [Event Loop (JS)](../js/43-Event-Loop.md)
- [asyncio Event Loop](./42-asyncio-Event-Loop.md)
- [Coroutine & async/await](./41-async-await.md)
- [Race Condition](../be/race-condition.md)
- [Background Worker](../be/background-worker.md)
- [Batch Job](../be/batch-job.md)
