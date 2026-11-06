# 物化视图

关于何时建议使用 BigQuery 物化视图，以及如何编写 `CREATE MATERIALIZED VIEW` 语句的指导。将 DDL 提供给用户，由用户自行执行；切勿自行运行。

## 目录

-   [何时建议使用物化视图](#何时建议使用物化视图)（第 18-24 行）
-   [智能调优要求](#智能调优要求)（第 26-49 行）
-   [分区对齐、聚簇与提供 DDL](#分区对齐聚簇与提供-ddl)（第 51-87 行）
-   [验证视图](#验证视图)（第 89-97 行）
-   [遥测与数据检索参考](#遥测与数据检索参考)（第 99-111 行）

## 何时建议使用物化视图

当查询反复对大型基表执行相同的聚合、过滤或连接时（例如，仪表盘查询相同的汇总指标），建议使用物化视图。物化视图会预计算并存储聚合结果，并自动将其与基表中的增量变更合并。

## 智能调优要求

BigQuery 会自动将对基表的查询重写为读取物化视图（智能调优），因此用户无需修改查询。仅当视图满足以下条件时，查询才会被重写：

*   与某个基表属于同一项目，或与查询运行所在的项目相同。
*   使用与查询相同的基表集合。
*   包含查询读取的所有列。例如，查询读取了视图未包含的列，则不会被重写。如果视图输出 `CAST(sold_datetime AS DATE)`，则对 `sold_datetime` 本身进行过滤的查询也不会被重写。
*   包含查询读取的所有行。例如，查询读取的日期超出视图的 `WHERE` 范围，或对视图过滤但未输出的列添加了更严格的过滤条件，则不会被重写。

以下物化视图不支持智能调优：

*   引用逻辑视图。
*   使用 `UNION ALL` 或 `LEFT OUTER JOIN`。
*   非增量（`allow_non_incremental_definition = true`）。用户必须直接查询这些视图。
*   引用启用了变更数据捕获（CDC）的表。

## 分区对齐、聚簇与提供 DDL

视图定义应基于对基表的查询模式，使其服务于广泛的查询，而非单一查询。例如，如果用户经常按 `user_id` 或 `department` 过滤，则应按这些列分组（并可选地聚簇），而不是在视图中添加 `user_id = 123` 之类的过滤条件。对于日期过滤，省略狭窄的日期过滤，或使用覆盖查询所读取全部范围的日期范围。

关于分区和聚簇：

*   **分区对齐：** 如果基表已分区，则按相同的分区列对视图分区，以降低刷新和查询成本。对于基于时间的分区，粒度必须与基表匹配，且应用于分区列的截断函数的粒度必须至少与基表分区粒度相同。对于整数范围分区，范围必须完全匹配。基于未分区基表的视图无法分区。
*   **聚簇：** 按查询过滤的输出列进行聚簇。聚合输出列不能作为聚簇列。

对于按 `DATE(event_ts)` 分区的基表：

```sql
CREATE MATERIALIZED VIEW `{project_id}.{dataset_id}.{view_name}`
  PARTITION BY DATE(event_day)
  CLUSTER BY service_name
AS (
  SELECT
    TIMESTAMP_TRUNC(event_ts, DAY) AS event_day,
    service_name,
    COUNT(*) AS request_count,
    SUM(bytes_sent) AS total_bytes_sent
  FROM `{project_id}.{dataset_id}.{table_id}`
  GROUP BY event_day, service_name
);
```

## 验证视图

检查视图是否出现在 `materialized_view_statistics` 中且 `chosen` 为 `TRUE`，或查询计划是否包含 `READ {view_name}` 步骤。如果 `chosen` 为 `FALSE`，请查看 `rejected_reason`（例如，`NO_DATA` 表示视图尚未刷新，`COST` 表示估计其他来源成本更低）。如果视图完全未列出，则查询形态与视图不匹配；请对照[智能调优要求](#智能调优要求)进行检查。如果考虑了大量视图，列表可能不完整。

## 遥测与数据检索参考

如果工作区中有 `bigquery-observability` 技能，请参阅其遥测参考中的查询模板。所有决策规则和 DDL 模板均包含在本指南中。若没有该技能，请检查区域限定的 `INFORMATION_SCHEMA.JOBS` 视图中的 `materialized_view_statistics`。

*   **重写使用情况：** 参见 `bigquery-observability` 技能（`references/job_performance_queries.md` 中“Materialized View Rewrite Usage”一节）。对于单个作业，`bq show --format=prettyjson --location={location} -j {project_id}:{job_id}` 会返回 `statistics.query.materializedViewStatistics`。