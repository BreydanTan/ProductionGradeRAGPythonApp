# 后端分析 / Backend Analysis

**生成时间 / Generated**: 2025-11-16
**后端框架 / Backend Framework**: FastAPI + Inngest

---

## 📋 后端架构概览 / Backend Architecture Overview

### 中文
本项目的后端采用**FastAPI + Inngest**的混合架构。FastAPI作为Web框架提供HTTP服务器，而Inngest作为事件驱动的工作流引擎负责处理复杂的异步任务，提供了限流、节流、重试和监控等生产级特性。

**架构特点**：
- 🚀 **FastAPI**: 高性能异步Web框架
- 🔄 **Inngest**: 事件驱动工作流编排
- 📊 **可观测性**: 内置监控和追踪
- 🛡️ **容错性**: 自动重试和错误处理
- ⚡ **限流保护**: 防止滥用和过载

### English
The backend uses a **FastAPI + Inngest** hybrid architecture. FastAPI serves as the web framework providing HTTP server capabilities, while Inngest acts as an event-driven workflow engine handling complex asynchronous tasks with production-grade features like rate limiting, throttling, retries, and monitoring.

**Architecture Characteristics**:
- 🚀 **FastAPI**: High-performance async web framework
- 🔄 **Inngest**: Event-driven workflow orchestration
- 📊 **Observability**: Built-in monitoring and tracing
- 🛡️ **Fault Tolerance**: Automatic retries and error handling
- ⚡ **Rate Limiting**: Protection against abuse and overload

---

## 🏗️ 入口文件 / Entry Point

**文件 / File**: `main.py` (103行 / 103 lines)
**位置 / Location**: `/main.py`

### 导入依赖 / Imports
```python
# main.py:1-12
import logging
from fastapi import FastAPI
import inngest
import inngest.fast_api
from inngest.experimental import ai
from dotenv import load_dotenv
import uuid
import os
import datetime
from data_loader import load_and_chunk_pdf, embed_texts
from vector_db import QdrantStorage
from custom_types import RAQQueryResult, RAGSearchResult, RAGUpsertResult, RAGChunkAndSrc
```

### 初始化 / Initialization
```python
# main.py:14
load_dotenv()  # 加载环境变量 / Load environment variables

# main.py:16-21
inngest_client = inngest.Inngest(
    app_id="rag_app",                              # 应用ID / App ID
    logger=logging.getLogger("uvicorn"),          # 日志器 / Logger
    is_production=False,                          # 开发模式 / Dev mode
    serializer=inngest.PydanticSerializer()       # Pydantic序列化 / Pydantic serializer
)
```

### FastAPI应用 / FastAPI App
```python
# main.py:101
app = FastAPI()

# main.py:103
inngest.fast_api.serve(app, inngest_client, [rag_ingest_pdf, rag_query_pdf_ai])
```

**说明 / Explanation**: `inngest.fast_api.serve` 会自动在FastAPI应用中注册Inngest端点，用于接收和处理Inngest事件。

---

## 🔌 API端点清单 / API Endpoints

### 自动生成的Inngest端点 / Auto-Generated Inngest Endpoints

Inngest会自动创建以下端点：

| 方法 / Method | 路径 / Path | 作用 / Purpose |
|--------------|------------|----------------|
| `POST` | `/api/inngest` | Inngest事件接收端点 / Inngest event receiver |
| `GET` | `/api/inngest` | Inngest健康检查和元数据 / Health check & metadata |
| `PUT` | `/api/inngest` | Inngest函数注册 / Function registration |

**注意 / Note**: 这些端点由Inngest SDK自动管理，不需要手动定义。

### 事件触发方式 / Event Triggering

应用不直接暴露RESTful API，而是通过Inngest事件系统：

```python
# 客户端（如streamlit_app.py）发送事件 / Client sends event
await client.send(
    inngest.Event(
        name="rag/ingest_pdf",  # 事件名称 / Event name
        data={...}              # 事件数据 / Event data
    )
)
```

---

## 🎯 核心业务函数 / Core Business Functions

### 1. rag_ingest_pdf（PDF摄取函数）

**位置 / Location**: `main.py:23-53`
**触发事件 / Trigger Event**: `rag/ingest_pdf`
**函数ID / Function ID**: `"RAG: Ingest PDF"`

