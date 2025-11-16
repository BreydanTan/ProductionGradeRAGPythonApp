# 安全分析 / Security Analysis

**生成时间 / Generated**: 2025-11-16
**安全级别 / Security Level**: 开发环境 / Development Environment

---

## 📋 安全概览 / Security Overview

### 中文
本项目是一个**本地开发版本**的RAG应用，主要用于学习和演示目的。项目实现了一些基本的安全措施，但**不适合直接部署到生产环境**。需要额外的安全加固才能用于生产。

**当前安全状态 / Current Security Status**:
- ✅ 环境变量管理（API密钥）
- ✅ 限流和节流保护
- ⚠️ 缺少用户认证
- ⚠️ 缺少输入验证
- ⚠️ 缺少文件安全检查
- ⚠️ 缺少HTTPS加密

### English
This project is a **local development version** of a RAG application, primarily for learning and demonstration purposes. The project implements some basic security measures but is **not suitable for direct production deployment**. Additional security hardening is required for production use.

**Current Security Status**:
- ✅ Environment variable management (API keys)
- ✅ Rate limiting and throttling protection
- ⚠️ Missing user authentication
- ⚠️ Missing input validation
- ⚠️ Missing file security checks
- ⚠️ Missing HTTPS encryption

---

## 🔐 认证机制 / Authentication Mechanism

### 当前状态 / Current Status

**实现状态 / Implementation Status**: ❌ **未实现** / **Not Implemented**

**说明 / Description**:
- 没有用户登录系统 / No user login system
- 没有API密钥验证 / No API key validation
- 任何人都可以访问应用 / Anyone can access the application
- 任何人都可以调用Inngest端点 / Anyone can call Inngest endpoints

### 潜在风险 / Potential Risks

1. **未授权访问 / Unauthorized Access**
   - 任何人都可以上传PDF / Anyone can upload PDFs
   - 任何人都可以查询数据库 / Anyone can query the database
   - 可能导致滥用 / May lead to abuse

2. **数据泄露 / Data Leakage**
   - 所有用户共享同一个向量数据库 / All users share the same vector DB
   - 用户A可以查询用户B上传的文档 / User A can query User B's documents
   - 没有数据隔离 / No data isolation

### 生产环境建议 / Production Recommendations

#### 1. 实现用户认证 / Implement User Authentication

**建议方案 / Recommended Solution**:
```python
# 使用FastAPI的OAuth2 / Use FastAPI's OAuth2
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

@app.post("/token")
async def login(form_data: OAuth2PasswordRequestForm = Depends()):
    # 验证用户名和密码 / Verify username and password
    user = authenticate_user(form_data.username, form_data.password)
    if not user:
        raise HTTPException(status_code=401, detail="Incorrect credentials")
    # 生成JWT / Generate JWT
    access_token = create_access_token(data={"sub": user.username})
    return {"access_token": access_token, "token_type": "bearer"}
```

#### 2. 添加Inngest签名验证 / Add Inngest Signature Verification

**建议方案 / Recommended Solution**:
```python
# 验证Inngest请求签名 / Verify Inngest request signature
from inngest.fast_api import serve

serve(
    app,
    inngest_client,
    [rag_ingest_pdf, rag_query_pdf_ai],
    signing_key=os.getenv("INNGEST_SIGNING_KEY")  # 添加签名密钥 / Add signing key
)
```

#### 3. 实现多租户隔离 / Implement Multi-Tenancy Isolation

**建议方案 / Recommended Solution**:
```python
# 为每个用户创建独立的Qdrant集合 / Create separate Qdrant collection per user
def get_user_collection(user_id: str) -> str:
    return f"docs_user_{user_id}"

# 在上传和查询时使用用户专属集合 / Use user-specific collection
QdrantStorage(collection=get_user_collection(current_user.id))
```

---

## 🛡️ 授权规则 / Authorization Rules

### 当前状态 / Current Status

**实现状态 / Implementation Status**: ❌ **未实现** / **Not Implemented**

**说明 / Description**:
- 没有基于角色的访问控制（RBAC）/ No Role-Based Access Control (RBAC)
- 没有资源级权限检查 / No resource-level permission checks
- 所有操作对所有用户开放 / All operations open to all users

### 生产环境建议 / Production Recommendations

