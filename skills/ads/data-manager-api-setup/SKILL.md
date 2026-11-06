---
name: data-manager-api-setup
description: >-
  指导开发者完成 Data Manager API 的客户端库安装和身份验证设置步骤。当用户刚开始使用 Data Manager API，需要设置本地环境、安装客户端库或设置 API 访问权限时，使用此技能。不要用于实现受众或事件导入逻辑（请改用 data-manager-api-audience-ingestion 或 data-manager-api-event-ingestion 技能）。
metadata:
  version: "1.1.0"
  category: GoogleAds
---
# Data Manager API 设置

## 设置身份验证

详情请参阅[设置 API 访问权限](https://developers.google.com/data-manager/api/devguides/quickstart/set-up-access.md.txt)。

### 1. 启用 API

**前提条件**：一个 Google Cloud 项目、Google Cloud CLI（`gcloud`），以及一个具有可启用 API 的项目角色（如 **Owner**、**Editor** 或 **Service Usage Admin**）的 Google 账号。务必告知用户所需的角色。如果未安装 `gcloud`，请先提示用户安装。

```shell
gcloud services enable datamanager.googleapis.com --project={project_id}
gcloud services list --enabled --project=PROJECT_ID --filter="datamanager.googleapis.com"
```

### 2. 生成应用默认凭据（ADC）

选择与用户场景匹配的方法：

*   **用户账号**（以用户自己的 Google 账号发起 API 调用）：Data Manager API 的 scope 属于敏感范围，因此用户必须先配置其项目的 Google Auth Platform：
    1.  **Data Access** 页面：添加 Data Manager API 的 scope。
    2.  **Audience** 页面：将 **Publishing status** 设为 **Testing**。Google Workspace 组织使用 **Internal** 用户类型；否则使用 **External**，并将用户账号添加为 **Test user**。
    3.  创建（或复用）**Desktop app** OAuth 客户端，并下载其 JSON 配置文件。
    4.  用户运行以下命令，会打开浏览器登录：

        ```shell
        gcloud auth application-default login \
          --scopes="https://www.googleapis.com/auth/datamanager,https://www.googleapis.com/auth/cloud-platform" \
          --client-id-file="{client_id_file_path}"
        ```

    **未验证应用提示**：该警告出现是因为 Data Manager API 的 scope 属于敏感范围。在 **Testing** 阶段，External 测试用户仍会看到此提示，但可以继续。其他用户登录前，应用必须完成
    [OAuth 应用验证](https://developers.google.com/data-manager/api/devguides/concepts/oauth-verification.md.txt)。
    服务账号无需验证。
*   **服务账号**（以应用身份而非个人身份发起 API 调用）：本地开发请使用服务账号模拟，而非服务账号密钥，因为密钥若管理不当会带来安全风险。
    *   为该服务账号授予项目上的 `Service Usage Consumer` IAM 角色
        （`roles/serviceusage.serviceUsageConsumer`）。
    *   为执行 `gcloud` 的用户授予该服务账号上的 Token Creator 角色
        （`roles/iam.serviceAccountTokenCreator`）。
    *   运行：

        ```shell
        gcloud auth application-default login \
          --impersonate-service-account={service_account_email} \
          --scopes="https://www.googleapis.com/auth/datamanager,https://www.googleapis.com/auth/cloud-platform"
        ```
*   **在广告主账号中创建合作伙伴链接的数据合作伙伴**：请阅读
    [合作伙伴链接](references/partner-links.md)。

**多个 API**：如需将同一凭据用于其他 API，请将其 scope 追加到上述命令的 `--scopes` 列表中（例如，Google Ads API 使用
`https://www.googleapis.com/auth/adwords`，Display & Video 360 API 使用
`https://www.googleapis.com/auth/display-video`）。

**验证**：凭据设置正确时，`gcloud auth application-default print-access-token` 会输出一个访问令牌。

**令牌有效但出现 `PERMISSION_DENIED` 的排查**：将用户或服务账号添加到目标账号（例如 Google Ads 账号或其经理账号）。对于服务账号，还需确认其在 Cloud 项目上具有
`Service Usage Consumer` 角色。

**生产环境**：有关生产环境身份验证（Google Cloud 内或外的工作负载、代表其他用户行事的应用或合作伙伴链接），请参阅
[部署到生产环境](https://developers.google.com/data-manager/api/devguides/concepts/deploy.md.txt)。

## 安装客户端与工具库

详情请参阅[安装客户端库](https://developers.google.com/data-manager/api/devguides/quickstart/install-library.md.txt)。

配套工具库提供预构建的辅助类和函数，用于在 API 导入前正确格式化、哈希和加密用户标识符（如电子邮件、电话号码和实际地址）。强烈建议使用这些库，以确保用户标识符格式符合 API 规范。

选择以下语言对应的安装指南：

*   [Python 设置参考](references/python.md)（包：`google-ads-datamanager` 和 `google-ads-datamanager-util`）
*   [Java 设置参考](references/java.md)（包：`com.google.api-ads:data-manager` 和 `com.google.api-ads:data-manager-util`）
*   [Node 设置参考](references/node.md)（包：`@google-ads/datamanager` 和 `@google-ads/datamanager-util`）
*   [PHP 设置参考](references/php.md)（包：`googleads/data-manager` 和 `googleads/data-manager-util`）
*   [.NET 设置参考](references/dotnet.md)（包：`Google.Ads.DataManager.V1` 和 `Google.Ads.DataManager.Util`）