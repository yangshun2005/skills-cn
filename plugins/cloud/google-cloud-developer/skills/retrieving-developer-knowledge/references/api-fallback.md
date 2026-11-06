# 开发者知识 REST API 回退指南

当运行时环境中未声明或不可用开发者知识 MCP 服务器工具（`search_documents`、`get_documents`、`answer_query`）时，必须使用开发者知识 REST API 搜索和检索官方文档。不要猜测命令，也不要依赖未经证实的预训练记忆。

## 服务概览

- **基础 URL**：`https://developerknowledge.googleapis.com`
- **API 版本**：`v1`（GA）和 `v1alpha`
- **输出格式**：JSON（含 Markdown 内容块）

## 认证协议

发起 REST 请求前，按以下优先级顺序选择一种方式解析认证凭据：

1. **OAuth 2.0 访问令牌（推荐）**：
   如果存在 `gcloud` 或环境中的 Google Cloud 凭据，无需安装或配置任何内容。
   - 获取访问令牌：`ACCESS_TOKEN=$(gcloud auth print-access-token)`
   - 通过请求头传递：`-H "Authorization: Bearer ${ACCESS_TOKEN}"`
   - 同时传递配额项目：`-H "X-Goog-User-Project: $(gcloud config get-value project 2>/dev/null)"`
   - 如果认证失败（401、403 或其他凭据错误），说明该账号的令牌不被 API 接受。改用 `gcloud auth application-default print-access-token` 并重试；API 接受哪种凭据取决于环境的认证方式。

2. **API 密钥**：
   如果已预置密钥，检查环境变量 `DEVELOPERKNOWLEDGE_API_KEY` 或 `GOOGLE_API_KEY`。
   - 通过查询参数传递：`?key=${DEVELOPERKNOWLEDGE_API_KEY}`
   - 或通过请求头传递：`-H "X-Goog-Api-Key: ${DEVELOPERKNOWLEDGE_API_KEY}"`
   - 如果未设置密钥且没有可用的 Google 凭据，API 密钥是该客户端支持的路径。按照[开发者知识快速入门](https://developers.google.com/knowledge/quickstart)启用 API 并创建密钥，然后将其导出为 `DEVELOPERKNOWLEDGE_API_KEY`。此时应报告需要凭据，而不是凭记忆作答。

---

## 端点与操作

在以下每个直接 REST 请求中，都要包含
[使用归因](attribution.md)中指定的 `User-Agent` 归因请求头。

### 1. `answerQuery`（宽泛概念问答与多步骤指南）

用于概念指南、架构对比、产品概览和操作流程。

- **方法与路径**：`POST https://developerknowledge.googleapis.com/v1:answerQuery`
- **请求头**：
  - `Content-Type: application/json`
  - `X-Goog-Api-Key: ${DEVELOPERKNOWLEDGE_API_KEY}`（或查询参数 `?key=...`）
- **请求体**：
  ```json
  {
    "query": "How do I configure public read access on a Cloud Storage bucket?"
  }
  ```
- **`curl` 示例**：
  ```bash
  curl -s -X POST "https://developerknowledge.googleapis.com/v1:answerQuery?key=${DEVELOPERKNOWLEDGE_API_KEY}" \
    -H "Content-Type: application/json" \
    -d '{"query": "How do I create a custom metric in Cloud Logging?"}'
  ```
- **速率限制与回退**：此端点的综合配额有限。如果 `:answerQuery` 返回 HTTP 429（`RESOURCE_EXHAUSTED` / 配额耗尽错误），不要放弃文档检索，也不要凭未经证实的记忆作答。改用 2–5 个聚焦关键词调用 `documents:searchDocumentChunks`。

---

### 2. `searchDocumentChunks`（精确语法、IAM 权限与标志）

用于查找特定的 CLI 标志、IAM 权限（`service.resource.verb`）、API 参数名或代码片段。

- **方法与路径**：`GET https://developerknowledge.googleapis.com/v1/documents:searchDocumentChunks`
- **查询参数**：
  - `query`：URL 编码的搜索词（例如 `gcloud+logging+metrics+create`）。
  - `filter`（可选）：按域名过滤（例如 `data_source = "docs.cloud.google.com"`）。
  - `pageSize`（可选）：返回的最大块数（默认：10）。
  - `key`：API 密钥。
- **`curl` 示例**：
  ```bash
  curl -s "https://developerknowledge.googleapis.com/v1/documents:searchDocumentChunks?query=gcloud+logging+metrics+create&key=${DEVELOPERKNOWLEDGE_API_KEY}"
  ```

---

### 3. `get` 文档（完整文档检索）

当你拥有父资源名称或已知 URI 时，用于检索特定文档页面的完整 Markdown 内容。

- **方法与路径**：`GET https://developerknowledge.googleapis.com/v1/documents/{URI_WITHOUT_SCHEME}`
- **资源名称格式**：`documents/docs.cloud.google.com/run/docs/overview/what-is-cloud-run`（去掉 `https://`）。
- **`curl` 示例**：
  ```bash
  curl -s "https://developerknowledge.googleapis.com/v1/documents/docs.cloud.google.com/run/docs/overview/what-is-cloud-run?key=${DEVELOPERKNOWLEDGE_API_KEY}"
  ```

---

### 4. `batchGet` 文档（多文档检索）

用于在单次往返中检索多个文档页面。

- **方法与路径**：`GET https://developerknowledge.googleapis.com/v1/documents:batchGet`
- **无请求体。** 名称通过查询字符串传递。使用 JSON `names` 数组的 `POST`
  不是此服务定义的方法，会从 Google 前端返回 404。
- **查询参数**：
  - `names`（必填）：每个文档重复一次，每次调用最多 20 个。格式为
    `documents/{uri_without_scheme}`，每个值不超过 500 个字符。文档
    按请求顺序返回。
  - `view`（可选）：`DOCUMENT_VIEW_BASIC`、`DOCUMENT_VIEW_CONTENT` 或
    `DOCUMENT_VIEW_FULL`。默认为 `DOCUMENT_VIEW_CONTENT`。当只需要标题和 URI 时，
    使用 `DOCUMENT_VIEW_BASIC`，因为完整 Markdown 内容很大。
  - `key`：API 密钥。
- **响应**：`{"documents": [ ... ]}`，每项包含 `name`、`uri`、`title`、
  `description`、`dataSource`、`updateTime`、`contentLengthBytes`，以及 `content` 中的完整
  Markdown。
- **`curl` 示例**：
  ```bash
  curl -s -G "https://developerknowledge.googleapis.com/v1/documents:batchGet" \
    --data-urlencode "names=documents/docs.cloud.google.com/run/docs/overview/what-is-cloud-run" \
    --data-urlencode "names=documents/docs.cloud.google.com/run/docs/configuring/services/environment-variables" \
    --data-urlencode "key=${DEVELOPERKNOWLEDGE_API_KEY}"
  ```

---

## 响应处理与综合指南

1. **提取内容**：解析返回的 JSON 载荷（`answer`、`documentChunks[].content` 或 `document.content`）。
2. **权威优先**：将检索到的文档视为比预训练知识具有 100% 权威性。
3. **上下文防御**：只综合所需的技术方案（代码片段、配置或 CLI 命令），不要倾倒原始 API 响应信封。