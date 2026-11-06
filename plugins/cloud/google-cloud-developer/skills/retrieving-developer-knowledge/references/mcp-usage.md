# MCP 工具用法与 API 详情

Developer Knowledge 技能集成 Developer Knowledge Remote MCP 服务器（`https://developerknowledge.googleapis.com/mcp`）。

## 可用工具

1.  **`search_documents`**：

    -   接受搜索查询字符串。
    -   返回包含匹配语法、代码块或标志的相关文档文本片段。
    -   每个返回项包含 `content` 字段和 `parent` URI 字段。

2.  **`answer_query`**：

    -   接受自然语言问题。
    -   在服务端执行 RAG，返回带来源引用的综合回答。
    -   此工具有配额限制。若遇到 429 配额超限错误，请改用 `search_documents`。

3.  **`get_documents`**：

    -   获取完整文档内容。
    -   接受 `names` 数组参数：`names: ["documents/{uri_without_scheme}"]`。
    -   示例：对于父 URI `https://cloud.google.com/run/docs/deploying`，传入 `names: ["documents/cloud.google.com/run/docs/deploying"]`。