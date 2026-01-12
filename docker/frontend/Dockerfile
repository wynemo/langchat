# LangChat 前端 Dockerfile
# 多阶段构建

# ==================== 构建阶段 ====================
FROM node:22-alpine AS builder

WORKDIR /build

# 安装 pnpm
RUN npm install -g pnpm

# 复制 package 文件
COPY langchat-ui/package.json langchat-ui/pnpm-lock.yaml ./

# 安装依赖
RUN pnpm install --frozen-lockfile

# 复制源代码
COPY langchat-ui .

# 构建生产版本
RUN pnpm build

# ==================== 运行阶段 ====================
FROM nginx:alpine

LABEL maintainer="LangChat Team"
LABEL description="LangChat Frontend Service"

# 设置时区
RUN apk add --no-cache tzdata && \
    cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

# 从构建阶段复制构建产物
COPY --from=builder /build/dist /usr/share/nginx/html

# 复制 nginx 配置
COPY docker/nginx/default.conf /etc/nginx/conf.d/default.conf

# 暴露端口
EXPOSE 80

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --retries=3 \
    CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
