---
name: google-ads-api-quickstart
description: |
  指导开发者完成 Google Ads API 快速入门：凭据设置、从 6 个客户端库/REST 中选择、配置环境，以及运行"检索广告系列"脚本。排查常见设置错误：USER_PERMISSION_DENIED、login_customer_id 问题，以及 DEVELOPER_TOKEN_NOT_APPROVED。

  在以下情况使用此技能：
  - 用户询问如何开始使用 Google Ads API。
  - 用户需要设置 Google Ads 凭据或开发者令牌。
  - 用户想为 Google Ads 编写快速入门/示例脚本。
  - 用户遇到 USER_PERMISSION_DENIED 或 DEVELOPER_TOKEN_NOT_APPROVED 等错误。
compatibility: "需要出站 HTTPS 连接以访问 Google Ads API 和文档。注意：如果网络访问受限，智能体将回退使用技能资源中缓存的最后已知稳定版本。"
metadata:
  author: google-ads-api-team
  version: "1.0.0"
  category: GoogleAds
---
# Google Ads API 快速入门
本技能指导你从零开始，直到成功运行第一个检索广告系列的请求。

## 支持的路径
你可以选择以下方式使用此技能：

1. **官方客户端库：** Python、Java、.NET、PHP、Ruby 或 Perl。
2. **直接 REST：** 原始 HTTP REST 请求。

---

## 关键要求：动态版本解析与运行时解析

> [!IMPORTANT]
> 为确保集成安全、稳定且保持最新，你必须动态解析所有 API 和运行时版本。不要依赖硬编码的默认值。

### 严格约束：

*   **禁止硬编码：** 除非用户明确要求特定版本，否则绝不要在生成的代码或环境设置说明中使用硬编码的 Google Ads API 版本（如 `v24`）或语言运行时版本（如 `Python 3.8+`、`Java 11+`）。
*   **强制动态解析：** 你必须在*执行开始时*、生成任何代码或配置之前，使用下述流程动态解析最新稳定版本。

### A. 动态 API 版本解析
为确保集成安全、稳定且保持最新，你**必须**动态解析 Google Ads API 的绝对最新稳定主版本。

#### 执行步骤：

