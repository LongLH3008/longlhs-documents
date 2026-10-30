[← Mục lục](./index.md)

# Volume và Bind Mount

## 1. Định nghĩa kỹ thuật

Writable layer của container **mất khi container bị xóa**. Để lưu dữ liệu bền vững hoặc chia sẻ file giữa host và container, Docker cung cấp ba kiểu **mount**:

| Kiểu | Định nghĩa | Nơi lưu |
|---|---|---|
| **Volume** | Vùng lưu trữ do **Docker quản lý**, tồn tại độc lập với vòng đời container | `/var/lib/docker/volumes/<tên>/_data` (trong VM của Docker Desktop) |
| **Bind mount** | Gắn **một đường dẫn có sẵn trên host** vào container | Bất kỳ đâu trên host, do bạn chỉ định |
| **tmpfs mount** | Mount trong **RAM**, mất khi container dừng | Bộ nhớ |

## 2. Giải thích đơn giản

Container giống **phòng khách sạn**: trả phòng là dọn sạch, đồ để lại sẽ mất.

- **Volume** giống **tủ gửi đồ của khách sạn**: khách sạn quản lý, bạn đổi phòng (container mới) vẫn lấy lại được đồ. Bạn không cần biết tủ nằm ở đâu.
- **Bind mount** giống **mang chiếc vali của chính bạn vào phòng**: đồ trong vali là đồ thật của bạn, sửa ở đâu cũng thấy ở đó. Tiện, nhưng phòng có thể làm hỏng đồ của bạn.
- **tmpfs** giống **giấy nháp trên bàn**: dùng tạm, trả phòng là bỏ.

```
 Host                                   Container
 ┌───────────────────────────┐         ┌──────────────────────────┐
 │ ~/project/src  ───────────┼─ bind ─►│ /app/src                 │
 │                           │         │                          │
 │ Docker area               │         │                          │
 │  volumes/pgdata ──────────┼─ volume►│ /var/lib/postgresql/data │
 │                           │         │                          │
 │ RAM ──────────────────────┼─ tmpfs ►│ /tmp                     │
 └───────────────────────────┘         └──────────────────────────┘
```

## 3. Ví dụ

### Volume cho database

```bash
docker volume create pgdata

docker run -d --name db \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16

docker rm -f db                       # xóa container
docker run -d --name db2 \
  -e POSTGRES_PASSWORD=secret \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16                         # dữ liệu cũ vẫn còn
```

### Bind mount khi phát triển (hot reload)

```bash
docker run -d --name dev \
  -p 3000:3000 \
  -v "$(pwd)/src:/app/src" \
  myapp:dev npm run dev
```

Sửa file trong `./src` trên máy, app trong container thấy ngay và reload.

### Cú pháp `--mount` (rõ ràng hơn `-v`)

```bash
docker run --mount type=volume,source=pgdata,target=/var/lib/postgresql/data postgres:16
docker run --mount type=bind,source="$(pwd)"/config,target=/etc/app,readonly myapp
docker run --mount type=tmpfs,target=/tmp,tmpfs-size=64m myapp
```

Khác biệt quan trọng: với `-v`, nếu thư mục host chưa tồn tại thì Docker **tự tạo thư mục rỗng**. Với `--mount type=bind`, Docker **báo lỗi**, giúp phát hiện gõ sai đường dẫn.

### Backup một volume

```bash
docker run --rm \
  -v pgdata:/data:ro \
  -v "$(pwd)":/backup \
  alpine tar czf /backup/pgdata.tgz -C /data .
```

## 4. Khi nào dùng loại nào

| Tình huống | Nên dùng |
|---|---|
| Dữ liệu database, file upload ở production | **Volume** |
| Mount source code để dev, hot reload | **Bind mount** |
| Đưa file cấu hình từ host vào (chỉ đọc) | **Bind mount** + `readonly` |
| Dữ liệu tạm, nhạy cảm, không muốn ghi xuống đĩa | **tmpfs** |

## 5. Lưu ý

- **Bind mount trên Docker Desktop (Windows/macOS) chậm** vì file phải đồng bộ qua VM. Với `node_modules`, dùng một volume riêng để "che" thư mục đó:

  ```bash
  -v "$(pwd)":/app -v /app/node_modules
  ```

- **Quyền file (UID/GID)**: file tạo trong container bằng user `root` sẽ thuộc root trên host khi dùng bind mount. Chạy container bằng đúng UID của bạn (`--user $(id -u):$(id -g)`) nếu cần.
- **Volume không tự bị xóa** khi xóa container. Dọn bằng `docker volume prune` (cẩn thận, mất dữ liệu).
- **Không bind mount `/var/run/docker.sock`** vào container nếu không thật sự cần: làm vậy tương đương cấp quyền root trên host.

## 6. Xem thêm

- [Image và Container](./image-container.md): vì sao writable layer là tạm thời.
- [Docker Compose](./docker-compose.md): khai báo volume trong `compose.yaml`.
- [Upload file lớn](../be/upload-heavy-file.md): file upload nên đi vào volume hoặc object storage, không vào writable layer.
- [ACID và Transaction](../db/acid-transaction.md): tính bền vững (Durability) của DB chỉ có nghĩa khi dữ liệu nằm trên volume.
