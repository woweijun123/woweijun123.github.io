---
{"dg-publish":true,"permalink":"/Work/Tools/Gareway/Nginx + APISIX + Hyperf/","title":"Nginx + APISIX + Hyperf","tags":["flashcards"],"noteIcon":"","created":"2026-03-29T22:06:43.000+08:00","updated":"2026-03-31T09:54:45.058+08:00","dg-note-properties":{"title":"Nginx + APISIX + Hyperf","tags":["flashcards"],"reference linking":null}}
---

## 整体架构
```
┌─────────────────────────────────────────────────────────────┐
│                      Docker Compose                         │
├─────────────────────────────────────────────────────────────┤
│  ┌──────────┐    ┌─────────────────────┐    ┌────────────┐  │
│  │  Nginx   │───▶│      APISIX         │───▶│  Hyperf    │  │
│  │ (边缘层)  │    │  + etcd + Dashboard │    │  (应用层)   │  │
│  └──────────┘    └─────────────────────┘    └────────────┘  │
└─────────────────────────────────────────────────────────────┘
```
- **Nginx**：SSL 卸载、静态资源、连接缓冲、基础限流
- **APISIX**：动态路由、精细化限流、认证鉴权、熔断
- **etcd**：APISIX 配置存储
- **APISIX Dashboard**：可视化管理界面
- **Hyperf**：业务逻辑处理（Swoole 协程）
## 目录结构
```bash
/opt/apisix-hyperf/
├── docker-compose.yml
├── .env
├── nginx/
│   ├── nginx.conf
│   └── conf.d/
│       └── default.conf
├── apisix/
│   └── config.yaml
├── apisix-dashboard/
│   └── conf.yaml
├── hyperf/
│   ├── Dockerfile
│   └── (Hyperf 项目代码)
├── etcd/
│   └── data/               # etcd 数据持久化
├── prometheus/
│   └── prometheus.yml      # 可选，监控配置
└── grafana/                 # 可选，监控可视化
```
## 核心配置文件
### 环境变量文件 `.env`
```bash
# 网络配置
NETWORK_SUBNET=172.20.0.0/24

# APISIX 配置
APISIX_IMAGE_TAG=3.12.0-debian
APISIX_ADMIN_KEY=edd1c9f034335f136f87ad84b625c8f1
# 生产环境务必修改 Admin Key
# APISIX_ADMIN_KEY=your-secure-key-here

# etcd 配置
ETCD_VERSION=v3.5.21

# Hyperf 配置
HYPERF_PORT=9501
HYPERF_WORKER_NUM=4

# Nginx 配置
NGINX_HTTP_PORT=80
NGINX_HTTPS_PORT=443
```
### 主编排文件 `docker-compose.yml`
```yaml
version: '3.8'

networks:
  apisix-hyperf-network:
    driver: bridge
    ipam:
      config:
        - subnet: ${NETWORK_SUBNET:-172.20.0.0/24}

volumes:
  etcd_data:
    driver: local
  hyperf_logs:
    driver: local

services:
  # ==================== 1. etcd 配置中心 ====================
  etcd:
    image: quay.io/coreos/etcd:${ETCD_VERSION:-v3.5.21}
    container_name: apisix-etcd
    restart: unless-stopped
    environment:
      ALLOW_NONE_AUTHENTICATION: "yes"
      ETCD_ENABLE_V2: "true"
      ETCD_ADVERTISE_CLIENT_URLS: "http://etcd:2379"
      ETCD_LISTEN_CLIENT_URLS: "http://0.0.0.0:2379"
      ETCD_DATA_DIR: "/etcd_data"
    volumes:
      - etcd_data:/etcd_data
    networks:
      apisix-hyperf-network:
        ipv4_address: 172.20.0.2
    healthcheck:
      test: ["CMD", "etcdctl", "endpoint", "health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          memory: 512M
          cpus: '0.5'
        reservations:
          memory: 256M
          cpus: '0.25'

  # ==================== 2. APISIX 网关 ====================
  apisix:
    image: apache/apisix:${APISIX_IMAGE_TAG:-3.12.0-debian}
    container_name: apisix-gateway
    restart: unless-stopped
    depends_on:
      etcd:
        condition: service_healthy
    volumes:
      - ./apisix/config.yaml:/usr/local/apisix/conf/config.yaml:ro
    ports:
      - "9080:9080"   # 数据面 HTTP 端口
      - "9180:9180"   # Admin API 端口
      - "9443:9443"   # 数据面 HTTPS 端口
      - "9091:9091"   # Prometheus 指标端口
    networks:
      apisix-hyperf-network:
        ipv4_address: 172.20.0.3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9080/"]
      interval: 30s
      timeout: 10s
      retries: 5
	environment:  
	  - APISIX_DEPLOYMENT_ETCD_HOST=["http://apisix-etcd:2379"]
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '1'
        reservations:
          memory: 512M
          cpus: '0.5'

  # ==================== 3. Hyperf 应用 ====================
  hyperf:
    build: ./hyperf
    container_name: hyperf-app
    restart: unless-stopped
    environment:
      - PORT=${HYPERF_PORT:-9501}
      - APP_ENV=prod
      - SCAN_CACHEABLE=true
    volumes:
      - ./hyperf:/var/www/hyperf
      - hyperf_logs:/var/www/hyperf/runtime
    expose:
      - "${HYPERF_PORT:-9501}"
    networks:
      apisix-hyperf-network:
        ipv4_address: 172.20.0.5
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:${HYPERF_PORT:-9501}/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          memory: 2G
          cpus: '2'
        reservations:
          memory: 1G
          cpus: '1'

  # ==================== 4. Nginx 边缘层 ====================
  nginx:
    image: nginx:1.25-alpine
    container_name: nginx-edge
    restart: unless-stopped
    depends_on:
      - apisix
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/conf.d:/etc/nginx/conf.d:ro
      - ./static:/var/www/static:ro           # 静态资源目录
      - ./logs/nginx:/var/log/nginx
    ports:
      - "${NGINX_HTTP_PORT:-80}:80"
      - "${NGINX_HTTPS_PORT:-443}:443"
    networks:
      apisix-hyperf-network:
        ipv4_address: 172.20.0.6
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: '0.5'
```
### APISIX 配置 `apisix/config.yaml`
```yaml
# =============================================================================
# APISIX 基础核心配置
# =============================================================================
apisix:
  node_listen: 9080              # 网关对外提供服务的 HTTP 监听端口
  enable_admin: true             # 是否开启 Admin API (Dashboard 和命令行管理依赖此项)
  enable_heartbeat: true         # 开启节点心跳检测
  show_instance_id: true         # 在响应头或日志中显示实例 ID (生产环境建议开启，方便多节点排障)
  admin_listen: "0.0.0.0:9180"   # 移到 apisix 块下
# =============================================================================
# 部署与数据存储配置 (Control Plane)
# =============================================================================
deployment:
  role: traditional              # 部署模式：traditional (传统模式，控制面与数据面合一)
  role_traditional:
    config_provider: etcd        # 配置来源：使用 etcd 作为存储
  etcd:
    host:
      - "http://apisix-etcd:2379" # etcd 服务地址 (必须与 Docker Compose 中的服务名一致)
    prefix: "/apisix"            # APISIX 在 etcd 中存储数据的根路径前缀
    timeout: 30                  # 访问 etcd 的超时时间 (单位：秒)

  # Admin API 管理接口的安全配置
  admin:
    admin_key:
      - name: "admin"
        # 从环境变量读取
        key: ${{ADMIN_KEY}}
        role: admin              # 该 Key 的角色权限：admin
    enable_admin_cors: true      # 开启 Admin API 的跨域支持 (方便浏览器直接调用)
    # 启用内嵌 APISIX Dashboard
    enable_admin_ui: true
    allow_admin:
      - 0.0.0.0/0                # 允许调用 Admin API 的 IP 范围 (0.0.0.0/0 表示不限制，生产环境建议限制为内网)

# =============================================================================
# 底层 Nginx 性能与日志优化 (Data Plane)
# =============================================================================
nginx_config:
  worker_processes: auto         # 自动根据 CPU 核心数启动工作进程
  worker_rlimit_nofile: 65535    # 单个进程可打开的最大文件句柄数 (影响最大并发连接)
  event:
    worker_connections: 10240    # 每个进程的最大连接数
  http:
    enable_access_log: true      # 开启访问日志
    # 自定义 JSON 格式日志：包含请求 ID、IP、状态码、响应时间、上游耗时等关键指标
    access_log_format: '{"time_local":"$time_local","remote_addr":"$remote_addr","request_id":"$request_id","request":"$request","status":$status,"body_bytes_sent":$body_bytes_sent,"request_time":$request_time,"upstream_response_time":"$upstream_response_time","http_referer":"$http_referer","http_user_agent":"$http_user_agent"}'
    keepalive_timeout: 60s       # HTTP 长连接超时时间
    client_header_timeout: 60s   # 读取客户端 Header 的超时时间
    client_body_timeout: 60s     # 读取客户端 Body 的超时时间
    send_timeout: 60s            # 发送响应给客户端的超时时间
    gzip: on                     # 开启 Gzip 压缩，减少传输带宽
    gzip_min_length: 1024        # 超过 1KB 的响应才进行压缩
    gzip_types: text/plain text/css application/json application/javascript application/xml # 开启压缩的 MIME 类型

# =============================================================================
# 插件启用列表 (按需加载，提升性能)
# =============================================================================
plugins:
  # --- 监控与可观测性 (Monitoring) ---
  - prometheus              # 业务指标监控 (导出 QPS、耗时、状态码等数据给 Prometheus/Grafana)
  - request-id              # 链路追踪 ID (为每个请求生成唯一 ID，便于全链路日志追踪)
  - file-logger             # 本地文件日志 (将请求详细记录到指定磁盘文件)
  - http-logger             # HTTP 远程日志 (将日志通过 HTTP 协议异步发送给 ELK 或审计系统)

  # --- 流量治理与性能 (Traffic Control) ---
  - limit-req               # 限速插件 (基于漏桶算法，平滑限制请求速率，防止突发流量)
  - limit-conn              # 限制并发连接数 (防止单个客户端占用过多并发连接)
  - limit-count             # 固定窗口限流 (限制单位时间内的请求总数，如 1 分钟 100 次)
  - api-breaker             # 熔断器 (当后端持续报错或超时，自动切断流量保护后端)
  - proxy-cache             # 接口缓存 (在网关层缓存 GET 请求结果，极大降低后端压力)
  - traffic-split           # 流量拆分 (支持灰度发布、蓝绿部署、A/B 测试)

  # --- 认证与授权 (Auth) ---
  - key-auth                # API Key 认证 (基于简单的 apikey Header 进行身份识别)
  - jwt-auth                # JWT 令牌认证 (标准的 Token 验证，适用于 APP/H5 前后端分离)
  - cors                    # 跨域配置 (处理 OPTIONS 预检请求并注入浏览器跨域头)

  # --- 安全防护 (Security) ---
  - ip-restriction          # IP 黑白名单 (限制特定 IP 或 CIDR 网段的访问权限)
  - uri-blocker             # 路径拦截 (基于正则过滤恶意扫描路径，如 .git/config 等)
  - ua-restriction          # User-Agent 拦截 (屏蔽爬虫、特定浏览器或恶意工具)

  # --- 请求/响应处理 (Transformation) ---
  - proxy-rewrite           # 请求改写 (在转发给后端前修改 URI、Host、Headers)
  - response-rewrite        # 响应改写 (在返回给客户端前修改状态码、Body 内容或 Headers)

  # --- 扩展开发 (Extensibility) ---
  - serverless-pre-function # 自定义前置脚本 (允许在请求处理最早期运行一段自定义的 Lua 代码)

# =============================================================================
# 插件特定属性配置
# =============================================================================
plugin_attr:
  prometheus:
    export_addr:
      ip: "0.0.0.0"              # Prometheus 抓取数据的监听 IP
      port: 9091                 # Prometheus 抓取数据的独立端口 (建议与业务端口 9080 分离)
```
### Nginx 配置
#### 主配置 `nginx/nginx.conf`
```nginx
user nginx;
worker_processes auto;
worker_rlimit_nofile 65535;

error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 10240;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    log_format json escape=json '{'
        '"time_local":"$time_local",'
        '"remote_addr":"$remote_addr",'
        '"remote_user":"$remote_user",'
        '"request":"$request",'
        '"status":$status,'
        '"body_bytes_sent":$body_bytes_sent,'
        '"request_time":$request_time,'
        '"http_referrer":"$http_referer",'
        '"http_user_agent":"$http_user_agent",'
        '"http_x_forwarded_for":"$http_x_forwarded_for"'
    '}';

    access_log /var/log/nginx/access.log json;

    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    client_max_body_size 100M;

    # Gzip 压缩
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/json application/javascript application/xml+rss application/rss+xml;

    include /etc/nginx/conf.d/*.conf;
}
```
#### 站点配置 `nginx/conf.d/default.conf`
```nginx
# 上游指向 APISIX
upstream apisix_gateway {
    server apisix-gateway:9080
    keepalive 64;
}

server {
    listen 80;
    server_name api.your-domain.com;

    # 健康检查端点
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }

    # 静态资源直接处理
    location /static/ {
        alias /var/www/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # 所有动态请求转发到 APISIX
    location / {
        proxy_pass http://apisix_gateway;

        # 传递真实客户端信息
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # 启用 HTTP 长连接
        proxy_http_version 1.1;
        proxy_set_header Connection "";

        # 缓冲配置（防御慢速攻击）
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 4k;
        proxy_busy_buffers_size 8k;

        # 超时配置
        proxy_connect_timeout 5s;
        proxy_send_timeout 10s;
        proxy_read_timeout 30s;
    }
}

# HTTPS 配置（可选）
server {
    listen 443 ssl http2;
    server_name api.your-domain.com;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    # 复用 HTTP 配置
    location /static/ {
        alias /var/www/static/;
        expires 30d;
    }

    location / {
        proxy_pass http://apisix_gateway;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
        proxy_buffering on;
        proxy_connect_timeout 5s;
        proxy_send_timeout 10s;
        proxy_read_timeout 30s;
    }
}
```
### Hyperf Dockerfile `hyperf/Dockerfile`
```dockerfile
FROM hyperf/hyperf:8.3-alpine-v3.19-swoole

LABEL maintainer="Hyperf Team"
LABEL description="Hyperf Application for APISIX Integration"

WORKDIR /var/www/hyperf

# 复制项目文件
COPY . /var/www/hyperf

# 安装依赖
RUN composer install --optimize-autoloader --no-dev \
    && composer dump-autoload --optimize

# 设置权限
RUN chmod -R 755 runtime

# 暴露端口
EXPOSE 9501

# 健康检查
HEALTHCHECK --interval=30s --timeout=10s --start-period=30s --retries=3 \
    CMD curl -f http://localhost:9501/health || exit 1

# 启动命令
CMD ["php", "bin/hyperf.php", "start"]
```
## 启动与验证
### 启动所有服务
```bash
cd /opt/apisix-hyperf

# 创建必要目录
mkdir -p nginx/conf.d nginx/ssl logs/nginx static apisix-dashboard hyperf

# 启动服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f
```
### 验证各层连通性
```bash
# 1. 验证 etcd
docker exec apisix-etcd etcdctl endpoint health

# 2. 验证 APISIX Admin API
curl http://localhost:9180/apisix/admin/routes \
  -H "X-API-KEY: edd1c9f034335f136f87ad84b625c8f1"

# 3. 验证 APISIX Dashboard
# 浏览器访问 http://localhost:9000
# 用户名/密码: admin/admin

# 4. 验证 Hyperf
curl http://localhost:9501/

# 5. 验证 Nginx 边缘层
curl http://localhost/
```
### 配置 APISIX 路由
通过 Admin API 创建路由，将流量转发到 Hyperf：
```bash
# 创建上游（指向 Hyperf）
curl -X PUT http://localhost:9180/apisix/admin/upstreams/hyperf \
  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' \
  -H 'Content-Type: application/json' \
  -d '{
  "type": "roundrobin",
  "nodes": {
    "hyperf:9501": 1
  },
  "keepalive_pool": {
    "size": 128,
    "idle_timeout": 60
  }
}'

# 创建路由
curl -X PUT http://localhost:9180/apisix/admin/routes/hyperf \
  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' \
  -H 'Content-Type: application/json' \
  -d '{
  "uri": "/*",
  "upstream_id": "hyperf",
  "plugins": {
    "limit-req": {
      "rate": 100,
      "burst": 200,
      "key": "remote_addr"
    },
    "limit-conn": {
      "conn": 2,
      "burst": 1,
      "default_conn_delay": 0.1,
      "key": "remote_addr"
    },
    "prometheus": {}
  }
}'
```
## 生产环境最佳实践要点
### 安全加固
| 项目                 | 配置建议                                   |
| ------------------ | -------------------------------------- |
| **Admin API Key**  | 必须修改默认 key，使用强随机字符串                    |
| **etcd 认证**        | 生产环境启用 `ETCD_USERNAME`/`ETCD_PASSWORD` |
| **API 访问控制**       | 配置 `allow_admin` 仅允许内网访问 Admin API     |
| **CORS 配置**        | 根据业务需求严格配置跨域策略                         |
### 资源限制
所有服务均配置了 `deploy.resources`：
- **etcd**：内存 512M，CPU 0.5 核
- **APISIX**：内存 1G，CPU 1 核（可根据流量调整）
- **Hyperf**：内存 2G，CPU 2 核（建议根据业务调整）
- **Nginx**：内存 256M，CPU 0.5 核
### 健康检查
所有服务均配置了 `healthcheck`：
- **etcd**：`etcdctl endpoint health`
- **APISIX**：`curl http://localhost:9080/`
- **Dashboard**：`curl http://localhost:9000/`
- **Hyperf**：需在应用中添加 `/health` 端点
- **Nginx**：添加 `/health` 返回 200
### 数据持久化
| 组件     | 持久化目录              | 说明           |
| ------ | ------------------ | ------------ |
| etcd   | `./etcd/data`      | 配置数据         |
| Hyperf | `./hyperf/runtime` | 日志和运行时文件     |
| Nginx  | `./logs/nginx`     | 访问和错误日志      |
| 静态资源   | `./static`         | 由 Nginx 直接服务 |
### 监控配置（可选扩展）
如需启用 Prometheus + Grafana 监控，可在 `docker-compose.yml` 中添加：
```yaml
prometheus:
  image: prom/prometheus:v2.48.0
  container_name: prometheus
  volumes:
    - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
  ports:
    - "9090:9090"
  networks:
    - apisix-hyperf-network

grafana:
  image: grafana/grafana:10.2.0
  container_name: grafana
  ports:
    - "3000:3000"
  networks:
    - apisix-hyperf-network
```
## 常用运维命令
```bash
# 启动所有服务
docker-compose up -d

# 停止所有服务
docker-compose down

# 重启单个服务
docker-compose restart apisix

# 查看日志
docker-compose logs -f nginx

# 进入容器调试
docker exec -it apisix-gateway /bin/bash

# 更新 Hyperf 代码后重启
docker-compose build hyperf && docker-compose up -d hyperf

# 备份 etcd 数据
docker exec apisix-etcd etcdctl snapshot save /tmp/snapshot.db
```
## 服务配置

