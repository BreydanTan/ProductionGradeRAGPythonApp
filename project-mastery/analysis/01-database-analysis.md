# 数据库分析 / Database Analysis

**生成时间 / Generated**: 2025-11-16
**数据库类型 / Database Type**: Qdrant 向量数据库 / Qdrant Vector Database

---

## 📋 数据库概览 / Database Overview

### 中文
本项目使用 **Qdrant** 作为向量数据库，这是一个开源的高性能向量搜索引擎。与传统关系型数据库不同，向量数据库专门用于存储和检索高维向量数据，非常适合语义搜索、推荐系统等AI应用。

**数据库特点**：
- 🔢 **向量维度**: 3072维（OpenAI text-embedding-3-large）
- 📊 **相似度计算**: 余弦相似度（Cosine Similarity）
- 🏗️ **存储方式**: 本地文件系统存储（`qdrant_storage/`目录）
- 🚀 **访问方式**: 通过QdrantClient连接本地或远程实例

### English
This project uses **Qdrant** as its vector database, an open-source high-performance vector search engine. Unlike traditional relational databases, vector databases are specifically designed to store and retrieve high-dimensional vector data, making them ideal for semantic search, recommendation systems, and other AI applications.

**Database Characteristics**:
- 🔢 **Vector Dimension**: 3072-dimensional (OpenAI text-embedding-3-large)
- 📊 **Similarity Metric**: Cosine Similarity
- 🏗️ **Storage**: Local filesystem (`qdrant_storage/` directory)
- 🚀 **Access**: Via QdrantClient to local or remote instance

---

## 🗄️ 数据库配置 / Database Configuration

### 连接配置 / Connection Configuration
**代码位置 / Code Location**: `vector_db.py:6-13`

```python
class QdrantStorage:
    def __init__(self, url="http://localhost:6333", collection="docs", dim=3072):
        self.client = QdrantClient(url=url, timeout=30)
        self.collection = collection
        if not self.client.collection_exists(self.collection):
            self.client.create_collection(
                collection_name=self.collection,
                vectors_config=VectorParams(size=dim, distance=Distance.COSINE),
            )
```

| 参数 / Parameter | 默认值 / Default | 说明 / Description |
|-----------------|------------------|-------------------|
| `url` | `http://localhost:6333` | Qdrant服务器地址 / Qdrant server URL |
| `collection` | `"docs"` | 集合名称 / Collection name |
| `dim` | `3072` | 向量维度 / Vector dimension |
| `timeout` | `30` | 连接超时（秒）/ Connection timeout (seconds) |
| `distance` | `Distance.COSINE` | 距离度量方式 / Distance metric |

---

## 📦 集合结构 / Collection Structure

### 主集合：`docs`
**位置 / Location**: `qdrant_storage/collections/docs/`

```
docs/
├── config.json                    # 集合配置 / Collection config
├── shard_key_mapping.json         # 分片映射 / Shard key mapping
└── 0/                             # 分片0 / Shard 0
    ├── shard_config.json
    ├── replica_state.json
    ├── newest_clocks.json
    └── segments/                  # 向量段 / Vector segments
        ├── a8146559-bf6e-4daa-9821-763326993201/
        ├── ffbf38c6-d662-4e55-a3db-5288a289f601/
        ├── 799646cf-0859-44b7-938a-f2a7dd699916/
        ├── 0a479712-b4ee-4eac-ae24-32781b54d7f4/
        ├── 5b6e23f9-632d-41ec-ad35-3bd6c6bf0fea/
        ├── da48a8c9-0e1f-4173-b3fd-8b7ede5fb29f/
        ├── 62205516-fdd8-4d26-ae0e-bc9c79076f59/
        └── ef1c5111-7282-4eb4-80b4-54bd2a6920db/
```

### 集合配置说明 / Collection Configuration

| 属性 / Property | 值 / Value | 说明 / Description |
|-----------------|------------|-------------------|
| **Collection Name** | `docs` | 文档集合 / Documents collection |
| **Vector Size** | `3072` | 使用OpenAI text-embedding-3-large模型 / Using OpenAI text-embedding-3-large |
| **Distance Metric** | `COSINE` | 余弦相似度，范围0-2，值越小越相似 / Cosine similarity, range 0-2, smaller is more similar |
| **Segments** | `8` | 当前有8个向量段 / Currently 8 vector segments |

---

## 🗂️ 数据模型 / Data Models

### 1. Point（向量点）结构

每个存储在Qdrant中的向量点包含：

```
Point {
    id: UUID (字符串形式 / String UUID)
    vector: List[float] (3072维 / 3072-dimensional)
    payload: Dict {
        "source": str,      # PDF文件名 / PDF filename
        "text": str         # 文本块内容 / Text chunk content
    }
}
```

