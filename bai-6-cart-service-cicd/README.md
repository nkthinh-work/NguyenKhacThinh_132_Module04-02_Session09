# Bài tập 6: Thiết kế Pipeline CI/CD hoàn chỉnh cho Microservices (`Cart Service` – Tiki-clone)

## 1. Kiến trúc & Luồng hoạt động của Pipeline
```
[Developer Push Code (main)]
            │
            ▼
┌───────────────────────────────────────────┐
│ Stage 1: test (gradle:7.6-jdk17)          │
│ - Khởi chạy service: postgres:14-alpine   │
│ - Kết nối DB: jdbc:postgresql://postgres  │
│ - Thực thi: ./gradlew test                │
└───────────────────────────────────────────┘
            │
     (Pass?) ── No ──► [HỦY PIPELINE NGAY LẬP TỨC - Không build/push]
            │ Yes
            ▼
┌───────────────────────────────────────────┐
│ Stage 2: build (gradle:7.6-jdk17)         │
│ - Thực thi: ./gradlew build -x test       │
│ - Xuất Artifacts: build/libs/*.jar (1d)   │
└───────────────────────────────────────────┘
            │
     (Pass?) ── No ──► [HỦY PIPELINE]
            │ Yes
            ▼
┌───────────────────────────────────────────┐
│ Stage 3: dockerize (docker:dind)          │
│ - Chỉ chạy trên branch: main              │
│ - Nhận file .jar từ Artifacts của build   │
│ - Build & Push:                           │
│   $DOCKER_USERNAME/cart-service:$CI_COMMIT_SHORT_SHA │
└───────────────────────────────────────────┘
```

## 2. Cơ chế đảm bảo chất lượng & Khả năng Rollback
1. **Chặn lỗi từ gốc (Quality Gate):** Nhờ cấu trúc tuần tự `stages` kết hợp từ khóa `needs`, nếu bất kỳ test case nào kết nối PostgreSQL ở stage `test` thất bại, pipeline lập tức dừng lại (`Failed`), tuyệt đối không sinh artifacts và không đẩy image lỗi lên Registry.
2. **Định danh phiên bản theo Commit SHA (`$CI_COMMIT_SHORT_SHA`):** Thay vì dùng tag `latest` dễ gây ghi đè và khó truy vết, mỗi image được gắn tag theo mã băm ngắn của commit (ví dụ `cart-service:a1b2c3d`). Khi có sự cố trên Production, đội vận hành chỉ cần trỏ deployment về tag của commit ổn định trước đó để rollback tức thì.
3. **Bảo mật thông tin xác thực:** Toàn bộ thông tin đăng nhập Registry được lưu dưới dạng Masked Variables (`DOCKER_USERNAME`, `DOCKER_PASSWORD`) trong GitLab CI/CD Settings.
