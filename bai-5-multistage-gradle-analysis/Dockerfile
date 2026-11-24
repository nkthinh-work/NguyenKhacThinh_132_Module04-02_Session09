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
