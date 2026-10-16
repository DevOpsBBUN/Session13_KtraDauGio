# Campus Hub Base - Bài Kiểm Tra Đóng Gói Docker & Docker Compose

## Thông tin sinh viên
- **Họ và tên:** Trần Võ Hữu Tài

---

## Hướng dẫn chạy và kiểm tra hệ thống

### 1. Kiểm thử backend với Gradle
```bash
cd backend
./gradlew test
```
*(Trên Windows: `.\gradlew.bat test`)*

### 2. Đóng gói backend thành Docker Image
```bash
# Build Docker image cho backend (từ thư mục backend)
cd backend
docker build -t campus-hub-backend .

# Kiểm tra Docker Image đã tạo
docker images | grep campus-hub-backend
```

### 3. Chạy hệ thống bằng Docker Compose (Backend + MySQL)
```bash
# Tại thư mục gốc của repository:
docker compose config
docker compose up -d

# Kiểm tra trạng thái các service
docker compose ps
```

### 4. Kiểm tra sức khỏe hệ thống (Health Check)
- **URL kiểm tra:** `http://localhost:8080/actuator/health`
- **Lệnh kiểm tra:**
```bash
curl -i http://localhost:8080/actuator/health
```
- **Kết quả trả về:** HTTP 200 OK với body:
```json
{
  "status": "UP",
  "groups": [
    "liveness",
    "readiness"
  ]
}
```

---

## Minh chứng kết quả thực hiện

### Minh chứng 1: Docker Image đã build thành công
```text
REPOSITORY           TAG       IMAGE ID       CREATED          SIZE
campus-hub-backend   latest    6a9de04fc15c   Few minutes ago  131MB
```
*(Chèn ảnh chụp màn hình terminal lệnh `docker images` tại đây nếu cần)*

### Minh chứng 2: Trạng thái các service sau khi chạy Compose
```text
NAME                 IMAGE                       COMMAND                  SERVICE   STATUS                    PORTS
campus-hub-backend   campus-hub-backend:latest   "java -Djava.securit…"   backend   Up (healthy)              0.0.0.0:8080->8080/tcp
campus-hub-mysql     mysql:8.0                   "docker-entrypoint.s…"   mysql     Up (healthy)              0.0.0.0:3307->3306/tcp
```
*(Chèn ảnh chụp màn hình terminal lệnh `docker compose ps` tại đây nếu cần)*

### Minh chứng 3: Kết quả gọi endpoint /actuator/health
```http
HTTP/1.1 200 OK
Content-Type: application/vnd.spring-boot.actuator.v3+json
{"status":"UP","groups":["liveness","readiness"]}
```
*(Chèn ảnh chụp màn hình lệnh curl hoặc trình duyệt truy cập `http://localhost:8080/actuator/health` tại đây nếu cần)*

---

# Thông tin dự án gốc


Dự án được giữ ở mức nhỏ để dễ đọc và sửa đổi, nhưng đã có đủ luồng giao diện, API, xác thực, migration và kiểm thử.

## Chức năng có sẵn

- Xem, tìm kiếm và lọc bài đăng.
- Đăng ký, đăng nhập bằng HTTP Basic.
- Tạo bài đăng theo chuyên mục.
- Bình luận và bày tỏ quan tâm.
- Chủ bài đăng có thể đánh dấu bài đã hoàn tất.
- Quản trị viên có quyền quản lý trạng thái bài đăng qua API.
- Health check tại `/actuator/health`.

Tài khoản mẫu được tạo khi ứng dụng khởi động lần đầu:

| Vai trò | Tên đăng nhập | Mật khẩu |
|---|---|---|
| Sinh viên | `student` | `student123` |
| Quản trị viên | `admin` | `admin123` |

Các tài khoản này chỉ dùng để trình diễn. Nhóm cần thay đổi hoặc loại bỏ trước khi sử dụng trong bối cảnh thực tế.

## Kiến trúc

```text
React frontend
    │ /api
    ▼
Spring Boot monolith
    │ JDBC
    ▼
MySQL
```

Backend gồm năm bảng chính: `users`, `categories`, `posts`, `comments` và `reactions`. Frontend không sử dụng thư viện giao diện hay thư viện quản lý trạng thái để sinh viên dễ đọc mã nguồn.

## Chạy khi phát triển

Tạo database MySQL tên `campus_hub`, sau đó đặt các biến môi trường nếu thông tin kết nối khác giá trị mặc định:

```powershell
$env:DB_URL = "jdbc:mysql://localhost:3306/campus_hub"
$env:DB_USERNAME = "root"
$env:DB_PASSWORD = "your-password"
```

Chạy backend:

```powershell
cd backend
./gradlew.bat bootRun
```

Mở terminal khác để chạy frontend:

```powershell
cd frontend
npm install
npm run dev
```

Frontend phát triển chạy tại `http://localhost:5173` và chuyển tiếp `/api` tới backend tại cổng `8080`.

## API chính

| Phương thức | Đường dẫn | Mục đích | Xác thực |
|---|---|---|---|
| `POST` | `/api/auth/register` | Tạo tài khoản | Không |
| `GET` | `/api/auth/me` | Lấy người dùng hiện tại | Có |
| `GET` | `/api/categories` | Danh sách chuyên mục | Không |
| `GET` | `/api/posts` | Tìm và lọc bài đăng | Không |
| `GET` | `/api/posts/{id}` | Chi tiết bài đăng | Không |
| `POST` | `/api/posts` | Tạo bài đăng | Có |
| `PUT` | `/api/posts/{id}` | Sửa bài của mình | Có |
| `PATCH` | `/api/posts/{id}/status` | Đổi trạng thái | Có |
| `DELETE` | `/api/posts/{id}` | Xóa bài của mình | Có |
| `POST` | `/api/posts/{id}/comments` | Thêm bình luận | Có |
| `POST` | `/api/posts/{id}/reaction` | Bật hoặc tắt quan tâm | Có |

Các API cần xác thực sử dụng header HTTP Basic. Trong môi trường công khai, hệ thống phải được đặt sau HTTPS.

## Kiểm tra trước khi bàn giao

```powershell
cd backend
./gradlew.bat test

cd ../frontend
npm ci
npm run lint
npm run build
```

## Phạm vi cố ý để mở

Dự án nền chưa có tải tệp, thông báo thời gian thực, email, phân trang trên giao diện, lịch sử thay đổi, dashboard và quy trình kiểm duyệt hoàn chỉnh. Nhóm có thể phát triển các hướng này, sửa sâu dự án nền hoặc thay thế toàn bộ công nghệ nếu phù hợp với ý tưởng của mình.

Dự án không cung cấp sẵn Dockerfile, Docker Compose, cấu hình web server hoặc quy trình CI/CD. Phân tích hệ thống và lựa chọn toàn bộ phương án xây dựng, đóng gói, triển khai là công việc của từng nhóm trong Mini Project.
