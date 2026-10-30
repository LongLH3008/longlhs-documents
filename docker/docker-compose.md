[← Mục lục](./index.md)

# Docker Compose

## 1. Định nghĩa kỹ thuật

**Docker Compose** là công cụ **định nghĩa và chạy ứng dụng nhiều container** bằng một file khai báo YAML (`compose.yaml`, tên cũ `docker-compose.yml`). File mô tả các **service**, **network**, **volume**, biến môi trường, thứ tự phụ thuộc. Lệnh `docker compose up` đọc file và tạo toàn bộ hạ tầng, `docker compose down` gỡ tất cả.

Compose V2 là plugin của Docker CLI, gọi bằng `docker compose` (có dấu cách). Bản cũ `docker-compose` (có gạch nối) viết bằng Python đã ngừng hỗ trợ.

## 2. Giải thích đơn giản

Không có Compose, bạn chạy app gồm web + API + DB + Redis giống **tự tay bật từng thiết bị trong nhà**, nhớ đúng thứ tự, đúng cấu hình, mỗi lần gõ một tràng lệnh `docker run` dài.

Compose là **một kịch bản "chế độ về nhà"**: ghi một lần trong file, sau đó chỉ cần bấm một nút (`docker compose up`) là mọi thứ bật đúng cách. Ai trong team cũng dùng chung kịch bản đó.

```
 compose.yaml ──► docker compose up
                       │
        ┌──────────────┼──────────────────┐
        ▼              ▼                  ▼
   network         volumes            containers
  myapp_default    myapp_pgdata    web · api · db · redis
```

## 3. Ví dụ: stack Node.js + PostgreSQL + Redis

```yaml
# compose.yaml
services:
  api:
    build: .                       # build từ Dockerfile trong thư mục hiện tại
    ports:
      - "3000:3000"
    environment:
      DATABASE_URL: postgres://app:${DB_PASSWORD}@db:5432/app
      REDIS_URL: redis://redis:6379
    depends_on:
      db:
        condition: service_healthy # đợi DB sẵn sàng, không chỉ đợi container khởi động
      redis:
        condition: service_started
    restart: unless-stopped

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: ${DB_PASSWORD}
      POSTGRES_DB: app
    volumes:
      - pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U app"]
      interval: 5s
      timeout: 3s
      retries: 10

  redis:
    image: redis:7-alpine

volumes:
  pgdata:
```

File `.env` cùng thư mục (không commit lên git):

```dotenv
DB_PASSWORD=super-secret
```

Các lệnh hằng ngày:

```bash
docker compose up -d            # tạo và chạy toàn bộ stack ở chế độ nền
docker compose ps               # trạng thái các service
docker compose logs -f api      # xem log service api
docker compose exec db psql -U app   # mở shell psql trong container db
docker compose up -d --build api     # build lại và khởi động lại riêng api
docker compose down             # dừng và xóa container + network
docker compose down -v          # xóa luôn volume (MẤT dữ liệu DB)
```

Những gì Compose tự làm:

- Tạo network `<project>_default`, mọi service gọi nhau bằng **tên service** (`db`, `redis`).
- Đặt tên container theo dạng `<project>-<service>-<số>`.
- Tạo volume có tên `<project>_pgdata`.

## 4. Các khái niệm hay dùng

### `depends_on` không có nghĩa là "đã sẵn sàng"

`depends_on` mặc định chỉ đảm bảo **container khởi động trước**, không đảm bảo DB đã nhận kết nối. Dùng `condition: service_healthy` kèm `healthcheck`, và app vẫn nên **tự retry** khi kết nối thất bại.

### Tách cấu hình dev và production

```bash
# compose.yaml: cấu hình chung
# compose.override.yaml: tự động được áp dụng khi chạy `docker compose up` (dùng cho dev)
docker compose up

# Production: chỉ định file rõ ràng
docker compose -f compose.yaml -f compose.prod.yaml up -d
```

Ví dụ `compose.override.yaml` cho dev:

```yaml
services:
  api:
    command: npm run dev
    volumes:
      - ./src:/app/src            # bind mount để hot reload
```

### Profiles: bật service theo nhu cầu

```yaml
services:
  adminer:
    image: adminer
    profiles: ["tools"]
    ports: ["8081:8080"]
```

```bash
docker compose --profile tools up -d   # chỉ khi cần mới bật adminer
```

### Scale một service

```bash
docker compose up -d --scale worker=3
```

Service được scale không nên publish cổng cố định (`3000:3000`), vì nhiều container không thể cùng chiếm một cổng host.

## 5. Compose ở production?

Compose phù hợp cho **dev, CI, và production quy mô nhỏ trên một máy**. Khi cần nhiều máy, tự phục hồi khi node chết, rolling update, autoscale, hãy chuyển sang orchestrator như **Kubernetes**, **Amazon ECS**, hoặc **Docker Swarm**.

## 6. Xem thêm

- [Docker Network](./network.md): DNS theo tên service hoạt động thế nào.
- [Volume và Bind Mount](./volume.md)
- [Dockerfile](./dockerfile.md): `build: .` dựa vào Dockerfile.
- [Cache](../be/cache.md) và [Queue và Event](../be/queue-event.md): Redis và message queue thường là service trong compose.
- [Background Worker](../be/background-worker.md): worker là một service riêng, dùng chung image với API nhưng khác `command`.
