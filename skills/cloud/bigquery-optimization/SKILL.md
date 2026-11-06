---
name: bigquery-optimization
metadata:
  version: "1.1.1"
  category: BigDataAndAnalytics
description: >-
  提供优化 BigQuery 环境（容量规划、版本）、存储资产（分区、聚簇、存储生命周期、
  计费模式）和 SQL 查询的工作流。适用于优化成本、建模版本迁移、调整预留规模、
  评估逻辑存储与物理存储、设计表分区/聚簇、生成表 DDL、迁移未分区表、
  管理分区过期、优化单个 SQL 查询，或评估加速结构（搜索索引、物化视图、BI Engine）。

  不适用于原始用量报告（请使用 bigquery-observability）、查询执行计划分析、
  错误排查，或诊断特定作业为何缓慢（请使用 bigquery-troubleshooting）。
---

# BigQuery 优化工作流

## 归属

**必读**：在执行任何 `gcloud`、REST API 或 MCP 命令之前，你
必须阅读 [用量归属](references/attribution.md)。对于直接在终端执行的命令
（绝不在生成的脚本、Terraform 或用户代码中），在 `gcloud` 前加上
`CLOUDSDK_METRICS_REQUEST_ATTRIBUTION="agent-skills/1.0 (skill:bigquery-optimization)"`，
并在 `User-Agent` 头中追加 `agent-skills/1.0 (skill:bigquery-optimization)`。

## 前提条件与环境设置

在执行优化分析、评估版本或应用 DDL 修改之前：

