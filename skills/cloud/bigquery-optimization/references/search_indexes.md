# 搜索索引

关于何时建议创建 BigQuery 搜索索引，以及如何编写 `CREATE SEARCH INDEX` 语句的指南。将 DDL 建议给用户，由用户自行执行；切勿自行执行。

## 目录

-   [何时建议创建搜索索引](#when-to-propose-a-search-index)（第 16-34 行）
-   [建议 DDL](#proposing-the-ddl)（第 36-71 行）
-   [验证索引](#verifying-the-index)（第 73-83 行）
-   [遥测与数据检索参考](#telemetry-data-retrieval-reference)（第 85-100 行）

## 何时建议创建搜索索引

当以下条件全部满足时，建议创建搜索索引：

*   查询反复在 `STRING` 或 `JSON` 数据中查找特定值（例如 IP 地址、ID 或错误码）。搜索在返回少量结果时效果最佳。
*   表至少 10 GB。搜索索引专为大型表设计：小于 10 GB 的表上的索引不会被填充，查询也不会使用它（`IndexUnusedReason` 代码 `BASE_TABLE_TOO_SMALL`）。
*   查找使用 `SEARCH` 函数，或可以改写为使用 `SEARCH`。由于 `SEARCH` 匹配的是词元而非子串，其结果可能与 `REGEXP_CONTAINS` 或 `LIKE '%term%'` 不同（将包含分隔符的精确词元用反引号包裹，例如 `` '`req-7f3a9c`' ``，并与用户确认）。对于 `REGEXP_CONTAINS` 或 `LIKE '%term%'` 过滤条件，建议索引时一并给出 [sql_optimization.md](sql_optimization.md) 中"在带搜索索引的列上使用 SEARCH"规则的 `SEARCH` 改写，待索引的 `coverage_percentage` 大于 0 后应用。

## 建议 DDL

仅对查询所搜索的 `STRING` 或 `JSON` 列建索引：

```sql
CREATE
  SEARCH INDEX {index_name}
ON `{project_id}.{dataset_id}.{table_id}`({column_1}, {column_2});
```

或对表中所有 `STRING` 和 `JSON` 数据建索引：

```sql
CREATE
  SEARCH INDEX {index_name}
ON `{project_id}.{dataset_id}.{table_id}`(ALL COLUMNS);
```

分析器选项（`analyzer`）：

*   `LOG_ANALYZER`（默认）适用于机器生成的日志，对 IP 地址和电子邮件等词元有相应规则。`NO_OP_ANALYZER` 精确匹配预处理后的数据。`PATTERN_ANALYZER` 使用正则表达式提取词元。使用非默认分析器时，查询必须向 `SEARCH` 传入相同的分析器（例如 `SEARCH(email, 'kim', analyzer => 'NO_OP_ANALYZER')`）。

例如：

```sql
CREATE
  SEARCH INDEX {index_name}
ON
  `{project_id}.{dataset_id}.{table_id}`(ALL COLUMNS)
  OPTIONS (analyzer = 'NO_OP_ANALYZER');
```

## 验证索引

每个搜索索引建议都必须告知用户如何验证，分两步：

1.  **覆盖率，在依赖索引之前：** 检查 `index_status` 为 `ACTIVE` 且 `coverage_percentage` 大于 0。`coverage_percentage` 为 0 意味着该索引无法用于 `SEARCH` 查询，即使部分数据已被索引。
2.  **使用情况，在查询运行之后：** 检查 `index_usage_mode` 为 `FULLY_USED` 或 `PARTIALLY_USED`。若为 `UNUSED` 或 `PARTIALLY_USED`，请阅读 `index_unused_reasons` 代码（例如 `BASE_TABLE_TOO_SMALL`）。

## 遥测与数据检索参考

如果工作区中有 `bigquery-observability` 技能，请参阅其遥测参考获取查询模板。所有决策规则和 DDL 模板均自包含于本指南中。若没有该技能，请在带数据集限定的 `INFORMATION_SCHEMA.SEARCH_INDEXES` 视图中检查 `index_status` 和 `coverage_percentage`，并在带区域限定的 `INFORMATION_SCHEMA.JOBS` 视图中检查 `search_statistics`。

*   **索引覆盖率：** 参见 `bigquery-observability` 技能（`references/storage_footprints.md`，"Search Index Coverage Query" 一节）。
*   **索引使用情况：** 参见 `bigquery-observability` 技能（`references/job_performance_queries.md`，"Search Index Usage" 一节）了解 `search_statistics.index_usage_mode` 和 `index_unused_reasons`。对于单个作业，`bq show --format=prettyjson --location={location} -j {project_id}:{job_id}` 会返回 `statistics.query.searchStatistics`。