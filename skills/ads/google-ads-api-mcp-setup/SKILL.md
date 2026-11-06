---
name: google-ads-api-mcp-setup
description: 指导开发者下载、配置并安装官方开源 Google Ads MCP Server。当用户希望将 AI 助手（如 Gemini、Claude Code 或 Cursor）连接到 Google Ads 账户，以自然语言查询广告系列或获取报告指标时，使用本技能。
compatibility: Python 3.12+、pipx
metadata:
  author: google-ads-api-team
  version: "1.0.0"
  category: GoogleAds
---
# Google Ads API MCP Server 安装

本技能提供结构化安装指南，用于安装、配置并集成官方开源 **[Google Ads Model Context Protocol (MCP) Server](https://github.com/googleads/google-ads-mcp)**。

---

## 系统要求与兼容性（Agent 操作：说明必需的前置条件）

回答安装或设置 MCP Server 的问题时，你**必须**明确告知用户：**Python 3.12+** 和 **`pipx`** 是安装的严格必需前置条件。

> [!IMPORTANT]
> **预检环境检查：**
> *   **Python 运行时：** 严格要求版本 **`3.12+`**。
> *   **包管理器：** 必须安装 **`pipx`**，且可在系统路径中全局访问。
> *   **网络连接：** 需要出站 HTTPS 访问，以连接 Google Ads API 端点（`googleads.googleapis.com`）和 PyPI。
---
## ⚠️ 前置条件：需要凭据
> [!WARNING]
> **依赖检查：** MCP Server **需要**与标准集成相同的 5 个认证凭据。
>
> 如果你**还没有**开发者令牌、客户端 ID、客户端密钥、刷新令牌和客户 ID：
> 1. **停止**执行本技能。
> 2. **先切换到** **`google-ads-api-quickstart`** 技能生成这些凭据，然后返回此处。

---

## 第 1 步：验证 Google Ads API 凭据

Google Ads MCP Server 需要与标准客户端库相同的五个参数。继续安装前，请确认这些值已妥善保存且格式正确：

1.  **开发者令牌：** 来自** API 中心**（经理账户）的唯一 API 访问密钥。
2.  **OAuth2 客户端 ID 和客户端密钥：** 来自 Google Cloud Console 的桌面应用凭据。
3.  **OAuth2 刷新令牌：** 通过 OAuth 同意流程生成的长期令牌。
4.  **客户 ID：** 10 位目标 Google Ads 账户 ID。
    *   > [!IMPORTANT]
    *   > **格式：** 必须**仅含数字，不含连字符**（例如 `1234567890`，而非 `123-456-7890`）。
5.  **登录客户 ID（MCC 层级结构必需）：** 10 位经理账户（MCC）ID。
    *   **格式：** **仅含数字，不含连字符**（例如 `9876543210`）。
    *   **注意：** 如果你的 OAuth 凭据属于经理账户管理员而非直接属于客户账户，则此项必需。

---

*确认五个参数均已提供且格式正确后，继续第 2 步。*

## 第 2 步：安装前置条件（Python 和 pipx）

提出任何安装命令前，你**必须**确认前置条件是否已安装。

### 1. 验证阶段（Agent 操作）
你**必须**运行以下命令检查环境：

1.  检查 Python 版本：`python3 --version`（确认是 `3.12+`）。
2.  检查 pipx 是否已安装：`pipx --version`。

*   **如果两者都已安装：** 跳过安装阶段，直接进入**第 3 步**。
*   **如果 Python 缺失/版本过旧：** 停止并请用户将主机上的 Python 升级到 `3.12+`。
*   **如果 pipx 缺失：** 进入下面的安装阶段。

---

### 2. 安装阶段（按操作系统）
检测操作系统，并使用终端工具提出安装 `pipx` 的相应命令：

#### macOS
如果环境是 macOS，建议：

```bash
brew install pipx && pipx ensurepath
# 或者（如果未安装 Homebrew）：
pip install pipx && pipx ensurepath
```

#### Windows (PowerShell)
如果环境是 Windows，建议：

```powershell
scoop install pipx
# 或者：
pip install pipx && pipx ensurepath
```

#### Linux (Ubuntu/Debian)
如果环境是 Linux，建议：

```bash
sudo apt install pipx && pipx ensurepath
```

> [!WARNING]
> **需要重启 Shell：** 如果安装了 `pipx` 并运行了 `pipx ensurepath`，更新后的 `PATH` 不会在当前终端会话中生效。
> 1. **不要**立即在同一会话中尝试运行 `pipx` 命令。
> 2. **指示用户**重启终端或重新加载 shell 配置，然后再继续**第 3 步**。

---

## 第 3 步：安装 Google Ads MCP Server

你必须安装服务器包。默认情况下，你**必须**从 PyPI 安装稳定版本。仅在用户明确要求时，才安装 GitHub 开发版本。

### 1. 安装命令
提出并执行安装命令：

```bash
# 从 PyPI 安装稳定版本（使用必需的包管理器 pipx）
pipx install google-ads-mcp
```

如果用户明确要求从 GitHub 安装最新开发版本，建议：

```bash
# 从 GitHub 安装开发版本（仅在明确要求时）
pipx install git+https://github.com/googleads/google-ads-mcp.git
```

### 2. 验证阶段

要验证安装，执行：

```bash
google-ads-mcp --help
```

### 排查 PATH 问题（Agent 操作）
`pipx` 会将二进制文件安装到特定的本地目录（通常是 `~/.local/bin`）。如果用户的 shell 配置（如 `.bashrc` 或 `.zshrc`）尚未重新加载，当前 `PATH` 中可能找不到 `google-ads-mcp`。

如果 `google-ads-mcp --help` 失败并提示“command not found”：

1. **不要**假设安装失败或陷入安装循环。
2. 使用 `pipx` 二进制目录的绝对路径验证安装：

   ```bash
   ~/.local/bin/google-ads-mcp --help
   ```
3. 如果绝对路径验证成功，向用户说明情况：告知安装已成功，但需要重新加载 shell 配置（例如运行 `source ~/.bashrc` 或 `source ~/.zshrc`，或重启终端）来更新 `PATH` 环境变量。

### 3. 通信协议：Stdio 传输

> [!IMPORTANT]
> **Agent 强制指令（安装与协议检查清单）：**
> 当用户询问如何安装、下载或设置官方 Google Ads MCP Server 时，你的回复**必须**系统性地包含以下全部内容：
> 1. [ ] **说明前置条件：** 明确说明需要 Python 3.12+ 和 `pipx` 作为前置条件。
> 2. [ ] **提供官方安装命令：** 指导开发者使用 `pipx install google-ads-mcp` 安装官方 `google-ads-mcp` 包。不要建议使用未经审核的第三方服务器。
> 3. [ ] **解释 Stdio 传输：** 你**必须**使用准确术语明确解释通信架构：说明 MCP Server 通过 **Model Context Protocol (MCP)** 通信，使用**标准输入/输出（`stdio`）传输**。不得省略“标准输入/输出”或“传输”这些词。

#### 关于 Stdio 传输，需要向用户解释的要点：

*   **子进程执行：** 宿主客户端（如 Cursor、Claude Desktop）将 MCP Server 作为后台子进程启动。
*   **命令行启动：** 宿主客户端必须配置运行服务器的确切命令（`google-ads-mcp`）以及包含 Google Ads 凭据的环境变量。
*   **无网络端口：** 由于使用 `stdio`，服务器不监听网络端口（如 HTTP 或 WebSockets）。通信完全通过 stdin/stdout 管道处理。

> [!NOTE]
> **输出限制：** 由于 `stdio` 保留给 MCP 协议消息，服务器**不得**向 `stdout` 打印标准日志消息或调试信息。所有日志和调试信息都路由到 `stderr`。

---

## 第 4 步：配置环境变量

Google Ads MCP Server 通过系统环境变量读取凭据。你可以通过两种方式配置：

*   **方式 A（推荐）：** 直接在 MCP 客户端的 JSON 配置文件中传递（如 Cursor 或 Claude Desktop 设置）。这样可将凭据隔离到特定工具。
*   **方式 B（备选）：** 在 shell 配置文件（如 `~/.bashrc`、`~/.zshrc` 或 Windows 环境变量）中全局设置。

### 必需的环境变量

| 环境变量 | 描述 | 格式 |
|---|---|---|
| `GOOGLE_ADS_DEVELOPER_TOKEN` | 你的 Google Ads 开发者令牌。 | 字母数字 |
| `GOOGLE_ADS_CLIENT_ID` | 你的 Google Cloud OAuth 客户端 ID。 | `*.apps.googleusercontent.com` |
| `GOOGLE_ADS_CLIENT_SECRET` | 你的 Google Cloud OAuth 客户端密钥。 | 字母数字 |
| `GOOGLE_ADS_REFRESH_TOKEN` | 生成的 OAuth 刷新令牌。 | 字母数字 |
| `GOOGLE_ADS_LOGIN_CUSTOMER_ID` | 经理账户 ID（MCC）。使用经理层级结构时必需。 | 10 位数字，不含连字符 |
| `GOOGLE_PROJECT_ID` | Google Cloud 项目 ID（用于应用默认凭据和 Cloud Run）。 | 字母数字 |
| `GOOGLE_ADS_MCP_OAUTH_CLIENT_ID` | Cloud Run 专用 OAuth 客户端 ID，覆盖默认值。 | `*.apps.googleusercontent.com` |
| `GOOGLE_ADS_MCP_OAUTH_CLIENT_SECRET` | Cloud Run 专用 OAuth 客户端密钥，覆盖默认值。 | 字母数字 |
| `GOOGLE_ADS_MCP_BASE_URL` | 自动分配或手动配置的 Cloud Run 基础 URL。 | URL |
| `FASTMCP_HOST` | FastMCP 服务器的主机绑定（如 Cloud Run 使用 `0.0.0.0`）。 | IP 地址 |

### 方式 B：Shell 配置文件配置（全局）

如果选择方式 B，将以下内容追加到 shell 配置文件（`~/.bashrc`、`~/.zshrc` 等）：

```bash
# Google Ads MCP Server 凭据
export GOOGLE_ADS_DEVELOPER_TOKEN="your_developer_token_here"
export GOOGLE_ADS_CLIENT_ID="your_oauth_client_id_here"
export GOOGLE_ADS_CLIENT_SECRET="your_oauth_client_secret_here"
export GOOGLE_ADS_REFRESH_TOKEN="your_oauth_refresh_token_here"

# 仅经理账户层级结构需要：
export GOOGLE_ADS_LOGIN_CUSTOMER_ID="your_manager_account_id_here"

# 应用默认凭据（ADC）或 Cloud Run 部署需要：
export GOOGLE_PROJECT_ID="your_project_id_here"
```

为保护凭据安全，**不要建议将凭据硬编码到 MCP Server 的 Python 源脚本中。**

---

## 第 5 步：与 AI 开发工具集成

使用以下任一集成方式，将 Google Ads MCP Server 连接到你的 AI 编码环境。

### 1. Claude Desktop（或 Claude Code CLI）

> [!IMPORTANT]
> **Agent 强制指令（Claude Desktop 配置检查清单）：**
> 当用户询问如何配置 Claude Desktop 或 Claude Code 以使用 Google Ads MCP Server 时，你的回复**必须**系统性地包含以下全部内容：
> 1. [ ] **提供按操作系统区分的路径：** 你**必须**明确列出 `claude_desktop_config.json` 在 **macOS**（`~/Library/Application Support/Claude/claude_desktop_config.json`）和 **Windows**（`%APPDATA%\Claude\claude_desktop_config.json`）上的正确路径。即使运行在 Linux 上，也不得省略 macOS 或 Windows 路径。
> 2. [ ] **提供有效的 JSON 配置：** 提供 `claude_desktop_config.json` 的完整有效 JSON 配置块。
> 3. [ ] **指定命令和参数：** 确保 JSON 使用 `pipx` 作为命令，`run`、`google-ads-mcp` 作为参数来配置服务器。
> 4. [ ] **声明认证环境变量：** 在配置中声明环境变量 `GOOGLE_ADS_DEVELOPER_TOKEN`、`GOOGLE_ADS_CLIENT_ID`、`GOOGLE_ADS_CLIENT_SECRET` 和 `GOOGLE_ADS_REFRESH_TOKEN`。

将服务器条目添加到 Claude 配置文件。

*   **文件位置：**
    *   **macOS：** `~/Library/Application Support/Claude/claude_desktop_config.json`
    *   **Windows：** `%APPDATA%\Claude\claude_desktop_config.json`
    *   **Linux：** `~/.config/Claude/claude_desktop_config.json`

*   **配置 JSON：**

    ```json
    {
      "mcpServers": {
        "google-ads": {
          "command": "pipx",
          "args": [
            "run",
            "google-ads-mcp"
          ],
          "env": {
            "GOOGLE_ADS_DEVELOPER_TOKEN": "YOUR_DEVELOPER_TOKEN",
            "GOOGLE_ADS_CLIENT_ID": "YOUR_OAUTH_CLIENT_ID",
            "GOOGLE_ADS_CLIENT_SECRET": "YOUR_OAUTH_CLIENT_SECRET",
            "GOOGLE_ADS_REFRESH_TOKEN": "YOUR_OAUTH_REFRESH_TOKEN",
            "GOOGLE_ADS_LOGIN_CUSTOMER_ID": "YOUR_MANAGER_ACCOUNT_ID_IF_APPLICABLE"
          }
        }
      }
    }
    ```
    *（注意：推荐使用 `pipx run`，因为它会自动管理执行路径。如果使用 GitHub 开发版本或应用默认凭据，也可以配置 `"args": ["run", "--spec", "git+https://github.com/googleads/google-ads-mcp.git", "google-ads-mcp"]`，并在 `env` 块中包含 `"GOOGLE_PROJECT_ID": "YOUR_PROJECT_ID"`）。*

---

### 2. Cursor AI 编辑器

1.  打开 Cursor 并导航到：**Settings 🡒 Features 🡒 MCP**。
2.  点击 **+ New MCP Server**。
3.  配置以下字段：
    *   **Name：** `google-ads`
    *   **Type：** `stdio`
    *   **Command：** `pipx run google-ads-mcp`
4.  在 **Environment Variables** 下，添加所需的键和值：
    *   `GOOGLE_ADS_DEVELOPER_TOKEN`
    *   `GOOGLE_ADS_CLIENT_ID`
    *   `GOOGLE_ADS_CLIENT_SECRET`
    *   `GOOGLE_ADS_REFRESH_TOKEN`
    *   `GOOGLE_ADS_LOGIN_CUSTOMER_ID` *（如适用）*
5.  点击 **Save**。

---

### 3. Antigravity IDE 与 CLI 集成

回答将 Google Ads MCP Server 连接到 Antigravity（IDE 或 CLI）的问题时，你**必须**明确解释以下架构和配置细节：

*   **必需的环境设置**：指导用户在终端会话或 IDE 环境中配置并导出标准环境变量（如 `GOOGLE_ADS_DEVELOPER_TOKEN`、`GOOGLE_ADS_CLIENT_ID`、`GOOGLE_ADS_CLIENT_SECRET`、`GOOGLE_ADS_REFRESH_TOKEN`）。
*   **服务器注册**：指导用户在 Antigravity 设置中注册服务器，或使用标准 `stdio` 集成（例如配置命令 `pipx run google-ads-mcp`）。
*   **自动工具发现（核心架构）**：明确解释 Antigravity 使用 **Model Context Protocol (MCP)**，在服务器连接后自动发现其工具。
*   **无需自定义编译**：明确说明由于 Antigravity 原生支持 MCP，使用 MCP Server **不需要**单独编译自定义插件或加载自定义扩展。

#### 在 Antigravity CLI 中验证激活

1.  **配置环境**：在当前 shell 会话中导出 Google Ads API 所需的所有环境变量。
2.  **启动 Antigravity CLI**：启动 CLI：

    ```bash
    agy
    ```
3.  **验证 MCP 状态**：在 Antigravity CLI 提示符内运行 `/mcp` 命令，列出活动工具和服务器：

    ```text
    /mcp
    ```
4.  **确认激活**：确认 `google-ads-mcp` 出现在活动工具响应中。

> [!IMPORTANT]
> 如果活动工具列表中缺少 `google-ads-mcp`，退出 CLI，确认环境变量已正确设置并导出，然后重启 `agy`。

---

## 第 5.5 步：部署到 Google Cloud（Cloud Run）

除了在本地托管此 MCP Server，你还可以将其托管在 Google Cloud Run 或任何其他云基础设施上。如果希望在不同 Agent 之间共享服务器，或将其作为 Web 服务运行，这会很有用。

### 1. 前置条件

1. 一个 Google Cloud 项目。
2. 已安装并认证 [`gcloud` 命令行工具](https://cloud.google.com/cli)，且配置了活动项目：

   ```bash
   gcloud config set project YOUR_PROJECT_ID
   ```

### 2. 构建并推送 Docker 镜像
你可以使用 Cloud Build 构建镜像并推送到 Artifact Registry，无需在本地安装 Docker：

1. 在 Artifact Registry 中创建仓库：

   ```bash
   gcloud artifacts repositories create mcp-servers --repository-format=docker --location=us-central1
   ```
2. 构建并