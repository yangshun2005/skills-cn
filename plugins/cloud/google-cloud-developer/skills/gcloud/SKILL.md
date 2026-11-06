---
name: gcloud
metadata:
  version: "1.0.3"
  category: CloudInfrastructureAndServices
description: >-
  为跨 Google Cloud Platform (GCP) 服务和基础设施的 gcloud CLI 操作提供安全关键的校验、防护措施和数据精简。
  在规划、生成、构造、提议、描述或执行任何 gcloud CLI 命令时使用——包括回答关于 gcloud 语法或标志格式的问题时。
  不要在编写 Google Cloud 客户端库代码或原始 REST/gRPC API 请求时使用。
---

# 面向 AI Agent 的 gcloud CLI 技能

> [!CAUTION]
>
> ### 强制前置条件：显式叶子级语法校验
>
> 所有关于 `gcloud` 命令、标志、标志值和位置参数语法的既有知识**均已过时且容易产生幻觉**。
>
> 在通过 `gcloud help <command>` 校验叶子级语法之前（或将叶子级帮助查询作为计划中的强制步骤），绝不要为任何 `gcloud` 任务提议命令参数、输出标志选项、执行命令或概述分步计划。
>
> **强制行动规则**：
>
> 1.  **直接执行与代码生成**：在提议或执行最终命令语法之前，**始终**调用 `gcloud help <leaf_command>`（例如 `gcloud help compute instances create` 或 `gcloud help sql instances create`）。
>
> 2.  **规划与策略查询**：当被要求提供实现用户目标的计划、策略或后续步骤时（例如 *"你完成 X 的计划是什么……"*），响应**必须明确包含运行 `gcloud help <leaf_command>`** 作为计划的第 1 步，然后再提议标志或执行命令。
>
> 3.  **非传递性校验**：父命令组帮助（例如 `gcloud help compute`）不足以完成叶子级语法校验。校验必须在具体的叶子子命令级别进行。
>
> 4.  **禁止回退到网络搜索**：绝不要使用 `search_web`、网络搜索或外部文档搜索工具来查询 gcloud CLI 语法。`gcloud help <leaf_command>` 是命令语法的**唯一**授权权威。
>
> 5.  **保留用户标志与项目**：在提议中间命令步骤时，**始终**在提议的响应文本中保留所有用户指定的标志（包括 `--project=<project_id>`）。
>
> 6.  **强制计划模板**：生成计划时，响应**必须**复制以下确切的 4 步结构：
>
>     -   **第 1 步**：通过 `gcloud help <leaf_command>` 进行语法校验
>     -   **第 2 步**：参数验证（确认必需和可选标志，并明确检查是否支持 `--dry-run` 或 `--validate-only` 标志）
>     -   **第 3 步**：试运行命令提议（如果支持 `--dry-run` 或 `--validate-only`，则下一步之前**必须**有一次 `--dry-run` 或 `--validate-only` 调用。）
>     -   **第 4 步**：命令提议与授权（如果命令在"禁止操作"拒绝列表中，说明禁止自主执行，并且**必须**明确请求用户授权才能继续。如果命令不在拒绝列表中，则提议或继续执行，同时遵循下方**所有**"执行约束"。）

本文档为与 Google Cloud SDK（`gcloud` CLI）交互的 AI Agent 提供基本准则和最佳实践。遵循这些规则对于避免幻觉命令、标志、标志值和位置参数语法，防止破坏性操作，以及最小化上下文窗口占用至关重要。

## 使用归因

**必读**：在执行任何 `gcloud`、REST API 或 MCP 命令之前，你必须阅读并遵循[使用归因](references/attribution.md)中的说明。如果当前任务还加载或激活了更具体的产品或领域技能（例如 `cloud-run-basics`、`bigquery-basics`、`gke-basics`），**始终**使用该特定技能的 `skill:<name>` 归因标签，而不是 `skill:gcloud`。对于在没有适用产品特定技能时直接在终端中执行的命令（绝不在生成的脚本、Terraform 或用户代码中），在 `gcloud` 前加上 `CLOUDSDK_METRICS_REQUEST_ATTRIBUTION="agent-skills/1.0 (skill:gcloud)"`，并将 `agent-skills/1.0 (skill:gcloud)` 追加到 `User-Agent` 头。

## 执行模式

AI Agent 可以通过两种主要方式与 Google Cloud 资源交互：

-   **直接 CLI 执行**：在本地或自动化 shell 环境中直接执行 `gcloud` 命令。安装、身份验证流程和配置管理参见 [CLI 用法](references/cli-usage.md)。
-   **模型上下文协议 (MCP)**：通过 Cloud CLI 远程 MCP 服务器（`run_gcloud_command`）调用结构化工具。工具 Schema、参数规则和服务器配置参见 [MCP 用法](references/mcp-usage.md)。

