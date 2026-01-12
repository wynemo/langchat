# LangChat Docker 部署指南

本文档介绍如何使用 Docker Compose 一键部署 LangChat 项目。

## 环境要求

- Docker >= 20.10
- Docker Compose >= 2.0
- 内存 >= 4GB (推荐 8GB)
- 磁盘空间 >= 20GB

## 服务组件

| 服务 | 端口 | 说明 |
|------|------|------|
| MySQL | 3306 | 主数据库 |
| Redis | 6379 | 缓存服务 |
| PgVector | 5432 | 向量数据库 |
| langchat-server | 8100 | 后端 API 服务 |
| langchat-web | 80 | 前端 Web 服务 |

## 快速部署

### 1. 克隆项目

```bash
git clone https://github.com/tycoding/langchat.git
cd langchat
```

### 2. 配置环境变量

```bash
# 复制环境变量模板
cp .env.example .env

# 编辑配置 (可选，默认配置可直接使用)
vim .env
```

环境变量说明：

```bash
# MySQL 配置
MYSQL_ROOT_PASSWORD=root      # MySQL root 密码

# PgVector 配置
PGVECTOR_USER=root           # PostgreSQL 用户名
PGVECTOR_PASSWORD=root       # PostgreSQL 密码
```

### 3. 启动服务

```bash
# 构建并启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

### 4. 访问应用

- 前端地址: http://localhost
- 后端 API: http://localhost:8100
- 默认账号: admin / 123456

## 单独部署组件

### 仅启动基础服务 (数据库)

```bash
docker-compose up -d mysql redis pgvector
```

### 仅构建后端

```bash
docker-compose build langchat-server
docker-compose up -d langchat-server
```

### 仅构建前端

```bash
docker-compose build langchat-web
docker-compose up -d langchat-web
```

## 配置说明

### MySQL 初始化

`docs/langchat.sql` 会在 MySQL 首次启动时自动执行，包含：
- 创建 `langchat` 数据库
- 创建所有数据表
- 插入默认数据 (管理员账号等)

### 后端配置

后端通过环境变量配置，主要参数：

| 变量 | 默认值 | 说明 |
|------|--------|------|
| SPRING_DATASOURCE_URL | jdbc:mysql://mysql:3306/langchat | MySQL 连接地址 |
| SPRING_DATASOURCE_USERNAME | root | MySQL 用户名 |
| SPRING_DATASOURCE_PASSWORD | root | MySQL 密码 |
| SPRING_DATA_REDIS_HOST | redis | Redis 地址 |
| SPRING_DATA_REDIS_PORT | 6379 | Redis 端口 |
| JAVA_OPTS | -Xms512m -Xmx1024m | JVM 参数 |

### 前端 Nginx 配置

前端通过 Nginx 提供服务，配置文件位于 `docker/nginx/default.conf`：
- 静态资源服务
- API 代理到后端 (`/api/` -> `langchat-server:8100`)
- WebSocket/SSE 流式响应支持

### OSS 文件存储

默认使用本地存储 (local)，文件保存在 Docker 卷 `upload_data` 中。

如需使用云存储，请在管理后台配置相应的 OSS 信息。

## 常用命令

```bash
# 启动所有服务
docker-compose up -d

# 停止所有服务
docker-compose down

# 停止并删除数据卷 (会删除所有数据!)
docker-compose down -v

# 重新构建并启动
docker-compose up -d --build

# 查看服务日志
docker-compose logs -f [service_name]

# 进入容器
docker-compose exec langchat-server sh
docker-compose exec mysql mysql -uroot -proot langchat

# 重启单个服务
docker-compose restart langchat-server
```

## 数据备份

### 备份 MySQL

```bash
docker-compose exec mysql mysqldump -uroot -proot langchat > backup.sql
```

### 备份 PgVector

```bash
docker-compose exec pgvector pg_dump -U root langchat > pgvector_backup.sql
```

### 备份数据卷

```bash
# 备份所有数据卷
docker run --rm -v langchat_mysql_data:/data -v $(pwd):/backup alpine tar czf /backup/mysql_data.tar.gz /data
docker run --rm -v langchat_redis_data:/data -v $(pwd):/backup alpine tar czf /backup/redis_data.tar.gz /data
docker run --rm -v langchat_pgvector_data:/data -v $(pwd):/backup alpine tar czf /backup/pgvector_data.tar.gz /data
```

## 生产环境建议

### 1. 修改默认密码

```bash
# .env 文件
MYSQL_ROOT_PASSWORD=your_secure_password
PGVECTOR_PASSWORD=your_secure_password
```

### 2. 配置 HTTPS

建议使用外部反向代理 (如 Nginx、Traefik) 配置 SSL 证书。

### 3. 资源限制

在 `docker-compose.yml` 中添加资源限制：

```yaml
services:
  langchat-server:
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
```

### 4. 日志管理

```yaml
services:
  langchat-server:
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"
```

## 故障排查

### 服务无法启动

```bash
# 查看详细日志
docker-compose logs langchat-server

# 检查容器状态
docker-compose ps
```

### 数据库连接失败

1. 确认 MySQL 服务已启动且健康
2. 检查网络连接: `docker-compose exec langchat-server ping mysql`
3. 检查数据库是否初始化成功

### 前端无法访问 API

1. 检查后端服务是否正常运行
2. 检查 Nginx 配置中的代理设置
3. 查看浏览器控制台网络请求

## 更新升级

```bash
# 拉取最新代码
git pull

# 重新构建并启动
docker-compose up -d --build
```

## 相关文档

- [LangChat 官方文档](http://docs.langchat.cn/)
- [GitHub 仓库](https://github.com/tycoding/langchat)
- [Gitee 仓库](https://gitee.com/langchat/langchat)
