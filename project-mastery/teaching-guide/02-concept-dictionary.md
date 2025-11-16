# 概念词典 / Concept Dictionary

**用途**: 解释项目中的技术术语

---

## 核心概念

### RAG (检索增强生成)
**简单解释**: 像是给AI配了一个图书馆。AI不直接回答问题，而是先查图书馆找相关资料，然后基于资料回答。

**在本项目中**:
- 图书馆 = Qdrant向量数据库
- 查找 = 向量相似度搜索
- 回答 = OpenAI GPT-4o-mini

**代码位置**: `main.py:60-99`

### 向量嵌入 (Vector Embedding)
**简单解释**: 把文字转换成数字列表，相似的文字会变成相似的数字。就像给每个句子打一个"指纹"。

**在本项目中**: 使用OpenAI的text-embedding-3-large模型，每个文本变成3072个数字。

**代码位置**: `data_loader.py:23-28`

### Qdrant
**简单解释**: 一个专门存储"文字指纹"（向量）的数据库。可以快速找到"指纹相似"的文本。

**在本项目中**: 本地存储在`qdrant_storage/`目录。

**代码位置**: `vector_db.py`

### Inngest
**简单解释**: 像是工作流管理器，确保任务按顺序执行，失败了会重试，还能限制速度防止过载。

**在本项目中**: 管理PDF摄取和查询两个核心流程。

**代码位置**: `main.py:16-21, 23-99`

### Streamlit
**简单解释**: 用纯Python写Web界面的框架，不需要学HTML/CSS/JavaScript。

**在本项目中**: 提供PDF上传和问答界面。

**代码位置**: `streamlit_app.py`

### FastAPI
**简单解释**: 快速的Python Web框架，用来创建API服务器。

**在本项目中**: 托管Inngest端点。

**代码位置**: `main.py:101-103`

---

## 数据库概念

### Collection（集合）
**简单解释**: 数据库中的一个容器，类似MySQL的表。

**在本项目中**: 只有一个集合叫"docs"，存储所有文档。

**代码位置**: `vector_db.py:8`

### Payload（负载）
**简单解释**: 向量携带的额外信息，比如原始文本和来源文件名。

**在本项目中**: 每个向量携带`{"source": "文件名", "text": "文本内容"}`。

**代码位置**: `main.py:47`, `vector_db.py:30-35`

---

## AI概念

### LLM（大语言模型）
**简单解释**: 训练过海量文本的AI模型，可以理解和生成人类语言。

**在本项目中**: 使用OpenAI的GPT-4o-mini。

**代码位置**: `main.py:82`

### Token
**简单解释**: AI处理文本的基本单位，大约1个token = 0.75个英文单词。

**在本项目中**: 每次查询最多生成1024个tokens的答案。

**代码位置**: `main.py:89` (max_tokens: 1024)

### Temperature（温度）
**简单解释**: 控制AI的"创造性"。0=非常确定，1=非常随机。

**在本项目中**: 设为0.2，比较确定性的回答。

**代码位置**: `main.py:90`

---

参考：
- project-mastery/analysis/00-project-overview.md
- project-mastery/specifications/PROJECT_SPEC_CN.md