#### 权限矩阵 / Permission Matrix

| 操作 / Operation | 需要权限 / Required Permission |
|-----------------|-------------------------------|
| 上传PDF / Upload PDF | `rag:ingest` |
| 查询PDF / Query PDF | `rag:query` |
| 删除PDF / Delete PDF | `rag:delete` + `owns_resource` |
| 查看所有文档 / View all docs | `rag:admin` |

**实现示例 / Implementation Example**:
```python
from functools import wraps

def require_permission(permission: str):
    def decorator(func):
        @wraps(func)
        async def wrapper(ctx: inngest.Context):
            user = get_current_user(ctx)
            if not user.has_permission(permission):
                raise PermissionError(f"Missing permission: {permission}")
            return await func(ctx)
        return wrapper
    return decorator

@require_permission("rag:ingest")
async def rag_ingest_pdf(ctx: inngest.Context):
    ...
```

---

## 🔒 数据保护 / Data Protection

### 1. 输入验证 / Input Validation

#### 当前实现 / Current Implementation

**文件类型验证 / File Type Validation** (前端):
```python
# streamlit_app.py:44
st.file_uploader("Choose a PDF", type=["pdf"])
```

**问题验证 / Question Validation** (前端):
```python
# streamlit_app.py:111
if submitted and question.strip():
    # 只检查非空 / Only checks non-empty
    ...
```

#### 安全问题 / Security Issues

❌ **缺少后端验证 / Missing Backend Validation**:
- 后端不验证文件类型 / Backend doesn't validate file type
- 恶意用户可以绕过前端直接发送Inngest事件 / Malicious users can bypass frontend and send Inngest events directly
- 没有文件大小限制 / No file size limit
- 没有文件内容检查 / No file content inspection

❌ **缺少输入消毒 / Missing Input Sanitization**:
- 用户问题没有过滤恶意内容 / User questions not filtered for malicious content
- PDF文件名没有消毒 / PDF filenames not sanitized
- 可能导致路径遍历攻击 / May lead to path traversal attacks

#### 加固建议 / Hardening Recommendations

**1. 后端文件验证 / Backend File Validation**:
```python
# main.py
import magic  # python-magic库 / python-magic library

def _load(ctx: inngest.Context) -> RAGChunkAndSrc:
    pdf_path = ctx.event.data["pdf_path"]

    # 验证文件存在 / Verify file exists
    if not Path(pdf_path).exists():
        raise FileNotFoundError(f"File not found: {pdf_path}")

    # 验证文件类型（魔数检查）/ Verify file type (magic number check)
    file_type = magic.from_file(pdf_path, mime=True)
    if file_type != "application/pdf":
        raise ValueError(f"Invalid file type: {file_type}. Expected application/pdf")

    # 验证文件大小 / Verify file size
    file_size = Path(pdf_path).stat().st_size
    MAX_SIZE = 10 * 1024 * 1024  # 10MB
    if file_size > MAX_SIZE:
        raise ValueError(f"File too large: {file_size} bytes. Max: {MAX_SIZE}")

    # 原有逻辑 / Original logic
    chunks = load_and_chunk_pdf(pdf_path)
    return RAGChunkAndSrc(chunks=chunks, source_id=source_id)
```

**2. 路径遍历防护 / Path Traversal Protection**:
```python
# streamlit_app.py
from pathlib import Path

def save_uploaded_pdf(file) -> Path:
    uploads_dir = Path("uploads").resolve()  # 获取绝对路径 / Get absolute path
    uploads_dir.mkdir(parents=True, exist_ok=True)

    # 消毒文件名 / Sanitize filename
    safe_filename = Path(file.name).name  # 只取文件名，去除路径 / Only filename, remove path
    safe_filename = "".join(c for c in safe_filename if c.isalnum() or c in "._- ")  # 过滤特殊字符 / Filter special chars

    file_path = uploads_dir / safe_filename

    # 验证路径在uploads目录内 / Verify path is within uploads dir
    if not file_path.resolve().is_relative_to(uploads_dir):
        raise ValueError("Invalid file path")

    file_path.write_bytes(file.getbuffer())
    return file_path
```