针对 **APISIX + Nginx + Hyperf** 的生产级组合，我为你准备了两个阶段的实战示例：一个是**基础手动对接**（适合初期上线），另一个是**基于 Etcd 的自动服务发现**（适合弹性扩容）。
### 基础方案：手动配置 Upstream (固定节点)
这种方式最稳定，适合 Hyperf 节点变动不频繁的场景。
#### APISIX 路由配置 (Admin API)
创建一个指向 Hyperf 服务的路由，开启 `prometheus` 监控和 `key-auth` 鉴权。
```bash
curl http://127.0.0.1:19180/apisix/admin/routes/hyperf-route -X PUT \
-H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' \
-d '
{
    "uri": "/api/*",
    "name": "hyperf-service-route",
    "plugins": {
        "prometheus": {},
        "key-auth": {}
    },
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "hyperf-service-container:9501": 1
        },
        "keepalive_pool": {
            "size": 320,
            "idle_timeout": 60,
            "requests": 1000
        }
    }
}'
```
#### Nginx 前置接入配置
Nginx 处理 HTTPS 卸载，并透传客户端真实 IP 到 APISIX。
```nginx
upstream apisix_backend {
    server 127.0.0.1:19080;
    keepalive 32;
}

server {
    listen 443 ssl http2;
    server_name api.weichengjun.com;

    ssl_certificate /etc/nginx/ssl/fullchain.pem;
    ssl_certificate_key /etc/nginx/ssl/privkey.pem;

    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        
        proxy_pass http://apisix_backend;
        proxy_http_version 1.1;
        proxy_set_header Connection "";
    }
}
```
### 进阶方案：服务发现 (Etcd 模式)
在生产环境中，Hyperf 容器可能会频繁启停。利用 Hyperf 自带的 `service-governance` 组件配合 APISIX 的 `discovery` 模块，可以实现自动下线死节点。
#### Step 1: APISIX `config.yaml` 开启 Etcd 发现
修改你的 APISIX 配置文件，让它去监听 Etcd 里的服务变动。
```yaml
discovery:
  etcd:
    host:
      - "http://apisix-etcd:2379"
    prefix: "/services/" # 对应 Hyperf 注册的路径前缀
```
#### Step 2: Hyperf 配置服务注册
在 Hyperf 项目中安装 `hyperf/service-governance-etcd`，修改 `config/autoload/services.php`：
```php
return [
    'enable' => [
        'discovery' => true,
        'register' => true,
    ],
    'consumers' => [],
    'providers' => [],
    'drivers' => [
        'etcd' => [
            'host' => 'http://apisix-etcd:2379',
            'retry_attempts' => 20,
            'retry_delay' => 100,
            'nodes' => [
                ['host' => 'apisix-etcd', 'port' => 2379],
            ],
        ],
    ],
];
```
#### Step 3: APISIX 绑定发现的服务
此时创建路由，`upstream` 不再写死 IP，而是写服务名称。
```bash
curl http://127.0.0.1:19180/apisix/admin/routes/hyperf-auto -X PUT \
-H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' \
-d '
{
    "uri": "/user/*",
    "upstream": {
        "service_name": "HyperfService", 
        "discovery_type": "etcd",
        "type": "roundrobin"
    }
}'
```
### 生产环境避坑点汇总
| 维度         | 建议配置                                        | 原因                                                  |
| :--------- | :------------------------------------------ | :-------------------------------------------------- |
| **超时控制**   | APISIX timeout (60s) > Hyperf timeout (30s) | 避免 APISIX 提前断开导致 Hyperf 业务逻辑孤儿运行。                   |
| **长连接**    | `keepalive_pool` 必开                         | Hyperf 基于 Swoole，频繁短连接会撑爆 FD（文件句柄）。                 |
| **客户端 IP** | 使用 `real-ip` 插件                             | Nginx -> APISIX -> Hyperf 链路长，需逐层传递 `$remote_addr`。 |
| **健康检查**   | 开启 `active` 健康检查                            | 如果 Hyperf 进程假死（僵尸进程），APISIX 能自动摘除该节点。               |
## 管理命令
### 连通性与 Admin API 探测
在 Dashboard 抽风或配置不生效时，这是最快的自检手段。
* **检查插件列表 (确认 Key 和端口)**：
```bash
curl -i http://127.0.0.1:19180/apisix/admin/plugins?all=true -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'
```
* **查看已创建的所有路由**：
```bash
curl -i http://127.0.0.1:19180/apisix/admin/routes -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1'
```
### 路由管理 (增删改查)
* **快速创建一个路由 (指向 Hyperf)**：
```bash
curl http://127.0.0.1:19180/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/index/index",
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "hyperf:9501": 1
        }
    }
}'
```
* **局部更新插件 (PATCH)**：不影响原有路由结构，只增改插件。
```bash
curl http://127.0.0.1:19180/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PATCH -d '
{
    "plugins": {
        "limit-count": {
            "count": 2,
            "time_window": 60,
            "key": "remote_addr"
        }
    }
}'
```
* **删除指定路由**：
```bash
curl -i http://127.0.0.1:19180/apisix/admin/routes/1 -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X DELETE
```
### 核心插件配置模版
| 插件名                    | 功能         | 核心参数                                           |
| :--------------------- | :--------- | :--------------------------------------------- |
| **`response-rewrite`** | 修改响应头/体    | `headers`: `{"set": {"X-Header": "Value"}}`    |
| **`limit-count`**      | 固定窗口限流     | `count`: 次数, `time_window`: 秒                  |
| **`key-auth`**         | API Key 认证 | 需要先创建 `consumer`                               |
| **`proxy-rewrite`**    | 修改转发路径     | `uri`: "/new-path"                             |
| **`ip-restriction`**   | IP 黑白名单    | `whitelist`: ["127.0.0.1"], `blacklist`: [...] |
## 限流、熔断
### 限流 (Limit Count) —— 固定窗口计数
最常用的场景：限制一个接口每分钟只能被调用 100 次。
```bash
# 限制 60 秒内 10 次，超出返回 429
curl http://127.0.0.1:19180/apisix/admin/routes/{id} -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PATCH -d '
{
    "plugins": {
        "limit-count": {
            "count": 10,
            "time_window": 60,
            "rejected_code": 429,
            "key": "remote_addr" 
        }
    }
}'
```
* **key**: `remote_addr` (按 IP 限流), `consumer_name` (按用户限流), `http_x_api_key` (按自定义 Header)。
### 限速 (Limit Req) —— 漏桶算法
用于应对突发流量，使请求平滑均匀地到达后端。
```bash
# 每秒处理 1 个请求，允许瞬间爆发 2 个（burst），多余的直接拒绝
curl http://127.0.0.1:19180/apisix/admin/routes/{id} -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PATCH -d '
{
    "plugins": {
        "limit-req": {
            "rate": 1,
            "burst": 2,
            "rejected_code": 503,
            "key": "remote_addr"
        }
    }
}'
```
### 限并发 (Limit Conn)
防止后端数据库或服务被过多的长连接撑爆。
```bash
# 限制单个 IP 同时只能有 2 个并发连接
curl http://127.0.0.1:19180/apisix/admin/routes/{id} -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PATCH -d '
{
    "plugins": {
        "limit-conn": {
            "conn": 2,
            "burst": 1,
            "default_conn_delay": 0.1,
            "key": "remote_addr"
        }
    }
}'
```
### 熔断 (Api Breaker) —— 自动切断故障后端
当后端 Hyperf 疯狂报错（比如 500）时，APISIX 自动停止转发请求，直接返回错误，给后端喘息时间。
```bash
# 如果后端连续返回 3 次 500 或 503，触发熔断 60 秒
curl http://127.0.0.1:19180/apisix/admin/routes/{id} -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PATCH -d '
{
    "plugins": {
        "api-breaker": {
            "break_response_code": 502,
            "unhealthy": {
                "http_statuses": [500, 503],
                "failures": 3
            },
            "healthy": {
                "http_statuses": [200],
                "successes": 3
            }
        }
    }
}'
```
### 黑白名单 (IP Restriction)
最直接的安全防护手段。
```bash
# 仅允许 192.168.1.0 段访问，其他全部拦截
curl http://127.0.0.1:19180/apisix/admin/routes/{id} -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PATCH -d '
{
    "plugins": {
        "ip-restriction": {
            "whitelist": ["192.168.1.0/24", "127.0.0.1"]
        }
    }
}'
```
### 请求改写 (Proxy Rewrite) —— 笔记重点
如果你后端 Hyperf 的接口是 `/api/v1/user`，但你想对外暴露为 `/user`：
```bash
curl http://127.0.0.1:19180/apisix/admin/routes/{id} -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PATCH -d '
{
    "plugins": {
        "proxy-rewrite": {
            "uri": "/api/v1/user",
            "host": "internal.hyperf.com"
        }
    }
}'
```
### 💡 笔记核心技巧：
1.  **PATCH vs PUT**: `PUT` 会覆盖整个路由配置，`PATCH` 可以在不改变其他配置的情况下只更新 `plugins`。
2.  **组合拳**: 一个路由可以同时配置 `limit-count` + `ip-restriction` + `response-rewrite`。
3.  **生效优先级**: 插件执行是有先后顺序的（Priority），通常安全/认证类插件最先执行。
## DDos防御
### 1. 针对登录接口的“暴力破解”与“DDoS”防御
这个示例结合了 **请求频率限制** 和 **并发连接限制**，是最有效的“速效药”。
**场景：** 限制每个客户端 IP 每秒只能请求 1 次登录，允许 2 次突发请求，且同时只能保持 1 个活跃连接。
```bash
curl http://127.0.0.1:9180/apisix/admin/routes/login_split \
-H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "uri": "/api/login",
    "plugins": {
        "limit-req": {
            "rate": 1,
            "burst": 2,
            "key": "remote_addr",
            "rejected_code": 429,
            "rejected_msg": "Too Many Requests, please try again later"
        },
        "limit-conn": {
            "conn": 1,
            "burst": 1,
            "key": "remote_addr",
            "rejected_code": 429
        }
    },
    "upstream": {
        "type": "roundrobin",
        "nodes": {
            "127.0.0.1:8080": 1
        }
    }
}'
```
### 2. 识别并拦截“恶意爬虫/人机”
如果你发现攻击者使用 `python` 或 `curl` 等脚本工具不断刷接口，可以使用 `bot-restriction`。
**场景：** 禁止常见的工具类 User-Agent，但允许搜索引擎（如百度、谷歌）抓取。
```bash
curl http://127.0.0.1:9180/apisix/admin/global_rules/2 \
-H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "plugins": {
        "bot-restriction": {
            "whitelist": [
                "googlebot",
                "baiduspider"
            ],
            "blacklist": [
                "curl",
                "python-requests",
                "httpclient",
                "wget"
            ]
        }
    }
}'
```
### 3. 开启全链路监控（Prometheus + 自定义 Header）
为了确认防御是否生效，以及请求到底有没有走到 APISIX，我们需要开启监控和来源标识。
**场景：** 全局开启监控，并在所有响应中添加 `X-Proxy-By` 标识。
```bash
curl http://127.0.0.1:9180/apisix/admin/global_rules/1 \
-H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' -X PUT -d '
{
    "plugins": {
        "prometheus": {},
        "response-rewrite": {
            "headers": {
                "set": {
                    "X-Proxy-By": "APISIX-Gateway",
                    "X-Server-ID": "Chengdu-Node-01"
                }
            }
        }
    }
}'
```
### 4. 配合日志分析攻击特征
如果你想把拦截到的攻击记录下来发给日志服务器（如 ELK），可以配置 `http-logger`。
```bash
"http-logger": {
    "uri": "http://127.0.0.1:5000/log-collector",
    "batch_max_size": 10,
    "include_req_body": true,
    "concat_method": "json"
}
```
### 小结：防御生效检查清单
1.  **验证标识：** 执行 `curl -I https://你的域名/接口`，检查 Header 是否包含 `X-Proxy-By: APISIX-Gateway`。
2.  **验证限流：** 快速连续刷新页面，确认是否触发了 `HTTP 429` 错误码。
3.  **验证监控：** 访问 `http://apisix-ip:9091/apisix/prometheus/metrics`，搜索 `apisix_http_status`。如果看到 `code="429"` 的数值在增加，说明 APISIX 正在帮你挡刀。
4.  **防火墙锁死：** > **关键安全步骤：** 登录你的服务器执行 `iptables` 规则，只允许来自 Cloudflare 的回源 IP 段访问 APISIX 的监听端口（如 81）。这样攻击者即便知道你的 IP，也无法绕过 CF 直接攻击 APISIX。
## Hyperf 监控接口响应时间
### 1. 安装与配置组件
首先，你需要安装 Hyperf 的监控组件以及 Prometheus 适配器。
```shell
composer require hyperf/metric
# 如果你使用 Prometheus 适配器
composer require promphp/prometheus_client_php
```
#### 若上一步安装不了则仅安装 `hyperf/metric` 核心，不更新其他包
报错提示中提到 `phpoffice/phpspreadsheet` 被锁定在 1.30.2 且未请求更新。由于 `hyperf/metric` 本身并不依赖 `phpspreadsheet`，你可以尝试只安装这一个包：
```shell
composer require "hyperf/metric:*" --no-update
composer update hyperf/metric --ignore-platform-reqs
```
发布配置：
```shell
php bin/hyperf.php vendor:publish hyperf/metric
```
在 `config/autoload/metric.php` 中，将 `default` 驱动设置为 `prometheus`。同时，建议将 `mode` 设置为 `scrape`（拉取模式），这是 Prometheus 的标准工作方式。
### 2. 注册中间件（核心步骤）
为了自动监控所有接口的响应时间，你需要利用 Hyperf 的中间件机制。
在 `config/autoload/middlewares.php` 中，为对应的 Server 注册内置的监控中间件：
```php
return [
    'http' => [
        // 建议放在中间件列表的最外层，以确保记录最准确的耗时
        Hyperf\Metric\Middleware\MetricMiddleware::class,
    ],
];
```
**该中间件的作用：**
* 它会自动收集 HTTP 请求的**总数**。
* 它会自动记录**接口响应时间**。默认情况下，它会生成名为 `http_request_duration_seconds` 的 **Histogram（直方图）** 指标。
### 3. 暴露指标接口 (Scrape Endpoint)
Prometheus 需要通过一个 HTTP 接口来拉取这些数据。通常我们在 `config/routes.php` 中定义：
```php
use Hyperf\HttpServer\Router\Router;

// 暴露指标接口给 Prometheus
Router::addRoute(['GET'], '/metrics', function() {
    $registry = Hyperf\Context\ApplicationContext::getContainer()->get(Prometheus\CollectorRegistry::class);
    $renderer = new Prometheus\RenderTextFormat();
    return $renderer->render($registry->getMetricFamilySamples());
});
```
> **提示：** 生产环境下，请确保 `/metrics` 接口只能被 Prometheus 服务器内网访问，或者增加基础认证。
### 4. Grafana 配置与查询
在 Grafana 中添加 Prometheus 数据源后，你可以创建 Dashboard 来可视化响应时间，下载控制台 [json 文件](https://cdn.jsdelivr.net/gh/hyperf/hyperf/src/metric/grafana.json)，导入 Grafana 中即可使用。
![Pasted image 20260331094539](http://wcj.x3322.net:88/i/2026/03/31/69cb281a30bdf.png)
## 总结
这套 Docker Compose 方案完整实现了 **Nginx（边缘层）→ APISIX（网关层）→ Hyperf（应用层）** 的三层架构，具备以下生产级特性：
1. **职责清晰**：Nginx 处理连接、静态资源和基础防护；APISIX 处理动态路由、精细化限流、认证鉴权；Hyperf 专注业务逻辑 
2. **高可用设计**：健康检查、自动重启、资源限制
3. **数据持久化**：etcd 配置、Hyperf 日志、Nginx 日志均持久化到宿主机
4. **安全加固**：Admin API 密钥可配置、Dashboard 默认密码需修改
5. **可观测性**：APISIX 集成 Prometheus，可接入 Grafana 可视化
6. **云原生兼容**：完整 Docker Compose 编排，可平滑迁移至 Kubernetes（Helm）