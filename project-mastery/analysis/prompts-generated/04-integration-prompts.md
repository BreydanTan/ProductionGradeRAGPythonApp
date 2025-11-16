# 集成和测试提示词 / Integration & Testing Prompts

**用途 / Purpose**: 完成项目集成和测试

---

## 提示词 4.1：启动服务 / Prompt 4.1: Start Services

```
按以下顺序启动所有服务：

**Terminal 1: Inngest开发服务器**
```bash
npx inngest-cli@latest dev
```
预期输出：
- Inngest Dev Server running at http://127.0.0.1:8288
- 可以访问UI查看工作流

**Terminal 2: FastAPI后端**
```bash
uvicorn main:app --reload
```
预期输出：
- Uvicorn running on http://127.0.0.1:8000
- Application startup complete

**Terminal 3: Streamlit前端**
```bash
streamlit run streamlit_app.py
```
预期输出：
- You can now view your Streamlit app in your browser
- Local URL: http://localhost:8501
- Network URL: http://192.168.x.x:8501

验证：
- 访问 http://localhost:8501 应看到上传界面
- 访问 http://127.0.0.1:8288 应看到Inngest UI
- 访问 http://127.0.0.1:8000/docs 应看到FastAPI文档
```

---

## 提示词 4.2：测试PDF摄取 / Prompt 4.2: Test PDF Ingestion

```
测试PDF上传和摄取功能：

1. **准备测试PDF**：
   - 找一个PDF文件（例如：test.pdf）
   - 确保文件大小合理（建议<5MB）

2. **上传PDF**：
   - 打开 http://localhost:8501
   - 点击"Choose a PDF"
   - 选择测试PDF文件
   - 应看到："Uploading and triggering ingestion..."
   - 等待成功消息："Triggered ingestion for: test.pdf"

3. **在Inngest UI验证**：
   - 打开 http://127.0.0.1:8288
   - 应看到 "RAG: Ingest PDF" 函数运行
   - 点击查看详情，应看到两个步骤：
     * Step 1: load-and-chunk
     * Step 2: embed-and-upsert
   - 状态应为 "Completed"

4. **检查数据存储**：
   - 检查 uploads/ 目录，应有 test.pdf
   - 检查 qdrant_storage/ 目录，应有数据文件

预期结果：
- 摄取成功，返回 {"ingested": <chunk数量>}
- 文本被分块并存储到Qdrant
- 可以进行查询
```

---

## 提示词 4.3：测试AI查询 / Prompt 4.3: Test AI Query

```
测试问答功能：

1. **提出问题**：
   - 在Streamlit界面的表单中输入问题
   - 例如："What is the main topic of this document?"
   - 设置 top_k=5（默认值）
   - 点击 "Ask" 按钮

2. **等待结果**：
   - 应看到："Sending event and generating answer..."
   - 等待时间取决于文档大小和OpenAI API响应（通常5-30秒）

3. **查看答案**：
   - 应显示 "Answer" 标题
   - 下方显示AI生成的答案
   - 如果有来源，应显示 "Sources" 和PDF文件名列表

4. **在Inngest UI验证**：
   - 打开 http://127.0.0.1:8288
   - 应看到 "RAG: Query PDF" 函数运行
   - 点击查看详情，应看到两个步骤：
     * Step 1: embed-and-search
     * Step 2: llm-answer
   - 状态应为 "Completed"

预期结果：
- 查询成功，返回答案和来源
- 答案基于PDF内容
- 来源列表包含相关PDF文件名
```

---

## 提示词 4.4：验证限流功能 / Prompt 4.4: Verify Rate Limiting

```
测试Inngest限流和节流功能：

1. **测试Rate Limit（每source_id每4小时1次）**：
   - 上传相同的PDF文件两次
   - 第一次应成功
   - 第二次应被限流（如果在4小时内）
   - 检查Inngest UI，应看到限流信息

2. **测试Throttle（每分钟2次）**：
   - 快速上传3个不同的PDF文件
   - 前2个应立即处理
   - 第3个应排队等待
   - 检查Inngest UI，应看到节流效果

3. **验证查询无限流**：
   - 快速提交多个查询
   - 所有查询应正常处理（无限流）

预期结果：
- 限流功能正常工作
- 超过限制的请求被拒绝或排队
- Inngest UI显示限流状态
```

---

## 提示词 4.5：故障排查 / Prompt 4.5: Troubleshooting

```
常见问题和解决方案：

1. **Streamlit无法连接到Inngest**：
   - 错误：ConnectionError
   - 解决：确保Inngest Dev Server正在运行（npx inngest-cli@latest dev）
   - 验证：访问 http://127.0.0.1:8288

2. **OpenAI API错误**：
   - 错误：AuthenticationError或RateLimitError
   - 解决：检查 .env 文件中的 OPENAI_API_KEY
   - 验证：echo $OPENAI_API_KEY 或查看 .env 文件
   - 检查API余额：https://platform.openai.com/account/usage

3. **PDF加载失败**：
   - 错误：PDFReader加载错误
   - 解决：确保PDF文件有效且未加密
   - 尝试使用其他PDF文件

4. **Qdrant连接失败**：
   - 错误：Connection refused (localhost:6333)
   - 解决：项目使用内嵌Qdrant（不需要外部服务器）
   - 检查 qdrant_storage/ 目录权限

5. **Inngest函数未注册**：
   - 错误：Function not found
   - 解决：重启FastAPI服务器
   - 检查Inngest UI的Functions页面

6. **结果轮询超时**：
   - 错误：TimeoutError after 120s
   - 解决：增加超时时间或检查Inngest函数是否卡住
   - 查看Inngest UI的函数执行日志
```

---

**文档生成完成 / Document Generated**: ✅
**下一步 / Next Step**: 生成规格文档
