# 部署分析 / Deployment Analysis

**生成时间 / Generated**: 2025-11-16
**部署环境 / Deployment Environment**: 本地开发 / Local Development

---

## 📋 部署概览 / Deployment Overview

本项目是本地开发版本，需要启动多个服务才能运行。生产部署需要额外配置。

---

## 🛠️ 本地开发部署 / Local Development Deployment

### 前置要求 / Prerequisites

| 要求 / Requirement | 版本 / Version |
|-------------------|----------------|
| Python | >=3.13 |
| Node.js | >=18 (用于Inngest CLI / for Inngest CLI) |
| Qdrant | 本地实例或Docker / Local or Docker |

### 环境配置 / Environment Setup

**1. 创建 `.env` 文件**:
```bash
OPENAI_API_KEY=sk-your-key-here
INNGEST_API_BASE=http://127.0.0.1:8288/v1
```

**2. 安装依赖**:
```bash
# 使用uv（推荐）/ Using uv (recommended)
uv pip install

# 或使用pip / Or using pip
pip install -r requirements.txt
```

### 启动步骤 / Startup Steps

**Terminal 1: 启动Inngest开发服务器**:
```bash
npx inngest-cli@latest dev
# 访问 http://127.0.0.1:8288
```

**Terminal 2: 启动FastAPI后端**:
```bash
uvicorn main:app --reload
# 访问 http://127.0.0.1:8000
```

**Terminal 3: 启动Streamlit前端**:
```bash
streamlit run streamlit_app.py
# 访问 http://localhost:8501
```

---

## 🐳 Docker部署 / Docker Deployment

### Dockerfile（建议）/ Dockerfile (Suggested)

```dockerfile
# Dockerfile
FROM python:3.13-slim

WORKDIR /app

# 安装依赖 / Install dependencies
COPY pyproject.toml uv.lock ./
RUN pip install uv && uv pip install --system

# 复制代码 / Copy code
COPY . .

# 暴露端口 / Expose ports
EXPOSE 8000 8501

# 启动命令 / Start command
CMD ["bash", "-c", "uvicorn main:app --host 0.0.0.0 --port 8000 & streamlit run streamlit_app.py --server.port 8501 --server.address 0.0.0.0"]
```

### docker-compose.yml（建议）/ docker-compose.yml (Suggested)

```yaml
version: '3.8'

services:
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
    volumes:
      - ./qdrant_storage:/qdrant/storage

  app:
    build: .
    ports:
      - "8000:8000"
      - "8501:8501"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
    volumes:
      - ./uploads:/app/uploads
    depends_on:
      - qdrant
```

---

## ☁️ 云平台部署 / Cloud Platform Deployment

### 推荐平台 / Recommended Platforms

1. **Railway.app** - 适合全栈应用 / Good for full-stack apps
2. **Render.com** - 支持Docker / Supports Docker
3. **Fly.io** - 低延迟全球部署 / Low-latency global deployment
4. **AWS EC2** - 完全控制 / Full control

### 部署注意事项 / Deployment Considerations

- ⚠️ Inngest需要独立部署或使用Inngest Cloud
- ⚠️ 需要配置环境变量
- ⚠️ 需要持久化存储（Qdrant数据）
- ⚠️ 需要配置域名和HTTPS

---

**文档生成完成 / Document Generated**: ✅
**下一步 / Next Step**: 生成重建提示词库