**3. 输入长度限制 / Input Length Limiting**:
```python
# main.py
def _search(question: str, top_k: int = 5) -> RAGSearchResult:
    # 限制问题长度 / Limit question length
    MAX_QUESTION_LENGTH = 500
    if len(question) > MAX_QUESTION_LENGTH:
        raise ValueError(f"Question too long: {len(question)} chars. Max: {MAX_QUESTION_LENGTH}")

    # 限制top_k范围 / Limit top_k range
    if not 1 <= top_k <= 20:
        raise ValueError(f"Invalid top_k: {top_k}. Must be between 1 and 20")

    # 原有逻辑 / Original logic
    query_vec = embed_texts([question])[0]
    store = QdrantStorage()
    found = store.search(query_vec, top_k)
    return RAGSearchResult(contexts=found["contexts"], sources=found["sources"])
```

---

### 2. SQL注入防护 / SQL Injection Protection

**当前状态 / Current Status**: ✅ **不适用** / **Not Applicable**

**说明 / Description**:
- 项目使用Qdrant向量数据库，不是SQL数据库 / Uses Qdrant vector DB, not SQL
- QdrantClient使用参数化查询，自动防止注入 / QdrantClient uses parameterized queries, auto-prevents injection
- 没有直接的SQL查询 / No direct SQL queries

---

### 3. XSS防护 / XSS Protection

**当前状态 / Current Status**: ✅ **Streamlit自动防护** / **Auto-Protected by Streamlit**

**说明 / Description**:
- Streamlit自动转义所有文本输出 / Streamlit auto-escapes all text output
- `st.write()` 不会执行HTML / `st.write()` doesn't execute HTML
- 用户输入不会直接渲染为HTML / User input not directly rendered as HTML

**示例 / Example**:
```python
# 即使用户输入包含HTML，也会被转义 / Even if user input contains HTML, it's escaped
question = "<script>alert('XSS')</script>"
st.write(question)  # 显示为文本，不执行 / Displays as text, doesn't execute
```

---

### 4. CSRF防护 / CSRF Protection

**当前状态 / Current Status**: ⚠️ **部分防护** / **Partially Protected**

**说明 / Description**:
- Streamlit内置CSRF token机制 / Streamlit has built-in CSRF token mechanism
- 但Inngest端点没有CSRF保护 / But Inngest endpoints lack CSRF protection
- 外部攻击者可以伪造Inngest请求 / External attackers can forge Inngest requests

#### 加固建议 / Hardening Recommendations

**1. 启用Inngest签名验证 / Enable Inngest Signature Verification**:
```python
# main.py
inngest.fast_api.serve(
    app,
    inngest_client,
    [rag_ingest_pdf, rag_query_pdf_ai],
    signing_key=os.getenv("INNGEST_SIGNING_KEY"),  # 添加签名密钥 / Add signing key
    signing_key_fallback=None  # 禁用回退，强制验证 / Disable fallback, force verification
)
```

**2. 添加CORS限制 / Add CORS Restrictions**:
```python
# main.py
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:8501"],  # 只允许Streamlit / Only allow Streamlit
    allow_credentials=True,
    allow_methods=["POST"],
    allow_headers=["*"],
)
```

---

## 🔑 环境变量管理 / Environment Variable Management

### 当前实现 / Current Implementation

#### .env文件 / .env File
**位置 / Location**: `/.env` (未包含在Git仓库中 / Not in Git repo)

```bash
# 必需的环境变量 / Required environment variables
OPENAI_API_KEY=sk-...

# 可选的环境变量 / Optional environment variables
INNGEST_API_BASE=http://127.0.0.1:8288/v1
```

#### 加载方式 / Loading Method
```python
# main.py:14, data_loader.py:6, streamlit_app.py:11
from dotenv import load_dotenv
load_dotenv()
```

### 安全问题 / Security Issues

❌ **缺少环境变量验证 / Missing Environment Variable Validation**:
```python
# 如果OPENAI_API_KEY未设置，会在运行时崩溃 / If OPENAI_API_KEY not set, crashes at runtime
client = OpenAI()  # 会抛出错误 / Will throw error
```

❌ **.env文件权限 / .env File Permissions**:
- 文件可能有过宽的权限 / File may have overly permissive permissions
- 应设置为只有所有者可读 / Should be readable only by owner

### 加固建议 / Hardening Recommendations

