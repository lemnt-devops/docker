# Docker 基础设施服务

本地开发环境 Docker 基础设施部署，包含以下服务：

| 服务 | 版本 | 端口 | 说明 |
|------|------|------|------|
| MySQL | 8.0 | 3306 | 关系型数据库 |
| Redis | 7.2 | 6379 | 缓存 / NoSQL |
| Nacos | 3.2.1 | 8084 / 8848 / 9848 | 服务注册与配置中心 |
| Elasticsearch | 8.11.0 | 9020 | 全文检索与日志存储 |
| Logstash | 8.11.0 | 5001 (tcp) | 日志采集与传输 |
| Kibana | 8.11.0 | 5601 | 日志可视化 |
| XXL-Job | 2.4.0 | 9900 | 分布式任务调度 |

## 目录结构

```
docker/
├── mysql/            # MySQL 8.0
│   ├── docker-compose.yaml
│   ├── conf/         # 自定义配置
│   ├── data/         # 数据持久化（已 gitignore）
│   └── logs/         # 日志
├── redis/            # Redis 7.2
│   ├── docker-compose.yaml
│   ├── data/         # RDB / AOF 持久化（已 gitignore）
│   └── logs/
├── nacos/            # Nacos 3.2.1
│   ├── docker-compose.yaml
│   ├── env/          # 环境变量配置
│   └── logs/         # 日志（已 gitignore）
├── logstash/         # ELK — Logstash + ES + Kibana
│   ├── docker-compose.yaml
│   └── pipeline/     # Logstash pipeline 配置
├── xxl-job/          # XXL-Job 2.4.0
│   ├── docker-compose.yaml
│   ├── env/
│   └── logs/         # 日志（已 gitignore）
└── README.md
```

## 快速开始

### 前置要求

- Docker Engine >= 24
- Docker Compose >= 2.20
- 已创建 `ad-net` 网络（MySQL / Redis / Nacos / Elasticsearch 共用）

```bash
docker network create ad-net
```

### 启动服务

各服务独立部署，按需逐个启动：

```bash
# MySQL
cd mysql && docker compose up -d

# Redis
cd redis && docker compose up -d

# Nacos（依赖 MySQL）
cd nacos && docker compose up -d

# ELK
cd logstash && docker compose up -d

# XXL-Job（依赖 MySQL，需先创建 dsp-job 库）
cd xxl-job && docker compose up -d
```

### 停止服务

```bash
docker compose down
```

如需清理持久化数据（谨慎）：

```bash
docker compose down -v
```

## 服务明细

### MySQL

- 配置文件：`mysql/conf/`
- 数据目录：`mysql/data/`（已 gitignore）
- 默认密码：`123456`
- 字符集：`utf8mb4`，排序规则：`utf8mb4_general_ci`
- 认证方式：`mysql_native_password`

### Redis

- 密码：`123456`
- 持久化：AOF（appendonly yes）
- 数据目录：`redis/data/`（已 gitignore）

### Nacos

- 认证 key/secret：`nacos`
- JWT Token 已配置默认值
- 使用 MySQL 作为后端存储（数据库 `nacos_server` 需提前创建）
- 运行模式：单机 standalone

### ELK（Logstash + ES + Kibana）

- Elasticsearch 安全认证已禁用
- Kibana 中文界面（`zh-CN`）
- Logstash 监听 TCP 5001 端口（映射自容器 5000）
- 通过 JSON 行协议接收日志
- 日志索引格式：`{appName}-YYYY.MM.dd`
- Logstash pipeline 配置位于 `logstash/pipeline/logstash.conf`

### XXL-Job

- 访问地址：`http://localhost:9900/xxl-job-admin`
- 接入 Token：`xxl-job`
- 需提前创建 `dsp-job` 数据库
- 连接外部 MySQL（非本服务的 MySQL）

## 网络说明

MySQL、Redis、Nacos、Elasticsearch 共享 `ad-net` 网络，可通过容器名互相访问：

- `mysql8`
- `redis7`
- `nacos-server`
- `elasticsearch`

## 注意事项

1. `.gitignore` 已忽略数据目录和日志目录，`git pull` 后需自行初始化数据
2. Nacos 启动前需先在 MySQL 中创建 `nacos_server` 数据库并执行初始化 SQL
3. XXL-Job 连接的是外部 MySQL，需确保 `192.168.150.252:3306` 可达
4. 生产环境请修改所有默认密码、Token 和安全配置
5. MySQL 配置目录 `mysql/conf/` 可挂载自定义 `.cnf` 文件
