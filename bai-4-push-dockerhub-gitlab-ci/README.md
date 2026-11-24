# Bài tập 4: Đẩy Docker Image lên Docker Hub qua GitLab CI

## Hướng dẫn cấu hình GitLab CI/CD Variables
1. Truy cập dự án trên GitLab -> **Settings -> CI/CD -> Variables -> Add variable**.
2. Thêm 2 biến bảo mật (tích chọn **Mask variable**):
   - `DOCKER_USERNAME`: Tên đăng nhập Docker Hub của bạn (ví dụ: `thinhbanto`).
   - `DOCKER_PASSWORD`: Personal Access Token (PAT) tạo từ Docker Hub (**Account Settings -> Security -> New Access Token**).
3. Khi pipeline chạy qua 3 stage (`test` -> `build` -> `dockerize`), file `.jar` từ artifacts của job `build` sẽ tự động được tải về thư mục làm việc của job `dockerize` để đóng gói và push lên `https://hub.docker.com/r/<DOCKER_USERNAME>/payment-service`.