**示例 / Example**:
```python
{
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "vector": [0.123, -0.456, 0.789, ... ],  # 3072个浮点数 / 3072 floats
    "payload": {
        "source": "machine_learning.pdf",
        "text": "Machine learning is a subset of artificial intelligence..."
    }
}
```

### 2. Pydantic数据模型

**代码位置 / Code Location**: `custom_types.py`

#### RAGChunkAndSrc
```python
class RAGChunkAndSrc(pydantic.BaseModel):
    chunks: list[str]        # 文本块列表 / List of text chunks
    source_id: str = None    # 来源ID（PDF文件名）/ Source ID (PDF filename)
```

**用途 / Usage**: 在PDF加载和分块后，传递给嵌入步骤
**使用位置 / Used in**: `main.py:36-40, 51`

#### RAGUpsertResult
```python
class RAGUpsertResult(pydantic.BaseModel):
    ingested: int            # 成功摄取的文本块数量 / Number of chunks ingested
```

**用途 / Usage**: 返回PDF摄取结果
**使用位置 / Used in**: `main.py:42-49, 52`

#### RAGSearchResult
```python
class RAGSearchResult(pydantic.BaseModel):
    contexts: list[str]      # 检索到的文本块 / Retrieved text chunks
    sources: list[str]       # 来源文件列表 / List of source files
```

**用途 / Usage**: 返回向量搜索结果
**使用位置 / Used in**: `main.py:61-65, 70`

#### RAQQueryResult
```python
class RAQQueryResult(pydantic.BaseModel):
    answer: str              # AI生成的答案 / AI-generated answer
    sources: list[str]       # 来源文件 / Source files
    num_contexts: int        # 使用的上下文数量 / Number of contexts used
```

**用途 / Usage**: 返回最终查询结果（注意：代码中定义了但未直接使用）
**使用位置 / Used in**: `main.py:99` (返回值手动构造)

---

## 🔄 数据流程 / Data Flow

### 1. 数据创建流程（Upsert）/ Data Creation (Upsert)

```
PDF文档 (uploads/)
    ↓
[1] load_and_chunk_pdf (data_loader.py:14-20)
    - 使用PDFReader读取 / Read with PDFReader
    - 使用SentenceSplitter分块 / Chunk with SentenceSplitter
    │   chunk_size: 1000字符 / 1000 characters
    │   chunk_overlap: 200字符 / 200 characters overlap
    ↓
文本块列表 (chunks: list[str])
    ↓
[2] embed_texts (data_loader.py:23-28)
    - 调用OpenAI API / Call OpenAI API
    - 模型: text-embedding-3-large / Model: text-embedding-3-large
    - 返回: List[List[float]] (每个3072维) / Returns: 3072-dim vectors
    ↓
向量列表 (vectors: list[list[float]])
    ↓
[3] 生成ID和Payload (main.py:46-47)
    - ID生成方式: uuid.uuid5(NAMESPACE_URL, f"{source_id}:{i}")
    - Payload: {"source": source_id, "text": chunk_text}
    ↓
[4] QdrantStorage.upsert (vector_db.py:15-17)
    - 构造PointStruct对象 / Create PointStruct objects
    - 批量插入Qdrant / Batch upsert to Qdrant
    ↓
✅ 存储完成 / Storage Complete
```

**关键代码 / Key Code**:
```python
# main.py:46-48
ids = [str(uuid.uuid5(uuid.NAMESPACE_URL, f"{source_id}:{i}")) for i in range(len(chunks))]
payloads = [{"source": source_id, "text": chunks[i]} for i in range(len(chunks))]
QdrantStorage().upsert(ids, vecs, payloads)
```

### 2. 数据查询流程（Search）/ Data Query (Search)

```
用户问题 (question: str)
    ↓
[1] embed_texts (data_loader.py:23-28)
    - 将问题转换为向量 / Convert question to vector
    - 返回: List[float] (3072维) / Returns: 3072-dim vector
    ↓
查询向量 (query_vector: list[float])
    ↓
[2] QdrantStorage.search (vector_db.py:19-36)
    - 使用余弦相似度搜索 / Search with cosine similarity
    - top_k: 默认5个最相似结果 / Default top 5 results
    - with_payload=True: 返回完整payload / Return full payload
    ↓
搜索结果 (results: list[ScoredPoint])
    ↓
[3] 提取上下文和来源 (vector_db.py:26-35)
    contexts = [r.payload["text"] for r in results]
    sources = set([r.payload["source"] for r in results])
    ↓
✅ 返回 {"contexts": [...], "sources": [...]}
```

**关键代码 / Key Code**:
```python
# vector_db.py:19-25
results = self.client.search(
    collection_name=self.collection,
    query_vector=query_vector,
    with_payload=True,
    limit=top_k
)
```