#### 函数签名 / Function Signature
```python
@inngest_client.create_function(
    fn_id="RAG: Ingest PDF",
    trigger=inngest.TriggerEvent(event="rag/ingest_pdf"),
    throttle=inngest.Throttle(
        count=2,
        period=datetime.timedelta(minutes=1)
    ),
    rate_limit=inngest.RateLimit(
        limit=1,
        period=datetime.timedelta(hours=4),
        key="event.data.source_id",
    ),
)
async def rag_ingest_pdf(ctx: inngest.Context):
    ...
```

#### 配置参数 / Configuration Parameters

| 参数 / Parameter | 值 / Value | 说明 / Description |
|-----------------|------------|-------------------|
| **Throttle** | 2次/分钟 / 2 per minute | 限制整体调用频率 / Limit overall call rate |
| **Rate Limit** | 1次/4小时 per source_id | 限制每个PDF文件的摄取频率 / Limit ingestion per PDF |
| **Key** | `event.data.source_id` | 按source_id分组限流 / Group rate limit by source_id |

#### 输入参数 / Input Parameters
```python
# 事件数据结构 / Event data structure
{
    "pdf_path": str,        # PDF文件路径 / PDF file path
    "source_id": str        # 来源ID（可选，默认为pdf_path）/ Source ID (optional)
}
```

**示例 / Example**:
```json
{
    "pdf_path": "/home/user/uploads/machine_learning.pdf",
    "source_id": "machine_learning.pdf"
}
```

#### 执行流程 / Execution Flow

```
Step 1: load-and-chunk (main.py:36-40)
├─ 读取PDF文件 / Read PDF file
├─ 提取文本 / Extract text
├─ 分块处理 / Chunk text
└─ 返回 RAGChunkAndSrc
    ↓
Step 2: embed-and-upsert (main.py:42-49)
├─ 调用OpenAI生成嵌入 / Call OpenAI for embeddings
├─ 生成UUID / Generate UUIDs
├─ 构造Payload / Construct payloads
└─ 存储到Qdrant / Store in Qdrant
    ↓
返回结果 / Return result
{"ingested": <chunk_count>}
```

#### 关键代码 / Key Code

**Step 1: 加载和分块 / Load and Chunk**
```python
def _load(ctx: inngest.Context) -> RAGChunkAndSrc:
    pdf_path = ctx.event.data["pdf_path"]
    source_id = ctx.event.data.get("source_id", pdf_path)
    chunks = load_and_chunk_pdf(pdf_path)
    return RAGChunkAndSrc(chunks=chunks, source_id=source_id)
```

**Step 2: 嵌入和存储 / Embed and Upsert**
```python
def _upsert(chunks_and_src: RAGChunkAndSrc) -> RAGUpsertResult:
    chunks = chunks_and_src.chunks
    source_id = chunks_and_src.source_id

    # 生成嵌入向量 / Generate embeddings
    vecs = embed_texts(chunks)

    # 生成确定性UUID / Generate deterministic UUIDs
    ids = [str(uuid.uuid5(uuid.NAMESPACE_URL, f"{source_id}:{i}"))
           for i in range(len(chunks))]

    # 构造payload / Construct payloads
    payloads = [{"source": source_id, "text": chunks[i]}
                for i in range(len(chunks))]

    # 存储到Qdrant / Store in Qdrant
    QdrantStorage().upsert(ids, vecs, payloads)

    return RAGUpsertResult(ingested=len(chunks))
```

#### 返回值 / Return Value
```python
# main.py:53
return ingested.model_dump()

# 示例 / Example
{
    "ingested": 42  # 成功摄取的文本块数量 / Number of chunks ingested
}
```

---

### 2. rag_query_pdf_ai（AI查询函数）

**位置 / Location**: `main.py:56-99`
**触发事件 / Trigger Event**: `rag/query_pdf_ai`
**函数ID / Function ID**: `"RAG: Query PDF"`

#### 函数签名 / Function Signature
```python
@inngest_client.create_function(
    fn_id="RAG: Query PDF",
    trigger=inngest.TriggerEvent(event="rag/query_pdf_ai")
)
async def rag_query_pdf_ai(ctx: inngest.Context):
    ...
```

#### 配置参数 / Configuration Parameters

| 参数 / Parameter | 值 / Value | 说明 / Description |
|-----------------|------------|-------------------|
| **Throttle** | ❌ 无 / None | 无节流限制 / No throttling |
| **Rate Limit** | ❌ 无 / None | 无限流限制 / No rate limiting |

**注意 / Note**: 查询函数没有限流，但可以根据需要添加。

