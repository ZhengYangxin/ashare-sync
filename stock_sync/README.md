# Ashare-sync: A股数据同步系统

> Docker 一键部署，无需下载源码，保护你的代码隐私

## 特性

- **零源码部署**: 只需 Docker，一键启动完整系统
- **全自动部署**: 一条命令启动 MySQL + 调度器 + Web API + 前端
- **24小时数据同步**: 自动同步 Tushare 行情、资金流向、涨停数据等
- **实时监控**: Web 界面查看同步状态和日志

## 系统架构

```
┌──────────────────────────────────────────────────┐
│                   Nginx (端口 8080)               │
│            前端静态文件 + API 反向代理             │
└─────────────────────────┬────────────────────────┘
                          │
        ┌─────────────────┴─────────────────┐
        │                                   │
        ▼                                   ▼
┌───────────────────┐            ┌───────────────────┐
│ Scheduler (调度器) │            │   Backend (API)   │
│  - APScheduler    │            │  - Flask Web UI   │
│  - 24+ 数据抓取器  │            │  - 任务管理接口    │
└─────────┬─────────┘            └─────────┬─────────┘
          │                                 │
          └─────────────┬───────────────────┘
                        │
                        ▼
              ┌───────────────────┐
              │     MySQL 8.4     │
              │   30+ 数据表       │
              └───────────────────┘
```

## 快速开始

### 方式一：从 Docker Hub 部署（推荐）

无需下载源码，一条命令启动：

```bash
# 1. 创建目录
mkdir -p ashare-sync && cd ashare-sync

# 2. 下载配置文件
curl -O https://raw.githubusercontent.com/ZhengYangxin/ashare-sync/main/stock_sync/docker-compose.hub.yml
curl -O https://raw.githubusercontent.com/ZhengYangxin/ashare-sync/main/stock_sync/.env.example
mv .env.example .env

# 3. 启动所有服务（数据库自动初始化）
docker compose -f docker-compose.hub.yml up -d

# 4. 访问 http://localhost:8080 → 数据源配置 填写 Token
```

### 方式二：本地源码部署（开发测试用）

如果你需要修改代码或定制开发：

```bash
# 1. 克隆源码
git clone https://github.com/ZhengYangxin/SdShare.git
cd SdShare/stock_sync

# 2. 复制配置
cp .env.example .env

# 3. 启动所有服务（从本地源码构建，数据库自动初始化）
docker compose up -d

# 4. 访问 http://localhost:8080 → 数据源配置 填写 Token
```

> **注意**: 本地源码部署会从本地代码构建 Docker 镜像，适合开发测试。
> 正式部署推荐使用方式一，从 Docker Hub 拉取预构建镜像。

### 访问服务

| 服务 | 地址 |
|------|------|
| Web 界面 | http://localhost:8080 |
| phpMyAdmin | http://localhost:8090 |

## 配置说明

`.env` 文件所有字段均为可选，可以直接留空使用。
通过 Web 界面配置数据源更方便。

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `DB_PASSWORD` | 空（root无密码） | 生产环境建议设置 |
| `NGINX_PORT` | 8080 | Web 端口 |
| `PMA_PORT` | 8090 | phpMyAdmin 端口 |

> Tushare Token 通过 Web 界面「数据源配置」填写（不在 .env 文件中）。如需在 .env 中预注入，请设置 `TUSHARE_TOKEN_1` 和 `TUSHARE_ACTIVE` 环境变量。

### 配置 Tushare Token

1. 启动服务后访问 http://localhost:8080
2. 点击「数据源配置」填写 Token
3. 保存后自动生效

## 常用命令

```bash
# 查看所有服务状态
docker compose -f docker-compose.hub.yml ps

# 查看实时日志
docker compose -f docker-compose.hub.yml logs -f

# 查看特定服务日志
docker compose -f docker-compose.hub.yml logs -f backend

# 停止所有服务
docker compose -f docker-compose.hub.yml down

# 重启所有服务
docker compose -f docker-compose.hub.yml restart

# 清理所有数据（重新开始）
docker compose -f docker-compose.hub.yml down -v
docker compose -f docker-compose.hub.yml up -d
```

## 数据说明

### 同步的数据

- **行情数据**: 日线行情、技术指标因子
- **资金流向**: 龙虎榜、个股资金流向
- **涨停数据**: 实时涨停池、连板统计、题材概念
- **指数数据**: 同花顺指数、成分股

### 同步频率

| 数据类型 | 同步频率 | 说明 |
|----------|----------|------|
| 实时行情 | 10秒 | 交易时间内 |
| 涨停池 | 5秒 | 交易时间内 |
| 日线数据 | 收盘后 | 16:00-22:00 |
| 资金流向 | 日更 | 收盘后 |

## 故障排查

### 服务启动失败

```bash
# 1. 检查容器状态
docker-compose ps

# 2. 查看错误日志
docker-compose logs backend --tail=100

# 3. 检查端口占用
netstat -an | grep 8080
```

### 数据库连接失败

```bash
# 1. 等待 MySQL 就绪（约10秒）
docker-compose logs mysql

# 2. 检查健康状态
docker-compose ps mysql
```

### 数据不同步

```bash
# 1. 检查调度器日志
docker-compose logs scheduler --tail=50

# 2. 验证 Token 配置
docker exec ashare_sync_scheduler env | grep TUSHARE
```

## Docker Hub 镜像

| 镜像 | 说明 | 大小 |
|------|------|------|
| `zhengyancy/ashare-sync:backend` | 后端服务（调度器 + API） | 889MB |
| `zhengyancy/ashare-sync:frontend` | 前端服务（Nginx） | 26.5MB |

> 版本标签：`backend-latest` / `backend-vX.Y.Z` / `frontend-latest` / `frontend-vX.Y.Z`
> 镜像发布规范详见 [docs/RELEASE_SPEC.md](docs/RELEASE_SPEC.md)

## License

MIT License