### 3. 数据更新流程 / Data Update Flow

**重要说明 / Important Note**: 本项目使用 `upsert` 操作，这意味着：
- 如果ID已存在 → 更新向量和payload / If ID exists → Update vector and payload
- 如果ID不存在 → 创建新记录 / If ID doesn't exist → Create new record

由于ID是基于 `source_id` 和索引生成的（`uuid.uuid5(NAMESPACE_URL, f"{source_id}:{i}")`），重新上传相同的PDF会覆盖旧数据。

### 4. 数据删除流程 / Data Deletion Flow

**当前状态 / Current Status**: ❌ 项目中**未实现**删除功能 / Deletion **NOT implemented**

**潜在实现方式 / Potential Implementation**:
```python
# 可以通过以下方式删除 / Can delete with:
QdrantClient.delete(
    collection_name="docs",
    points_selector={"filter": {"must": [{"key": "source", "match": {"value": "file.pdf"}}]}}
)
```

---

## 📊 数据关系图 / Data Relationship Diagram

### ER图（实体关系图）/ Entity-Relationship Diagram

虽然向量数据库不是传统的关系型数据库，但我们可以用ER图表示数据结构：

```
┌─────────────────────────────────────────────────────────────┐
│                     PDF Document                            │
│                     PDF文档                                 │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Attributes / 属性:                                   │   │
│  │ • filename: str (例: "ml_guide.pdf")                │   │
│  │ • upload_time: datetime                             │   │
│  │ • file_path: Path (例: "uploads/ml_guide.pdf")      │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │ (1对多 / 1-to-Many)
                        │ 一个PDF → 多个文本块
                        │ One PDF → Multiple Chunks
                        ↓
┌─────────────────────────────────────────────────────────────┐
│                  Text Chunk (在Qdrant中的Point)             │
│                  文本块（Qdrant中的点）                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Primary Key / 主键:                                  │   │
│  │ • id: UUID (生成规则: uuid5(source_id + index))      │   │
│  │                                                      │   │
│  │ Vector / 向量:                                       │   │
│  │ • embedding: List[float] (3072维)                   │   │
│  │                                                      │   │
│  │ Payload / 负载:                                      │   │
│  │ • source: str (外键 → PDF filename)                 │   │
│  │ • text: str (文本内容，最多~1000字符)                │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                        │
                        │ (多对多 / Many-to-Many)
                        │ 多个块 → 多个查询
                        │ Multiple chunks → Multiple queries
                        ↓
┌─────────────────────────────────────────────────────────────┐
│                      User Query                             │
│                      用户查询                                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │ Attributes / 属性:                                   │   │
│  │ • question: str                                     │   │
│  │ • query_embedding: List[float] (3072维)            │   │
│  │ • top_k: int (检索数量，默认5)                       │   │
│  │                                                      │   │
│  │ Results / 结果:                                      │   │
│  │ • matched_chunks: List[TextChunk]                   │   │
│  │ • sources: List[str] (去重的PDF文件名)               │   │
│  │ • answer: str (AI生成的答案)                        │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

### 关系说明 / Relationship Explanation

1. **PDF → Chunks (1对多 / 1-to-Many)**
   - 一个PDF文档被分割成多个文本块
   - 每个块通过 `payload.source` 字段关联到原始PDF
   - 关系维护方式：payload中的 "source" 字段

2. **Chunks → Queries (多对多 / Many-to-Many)**
   - 一个查询可以匹配多个文本块（top_k个）
   - 一个文本块可以被多个不同查询匹配
   - 关系维护方式：余弦相似度计算

---

## 🔍 数据验证规则 / Data Validation Rules

### 1. 向量维度验证 / Vector Dimension Validation
**位置 / Location**: `vector_db.py:12`

```python
vectors_config=VectorParams(size=dim, distance=Distance.COSINE)
```

- ✅ **必须**: 所有向量必须是3072维 / All vectors must be 3072-dimensional
- ❌ **拒绝**: 不同维度的向量会被拒绝 / Vectors with different dimensions will be rejected

### 2. ID唯一性 / ID Uniqueness
**位置 / Location**: `main.py:46`

```python
ids = [str(uuid.uuid5(uuid.NAMESPACE_URL, f"{source_id}:{i}")) for i in range(len(chunks))]
```

- ✅ **确定性生成**: 相同的source_id和索引总是生成相同的UUID / Deterministic: same source_id + index = same UUID
- ✅ **幂等性**: 重新上传相同PDF会覆盖旧数据 / Idempotent: re-uploading same PDF overwrites old data

### 3. Payload验证 / Payload Validation
**位置 / Location**: `main.py:47`

```python
payloads = [{"source": source_id, "text": chunks[i]} for i in range(len(chunks))]
```

- ✅ **必需字段**: "source" 和 "text" / Required fields: "source" and "text"
- ✅ **类型**: 两者都必须是字符串 / Type: both must be strings

---

## 📈 数据统计 / Data Statistics

### 当前数据库状态 / Current Database State

**代码位置 / Code Location**: `qdrant_storage/collections/docs/`

| 指标 / Metric | 值 / Value |
|--------------|------------|
| **集合数量 / Collections** | 1 ("docs") |
| **分片数量 / Shards** | 1 (shard 0) |
| **向量段数量 / Vector Segments** | 8 |
| **存储位置 / Storage Location** | `qdrant_storage/` (本地文件系统 / Local filesystem) |

---

## 🛠️ 数据库操作示例 / Database Operation Examples

### 示例1：插入数据 / Example 1: Insert Data

```python
# 位置 / Location: main.py:42-49

