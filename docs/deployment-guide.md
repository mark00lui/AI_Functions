# 部署指南

## 📋 概述

本文檔提供Quectel AI功能的完整部署指南，包括開發環境、測試環境和生產環境的部署方案。

## 🏗️ 部署架構

### 整體架構
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Load Balancer │    │   API Gateway   │    │   Microservices │
│                 │◄──►│                 │◄──►│                 │
│ - Nginx         │    │ - Kong          │    │ - MCP Servers   │
│ - HAProxy       │    │ - Auth          │    │ - AI Services   │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │   Data Layer    │
                       │                 │
                       │ - PostgreSQL    │
                       │ - Redis         │
                       │ - MongoDB       │
                       └─────────────────┘
```

## 🔧 環境要求

### 硬件要求
- **CPU**: 8核心以上
- **內存**: 16GB以上
- **存儲**: 100GB以上SSD
- **網絡**: 千兆網卡

### 軟件要求
- **操作系統**: Ubuntu 20.04 LTS / CentOS 8
- **Docker**: 20.10+
- **Kubernetes**: 1.21+
- **Python**: 3.8+
- **Node.js**: 16+

## 🚀 快速部署

### 1. 使用Docker Compose

#### 創建docker-compose.yml
```yaml
version: '3.8'

services:
  # API Gateway
  kong:
    image: kong:2.8
    ports:
      - "8000:8000"
      - "8001:8001"
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: postgres
      KONG_PG_DATABASE: kong
    depends_on:
      - postgres

  # MCP Servers
  mcp-cellular:
    image: quectel/mcp-cellular:latest
    ports:
      - "8002:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/quectel

  mcp-wifi:
    image: quectel/mcp-wifi:latest
    ports:
      - "8003:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/quectel

  mcp-gps:
    image: quectel/mcp-gps:latest
    ports:
      - "8004:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/quectel

  mcp-bluetooth:
    image: quectel/mcp-bluetooth:latest
    ports:
      - "8005:8000"
    environment:
      - DATABASE_URL=postgresql://user:pass@postgres:5432/quectel

  # AI Services
  ai-agent:
    image: quectel/ai-agent:latest
    ports:
      - "8006:8000"
    environment:
      - OPENAI_API_KEY=your_openai_key
      - ANTHROPIC_API_KEY=your_anthropic_key

  # Voice Services
  voice-asr:
    image: quectel/voice-asr:latest
    ports:
      - "8007:8000"

  voice-tts:
    image: quectel/voice-tts:latest
    ports:
      - "8008:8000"

  # Database
  postgres:
    image: postgres:13
    environment:
      POSTGRES_DB: quectel
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

  mongodb:
    image: mongo:5
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: pass
    volumes:
      - mongo_data:/data/db

volumes:
  postgres_data:
  mongo_data:
```

#### 啟動服務
```bash
# 克隆部署腳本
git clone https://github.com/quectel/ai-deployment
cd ai-deployment

# 啟動所有服務
docker-compose up -d

# 檢查服務狀態
docker-compose ps

# 查看日誌
docker-compose logs -f
```

### 2. 使用Kubernetes

#### 創建namespace
```bash
kubectl create namespace quectel-ai
kubectl config set-context --current --namespace=quectel-ai
```

#### 部署數據庫
```yaml
# postgres-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:13
        env:
        - name: POSTGRES_DB
          value: quectel
        - name: POSTGRES_USER
          value: user
        - name: POSTGRES_PASSWORD
          value: pass
        ports:
        - containerPort: 5432
        volumeMounts:
        - name: postgres-storage
          mountPath: /var/lib/postgresql/data
      volumes:
      - name: postgres-storage
        persistentVolumeClaim:
          claimName: postgres-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: postgres
spec:
  selector:
    app: postgres
  ports:
  - port: 5432
    targetPort: 5432
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

#### 部署MCP服務
```yaml
# mcp-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mcp-cellular
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mcp-cellular
  template:
    metadata:
      labels:
        app: mcp-cellular
    spec:
      containers:
      - name: mcp-cellular
        image: quectel/mcp-cellular:latest
        ports:
        - containerPort: 8000
        env:
        - name: DATABASE_URL
          value: postgresql://user:pass@postgres:5432/quectel
        resources:
          requests:
            memory: "256Mi"
            cpu: "250m"
          limits:
            memory: "512Mi"
            cpu: "500m"
---
apiVersion: v1
kind: Service
metadata:
  name: mcp-cellular
spec:
  selector:
    app: mcp-cellular
  ports:
  - port: 8000
    targetPort: 8000
```

