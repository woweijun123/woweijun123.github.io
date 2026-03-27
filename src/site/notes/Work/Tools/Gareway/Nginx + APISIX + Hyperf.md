---
{"dg-publish":true,"permalink":"/Work/Tools/Gareway/Nginx + APISIX + Hyperf/","title":"Nginx + APISIX + Hyperf","tags":["flashcards"],"noteIcon":"","created":"2026-03-27T09:35:36.647+08:00","updated":"2026-03-27T18:19:34.170+08:00","dg-note-properties":{"title":"Nginx + APISIX + Hyperf","tags":["flashcards"],"reference linking":null}}
---

## 一、整体架构
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
## 二、目录结构
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
## 三、核心配置文件
### 3.1 环境变量文件 `.env`
```bash
# 网络配置
NETWORK_SUBNET=172.20.0.0/24

# APISIX 配置
APISIX_IMAGE_TAG=3.12.0-debian
APISIX_ADMIN_KEY=edd1c9f034335f136f87ad84b625c8f1
# 生产环境务必修改 Admin Key
# APISIX_ADMIN_KEY=your-secure-key-here

# etcd 配置
ETCD_VERSION=3.5.11

# Hyperf 配置
HYPERF_PORT=9501
HYPERF_WORKER_NUM=4

# Nginx 配置
NGINX_HTTP_PORT=80
NGINX_HTTPS_PORT=443
```
### 3.2 主编排文件 `docker-compose.yml`
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
    image: bitnami/etcd:${ETCD_VERSION:-3.5.11}
    container_name: apisix-etcd
    restart: unless-stopped
    environment:
      ALLOW_NONE_AUTHENTICATION: "yes"
      ETCD_ENABLE_V2: "true"
      ETCD_ADVERTISE_CLIENT_URLS: "http://etcd:2379"
      ETCD_LISTEN_CLIENT_URLS: "http://0.0.0.0:2379"
      ETCD_DATA_DIR: "/bitnami/etcd/data"
    volumes:
      - etcd_data:/bitnami/etcd
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
    deploy:
      resources:
        limits:
          memory: 1G
          cpus: '1'
        reservations:
          memory: 512M
          cpus: '0.5'

  # ==================== 3. APISIX Dashboard ====================
  apisix-dashboard:
    image: apache/apisix-dashboard:3.0.1
    container_name: apisix-dashboard
    restart: unless-stopped
    depends_on:
      - apisix
      - etcd
    volumes:
      - ./apisix-dashboard/conf.yaml:/usr/local/apisix-dashboard/conf/conf.yaml:ro
    ports:
      - "9000:9000"   # Dashboard Web 端口
    networks:
      apisix-hyperf-network:
        ipv4_address: 172.20.0.4
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:9000/"]
      interval: 30s
      timeout: 10s
      retries: 3
    deploy:
      resources:
        limits:
          memory: 256M
          cpus: '0.25'

  # ==================== 4. Hyperf 应用 ====================
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

  # ==================== 5. Nginx 边缘层 ====================
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
### 3.3 APISIX 配置 `apisix/config.yaml`
```yaml
apisix:
  node_listen: 9080
  enable_admin: true
  enable_heartbeat: true
  # 生产环境建议开启，便于监控节点存活
  show_instance_id: true 

deployment:
  role: traditional
  role_traditional:
    config_provider: etcd
  etcd:
    host:
      - "http://apisix-etcd:2379" # 必须与 docker-compose 服务名一致
    prefix: "/apisix"
    timeout: 30
  admin:
    # 注意：APISIX 不支持 YAML 内读取 .env 变量，需手动填入或使用脚本替换
    admin_key:
      - name: "admin"
        key: "edd1c9f034335f136f87ad84b625c8f1" # 建议此处与 .env 保持物理一致
        role: admin
    enable_admin_cors: true
    allow_admin:
      - 0.0.0.0/0 # 生产环境建议缩小范围，如仅允许内网网段

nginx_config:
  worker_processes: auto
  worker_rlimit_nofile: 65535
  event:
    worker_connections: 10240
  http:
    enable_access_log: true
    # 增加 request_id 方便链路追踪
    access_log_format: '{"time_local":"$time_local","remote_addr":"$remote_addr","request_id":"$request_id","request":"$request","status":$status,"body_bytes_sent":$body_bytes_sent,"request_time":$request_time,"upstream_response_time":"$upstream_response_time","http_referer":"$http_referer","http_user_agent":"$http_user_agent"}'
    keepalive_timeout: 60s
    client_header_timeout: 60s
    client_body_timeout: 60s
    send_timeout: 60s
    gzip: on
    gzip_min_length: 1024
    gzip_types: text/plain text/css application/json application/javascript application/xml

# 插件列表建议精简，仅加载需要的
plugins:
  - prometheus
  - limit-req
  - limit-conn
  - limit-count
  - key-auth
  - jwt-auth
  - cors
  - ip-restriction
  - proxy-rewrite
  - response-rewrite
  - file-logger # 建议加上，方便调试

plugin_attr:
  prometheus:
    export_addr:
      ip: "0.0.0.0"
      port: 9091
```
### 3.4 APISIX Dashboard 配置 `apisix-dashboard/conf.yaml`
```yaml
conf:
  listen:
    host: 0.0.0.0
    port: 9000
  etcd:
    endpoints:
      - http://apisix-etcd:2379 # 必须与 APISIX 连的是同一个 ETCD
    prefix: "/apisix"
  # 必须配置，否则 Dashboard 无法操作 APISIX 实例
  apisix:
    admin_key: "edd1c9f034335f136f87ad84b625c8f1" 
  log:
    error_log:
      level: warn
      file_path: logs/error.log
  security:
    csrf: true
    session_timeout: 3600

authentication:
  # 务必修改这个随机字符串
  secret: "3d9a1b2c4e5f6g7h8i9j0k1l2m3n4o5p" 
  expire_time: 3600
  users:
    - username: admin
      # 建议使用强密码
      password: "YourStrongPassword123!"
```
### 3.5 Nginx 配置
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
### 3.6 Hyperf Dockerfile `hyperf/Dockerfile`
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
## 四、启动与验证
### 4.1 启动所有服务
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
### 4.2 验证各层连通性
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
### 4.3 配置 APISIX 路由
通过 Admin API 创建路由，将流量转发到 Hyperf：
```bash
# 创建上游（指向 Hyperf）
curl -X PUT http://localhost:9180/apisix/admin/upstreams/1 \
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
curl -X PUT http://localhost:9180/apisix/admin/routes/1 \
  -H 'X-API-KEY: edd1c9f034335f136f87ad84b625c8f1' \
  -H 'Content-Type: application/json' \
  -d '{
    "uri": "/*",
    "upstream_id": 1,
    "plugins": {
      "limit-req": {
        "rate": 100,
        "burst": 200,
        "key": "remote_addr"
      },
      "limit-conn": {
        "conn": 1000,
        "burst": 200,
        "key": "remote_addr"
      },
      "prometheus": {}
    }
  }'
```
## 五、生产环境最佳实践要点
### 5.1 安全加固
| 项目                 | 配置建议                                   |
| ------------------ | -------------------------------------- |
| **Admin API Key**  | 必须修改默认 key，使用强随机字符串                    |
| **Dashboard 默认密码** | 修改 admin/user 默认密码                     |
| **etcd 认证**        | 生产环境启用 `ETCD_USERNAME`/`ETCD_PASSWORD` |
| **API 访问控制**       | 配置 `allow_admin` 仅允许内网访问 Admin API     |
| **CORS 配置**        | 根据业务需求严格配置跨域策略                         |
### 5.2 资源限制
所有服务均配置了 `deploy.resources`：
- **etcd**：内存 512M，CPU 0.5 核
- **APISIX**：内存 1G，CPU 1 核（可根据流量调整）
- **Dashboard**：内存 256M，CPU 0.25 核
- **Hyperf**：内存 2G，CPU 2 核（建议根据业务调整）
- **Nginx**：内存 256M，CPU 0.5 核
### 5.3 健康检查
所有服务均配置了 `healthcheck`：
- **etcd**：`etcdctl endpoint health`
- **APISIX**：`curl http://localhost:9080/`
- **Dashboard**：`curl http://localhost:9000/`
- **Hyperf**：需在应用中添加 `/health` 端点
- **Nginx**：添加 `/health` 返回 200
### 5.4 数据持久化
| 组件     | 持久化目录              | 说明           |
| ------ | ------------------ | ------------ |
| etcd   | `./etcd/data`      | 配置数据         |
| Hyperf | `./hyperf/runtime` | 日志和运行时文件     |
| Nginx  | `./logs/nginx`     | 访问和错误日志      |
| 静态资源   | `./static`         | 由 Nginx 直接服务 |
### 5.5 监控配置（可选扩展）
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
## 六、常用运维命令
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
## 七、总结
这套 Docker Compose 方案完整实现了 **Nginx（边缘层）→ APISIX（网关层）→ Hyperf（应用层）** 的三层架构，具备以下生产级特性：
1. **职责清晰**：Nginx 处理连接、静态资源和基础防护；APISIX 处理动态路由、精细化限流、认证鉴权；Hyperf 专注业务逻辑 
2. **高可用设计**：健康检查、自动重启、资源限制
3. **数据持久化**：etcd 配置、Hyperf 日志、Nginx 日志均持久化到宿主机
4. **安全加固**：Admin API 密钥可配置、Dashboard 默认密码需修改
5. **可观测性**：APISIX 集成 Prometheus，可接入 Grafana 可视化
6. **云原生兼容**：完整 Docker Compose 编排，可平滑迁移至 Kubernetes（Helm）