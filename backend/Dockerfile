# Stage 1: Build stage with Gradle and JDK 21
FROM eclipse-temurin:21-jdk-alpine AS builder

WORKDIR /build

# Copy Gradle wrapper and configuration
COPY gradlew .
COPY gradle ./gradle
COPY build.gradle settings.gradle ./

# Fix CRLF line endings (Windows compatibility) and grant execution permission
RUN sed -i 's/\r$//' gradlew && chmod +x gradlew

# Download dependencies (caching layer)
RUN ./gradlew dependencies --no-daemon -q || true

# Copy source code and build production jar
COPY src ./src
RUN ./gradlew bootJar --no-daemon -x test

# Stage 2: Lightweight runtime stage with JRE 21
FROM eclipse-temurin:21-jre-alpine

WORKDIR /app

# Install curl for healthcheck probing
RUN apk add --no-cache curl

# Security best practice: Run application as non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser

# Copy generated executable jar from builder stage
COPY --from=builder /build/build/libs/*.jar app.jar

# Expose default application port
EXPOSE 8080

# Configure JVM options and entry point
ENTRYPOINT ["java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "app.jar"]