**1. 启动时验证环境变量 / Validate Environment Variables at Startup**:
```python
# main.py
import os
import sys

def validate_env_vars():
    required_vars = ["OPENAI_API_KEY"]
    missing = [var for var in required_vars if not os.getenv(var)]

    if missing:
        print(f"❌ Missing required environment variables: {', '.join(missing)}")
        print("Please set them in .env file")
        sys.exit(1)

validate_env_vars()
```

**2. 使用密钥管理服务 / Use Secret Management Service**:
```python
# 生产环境使用AWS Secrets Manager / Production: use AWS Secrets Manager
import boto3

def get_secret(secret_name: str) -> str:
    client = boto3.client("secretsmanager")
    response = client.get_secret_value(SecretId=secret_name)
    return response["SecretString"]

OPENAI_API_KEY = get_secret("openai_api_key")
```

**3. 设置.env文件权限 / Set .env File Permissions**:
```bash
# 只有所有者可读写 / Only owner can read/write
chmod 600 .env
```

---

## 🛡️ 限流和防滥用 / Rate Limiting & Abuse Prevention

### 当前实现 / Current Implementation

#### 1. Inngest限流 / Inngest Rate Limiting

**PDF摄取限流 / PDF Ingestion Rate Limit**:
```python
# main.py:29-33
rate_limit=inngest.RateLimit(
    limit=1,                               # 限制次数 / Limit count
    period=datetime.timedelta(hours=4),    # 时间周期 / Time period
    key="event.data.source_id",            # 分组键 / Grouping key
)
```

**效果 / Effect**:
- 每个 `source_id` 每4小时最多摄取1次 / Max 1 ingestion per `source_id` per 4 hours
- 防止同一PDF被重复上传 / Prevent same PDF from being re-uploaded
- 按文件名分组限流 / Rate limit grouped by filename

#### 2. Inngest节流 / Inngest Throttling

**PDF摄取节流 / PDF Ingestion Throttling**:
```python
# main.py:26-28
throttle=inngest.Throttle(
    count=2,                               # 最大并发数 / Max concurrent
    period=datetime.timedelta(minutes=1)   # 时间窗口 / Time window
)
```

**效果 / Effect**:
- 每分钟最多处理2个PDF摄取请求 / Max 2 PDF ingestions per minute
- 超过限制的请求会排队 / Excess requests are queued
- 全局节流（不分用户）/ Global throttling (not per user)

### 安全问题 / Security Issues

⚠️ **缺少查询限流 / Missing Query Rate Limiting**:
- `rag_query_pdf_ai` 函数没有限流 / `rag_query_pdf_ai` has no rate limiting
- 恶意用户可以发送大量查询 / Malicious users can send many queries
- 可能导致OpenAI API成本激增 / May cause OpenAI API costs to surge

⚠️ **缺少IP级限流 / Missing IP-Level Rate Limiting**:
- 当前限流基于 `source_id`，可以通过改变文件名绕过 / Current rate limit based on `source_id`, can bypass by changing filename
- 没有基于IP地址的限流 / No IP-based rate limiting

### 加固建议 / Hardening Recommendations

**1. 添加查询限流 / Add Query Rate Limiting**:
```python
# main.py
@inngest_client.create_function(
    fn_id="RAG: Query PDF",
    trigger=inngest.TriggerEvent(event="rag/query_pdf_ai"),
    # 添加限流 / Add rate limiting
    rate_limit=inngest.RateLimit(
        limit=10,                              # 每用户10次 / 10 per user
        period=datetime.timedelta(minutes=1),  # 每分钟 / per minute
        key="event.data.user_id",              # 按用户限流 / Rate limit by user
    ),
)
async def rag_query_pdf_ai(ctx: inngest.Context):
    ...
```

**2. 添加IP级限流（使用FastAPI）/ Add IP-Level Rate Limiting (FastAPI)**:
```python
# main.py
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

@app.post("/api/inngest")
@limiter.limit("100/minute")  # 每IP每分钟100次 / 100 per IP per minute
async def inngest_endpoint(request: Request):
    ...
```

---

## 🔍 日志和审计 / Logging & Auditing

### 当前实现 / Current Implementation

#### Inngest日志 / Inngest Logging
```python
# main.py:18
logger=logging.getLogger("uvicorn")
```

