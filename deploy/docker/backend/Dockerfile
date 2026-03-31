# ============================================================
# linkwork-backend  —  三阶段多模块构建
# Build context: LinkWork/ 根目录 (需同时访问 linkwork-server/ 和 backend/)
# ============================================================

# ------ Stage 1: 编译 linkwork-server（9 子模块 → 本地 Maven 仓库） ------
FROM maven:3.9-eclipse-temurin-21 AS server-builder
WORKDIR /build/linkwork-server

# 先复制 POM 以利用 Docker layer cache 缓存依赖下载
COPY linkwork-server/pom.xml .
COPY linkwork-server/linkwork-bom linkwork-bom
COPY linkwork-server/linkwork-sandbox-core linkwork-sandbox-core
COPY linkwork-server/linkwork-skill-core linkwork-skill-core
COPY linkwork-server/linkwork-skill-starter linkwork-skill-starter
COPY linkwork-server/linkwork-storage-core linkwork-storage-core
COPY linkwork-server/linkwork-storage-starter linkwork-storage-starter
COPY linkwork-server/linkwork-k8s-starter linkwork-k8s-starter
COPY linkwork-server/linkwork-mcp-core linkwork-mcp-core
COPY linkwork-server/linkwork-mcp-starter linkwork-mcp-starter

RUN mvn install -DskipTests -B -q

# ------ Stage 2: 编译 linkwork-backend ------
FROM maven:3.9-eclipse-temurin-21 AS backend-builder
COPY --from=server-builder /root/.m2/repository /root/.m2/repository
WORKDIR /build/backend

COPY backend/pom.xml .
RUN mvn dependency:go-offline -B -q || true

COPY backend/src src
RUN mvn package -DskipTests -B -q

# ------ Stage 3: 运行时 ------
FROM eclipse-temurin:21-jre-alpine
RUN apk add --no-cache curl tini
WORKDIR /app

COPY --from=backend-builder /build/backend/target/*.jar app.jar

EXPOSE 8081
ENTRYPOINT ["tini", "--"]
CMD ["java", \
     "-XX:+UseG1GC", \
     "-XX:MaxRAMPercentage=75.0", \
     "-Djava.security.egd=file:/dev/./urandom", \
     "-jar", "app.jar"]
