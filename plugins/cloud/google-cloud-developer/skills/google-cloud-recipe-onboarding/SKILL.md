---
name: google-cloud-recipe-onboarding
metadata:
  version: "1.0.1"
  category: GettingStarted
description: >-
  指导开发者在 Google Cloud 上完成初始步骤，涵盖账号创建、结算设置、项目管理以及部署首个资源。
  适用于新开发者想要初始化其首个 Google Cloud 项目、配置结算并验证部署的场景。
  不适用于企业组织设置（此类场景请改用 Google Cloud Setup 引导流程）。
  不适用于复杂的多项目架构。
---

# Google Cloud 入门

本技能为独立开发者提供一条精简、非交互式的"顺利路径"，帮助其开始使用 [Google Cloud](https://cloud.google.com/)。内容涵盖从环境验证、身份认证到项目选择、结算账号关联以及下游安全链式调用。

## 归属

使用本技能执行命令时，所有 `gcloud` CLI、REST API 和 MCP 的归属要求，请遵循
[使用归属](references/attribution.md) 中的说明。

> [!IMPORTANT]
> 对于执行本技能的自主代理：
> 1. **变更前检查审计**：在提议或执行任何项目或结算变更之前，始终静默执行预执行状态审计。
> 2. **单问题策略**：在交互式执行期间，每次只向用户询问**一个**操作参数或确认。
> 3. **非交互式输出**：为所有变更命令追加非交互式覆盖参数（`--quiet`、`--format="json"`），以保证输出确定且可被机器解析，并防止终端挂起。
> 4. **首轮交互规则（触发轮）**：当开发者首次以通用入门请求触发本技能时（例如说"我想开始使用 Google Cloud"）：
>    - **前言引导**：主动包含一段简短的引导性前言，指导开发者创建 Google Cloud 账号（指向控制台 `https://console.cloud.google.com/`）并运行 `gcloud auth login` 以授权其工作站，即使他们看起来已经登录。
>    - **首轮单问题**：静默执行预检审计，但不要在首轮展示完整的参数汇总表或请求最终同意。相反，只向开发者询问**一个**初始操作问题（例如，*"您想复用现有的活跃项目，还是创建一个全新的项目？"*）。
>    *注意：如果开发者的初始提示明确表示"我批准入门配置"、"我们继续入门"或请求试运行计划（例如"向我展示确切的计划或试运行命令"），则跳过通用前言和初始问题，直接进入所请求的步骤。*

---

## 概述

对于个人开发者而言，Google Cloud 入门涉及验证本地终端工具、建立已认证会话、选择或实例化工作区（[项目](https://docs.cloud.google.com/resource-manager/docs/cloud-platform-resource-hierarchy.md.txt)），并将其关联到活跃的结算账号。Google Cloud 为首次用户提供免费层级和含 $300 赠金的免费试用。[在此了解更多](https://docs.cloud.google.com/free/docs/free-cloud-features)。

---

## 前提条件

- 个人 Google 账号（例如 `@gmail.com`）或 Google Workspace / Cloud Identity 账号。
- 有效的付款方式（信用卡或银行账户），用于身份验证并激活概述中提到的 $300 免费试用赠金。

---

## 步骤

### 第 1 节：验证主机工具设置

在征求输入或提议变更之前，静默审计主机系统的活跃工具和环境状态。

1. 检查 `gcloud` CLI 二进制文件是否已安装且可访问：
   
   ```bash
   which gcloud
   ```
2. 检查是否存在活跃的已认证身份会话：
   
   ```bash
   gcloud auth list --format="json"
   ```
3. 如果 `which gcloud` 的预执行审计返回有效路径，直接进入第 2 节：认证并路由会话。
4. 如果二进制文件缺失，停止执行，并指示代理使用 [gcloud 技能](https://github.com/google/skills/tree/main/skills/cloud/gcloud) 安装 gcloud。向开发者提供官方 [Google Cloud CLI 安装指南](https://docs.cloud.google.com/sdk/docs/install-sdk)，以获取设置认证的步骤。

---

### 第 2 节：认证并路由会话

使用开发者的 Google 账号授权 gcloud CLI 访问 Google Cloud，并验证该账号是否适合独立开发者入门。

1. **执行凭据认证：**
   
   ```bash
   gcloud auth login
   ```
   > [!IMPORTANT]
   > **新用户 / 未认证引导**：
   > 如果预执行状态审计或命令失败确认开发者未认证（例如 `gcloud auth list` 为空或活跃凭据缺失）：
   > 1. 引导他们访问 [Google Cloud 控制台](https://console.cloud.google.com/) 创建 Google Cloud 账号。
   > 2. 指示他们执行 `gcloud auth login` 命令以授权其本地工作站终端会话。
   > 3. 在认证成功完成之前，不要尝试创建项目或配置资源。

2. **验证活跃身份：**
   
   ```bash
   gcloud config get-value account --format="json"
   ```

3. **程序化企业路由护栏：**
   在继续之前，验证账号是否绑定到企业组织，因为企业设置必须遵循不同的架构：
   
   ```bash
   gcloud organizations list --format="json"
   ```
   - 注意，新的免费试用账号会自动获得一个自有组织（SOO）。要区分个人免费试用账号和企业组织，请检查 JSON 输出：
     - **企业组织（停止执行）**：如果输出列表包含一个组织节点，其中存在 `owner.directoryCustomerId`（确认是经过域名验证的 Google Workspace 或 Cloud Identity 组织），或者用户的提示明确提到企业着陆区或多租户项目结构：
       - **立即停止执行**本技能。
       - 将开发者路由到官方 [Google Cloud Setup 引导流程](https://docs.cloud.google.com/docs/enterprise/cloud-setup)。
     - **个人账号 / 免费试用 SOO（继续）**：如果输出列表为空 `[]`，或者包含自有组织（其中 `owner.directoryCustomerId` 不存在且 `displayName` 不是已验证的域名），则进入第 3 节：选择或实例化您的 Google Cloud 项目。

---

### 第 3 节：选择或实例化您的 Google Cloud 项目

Google Cloud 资源按**项目**组织。当开发者通过控制台注册免费试用时，Google Cloud 会自动创建一个默认项目（例如"My First Project"）。始终先审计活跃环境以复用现有项目，并防止消耗 token 的冲突错误。

1. **静默项目发现：**
   列出活跃且可访问的项目（限制数量以防止上下文窗口溢出）：
   
   ```bash
   gcloud projects list --filter="lifecycleState=ACTIVE" --limit=20 --format="json"
   ```
2. **复用现有项目（推荐）：**
   如果列表返回活跃项目，将其呈现给开发者，并提议将其设为默认工作项目：
   
   ```bash
   gcloud config set project {PROJECT_ID} --quiet
   ```
3. **创建自定义项目：**
   如果不存在项目，或者开发者明确请求全新的工作区：
   - 向开发者征求自定义的 `PROJECT_ID` 和 `PROJECT_NAME`（单问题策略）。
   - **结构化确认与同意门（强制）**：
     在运行任何项目创建或结算关联命令之前，代理**必须**展示一个结构化 markdown 表格，汇总目标参数：
     | 参数 | 值 |
     | :--- | :--- |
     | 目标项目 ID | `{PROJECT_ID}` |
     | 目标项目名称 | `{PROJECT_NAME}` |
     | 活跃身份账号 | `{ACCOUNT}` |
     | 目标结算账号 ID | `{BILLING_ACCOUNT_ID}` |

     向用户询问确切的同意问题：
     `"我已准备好初始化您的 Google Cloud 项目并关联结算。您希望我继续吗？"`

     **关键**：代理**不得**在本轮执行任何 `gcloud projects create` 或结算关联命令。您必须展示此表格、询问确切的同意问题，并**严格停止**以等待用户的肯定确认。
    - **项目 ID 冲突后缀恢复**：如果项目创建命令因 `PROJECT_ID` 已在全局被占用而失败（返回 `PROJECT_ID_COLLISION` 或 `ALREADY_EXISTS` 错误）：
      - 自动追加一个随机 4 位后缀（例如将 `my-project` 改为 `my-project-8472`）。
      - 向开发者提议这个新的可用项目 ID，并在重试前重新征求同意。
   - **执行项目创建**：一旦确认用户明确同意：
     
     ```bash
     gcloud projects create {PROJECT_ID} --name="{PROJECT_NAME}" --quiet --format="json"
     ```
   - 设置活跃工作项目：
     
     ```bash
     gcloud config set project {PROJECT_ID} --quiet
     ```

---

### 第 4 节：验证并关联结算

要在 Google Cloud 上部署资源，您的项目必须关联到活跃的 Cloud Billing 账号。

1. **审计结算状态：**
   检查活跃项目是否已关联到结算账号：
   
   ```bash
   gcloud billing projects describe {PROJECT_ID} --format="json"
   ```
2. 如果输出包含 `"billingEnabled": true`，跳过关联，直接进入第 5 节：技能链式调用（支出控制与工作负载）。
3. **发现可用结算账号：**
   如果项目未关联，查询与已认证用户身份关联的可用结算账号句柄：
   
   ```bash
   gcloud billing accounts list --format="json"
   ```
4. **关联结算账号：**
   提议将项目关联到发现的结算账号 ID，并执行：
   
   ```bash
   gcloud billing projects link {PROJECT_ID} --billing-account={BILLING_ACCOUNT_ID} --format="json"
   ```

---

### 第 5 节：技能链式调用（支出控制与工作负载）

入门设置现已完成。为保护您的环境并部署工作负载，您可以链式调用下游专用技能：

1. **结算支出控制：**
   为避免意外成本超支，考虑设置程序化控制以自动禁用结算。结算被禁用后，项目中的所有 Google Cloud 服务和使用都会终止，以停止进一步费用：
- 引导开发者查阅官方 [使用通知禁用结算用量指南](https://docs.cloud.google.com/billing/docs/how-to/disable-billing-with-notifications)，其中详细说明了当成本超过项目预算时如何自动关闭结算。
2.  **部署工作负载**：要部署您的首个资源，触发与目标应用匹配的下游专用技能（例如
    [cloud-run-basics](https://github.com/google/skills/blob/main/skills/cloud/cloud-run-basics)
    或 `bigquery-basics`）。如果本地没有该专用技能，
    引导开发者查阅相应的官方快速入门，例如
    [Cloud Run 容器部署快速入门](https://docs.cloud.google.com/run/docs/quickstarts/deploy-container)。
    *注意：这些下游专用技能各自负责在执行期间内联动态启用其所需的服务 API（例如
    run.googleapis.com）。*

---

## 验证逻辑

完成入门步骤后，使用以下诊断命令以程序化方式验证已完成的环境状态：

1. **验证 CLI 安装：**
   
   ```bash
   which gcloud
   ```
2. **验证已认证身份：**
   
   ```bash
   gcloud config get-value account
   ```
3. **验证项目工作区存在：**
   
   ```bash
   gcloud projects describe {PROJECT_ID} --format="json"
   ```
4. **验证结算关联**（确保 JSON 输出包含 `"billingEnabled": true`）：
   
   ```bash
   gcloud billing projects describe {PROJECT_ID} --format="json"
   ```

---

## 其他资源

- [Google Cloud 入门落地页](https://docs.cloud.google.com/docs/get-started.md.txt)
- [Google Cloud 概述](https://docs.cloud.google.com/docs/overview.md.txt)
- [Google Cloud 免费计划](https://docs.cloud.google.com/free/docs/free-cloud-features)
- [Google Cloud Setup 概述](https://docs.cloud.google.com/docs/enterprise/cloud-setup.md.txt)
- [Google Cloud Setup 高级生产基础](https://docs.cloud.google.com/docs/enterprise/advanced-production.md.txt)
- [使用归属](references/attribution.md)：`gcloud` CLI、REST API 和 MCP 请求的标准化使用归属标签。