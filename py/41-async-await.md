[← Mục lục](./index.md)

# Coroutine & async/await

## Định nghĩa

- **Coroutine function**: hàm khai báo bằng `async def`. Gọi nó **không chạy thân hàm**, mà trả về một **coroutine object**.
- **`await`**: tạm dừng coroutine hiện tại cho đến khi một **awaitable** (coroutine, `Task`, `Future`) hoàn tất, trả quyền điều khiển lại cho **event loop** trong lúc chờ.
- **`asyncio`**: thư viện chuẩn cung cấp event loop và công cụ chạy đồng thời: `asyncio.run()`, `create_task()`, `gather()`, `TaskGroup`, `wait_for()`, `sleep()`.
- **Task**: một coroutine đã được lên lịch chạy trên event loop (tương tự một Promise đang chạy).
- **Future**: object đại diện cho kết quả sẽ có trong tương lai (gần với Promise ở mức thấp).

## Giải thích đơn giản

Cú pháp gần như giống JS, nhưng có **một khác biệt rất quan trọng**:

- JS: gọi `fetchData()` là **Promise bắt đầu chạy ngay** (eager).
- Python: gọi `fetch_data()` chỉ tạo coroutine object, **chưa chạy gì cả** (lazy). Nó chỉ chạy khi bị `await`, hoặc được gói thành Task (`asyncio.create_task`).

Ngoài ra JS có event loop **sẵn trong runtime**, còn Python phải **tự khởi động** bằng `asyncio.run(main())`.

Ví von: coroutine của Python là **công thức nấu ăn viết trên giấy**: viết xong chưa có món. Phải giao cho đầu bếp (event loop) qua `await` hoặc `create_task` thì mới nấu. Promise của JS thì giống **gọi món ở nhà hàng**: vừa gọi là bếp đã bắt đầu làm.

## Ví dụ

### Cơ bản

```python
import asyncio

async def fetch_user(user_id):
    print(f"Bắt đầu lấy user {user_id}")
    await asyncio.sleep(1)               # giả lập I/O, JS: await new Promise(r => setTimeout(r, 1000))
    return {"id": user_id, "name": f"User{user_id}"}

async def main():
    user = await fetch_user(1)
    print(user)

asyncio.run(main())                       # khởi động event loop, chạy main, đóng loop
```

### Coroutine không await thì không chạy

```python
async def main():
    fetch_user(1)        # RuntimeWarning: coroutine 'fetch_user' was never awaited
                         # và KHÔNG in "Bắt đầu lấy user 1"
```

### Chạy song song: gather (≈ Promise.all)

```python
import asyncio, time

async def main():
    start = time.perf_counter()

    # Tuần tự: ~3 giây
    # for i in range(3): await fetch_user(i)

    # Đồng thời: ~1 giây
    users = await asyncio.gather(fetch_user(1), fetch_user(2), fetch_user(3))
    print(users)
    print(f"{time.perf_counter() - start:.1f}s")   # 1.0s

asyncio.run(main())
```

### create_task: bắt đầu chạy ngay (giống Promise)

```python
async def main():
    task1 = asyncio.create_task(fetch_user(1))   # được lên lịch chạy luôn
    task2 = asyncio.create_task(fetch_user(2))
    print("Làm việc khác trong khi chờ...")
    u1 = await task1
    u2 = await task2
    print(u1, u2)
```

### TaskGroup (Python 3.11+): an toàn hơn gather

```python
async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(fetch_user(1))
        t2 = tg.create_task(fetch_user(2))
    # Ra khỏi khối: mọi task đã xong. Nếu một task lỗi, các task còn lại bị hủy.
    print(t1.result(), t2.result())
```

### Xử lý lỗi & timeout

```python
async def failing():
    await asyncio.sleep(0.1)
    raise ValueError("lỗi API")

async def main():
    try:
        await failing()                         # JS: try { await p } catch (e) {}
    except ValueError as e:
        print("Bắt được:", e)

    # Giống Promise.allSettled
    results = await asyncio.gather(fetch_user(1), failing(), return_exceptions=True)
    print(results)   # [{'id': 1, ...}, ValueError('lỗi API')]

    # Timeout (≈ Promise.race với setTimeout)
    try:
        await asyncio.wait_for(asyncio.sleep(10), timeout=1)
    except TimeoutError:
        print("Quá thời gian")

asyncio.run(main())
```

### Gọi HTTP thật

```python
import asyncio
import httpx

async def main():
    async with httpx.AsyncClient() as client:
        responses = await asyncio.gather(
            client.get("https://api.github.com"),
            client.get("https://httpbin.org/get"),
        )
        print([r.status_code for r in responses])

asyncio.run(main())
```

### Không dùng hàm blocking trong coroutine

```python
import time

async def bad():
    time.sleep(2)          # chặn CẢ event loop, mọi task khác đứng im

async def good():
    await asyncio.sleep(2) # nhường event loop

async def run_blocking():
    # Bắt buộc gọi hàm blocking (thư viện không hỗ trợ async)? Đẩy sang thread
    result = await asyncio.to_thread(time.sleep, 2)
```

## So sánh với JavaScript

| JavaScript | Python |
|---|---|
| `async function f()` | `async def f():` |
| gọi `f()` → Promise **chạy ngay** | gọi `f()` → coroutine, **chưa chạy** |
| event loop có sẵn | `asyncio.run(main())` |
| top-level `await` (ESM) | chỉ trong REPL `python -m asyncio`, còn lại cần `asyncio.run` |
| `Promise.all([...])` | `asyncio.gather(...)` / `TaskGroup` |
| `Promise.allSettled` | `gather(..., return_exceptions=True)` |
| `Promise.race` | `asyncio.wait(..., return_when=FIRST_COMPLETED)` |
| `setTimeout` promise | `asyncio.sleep()` |
| `AbortController` | `task.cancel()` / `asyncio.timeout()` |
| `fetch` | `httpx` / `aiohttp` |
| `for await (const x of it)` | `async for x in it:` |

## Xem thêm

- [Promise (JS)](../js/41-Promise.md)
- [async/await (JS)](../js/42-async-await.md)
- [asyncio Event Loop](./42-asyncio-Event-Loop.md)
- [GIL, Threading & Multiprocessing](./43-GIL-Concurrency.md)
- [AbortController](../fe/abort-controller.md)
