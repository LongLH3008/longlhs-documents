[← Mục lục](./index.md)

# Docker CLI Cheatsheet

## 1. Định nghĩa kỹ thuật

**Docker CLI** (`docker`) là client dòng lệnh gửi yêu cầu tới **Docker Engine** qua REST API (mặc định qua Unix socket `/var/run/docker.sock`, hoặc named pipe trên Windows). Lệnh được tổ chức theo dạng **`docker <đối tượng> <hành động>`**: `docker container ls`, `docker image rm`, `docker volume create`... Nhiều lệnh có dạng viết tắt quen thuộc (`docker ps` = `docker container ls`).

## 2. Giải thích đơn giản

CLI là **cái điều khiển từ xa**, Docker Engine là **cái TV**. Bấm nút trên điều khiển (gõ lệnh), TV làm việc (tạo, chạy, xóa container). Các nút được nhóm theo chủ đề: nhóm nút cho image, nhóm cho container, nhóm cho mạng, nhóm cho ổ lưu trữ.

## 3. Ví dụ theo nhóm lệnh

### Image

```bash
docker pull node:20-alpine            # tải image
docker images                         # liệt kê image (= docker image ls)
docker build -t myapp:1.0 .           # build từ Dockerfile
docker build --no-cache -t myapp .    # build không dùng cache
docker tag myapp:1.0 user/myapp:1.0   # gắn thêm tên/tag
docker push user/myapp:1.0            # đẩy lên registry
docker image history myapp:1.0        # xem các layer
docker rmi myapp:1.0                  # xóa image
```

### Container

```bash
docker run -d --name app -p 3000:3000 myapp:1.0   # chạy nền, map cổng
docker run --rm -it alpine sh                     # chạy tương tác, tự xóa khi thoát
docker run -e NODE_ENV=production --env-file .env myapp
docker run --memory=512m --cpus=1 myapp           # giới hạn tài nguyên (cgroups)
docker run --restart unless-stopped myapp         # tự khởi động lại

docker ps                     # container đang chạy
docker ps -a                  # tất cả, kể cả đã dừng
docker stop app               # dừng êm (SIGTERM rồi SIGKILL sau 10s)
docker start app
docker restart app
docker rm app                 # xóa container đã dừng
docker rm -f app              # dừng và xóa luôn
```

### Debug

```bash
docker logs -f --tail 100 app        # theo dõi log
docker exec -it app sh               # mở shell trong container đang chạy
docker inspect app                   # toàn bộ cấu hình dạng JSON
docker inspect -f '{{.State.Status}}' app
docker stats                         # CPU/RAM theo thời gian thực
docker top app                       # tiến trình trong container
docker cp app:/app/log.txt ./        # copy file từ container ra host
docker diff app                      # file thay đổi so với image
docker events                        # sự kiện của engine theo thời gian thực
```

### Volume và network

```bash
docker volume ls
docker volume create pgdata
docker volume inspect pgdata
docker volume rm pgdata

docker network ls
docker network create app-net
docker network connect app-net app
docker network inspect app-net
```

### Dọn dẹp

```bash
docker system df                 # xem Docker đang chiếm bao nhiêu dung lượng
docker container prune           # xóa container đã dừng
docker image prune               # xóa image "dangling" (không tag)
docker image prune -a            # xóa mọi image không được container nào dùng
docker builder prune             # xóa build cache
docker system prune              # gộp các lệnh trên (trừ volume)
docker system prune -a --volumes # dọn sạch cả volume: CẨN THẬN, mất dữ liệu
```

### Compose

```bash
docker compose up -d
docker compose down
docker compose ps
docker compose logs -f <service>
docker compose exec <service> sh
docker compose build
docker compose pull
```

## 4. Mẹo hay dùng

```bash
# Dừng và xóa mọi container (bash)
docker rm -f $(docker ps -aq)

# Chạy nhanh một công cụ mà không cần cài
docker run --rm -v "$(pwd)":/work -w /work node:20-alpine npx prettier --write .

# Định dạng lại output cho dễ đọc
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

## 5. Xem thêm

- [Image và Container](./image-container.md): vòng đời container gắn với các lệnh `create`, `start`, `stop`, `rm`.
- [Docker Compose](./docker-compose.md)
- [Volume và Bind Mount](./volume.md)
- [Docker Network](./network.md)