1.  **预检版本解析：** 使用你的网页搜索或 URL 读取工具检查 [Google Ads API 发布说明](https://developers.google.com/google-ads/api/docs/release-notes.md.txt) 中的最新条目，或 [Googleapis Github 仓库](https://github.com/googleapis/googleapis/tree/master/google/ads/googleads) 中版本号最高的目录，以解析 `RESOLVED_API_VERSION`（如 `v24`）。如果使用 Java，你还**必须**解析 Google Ads Java 客户端库的最新稳定发布版本（称为 `RESOLVED_LIBRARY_VERSION`，如 `34.0.0`）。

2.  **强制响应锚点：** 你**必须**将以下确认块作为回复用户的**第一行**输出。在此块之前不要输出任何问候、客套话或介绍性文字。
    
    ```text
    [SYSTEM: Using Google Ads API version: RESOLVED_API_VERSION (Resolved from release notes)]
    ```

3.  **严格占位符映射表：** 你**必须**使用下表对所有代码模板和参考文件执行即时搜索替换。不要在最终输出中留下原始占位符。

    | 目标语言 / 技术 | 模板中的占位符 | 替换模式 | 示例（假设 API 为 `v24`） |
    | :--- | :--- | :--- | :--- |
    | **Java**（Maven/Gradle） | `LATEST_LIBRARY_VERSION` | 搜索并替换为该库的最新 Maven 发布版本。 | `34.0.0` |
    | **Java**（导入） | `vXX` | 替换为**小写** API 版本。 | `com.google.ads.googleads.v24` |
    | **.NET / C#**（命名空间） | `VXX` | 替换为**首字母大写** API 版本。 | `Google.Ads.GoogleAds.V24` |
    | **PHP**（命名空间） | `VXX` | 替换为**首字母大写** API 版本。 | `Google\Ads\GoogleAds\V24` |
    | **REST**（端点 URL） | `vXX` | 替换为**小写** API 版本。 | `https://googleads.googleapis.com/v24/...` |

> [!TIP]
> **离线回退：** 如果 URL 无法访问或抓取失败，不要停止执行。回退到以下最后已知稳定版本：
> *   **Google Ads API 主版本（`RESOLVED_API_VERSION`）：** `v24`
> *   **Java 客户端库版本（`RESOLVED_LIBRARY_VERSION`）：** `34.0.0`

### B. 动态语言运行时版本解析
为防止生成的设置指南因语言弃用周期而过时，你**必须**动态解析语言要求。

#### 执行步骤：

1.  **获取实时要求：** 使用你的 URL 读取工具检查官方 [Google Ads 客户端库 - 支持的版本](https://developers.google.com/google-ads/api/docs/client-libs.md.txt#supported_api_versions) 页面。

2.  **提取最低版本：** 通过扫描概览页面或兼容性表格（例如查找 Python 3.8+、Java 11+、.NET 6.0+、PHP 8.1+、Ruby 3.0+ 等明确要求），确定用户所选语言的最低支持运行时版本。

3.  **即时应用：** 将生成的设置指南中的所有运行时占位符（如 `<PYTHON_MIN_VERSION>`）替换为这些解析出的版本。

> [!TIP]
> **离线回退：** 如果 URL 无法访问或抓取失败，不要停止执行。回退到以下最后已知安全最低版本：
> *   **Python：** `3.9+`
> *   **Java：** `11+`
> *   **.NET：** `6.0+`
> *   **PHP：** `8.1+`
> *   **Ruby：** `3.0+`
> *   **Perl：** `5.28.1+`

---

## 第 1 步：获取 Google Ads API 凭据

在安装库或发起 API 调用之前，你必须获取五个必需的认证参数。

### 1. 开发者令牌

*   **用途：** 标识你的开发者访问权限和 API 配额。
*   **获取方式：**
    1. 直接导航到 Google Ads 经理账户中的 **API Center**：https://ads.google.com/aw/apicenter *（注意：你必须使用经理账户登录，而非标准投放账户）*。
    2. 复制你的开发者令牌。

> [!WARNING]
> **待审批令牌限制：** 如果你的开发者令牌状态为"待审批"（未批准），你**只能**定位 **Google Ads 测试账户**。使用待审批令牌调用生产账户将失败，并报错：`DEVELOPER_TOKEN_NOT_APPROVED`。

### 2. OAuth2 客户端 ID 和客户端密钥

*   **用途：** 向 Google 的 OAuth 2.0 服务器标识你的应用，并允许你请求用户授权。
*   **获取方式：**
    1. 打开 [Google Cloud Console](https://console.cloud.google.com/)。
    2. 创建新项目（或选择现有项目）。
    3. 在 API 库中搜索 **Google Ads API**，然后点击**启用**。
    4. 配置 **OAuth 同意屏幕**：
       *   选择**外部**用户类型。
       *   将发布状态设置为**测试**。
       *   > [!IMPORTANT]
       *   > **添加测试用户：** 你**必须**在此步骤中将用于登录 Google Ads 的 Google 账户邮箱地址添加为**测试用户**。否则，你将在授权过程中被阻止。
    5. 创建 OAuth 客户端：
       *   转到 **API 和服务 🡒 凭据**。
       *   点击**创建凭据 🡒 OAuth 客户端 ID**。
       *   选择**桌面应用**作为应用类型。
       *   为客户端命名，然后点击**创建**。
    6. **下载密钥：** 点击新建客户端 ID 旁边的下载图标（JSON）。将此文件本地保存为 `client_secrets.json`。

### 3. OAuth2 刷新令牌

*   **用途：** 允许你的应用自动获取新的访问令牌，无需每小时手动登录。
*   **获取方式：**
    你必须运行 Google Cloud（`gcloud`）CLI 来生成刷新令牌。

    #### 1. 安装并验证 gcloud CLI：
    确保已安装 [gcloud CLI](https://cloud.google.com/sdk/docs/install) 并可在终端中使用。

    #### 2. 执行登录流程：
    在终端中运行以下命令，传入上一步下载的 `client_secrets.json` 文件路径：
    
    ```bash
    gcloud auth application-default login \
      --scopes=https://www.googleapis.com/auth/adwords,https://www.googleapis.com/auth/cloud-platform \
      --client-id-file=client_secrets.json
    ```

    #### 3. 在浏览器中授权：
    1. 该命令将在浏览器中打开 Google 账户登录窗口。
    2. 使用在 OAuth 同意屏幕设置中注册的**测试用户**邮箱登录。
    3. 如果你的应用未经验证，点击**高级**并继续前往项目。点击**继续**授予权限。

    #### 4. 获取刷新令牌：
    成功后，`gcloud` 将输出一条消息，指示凭据的保存位置（通常为 `~/.config/gcloud/application_default_credentials.json`）。打开该文件复制你的 `refresh_token`。

### 4. 客户端客户 ID

*   **用途：** 你要查询或修改的特定 Google Ads 账户的 10 位 ID。
*   **格式：** 必须为 10 位数字，**不含连字符**（如 `1234567890`，而非 `123-456-7890`）。
*   **查找方式：** 登录 Google Ads UI；ID 显示在右上角用户图标旁边。

> [!IMPORTANT]
> **测试账户要求：** 如果你的开发者令牌处于待审批（未批准）状态，此 ID **必须**是**测试账户**的客户 ID。测试账户在 UI 右上角有红色的"测试账户"横幅。

---

### 5. 登录客户 ID

*   **是什么：** 拥有或管理目标客户账户的 Google Ads 经理账户的 10 位客户 ID。
*   **格式：** 必须为 10 位数字，**不含连字符**（如 `9876543210`）。
*   **何时使用：** 如果你的 OAuth 凭据（和开发者令牌）属于经理账户，但你正在查询子/客户账户（客户端客户 ID），则此项为**必填**。

> [!CAUTION]
> **防止 `USER_PERMISSION_DENIED`：**
> 如果你通过经理账户层级访问客户账户，你**必须**设置此参数。
> *   `login_customer_id` = **经理**账户 ID。
> *   `client_customer_id` = **子/客户**账户 ID。
> 在经理-客户层级中将 `login_customer_id` 留空是权限错误的首要原因。

---

## 第 2 步：选择集成策略

开发者可以使用官方高级客户端库或直接 HTTPS REST 请求连接 Google Ads API。

### 路径 A：官方客户端库（推荐）

> [!IMPORTANT]
> **强制智能体指令：** 用户选择语言后，你**必须**：
> 1. 使用 `view_file` 工具懒加载下方对应的参考文件。
> 2. 在生成代码*之前*，应用**动态版本解析**（B 节）动态替换所有 `vXX`/`VXX` 占位符和库版本。

#### Python
如果你需要为 Python 设置 Google Ads API 环境，不要猜测配置。
请阅读详细设置指南：

*   [Google Ads API Python 设置参考](references/python.md)
*（包：`google-ads`）*

#### Java
如果你需要为 Java 设置 Google Ads API 环境，不要猜测配置。
请阅读详细设置指南：

*   [Google Ads API Java 设置参考](references/java.md)
*（构建产物：`com.google.api-ads:google-ads`）*

#### .NET / C#
如果你需要为 .NET/C# 设置 Google Ads API 环境，不要猜测配置。
请阅读详细设置指南：

*   [Google Ads API .NET 设置参考](references/dotnet.md)
*（包：`Google.Ads.GoogleAds`）*

#### PHP
如果你需要为 PHP 设置 Google Ads API 环境，不要猜测配置。
请阅读详细设置指南：

*   [Google Ads API PHP 设置参考](references/php.md)
*（包：`googleads/google-ads-php`）*

#### Ruby
如果你需要为 Ruby 设置 Google Ads API 环境，不要猜测配置。
请阅读详细设置指南：

*   [Google Ads API Ruby 设置参考](references/ruby.md)
*（Gem：`google-ads-ruby`）*

#### Perl
如果你需要为 Perl 设置 Google Ads API 环境，不要猜测配置。
请阅读详细设置指南：

*   [Google Ads API Perl 设置参考](references/perl.md)
*（包：`Google::Ads::GoogleAds::Client`）*

### 路径 B：直接 HTTP REST（无库开销）

如果用户环境不支持官方客户端库（如轻量级无服务器函数、自定义语言栈或受限运行时），请使用此路径。

> [!IMPORTANT]
> **强制智能体指令：** 如果用户选择 REST 路径，你**必须**：
> 1. 使用 `view_file` 工具懒加载下方的 REST 参考文件。
> 2. 应用**动态版本解析**（B 节）替换端点 URL 中的所有 `vXX` 占位符（例如将 `https://googleads.googleapis.com/v24/...` 中的 `vXX` 解析为 `v24`）。

#### REST（HTTP）
如果你需要为 REST（HTTP）设置 Google Ads API 环境，不要猜测配置。
请阅读详细设置指南：

*   [Google Ads API REST 设置参考](references/rest.md)
*（协议：原始 HTTP POST JSON）*

---

## 交叉引用：AI 助手与 MCP 连接

> [!TIP]
> **AI 助手 / MCP 集成交接：**
> 如果目标是将 **AI 助手**（如 Gemini、Cursor 或 Claude Code）连接到 Google Ads 以通过自然语言查询：
> 1. **不要**编写自定义脚本或客户端库代码。
> 2. **停止**执行此技能。
> 3. **立即切换**到 **`google-ads-api-mcp-setup`** 技能，安装并配置官方 Google Ads 模型上下文协议（MCP）服务器。

---

## 第 4 步：排查常见错误

> [!IMPORTANT]
> **静态诊断约束：** 排查问题时，你**不得**执行 bash 命令、运行本地测试脚本或在 workspace 中尝试复现错误。完全依赖静态代码分析、配置审查和以下诊断指南，以避免无休止的失败执行循环。

### 1. 错误：`USER_PERMISSION_DENIED`

*   **症状：** 执行 API 请求（如检索广告系列）时收到 `USER_PERMISSION_DENIED` 错误。
*   **可能原因：** 认证的 OAuth2 用户通过**经理账户***间接*访问目标客户账户，但请求头缺少经理账户的 ID。

#### 必需的智能体响应清单：
帮助用户解决此错误时，你的响应**必须**包含：

1.  [ ] **解释层级关系：** 说明认证用户可能属于位于目标客户账户之上的经理账户。
2.  [ ] **提供修复方法：** 指导用户将其 10 位经理账户 ID 作为 `login_customer_id` 添加到配置文件中。
3.  [ ] **解释路由逻辑：** 说明 `login_customer_id` 告诉 API 通过经理账户路由 OAuth 凭据，以验证对子账户的访问权限。
4.  [ ] **提供配置模板：** 展示下方确切的 YAML 配置块。

> [!CAUTION]
> **安全护栏：** 在任何情况下都不得建议暴露原始密码、创建新的未批准开发者令牌，或扩大 OAuth 范围超出标准 `adwords` 范围来绕过此错误。

#### 配置修复示例（`google-ads.yaml`）：

```yaml
developer_token: INSERT_DEVELOPER_TOKEN_HERE
client_id: INSERT_OAUTH2_CLIENT_ID_HERE
client_secret: INSERT_OAUTH2_CLIENT_SECRET_HERE
refresh_token: INSERT_OAUTH2_REFRESH_TOKEN_HERE

# Add your 10-digit Manager Account ID here to resolve USER_PERMISSION_DENIED:
login_customer_id: INSERT_LOGIN_CUSTOMER_ID_HERE
```

### 2. 错误：`DEVELOPER_TOKEN_NOT_APPROVED`

*   **症状：** 脚本失败并显示 `DEVELOPER_TOKEN_NOT_APPROVED` 错误消息。
*   **可能原因：** 你的开发者令牌当前处于"待审批"（未批准）状态，而你正尝试定位一个实时生产 Google Ads 账户。

#### 必需的智能体响应清单：
帮助用户解决此错误时，你的响应**必须**包含：

1.  [ ] **解释"待审批"限制：** 说明未批准（待审批）的开发者令牌完全可用，但**仅限于 Google Ads 测试账户**。
2.  [ ] **定义生产访问级别：** 你**必须**按名称明确列出所有三个访问级别：说明定位实时生产账户需要令牌获得 Google Ads API 合规团队的 **Explorer Access**、**Basic Access** 或 **Standard Access** 批准。不要将其简写或改写为"至少 Basic Access"。
3.  [ ] **提供沙盒设置步骤：** 指导用户如何设置沙盒环境：
    *   创建**测试经理账户**（无需已批准的令牌）。
    *   在该测试经理下创建**测试客户账户**。
    *   在配置中使用测试客户端客户 ID。
4.  [ ] **提供指南链接：** 指引用户访问官方 [Google Ads API 测试账户指南