def _upsert(chunks_and_src: RAGChunkAndSrc) -> RAGUpsertResult:
    chunks = chunks_and_src.chunks
    source_id = chunks_and_src.source_id

    # 生成嵌入 / Generate embeddings
    vecs = embed_texts(chunks)

    # 生成ID / Generate IDs
    ids = [str(uuid.uuid5(uuid.NAMESPACE_URL, f"{source_id}:{i}"))
           for i in range(len(chunks))]

    # 构造Payload / Construct payload
    payloads = [{"source": source_id, "text": chunks[i]}
                for i in range(len(chunks))]

    # 插入Qdrant / Insert to Qdrant
    QdrantStorage().upsert(ids, vecs, payloads)

    return RAGUpsertResult(ingested=len(chunks))
```

### 示例2：查询数据 / Example 2: Query Data

```python
# 位置 / Location: main.py:61-65

def _search(question: str, top_k: int = 5) -> RAGSearchResult:
    # 问题转向量 / Question to vector
    query_vec = embed_texts([question])[0]

    # 向量搜索 / Vector search
    store = QdrantStorage()
    found = store.search(query_vec, top_k)

    return RAGSearchResult(
        contexts=found["contexts"],
        sources=found["sources"]
    )
```

### 示例3：批量检索 / Example 3: Batch Retrieval

```python
# 位置 / Location: vector_db.py:26-36

contexts = []
sources = set()

for r in results:
    payload = getattr(r, "payload", None) or {}
    text = payload.get("text", "")
    source = payload.get("source", "")

    if text:
        contexts.append(text)
        sources.add(source)

return {"contexts": contexts, "sources": list(sources)}
```

---

## 🔐 数据索引 / Data Indexes

### 向量索引 / Vector Index
**类型 / Type**: HNSW (Hierarchical Navigable Small World)
**用途 / Purpose**: 加速高维向量的近似最近邻搜索 / Accelerate approximate nearest neighbor search

### Payload索引 / Payload Index
**位置 / Location**: `qdrant_storage/collections/docs/0/segments/*/payload_index/`
**索引字段 / Indexed Fields**:
- `source` (字符串 / String)

---

## 💡 数据库最佳实践 / Database Best Practices

### 1. ID生成策略 / ID Generation Strategy
✅ **优点 / Advantages**:
- 确定性：相同输入产生相同ID / Deterministic: same input → same ID
- 避免重复：重新上传自动更新 / Avoid duplicates: re-upload auto-updates

❌ **潜在问题 / Potential Issues**:
- 如果PDF内容变化但文件名相同，旧数据会被覆盖 / If PDF content changes but filename stays same, old data is overwritten

### 2. 文本分块策略 / Text Chunking Strategy
**参数 / Parameters**: `chunk_size=1000, chunk_overlap=200`

✅ **优点 / Advantages**:
- 重叠确保上下文连续性 / Overlap ensures context continuity
- 1000字符适合大多数查询 / 1000 chars suitable for most queries

### 3. 向量维度选择 / Vector Dimension Selection
**选择 / Choice**: 3072维（text-embedding-3-large）

✅ **优点 / Advantages**:
- 高维度 = 更精确的语义表示 / Higher dimension = more precise semantics
- OpenAI最强嵌入模型 / OpenAI's most powerful embedding model

❌ **权衡 / Trade-offs**:
- 更大的存储空间 / Larger storage space
- 更长的查询时间 / Longer query time

---

## 📚 迁移历史 / Migration History

**当前状态 / Current Status**: ❌ 无迁移系统 / No migration system

**说明 / Note**: Qdrant集合在首次运行时自动创建（`vector_db.py:9-13`）。如果需要更改配置（如向量维度），需要：
1. 删除 `qdrant_storage/` 目录
2. 重新运行应用以重新创建集合
3. 重新上传所有PDF

---

**文档生成完成 / Document Generated**: ✅
**下一步 / Next Step**: 02-backend-analysis.md