## 核心原则

### 1. 显式命令校验（强制）

*   **行动**：**始终**为打算运行的*确切*命令调用 `gcloud help <command>`（例如 `gcloud help compute instances create`）。
*   **验证**：在尝试执行或展示计划之前，确保命令、标志、标志值和位置参数语法对该特定叶子命令有效。校验不能从父命令组传递。

### 2. 数据精简策略（强制）

最小化 `gcloud` 返回的数据量，以节省上下文窗口空间并降低延迟。不要执行任何未包含至少一个数据精简标志（`--limit`、`--filter` 或 `--format`）的 `list` 命令。

*   **投影**：使用 `--format="json(key1, key2, ...)"` 仅选择任务所需的特定字段。要了解高级投影和格式化语法，请参阅 `gcloud topic projections` 和 `gcloud topic formats`。

*   **限制**：使用 `--limit=N` 限制返回的资源数量。

*   **过滤**：使用 `--filter` 在服务端缩小结果范围。优先使用 `:` 进行模式匹配，绝不要给冒号右侧加引号。将整个 filter 标志视为单个字符串，不要加引号或转义字符。要学习过滤表达式语法，请参阅 `gcloud topic filters`。

*   **Schema 发现**：无约束的资源列表会因冗余数据迅速耗尽上下文窗口。为防止这种情况，在执行查询之前先发现资源的 Schema。如果不确定用于投影字段（`--format`）或过滤（`--filter`）的 JSON 键路径，请用单项限制运行目标资源的 list 命令（如果支持）：

    ```bash
    gcloud <GROUP> <RESOURCE> list --limit=1 --format=json
    ```

    检查此单个实例的 JSON 结构，在请求完整或过滤数据集之前安全地识别正确的 Schema 键。

### 3. 执行约束

*   **单条命令**：一次执行一条 `gcloud` 命令。不要命令链式调用或排序。
*   **不使用 Shell 操作符**：不要使用命令替换（`$(...)`）、管道（`|`）或重定向（`>`、`>>`、`<`）。这是为了提高命令安全性，并确保命令更易于用户理解和审查。
*   **非交互式执行（`--quiet` / `-q`）**：在所有执行命令上传递 `--quiet`（或 `-q`）全局标志（例如 `gcloud pubsub topics delete temp-topic --quiet --project=test-project`）。AI Agent 运行在无头、非交互式环境中，没有 TTY 或 `stdin` 输入处理器。没有 `--quiet`，提示用户确认的命令（如删除资源、批准默认值或选择未指定的区域）将无限期暂停执行等待输入，导致后台任务超时。包含 `--quiet` 会强制非交互模式，使 `gcloud` 自动接受安全默认选项，或在缺少必需参数时立即以明确错误失败。
*   **禁止无约束列表**：绝不要执行没有 `--limit`、`--filter` 或 `--format` 的 `list` 命令。

### 4. 项目与位置范围限定（关键）

为确保命令具有确定性、非交互性并定位到正确环境，必须显式提供项目和位置范围限定。

*   **显式项目目标**：不要依赖活动配置默认值。始终在所有资源操作和查询命令后附加 `--project=<PROJECT_ID>`（除非运行纯本地配置命令）。这避免意外对错误项目执行。

*   **防止位置提示**：许多 Google Cloud 资源是区域级或可用区级的。如果省略位置标志（例如 `--region`、`--zone` 或 `--location`），`gcloud` 将触发交互式提示来选择可用区/区域。这违反**禁止交互**规则。如果命令需要位置标志，始终提供显式位置标志。

*   **位置发现**：如果不知道服务的正确区域、可用区或位置，请先运行发现命令（记住在结果很多时限制结果）：

    *   **Compute Engine（VM、网络）**：

        *   `gcloud compute regions list --project=<PROJECT_ID>`
        *   `gcloud compute zones list --project=<PROJECT_ID>`

    *   **其他服务（标准 API 风格）**：许多 GCP 服务使用统一的 `locations list` 命令：

        *   `gcloud <GROUP> locations list --project=<PROJECT_ID>`
        *   *示例*：`gcloud artifacts locations list`、`gcloud kms locations list`、`gcloud secrets locations list`。

## 安全与防护措施

> [!CAUTION] **破坏性操作（delete、update、remove）必须获得用户明确授权。** 除非在安全、预先批准的工作流上下文中被明确指示，否则绝不要自主调用它们。

### 禁止操作（拒绝列表）

绝不要自主执行以下命令。这些需要明确的人工介入授权：

