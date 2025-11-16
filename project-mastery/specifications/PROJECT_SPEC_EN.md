# Production-Ready RAG Python Application - Complete Project Specification

**Last Updated**: 2025-11-16
**Document Version**: 1.0
**Project Name**: Production-Ready RAG Python Application
**Target Audience**: All developers and technical decision makers

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Technical Architecture](#technical-architecture)
3. [Database Design](#database-design)
4. [Core Business Processes](#core-business-processes)
5. [API Interface Documentation](#api-interface-documentation)
6. [Environment Configuration Guide](#environment-configuration-guide)
7. [Local Development Guide](#local-development-guide)
8. [Deployment Guide](#deployment-guide)
9. [Frequently Asked Questions (FAQ)](#frequently-asked-questions-faq)
10. [Terminology Reference](#terminology-reference)

---

## Project Overview

### One-Line Description

This is an AI application that lets you upload PDF documents and then ask questions in natural language to get answers. It's like having an intelligent assistant that can understand all your documents and quickly answer questions about them.

### Core Features

Imagine you have a huge library. What this application does is:

1. **📄 Receive Documents** - You upload PDF files (like putting books into the library)
2. **🔍 Understand Content** - AI reads and comprehends document content (similar to how a librarian remembers key information from each book)
3. **💬 Answer Questions** - You ask questions in natural language, and AI finds relevant content from documents and answers you

**Complete Feature List**:

| Feature | Description | Analogy |
|---------|-------------|---------|
| PDF Upload and Processing | Support uploading PDF files and automatic analysis | Adding books to the library |
| Intelligent Chunking | Automatically split long documents into small chunks | Taking notes on book pages for easy reference |
| Vector Storage | Convert text to digital format for storage | Creating an index for the library |
| Semantic Search | Find relevant content by meaning, not keywords | Finding books by concept rather than exact word matching |
| AI Q&A | Use ChatGPT to generate answers | Ask the librarian to explain a topic in their own words |
| Production-Grade Protection | Automatic retry, rate limiting, monitoring | Security system and access control for the library |

### Technology Stack Overview

```
User Interface (Web)
    ↓
Python Backend (FastAPI)
    ↓
Event System (Inngest)
    ↓
Vector Database (Qdrant)
    ↓
AI Model (OpenAI)
```

---

## Technical Architecture

### System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                      User / User                                │
│                  (Browser Access Application)                   │
└───────────────────────────┬─────────────────────────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                   Streamlit Web Interface                        │
│          (streamlit_app.py - Frontend User Interface)            │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │  Top Section: PDF Upload Area                           │   │
│  │  ├─ File Picker: Select PDF file                        │   │
│  │  └─ Upload Button: Start uploading                      │   │
│  │                                                          │   │
│  │  Bottom Section: Question Area                          │   │
│  │  ├─ Input Box: Enter your question                      │   │
│  │  ├─ Slider: Select how many relevant paragraphs (1-20)  │   │
│  │  └─ Submit Button: Get answer                           │   │
│  │                                                          │   │
│  │  Result Display Area                                    │   │
│  │  ├─ AI-generated answer                                 │   │
│  │  └─ Source file list                                    │   │
│  └──────────────────────────────────────────────────────────┘   │
└───────────────────────────┬───────────────────┬───────────────────┘
                        │                   │
                   (Inngest Events / Events)│
                        │                   │
                        ▼                   ▼
┌──────────────────────────────────────────────────────────────────┐
│            Inngest Event-Driven Engine                           │
│     (Automatic handling of rate limiting, retries, monitoring)   │
│  ┌────────────────────────────────────┐                          │
│  │ Control Layer (Rate Limiting & Throttling) │                  │
│  │ • Maximum 1 PDF per filename per 4 hours   │                  │
│  │ • Maximum 2 upload requests per minute     │                  │
│  └────────────────────────────────────┘                          │
│  ┌────────────────────────────────────┐                          │
│  │ Event Routing                       │                          │
│  │ ├─ rag/ingest_pdf (Upload and process) │                      │
│  │ └─ rag/query_pdf_ai (Q&A)           │                          │
│  └────────────────────────────────────┘                          │
└───────────┬──────────────────────────────┬───────────────────────┘
            │                              │
            ▼                              ▼
┌──────────────────────────┐    ┌──────────────────────────┐
│  FastAPI Backend Core    │    │  Search Engine           │
│  Logic (main.py)         │    │  (vector_db.py)          │
│                          │    │                          │
│  PDF Processing Steps:  │    │  Search Relevant Passages │
│  1. Read PDF file        │    │  1. Embed question       │
│  2. Chunk processing     │    │  2. Vector similarity    │
│     (1000 characters)    │    │     query                │
│  3. Generate embeddings  │    │  3. Return top_k passages│
│  4. Store to vector DB   │    │                          │
└──────────────┬───────────┘    └──────────────┬───────────┘
               │                               │
               │            ┌──────────────────┘
               │            │
               ▼            ▼
┌──────────────────────────────────────┐
│    Qdrant Vector Database             │
│    (qdrant_storage/)                 │
│                                      │
│  Collection "docs"                    │
│  ├─ Document paragraph vectors (3072D) │
│  ├─ Paragraph text content             │
│  └─ Source file information            │
└────────────┬─────────────────────────┘
             │
      ▲      │
      │      ▼
      │  ┌──────────────────┐
      │  │ OpenAI API       │
      │  │                  │
      │  │ • Text embedding │
      │  │   (embedding)    │
      │  │                  │
      │  │ • ChatGPT answer │
      │  │   (gpt-4o-mini)  │
      │  └──────────────────┘
      │
      └─ Network Request
```

### Module Description

The project contains 5 core Python files:

```
ProductionGradeRAGPythonApp/
│
├── main.py                  # 🟡 Backend Server (103 lines)
│   ├─ FastAPI application entry
│   ├─ Inngest function definition
│   └─ Rate limiting and throttling configuration
│
├── streamlit_app.py         # 🟡 Frontend Interface (127 lines)
│   ├─ Web user interface
│   ├─ Event sending logic
│   └─ Result presentation
│
├── vector_db.py             # 🟡 Vector Database (37 lines)
│   ├─ Qdrant connection management
│   └─ Search functionality
│
├── data_loader.py           # 🟡 Data Processing (28 lines)
│   ├─ PDF reading
│   ├─ Text chunking
│   └─ Vector generation
│
└── custom_types.py          # 🟡 Data Models (21 lines)
    └─ Pydantic model definitions
```

### Data Flow Diagram

#### Process 1: PDF Upload and Processing

```
👤 User selects PDF file
    │
    ▼
📤 Streamlit uploader
    │ Save file to uploads/
    │
    ▼
📨 Send Inngest event
    │ Event: "rag/ingest_pdf"
    │
    ▼
⏳ Inngest Engine
    │ Check rate limit: 1 per file per 4 hours
    │ Check throttling: max 2 per minute
    │
    ▼
📖 PDF Reading (data_loader.py)
    │ • Read PDF with LlamaIndex
    │ • Chunk with SentenceSplitter
    │   - Chunk size: 1000 characters
    │   - Overlap: 200 characters
    │
    ▼
🔢 Generate Embeddings (OpenAI API)
    │ • Model: text-embedding-3-large
    │ • Dimension: 3072
    │ • Batch processing: embed all chunks at once
    │
    ▼
💾 Store to Qdrant
    │ • Generate unique ID (UUID)
    │ • Store vector and raw text
    │ • Record source filename
    │
    ▼
✅ Return Success
    │ Tell user how many paragraphs were uploaded
```

#### Process 2: Question Answering

```
💬 User inputs question
    │
    ▼
📨 Send Inngest event
    │ Event: "rag/query_pdf_ai"
    │ Data: {question, top_k}
    │
    ▼
🔍 Embed Question (OpenAI API)
    │ Convert question to 3072-dimensional vector
    │
    ▼
🔎 Qdrant Search
    │ • Calculate cosine similarity
    │ • Return most similar top_k passages
    │ • Sort by similarity score
    │
    ▼
🧠 Generate Answer (GPT-4o-mini)
    │ Input:
    │ ├─ System prompt: "Only answer using provided passages"
    │ ├─ Relevant passages (top_k)
    │ └─ User question
    │
    ▼
💭 Return Answer
    │ ├─ AI-generated text
    │ ├─ Source file list
    │ └─ Number of passages used
```

---

## Database Design

### Database Type

This project uses a **Vector Database**, not a traditional SQL database.

**Why Vector Database?**
- Traditional databases store data; vector databases store "data meaning"
- Similar to brain memory: remembers concepts, not words
- Supports "semantic search": find results by meaning, not keywords

### Data Model

#### Main Data Structure

```
One Document (PDF)
    │
    ├─ Split into N passages
    │
    └─ Each passage contains:
        ├─ ID (unique identifier)
        ├─ Text content (around 1000 characters)
        ├─ Vector (3072 numbers)
        └─ Source filename
```

#### Qdrant Collection Structure

```
Collection Name: "docs"
├─ Vector Dimension: 3072 (OpenAI text-embedding-3-large)
├─ Distance Metric: Cosine Similarity
│   └─ Range: 0-2
│      0 = identical
│      2 = completely different
│
└─ Storage Location: qdrant_storage/ (local filesystem)
```

#### Data Models (Pydantic)

```python
# Input: PDF and chunk information
class RAGChunkAndSrc(BaseModel):
    chunks: list[str]       # List of text chunks
    source_id: str          # PDF filename

# Output: Upload result
class RAGUpsertResult(BaseModel):
    ingested: int           # How many chunks were uploaded

# Output: Search result
class RAGSearchResult(BaseModel):
    contexts: list[str]     # Found text chunks
    sources: list[str]      # Source file list

# Output: Final answer
class RAGQueryResult(BaseModel):
    answer: str             # AI-generated answer
    sources: list[str]      # Source files
    num_contexts: int       # Number of chunks used
```

### Data Relationship Diagram

```
PDF Document (uploads/)
    │
    ├─ Filename: "machine_learning.pdf"
    ├─ Size: 1.5MB
    └─ Upload Time: 2025-11-16
        │
        │ (One-to-many relationship)
        │
        ▼
┌─────────────────────────────────┐
│ Text Chunk (Point in Qdrant)    │
│                                 │
│ ID: uuid5(filename + index)     │
│ Text: "Machine learning is..."  │
│ Vector: [0.123, -0.456, ...]    │
│ Source: "machine_learning.pdf"  │
└─────────────────────────────────┘
        │
        │ (Many-to-many relationship)
        │
        ▼
User Query
│ Question: "What is supervised learning?"
└─ Qdrant returns most similar 5 chunks
   - Chunk 1 (similarity 0.8)
   - Chunk 2 (similarity 0.75)
   - ...
```

### Data Integrity

**ID Generation Strategy (Deterministic UUID)**:
```
ID = SHA-UUID(filename + chunk_index)
Example: uuid5(filename="ai.pdf", index=0)

Advantages:
✓ Same file produces same ID
✓ Duplicate uploads auto-overwrite
✓ Avoids duplicate data

Disadvantages:
✗ Cannot have multiple versions simultaneously
```

**No Foreign Key Constraints**:
- Vector databases don't support foreign keys
- Rely on "source" field in payload for association
- Vectors persist after PDF deletion (manual cleanup needed)

---

## Core Business Processes

### Process 1: PDF Upload Flow

```
Timeline: From user selecting file to completion
═══════════════════════════════════════════════════════════════

User Interface Layer (Streamlit Frontend)
┌────────────────────────────────┐
│ Step 1: User clicks "Choose a PDF"  │ ← Open file selection dialog
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ Step 2: User selects PDF        │ ← File object passed to Streamlit
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ Step 3: Save file locally       │ ← uploads/filename.pdf
│        save_uploaded_pdf()      │
└────────────┬───────────────────┘
             │
             ▼
┌────────────────────────────────┐
│ Step 4: Display loading animation │ ← with st.spinner(...)
│        Send Inngest event       │ ← asyncio.run(...)
└────────────┬───────────────────┘
             │
             ▼ (Network Request)

Event Processing Layer (Inngest + FastAPI Backend)
┌────────────────────────────────────────────┐
│ Step 5: Inngest receives "rag/ingest_pdf" event │
│        Check rate limiting                 │
│        • Frequency: max 1 per source_id    │
│          per 4 hours                       │
│        • Throughput: max 2 per minute      │
└────────────┬───────────────────────────────┘
             │
             ├─ If throttled → Return to queue
             │
             ▼
┌────────────────────────────────────────────┐
│ Step 6: Load and Chunk (load_and_chunk_pdf) │
│        • PDFReader.load_data()             │
│        • Extract all text                  │
│        • SentenceSplitter.split_text()     │
│          - chunk_size: 1000 characters     │
│          - overlap: 200 characters         │
│        • Return chunks list                │
└────────────┬───────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────┐
│ Step 7: Generate embeddings (embed_texts)  │
│        • Call OpenAI API                   │
│        • Model: text-embedding-3-large     │
│        • Return List[List[float]] (3072D)  │
│        (Batch processing, one call)        │
└────────────┬───────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────┐
│ Step 8: Construct Point objects            │
│        • ID: uuid5(source_id + index)      │
│        • Vector: 3072-dimensional vector   │
│        • Payload: {source, text}           │
└────────────┬───────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────┐
│ Step 9: Batch Upsert to Qdrant             │
│        QdrantStorage.upsert(...)           │
│        • If ID exists, update              │
│        • If ID doesn't exist, create       │
└────────────┬───────────────────────────────┘
             │
             ▼ (Return Result)

User Feedback Layer (Streamlit Frontend)
┌────────────────────────────────────────────┐
│ Step 10: Display success message            │
│         st.success("Triggered ingestion...") │
│         Show how many chunks uploaded        │
└────────────────────────────────────────────┘
             │
             ▼
┌────────────────────────────────────────────┐
│ Step 11: Optional next steps                │
│         "You can upload another PDF..."    │
│         Continue uploading or start asking  │
└────────────────────────────────────────────┘

===== Complete Time Example =====
User selects file        2025-11-16 10:00:00
File upload completes    2025-11-16 10:00:01
Inngest event sent       2025-11-16 10:00:02
PDF reading and chunking 2025-11-16 10:00:03  (1 second)
OpenAI embedding         2025-11-16 10:00:08  (5 seconds, depends on document size)
Qdrant storage complete  2025-11-16 10:00:09  (1 second)
User sees success        2025-11-16 10:00:10

Total Time: 10 seconds
```

### Process 2: Question and Answer Flow

```
Timeline: From user inputting question to seeing answer
═══════════════════════════════════════════════════════════════

User Interface Layer
┌──────────────────────────────────┐
│ Step 1: User inputs question      │ ← "What is supervised learning?"
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│ Step 2: User sets top_k (optional) │ ← Default 5, range 1-20
│        Controls returned passages  │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│ Step 3: User clicks "Ask" button   │
│        Validate question is not    │
│        empty                       │
└────────────┬─────────────────────┘
             │
             ▼
┌──────────────────────────────────┐
│ Step 4: Display loading animation  │ ← with st.spinner(...)
│        Send Inngest event          │ ← asyncio.run(...)
└────────────┬─────────────────────┘
             │
             ▼ (Network Request)

Search Layer
┌──────────────────────────────────────────────┐
│ Step 5: Inngest receives "rag/query_pdf_ai" event │
│        (Queries have no rate limiting)       │
└────────────┬───────────────────────────────────┘
             │
             ▼
┌──────────────────────────────────────────────┐
│ Step 6: Embed question (embed_texts)        │
│        • Call OpenAI API                    │
│        • Convert question to 3072D vector   │
│        • Return single vector               │
└────────────┬───────────────────────────────────┘
             │
             ▼
┌──────────────────────────────────────────────┐
│ Step 7: Vector search (QdrantStorage.search) │
│        • Search in Qdrant                    │
│        • Calculate cosine similarity (0-2)   │
│        • Return top_k most similar Points    │
│        • Extract text and sources            │
└────────────┬───────────────────────────────────┘
             │
             ▼
┌──────────────────────────────────────────────┐
│ Step 8: Construct prompt                     │
│                                              │
│ System Role:                                 │
│ "You answer questions using only the        │
│  provided context."                         │
│                                              │
│ User Message:                                │
│ "Use the following context...                │
│  Context:                                    │
│  - Passage 1: xxx                            │
│  - Passage 2: yyy                            │
│  - Passage 3: zzz                            │
│                                              │
│  Question: {user_question}                   │
│  Answer concisely..."                        │
└────────────┬───────────────────────────────────┘
             │
             ▼
AI Inference Layer
┌──────────────────────────────────────────────┐
│ Step 9: Call GPT-4o-mini API                 │
│        ctx.step.ai.infer(...)                │
│                                              │
│        Parameters:                           │
│        • max_tokens: 1024                    │
│        • temperature: 0.2 (more deterministic) │
│        • messages: [system, user]            │
└────────────┬───────────────────────────────────┘
             │
             ▼ (Wait for AI response, typically 2-5s)

Result Processing Layer
┌──────────────────────────────────────────────┐
│ Step 10: Parse AI response                   │
│         Extract answer content               │
└────────────┬───────────────────────────────────┘
             │
             ▼ (Return Result)

User Display Layer
┌──────────────────────────────────────────────┐
│ Step 11: Display answer                      │
│         st.subheader("Answer")               │
│         st.write(answer)                     │
└────────────┬───────────────────────────────────┘
             │
             ▼
┌──────────────────────────────────────────────┐
│ Step 12: Display source files                │
│         st.caption("Sources")                │
│         List each source file                │
└────────────┬───────────────────────────────────┘
             │
             ▼
┌──────────────────────────────────────────────┐
│ Step 13: Optional continue asking            │
│         User can enter new question           │
└──────────────────────────────────────────────┘

===== Complete Time Example =====
User inputs question      2025-11-16 10:30:00
Click Ask button          2025-11-16 10:30:01
Send Inngest event        2025-11-16 10:30:02
Embed question            2025-11-16 10:30:02  (0.5 seconds)
Qdrant search             2025-11-16 10:30:03  (0.5 seconds)
Construct prompt          2025-11-16 10:30:03  (instant)
GPT-4o-mini inference     2025-11-16 10:30:08  (5 seconds)
Poll for results          2025-11-16 10:30:10  (0.5s poll interval)
Display answer            2025-11-16 10:30:11

Total Time: 11 seconds

===== Cost Estimate =====
Each query call:
1. Embed question: text_token_count × 0.00000002 USD
2. AI answer (gpt-4o-mini): input_tokens + output_tokens × rate

Assumptions:
- Question: 50 words = ~30 tokens
- Context: 5 passages × 100 words = 500 words = ~300 tokens
- Answer: 200 words = ~150 tokens

Cost: approximately $0.001-0.005 per query (depends on content)
```

---

## API Interface Documentation

### Important Note

**This application has no direct REST API**. All communication goes through the Inngest event system.

### Inngest Event Interface

#### Event 1: PDF Upload and Processing

**Event Name**: `rag/ingest_pdf`

```json
{
  "name": "rag/ingest_pdf",
  "data": {
    "pdf_path": "/home/user/uploads/document.pdf",
    "source_id": "document.pdf"
  }
}
```

**Parameter Description**:

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| pdf_path | string | ✓ | Absolute path to PDF file |
| source_id | string | ✓ | Unique identifier, typically filename |

**Return Value**:

```json
{
  "ingested": 42
}
```

| Field | Type | Description |
|-------|------|-------------|
| ingested | integer | Number of text chunks successfully processed |

**Rate Limiting Rules**:

- **Frequency Limit**: Maximum 1 per `source_id` per 4 hours
- **Throughput Limit**: Global maximum 2 requests per minute
- **Over-limit Handling**: Requests are queued, waiting for limit reset

**Timeout**: 60 seconds

**Retry Strategy**: Automatic retry up to 3 times with exponential backoff

**Sample Code**:

```python
# Python Client
import inngest

client = inngest.Inngest(app_id="rag_app", is_production=False)

# Send upload event
event_id = await client.send(
    inngest.Event(
        name="rag/ingest_pdf",
        data={
            "pdf_path": "/home/user/uploads/machine_learning.pdf",
            "source_id": "machine_learning.pdf"
        }
    )
)
```

---

#### Event 2: Question and Answer

**Event Name**: `rag/query_pdf_ai`

```json
{
  "name": "rag/query_pdf_ai",
  "data": {
    "question": "What is machine learning?",
    "top_k": 5
  }
}
```

**Parameter Description**:

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| question | string | ✓ | - | User's question |
| top_k | integer | ✗ | 5 | Number of relevant passages returned (1-20) |

**Return Value**:

```json
{
  "answer": "Machine learning is a subset of artificial intelligence...",
  "sources": ["machine_learning.pdf", "ai_guide.pdf"],
  "num_contexts": 5
}
```

| Field | Type | Description |
|-------|------|-------------|
| answer | string | AI-generated answer |
| sources | array | List of PDF filenames referenced in answer |
| num_contexts | integer | Number of context passages actually used |

**Rate Limiting Rules**:

- **Frequency Limit**: No limit
- **Throughput Limit**: No limit
- **Recommendation**: Add rate limiting in production to prevent abuse

**Timeout**: 120 seconds

**Retry Strategy**: Automatic retry up to 3 times

**Sample Code**:

```python
# Python Client
import inngest

client = inngest.Inngest(app_id="rag_app", is_production=False)

# Send query event
event_id = await client.send(
    inngest.Event(
        name="rag/query_pdf_ai",
        data={
            "question": "What is machine learning?",
            "top_k": 5
        }
    )
)

# Poll for result
import time
import requests

while True:
    response = requests.get(
        f"http://127.0.0.1:8288/v1/events/{event_id}/runs"
    )
    runs = response.json().get("data", [])

    if runs:
        run = runs[0]
        status = run.get("status")

        if status in ("Succeeded", "Completed"):
            return run.get("output")
        elif status in ("Failed", "Cancelled"):
            raise Exception(f"Run failed: {status}")

    time.sleep(0.5)  # Poll every 0.5 seconds
```

### FastAPI Endpoints

Inngest automatically generates the following endpoints (internal use):

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/api/inngest` | Receive events |
| GET | `/api/inngest` | Health check |
| PUT | `/api/inngest` | Function registration |

**Note**: These endpoints typically don't need direct calling, managed automatically by Inngest SDK.

---

## Environment Configuration Guide

### Required Environment Variables

#### Mandatory Variables

```bash
# OpenAI API Key
# Get from https://platform.openai.com/api-keys
OPENAI_API_KEY=sk-proj-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

**Steps to Get Key**:
1. Visit https://platform.openai.com/api-keys
2. Click "Create new secret key"
3. Copy key to .env file

#### Optional Variables

```bash
# Inngest local development server address
# Default: http://127.0.0.1:8288/v1
INNGEST_API_BASE=http://127.0.0.1:8288/v1
```

### .env File Configuration

**Location**: Project root directory

**File Content**:

```bash
# .env Example

# Required
OPENAI_API_KEY=sk-proj-your-api-key-here

# Optional (has default values)
# INNGEST_API_BASE=http://127.0.0.1:8288/v1
```

**Security Tips**:
```bash
# Set correct permissions (readable only by owner)
chmod 600 .env

# Make sure .env is in .gitignore (don't upload to Git)
echo ".env" >> .gitignore

# Verify .env is actually ignored
git check-ignore .env
```

### Environment Verification

```bash
# Verify Python version
python --version
# Should output: Python 3.13.x or higher

# Verify dependency installation
python -c "import fastapi; import streamlit; import inngest"
# No error means dependencies installed

# Verify OpenAI API key
python -c "import os; from dotenv import load_dotenv; load_dotenv(); print('OPENAI_API_KEY:', 'OK' if os.getenv('OPENAI_API_KEY') else 'MISSING')"
```

---

## Local Development Guide

### Prerequisites

**System Requirements**:

| Requirement | Version | Note |
|-------------|---------|------|
| Python | >= 3.13 | Recommended 3.13+ |
| Node.js | >= 18 | For Inngest CLI |
| Disk Space | >= 5GB | For dependencies and database |
| Memory | >= 4GB | To run all services |

**Operating System Support**:
- ✅ Linux (Ubuntu, Debian, CentOS)
- ✅ macOS (Intel and Apple Silicon)
- ✅ Windows (with WSL2 recommended)

### Environment Setup Steps

#### Step 1: Clone Project

```bash
git clone https://github.com/YOUR_REPO/ProductionGradeRAGPythonApp.git
cd ProductionGradeRAGPythonApp
```

#### Step 2: Create Python Virtual Environment

```bash
# Option A: Using venv (built-in)
python -m venv venv
source venv/bin/activate  # Linux/macOS
# or
venv\Scripts\activate  # Windows

# Option B: Using uv (recommended, faster)
pip install uv
uv venv
source .venv/bin/activate  # Linux/macOS
```

#### Step 3: Install Dependencies

```bash
# Using uv (recommended)
uv pip install

# Or using pip
pip install -r requirements.txt

# Or directly from pyproject.toml
pip install -e .
```

#### Step 4: Configure Environment Variables

```bash
# Create .env file
cp .env.example .env  # If example file exists

# Or manually create
cat > .env << EOF
OPENAI_API_KEY=sk-proj-your-key-here
INNGEST_API_BASE=http://127.0.0.1:8288/v1
EOF

# Verify
cat .env
```

#### Step 5: Install Node.js Dependencies (for Inngest CLI)

```bash
# Install Node.js
# macOS
brew install node

# Linux (Ubuntu/Debian)
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt-get install -y nodejs

# Windows
# Download installer from https://nodejs.org
```

### Start Development Server

**Need 3 separate terminal windows**:

#### Terminal 1: Start Inngest Development Server

```bash
# Install Inngest CLI (one time only)
npm install -g inngest-cli

# Start Inngest development server
npx inngest-cli@latest dev

# Output should show:
# ✓ Event stream
# ✓ API
# ✓ UI
#
# Inngest Dev Server is ready
# http://127.0.0.1:8288
```

**Access Inngest UI**: http://127.0.0.1:8288
- Real-time view of function executions
- Check logs and errors
- View performance metrics

#### Terminal 2: Start FastAPI Backend

```bash
# Make sure you're in virtual environment
source venv/bin/activate  # or your virtual environment

# Start development server (with hot reload)
uvicorn main:app --reload --host 127.0.0.1 --port 8000

# Output should show:
# Uvicorn running on http://127.0.0.1:8000
# Press CTRL+C to quit
```

**API Documentation**: http://127.0.0.1:8000/docs (Swagger UI)

**Visit this address to see interactive API documentation**

#### Terminal 3: Start Streamlit Frontend

```bash
# Make sure you're in virtual environment
source venv/bin/activate

# Start Streamlit server
streamlit run streamlit_app.py

# Output should show:
# You can now view your Streamlit app in your browser.
# Local URL: http://localhost:8501
```

**Streamlit Application**: http://localhost:8501

### Complete Startup Script

Create `start_dev.sh`:

```bash
#!/bin/bash

# Development environment one-click startup script

echo "=== RAG Application Development Environment Startup Script ==="
echo ""

# Check prerequisites
echo "Checking prerequisites..."
python --version || (echo "Python not installed"; exit 1)
node --version || (echo "Node.js not installed"; exit 1)

# Activate virtual environment
echo "Activating virtual environment..."
source venv/bin/activate || python -m venv venv && source venv/bin/activate

# Install dependencies
echo "Installing dependencies..."
uv pip install > /dev/null 2>&1 || pip install -q -r requirements.txt

# Create directories
mkdir -p uploads
mkdir -p qdrant_storage

echo ""
echo "✅ Environment ready!"
echo ""
echo "Please run the following commands in 3 separate terminals:"
echo ""
echo "1️⃣ Terminal 1 (Inngest):"
echo "   npx inngest-cli@latest dev"
echo ""
echo "2️⃣ Terminal 2 (Backend):"
echo "   uvicorn main:app --reload"
echo ""
echo "3️⃣ Terminal 3 (Frontend):"
echo "   streamlit run streamlit_app.py"
echo ""
echo "Application Addresses:"
echo "  - Frontend: http://localhost:8501"
echo "  - Backend: http://127.0.0.1:8000"
echo "  - Inngest UI: http://127.0.0.1:8288"
```

Run:
```bash
chmod +x start_dev.sh
./start_dev.sh
```

### Common Development Tasks

#### Task 1: View Logs

```bash
# Inngest logs
# View in Inngest UI: http://127.0.0.1:8288

# FastAPI logs
# Output in Terminal 2

# Streamlit logs
# Output in Terminal 3
```

#### Task 2: Debug Code

```python
# Add debug code in main.py
import logging
logging.basicConfig(level=logging.DEBUG)

# Add logging
logger = logging.getLogger(__name__)
logger.debug(f"Processing PDF: {pdf_path}")

# See DEBUG output when running
```

#### Task 3: Test Individual Functions

```python
# Create test_functions.py

import asyncio
from main import rag_ingest_pdf
from custom_types import RAGChunkAndSrc

async def test_ingest():
    # Mock Inngest Context
    class MockContext:
        class event:
            data = {
                "pdf_path": "path/to/file.pdf",
                "source_id": "file.pdf"
            }

        class step:
            async def run(self, name, func, *args):
                return await func(*args)

    ctx = MockContext()
    result = await rag_ingest_pdf(ctx)
    print(result)

# Run
asyncio.run(test_ingest())
```

#### Task 4: Reset Database

```bash
# Delete all Qdrant data
rm -rf qdrant_storage/

# Delete all uploaded PDFs
rm -rf uploads/*

# Collections will be recreated on next startup
```

#### Task 5: View Dependency Tree

```bash
# View all installed packages
pip list

# View dependency tree (install first)
pip install pipdeptree
pipdeptree
```

---

## Deployment Guide

### Pre-Deployment Checklist

- [ ] Update all dependencies to latest versions
- [ ] Run all tests
- [ ] Code review
- [ ] Check logs output
- [ ] Verify error handling
- [ ] Perform security audit
- [ ] Prepare data backup strategy
- [ ] Prepare rollback plan

### Local Deployment (Single Machine)

#### Docker Method (Recommended)

**Dockerfile**:

```dockerfile
# Dockerfile
FROM python:3.13-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Copy dependency files
COPY pyproject.toml uv.lock ./

# Install Python dependencies
RUN pip install uv && uv pip install --system

# Copy application code
COPY . .

# Expose ports
EXPOSE 8000 8501

# Create necessary directories
RUN mkdir -p uploads qdrant_storage

# Startup script
CMD ["bash", "-c", "uvicorn main:app --host 0.0.0.0 --port 8000 & streamlit run streamlit_app.py --server.port 8501 --server.address 0.0.0.0"]
```

**Docker Compose**:

```yaml
# docker-compose.yml
version: '3.8'

services:
  # Qdrant Vector Database
  qdrant:
    image: qdrant/qdrant:latest
    container_name: qdrant
    ports:
      - "6333:6333"
    volumes:
      - qdrant_storage:/qdrant/storage
    environment:
      - QDRANT_API_KEY=${QDRANT_API_KEY:-}
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:6333/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  # FastAPI Backend + Streamlit Frontend
  app:
    build: .
    container_name: rag-app
    ports:
      - "8000:8000"
      - "8501:8501"
    environment:
      - OPENAI_API_KEY=${OPENAI_API_KEY}
      - INNGEST_API_BASE=${INNGEST_API_BASE:-http://inngest:8288/v1}
    volumes:
      - ./uploads:/app/uploads
      - ./qdrant_storage:/app/qdrant_storage
    depends_on:
      qdrant:
        condition: service_healthy
    restart: unless-stopped

  # Inngest Event Processing (optional, for local development)
  inngest:
    image: inngest/inngest:latest
    container_name: inngest
    ports:
      - "8288:8288"
    environment:
      - INNGEST_DEV=1
    restart: unless-stopped

volumes:
  qdrant_storage:
    driver: local
```

**Startup Method**:

```bash
# Copy .env file
cp .env.example .env
# Edit .env, fill in OPENAI_API_KEY

# Build image (one time only)
docker-compose build

# Start all services
docker-compose up -d

# View logs
docker-compose logs -f

# Stop services
docker-compose down

# Stop while keeping data
docker-compose stop

# Delete all data and containers
docker-compose down -v
```

**Access Application**:
- Frontend: http://localhost:8501
- Backend API: http://localhost:8000
- Qdrant: http://localhost:6333
- Inngest: http://localhost:8288

### Cloud Platform Deployment

#### Railway.app Deployment Example

```yaml
# railway.json
{
  "name": "RAG Application",
  "description": "Production-ready RAG Python Application",
  "buildCommand": "pip install -r requirements.txt",
  "startCommand": "uvicorn main:app --host 0.0.0.0 --port $PORT"
}
```

**Deployment Steps**:

1. Create Railway account (https://railway.app)
2. Connect GitHub repository
3. Set environment variables:
   - `OPENAI_API_KEY`: Your API key
   - `INNGEST_API_BASE`: Inngest server address
4. Deploy

#### Render.com Deployment Example

1. Create Render account
2. New → Web Service
3. Connect GitHub repository
4. Configure:
   - Build Command: `pip install -r requirements.txt`
   - Start Command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
5. Set environment variables
6. Deploy

### Performance Optimization Tips

#### Production Environment Configuration

```bash
# Uvicorn Optimization
uvicorn main:app \
  --host 0.0.0.0 \
  --port 8000 \
  --workers 4 \              # Multiple worker processes
  --worker-class uvicorn.workers.UvicornWorker \
  --loop uvloop \             # Faster event loop
  --http httptools            # C-accelerated HTTP parsing
```

#### Database Optimization

```python
# Qdrant connection pool optimization
class QdrantStorage:
    def __init__(self, url="http://localhost:6333"):
        self.client = QdrantClient(
            url=url,
            timeout=30,
            prefer_grpc=True,  # Use gRPC for speed
            grpc_port=6334     # gRPC port
        )
```

---

## Frequently Asked Questions (FAQ)

### Q1: "ModuleNotFoundError: No module named 'openai'"

**Problem**: Import error when running

**Solution**:
```bash
# Reinstall dependencies
pip install --upgrade openai

# Or check virtual environment
which python  # Should point to virtual environment
source venv/bin/activate  # Reactivate
```

---

### Q2: "OpenAI API key not found"

**Problem**: Application can't find API key

**Solution**:
```bash
# Check .env file exists
ls -la .env

# Check key format
cat .env | grep OPENAI

# Should output: OPENAI_API_KEY=sk-proj-xxxx

# Verify key validity
python -c "
import os
from dotenv import load_dotenv
load_dotenv()
key = os.getenv('OPENAI_API_KEY')
if key and key.startswith('sk-'):
    print('✓ Key format correct')
else:
    print('✗ Key format wrong or not set')
"
```

---

### Q3: "ConnectionError: Failed to connect to Qdrant"

**Problem**: Can't connect to vector database

**Solution**:
```bash
# Check if Qdrant is running
curl http://localhost:6333/health

# If failed, start Qdrant
docker run -p 6333:6333 qdrant/qdrant

# Or use Docker Compose
docker-compose up qdrant
```

---

### Q4: "Inngest connection failed"

**Problem**: Can't connect to Inngest event engine

**Solution**:
```bash
# Start Inngest development server
npx inngest-cli@latest dev

# Verify connection
curl http://127.0.0.1:8288

# Check INNGEST_API_BASE in .env
# Should be: http://127.0.0.1:8288/v1
```

---

### Q5: "No response after PDF upload"

**Problem**: Application doesn't process after PDF upload

**Solution**:
```bash
# 1. Check backend logs (Terminal 2)
# Should see "load-and-chunk" and "embed-and-upsert" steps

# 2. Check Inngest UI
# http://127.0.0.1:8288 to view function execution

# 3. Check if file saved
ls -la uploads/

# 4. Check OpenAI API calls
# Should see API requests in backend logs

# 5. Verify API quota
# Visit https://platform.openai.com/account/usage
```

---

### Q6: "Timeout waiting for run output"

**Problem**: Timeout when asking questions, can't get answers

**Solution**:
```bash
# 1. Check backend responding
curl http://127.0.0.1:8000/docs

# 2. Check OpenAI API speed
# Sometimes API responds slowly
# Try again after waiting seconds

# 3. Increase timeout (streamlit_app.py)
# Modify wait_for_run_output timeout_s parameter
def wait_for_run_output(event_id: str, timeout_s: float = 180.0):
    # Change to 180 seconds
    ...

# 4. Check network connection
ping api.openai.com
```

---

### Q7: "Too many requests" error

**Problem**: API call exceeds rate limit threshold

**Solution**:
```bash
# For PDF upload:
# Current limit: 1 per file per 4 hours
# Solution: Change filename or wait 4 hours

# For question query:
# Currently no limit
# Recommend adding in production

# Check rate limiting rules (main.py)
rate_limit=inngest.RateLimit(
    limit=1,
    period=datetime.timedelta(hours=4),
    key="event.data.source_id",
)
```

---

### Q8: "Vector database becoming very large"

**Problem**: qdrant_storage directory takes up large disk space

**Causes**:
- Uploaded too many documents
- Same document uploaded multiple times

**Solution**:
```bash
# Clear entire database
rm -rf qdrant_storage/
# Recreated on next startup

# Or delete specific collection
# (Implement deletion logic yourself)

# View database size
du -sh qdrant_storage/

# Unit reference
# K = KB, M = MB, G = GB
```

---

### Q9: "GPU/Acceleration Hardware Related Errors"

**Problem**: CUDA, MPS, or other acceleration errors

**Note**: This application doesn't use GPU
- All computations run on CPU
- OpenAI API calls are remote, no local GPU needed
- Vector search on Qdrant, no GPU needed

**Solution**: Can safely ignore these warnings

---

### Q10: "Production Deployment Issues"

**Common Problem**: Works locally but fails after deployment

**Verification Checklist**:

```
□ All environment variables set
□ OPENAI_API_KEY correct
□ INNGEST_API_BASE points to correct server
□ Database connection open
□ File system permissions correct
□ Firewall allows necessary ports
□ SSL/HTTPS properly configured
□ Logs readable
□ Backup strategy implemented
□ Monitoring alerts configured
```

**Common Deployment Errors**:

| Error | Cause | Solution |
|-------|-------|----------|
| 502 Bad Gateway | Backend not running | Check process |
| Connection timeout | Firewall blocking | Configure security group |
| File permission error | Insufficient directory permissions | chmod 755 |
| Data loss | Data volume not mounted | Configure persistence |

---

## Terminology Reference

### English Term → Chinese Meaning

| English | Chinese | Explanation |
|---------|---------|-------------|
| RAG | Retrieval-Augmented Generation | AI technique that first searches for relevant information, then generates answers |
| Vector Database | 向量数据库 | Database that stores data's "meaning" rather than data itself |
| Embedding | 嵌入/向量化 | Process of converting text to numeric vectors |
| Semantic Search | 语义搜索 | Find content by meaning rather than keywords |
| Chunk | 块/段落 | Small fragments of longer text |
| Context | 上下文/背景 | Relevant information provided to AI |
| Prompt | 提示词 | Instructions and background info for AI |
| Token | 令牌 | Smallest unit AI computes, typically few characters |
| Rate Limiting | 限流 | Restrict request frequency to prevent abuse |
| Throttling | 节流 | Limit number of tasks processed simultaneously |
| Qdrant | Qdrant | Vector database name (brand) |
| Inngest | Inngest | Event processing engine name (brand) |
| FastAPI | FastAPI | Web framework name (brand) |
| Streamlit | Streamlit | Web UI framework name (brand) |
| Async | 异步 | Non-blocking execution, handle multiple tasks simultaneously |
| Callback | 回调 | Function passed as parameter to another function |
| Payload | 负载/数据包 | Actual data in request or response |
| UUID | Universal Unique Identifier | 128-bit number to uniquely identify objects |

---

## Appendix: Quick Reference

### Quick Start

```bash
# 1️⃣ Environment Setup
python -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# 2️⃣ Configuration
cat > .env << EOF
OPENAI_API_KEY=sk-your-key
EOF

# 3️⃣ Start (need 3 terminals)
# Terminal 1
npx inngest-cli@latest dev

# Terminal 2
uvicorn main:app --reload

# Terminal 3
streamlit run streamlit_app.py

# 4️⃣ Access
# Frontend: http://localhost:8501
# Backend API: http://127.0.0.1:8000/docs
# Inngest UI: http://127.0.0.1:8288
```

### Key File Quick Reference

| File | Location | Function |
|------|----------|----------|
| Backend Main | `/main.py` | FastAPI app, define Inngest functions |
| Frontend Page | `/streamlit_app.py` | Web interface, user interaction |
| Database Operations | `/vector_db.py` | Qdrant connection and queries |
| Data Processing | `/data_loader.py` | PDF reading, vector generation |
| Data Models | `/custom_types.py` | Pydantic model definitions |
| Config File | `/pyproject.toml` | Project metadata and dependencies |
| Environment Variables | `/.env` | API keys and sensitive info |

### Common Commands

```bash
# Development
python -m pytest                    # Run tests
black main.py                       # Code formatting
pylint main.py                      # Code checking
uvicorn main:app --reload          # Development server

# Production
docker-compose up -d                # Start containers
docker-compose logs -f              # View logs
docker-compose down                 # Stop containers

# Database
curl http://localhost:6333/health   # Check Qdrant
curl http://localhost:6333/collections # View collections

# Debugging
python -c "from dotenv import load_dotenv; load_dotenv()"  # Test .env
python -c "from openai import OpenAI; OpenAI()"  # Test API
```

---

## Summary

This project demonstrates how to build a production-grade AI application using modern Python technology stack:

✅ **Frontend**: Streamlit provides clean web interface
✅ **Backend**: FastAPI provides high-performance async server
✅ **Events**: Inngest provides reliable async task processing
✅ **Database**: Qdrant provides vector search capability
✅ **AI**: OpenAI provides powerful language models

**Next Steps**:
1. Run local development environment (see Local Development Guide)
2. Upload PDFs and test Q&A functionality
3. Adjust parameters and configuration as needed
4. Deploy to production environment

**Get Help**:
- Check analysis documentation: `project-mastery/analysis/`
- Check source code comments
- Check Inngest UI real-time logs
- Check OpenAI API documentation

---

**Documentation Completed** ✅
**Last Updated**: 2025-11-16
**Version**: 1.0
