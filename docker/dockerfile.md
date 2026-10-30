[← Mục lục](./index.md)

# Dockerfile

## 1. Định nghĩa kỹ thuật

**Dockerfile** là file văn bản chứa **chuỗi chỉ thị (instruction)** mà Docker (BuildKit) thực thi tuần tự để **build ra một image**. Lệnh `docker build` đọc Dockerfile cùng **build context** (thư mục được gửi cho builder) và tạo image.

Mỗi chỉ thị thay đổi filesystem (`RUN`, `COPY`, `ADD`) tạo ra **một layer mới**. Các chỉ thị còn lại (`ENV`, `CMD`, `EXPOSE`...) chỉ ghi metadata.

## 2. Giải thích đơn giản

Dockerfile giống **công thức nấu ăn viết từng bước**: "lấy nồi Node 20, đổ danh sách nguyên liệu vào, nấu cho chín (cài thư viện), cho thịt vào (copy code), khi khách gọi thì dọn món ra (CMD)". Ai cầm công thức này cũng nấu ra đúng một món.

Docker còn có **build cache**: bước nào không đổi so với lần trước thì dùng lại kết quả cũ. Như nấu nồi nước dùng một lần, các lần sau chỉ cần thêm topping.

## 3. Các chỉ thị chính

| Chỉ thị | Ý nghĩa |
|---|---|
| `FROM` | Image nền. Bắt buộc, thường là dòng đầu |
| `WORKDIR` | Đặt thư mục làm việc (tự tạo nếu chưa có) |
| `COPY` | Copy file từ build context vào image |
| `ADD` | Như `COPY` nhưng tự giải nén tar và tải URL. Chỉ dùng khi thật sự cần |
| `RUN` | Chạy lệnh lúc **build** (cài package, biên dịch) |
| `ENV` | Biến môi trường, tồn tại cả lúc build và lúc chạy |
| `ARG` | Biến chỉ tồn tại lúc build, truyền bằng `--build-arg` |
| `EXPOSE` | Ghi chú cổng app lắng nghe. **Không** tự mở cổng ra ngoài |
| `USER` | Đổi user chạy các lệnh sau và chạy container |
| `CMD` | Lệnh mặc định khi **chạy** container, bị ghi đè bởi tham số `docker run` |
| `ENTRYPOINT` | Lệnh chính cố định của container, tham số `docker run` được nối vào sau |
| `HEALTHCHECK` | Lệnh kiểm tra container còn khỏe không |

### `RUN` vs `CMD` vs `ENTRYPOINT`

- `RUN` chạy **khi build**, kết quả lưu vào image.
- `CMD` và `ENTRYPOINT` chạy **khi container khởi động**.

```dockerfile
ENTRYPOINT ["python", "cli.py"]
CMD ["--help"]
```

- `docker run mycli` chạy `python cli.py --help`.
- `docker run mycli migrate` chạy `python cli.py migrate` (CMD bị thay).

### Exec form vs shell form

```dockerfile
CMD ["node", "server.js"]   # exec form: node là PID 1, nhận được SIGTERM
CMD node server.js          # shell form: chạy qua /bin/sh -c, sh là PID 1, app có thể không nhận tín hiệu
```

Luôn ưu tiên **exec form** cho `CMD` và `ENTRYPOINT`.

## 4. Ví dụ: Dockerfile cho app Node.js

```dockerfile
FROM node:20-alpine

WORKDIR /app

# 1. Copy riêng file khai báo dependency trước để tận dụng cache
COPY package.json package-lock.json ./
RUN npm ci --omit=dev

# 2. Sau đó mới copy source (thay đổi thường xuyên)
COPY . .

ENV NODE_ENV=production
EXPOSE 3000

# Không chạy bằng root
USER node

CMD ["node", "server.js"]
```

Đi kèm file `.dockerignore` để không gửi rác vào build context:

```gitignore
node_modules
.git
.env
dist
*.log
Dockerfile
```

Build và chạy:

```bash
docker build -t myapp:1.0.0 .
docker run -d -p 3000:3000 --name myapp myapp:1.0.0
```

## 5. Build cache hoạt động thế nào

Docker duyệt từ trên xuống. Với mỗi chỉ thị, nếu **chỉ thị và input không đổi** thì dùng lại layer cũ. **Một khi một bước bị miss cache, mọi bước phía sau đều build lại.**

```
 Sửa 1 dòng trong server.js:

 FROM node:20-alpine            ✔ cache
 COPY package*.json ./          ✔ cache   (package.json không đổi)
 RUN npm ci                     ✔ cache   ← tiết kiệm cả phút
 COPY . .                       ✘ rebuild (source đổi)
 CMD ...                        ✘ rebuild
```

Nếu viết `COPY . .` **trước** `RUN npm ci`, chỉ cần sửa một dòng code là `npm ci` chạy lại từ đầu. Quy tắc: **thứ ít thay đổi đặt trên, thứ hay thay đổi đặt dưới**.

## 6. Lỗi thường gặp

- **Quên `.dockerignore`**: copy cả `node_modules` của máy host (sai nền tảng) và file `.env` chứa bí mật vào image.
- **Chia `apt-get update` và `apt-get install` thành hai `RUN`**: layer `update` bị cache cũ, `install` cài bản lỗi thời. Gộp chung một `RUN` và xóa cache apt trong cùng lệnh:

  ```dockerfile
  RUN apt-get update \
   && apt-get install -y --no-install-recommends curl \
   && rm -rf /var/lib/apt/lists/*
  ```

- **Xóa file ở layer sau không làm image nhỏ đi**: file vẫn nằm trong layer trước. Muốn image gọn, dùng [Multi-stage Build](./multi-stage-build.md).
- **Truyền secret qua `ARG`/`ENV`**: giá trị lộ trong `docker history`. Dùng `RUN --mount=type=secret` (xem [Best Practices](./best-practices.md)).

## 7. Xem thêm

- [Image và Container](./image-container.md): layer là gì.
- [Multi-stage Build](./multi-stage-build.md): tách môi trường build và môi trường chạy.
- [Best Practices và Bảo mật](./best-practices.md)
- [Command Injection](../security/command-injection.md): cẩn thận khi ghép biến vào lệnh `RUN` hoặc shell trong container.
