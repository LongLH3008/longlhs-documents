[← Mục lục](./index.md)

# Best Practices và Bảo mật

## 1. Định nghĩa kỹ thuật

**Docker best practices** là tập nguyên tắc khi viết Dockerfile và vận hành container nhằm đạt bốn mục tiêu: **image nhỏ**, **build nhanh** (tận dụng cache), **chạy ổn định** (tắt êm, healthcheck, giới hạn tài nguyên) và **an toàn** (giảm bề mặt tấn công, không chạy root, không lộ secret, quét lỗ hổng). Đây là một phần của **container security** và **supply chain security**.

## 2. Giải thích đơn giản

Đóng gói image giống **đóng vali đi máy bay**:

- **Chỉ mang thứ cần thiết** (image nhỏ): vali nhẹ, qua cửa nhanh, ít đồ để mất.
- **Đồ hay dùng xếp trên cùng** (thứ tự layer): mở ra lấy nhanh, không phải bới cả vali.
- **Không nhét giấy tờ mật vào vali ký gửi** (không để secret trong image): ai cầm vali cũng mở ra xem được.
- **Không đưa chìa khóa nhà cho nhân viên sân bay** (không chạy root): lỡ có chuyện, thiệt hại có giới hạn.

## 3. Image nhỏ và build nhanh

| Nguyên tắc | Cách làm |
|---|---|
| Chọn base image gọn | `-alpine`, `-slim`, `distroless`. Cân nhắc: Alpine dùng `musl`, đôi khi lỗi với thư viện native |
| Pin phiên bản | `node:20.17-alpine3.20`, hoặc pin digest `@sha256:...`. Tránh `latest` |
| Tận dụng cache | Copy file dependency trước, `RUN install`, rồi mới copy source |
| Có `.dockerignore` | Loại `node_modules`, `.git`, `.env`, build output |
| Multi-stage | Chỉ đưa artifact vào image cuối, xem [Multi-stage Build](./multi-stage-build.md) |
| Gộp lệnh và dọn trong cùng `RUN` | `apt-get install ... && rm -rf /var/lib/apt/lists/*` |
| Dùng cache mount | `RUN --mount=type=cache,target=/root/.npm npm ci` để cache tải package giữa các lần build |

## 4. Bảo mật

### 4.1 Không chạy bằng root

Mặc định process trong container là **root (UID 0)**. Nếu thoát được container (container escape), kẻ tấn công có thể là root trên host.

```dockerfile
# Image node có sẵn user "node"
USER node

# Image khác: tự tạo user
RUN addgroup -S app && adduser -S app -G app
USER app
```

### 4.2 Không đưa secret vào image

```dockerfile
# SAI: secret nằm vĩnh viễn trong layer và docker history
ENV NPM_TOKEN=abc123
COPY .env .

# ĐÚNG: BuildKit secret mount, chỉ tồn tại trong lúc RUN, không ghi vào layer
RUN --mount=type=secret,id=npm_token \
    NPM_TOKEN=$(cat /run/secrets/npm_token) npm ci
```

```bash
docker build --secret id=npm_token,env=NPM_TOKEN -t myapp .
```

Secret lúc **chạy** truyền qua biến môi trường của orchestrator, Docker secrets, hoặc secret manager (Vault, AWS Secrets Manager), không bake vào image.

### 4.3 Giảm quyền khi chạy

```bash
docker run \
  --read-only \
  --tmpfs /tmp \
  --cap-drop ALL \
  --security-opt no-new-privileges \
  --memory 512m --cpus 1 \
  --pids-limit 200 \
  myapp:1.0
```

| Cờ | Tác dụng |
|---|---|
| `--read-only` | Root filesystem chỉ đọc, kẻ tấn công không ghi được mã độc |
| `--tmpfs /tmp` | Chỗ ghi tạm nếu app cần |
| `--cap-drop ALL` | Bỏ mọi Linux capability, thêm lại từng cái cần bằng `--cap-add` |
| `--security-opt no-new-privileges` | Chặn leo thang quyền qua binary setuid |
| `--memory`, `--cpus` | Giới hạn tài nguyên, tránh một container ăn hết máy |
| `--pids-limit` | Chống fork bomb |

Tránh tuyệt đối khi không thật sự cần:

- `--privileged`: gần như tắt mọi cơ chế cô lập.
- Mount `/var/run/docker.sock`: tương đương root trên host.
- `--network host`, `--pid host`.

### 4.4 Quét lỗ hổng và chuỗi cung ứng

```bash
docker scout cves myapp:1.0          # quét CVE trong image
trivy image myapp:1.0                # công cụ mã nguồn mở phổ biến
```

- Chỉ dùng base image từ nguồn tin cậy (Docker Official Images, Verified Publisher).
- Rebuild image định kỳ để nhận bản vá của base image.
- Ký và xác minh image (cosign, Docker Content Trust) trong pipeline CI/CD.

## 5. Vận hành ổn định

### Tắt êm (graceful shutdown)

`docker stop` gửi `SIGTERM`. App phải bắt tín hiệu, ngừng nhận request mới, xử lý xong việc đang làm rồi thoát.

```js
process.on('SIGTERM', async () => {
  server.close();          // ngừng nhận kết nối mới
  await db.end();          // đóng kết nối DB
  process.exit(0);
});
```

Điều kiện: app phải là **PID 1** (dùng exec form `CMD ["node", "server.js"]`). Nếu app không xử lý tín hiệu của zombie process, thêm `--init` hoặc dùng `tini` làm PID 1.

### Healthcheck

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD wget -qO- http://localhost:3000/health || exit 1
```

### Log ra stdout/stderr

Không ghi log vào file trong container. Ghi ra stdout/stderr để `docker logs` và hệ thống thu log (Loki, ELK, CloudWatch) thu thập. Cấu hình log rotation cho driver `json-file`, nếu không log có thể đầy ổ đĩa:

```json
{ "log-driver": "json-file", "log-opts": { "max-size": "10m", "max-file": "3" } }
```

### Cấu hình qua biến môi trường

Cùng một image chạy cho mọi môi trường (dev, staging, prod), chỉ khác biến môi trường. Theo nguyên tắc **12-factor app**: build một lần, deploy nhiều nơi.

## 6. Checklist nhanh

- [ ] Base image gọn, đã pin phiên bản.
- [ ] Có `.dockerignore`.
- [ ] Thứ tự layer tối ưu cho cache.
- [ ] Multi-stage, image cuối không có công cụ build.
- [ ] Chạy bằng user không phải root.
- [ ] Không có secret trong image (`docker history` sạch).
- [ ] `CMD`/`ENTRYPOINT` dùng exec form, app xử lý `SIGTERM`.
- [ ] Có `HEALTHCHECK` hoặc probe của orchestrator.
- [ ] Giới hạn memory/CPU khi chạy.
- [ ] Quét CVE trong CI.

## 7. Xem thêm

- [Dockerfile](./dockerfile.md)
- [Multi-stage Build](./multi-stage-build.md)
- [Docker Network](./network.md): đừng publish cổng database ra ngoài.
- [Command Injection và Path Traversal](../security/command-injection.md): container chạy root làm hậu quả tệ hơn nhiều.
- [SSRF](../security/ssrf.md): container trên cloud có thể bị lấy credential qua metadata endpoint.
- [Rate Limit](../be/rate-limit.md): giới hạn tài nguyên ở tầng container bổ sung cho giới hạn ở tầng ứng dụng.
