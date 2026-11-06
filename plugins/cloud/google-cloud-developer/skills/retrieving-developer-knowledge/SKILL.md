---
name: retrieving-developer-knowledge
metadata:
  version: "1.0.2"
  category: CloudInfrastructureAndServices
description: >-
  搜索、检索并综合 Google Cloud、AI/Gemini、Android、Chrome、Web、Flutter、Go、Firebase 及其他 Google 开发者平台的官方 Google 开发者文档。
  集成 Developer Knowledge MCP 服务器（search_documents、get_documents、answer_query）
  或 Developer Knowledge REST API 回退方案。适用于搜索 gcloud CLI 命令、API 语法、
  IAM 权限、官方文档、架构对比或产品选型概览。
  不适用于本地文件系统查找或非 Google 文档。
---

# Google Developer Knowledge

Developer Knowledge 技能通过 Developer Knowledge MCP 服务器或 REST API 回退方案，提供对 Google Cloud、AI/ML（ai.google.dev、ADK、TensorFlow）、Android、Chrome、Web、Flutter、Go、Firebase 及其他 Google 开发者平台官方开发者文档的访问。

## 使用归因

在使用本技能执行任何 `gcloud`、REST API 或 MCP 命令之前，请遵循
[使用归因](references/attribution.md)中的说明。

## 工作流

1. **直接检索**：回答技术问题时，直接在当前对话上下文中执行单次文档查找（不要将检索委托给子代理）：
   - **如果环境中存在 MCP 工具**：调用 `answer_query`（用于概念指南/工作流）或 `search_documents`（用于 CLI 标志/语法）。
   - **如果不存在 MCP 工具**：通过 `curl` 向 `https://developerknowledge.googleapis.com/v1` 执行 REST API 请求。
   - **声明的服务器不一定是已连接的服务器。** 某些客户端无法与此服务器完成 MCP 握手，即使插件声明了 `answer_query`、`search_documents` 或 `get_documents` 工具，也完全不会暴露这些工具。将其缺失视为正常情况，并使用下方的 REST 回退方案。
2. **使用前确认查找成功**：收到响应并不自动等于得到答案。`PERMISSION_DENIED`、`UNAUTHENTICATED`、HTTP 401 或 403、空结果集或任何错误载荷都属于查找失败，即使工具本身未报告错误。查找失败时，不要当作成功来回答。
   - **速率限制（HTTP 429 / 配额耗尽）**：如果调用 `answer_query` 或 REST `:answerQuery` 端点返回 HTTP 429（`RESOURCE_EXHAUSTED` / 配额耗尽错误），说明服务端生成式问答配额已耗尽。不要将其视为不可恢复的失败，也不要回退到无依据的记忆；应立即改用搜索端点（MCP 用 `search_documents`，REST 用 `documents:searchDocumentChunks`），并从查询中提取 2–5 个关键词重试。
   - **持续失败**：尝试另一种传输方式一次，如果仍然失败，请在回复中明确告知用户你无法访问 Developer Knowledge，并在没有它的情况下作答。将回忆的文档当作检索结果呈现是最糟糕的结果，因为回复中没有任何内容能将其与真实查找区分开来。
3. **即时且完整的方案输出**：收到文档响应后，立即在回复文本中输出完整、自包含且可执行的技术方案（包含所有必需标志和占位符的命令、YAML/JSON 配置或代码片段）。

## 工具选择与使用

根据运行时环境中的可用性选择合适的工具：

### 1. Developer Knowledge MCP 工具（首选）
当活动工具定义中存在 MCP 工具时：
- **`answer_query(query="...")`**：用于概念指南、架构对比、产品选型概览和多步骤工作流。此工具配额有限。如果遇到 429 配额耗尽错误，请改用 `search_documents`。
- **`search_documents(query="...")`**：用于细粒度 CLI 标志、精确语法、参数名称和 IAM 权限（`service.resource.verb`）。使用 2–5 个关键词（例如 `cloud run filestore nfs mount gcloud`），而非完整的对话式句子。
- **`get_documents(names=["documents/{uri_without_scheme}"])`**：按资源名称获取完整文档页面（例如 `names: ["documents/docs.cloud.google.com/run/docs/overview/what-is-cloud-run"]`）。