**功能 / Features**:
- 自动记录函数执行 / Auto-log function executions
- 记录错误和异常 / Log errors and exceptions
- 可在Inngest UI查看 / Viewable in Inngest UI

### 安全问题 / Security Issues

❌ **缺少安全事件日志 / Missing Security Event Logging**:
- 没有记录登录尝试（因为没有登录系统）/ No login attempt logs (no login system)
- 没有记录文件上传事件 / No file upload event logs
- 没有记录敏感操作 / No sensitive operation logs

❌ **日志中可能包含敏感信息 / Logs May Contain Sensitive Info**:
- 用户问题可能包含敏感数据 / User questions may contain sensitive data
- PDF内容可能出现在日志中 / PDF content may appear in logs

### 加固建议 / Hardening Recommendations

**1. 添加安全审计日志 / Add Security Audit Logging**:
```python
# 创建审计日志模块 / Create audit logging module
import logging
from datetime import datetime

audit_logger = logging.getLogger("security_audit")
audit_handler = logging.FileHandler("security_audit.log")
audit_logger.addHandler(audit_handler)

def log_security_event(event_type: str, user_id: str, details: dict):
    audit_logger.info({
        "timestamp": datetime.utcnow().isoformat(),
        "event_type": event_type,
        "user_id": user_id,
        "details": details
    })

# 使用示例 / Usage example
def _load(ctx: inngest.Context) -> RAGChunkAndSrc:
    pdf_path = ctx.event.data["pdf_path"]
    log_security_event(
        event_type="PDF_UPLOAD",
        user_id="anonymous",  # 应从认证系统获取 / Should get from auth system
        details={"filename": Path(pdf_path).name, "size": Path(pdf_path).stat().st_size}
    )
    ...
```

**2. 敏感数据脱敏 / Sensitive Data Masking**:
```python
def mask_sensitive_data(text: str) -> str:
    # 脱敏邮箱 / Mask emails
    text = re.sub(r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b', '***@***.***', text)
    # 脱敏手机号 / Mask phone numbers
    text = re.sub(r'\b\d{3}[-.]?\d{3}[-.]?\d{4}\b', '***-***-****', text)
    return text

# 记录日志前脱敏 / Mask before logging
logger.info(f"User question: {mask_sensitive_data(question)}")
```

---

## 🔐 数据加密 / Data Encryption

### 传输加密 / Encryption in Transit

**当前状态 / Current Status**: ❌ **HTTP（未加密）** / **HTTP (Unencrypted)**

**说明 / Description**:
- 所有通信使用HTTP / All communication uses HTTP
- 数据在网络中明文传输 / Data transmitted in plaintext
- 容易被中间人攻击 / Vulnerable to man-in-the-middle attacks

#### 加固建议 / Hardening Recommendations

**1. 启用HTTPS / Enable HTTPS**:
```bash
# 使用Nginx作为反向代理 / Use Nginx as reverse proxy
# nginx.conf
server {
    listen 443 ssl;
    server_name your-domain.com;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;

    location / {
        proxy_pass http://localhost:8000;
    }
}
```

**2. 使用Let's Encrypt自动证书 / Use Let's Encrypt Auto Certs**:
```bash
# 安装Certbot / Install Certbot
sudo apt-get install certbot python3-certbot-nginx

# 获取证书 / Get certificate
sudo certbot --nginx -d your-domain.com
```

---

### 存储加密 / Encryption at Rest

**当前状态 / Current Status**: ❌ **明文存储** / **Plaintext Storage**

**说明 / Description**:
- PDF文件以明文存储在 `uploads/` / PDF files stored in plaintext in `uploads/`
- Qdrant数据库以明文存储在 `qdrant_storage/` / Qdrant DB stored in plaintext in `qdrant_storage/`
- 嵌入向量未加密 / Embedding vectors unencrypted

#### 加固建议 / Hardening Recommendations

**1. 文件系统加密 / Filesystem Encryption**:
```bash
# 使用LUKS加密分区 / Use LUKS to encrypt partition
cryptsetup luksFormat /dev/sdX
cryptsetup open /dev/sdX encrypted_partition
mkfs.ext4 /dev/mapper/encrypted_partition
mount /dev/mapper/encrypted_partition /mnt/encrypted
```

