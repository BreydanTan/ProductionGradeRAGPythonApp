# 基础设施重建提示词 / Foundation Rebuild Prompts

**用途 / Purpose**: 从零开始重建项目的基础设施

---

## 提示词 1.1：项目初始化 / Prompt 1.1: Project Initialization

```
创建一个Python 3.13项目，使用uv作为包管理器。

项目名称：ragproductionapp

依赖包（精确版本）：
- fastapi>=0.116.1
- inngest>=0.5.6
- llama-index-core>=0.14.0
- llama-index-readers-file>=0.5.4
- openai>=1.107.0
- python-dotenv>=1.1.1
- qdrant-client>=1.15.1
- streamlit>=1.49.1
- uvicorn>=0.35.0

创建 pyproject.toml 文件，配置如下：
- name = "ragproductionapp"
- version = "0.1.0"
- requires-python = ">=3.13"

然后运行：uv pip install
```

---

## 提示词 1.2：目录结构创建 / Prompt 1.2: Directory Structure

```
创建以下目录结构：

ProductionGradeRAGPythonApp/
├── uploads/                 # PDF上传存储目录
├── qdrant_storage/          # Qdrant数据存储（自动生成）
├── .gitignore
├── .env                     # 环境变量（不提交到Git）
├── README.md
└── (Python文件稍后创建)

.gitignore 内容：
__pycache__/
*.pyc
.env
qdrant_storage/
uploads/
.idea/
uv.lock

README.md 内容：
# Production-Ready RAG Python Application

基于Inngest的生产级RAG应用。

## 功能
- PDF文档上传和摄取
- 向量化语义搜索
- AI驱动问答系统

## 启动方法
1. 配置 .env 文件
2. 运行 `npx inngest-cli@latest dev`
3. 运行 `uvicorn main:app --reload`
4. 运行 `streamlit run streamlit_app.py`
```

---

## 提示词 1.3：环境变量配置 / Prompt 1.3: Environment Variables

```
创建 .env 文件，包含以下变量：

OPENAI_API_KEY=sk-your-openai-api-key-here
INNGEST_API_BASE=http://127.0.0.1:8288/v1

注意事项：
1. 从 https://platform.openai.com/api-keys 获取 OpenAI API密钥
2. 确保有足够的API余额
3. .env 文件不应提交到Git仓库
```

---

**文档生成完成 / Document Generated**: ✅
