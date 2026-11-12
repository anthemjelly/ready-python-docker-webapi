> **⚠️ 已遷移（Archived / Migrated）**
> 這個專案已由 **[anthemjelly/ready-stack](https://github.com/anthemjelly/ready-stack)** 的
> `packages/ready-python-docker-webapi/` 取代（真正可運行的 FastAPI + PostgreSQL 服務，含 Alembic、測試、CI）。
> 新功能與修正只在 ready-stack 進行；本 repo 只保留作歷史紀錄。

# Python Web API 專案
[![Python Version](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.100+-green.svg)](https://fastapi.tiangolo.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## 📦 專案簡介
本專案是基於 **FastAPI** 構建的輕量級 Python Web API，支援 RESTful 接口、自動生成 API 文件、數據庫整合與容器化部署，適合作為後端服務的基礎模板。

## 🛠️ 技術棧
- **Web 框架**：FastAPI（高效能、自動文件生成）
- **ASGI 伺服器**：Uvicorn
- **數據庫**：PostgreSQL（範例）
- **數據驗證**：Pydantic
- **容器化**：Docker + Docker Compose

## 🚀 快速開始
### 前置條件
- 已安裝 Python 3.12 或更高版本
- 已安裝 pip（Python 套件管理工具）
- （選用）已安裝 PostgreSQL 數據庫（若需本地開發）

### 操作步驟
1. **複製專案**
   ```bash
   git clone https://github.com/anthemjelly/ready-python-docker-webapi.git
   cd ready-python-docker-webapi
   ```

2. **建立虛擬環境**
   ```bash
   # Linux/macOS
   python3 -m venv venv
   source venv/bin/activate

   # Windows
   python -m venv venv
   venv\Scripts\activate
   ```

3. **安裝依賴**
   ```bash
   pip install -r requirements.txt
   ```

4. **設定環境變數**
   複製 `.env.example` 為 `.env`，並填入你的配置：
   ```env
   # .env 範例
   DATABASE_URL=postgresql://user:password@localhost:5432/api_db
   SECRET_KEY=your-secret-key-here
   ```

5. **啟動開發伺服器**
   ```bash
   uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
   ```

6. **訪問 API 文件**
   - Swagger UI：http://localhost:8000/docs
   - ReDoc：http://localhost:8000/redoc

## 📁 專案結構
```
your-python-api/
├── app/
│   ├── __init__.py
│   ├── main.py          # 應用入口
│   ├── api/             # API 路由
│   │   ├── __init__.py
│   │   └── items.py     # 範例接口
│   ├── models/          # 數據庫模型
│   │   ├── __init__.py
│   │   └── item.py
│   ├── schemas/         # Pydantic 數據驗證
│   │   ├── __init__.py
│   │   └── item.py
│   └── database.py      # 數據庫連線
├── tests/               # 測試檔案
│   ├── __init__.py
│   └── test_api.py
├── .env.example         # 環境變數範本
├── .dockerignore        # Docker 忽略檔案
├── Dockerfile           # Docker 鏡像構建
├── docker-compose.yml   # Docker Compose 配置
├── requirements.txt     # Python 依賴
└── README.md            # 本文件
```

## 📝 開發指南
### 新增 API 接口
1. 在 `app/api/` 下建立新的路由檔案（如 `users.py`）
2. 在 `app/main.py` 中引入並註冊路由：
   ```python
   from app.api import users
   app.include_router(users.router, prefix="/users", tags=["users"])
   ```

### 執行測試
```bash
pytest tests/ -v
```

## 🤝 貢獻指南
1. Fork 本倉庫
2. 建立特性分支 (`git checkout -b feature/AmazingFeature`)
3. 提交修改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送至分支 (`git push origin feature/AmazingFeature`)
5. 打開 Pull Request

## 📄 許可證
本項目採用 MIT 許可證 - 詳見 [LICENSE](LICENSE) 文件