[← Mục lục](./index.md)

# Image và Container

## 1. Định nghĩa kỹ thuật

**Image** là một **template chỉ đọc (read-only)**, gồm nhiều **layer** xếp chồng lên nhau theo chuẩn OCI. Mỗi layer là một tập thay đổi trên filesystem (thêm, sửa, xóa file). Image còn kèm **metadata**: lệnh chạy mặc định (`CMD`/`ENTRYPOINT`), biến môi trường, cổng, user...

**Container** là một **instance đang chạy (hoặc đã dừng) của image**. Docker lấy các layer read-only của image và thêm lên trên cùng một **writable layer** mỏng. Mọi thay đổi khi container chạy chỉ ghi vào layer này (cơ chế **copy-on-write**).

Image được định danh bằng:

- **Tên + tag**: `node:20-alpine`, `myapp:1.4.2`. Tag mặc định là `latest`.
- **Digest**: `node@sha256:3a1f...`, là hash nội dung, bất biến.

## 2. Giải thích đơn giản

**Image giống công thức + nguyên liệu đóng gói sẵn**, còn **container là món ăn được nấu ra** từ công thức đó. Một công thức nấu ra được nhiều đĩa, mỗi đĩa nêm thêm chút gia vị riêng (writable layer) mà công thức gốc không đổi.

Hoặc theo góc nhìn lập trình: **image là class, container là object**.

```
           Container A          Container B
        ┌───────────────┐   ┌───────────────┐
        │ writable layer│   │ writable layer│   ← riêng từng container, mất khi xóa
        └───────┬───────┘   └───────┬───────┘
                └─────────┬─────────┘
        ┌─────────────────▼───────────────────┐
        │ Layer 3: COPY . /app                │
        │ Layer 2: RUN npm ci                 │   ← image, read-only, dùng chung
        │ Layer 1: node:20-alpine (base)      │
        └─────────────────────────────────────┘
```

Vì các layer read-only được **dùng chung**, chạy 10 container từ cùng một image không tốn gấp 10 lần dung lượng ổ đĩa.

## 3. Vòng đời container

```
   docker create        docker start           docker stop / app thoát
 ──────────────► created ───────────► running ─────────────────────► exited
                                        │  ▲                           │
                          docker pause  │  │ docker unpause            │ docker rm
                                        ▼  │                           ▼
                                       paused                       (đã xóa)
```

- `docker run` = `docker create` + `docker start`.
- Container **sống khi tiến trình chính (PID 1) còn sống**. Tiến trình chính thoát là container chuyển sang `exited`.
- `docker stop` gửi `SIGTERM`, chờ 10 giây, nếu chưa thoát thì gửi `SIGKILL`. App nên bắt `SIGTERM` để tắt êm (graceful shutdown).

## 4. Ví dụ

### Xem layer của một image

```bash
docker pull node:20-alpine
docker image history node:20-alpine
```

Mỗi dòng trong kết quả là một layer, kèm lệnh đã tạo ra nó và kích thước.

### Chứng minh writable layer là riêng và tạm thời

```bash
docker run --name c1 alpine sh -c "echo hello > /data.txt"
docker run --name c2 alpine cat /data.txt
# cat: can't open '/data.txt': No such file or directory  ← c2 không thấy file của c1

docker diff c1
# A /data.txt   ← A = Added, C = Changed, D = Deleted

docker rm c1 c2   # xóa container là mất luôn /data.txt
```

Muốn dữ liệu tồn tại lâu dài, dùng [Volume](./volume.md).

### Tag và đẩy image lên registry

```bash
docker tag myapp:latest ghcr.io/myteam/myapp:1.4.2
docker push ghcr.io/myteam/myapp:1.4.2
```

## 5. Lưu ý thực tế

- **Đừng dùng tag `latest` trong production.** `latest` chỉ là một tag bình thường, không có nghĩa "mới nhất", và có thể bị ghi đè bất cứ lúc nào. Dùng tag phiên bản cụ thể, hoặc pin bằng digest.
- **Container nên stateless**: coi container là thứ dùng xong vứt (cattle, not pets). Cấu hình qua biến môi trường, dữ liệu qua volume hoặc dịch vụ ngoài.
- **Một container, một mối quan tâm**: web server một container, database một container. Dễ scale và thay thế độc lập.

## 6. Xem thêm

- [Docker là gì](./docker-overview.md)
- [Dockerfile](./dockerfile.md): mỗi chỉ thị trong Dockerfile tạo ra một layer.
- [Volume và Bind Mount](./volume.md): lưu dữ liệu vượt qua vòng đời container.
- [Docker CLI Cheatsheet](./docker-cli-cheatsheet.md)