#### 输入参数 / Input Parameters
```python
# 事件数据结构 / Event data structure
{
    "question": str,        # 用户问题 / User question
    "top_k": int           # 检索数量（可选，默认5）/ Number of results (optional, default 5)
}
```

**示例 / Example**:
```json
{
    "question": "What is machine learning?",
    "top_k": 5
}
```

#### 执行流程 / Execution Flow

```
Step 1: embed-and-search (main.py:61-70)
├─ 将问题转为向量 / Convert question to vector
├─ 在Qdrant中搜索 / Search in Qdrant
└─ 返回 RAGSearchResult
    ↓
Step 2: llm-answer (main.py:72-96)
├─ 构造提示词 / Construct prompt
│  ├─ 系统提示 / System prompt
│  ├─ 上下文块 / Context chunks
│  └─ 用户问题 / User question
├─ 调用OpenAI GPT-4o-mini / Call OpenAI GPT-4o-mini
└─ 提取答案 / Extract answer
    ↓
返回结果 / Return result
{"answer": <text>, "sources": [...], "num_contexts": <int>}
```

#### 关键代码 / Key Code

**Step 1: 搜索 / Search**
```python
def _search(question: str, top_k: int = 5) -> RAGSearchResult:
    # 问题嵌入 / Embed question
    query_vec = embed_texts([question])[0]

    # 向量搜索 / Vector search
    store = QdrantStorage()
    found = store.search(query_vec, top_k)

    return RAGSearchResult(
        contexts=found["contexts"],
        sources=found["sources"]
    )
```

**Step 2: LLM推理 / LLM Inference**
```python
# 构造提示词 / Construct prompt
context_block = "\n\n".join(f"- {c}" for c in found.contexts)
user_content = (
    "Use the following context to answer the question.\n\n"
    f"Context:\n{context_block}\n\n"
    f"Question: {question}\n"
    "Answer concisely using the context above."
)

# 配置OpenAI适配器 / Configure OpenAI adapter
adapter = ai.openai.Adapter(
    auth_key=os.getenv("OPENAI_API_KEY"),
    model="gpt-4o-mini"
)

# 调用AI / Call AI
res = await ctx.step.ai.infer(
    "llm-answer",
    adapter=adapter,
    body={
        "max_tokens": 1024,
        "temperature": 0.2,
        "messages": [
            {
                "role": "system",
                "content": "You answer questions using only the provided context."
            },
            {
                "role": "user",
                "content": user_content
            }
        ]
    }
)

# 提取答案 / Extract answer
answer = res["choices"][0]["message"]["content"].strip()
```

#### 返回值 / Return Value
```python
# main.py:99
return {
    "answer": answer,               # AI生成的答案 / AI-generated answer
    "sources": found.sources,       # 来源PDF列表 / List of source PDFs
    "num_contexts": len(found.contexts)  # 使用的上下文数量 / Number of contexts used
}
```

**示例 / Example**:
```json
{
    "answer": "Machine learning is a subset of artificial intelligence...",
    "sources": ["machine_learning.pdf", "ai_guide.pdf"],
    "num_contexts": 5
}
```

---

## 🔄 中间件和拦截器 / Middleware & Interceptors

### Inngest内置中间件 / Built-in Inngest Middleware

Inngest自动提供以下中间件功能：

#### 1. 限流中间件 / Rate Limiting Middleware
**位置 / Location**: `main.py:29-33`

```python
rate_limit=inngest.RateLimit(
    limit=1,                                    # 限制次数 / Limit count
    period=datetime.timedelta(hours=4),        # 时间周期 / Time period
    key="event.data.source_id",                # 分组键 / Grouping key
)
```

**工作原理 / How it works**:
- 按 `source_id` 分组 / Group by `source_id`
- 每个 `source_id` 每4小时最多1次 / Max 1 per `source_id` per 4 hours
- 超过限制返回429错误 / Return 429 if exceeded

#### 2. 节流中间件 / Throttling Middleware
**位置 / Location**: `main.py:26-28`

```python
throttle=inngest.Throttle(
    count=2,                                   # 最大并发数 / Max concurrent
    period=datetime.timedelta(minutes=1)       # 时间窗口 / Time window
)
```

**工作原理 / How it works**:
- 每分钟最多处理2个PDF摄取请求 / Max 2 PDF ingestions per minute
- 超过限制的请求会排队 / Excess requests are queued

