# Bài tập 1: Phân tích và khắc phục sự cố Pipeline CI/CD cơ bản

## 1. Phân tích nguyên nhân gây lỗi
File `.gitlab-ci.yml` ban đầu gặp 3 lỗi/thiếu sót nghiêm trọng khiến pipeline thất bại ngay khi khởi chạy:

1. **Thiếu khai báo `image` chứa môi trường Java/Gradle:**
   - Khi không chỉ định keyword `image`, GitLab Runner (Docker executor) sẽ sử dụng default image (thường là `ruby:2.5` hoặc một base Linux tối giản không cài sẵn JDK hay Gradle). Do đó, khi chạy `./gradlew`, hệ thống không tìm thấy trình biên dịch Java (`java: command not found`) hoặc thiếu môi trường thực thi.
2. **Sai cú pháp khai báo danh sách (mảng) trong YAML tại `stages` và `script`:**
   - Trong YAML, `stages` và `script` bắt buộc phải là một danh sách (sequence) các phần tử bắt đầu bằng dấu gạch ngang `- `.
   - File gốc viết `stages:
  build_app` và `script:
  ./gradlew ...` khiến parser hiểu giá trị là một chuỗi (String) thay vì mảng (Array), gây lỗi cú pháp pipeline ngay từ bước validate.
3. **Thiếu cấp quyền thực thi (`chmod +x ./gradlew`) (rủi ro tiềm ẩn):**
   - Nếu mã nguồn được commit từ Windows, file `gradlew` thường không có bit quyền thực thi (`+x`) trên Linux container, dễ phát sinh lỗi `Permission denied`.

## 2. File `.gitlab-ci.yml` sau khi sửa
```yaml
image: gradle:7.6-jdk17

stages:
  - build_app

build_job:
  stage: build_app
  before_script:
    - chmod +x ./gradlew
  script:
    - ./gradlew clean build -x test
```
