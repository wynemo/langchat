# LangChat 后端 Dockerfile
# 多阶段构建

# ==================== 构建阶段 ====================
FROM maven:3.9-eclipse-temurin-17 AS builder

WORKDIR /build

# 复制 pom 文件
COPY pom.xml .
COPY langchat-common langchat-common
COPY langchat-auth langchat-auth
COPY langchat-upms langchat-upms
COPY langchat-ai langchat-ai
COPY langchat-server langchat-server

# 构建项目
RUN mvn clean package -DskipTests -B

# ==================== 运行阶段 ====================
FROM eclipse-temurin:17-jre-alpine

LABEL maintainer="LangChat Team"
LABEL description="LangChat Backend Service"

WORKDIR /app

# 安装必要工具并设置时区
RUN apk add --no-cache tzdata curl && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

# 从构建阶段复制 jar 文件
COPY --from=builder /build/langchat-server/target/*.jar app.jar

# 创建静态文件目录
RUN mkdir -p /app/static

# 暴露端口
EXPOSE 8100

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -f http://localhost:8100/actuator/health || exit 1

# JVM 参数
ENV JAVA_OPTS="-Xms512m -Xmx1024m -XX:+UseG1GC"

# 启动命令
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar app.jar"]
