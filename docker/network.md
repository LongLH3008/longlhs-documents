[← Mục lục](./index.md)

# Docker Network

## 1. Định nghĩa kỹ thuật

**Docker network** là lớp mạng ảo do Docker tạo ra (dựa trên **network namespace**, **virtual ethernet (veth)**, **Linux bridge** và **iptables/nftables**) để các container giao tiếp với nhau và với bên ngoài. Mỗi container có network namespace riêng: card mạng, IP, bảng định tuyến riêng.

Các **network driver** chính:

| Driver | Mô tả |
|---|---|
| `bridge` | Mặc định. Mạng riêng trên một host, container ra ngoài qua NAT |
| `host` | Container dùng thẳng network stack của host, không cô lập mạng |
| `none` | Không có mạng, chỉ có loopback |
| `overlay` | Mạng trải trên nhiều host (Docker Swarm) |
| `macvlan` | Container có địa chỉ MAC riêng, xuất hiện như một thiết bị thật trong LAN |

## 2. Giải thích đơn giản

Hình dung mỗi **network bridge là một tòa nhà văn phòng có tổng đài nội bộ**:

- Các container trong cùng tòa nhà **gọi nhau bằng tên** (`db`, `redis`) nhờ tổng đài DNS nội bộ của Docker, không cần biết số máy lẻ (IP).
- Muốn người ngoài gọi vào, phải **mở đường dây từ cổng tòa nhà** (port publishing `-p`).
- Container ở hai tòa nhà khác nhau (hai network khác nhau) **không gọi được cho nhau** trừ khi được nối vào cả hai.

```
                     Internet / máy host
                            │  -p 8080:80
 ┌──────────────────── network: app-net (bridge) ──────────────────┐
 │                                                                 │
 │ ┌─────────┐ http://api:3000 ┌─────────┐  db:5432  ┌──────────┐  │
 │ │   web   │ ──────────────► │   api   │ ────────► │    db    │  │
 │ │ (nginx) │                 │ (node)  │           │(postgres)│  │
 │ └─────────┘                 └─────────┘           └──────────┘  │
 │                                                                 │
 └─────────────────────────────────────────────────────────────────┘
       Embedded DNS (127.0.0.11) phân giải tên container → IP
```

## 3. Ví dụ

### Tạo network riêng và gọi nhau bằng tên

```bash
docker network create app-net

docker run -d --name db  --network app-net -e POSTGRES_PASSWORD=secret postgres:16
docker run -d --name api --network app-net -e DATABASE_URL=postgres://postgres:secret@db:5432/postgres myapi:1.0
docker run -d --name web --network app-net -p 8080:80 nginx:1.27-alpine
```

Trong container `api`, host `db` tự phân giải thành IP của container `db`. Chỉ `web` được publish ra ngoài, còn `db` không ai bên ngoài kết nối được.

Kiểm tra:

```bash
docker exec api ping -c 1 db
docker network inspect app-net
```

### Cổng: `EXPOSE` vs `-p`

```bash
docker run -p 8080:80 nginx            # host:8080 → container:80, mở trên mọi interface
docker run -p 127.0.0.1:8080:80 nginx  # chỉ máy host truy cập được
docker run -P nginx                    # publish mọi cổng EXPOSE ra cổng ngẫu nhiên
```

- `EXPOSE` trong Dockerfile chỉ là **ghi chú**.
- `-p` mới thật sự **mở cổng ra host**.
- Container **cùng network** gọi nhau bằng cổng nội bộ (`db:5432`), **không cần** `-p`.

### Từ container gọi ra dịch vụ trên máy host

```bash
# Docker Desktop: có sẵn tên host.docker.internal
# Linux: thêm thủ công
docker run --add-host=host.docker.internal:host-gateway myapp
```

## 4. Bridge mặc định vs bridge tự tạo

| | `bridge` mặc định | Bridge tự tạo (`docker network create`) |
|---|---|---|
| DNS theo tên container | Không | **Có** |
| Cô lập giữa các nhóm app | Mọi container chung một mạng | Mỗi app một mạng |
| Gắn/gỡ khi đang chạy | Không | Có (`docker network connect`) |

**Luôn tạo network riêng** cho từng ứng dụng. [Docker Compose](./docker-compose.md) tự làm việc này.

## 5. Lưu ý bảo mật

- **`-p 5432:5432` cho database là mở DB ra cả thế giới** (nếu máy có IP public). Trên Linux, Docker ghi rule iptables trực tiếp nên **có thể vượt qua firewall** như `ufw`. Bind vào `127.0.0.1` hoặc không publish.
- `--network host` bỏ hẳn cô lập mạng, chỉ dùng khi thật sự cần hiệu năng hoặc lắng nghe nhiều cổng.
- Tách network: frontend và database không nên chung một mạng nếu frontend không cần gọi DB trực tiếp.

## 6. Xem thêm

- [Docker Compose](./docker-compose.md): network được tạo tự động cho cả stack.
- [SSRF](../security/ssrf.md): container trong cùng network là mục tiêu hấp dẫn khi app bị SSRF.
- [CORS](../security/cors.md): khác biệt giữa "gọi được qua mạng" và "trình duyệt cho phép gọi".
- [Edge và CDN](../be/edge-cdn.md)
