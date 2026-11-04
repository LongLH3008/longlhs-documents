[← Mục lục](./index.md)

# asyncio Event Loop

## Định nghĩa

**Event loop** của `asyncio` là bộ lập lịch chạy trên **một thread**, điều phối nhiều coroutine theo mô hình **cooperative multitasking** (đa nhiệm hợp tác): một coroutine chạy cho tới khi tự **nhường quyền** tại `await`, lúc đó loop chọn task khác sẵn sàng để chạy tiếp.

Các thành phần chính:

| Thành phần | Vai trò |
|---|---|
| **Ready queue** | hàng đợi các callback/task sẵn sàng chạy ở vòng lặp kế tiếp |
| **Scheduled (timer heap)** | callback hẹn giờ (`call_later`, `asyncio.sleep`) |
| **Selector** | theo dõi socket/file descriptor qua `epoll`/`kqueue`/IOCP, báo khi I/O sẵn sàng |
| **Task** | gói coroutine; mỗi lần được "đánh thức" sẽ chạy tiếp tới `await` kế tiếp |
| **Future** | kết quả chưa có; khi được set kết quả, các task đang chờ được đưa vào ready queue |

Mỗi vòng lặp (iteration) của loop: chạy hết callback đang sẵn sàng → hỏi selector xem I/O nào xong (chờ tối đa tới timer gần nhất) → chuyển các timer tới hạn và I/O xong vào ready queue → lặp lại.

## Giải thích đơn giản

Ý tưởng **giống event loop của JS**: một thread, không chặn I/O, xử lý xong việc này thì nhặt việc khác. Khác biệt:

- JS có hai hàng đợi rõ ràng: **microtask** (Promise) ưu tiên trước **macrotask** (setTimeout). asyncio chỉ có **một ready queue** dạng FIFO, không phân chia micro/macro.
- JS event loop luôn chạy sẵn. asyncio phải khởi động bằng `asyncio.run()`, và code đồng bộ thông thường của Python **không chạy trên event loop**.
- Trong JS, `await` một Promise đã resolve vẫn luôn nhường quyền (qua microtask). Trong Python, `await` một coroutine **không nhất thiết nhường quyền**: nếu coroutine đó không thực sự chờ gì (không có I/O, không `sleep`), nó chạy thẳng như gọi hàm bình thường.

Ví von: event loop là **một người phục vụ duy nhất trong quán** phục vụ nhiều bàn. Bàn nào đang chờ bếp (I/O) thì người phục vụ đi lo bàn khác, khi bếp gọi "món xong" thì quay lại. Nhưng nếu một bàn giữ người phục vụ đứng nói chuyện mãi (code CPU nặng, `time.sleep`), cả quán phải đợi.

## Ví dụ

### Thứ tự chạy khi nhường quyền

```python
import asyncio

async def worker(name, delay):
    print(f"{name}: bắt đầu")
    await asyncio.sleep(delay)      # nhường quyền cho loop
    print(f"{name}: xong sau {delay}s")

async def main():
    await asyncio.gather(
        worker("A", 2),
        worker("B", 1),
        worker("C", 0),
    )

asyncio.run(main())
# A: bắt đầu
# B: bắt đầu
# C: bắt đầu
# C: xong sau 0s
# B: xong sau 1s
# A: xong sau 2s
```

### await không tự động nhường quyền

```python
import asyncio

async def no_io():
    return 42                      # không có await thật sự nào bên trong

async def ticker():
    for i in range(3):
        print("tick", i)
        await asyncio.sleep(0)     # sleep(0) = nhường quyền ngay (≈ setTimeout(fn, 0))

async def main():
    asyncio.create_task(ticker())
    for _ in range(3):
        await no_io()              # không nhường -> ticker chưa được chạy
    print("main xong vòng no_io")
    await asyncio.sleep(0.1)       # lúc này ticker mới chạy

asyncio.run(main())
# main xong vòng no_io
# tick 0
# tick 1
# tick 2
```

### Code CPU nặng chặn cả loop

```python
import asyncio, time

async def heartbeat():
    while True:
        print("♥", time.strftime("%X"))
        await asyncio.sleep(1)

def cpu_heavy():
    return sum(i * i for i in range(30_000_000))

async def main():
    asyncio.create_task(heartbeat())
    await asyncio.sleep(0)

    cpu_heavy()                                   # heartbeat bị "đóng băng" vài giây
    await asyncio.to_thread(cpu_heavy)            # chạy ở thread khác, loop vẫn thở
    # Với CPU thật sự nặng, dùng ProcessPoolExecutor (xem bài GIL)

asyncio.run(main())
```

### Callback cấp thấp

```python
import asyncio

async def main():
    loop = asyncio.get_running_loop()
    loop.call_soon(print, "call_soon: vào ready queue")
    loop.call_later(0.5, print, "call_later: sau 0.5s")
    print("đồng bộ chạy trước")
    await asyncio.sleep(1)

asyncio.run(main())
# đồng bộ chạy trước
# call_soon: vào ready queue
# call_later: sau 0.5s
```

### Future thủ công (≈ new Promise)

```python
import asyncio

async def main():
    loop = asyncio.get_running_loop()
    fut = loop.create_future()                 # JS: new Promise((resolve) => ...)
    loop.call_later(1, fut.set_result, "ok")   # resolve sau 1 giây
    print(await fut)                           # ok

asyncio.run(main())
```

## So sánh với JavaScript

| JavaScript | Python asyncio |
|---|---|
| event loop có sẵn trong runtime | `asyncio.run()` tạo và chạy loop |
| microtask queue + macrotask queue | một ready queue (FIFO) + timer heap |
| `queueMicrotask(fn)` | `loop.call_soon(fn)` |
| `setTimeout(fn, ms)` | `loop.call_later(sec, fn)` |
| `setTimeout(fn, 0)` để nhường | `await asyncio.sleep(0)` |
| `await` luôn nhường quyền | `await` coroutine không I/O thì không nhường |
| `new Promise(resolve => ...)` | `loop.create_future()` + `set_result` |
| Worker threads cho CPU nặng | `asyncio.to_thread`, `ProcessPoolExecutor` |
| libuv | selector (`epoll`/`kqueue`/IOCP), có thể thay bằng `uvloop` |

## Xem thêm

- [Event Loop (JS)](../js/43-Event-Loop.md)
- [Coroutine & async/await](./41-async-await.md)
- [GIL, Threading & Multiprocessing](./43-GIL-Concurrency.md)
- [Background Worker](../be/background-worker.md)