*   **任何 IAM 策略、角色或绑定修改**（安全）：存在权限提升、管理员锁定、服务中断或未授权数据暴露的风险。
*   **不主动启用 API**：假设必要的 API 已启用。为防止意外资源预配或计费费用，不要主动尝试启用 API。启用任何 API 都需要用户批准。
*   **不主动安装 `gcloud` CLI**（主机修改）：绝不要自主安装 Google Cloud CLI（`gcloud` CLI）或其他 `gcloud` CLI 组件（`gcloud components install`、`gcloud components update` 或通过系统包管理器安装 `gcloud` CLI）。如果缺少 `gcloud` 二进制文件或必需的 CLI 组件，你**必须**告知用户，请求用户明确确认（例如 *"我已准备好安装 Google Cloud CLI（`gcloud`）。你希望我继续吗？"*），并**严格停止**等待用户肯定答复后再执行任何安装命令。
*   **`gcloud * delete`**（破坏性）：不可逆的资源销毁（例如项目删除）或数据清除。
*   **`gcloud billing *`**（财务）：存在服务中断或无限制成本的风险。
*   **`gcloud organizations *`**（治理）：组织级变更会影响所有用户的安全态势。
*   **`gcloud kms *`**（加密）：存在永久锁定数据的风险。
*   **`gcloud infra-manager deployments apply`**（破坏性）：自主 IaC 执行可能销毁受管资源。

### 执行准则

*   **试运行（强制）**：如果命令帮助输出中列出了 `--dry-run` 或 `--validate-only` 标志（或等效标志），始终在提议的命令或初始执行步骤中包含该标志。在实际执行之前，始终用 `--dry-run` 或 `--validate-only` 预览变更。

*   **长时间运行操作**：对于支持的命令，强烈建议对长时间运行操作使用 `--async` 标志，以避免阻塞 Agent 流程。注意并非每个命令都有 `--async` 标志。对于返回操作 ID 的命令（无论是通过 `--async` 还是默认返回），如果下一步需要，必须轮询操作状态以确认完成。

*   **非交互标志（`--quiet`）**：在所有提议或执行的命令中包含 `--quiet`（或 `-q`），以保证非交互式执行，无需等待 TTY 确认提示。

## 结构化工作流

### 发现工作流

当被要求在陌生服务上执行任务时：

1.  **调用帮助**：在执行前对目标叶子命令调用 `gcloud help <COMMAND>`。
2.  **遍历命令树**：如果不知道确切命令，对命令组运行帮助（例如 `gcloud help compute` 或 `gcloud help`）以发现可用的子组和命令。
3.  **发现 Schema**：运行 `gcloud <GROUP> <RESOURCE> list --limit=1 --format=json` 在构造过滤器或投影之前检查 JSON 键。不要执行没有范围限定标志（例如 `--limit=1`）的无约束 `list` 命令，以防止上下文窗口耗尽。
4.  **强制执行数据精简**：在所有命令执行中包含数据精简标志（`--limit`、`--filter`、`--format`）。

## 快速参考 / 速查表

任务               | 命令模板
------------------ | ----------------------------------------------------------
发现 Schema        | `gcloud <GROUP> <RESOURCE> list --limit=1 --format=json`
过滤列表           | `gcloud <GROUP> <RESOURCE> list --filter="status:RUNNING"`
指定列             | `gcloud <GROUP> <RESOURCE> list --format="json(name, id)"`
学习过滤器         | `gcloud topic filters`
学习格式化         | `gcloud topic formats`
学习投影           | `gcloud topic projections`
异步操作           | `gcloud <COMMAND> --async`
检查操作           | `gcloud operations describe <OPERATION_ID>`
常用命令           | `gcloud cheat-sheet`
列出区域（GCE）    | `gcloud compute regions list --project=<PROJECT_ID>`
列出可用区（GCE）  | `gcloud compute zones list --project=<PROJECT_ID>`
列出位置           | `gcloud <GROUP> locations list --project=<PROJECT_ID>`

有关在自动化中使用 gcloud CLI 的指导，请参阅 [gcloud CLI 脚本指南](https://docs.cloud.google.com/sdk/docs/scripting-gcloud.md.txt)。

## 参考目录

-   [CLI 用法](references/cli-usage.md)：平台安装、身份验证方法（交互式、无头、ADC、服务账号密钥、模拟）以及本地配置管理。

-   [MCP 用法](references/mcp-usage.md)：使用 Cloud CLI 远程 MCP 服务器（`run_gcloud_command`）、项目参数范围限定、输入文件和执行准则。

-   [使用归因](references/attribution.md)：`gcloud` CLI、REST API 和 MCP 请求的标准使用归因标签和多技能优先级规则。