# Cloud CLI 远程 MCP 服务器使用说明

Google Cloud 资源可通过 Model Context Protocol (MCP) 管理，让 AI 代理使用结构化工具调用与 Google Cloud 交互，而非直接执行本地 shell 命令。

`gcloud` 的 MCP 操作通过 **Cloud CLI 远程 MCP 服务器**执行（由 Cloud CLI Execution API `cloudcli.googleapis.com` 提供支持）。

## 服务器端点与工具概览

-   **服务器端点：** `https://cloudcli.googleapis.com/mcp`
-   **传输方式：** HTTP (JSON-RPC 2.0)
-   **API 名称：** Cloud CLI Execution API (`cloudcli.googleapis.com`)
-   **可用工具：** `run_gcloud_command`

`run_gcloud_command` 工具代表用户在受管的远程环境中安全执行单条 `gcloud` 命令。

## 客户端配置（`mcp_config.json`）

要将 MCP 客户端（如 Jetski）连接到远程 Cloud CLI MCP 服务器，请在 `mcp_config.json` 中配置服务器条目，并将 `authProviderType` 设为 `"google_credentials"`：

```json
{
  "mcpServers": {
    "gcloud-remote": {
      "serverUrl": "https://cloudcli.googleapis.com/mcp",
      "authProviderType": "google_credentials"
    }
  }
}
```

> [!IMPORTANT] 必须指定 `"authProviderType": "google_credentials"`。它指示 MCP 客户端附加具有 `https://www.googleapis.com/auth/cloud-platform` OAuth 范围的 Application Default Credentials (ADC)。省略此字段会导致客户端发送未认证请求，从而产生 `401 Unauthorized` 错误。

## 前提条件与 IAM 要求

使用 Cloud CLI 远程 MCP 服务器前，目标项目和调用身份必须满足两项强制前提条件：

### 1. 启用 API

必须在目标项目上启用 Cloud CLI Execution API (`cloudcli.googleapis.com`)。

-   **通过 Google Cloud Console（无需 CLI）：**

    1.  打开 [Google Cloud Console](https://console.cloud.google.com/)。
    2.  导航到 **APIs & Services** --> **Library**。
    3.  搜索 **Cloud CLI Execution API**（或打开
        [Cloud CLI Execution API Library 页面](https://console.cloud.google.com/apis/library/cloudcli.googleapis.com)）。
    4.  从项目下拉列表中选择目标项目。
    5.  点击 **Enable**。

-   **通过 `gcloud` CLI：**

    ```bash
    gcloud services enable cloudcli.googleapis.com --project={project_id}
    ```

### 2. IAM 角色与权限

-   **MCP 访问角色：** 调用身份必须在目标项目上拥有 **MCP Tool User** 角色（`roles/mcp.toolUser`，授予 `mcp.tools.call` 权限）。
-   **下游资源角色：** 调用身份还必须对所查询或修改的底层资源拥有标准 IAM 权限（例如 `roles/compute.viewer`、`roles/run.developer`）。

> [!CAUTION] 如果未启用 Cloud CLI Execution API 或调用方缺少 `roles/mcp.toolUser` 角色，端点将在工具发现（`tools/list`）和工具调用（`tools/call`）期间返回 **`403 Forbidden`**。

## 工具参数

调用 `run_gcloud_command` 接受以下参数：

-   **`command`**（字符串，必填）：要执行的完整 `gcloud` 命令行字符串（例如 `"gcloud compute instances list --project={resource_project} --format=json"`）。
-   **`project`**（字符串，必填）：托管 Cloud CLI Execution API 的 Google Cloud 项目的资源名称，格式为 `"projects/{api_project}"`（例如 `"projects/my-api-project"`）。
-   **`input_files`**（对象列表，可选）：运行命令前在远程执行环境中预置的文件。每项包含相对 `path` 和字符串 `contents`。

> [!IMPORTANT] **API 宿主项目与资源项目上下文：**
>
> -   顶层 **`project`** 参数（`"projects/{api_project}"`）**仅用于** `cloudcli.googleapis.com` API 本身的配额、计费和 API 启用。它不会为所执行的命令设置项目上下文。
> -   对于**项目范围命令**，必须在 `command` 字符串中显式包含 `--project={resource_project}`。目标 `{resource_project}` 不必是托管 Cloud CLI Execution API 的项目。
> -   对于**非项目范围命令**（如计费或组织查询），如果底层 API 需要配额项目，必须在 `command` 字符串中包含 `--billing-project={billing_project}`。

### 调用示例

#### 1. 基本命令执行

```json
{
  "command": "gcloud compute instances list --project=my-resource-project --format=json",
  "project": "projects/my-cloudcli-api-project"
}
```

#### 2. 带输入文件的命令

```json
{
  "command": "gcloud run services replace service-config.yaml --region=us-central1 --project=my-resource-project",
  "project": "projects/my-cloudcli-api-project",
  "input_files": [
    {
      "path": "service-config.yaml",
      "contents": "apiVersion: serving.knative.dev/v1\nkind: Service\nmetadata:\n  name: my-service\n..."
    }
  ]
}
```

## 响应结构

工具返回的执行响应包含：

-   `exit_code`：命令执行的数字退出状态。**这是命令成功或失败的主要且权威指标。**
-   `stdout`：命令的标准输出流。
-   `stderr`：命令的标准错误流。
-   `output_files`：命令生成的任何文件。

> [!NOTE] - **退出码权威性：** 当且仅当 `exit_code == 0` 时命令成功。非零 `exit_code` 表示失败。
>
> -   **信息性 `stderr` 输出：** 在 `gcloud` 中，即使命令成功执行（`exit_code == 0`），`stderr` 也经常包含标准状态消息、进度更新和异步跟踪 ID（如 `--async` 操作 ID）。代理不得仅因 `stderr` 非空就假定命令失败。
> -   **错误诊断：** 如果 `exit_code != 0`，诊断错误消息可能出现在 `stderr` 或 `stdout` 中。检查两个流以理解失败原因并制定纠正措施。

## 禁止与不支持的命令

Cloud CLI 远程 MCP 服务器在沙盒化的非交互环境中运行。以下列表展示了一些不支持的 `gcloud` 命令示例（例如管理本地机器配置、凭据、交互式 shell 或元数据的命令组）。此列表并非详尽无遗，命令可能随时增删，恕不另行通知：

-   `gcloud auth`（本地认证与凭据管理）
-   `gcloud config`（本地 CLI 配置配置文件和属性）
-   `gcloud iam service-accounts`（服务账号管理）
-   `gcloud init`（交互式设置向导）
-   `gcloud survey`（用户反馈与调查）
-   `gcloud compute ssh` / `gcloud app instances ssh`（交互式 SSH shell）

## 安全与执行准则

-   **变更操作必须获得用户同意：** 破坏性或改变状态的命令（如 `create`、`delete`、`update` 或 `patch`）会修改或销毁 GCP 资源。除非用户明确授权该操作，否则不得自主调用这些命令。
-   **异步操作（`--async`）：** 对于长时间运行的操作（如创建 VM 实例、GKE 集群或数据库实例），始终在 `command` 字符串中追加 `--async` 标志，以避免执行超时。
-   **数据缩减与格式化：** 在 `command` 字符串中使用 `--format=json`、`--filter` 和 `--limit` 限制输出量，防止上下文窗口膨胀。
-   **非交互式执行（`--quiet`）：** 对可能提示交互式用户确认的命令，包含 `--quiet`（或 `-q`）。