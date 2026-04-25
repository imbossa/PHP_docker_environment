# 基于 Docker 的 PHP 开发环境

本项目提供一套本地 PHP 开发环境，包含 PHP-FPM、Nginx、MySQL、Redis、MongoDB 和 RabbitMQ。

[English](README.md) | 简体中文

## 环境要求

- Docker Desktop 或 Docker Engine，并支持 Docker Compose v2
- 主机端口未被占用：`80`、`443`、`9000`、`3306`、`6379`、`27017`、`5672`、`15672`
- 用于静态服务 IP 的 Docker bridge 网络

## 快速开始

1. 创建或复用外部 Docker 网络。

   ```bash
   docker network inspect my_com >/dev/null 2>&1 || \
     docker network create -d bridge --subnet 172.28.1.0/24 my_com
   ```

   如果 Docker 提示网段与已有网络冲突，可以在 `.env` 中把 `NETWORK_EXT_NAME` 改为已有网络名称，或选择新的网段并同步修改 `IPV4_PREFIX`。

2. 创建本地环境变量文件。

   ```bash
   cp .env.example .env
   ```

3. 编辑 `.env`。

   - 将 `LOCAL_WEB_PATH` 设置为需要挂载到容器 `/var/www` 的宿主机目录。
   - 为 MySQL、Redis、MongoDB、RabbitMQ 设置强密码。
   - 将 `NETWORK_EXT_NAME` 设置为外部 Docker 网络名称，例如 `my_com`。

4. 启动服务。

   ```bash
   docker compose up -d
   ```

5. 查看状态和日志。

   ```bash
   docker compose ps
   docker compose logs --tail=100
   ```

## 服务地址

| 服务 | 容器内地址 | 静态网络 IP | 主机端口 |
|------|------------|-------------|----------|
| PHP-FPM | `com_php:9000` | `172.28.1.11` | `9000` |
| Nginx | `com_nginx:80` | `172.28.1.12` | `80/443` |
| MySQL | `com_mysql:3306` | `172.28.1.13` | `3306` |
| Redis | `com_redis:6379` | `172.28.1.14` | `6379` |
| MongoDB | `com_mongo:27017` | `172.28.1.16` | `27017` |
| RabbitMQ | `com_rabbitmq:5672` | `172.28.1.20` | `5672` |

RabbitMQ 管理后台地址：`http://localhost:15672`。

## 数据与备份

持久化数据存放在 `app/*/db` 或 `app/*/data` 目录：

- MySQL：`app/mysql/db`
- Redis：`app/redis/db`
- MongoDB：`app/mongo/db`
- RabbitMQ：`app/rabbitmq/data/rabbitmq-4`

变更数据库镜像大版本前，请先备份对应的数据目录。临时备份可以放到 `app/backups/`，该目录已被 Git 忽略。

## 常用命令

```bash
# 启动或更新容器
docker compose up -d

# 停止并删除容器，保留挂载的数据目录
docker compose down

# 重新构建 PHP/Nginx 镜像
docker compose build php nginx

# 查看健康状态
docker compose ps

# 持续查看日志
docker compose logs -f --tail=100

# 校验生成后的 Compose 配置
docker compose config
```

## 故障排查

### Docker Hub EOF 或超时

如果 Docker 报错 `EOF`、`i/o timeout` 或 `failed to fetch oauth token`，通常是网络访问 Docker Hub 不稳定。网络恢复后重新执行：

```bash
docker compose pull
docker compose up -d
```

### 外部网络不存在

如果 Compose 提示外部网络不存在，请创建网络或将 `.env` 中的 `NETWORK_EXT_NAME` 改为已有兼容网络：

```bash
docker network ls
docker network inspect my_com >/dev/null 2>&1 || \
  docker network create -d bridge --subnet 172.28.1.0/24 my_com
```

### 健康检查一直不通过

查看失败服务的日志：

```bash
docker compose logs --tail=200 <service>
```

常见原因包括 `.env` 密码不正确、数据目录来自不兼容的镜像版本，或外部 Docker 网络缺失。

## 许可证

本项目使用 [MIT License](LICENSE) 许可证。