#### 部署Ingress
```yaml
# ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: quectel-ai-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: api.quectel.com
    http:
      paths:
      - path: /mcp/cellular
        pathType: Prefix
        backend:
          service:
            name: mcp-cellular
            port:
              number: 8000
      - path: /mcp/wifi
        pathType: Prefix
        backend:
          service:
            name: mcp-wifi
            port:
              number: 8000
```

## 🔧 配置管理

### 環境變量配置
```bash
# .env
# API配置
API_BASE_URL=https://api.quectel.com/v1
API_KEY=your_api_key

# 數據庫配置
DATABASE_URL=postgresql://user:pass@localhost:5432/quectel
REDIS_URL=redis://localhost:6379
MONGO_URL=mongodb://localhost:27017

# AI服務配置
OPENAI_API_KEY=your_openai_key
ANTHROPIC_API_KEY=your_anthropic_key
GOOGLE_API_KEY=your_google_key

# 語音服務配置
ASR_MODEL=whisper-large-v3
TTS_MODEL=tts-1
WAKE_WORD_MODEL=wake_word_v1

# 監控配置
PROMETHEUS_URL=http://localhost:9090
GRAFANA_URL=http://localhost:3000
```

### 配置文件
```yaml
# config.yaml
api:
  base_url: ${API_BASE_URL}
  api_key: ${API_KEY}
  timeout: 30

database:
  postgres:
    url: ${DATABASE_URL}
    pool_size: 10
  redis:
    url: ${REDIS_URL}
    pool_size: 5
  mongodb:
    url: ${MONGO_URL}

ai:
  openai:
    api_key: ${OPENAI_API_KEY}
    model: gpt-4
  anthropic:
    api_key: ${ANTHROPIC_API_KEY}
    model: claude-3
  google:
    api_key: ${GOOGLE_API_KEY}
    model: gemini-pro

voice:
  asr:
    model: ${ASR_MODEL}
    language: zh-CN
  tts:
    model: ${TTS_MODEL}
    voice: female
  wake_word:
    model: ${WAKE_WORD_MODEL}
    sensitivity: 0.8

monitoring:
  prometheus:
    url: ${PROMETHEUS_URL}
  grafana:
    url: ${GRAFANA_URL}
```

## 📊 監控和日誌

### Prometheus監控
```yaml
# prometheus-config.yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'quectel-ai'
    static_configs:
      - targets: ['mcp-cellular:8000', 'mcp-wifi:8000', 'ai-agent:8000']
    metrics_path: /metrics
```

### Grafana儀表板
```json
{
  "dashboard": {
    "title": "Quectel AI Dashboard",
    "panels": [
      {
        "title": "API請求數",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[5m])",
            "legendFormat": "{{service}}"
          }
        ]
      },
      {
        "title": "響應時間",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          }
        ]
      }
    ]
  }
}
```

### 日誌收集
```yaml
# fluentd-config.yaml
<source>
  @type tail
  path /var/log/quectel-ai/*.log
  pos_file /var/log/fluentd/quectel-ai.log.pos
  tag quectel-ai
  <parse>
    @type json
  </parse>
</source>

<match quectel-ai>
  @type elasticsearch
  host elasticsearch
  port 9200
  index_name quectel-ai-logs
</match>
```

## 🔐 安全配置

### SSL/TLS配置
```nginx
# nginx-ssl.conf
server {
    listen 443 ssl;
    server_name api.quectel.com;
    
    ssl_certificate /etc/ssl/certs/quectel.crt;
    ssl_certificate_key /etc/ssl/private/quectel.key;
    
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 防火牆配置
```bash
# 開放必要端口
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 22/tcp

# 限制訪問
sudo ufw deny from 192.168.1.0/24 to any port 22
```

## 📋 TODO 清單

### 高優先級
- [ ] 完善部署腳本
- [ ] 實現自動化部署
- [ ] 建立監控系統
- [ ] 配置安全設置

### 中優先級
- [ ] 實現藍綠部署
- [ ] 建立備份策略
- [ ] 實現自動擴展
- [ ] 建立災難恢復

### 低優先級
- [ ] 實現多區域部署
- [ ] 建立CDN配置
- [ ] 實現A/B測試
- [ ] 建立性能優化

## 🔗 相關文檔

- [API接口文檔](./api-documentation.md)
- [SDK使用指南](./sdk-guide.md)
- [MCP Server實現](./05-mcp-server.md)

---

*最後更新：2024年* 