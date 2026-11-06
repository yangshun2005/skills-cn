# Google Cloud Developer 插件

`google-cloud-developer` 插件为 AI 编码代理提供在 Google Cloud 上构建应用所需的基础技能、安全护栏和实时文档依据。

本插件基于开放的 [Agent Plugins 规范](https://agent-plugins.org/) 构建，将精选的 Agent Skills、路由规则以及 **Developer Knowledge MCP 服务器** 的配置打包为单一可移植包。

--------------------------------------------------------------------------------

## 安装

### Antigravity

从 Antigravity Marketplace 安装插件：

1. 在 Antigravity 2.0 中，进入 Customizations > Marketplace，搜索 "Google Cloud Developer"。
2. 点击 **Add** 按钮。

也可以使用 Antigravity CLI 安装插件：

```bash
agy plugin install google-cloud-developer
```

### Claude Code

1.  添加 Google 插件市场，然后安装插件：

    ```bash
    claude plugin marketplace add google/skills
    claude plugin install google-cloud-developer@google-plugins
    ```

1.  使用 gcloud CLI 在 Google Cloud 项目中启用 Developer Knowledge API：

    ```bash
    gcloud services enable developerknowledge.googleapis.com --project=<YOUR_PROJECT_ID>
    ```

1.  按照 [创建并保护 API 密钥](https://developers.google.com/knowledge/quickstart#create-secure-key) 的说明，为 Developer Knowledge API 创建 API 密钥。将创建的密钥保存到安全位置。

1.  在启动 Claude Code 之前，将 API 密钥导出到环境变量：

    ```bash
    export DEVELOPERKNOWLEDGE_API_KEY=<YOUR_API_KEY>
    ```

    > [!NOTE] 必须在启动 Claude Code 之前设置 `DEVELOPERKNOWLEDGE_API_KEY` 环境变量。为方便起见，可以将此导出语句添加到 shell 的启动脚本（如 `.bashrc`、`.zshrc`）中。

### Codex CLI

1.  添加 Google 插件市场，然后安装插件：

    ```bash
    codex plugin marketplace add google/skills
    codex plugin add google-cloud-developer@google-plugins
    ```

1.  使用 gcloud CLI 在 Google Cloud 项目中启用 Developer Knowledge API：

    ```bash
    gcloud services enable developerknowledge.googleapis.com --project=<YOUR_PROJECT_ID>
    ```

1.  按照 [创建并保护 API 密钥](https://developers.google.com/knowledge/quickstart#create-secure-key) 的说明，为 Developer Knowledge API 创建 API 密钥。将创建的密钥保存到安全位置。

1.  更新 `~/.codex/config.toml`（或项目的 `.codex/config.toml`），加入以下内容，以启用对 Developer Knowledge MCP 服务器的认证访问：

    ```toml
    [mcp_servers.developer-knowledge]
      url = "https://developerknowledge.googleapis.com/mcp"
      env_http_headers = { "X-Goog-Api-Key" = "DEVELOPERKNOWLEDGE_API_KEY" }
    ```

1.  在启动 Codex 之前，将 API 密钥导出到环境变量：

    ```bash
    export DEVELOPERKNOWLEDGE_API_KEY=<YOUR_API_KEY>
    ```

    > [!NOTE] 必须在启动 Codex 之前设置 `DEVELOPERKNOWLEDGE_API_KEY` 环境变量。为方便起见，可以将此导出语句添加到 shell 的启动脚本（如 `.bashrc`、`.zshrc`）中。

    > [!NOTE] 注意：更改配置后，服务器可能显示 *未登录* 状态。这是预期行为，不影响访问。

--------------------------------------------------------------------------------

## 包含内容

### 内置技能

-   **[gcloud](./skills/gcloud)**：`gcloud` CLI 命令的安全关键校验、最佳实践和执行护栏。
-   **[google-cloud-recipe-auth](./skills/google-cloud-recipe-auth)**：认证工作流、凭据选择和服务身份最佳实践。
-   **[google-cloud-recipe-onboarding](./skills/google-cloud-recipe-onboarding)**：首个项目入门、账号设置和结算配置。
-   **[finding-google-skills](./skills/finding-google-skills)**：从 Google Agent Skills 仓库发现并按需安装专用技能。
-   **[retrieving-developer-knowledge](./skills/retrieving-developer-knowledge)**：基于官方 Google 文档的依据检索。

### MCP 服务器

-   **[Developer Knowledge MCP 服务器](https://developers.google.com/knowledge/mcp)**：通过可流式 HTTP 为 AI 代理提供官方 Google 开发者文档的最新依据。

### 路由规则

-   **[google-cloud-discovery.md](./rules/google-cloud-discovery.md)**：常驻路由映射，当任务需要超出基础包的专业领域知识时，引导代理发现并建议更深入的产品专用技能（如 `gke-*`、`agent-platform-*`、`google-cloud-waf-*` 和 `<product>-basics`）。

--------------------------------------------------------------------------------

## 插件实战

要了解此插件如何工作，设想你在编写脚本时需要引导一个新项目。安装 `google-cloud-developer` 插件后，你可以向代理提示：

> *我是这个平台的新手，需要开通账号并设置好首个项目和结算。然后，我需要对本机进行认证，以便我正在编写的脚本能以服务身份而非我的身份调用 API。*

1.  **环境感知**：代理会静默地对你的实时环境运行后台检查，确认 CLI 可用性等前置条件，以及可能已存在的项目或组织。

2.  **审查**：代理会考虑 IAM 最佳实践，避免提示中可能隐含的风险，例如意外的密钥泄露或 git 提交。

3.  **护栏交互**：代理会列出工作流路线图，并在修改任何资源之前提议执行这些步骤。

--------------------------------------------------------------------------------

## 网络、沙箱和权限要求

如果在本地沙箱或企业防火墙/代理后运行代理，请配置环境以允许以下出站 URL 和终端命令。

> [!NOTE] 核心内置技能（`gcloud`、`google-cloud-recipe-auth`、`google-cloud-recipe-onboarding`）和 `developer-knowledge` MCP 服务器仅需要 `gcloud` 以及对 `*.googleapis.com`、`docs.cloud.google.com` 和 `developers.google.com` 的 HTTPS 访问。对 GitHub 域名（`raw.githubusercontent.com`、`api.github.com`）和 `curl` 的访问仅在运行时用于动态技能发现（`finding-google-skills`）以及 `retrieving-developer-knowledge` 中的 REST API 回退。动态获取的技能可能需要额外的产品专用 CLI 工具或 API 端点。

### 1. 必需出站 URL 允许列表

| 域名 / 端点 | 使用方 | 用途 |
| :--- | :--- | :--- |
| `https://raw.githubusercontent.com`（`githubusercontent.com`） | `finding-google-skills` | 从 `google/skills` 获取 `index.json` 目录以及按需的 `SKILL.md` / 参考文件。 |
| `https://github.com`、`https://api.github.com` | 安装器、`finding-google-skills`、`google-cloud-discovery` | 安装插件并查询 GitHub Trees API 回退（`/repos/google/skills/git/trees/main`）。 |
| `https://developerknowledge.googleapis.com` | `developer-knowledge` MCP、`retrieving-developer-knowledge` | 连接远程 MCP 服务器（`/mcp`）和 REST API v1 回退（`/v1/documents:*`、`/v1:answerQuery`）。 |
| `https://*.googleapis.com` | `gcloud`、`google-cloud-recipe-auth`、`google-cloud-recipe-onboarding` | 认证并针对 Google Cloud API 执行 `gcloud` CLI 命令。 |
| `https://docs.cloud.google.com`、`https://developers.google.com` | `gcloud`、`google-cloud-recipe-auth`、`google-cloud-recipe-onboarding` | 读取官方 Google Cloud 文档（`*.md.txt`）。 |

### 2. 必需终端命令权限

-   **技能发现与 REST 回退**（`finding-google-skills`、`retrieving-developer-knowledge`）：`curl`（`curl -sSL`、`curl -s -X POST`、`curl -s -G`）、`jq`、`grep`，以及可选的 `npx skills`（仅在本地安装发现的技能时需要）。
-   **只读 `gcloud` 检查**（`gcloud`、`google-cloud-recipe-onboarding`）：`which gcloud`、`gcloud help`、`gcloud topic`、`gcloud cheat-sheet`、`gcloud auth list`、`gcloud config get-value`、`gcloud config list`、`gcloud organizations list`、`gcloud projects list`、`gcloud projects describe`、`gcloud billing projects describe`、`gcloud billing accounts list`、`gcloud compute regions list`、`gcloud compute zones list`。
-   **交互式或改变状态的 `gcloud` 命令**（`google-cloud-recipe-onboarding`、`google-cloud-recipe-auth`）：`gcloud auth login`、`gcloud auth application-default login`、`gcloud config set`、`gcloud projects create`、`gcloud billing projects link`、`gcloud services enable`。不要自动允许这些命令；将其置于确认（**Ask**）之后，或事先手动运行。