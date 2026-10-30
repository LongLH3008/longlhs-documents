[← Mục lục](./index.md)

# Docker là gì

## 1. Định nghĩa kỹ thuật

**Docker** là nền tảng **containerization**: đóng gói ứng dụng cùng toàn bộ phụ thuộc (runtime, thư viện, file cấu hình) thành một **image**, rồi chạy image đó thành **container**, một tiến trình được **cô lập** trên cùng kernel của máy chủ.

Sự cô lập dựa trên hai tính năng của Linux kernel:

| Tính năng | Vai trò |
|---|---|
| **Namespaces** | Cô lập *những gì tiến trình nhìn thấy*: PID, network, mount (filesystem), user, hostname (UTS), IPC |
| **cgroups** | Giới hạn *tài nguyên tiến trình được dùng*: CPU, RAM, I/O, số tiến trình |

Kèm theo đó là **union filesystem** (OverlayFS) để xếp chồng các layer của image.

Các thành phần chính:

- **Docker Engine** (`dockerd`): daemon chạy nền, quản lý image, container, network, volume.
- **Docker CLI** (`docker`): công cụ dòng lệnh, gửi yêu cầu tới daemon qua REST API.
- **containerd / runc**: tầng runtime thực sự tạo container theo chuẩn **OCI**.
- **Registry** (Docker Hub, GHCR, ECR...): kho lưu và phân phối image.

## 2. Giải thích đơn giản

Hình dung **container vận chuyển hàng hóa**. Trước khi có container tiêu chuẩn, mỗi loại hàng cần cách bốc xếp riêng. Có container rồi thì tàu, cảng, xe tải chỉ cần biết "bốc một cái hộp kích thước chuẩn", không cần quan tâm bên trong là gì.

Docker làm điều tương tự cho phần mềm. Thay vì câu quen thuộc "*máy em chạy được mà*", bạn gửi đi cả cái hộp chứa app **và môi trường của nó**. Máy nào có Docker là chạy y hệt.

### Container khác máy ảo (VM) thế nào?

```
        Máy ảo (VM)                        Container
 ┌──────┐ ┌──────┐ ┌──────┐       ┌──────┐ ┌──────┐ ┌──────┐
 │ App A│ │ App B│ │ App C│       │ App A│ │ App B│ │ App C│
 │ Libs │ │ Libs │ │ Libs │       │ Libs │ │ Libs │ │ Libs │
 │Guest │ │Guest │ │Guest │       └──────┘ └──────┘ └──────┘
 │  OS  │ │  OS  │ │  OS  │       ┌──────────────────────────┐
 └──────┘ └──────┘ └──────┘       │     Docker Engine        │
 ┌──────────────────────────┐     ├──────────────────────────┤
 │       Hypervisor         │     │   Host OS (1 kernel)     │
 ├──────────────────────────┤     ├──────────────────────────┤
 │       Host OS / HW       │     │        Hardware          │
 └──────────────────────────┘     └──────────────────────────┘
```

- **VM** giống **xây nhiều căn nhà riêng**: mỗi nhà có móng, điện nước riêng (guest OS riêng). Cô lập mạnh nhưng nặng, khởi động mất cả phút.
- **Container** giống **các căn hộ trong một chung cư**: dùng chung móng và hệ thống điện nước (kernel), mỗi căn có cửa khóa riêng. Nhẹ, khởi động trong vài trăm mili giây.

| | VM | Container |
|---|---|---|
| Đơn vị cô lập | Cả hệ điều hành | Tiến trình |
| Kernel | Riêng cho từng VM | Dùng chung kernel host |
| Kích thước | Vài GB | Vài MB đến vài trăm MB |
| Khởi động | Phút | Mili giây đến giây |
| Mức cô lập | Mạnh | Yếu hơn (chung kernel) |

> Trên Windows và macOS, Docker Desktop chạy một **VM Linux nhỏ** ở bên dưới, vì container Linux cần kernel Linux.

## 3. Ví dụ

Chạy thử một web server Nginx mà không cần cài gì lên máy:

```bash
docker run -d --name web -p 8080:80 nginx:1.27-alpine
```

- `docker run`: tạo và chạy container từ image.
- `-d`: chạy nền (detached).
- `--name web`: đặt tên container.
- `-p 8080:80`: map cổng 8080 của máy bạn vào cổng 80 trong container.
- `nginx:1.27-alpine`: image `nginx`, tag `1.27-alpine`. Nếu máy chưa có, Docker tự kéo từ Docker Hub.

Mở `http://localhost:8080` là thấy trang chào của Nginx. Dọn dẹp:

```bash
docker stop web && docker rm web
```

### Tình huống thực tế

Team có 5 người, dự án cần Node 20, PostgreSQL 16, Redis 7. Không có Docker, mỗi người tự cài và lệch phiên bản. Có Docker, cả team chạy một lệnh `docker compose up` (xem [Docker Compose](./docker-compose.md)) là có đúng môi trường giống nhau, giống cả môi trường CI và production.

## 4. Docker giải quyết gì, không giải quyết gì

**Giải quyết:**

- Môi trường nhất quán giữa dev, CI, staging, production.
- Triển khai nhanh, rollback bằng cách chạy lại image cũ.
- Chạy nhiều phiên bản phần mềm song song mà không xung đột.
- Nền tảng cho orchestration (Kubernetes, ECS) để scale ngang.

**Không giải quyết:**

- Không tự làm app nhanh hơn hay an toàn hơn.
- Không thay thế VM khi cần cô lập mạnh (multi-tenant không tin cậy).
- Dữ liệu trong container **mất khi xóa container** nếu không dùng [volume](./volume.md).

## 5. Xem thêm

- [Image và Container](./image-container.md): hai khái niệm trung tâm của Docker.
- [Dockerfile](./dockerfile.md): cách tự đóng gói ứng dụng của mình.
- [Background Worker](../be/background-worker.md): worker thường chạy thành container riêng, scale độc lập với web server.
