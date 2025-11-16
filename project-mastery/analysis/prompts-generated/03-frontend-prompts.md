# 前端重建提示词 / Frontend Rebuild Prompts

**用途 / Purpose**: 重建Streamlit前端界面

---

## 提示词 3.1：创建Streamlit应用 / Prompt 3.1: Create Streamlit App

```
创建 streamlit_app.py 文件，实现以下功能：

1. **页面配置**：
   - 标题：RAG Ingest PDF
   - 图标：📄
   - 布局：居中（centered）

2. **PDF上传功能**：
   - 文件上传器（只接受PDF，单个文件）
   - 保存到 uploads/ 目录
   - 发送Inngest事件 "rag/ingest_pdf"
   - 显示成功消息

3. **问答功能**：
   - 表单包含：
     * 文本输入框（问题）
     * 数字输入框（检索数量top_k，范围1-20，默认5）
     * 提交按钮（"Ask"）
   - 发送Inngest事件 "rag/query_pdf_ai"
   - 轮询等待结果（超时120秒，间隔0.5秒）
   - 显示答案和来源

4. **辅助函数**：
   - get_inngest_client(): 缓存的Inngest客户端
   - save_uploaded_pdf(): 保存上传的PDF
   - send_rag_ingest_event(): 发送摄取事件
   - send_rag_query_event(): 发送查询事件
   - fetch_runs(): 获取Inngest运行记录
   - wait_for_run_output(): 轮询等待结果

参考文件：project-mastery/analysis/03-frontend-analysis.md
完整代码位置：streamlit_app.py (127行)
```

---

## 提示词 3.2：关键实现细节 / Prompt 3.2: Key Implementation Details

```
Streamlit应用的关键实现要点：

1. **Inngest客户端缓存**：
```python
@st.cache_resource
def get_inngest_client() -> inngest.Inngest:
    return inngest.Inngest(app_id="rag_app", is_production=False)
```
使用 @st.cache_resource 确保客户端单例。

2. **异步事件发送**：
```python
event_id = asyncio.run(send_rag_query_event(question, top_k))
```
使用 asyncio.run() 在同步Streamlit中调用异步函数。

3. **结果轮询**：
通过Inngest API轮询结果：
- URL: http://127.0.0.1:8288/v1/events/{event_id}/runs
- 检查状态：Completed, Succeeded, Success, Finished
- 失败状态：Failed, Cancelled
- 超时：120秒

4. **加载状态反馈**：
```python
with st.spinner("Sending event and generating answer..."):
    # 执行操作
    ...
```

5. **结果显示**：
```python
st.subheader("Answer")
st.write(answer or "(No answer)")
if sources:
    st.caption("Sources")
    for s in sources:
        st.write(f"- {s}")
```
```

---

**文档生成完成 / Document Generated**: ✅