#### 3. 重试中间件 / Retry Middleware
**默认配置 / Default Configuration**:
- Inngest默认启用自动重试 / Auto-retry enabled by default
- 指数退避策略 / Exponential backoff strategy
- 最多重试3次 / Max 3 retries

#### 4. 日志中间件 / Logging Middleware
**位置 / Location**: `main.py:18`

```python
logger=logging.getLogger("uvicorn")
```

**功能 / Features**:
- 自动记录函数执行 / Auto-log function executions
- 记录错误和异常 / Log errors and exceptions
- 集成到Inngest UI / Integrated into Inngest UI

---

## 🧩 业务逻辑层 / Business Logic Layer

### 数据加载模块 / Data Loader Module

**文件 / File**: `data_loader.py` (28行 / 28 lines)
**位置 / Location**: `/data_loader.py`

#### 1. load_and_chunk_pdf（PDF加载和分块）
**位置 / Location**: `data_loader.py:14-20`

```python
def load_and_chunk_pdf(path: str):
    # 使用LlamaIndex PDFReader读取PDF / Read PDF with LlamaIndex PDFReader
    docs = PDFReader().load_data(file=path)

    # 提取文本 / Extract text
    texts = [d.text for d in docs if getattr(d, "text", None)]

    # 分块 / Chunk text
    chunks = []
    for t in texts:
        chunks.extend(splitter.split_text(t))

    return chunks
```

**配置 / Configuration**:
```python
# data_loader.py:12
splitter = SentenceSplitter(
    chunk_size=1000,      # 每块最大1000字符 / Max 1000 chars per chunk
    chunk_overlap=200     # 块之间重叠200字符 / 200 chars overlap
)
```

#### 2. embed_texts（文本嵌入）
**位置 / Location**: `data_loader.py:23-28`

```python
def embed_texts(texts: list[str]) -> list[list[float]]:
    response = client.embeddings.create(
        model=EMBED_MODEL,       # "text-embedding-3-large"
        input=texts,
    )
    return [item.embedding for item in response.data]
```

**配置 / Configuration**:
```python
# data_loader.py:8-10
client = OpenAI()                          # OpenAI客户端 / OpenAI client
EMBED_MODEL = "text-embedding-3-large"    # 嵌入模型 / Embedding model
EMBED_DIM = 3072                          # 向量维度 / Vector dimension
```

---

### 向量数据库模块 / Vector Database Module

**文件 / File**: `vector_db.py` (37行 / 37 lines)
**详细分析 / Detailed Analysis**: 参见 `01-database-analysis.md` / See `01-database-analysis.md`

---

## 🌐 外部服务集成 / External Service Integration

### 1. OpenAI API集成

**用途 / Usage**:
- 文本嵌入生成 / Text embedding generation
- LLM推理 / LLM inference

#### 嵌入服务 / Embedding Service
**位置 / Location**: `data_loader.py:23-28`

| 参数 / Parameter | 值 / Value |
|-----------------|------------|
| **模型 / Model** | `text-embedding-3-large` |
| **维度 / Dimension** | `3072` |
| **API端点 / API Endpoint** | `https://api.openai.com/v1/embeddings` |
| **认证 / Authentication** | `OPENAI_API_KEY` 环境变量 / env var |

#### LLM服务 / LLM Service
**位置 / Location**: `main.py:80-96`

| 参数 / Parameter | 值 / Value |
|-----------------|------------|
| **模型 / Model** | `gpt-4o-mini` |
| **最大Token / Max Tokens** | `1024` |
| **温度 / Temperature** | `0.2` (更确定性 / More deterministic) |
| **API端点 / API Endpoint** | `https://api.openai.com/v1/chat/completions` |
| **认证 / Authentication** | `OPENAI_API_KEY` 环境变量 / env var |

**提示词结构 / Prompt Structure**:
```
System: "You answer questions using only the provided context."

User: "Use the following context to answer the question.

Context:
- <context_1>
- <context_2>
...

Question: <user_question>
Answer concisely using the context above."
```

---

### 2. Inngest服务集成

**用途 / Usage**: 工作流编排、事件处理、监控 / Workflow orchestration, event handling, monitoring

#### 本地开发服务器 / Local Dev Server
**位置 / Location**: `streamlit_app.py:77`

| 参数 / Parameter | 值 / Value |
|-----------------|------------|
| **API基础URL / API Base URL** | `http://127.0.0.1:8288/v1` |
| **环境变量 / Environment Variable** | `INNGEST_API_BASE` (可选 / optional) |