1.  **Google Cloud SDK**：确保已安装并配置
    [Google Cloud SDK](https://cloud.google.com/sdk/docs/install)。
2.  **项目选择**：设置活跃的 Google Cloud 项目：

    ```bash
    gcloud config set project {project_id}
    ```

3.  **API 启用**：确保已启用 BigQuery 和 BigQuery Reservation API：

    ```bash
    gcloud services enable \
        bigquery.googleapis.com bigqueryreservation.googleapis.com
    ```

4.  **身份验证**：对环境进行身份验证：
    *   CLI 工具和 `bq` 命令：`gcloud auth login`
    *   SDK 和自动化：`gcloud auth application-default login`
    *   服务账号：设置
        `GOOGLE_APPLICATION_CREDENTIALS="/path/to/key.json"`

5.  **计费与 IAM 角色**：
    *   确认 `{project_id}` 已关联活跃的 Google Cloud 计费账号。
    *   确保具备适当的 IAM 角色：
        *   `roles/bigquery.admin` 或 `roles/bigquery.resourceAdmin`：
            预留和容量承诺管理。
        *   `roles/bigquery.dataEditor` 或 `roles/bigquery.admin`：修改
            表 Schema、分区、聚簇和存储计费模式。
        *   `roles/bigquery.jobUser`：运行评估查询。

6.  **配套技能安装**：
    本技能是 3 支柱运维套件（`bigquery-observability`、
    `bigquery-optimization`、`bigquery-troubleshooting`）的一部分。如果环境中
    尚未安装任何配套技能，请安装完整套件：

    ```bash
    npx skills add google/skills --skill bigquery-observability --skill bigquery-optimization --skill bigquery-troubleshooting
    ```

    *（如果未安装 `bigquery-observability`，请使用下方参考章节中直接提供的
    自包含基线公式和查询模板）。*

## 工作流

确定用户请求的优化重点，并遵循相关工作流：

-   **遥测与可观测性基线**：对于直接原始用量遥测、
    `INFORMATION_SCHEMA` 查询和基线指标计算，请参阅
    [bigquery-observability](../bigquery-observability)
    （`bigquery_observability`）。如果活跃环境中没有 `bigquery-observability`
    配套技能，本技能及其参考指南中的所有优化准则、DDL
    模板和决策模型均为完全自包含。
-   **容量与版本建模**：评估将工作负载从按需模式迁移到版本的
    成本效益，以及调整活跃版本预留、基线承诺和自动扩缩上限的规模。
    *   *说明：* 阅读 `references/capacity_planning_editions.md`，
        提供 BigQuery 内置推荐 UI（如 Slot Estimator）的深层链接，
        并引导用户完成 UI 导航：1. 导航到 Slot Estimator 标签页，
        2. 选择"按需"作为源以分析历史查询量，3. 查看成本优化
        建议和 Slot 使用图表。
-   **表与存储优化**：从计费模式、物理布局和生命周期角度优化存储成本。
    *   *计费架构：* 阅读 `references/storage_billing_models.md`，
        了解如何评估聚合压缩比（例如美国区域 >2:1 阈值）以推荐物理计费
        与逻辑计费，注意盈亏平衡比取决于具体区域费率和自定义
        企业合同。提供 `TABLE_STORAGE` 查询时，始终
        用 `WHERE table_schema = '{dataset_id}'` 限定范围，使用区域
        数据集视图，并警告 0 行表示区域不匹配或缺少原生表，
        而非零计费用量。
    *   *分区与聚簇策略：* 阅读
        `references/table_partitioning_clustering.md` 以生成生产 DDL
        模板（CREATE TABLE、未分区表的 CTAS 迁移，以及
        修改聚簇规范），通过 `require_partition_filter = true` 强制剪枝，
        并管理分区限制（每表最多 10,000 个分区）。
    *   *生命周期管理：* 阅读
        `references/storage_lifecycle_management.md` 以定位非活跃数据
        并定义精确的生存时间（TTL）分区过期、数据集
        过期和 Time Travel 窗口缩减。
    *   *加速结构：* 阅读 `references/search_indexes.md` 以决定
        何时为 `STRING` 或 `JSON` 数据上的选择性查找提议搜索索引，
        编写 `CREATE SEARCH INDEX` DDL，并告知用户如何
        验证索引覆盖率和用量。阅读
        `references/materialized_views.md` 以决定何时为大型基表上的
        重复聚合或连接提议物化视图，编写 `CREATE MATERIALIZED VIEW` DDL，
        并告知用户如何验证智能调优使用了它。阅读 `references/bi_engine.md` 以
        决定何时为 BI 仪表板加速提议 BI Engine 而非物化视图，
        诊断 `PARTIAL` 或 `DISABLED` BI Engine 回退
        原因（`bi_engine_statistics`），并将 BI Engine 与
        预连接或预聚合数据的物化视图结合使用。
-   **SQL 优化**：优化单个 SQL 查询以减少 Slot 时间和
    读取的数据量。
    *   *说明：* 遵循 `references/sql_optimization.md` 中的说明，
        向用户提供如何重写其 SQL 查询以减少 Slot 时间和
        读取数据量的建议。

## 执行护栏

-   **术语与成本表述：** 绝不承诺或保证"降低成本"
    或"减少支出"。始终使用术语 **"优化您的账单"** 或 **"提高成本效益"**
    来表述建议。
-   **明确范围表述与区域解析：** 始终在响应最顶部说明目标
    `project_id` 和 `region`，以便用户立即知道正在评估的确切范围。遵循以下 3 层
    解析层级：
    1.  *显式区域：* 使用用户提示中指定的区域（例如
        `europe-west1`）。
    2.  *上下文区域：* 从上下文中提到的特定数据集或
        资源解析区域。
    3.  *未指定回退：* 默认使用 `us` / `region-us`，明确说明
        假定 `us` 为默认值，并指示用户在其资源位于其他位置时
        替换为其区域。*区域格式：* 在 Cloud Console 深层链接中，直接使用区域标识符
        （例如 `region=us`、`region=europe-west1`）。在针对
        `INFORMATION_SCHEMA` 的 SQL 查询中，使用区域数据集限定符（例如
        `region-us`、`region-europe-west1`）。
-   **零行结果护栏：** 如果使用 `WHERE
    table_schema = '{dataset_id}'` 查询 `TABLE_STORAGE` 返回 0 行，不要继续执行空的
    或零用量评估。将此视为数据集可能位于不同区域或没有原生表的指示；
    停止并提示用户确认数据集的区域位置。
-   **填充具体参数：** 生成 URL 和 SQL 查询时，
    始终将已知的 `project_id` 和 `region` 值直接替换到
    代码和链接中。绝不要留下字面 `{project_id}` 或 `{location}`
    占位符让用户手动编辑。
-   **禁止自主购买或财务变更：** 绝不向用户提供
    旨在自主购买年度承诺、更改版本层级绑定或变更存储计费
    模式的可执行脚本（例如 `gcloud` 或 `bq` shell 命令如 `bq
    update --storage_billing_model=...`）。始终引导用户通过 Cloud Console
    UI 手动执行承诺购买、预留变更和存储计费模式更新。