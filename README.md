# Rikkei Education - Bộ Bài Tập Thực Hành GitLab CI/CD & Docker (Phần 2)

Cấu trúc thư mục tương ứng từng bài tập trong bộ PDF:
- `bai-1-fix-basic-pipeline/`: File `.gitlab-ci.yml` đã sửa lỗi cú pháp YAML + khai báo `image: gradle:7.6-jdk17` kèm `giai-thich-loi.md`.
- `bai-2-containerize-payment-service/`: File `Dockerfile` hoàn chỉnh cho `Payment Service`.
- `bai-3-gitlab-ci-postgresql/`: File `.gitlab-ci.yml` gồm 2 stages `test` (kết nối service `postgres:14-alpine`) và `build` (lưu `artifacts` 1 ngày).
- `bai-4-push-dockerhub-gitlab-ci/`: File `.gitlab-ci.yml` bổ sung stage `dockerize` sử dụng Docker-in-Docker (`docker:dind`) và biến bảo mật `$DOCKER_USERNAME`, `$DOCKER_PASSWORD` để push lên Docker Hub.
- `bai-5-multistage-gradle-analysis/`: Bảng so sánh Single-stage vs Multi-stage (`phan-tich-so-sanh.docx` và `.md`) kèm `Dockerfile` tối ưu cho `Notification Service`.
- `bai-6-cart-service-cicd/`: Bộ thiết kế hoàn chỉnh (`Dockerfile`, `.gitlab-ci.yml`, `README.md` kèm sơ đồ luồng) cho `Cart Service` dự án Tiki-clone với 3 stages `test -> build -> dockerize` gắn tag `$CI_COMMIT_SHORT_SHA`.
