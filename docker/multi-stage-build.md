[← Mục lục](./index.md)

# Multi-stage Build

## 1. Định nghĩa kỹ thuật

**Multi-stage build** là kỹ thuật dùng **nhiều chỉ thị `FROM` trong cùng một Dockerfile**. Mỗi `FROM` mở ra một **stage** độc lập. Stage sau có thể copy file chọn lọc từ stage trước bằng `COPY --from=<stage>`. **Chỉ stage cuối cùng** (hoặc stage chỉ định bằng `--target`) trở thành image kết quả. Các stage trung gian bị bỏ lại, không nằm trong image cuối.

Mục đích: tách **môi trường build** (compiler, devDependencies, công cụ) khỏi **môi trường chạy** (chỉ cần artifact đã build), giúp image nhỏ hơn, ít lỗ hổng hơn.

## 2. Giải thích đơn giản

Xây nhà thì cần giàn giáo, máy trộn bê tông, xe cẩu. Nhưng khi giao nhà cho chủ, bạn **tháo hết giàn giáo**, chỉ giao căn nhà hoàn thiện.

- Stage `build` = công trường, đầy đủ đồ nghề.
- Stage cuối = căn nhà bàn giao, chỉ có thứ cần để ở.

Không có multi-stage, bạn giao luôn cả căn nhà lẫn giàn giáo: image nặng, nhiều công cụ thừa mà kẻ tấn công có thể lợi dụng.

```
 ┌──────────── stage: build ────────────┐      ┌──────── stage: runtime ────────┐
 │ node:20 (~1 GB)                      │      │ node:20-alpine (~130 MB)       │
 │ + devDependencies (TypeScript, ...)  │      │ + dependencies production      │
 │ + source .ts                         │ ───► │ + dist/ (COPY --from=build)    │
 │ → npm run build → dist/              │      │                                │
 └──────────────────────────────────────┘      └────────────────────────────────┘
          bị bỏ đi                                      image cuối cùng
```

## 3. Ví dụ

### App Node.js + TypeScript

```dockerfile
# ---------- Stage 1: cài toàn bộ dependency và build ----------
FROM node:20-alpine AS build
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci
COPY . .
RUN npm run build          # tạo ra /app/dist

# ---------- Stage 2: chỉ dependency production ----------
FROM node:20-alpine AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci --omit=dev

# ---------- Stage 3: image chạy thật ----------
FROM node:20-alpine AS runtime
WORKDIR /app
ENV NODE_ENV=production
COPY --from=deps  /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY package.json ./
USER node
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

### App Go: image cuối gần như rỗng

Go biên dịch ra một file binary tĩnh, nên image cuối có thể là `scratch` (image trống hoàn toàn) hoặc `distroless`:

```dockerfile
FROM golang:1.23 AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 go build -o /out/app ./cmd/server

FROM gcr.io/distroless/static-debian12
COPY --from=build /out/app /app
USER nonroot
ENTRYPOINT ["/app"]
```

Image cuối chỉ vài MB, không có shell, không có package manager.

### Build một stage cụ thể

```bash
docker build --target build -t myapp:build .   # dừng ở stage build, dùng để chạy test trong CI
docker build -t myapp:1.0.0 .                  # build tới stage cuối
```

## 4. So sánh kích thước điển hình

| Cách build | Kích thước image |
|---|---|
| Single stage `node:20` + devDependencies | ~1.1 GB |
| Single stage `node:20-alpine` | ~250 MB |
| Multi-stage, runtime `node:20-alpine` | ~150 MB |
| Go multi-stage, runtime `distroless/static` | ~10 MB |

Số liệu mang tính minh họa, phụ thuộc dependency của từng dự án.

## 5. Lưu ý

- Đặt tên stage bằng `AS <tên>` thay vì dùng chỉ số (`--from=0`) để dễ đọc và không vỡ khi thêm stage.
- BuildKit tự **bỏ qua stage không được stage cuối dùng tới**, và build song song các stage độc lập.
- `COPY --from` còn nhận cả image ngoài: `COPY --from=nginx:1.27 /etc/nginx/nginx.conf ./`.
- Image không có shell (`scratch`, `distroless`) khó debug bằng `docker exec`. Dùng `docker debug` hoặc build thêm một target `debug` khi cần.

## 6. Xem thêm

- [Dockerfile](./dockerfile.md): chỉ thị cơ bản và build cache.
- [Best Practices và Bảo mật](./best-practices.md): giảm bề mặt tấn công của image.
- [Image và Container](./image-container.md)
