# Bài tập 5: Phân tích tối ưu hóa Docker Image với Multi-stage Build (`Notification Service`)

## 1. Bảng so sánh Single-stage Build và Multi-stage Build

| Tiêu chí đánh giá | Single-stage Build (Fat Container) | Multi-stage Build (Builder + JRE Runtime) |
| :--- | :--- | :--- |
| **Thành phần bên trong Image** | Chứa đầy đủ hệ điều hành nền, bộ JDK đầy đủ, công cụ build Gradle, toàn bộ mã nguồn thô (`src/`), các file cấu hình build, thư mục cache `.gradle` và file `.jar`. | Chỉ chứa hệ điều hành tối giản (Alpine Linux), môi trường thực thi **JRE 17** và duy nhất file thực thi đóng gói `.jar`. |
| **Dung lượng dự kiến** | Rất lớn: **~750MB – 850MB+**. | Siêu nhẹ: **~130MB – 180MB** (giảm ~80% dung lượng). |
| **Bảo mật mã nguồn & bề mặt tấn công** | **Kém:** Toàn bộ mã nguồn (`src/`) và công cụ dòng lệnh (`gradle`, `javac`, `curl`, `bash`) tồn tại ngay trên Production. Nếu container bị xâm nhập, kẻ tấn công dễ dàng đọc mã nguồn hoặc tải mã độc. | **Cao:** Mã nguồn thô và công cụ biên dịch bị loại bỏ hoàn toàn sau Stage 1. Image Production không có trình biên dịch, giảm tối đa bề mặt tấn công (attack surface). |
| **Tốc độ Pull/Deploy** | Chậm do băng thông mạng và thời gian giải nén layer lớn khi scale pod/container. | Nhanh gấp 4–5 lần, tối ưu hóa đáng kể thời gian khởi tạo container và chi phí lưu trữ Registry. |
| **Ưu điểm** | Viết đơn giản, chỉ gồm một khối lệnh `FROM` duy nhất. | Gọn nhẹ, bảo mật cao, tách biệt rõ ràng môi trường biên dịch (Build-time) và môi trường chạy (Runtime). |
| **Nhược điểm** | Lãng phí tài nguyên lưu trữ/mạng, vi phạm nguyên tắc tối giản hóa container trên Production. | File `Dockerfile` dài hơn một chút do phải quản lý đường dẫn truyền file giữa các stage (`COPY --from=builder`). |

## 2. Giải pháp Multi-stage Dockerfile đề xuất
```dockerfile
# Stage 1: Build mã nguồn bằng môi trường Gradle + JDK 17
FROM gradle:7.6-jdk17 AS builder
WORKDIR /home/gradle/project
COPY . .
RUN chmod +x gradlew && ./gradlew clean build -x test

# Stage 2: Runtime gọn nhẹ chỉ chứa JRE 17 Alpine
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY --from=builder /home/gradle/project/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```