#### Inngest事件发送 / Inngest Event Sending
**位置 / Location**: `streamlit_app.py:30-40, 60-72`

```python
client = inngest.Inngest(app_id="rag_app", is_production=False)
await client.send(
    inngest.Event(
        name="rag/ingest_pdf",  # 或 "rag/query_pdf_ai" / or "rag/query_pdf_ai"
        data={...}
    )
)
```

---

### 3. Qdrant向量数据库集成

**详细分析 / Detailed Analysis**: 参见 `01-database-analysis.md` / See `01-database-analysis.md`

---

## 🔍 错误处理模式 / Error Handling Patterns

### 1. Inngest自动重试 / Inngest Auto-Retry

Inngest会自动重试失败的步骤：

```python
# main.py:51-52
chunks_and_src = await ctx.step.run("load-and-chunk", lambda: _load(ctx), ...)
ingested = await ctx.step.run("embed-and-upsert", lambda: _upsert(chunks_and_src), ...)
```

**重试策略 / Retry Strategy**:
- 自动重试最多3次 / Auto-retry up to 3 times
- 指数退避（1s, 2s, 4s）/ Exponential backoff (1s, 2s, 4s)
- 持久化步骤结果 / Persist step results

### 2. 防御性编程 / Defensive Programming

**位置 / Location**: `vector_db.py:30`

```python
payload = getattr(r, "payload", None) or {}
```

使用 `getattr` 和默认值防止属性不存在错误 / Use `getattr` with default to prevent attribute errors

**位置 / Location**: `data_loader.py:16`

```python
texts = [d.text for d in docs if getattr(d, "text", None)]
```

过滤掉没有文本的文档 / Filter out documents without text

### 3. 环境变量检查 / Environment Variable Checking

**位置 / Location**: `main.py:14, data_loader.py:6`

```python
load_dotenv()  # 确保环境变量加载 / Ensure env vars loaded
```

**注意 / Note**: 代码未显式检查 `OPENAI_API_KEY` 是否存在，如果缺失会在调用时抛出错误。

---

## 📊 API调用流程图 / API Call Flow Diagram

### PDF摄取流程 / PDF Ingestion Flow

```
Client (streamlit_app.py)
    │
    ├─ 上传PDF到 uploads/ / Upload PDF to uploads/
    │
    └─ 发送Inngest事件 / Send Inngest event
       POST /api/inngest
       Event: "rag/ingest_pdf"
       Data: {pdf_path, source_id}
            │
            ▼
    ┌───────────────────────────────────────┐
    │   Inngest Engine / Inngest引擎        │
    ├───────────────────────────────────────┤
    │ 1️⃣ 检查限流 / Check rate limit        │
    │    - source_id: 1次/4小时 / 1 per 4h  │
    │ 2️⃣ 检查节流 / Check throttle          │
    │    - 2次/分钟 / 2 per minute          │
    │ 3️⃣ 触发函数 / Trigger function        │
    └───────────────────────────────────────┘
            │
            ▼
    ┌───────────────────────────────────────┐
    │  rag_ingest_pdf (main.py:35)         │
    ├───────────────────────────────────────┤
    │ Step 1: load-and-chunk                │
    │  ├─ PDFReader.load_data()            │
    │  │   (llama_index)                   │
    │  └─ SentenceSplitter.split_text()    │
    │      (chunk_size=1000, overlap=200)  │
    │                                       │
    │ Step 2: embed-and-upsert              │
    │  ├─ embed_texts()                    │
    │  │   → OpenAI API                    │
    │  │   → text-embedding-3-large        │
    │  │   → 返回3072维向量 / Return 3072-d │
    │  ├─ 生成UUID / Generate UUIDs        │
    │  └─ QdrantStorage.upsert()           │
    │      → Qdrant向量数据库 / Qdrant DB   │
    └───────────────────────────────────────┘
            │
            ▼
    返回结果 / Return Result
    {"ingested": <chunk_count>}
```

### AI查询流程 / AI Query Flow