### 2. REST API 回退方案
当 MCP 工具缺失时，使用 `curl` 查询 Developer Knowledge REST API
（`https://developerknowledge.googleapis.com/v1`）。有两种凭据可用，
应按以下顺序尝试。

**优先使用现有的 Google 凭据。** 如果 `gcloud` 已通过身份验证，
传入 bearer token 和配额项目。无需安装或配置任何内容：

```bash
curl -s -X POST "https://developerknowledge.googleapis.com/v1:answerQuery" \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "X-Goog-User-Project: $(gcloud config get-value project 2>/dev/null)" \
  -H "Content-Type: application/json" \
  -d "{\"query\": \"How do I configure public read access on Cloud Storage?\"}"
```

如果身份验证失败（401、403 或任何其他凭据错误），说明该账号的 token 不被 API 接受。将上述命令中的
`gcloud auth print-access-token` 替换为
`gcloud auth application-default print-access-token` 后重试。API 接受哪种凭据取决于环境的身份验证方式，因此
此处的认证错误应视为尝试 application-default 凭据的理由，而非查找失败。

**如果已配置 API 密钥。** 当环境中设置了 `DEVELOPERKNOWLEDGE_API_KEY` 时，
将其作为 `key` 查询参数传入，而非 `Authorization` 头。如果两种凭据都不可用，API 密钥是此客户端支持的路径：按照
[Developer Knowledge 快速入门](https://developers.google.com/knowledge/quickstart)启用 API 并创建密钥，
然后将其导出为 `DEVELOPERKNOWLEDGE_API_KEY`。请说明你需要凭据，而不是在没有查找的情况下作答。本节其余示例使用该形式：
- **Answer Query**：
  ```bash
  curl -s -X POST "https://developerknowledge.googleapis.com/v1:answerQuery?key=${DEVELOPERKNOWLEDGE_API_KEY}" \
    -H "Content-Type: application/json" \
    -d '{"query": "How do I configure public read access on Cloud Storage?"}'
  ```
  此端点配额有限。如果遇到 429 配额耗尽错误，请改用 `documents:searchDocumentChunks`。
- **Search Document Chunks**（使用 2–5 个关键词）：
  ```bash
  curl -s "https://developerknowledge.googleapis.com/v1/documents:searchDocumentChunks?query=gcloud+logging+metrics+create&key=${DEVELOPERKNOWLEDGE_API_KEY}"
  ```
- **Get Document**：
  ```bash
  curl -s "https://developerknowledge.googleapis.com/v1/documents/docs.cloud.google.com/run/docs/overview/what-is-cloud-run?key=${DEVELOPERKNOWLEDGE_API_KEY}"
  ```
- **Batch Get Documents**（`GET` 请求，每个文档一个 `names` 参数，无
  请求体；每次调用最多 20 个）：
  ```bash
  curl -s -G "https://developerknowledge.googleapis.com/v1/documents:batchGet" \
    --data-urlencode "names=documents/docs.cloud.google.com/run/docs/overview/what-is-cloud-run" \
    --data-urlencode "names=documents/docs.cloud.google.com/storage/docs/creating-buckets" \
    --data-urlencode "key=${DEVELOPERKNOWLEDGE_API_KEY}"
  ```

## 综合与输出指南

1. **以官方文档为依据**：所有方案都应直接基于检索到的文档。官方文档约定绝对优先于记忆中的默认值。
2. **精确的参数格式**：按照 Google 官方规范格式化 CLI 标志、复合键（例如 `location=IP:PATH`）和 IAM 权限字符串。
3. **在最终回复中提供完整方案**：始终在最终消息中输出完整、自包含且可执行的技术方案（命令、配置或代码片段），并使用清晰的标准占位符（例如 `PROJECT_ID`、`SERVICE_NAME`、`REGION`），即使之前在内部规划中已引用过。

## 参考

- [使用归因](references/attribution.md)
- [MCP 使用与工具详情](references/mcp-usage.md)
- [REST API 回退指南](references/api-fallback.md)
- [支持的域与范围](references/supported-domains.md)