**2. 应用层加密 / Application-Layer Encryption**:
```python
# 加密上传的PDF / Encrypt uploaded PDFs
from cryptography.fernet import Fernet

# 生成密钥（存储在环境变量中）/ Generate key (store in env var)
encryption_key = os.getenv("ENCRYPTION_KEY")
fernet = Fernet(encryption_key)

def save_encrypted_pdf(file) -> Path:
    file_bytes = file.getbuffer()
    encrypted_bytes = fernet.encrypt(file_bytes)

    file_path = uploads_dir / f"{file.name}.encrypted"
    file_path.write_bytes(encrypted_bytes)
    return file_path

def load_encrypted_pdf(file_path: Path) -> bytes:
    encrypted_bytes = file_path.read_bytes()
    decrypted_bytes = fernet.decrypt(encrypted_bytes)
    return decrypted_bytes
```

---

## 📊 安全检查清单 / Security Checklist

### 开发环境 / Development Environment

- [x] 环境变量管理 / Environment variable management
- [x] .gitignore配置（排除.env）/ .gitignore configured (exclude .env)
- [x] 基本限流 / Basic rate limiting
- [ ] 输入验证 / Input validation
- [ ] 文件类型验证（后端）/ File type validation (backend)
- [ ] 错误处理 / Error handling

### 生产环境 / Production Environment

- [ ] HTTPS加密 / HTTPS encryption
- [ ] 用户认证 / User authentication
- [ ] 授权控制 / Authorization control
- [ ] API密钥验证 / API key validation
- [ ] Inngest签名验证 / Inngest signature verification
- [ ] CORS配置 / CORS configuration
- [ ] 输入验证和消毒 / Input validation and sanitization
- [ ] 文件大小限制 / File size limits
- [ ] 查询限流 / Query rate limiting
- [ ] IP级限流 / IP-level rate limiting
- [ ] 安全审计日志 / Security audit logging
- [ ] 数据加密（传输和存储）/ Data encryption (transit & rest)
- [ ] 定期安全扫描 / Regular security scans
- [ ] 依赖漏洞扫描 / Dependency vulnerability scans
- [ ] DDoS防护 / DDoS protection
- [ ] Web应用防火墙（WAF）/ Web Application Firewall (WAF)

---

## 🚨 已知漏洞和风险 / Known Vulnerabilities & Risks

### 高风险 / High Risk

1. **无认证访问 / Unauthenticated Access**
   - 严重性：🔴 高 / Severity: 🔴 High
   - 影响：任何人都可以使用应用 / Impact: Anyone can use the app
   - 缓解：实现用户认证 / Mitigation: Implement user authentication

2. **数据泄露风险 / Data Leakage Risk**
   - 严重性：🔴 高 / Severity: 🔴 High
   - 影响：用户可以查询其他用户的文档 / Impact: Users can query others' documents
   - 缓解：实现多租户隔离 / Mitigation: Implement multi-tenancy

3. **无限查询成本 / Unlimited Query Costs**
   - 严重性：🟡 中 / Severity: 🟡 Medium
   - 影响：恶意用户可能导致高额OpenAI费用 / Impact: Malicious users may cause high OpenAI costs
   - 缓解：添加查询限流 / Mitigation: Add query rate limiting

### 中风险 / Medium Risk

4. **路径遍历攻击 / Path Traversal Attack**
   - 严重性：🟡 中 / Severity: 🟡 Medium
   - 影响：恶意文件名可能访问系统文件 / Impact: Malicious filenames may access system files
   - 缓解：文件名消毒 / Mitigation: Filename sanitization

5. **HTTP明文传输 / HTTP Plaintext Transmission**
   - 严重性：🟡 中 / Severity: 🟡 Medium
   - 影响：数据可被窃听 / Impact: Data can be intercepted
   - 缓解：启用HTTPS / Mitigation: Enable HTTPS

### 低风险 / Low Risk

6. **缺少文件大小限制 / Missing File Size Limit**
   - 严重性：🟢 低 / Severity: 🟢 Low
   - 影响：大文件可能耗尽磁盘空间 / Impact: Large files may exhaust disk space
   - 缓解：添加文件大小限制 / Mitigation: Add file size limit

---

**文档生成完成 / Document Generated**: ✅
**下一步 / Next Step**: 05-deployment-analysis.md