```
Client (streamlit_app.py)
    │
    └─ 发送Inngest事件 / Send Inngest event
       POST /api/inngest
       Event: "rag/query_pdf_ai"
       Data: {question, top_k}
            │
            ▼
    ┌───────────────────────────────────────┐
    │   Inngest Engine / Inngest引擎        │
    ├───────────────────────────────────────┤
    │ 1️⃣ 触发函数 / Trigger function        │
    │    (无限流 / No rate limiting)         │
    └───────────────────────────────────────┘
            │
            ▼
    ┌───────────────────────────────────────┐
    │  rag_query_pdf_ai (main.py:60)       │
    ├───────────────────────────────────────┤
    │ Step 1: embed-and-search              │
    │  ├─ embed_texts([question])          │
    │  │   → OpenAI API                    │
    │  │   → text-embedding-3-large        │
    │  └─ QdrantStorage.search()           │
    │      → Qdrant向量搜索 / Qdrant search │
    │      → 返回top_k个相似块 / Return top_k │
    │                                       │
    │ Step 2: llm-answer                    │
    │  ├─ 构造提示词 / Construct prompt     │
    │  │   System: "只用上下文回答"         │
    │  │   User: "Context + Question"      │
    │  └─ ctx.step.ai.infer()              │
    │      → OpenAI API                    │
    │      → gpt-4o-mini                   │
    │      → max_tokens=1024               │
    │      → temperature=0.2               │
    └───────────────────────────────────────┘
            │
            ▼
    返回结果 / Return Result
    {
      "answer": <AI答案 / AI answer>,
      "sources": [<PDF文件名 / PDF filenames>],
      "num_contexts": <数量 / count>
    }
```

---

## 🔐 认证和授权 / Authentication & Authorization

**当前状态 / Current Status**: ❌ **未实现** / **Not Implemented**

**说明 / Note**:
- 本项目是本地开发版本 / This is a local development version
- 没有用户认证 / No user authentication
- 没有API密钥验证 / No API key validation
- 任何人都可以访问Inngest端点 / Anyone can access Inngest endpoints

**生产环境建议 / Production Recommendations**:
1. 添加API密钥认证 / Add API key authentication
2. 使用Inngest的签名验证 / Use Inngest signature verification
3. 限制IP白名单 / IP whitelisting
4. 添加用户会话管理 / Add user session management

---

## 📈 性能优化 / Performance Optimization

### 1. 批量嵌入 / Batch Embedding
**位置 / Location**: `data_loader.py:23-28`

```python
# 一次性嵌入所有文本块，而不是逐个嵌入 / Embed all chunks at once, not one by one
response = client.embeddings.create(
    model=EMBED_MODEL,
    input=texts,  # List[str] - 批量处理 / Batch processing
)
```

**优势 / Advantages**:
- 减少API调用次数 / Reduce API calls
- 降低网络延迟 / Lower network latency
- 提高吞吐量 / Increase throughput

### 2. 步骤隔离 / Step Isolation
**位置 / Location**: `main.py:51-52`

```python
# 每个步骤独立执行和重试 / Each step executes and retries independently
chunks_and_src = await ctx.step.run("load-and-chunk", ...)
ingested = await ctx.step.run("embed-and-upsert", ...)
```

**优势 / Advantages**:
- 失败步骤独立重试 / Failed steps retry independently
- 不需要重新执行成功的步骤 / No need to re-execute successful steps

### 3. 异步执行 / Asynchronous Execution

所有函数都是异步的 / All functions are async:
```python
async def rag_ingest_pdf(ctx: inngest.Context):
async def rag_query_pdf_ai(ctx: inngest.Context):
```

**优势 / Advantages**:
- 非阻塞I/O / Non-blocking I/O
- 更高的并发处理能力 / Higher concurrency

---

## 🧪 调试和监控 / Debugging & Monitoring

### Inngest UI
**访问地址 / Access URL**: `http://127.0.0.1:8288`

**功能 / Features**:
- 📊 实时查看函数执行 / View function executions in real-time
- 🔍 检查步骤详情 / Inspect step details
- 📝 查看日志 / View logs
- ⚠️ 查看错误和重试 / View errors and retries
- 📈 性能指标 / Performance metrics

### 日志输出 / Log Output
**位置 / Location**: `main.py:18`

```python
logger=logging.getLogger("uvicorn")
```

日志会输出到Uvicorn服务器控制台 / Logs output to Uvicorn server console

---

## 🚀 启动命令 / Startup Commands

### 启动后端服务器 / Start Backend Server
```bash
uvicorn main:app --reload
```

**默认地址 / Default Address**: `http://127.0.0.1:8000`

### 启动Inngest开发服务器 / Start Inngest Dev Server
```bash
npx inngest-cli@latest dev
```

**默认地址 / Default Address**: `http://127.0.0.1:8288`

---

**文档生成完成 / Document Generated**: ✅
**下一步 / Next Step**: 03-frontend-analysis.md
