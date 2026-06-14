# ==========================================
# Stage 1: Build the Spring Boot application
# ==========================================
FROM maven:3.9-eclipse-temurin-17-alpine AS builder

WORKDIR /build

# Copy all module pom files (Maven reactor needs all modules from parent POM)
COPY pom.xml .
COPY mall-common/pom.xml mall-common/
COPY mall-mbg/pom.xml mall-mbg/
COPY mall-security/pom.xml mall-security/
COPY mall-admin/pom.xml mall-admin/
COPY mall-portal/pom.xml mall-portal/
COPY mall-search/pom.xml mall-search/
COPY mall-demo/pom.xml mall-demo/

# Pre-download dependencies (leverages Docker layer cache)
RUN mvn dependency:go-offline -Ddocker.skip=true -B 2>&1 || true

# Copy source code (only for modules we need to build)
COPY mall-common/src mall-common/src/
COPY mall-mbg/src mall-mbg/src/
COPY mall-security/src mall-security/src/
COPY mall-admin/src mall-admin/src/

# Build the application (skip tests and Docker plugin)
RUN mvn clean package -DskipTests -Ddocker.skip=true -pl mall-common,mall-mbg,mall-security,mall-admin -am -B

# ==========================================
# Stage 2: Runtime with minimal JRE
# ==========================================
FROM eclipse-temurin:17-jre-alpine AS runtime

# Create non-root user
RUN addgroup -g 1000 appgroup && \
    adduser -u 1000 -G appgroup -D appuser

WORKDIR /app

# Copy the built jar from Stage 1
COPY --from=builder /build/mall-admin/target/mall-admin-*.jar app.jar

# Switch to non-root user
USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
    CMD wget -qO- http://localhost:8080/actuator/health || exit 1

ENTRYPOINT ["java", "-jar", "app.jar"]
