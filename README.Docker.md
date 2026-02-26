# README.docker.md - Python API Docker 部署指南
[![Docker Build](https://img.shields.io/badge/Docker-Ready-blue.svg)](https://docs.docker.com/)
[![Docker Compose](https://img.shields.io/badge/Compose-v2.0+-green.svg)](https://docs.docker.com/compose/)

## 📋 檔案說明
本文件專門說明 Python Web API 專案的 Docker 容器化部署細節，包含鏡像構建、多容器編排（API + PostgreSQL）、數據持久化等核心操作。

## 🛠️ 環境準備
### 1. 安裝 Docker
- **Linux**
  ```bash
  # Ubuntu/Debian
  sudo apt update && sudo apt install -y docker.io docker-compose-plugin
  sudo usermod -aG docker $USER  # 免 sudo 使用 Docker
  # 重啟終端機生效
  ```
- **macOS/Windows**
  下載安裝 [Docker Desktop](https://www.docker.com/products/docker-desktop/)（內含 Docker Compose）

### 2. 驗證安裝
```bash
docker --version          # 檢查 Docker 版本
docker compose version    # 檢查 Compose 版本
```

## 🔧 核心配置檔說明
### 1. Dockerfile
使用**多階段構建**減小鏡像體積，優化下載速度：
```dockerfile
# 階段1：構建依賴（安裝編譯工具與依賴）
FROM python:3.11-slim AS builder
WORKDIR /app
# 安裝系統依賴（PostgreSQL 連線需要）
RUN apt update && apt install -y --no-install-recommends gcc libpq-dev
# 建立虛擬環境
RUN python -m venv /venv
ENV PATH="/venv/bin:$PATH"
# 安裝 Python 依賴
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# 階段2：生產環境鏡像（輕量級，僅保留執行所需）
FROM python:3.11-slim
WORKDIR /app
# 安裝執行時系統依賴
RUN apt update && apt install -y --no-install-recommends libpq5 && rm -rf /var/lib/apt/lists/*
# 複製構建階段的虛擬環境
COPY --from=builder /venv /venv
ENV PATH="/venv/bin:$PATH"
# 複製應用程式碼
COPY app/ ./app/
# 暴露應用端口
EXPOSE 8000
# 啟動命令（Uvicorn 生產環境模式）
CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000", "--workers", "4"]
```

### 2. docker-compose.yml
整合 API 服務與 PostgreSQL 數據庫，支援數據持久化與自動重啟：
```yaml
version: '3.8'

# 自定義網路（隔離專案網路）
networks:
  api-network:
    driver: bridge

# 數據卷（持久化數據）
volumes:
  postgres-data:  # PostgreSQL 數據
  api-logs:       # API 日誌

services:
  # API 服務
  api:
    build:
      context: .
      dockerfile: Dockerfile
    restart: unless-stopped  # 異常退出自動重啟
    ports:
      - "8000:8000"
    environment:
      - DATABASE_URL=postgresql://api_user:api_password@postgres:5432/api_db
      - SECRET_KEY=your-production-secret-key
    volumes:
      - api-logs:/app/logs
    networks:
      - api-network
    depends_on:
      - postgres  # 先啟動數據庫

  # PostgreSQL 數據庫服務
  postgres:
    image: postgres:15-alpine
    restart: unless-stopped
    environment:
      - POSTGRES_USER=api_user
      - POSTGRES_PASSWORD=api_password
      - POSTGRES_DB=api_db
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - api-network
    ports:
      - "5432:5432"  # 本地開發時可直接連線
```

### 3. .dockerignore
排除無需打包的檔案，減小鏡像體積：
```
# 虛擬環境與依賴
venv/
__pycache__/
*.pyc
*.pyo

# 版本控制
.git/
.gitignore

# 日誌
logs/
*.log

# 環境配置
.env
.env.local

# 編輯器配置
.idea/
.vscode/
*.swp
*.swo

# Docker 自身配置
Dockerfile
docker-compose.yml
.dockerignore

# 測試與文件
tests/
README.md
```

## 🚀 常用操作指令
### 1. 基礎操作
```bash
# 構建並啟動所有服務（後台運行）
docker compose up -d --build

# 啟動已構建的服務
docker compose up -d

# 查看運行中的容器
docker compose ps

# 查看服務日誌（跟蹤實時日誌）
docker compose logs -f api       # 只看 API 日誌
docker compose logs -f postgres  # 只看數據庫日誌
docker compose logs -f           # 看所有服務日誌

# 停止所有服務（保留容器與數據）
docker compose stop

# 停止並刪除容器、網路（保留數據卷）
docker compose down

# 停止並刪除所有資源（包含數據卷，徹底清理）
docker compose down -v
```

### 2. 進階操作
```bash
# 進入 API 容器
docker compose exec api /bin/bash

# 進入 PostgreSQL 容器並連線數據庫
docker compose exec postgres psql -U api_user -d api_db

# 手動構建並標記鏡像
docker build -t your-registry/your-python-api:latest .

# 推送鏡像到私有倉庫
docker push your-registry/your-python-api:latest

# 清理無用資源
docker system prune -a  # 清理無用鏡像、容器、網路
docker volume prune     # 清理未使用的數據卷
```

## ⚡ 優化建議
### 1. 鏡像優化
- 使用多階段構建（已在 Dockerfile 中實現），最終鏡像體積可減少 50% 以上
- 選擇 `slim` 或 `alpine` 版本的基礎鏡像
- 合理使用 `.dockerignore`，排除無關檔案
- 將 `requirements.txt` 單獨複製，充分利用 Docker 層緩存

### 2. 生產環境優化
- 資源限制：在 `docker-compose.yml` 中添加 `deploy` 配置
  ```yaml
  services:
    api:
      deploy:
        resources:
          limits:
            cpus: '1.0'    # 最多使用 1 個 CPU 核心
            memory: 1G      # 最多使用 1GB 內存
  ```
- 敏感資訊：使用 Docker Secrets 或外部秘密管理工具，避免直接寫在環境變數中
- 日誌收集：配置 ELK Stack 或 Loki 收集容器日誌
- 反向代理：使用 Nginx 或 Traefik 作為反向代理，處理 SSL 與負載均衡

## ❌ 常見問題排查
| 問題現象 | 可能原因 | 解決方案 |
|----------|----------|----------|
| API 容器啟動失敗 | 數據庫未就緒、DATABASE_URL 錯誤 | 檢查 `depends_on` 配置；確認環境變數正確；查看日誌 |
| 數據庫連線失敗 | 容器不在同一網路、服務名錯誤 | 確保服務加入 `api-network`；使用服務名 `postgres` 而非 IP |
| 數據持久化失敗 | 數據卷配置錯誤 | 檢查 `volumes` 路徑；確認數據卷未被誤刪 |
| 鏡像構建緩慢 | 網路問題、依賴下載慢 | 更換國內 pip 鏡像源（在 Dockerfile 中添加 `RUN pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple`） |

## 📞 支援與反饋
- [Docker 官方文檔](https://docs.docker.com/)
- [FastAPI 官方文檔](https://fastapi.tiangolo.com/deployment/docker/)
- 提交 Issue 到專案倉庫，標註「Docker 部署」標籤