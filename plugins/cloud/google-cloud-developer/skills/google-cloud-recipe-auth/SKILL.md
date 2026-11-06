---
name: google-cloud-recipe-auth
metadata:
  version: "1.0.2"
  category: GettingStarted
description: 提供关于身份与访问管理（IAM）以及 Google Cloud 服务和 API 的身份验证与授权的专家指导，涵盖人类用户、服务身份、应用默认凭据（ADC）以及安全访问的最佳实践。
---

# 向 Google Cloud 进行身份验证与授权（IAM）

[身份验证](https://docs.cloud.google.com/docs/authentication.md.txt)是证明**你是谁**的过程。在 Google Cloud 中，你代表一个**主体**（Principal，即用户或服务等身份）。这是[授权](https://docs.cloud.google.com/iam/docs/overview.md.txt)（确定**你能做什么**）之前的第一步。

## 使用归因

**必读**：在执行任何 `gcloud`、REST API 或 MCP 命令之前，你必须阅读[使用归因](references/attribution.md)。对于直接在终端中执行的命令（绝不在生成的脚本、Terraform 或用户代码中），在 `gcloud` 前加上
`CLOUDSDK_METRICS_REQUEST_ATTRIBUTION="agent-skills/1.0 (skill:google-cloud-recipe-auth)"`，并在 `User-Agent` 头中追加 `agent-skills/1.0 (skill:google-cloud-recipe-auth)`。

## 身份验证

### 向 Agent 提出的澄清问题

在提供具体解决方案之前，请与用户澄清以下事项：

1.  **谁或什么在进行身份验证？**（人类开发者、本地脚本，还是生产环境中运行的应用？）
2.  **代码在哪里运行？**（本地笔记本电脑、[Compute
    Engine](https://docs.cloud.google.com/compute/docs.md.txt)、
    [GKE](https://docs.cloud.google.com/kubernetes-engine/docs.md.txt)、[Cloud
    Run](https://docs.cloud.google.com/run/docs.md.txt)，还是 AWS/Azure 等其他云？）
3.  **目标是什么？**（Storage/BigQuery 等 Google Cloud API，还是你构建的自定义应用？）
4.  **你是否在使用高层客户端库？**（例如，Python、Go、Node.js 库通常会自动处理 ADC。）

---

## 人类身份验证

用户要访问 Google Cloud，需要一个 Google Cloud 能识别的身份。

### 用户身份类型

Google Cloud 支持多种方式为内部员工（开发者、管理员、员工）配置身份：

*   **[Google 管理的账户](https://docs.cloud.google.com/iam/docs/user-identities.md.txt)**：
    你可以使用 Cloud Identity 或 Google Workspace 创建受管理的用户账户。之所以称为受管理账户，是因为你的组织控制其生命周期和配置。
*   **[使用 Cloud Identity 或 Google Workspace 进行联合身份验证](https://docs.cloud.google.com/iam/docs/user-identities.md.txt)**：
    你可以联合身份，允许用户使用其现有身份和凭据登录 Google 服务。用户向外部身份提供商（IdP）进行身份验证，但你必须使用 Google Cloud Directory Sync（GCDS）等工具或 Active Directory、Microsoft Entra ID 等外部权威来源，将账户同步到 Google Cloud。
*   **[员工身份联合](https://docs.cloud.google.com/iam/docs/user-identities.md.txt)**：
    允许你使用外部 IdP 直接通过 IAM 对员工进行身份验证和授权。与标准联合身份验证不同，你无需将用户身份从现有 IdP 同步到 Google Cloud 身份。它支持无同步、基于属性的单点登录。

### 开发人员和管理员的访问方式

用于在开发和管理过程中与 Google Cloud 资源和 API 交互。

*   **[Google Cloud 控制台](https://console.cloud.google.com/)**：主要的 Web 界面。使用你的 Google 账户（Gmail 或 [Google Workspace](https://workspace.google.com/)）进行身份验证。
*   **[gcloud CLI](https://docs.cloud.google.com/sdk/docs/install-sdk.md.txt)（`gcloud auth login`）**：用于对 CLI 本身进行身份验证，以便运行管理命令（例如 `gcloud compute instances list`）。它使用存储在本地的一个**凭据**（如 OAuth 2.0 刷新令牌）。
*   **使用[应用默认凭据（ADC）](https://docs.cloud.google.com/docs/authentication/application-default-credentials.md.txt)进行本地开发（`gcloud auth application-default login`）**：这与 CLI 身份验证不同。它会创建一个本地 JSON 文件，Google Cloud **客户端库**（Python、Java 等）在你于笔记本电脑上运行代码时用它来充当"你"。
*   **[服务账号模拟](https://docs.cloud.google.com/docs/authentication/use-service-account-impersonation.md.txt)**：
    出于安全考虑，开发者应完全避免下载服务账号密钥。相反，他们应以人类身份进行身份验证（`gcloud auth login`），并使用服务账号模拟来运行 CLI 命令或生成短期凭据。这是本地开发和故障排查的关键最佳实践。

### 面向最终用户和客户

当人类（非开发者）需要访问你部署在 Google Cloud 上的 Web 应用时使用。注意：这些与员工身份不同。

*   **[Identity-Aware Proxy（IAP）](https://docs.cloud.google.com/iap/docs.md.txt)**：
    作为 Web 应用的集中授权层。它拦截 Web 请求，在允许用户访问应用之前验证其身份（通过 Google Workspace、Cloud Identity 或外部提供商）。它常用于在没有 VPN 的情况下保护内部应用，或保护客户门户。
*   **[Identity
    Platform](https://docs.cloud.google.com/identity-platform/docs.md.txt)**：一种客户身份与访问管理（CIAM）解决方案，用于将消费者登录（邮箱/密码、电话、社交）直接添加到自定义构建应用的代码中。

---

## 服务间身份验证

当代码在生产环境中运行时，应使用**服务账号**而非人类用户账户。

### 服务账号与服务代理

*   **[服务账号](https://docs.cloud.google.com/iam/docs/service-account-overview.md.txt)**：
    一种专为非人类用户设计的特殊身份。它就像一个拥有自己电子邮件地址的"机器人身份"。
*   **[服务代理](https://docs.cloud.google.com/iam/docs/service-agents.md.txt)**：
    由 Google 管理的服务账号，允许某个服务（如 Pub/Sub）代表你访问你的资源。

### 最佳实践：附加服务账号

不要使用**服务账号密钥**（危险的 JSON 文件），而应将自定义服务账号**附加**到 Google Cloud 资源上。资源的环境随后通过本地元数据服务器提供**令牌**（一种短期数字对象）。

*   **[Compute
    Engine](https://docs.cloud.google.com/compute/docs/access/create-enable-service-accounts-for-instances.md.txt)**：
    在创建 VM 时分配服务账号。
*   **[Cloud
    Run](https://docs.cloud.google.com/run/docs/securing/service-identity.md.txt)**：
    在服务配置中分配服务账号。

### 特殊情况与高级主题

#### Kubernetes Engine（GKE）

使用 **[GKE 的工作负载身份联合](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/workload-identity.md.txt)**将 Kubernetes 身份映射到 IAM 主体标识符。这授予特定 Kubernetes 工作负载访问特定 Google Cloud API 的权限。[了解更多。](https://docs.cloud.google.com/kubernetes-engine/docs/how-to/workload-identity.md.txt)

#### 外部工作负载（[工作负载身份联合](https://docs.cloud.google.com/iam/docs/workload-identity-federation.md.txt)）

对于在 Google Cloud **之外**运行的代码（例如 AWS、Azure 或本地），不要使用密钥。而应使用工作负载身份联合，将外部令牌（如 AWS IAM 角色）交换为短期的 Google Cloud 访问令牌。

#### [API 密钥](https://docs.cloud.google.com/docs/authentication/api-keys.md.txt)

API 密钥是用于公开数据（如 Google Maps）或简化访问（如 **[Vertex AI 快速模式](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/start/express-mode/overview.md.txt)**，无需复杂设置即可快速测试 Gemini 模型）的加密字符串。人类和服务（如基于 Cloud Run 的 AI agent）均可对支持的服务使用 API 密钥。

注意：应将 API 密钥[限制](https://docs.cloud.google.com/api-keys/docs/add-restrictions-api-keys.md.txt)到特定 API 和项目，以最小化安全风险。将 API 密钥存储在 [Secret Manager](https://docs.cloud.google.com/secret-manager/docs.md.txt) 等密钥管理器中，以防意外泄露。

#### OAuth 2.0 访问范围

虽然 IAM 是现代授权方式，但旧版 Compute Engine VM 和 GKE 节点池仍依赖 IAM 之外的**访问范围**。如果 VM 的范围受限，附加的服务账号即使拥有正确的 IAM 权限，也无法发起 API 调用。如果附加的服务账号意外失败，请先检查此项。

#### 短期凭据

模拟和安全服务间通信的底层机制是 **IAM 服务账号凭据 API**。该 API 动态生成短期访问令牌、OpenID Connect（OIDC）ID 令牌或自签名 JSON Web Token（JWT），无需静态凭据。

---

## 授权

身份验证之后，Google Cloud 使用**[身份与访问管理（IAM）](https://docs.cloud.google.com/iam/docs/overview.md.txt)**来确定已通过身份验证的主体能做什么。

*   **允许策略**：将**主体**绑定到**资源**上的**角色**的记录。
*   **[预定义角色](https://docs.cloud.google.com/iam/docs/roles-permissions)**：
    预构建的角色，如 `roles/storage.objectViewer` 或 `roles/bigquery.dataEditor`。**始终优先尝试使用这些角色。**
*   **[自定义角色](https://docs.cloud.google.com/iam/docs/creating-custom-roles.md.txt)**：
    如果预定义角色过于宽泛，可使用用户定义的特定权限集合。

---

## 示例

### 人类到服务（本地 Python 开发）

1.  **身份验证**：运行 `gcloud auth application-default login` 创建本地凭据（ADC）。
2.  **授权**：为你的邮箱授予存储桶上的 `roles/storage.objectViewer` 角色。
3.  **代码**：使用 Python `storage.Client()`。它会通过 ADC 自动找到你的本地凭据。*注意：ADC 按特定顺序搜索——先检查 `GOOGLE_APPLICATION_CREDENTIALS` 环境变量，然后是本地 gcloud JSON 文件，最后是附加的服务账号元数据服务器。*

### 服务到服务（Cloud Run 到 Cloud SQL）

1.  **身份验证**：将自定义服务账号附加到你的 Cloud Run 服务。
2.  **授权**：为该服务账号授予项目上的 `roles/cloudsql.client` 角色。
3.  **代码**：Cloud Run 环境会自动向连接驱动程序提供令牌。

### 调用自定义应用（[OIDC](https://docs.cloud.google.com/docs/authentication/get-id-token.md.txt)）

当从另一个服务调用私有 Cloud Run 服务时，调用方生成一个 Google 签名的 **OpenID Connect（OIDC）ID 令牌**，并在 `Authorization: Bearer <TOKEN>` 头中传递。

---

## 验证清单

-   [ ] 用户是否在本地运行代码？建议使用 `gcloud auth application-default login` 或**服务账号模拟**。
-   [ ] 用户是否尝试在本地使用服务账号密钥？强烈劝阻，并推荐使用模拟。
-   [ ] 用户是否在生产环境中运行？推荐附加自定义的最小权限服务账号，而非使用密钥。
-   [ ] 用户是否依赖 Compute Engine 默认服务账号？建议改为创建自定义服务账号。
-   [ ] 用户是否在其他云上运行？推荐工作负载身份联合。
-   [ ] 用户是否在调用自定义应用？推荐 OIDC ID 令牌。
-   [ ] 用户是否限制了其 API 密钥？检查是否设置了适当的 [API 密钥限制](https://docs.cloud.google.com/docs/authentication/api-keys.md.txt)。

## 参考

-   [身份验证概览](https://docs.cloud.google.com/docs/authentication.md.txt)
-   [用户身份](https://docs.cloud.google.com/iam/docs/user-identities.md.txt)
-   [应用默认凭据](https://docs.cloud.google.com/docs/authentication/provide-credentials-adc.md.txt)
-   [服务账号最佳实践](https://docs.cloud.google.com/iam/docs/best-practices-service-accounts.md.txt)