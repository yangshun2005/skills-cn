# BI Engine 加速策略

使用 BigQuery BI Engine 加速 BI 仪表盘查询的架构指南、资格约束与修复流程。

## 目录

-   [1. 何时建议使用 BI Engine 与物化视图](#1-when-to-propose-bi-engine-vs-materialized-views)
    （第 19-43 行）
-   [2. BI Engine 容量与首选表](#2-bi-engine-capacity-preferred-tables)
    （第 45-61 行）
-   [3. 限制与回退原因（`PARTIAL` 或 `DISABLED`）](#3-limitations-fallback-causes-partial-or-disabled)
    （第 63-86 行）
-   [4. 使用物化视图修复 BI Engine 回退](#4-remediating-bi-engine-fallback-with-materialized-views)
    （第 88-110 行）
-   [5. 遥测与数据检索参考](#5-telemetry-data-retrieval-reference)
    （第 112-126 行）

## 1. 何时建议使用 BI Engine 与物化视图

根据瓶颈所在，可将 BI Engine 与物化视图配合使用或独立使用：

<!-- mdformat off -->
| 维度 | BigQuery BI Engine | 物化视图（`CREATE MATERIALIZED VIEW`） | 组合使用（BI Engine + 物化视图） |
| :--- | :--- | :--- | :--- |
| **主要目标** | 加速来自 Looker、Data Studio（原 Looker Studio）或 Tableau 等工具的 BI 仪表盘查询。 | 为反复聚合大基表的查询预计算并存储聚合结果（如 `SUM` 和 `COUNT`）。 | 加速基于物化视图的仪表盘，该视图已预连接或预聚合数据，避免每次查询都执行相同的连接。 |
| **机制** | 内存分析服务，仅缓存被查询的列和分区部分。 | BigQuery 自动将基表的增量变更添加到视图；智能调优可将符合条件的基表查询改路由到该视图。 | 物化视图连接并展平（或预聚合）数据，BI Engine 加速对视图的查询。 |
<!-- mdformat on -->

**决策规则：**

*   **建议使用 BI Engine**：当用户通过 BI 工具（如 Looker、Data Studio 或
    Tableau）分析数据，或部分表被查询更频繁、或支撑高关注度仪表盘时（将这些表指定为首选表）。
*   **建议使用物化视图**：当查询反复聚合或连接大基表时。
*   **建议两者组合使用**：当仪表盘反复连接或聚合大表时：使用物化视图预连接或预聚合数据，并将该视图及其基表标记为首选表。如果仪表盘仅显示近期数据，还应按时间对表分区，以便仅将最新分区加载到内存。

## 2. BI Engine 容量与首选表

*   **容量与成本：** 为 BI Engine 容量创建的预留会产生费用。
*   **配置 BI Engine 预留与首选表：**
    *   切勿自主执行 CLI 脚本购买或变更 BI Engine 容量预留。引导用户在 Google Cloud 控制台中
        **BigQuery** > **Administration** > **BI Engine** 下配置 BI Engine
        容量或**首选表**：
        `https://console.cloud.google.com/bigquery/admin/bi-engine?project={project_id}`
    *   **首选表：** 如果部分表被查询更频繁或支撑高关注度仪表盘，将这些表指定为首选表。对其他所有表的查询使用常规 BigQuery 槽位。访问多个表的查询（如 `JOIN`）仅当查询中所有表都在首选表列表中时才会被加速；对物化视图的查询仅当物化视图及其基表都在首选表列表中时才会被加速。

## 3. 限制与回退原因（`PARTIAL` 或 `DISABLED`）

当项目配置为使用 BI Engine 时，`INFORMATION_SCHEMA.JOBS` 会在
`bi_engine_statistics.bi_engine_mode`（`FULL`、`PARTIAL` 或 `DISABLED`）和
`bi_engine_statistics.acceleration_mode`（例如，所有查询输入均被加速时为 `FULL_INPUT`，整个查询被加速时为 `FULL_QUERY`）中报告每个查询的加速状态。对于 `PARTIAL` 或 `DISABLED`，
`bi_engine_statistics.bi_engine_reasons`（`code` 和 `message`）会说明原因。
BI Engine 无法加速的查询阶段会回退到标准 BigQuery 执行槽位，且不会导致查询失败。已记录的限制包括：

*   **连接限制：** BI Engine 可加速包含 `INNER` 和 `LEFT OUTER` 连接的叶子级子查询，其中大事实表最多与四个较小的维度表连接。每个维度表必须少于 500 万行，且不超过 5 GiB（未分区表）或引用的分区不超过 1 GiB（分区表）。
*   **不支持的功能：** BI Engine 加速不适用于：
    *   引用通配符表的查询
    *   外部表，包括 BigLake 表
    *   JavaScript UDF 和远程函数
    *   行级安全
    *   使用 `SEARCH` 或 `VECTOR_SEARCH` 函数，或由搜索索引或向量索引优化的查询

## 4. 使用物化视图修复 BI Engine 回退

当诊断遥测（`bi_engine_statistics`）显示仪表盘工作负载中 `PARTIAL` 或 `DISABLED` 查询比例较高时：

1.  **检查 `bi_engine_reasons`：** 按 `bi_engine_statistics.bi_engine_mode` 和
    `UNNEST(bi_engine_statistics.bi_engine_reasons)` 对 `INFORMATION_SCHEMA.JOBS_BY_PROJECT`
    分组，以确定确切的原因代码（如 `INPUT_TOO_LARGE`、`UNSUPPORTED_SQL_TEXT`、
    `TABLE_EXCLUDED` 或 `INSUFFICIENT_RESERVATION`）。
2.  **将繁重的聚合与连接卸载到物化视图：** 如果仪表盘查询反复聚合或连接大基表：
    *   创建分区、聚簇的物化视图（参见
        [materialized_views.md](materialized_views.md)），预聚合仪表盘维度和度量（如 `SUM` 和 `COUNT`）。
    *   智能调优可将符合条件的基表查询改写为读取物化视图。如果使用首选表，请同时添加物化视图及其基表，以便对视图的查询可被加速。
3.  **裁剪扫描分区：** 如果仪表盘仅显示近期数据，按时间对基表分区，并确保仪表盘查询按分区列过滤，以便仅将最新分区加载到内存。

## 5. 遥测与数据检索参考

如果工作区中提供了 `bigquery-observability` 技能，请参阅其共享遥测参考，获取规范查询模板和 Schema 字典。所有架构规则、资格限制和修复流程均自包含在本指南中。

*   **BI Engine 加速使用情况与回退原因：** 参见
    `bigquery-observability` 技能（`references/job_performance_queries.md`，
    “BI Engine Acceleration Usage” 一节），查询
    `bi_engine_statistics.bi_engine_mode` 和
    `UNNEST(bi_engine_statistics.bi_engine_reasons)`。
*   **作业 Schema（`bi_engine_statistics`）：** 参见 `bigquery-observability`
    技能（`references/schema_compute.md`，“1. Jobs & Compute Telemetry
    Views” 一节），获取 `INFORMATION_SCHEMA.JOBS` 列定义。