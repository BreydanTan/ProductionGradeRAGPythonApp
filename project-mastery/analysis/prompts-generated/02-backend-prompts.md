# 后端重建提示词 / Backend Rebuild Prompts

**用途 / Purpose**: 重建FastAPI + Inngest后端

---

## 提示词 2.1：创建数据模型 / Prompt 2.1: Create Data Models

```
创建 custom_types.py 文件，定义以下Pydantic数据模型：

```python
import pydantic

class RAGChunkAndSrc(pydantic.BaseModel):
    chunks: list[str]
    source_id: str = None

class RAGUpsertResult(pydantic.BaseModel):
    ingested: int

class RAGSearchResult(pydantic.BaseModel):
    contexts: list[str]
    sources: list[str]

class RAQQueryResult(pydantic.BaseModel):
    answer: str
    sources: list[str]
    num_contexts: int
```

说明：这些模型用于Inngest函数之间的数据传递。
```

---

## 提示词 2.2：创建向量数据库封装 / Prompt 2.2: Create Vector DB Wrapper

```
创建 vector_db.py 文件，封装Qdrant向量数据库操作：

```python
from qdrant_client import QdrantClient
from qdrant_client.models import VectorParams, Distance, PointStruct

class QdrantStorage:
    def __init__(self, url="http://localhost:6333", collection="docs", dim=3072):
        self.client = QdrantClient(url=url, timeout=30)
        self.collection = collection
        if not self.client.collection_exists(self.collection):
            self.client.create_collection(
                collection_name=self.collection,
                vectors_config=VectorParams(size=dim, distance=Distance.COSINE),
            )

    def upsert(self, ids, vectors, payloads):
        points = [PointStruct(id=ids[i], vector=vectors[i], payload=payloads[i]) for i in range(len(ids))]
        self.client.upsert(self.collection, points=points)

    def search(self, query_vector, top_k: int = 5):
        results = self.client.search(
            collection_name=self.collection,
            query_vector=query_vector,
            with_payload=True,
            limit=top_k
        )
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

说明：
- dim=3072 对应 OpenAI text-embedding-3-large
- 使用余弦相似度（COSINE）进行搜索
```

---

## 提示词 2.3：创建数据加载模块 / Prompt 2.3: Create Data Loader

```
创建 data_loader.py 文件，实现PDF加载和文本嵌入：

```python
from openai import OpenAI
from llama_index.readers.file import PDFReader
from llama_index.core.node_parser import SentenceSplitter
from dotenv import load_dotenv

load_dotenv()

client = OpenAI()
EMBED_MODEL = "text-embedding-3-large"
EMBED_DIM = 3072

splitter = SentenceSplitter(chunk_size=1000, chunk_overlap=200)

def load_and_chunk_pdf(path: str):
    docs = PDFReader().load_data(file=path)
    texts = [d.text for d in docs if getattr(d, "text", None)]
    chunks = []
    for t in texts:
        chunks.extend(splitter.split_text(t))
    return chunks

def embed_texts(texts: list[str]) -> list[list[float]]:
    response = client.embeddings.create(
        model=EMBED_MODEL,
        input=texts,
    )
    return [item.embedding for item in response.data]
```

说明：
- chunk_size=1000: 每个文本块最多1000字符
- chunk_overlap=200: 块之间重叠200字符以保持上下文连续性
- 使用OpenAI的text-embedding-3-large模型（3072维）
```

---

## 提示词 2.4：创建FastAPI + Inngest主文件 / Prompt 2.4: Create Main File

```
创建 main.py 文件，实现FastAPI应用和Inngest工作流：

（由于代码较长，参考 02-backend-analysis.md 中的完整代码）

关键点：
1. 初始化Inngest客户端（app_id="rag_app"）
2. 创建两个Inngest函数：
   - rag_ingest_pdf: PDF摄取，带限流和节流
   - rag_query_pdf_ai: AI查询，无限流
3. 使用ctx.step.run()分步执行，支持自动重试
4. 使用ctx.step.ai.infer()调用OpenAI GPT-4o-mini

限流配置：
- PDF摄取：每source_id每4小时1次，全局每分钟2次
- AI查询：无限流（可根据需要添加）

参考文件：project-mastery/analysis/02-backend-analysis.md
```

---

**文档生成完成 / Document Generated**: ✅
