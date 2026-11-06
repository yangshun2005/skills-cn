# 表分区与聚簇策略

表设计、查询裁剪优化和存储效率的架构指南、决策矩阵与 DDL 模式。

## 目录

-   [1. 决策矩阵：分区 vs. 聚簇 vs. 混合](#1-决策矩阵分区-vs-聚簇-vs-混合)（第 22-32 行）
-   [2. 分区设计原则](#2-分区设计原则)（第 34-66 行）
    -   [按列类型划分的分区类型](#按列类型划分的分区类型)（第 36-46 行）
    -   [核心分区不变量与规则](#核心分区不变量与规则)（第 48-66 行）
-   [3. 聚簇设计原则](#3-聚簇设计原则)（第 68-88 行）
    -   [聚簇规则与机制](#聚簇规则与机制)（第 70-88 行）
-   [4. 生产环境 DDL 模板](#4-生产环境-ddl-模板)（第 90-178 行）
    -   [推荐的混合模式（分区 + 聚簇）](#推荐的混合模式分区--聚簇)（第 92-112 行）
    -   [将现有未分区表迁移为分区表](#将现有未分区表迁移为分区表)（第 114-135 行）
    -   [修改聚簇规范](#修改聚簇规范)（第 137-178 行）
-   [5. 表设计与推荐工作流（给定表或 Schema）](#5-表设计与推荐工作流给定表或-schema)（第 180-235 行）
-   [6. 表设计反模式与架构约束](#6-表设计反模式与架构约束)（第 237-263 行）
-   [7. 遥测与数据检索参考](#7-遥测与数据检索参考)（第 265-287 行）

## 1. 决策矩阵：分区 vs. 聚簇 vs. 混合

<!-- mdformat off -->
| 维度 | 分区（`PARTITION BY`） | 聚簇（`CLUSTER BY`） | 混合（`PARTITION BY` + `CLUSTER BY`） |
| :--- | :--- | :--- | :--- |
| **适用场景** | 低基数取值、时间/日期边界、基于日期的 TTL 保留。 | 高基数取值、多个过滤/分组列、频繁 DML 修改。 | 带多维过滤的大型时间表（如日期 + 租户/状态）。 |
| **裁剪机制** | **分区裁剪：** 在执行前将整个分区从扫描中排除。 | **块裁剪：** 执行期间动态跳过文件内的 Capacitor 数据块。 | 两级裁剪：分区裁剪缩小时间窗口，块裁剪再跳过分区内的块。 |
| **成本估算** | 可通过 dry-run（`totalBytesProcessed`）在执行前确定。 | 在执行期间评估块时动态估算。 | 前期确定分区扫描边界，执行时节省块扫描。 |
| **存储生命周期** | 解锁**分区级 TTL** 和 90 天长期 50% 存储价格折扣。 | 无分区级过期（数据跨存储块共置）。 | 支持分区 TTL 保留，同时优化分区内查询性能。 |
| **限制** | 每表最多 **10,000 个分区**。 | 每表最多 **4 个聚簇列**。 | 适用 10,000 分区限制；聚簇在每个分区内生效。 |
<!-- mdformat on -->

## 2. 分区设计原则

### 按列类型划分的分区类型

*   **`TIMESTAMP` 列：** `PARTITION BY TIMESTAMP_TRUNC(column_name, DAY)`
*   **`DATE` 列：** `PARTITION BY date_column`（或按月粒度：
    `PARTITION BY DATE_TRUNC(date_column, MONTH)`）
*   **`DATETIME` 列：** `PARTITION BY DATETIME_TRUNC(datetime_column, DAY)`
*   **摄入时间分区：** BigQuery 使用伪列 `_PARTITIONDATE` /
    `_PARTITIONTIME` 按摄入时间将行分配到分区。
*   **整数范围分区：** 使用
    `RANGE_BUCKET(column, GENERATE_ARRAY(start, end, interval))` 按 `INT64` 列分区。

### 核心分区不变量与规则

*   **10,000 分区配额限制：** 每个分区表最多可有
    10,000 个分区。若表达到或接近此限制：
    *   *解决方案：* 放宽分区时间单位（如从 `HOUR` 改为
        `DAY`，或从 `DAY` 改为 `MONTH`），并**按细粒度时间列聚簇**
        （如按月分区、按时间戳聚簇）。BigQuery 会自动
        在每个分区内聚簇数据块。
*   **强制分区裁剪（`require_partition_filter = true`）：** 对于大型
    表，始终设置 `require_partition_filter = true`，强制查询在 `WHERE` 子句中
    提供分区谓词，防止意外的
    数 TB 级未裁剪全表扫描。
*   **避免过度分区：** 不要为低数据量表创建小时级分区。分区导致每个分区数据量很小
    （约小于 10 GB）会增加表的元数据，并可能
    影响查询表时的元数据访问时间。
*   **长期存储折扣：** 连续 90 天未修改的分区
    自动享受 **50% 更低的长期存储** 价格。

## 3. 聚簇设计原则

### 聚簇规则与机制

*   **最多 4 个高基数过滤与分组列：** 在 `CLUSTER BY` 子句中最多指定 4 个
    高基数过滤和分组列（`WHERE` 过滤和 `GROUP
    BY` 键）。
*   **列顺序至关重要：** BigQuery 根据指定的列顺序
    分层聚簇数据：
    *   将**最常过滤的列**或带**等值过滤**的列
        放在首位（`col1`）。
    *   将次要过滤列、范围过滤列或 `GROUP
        BY`/`JOIN` 键放在后面（`col2`、`col3`、`col4`）。
    *   *陷阱：* 在 `col1` 无谓词的情况下过滤 `col2` 会显著
        削弱块裁剪效果。
*   **支持的数据类型：** `STRING`（前 1,024 个字符）、`INT64`、
    `NUMERIC`、`BIGNUMERIC`、`BOOL`、`TIMESTAMP`、`DATE`、`DATETIME`、
    `GEOGRAPHY`、`RANGE`。
*   **自动重新聚簇：** BigQuery 执行持续、全托管的
    后台重新聚簇，在追加或修改新数据时保持块有序性，
    **零 slot 成本**。

## 4. 生产环境 DDL 模板

### 推荐的混合模式（分区 + 聚簇）

```sql
CREATE OR REPLACE TABLE `{project_id}.{dataset_id}.events`
(
  event_id STRING,
  customer_id STRING,
  event_type STRING,
  event_timestamp TIMESTAMP,
  payload JSON
)
-- Partition by DATE, TIMESTAMP, DATETIME, or INT64 range column:
PARTITION BY TIMESTAMP_TRUNC(event_timestamp, DAY)
-- Cluster on up to 4 high-cardinality filtering and grouping columns:
CLUSTER BY customer_id, event_type
OPTIONS (
  require_partition_filter = true,
  partition_expiration_days = 365,
  description = 'High-volume events partitioned by day and clustered by customer'
);
```

### 将现有未分区表迁移为分区表

无法通过 `ALTER TABLE` 直接将现有未分区表转换为分区表，
也无法使用 `OR REPLACE` 修饰符
将表替换为不同分区类型。应改用 `CREATE
TABLE ... PARTITION BY ... CLUSTER BY ... AS SELECT * FROM ...`（CTAS）语句，
通过查询现有表中的数据来创建新的分区表（或
先 `DROP` 表再重建）——按 `DATE`、
`TIMESTAMP`、`DATETIME` 或 `INT64`（`RANGE_BUCKET`）列分区，按最多
4 个高基数过滤和分组列（`WHERE` 和 `GROUP BY`）聚簇，并
设置 `require_partition_filter = true`：

```sql
CREATE TABLE `{project_id}.{dataset_id}.events_partitioned`
  -- Partition by DATE, TIMESTAMP, DATETIME, or INT64 range column:
  PARTITION BY TIMESTAMP_TRUNC(event_timestamp, DAY)
  -- Cluster on up to 4 high-cardinality filtering and grouping columns:
  CLUSTER BY customer_id, event_type
  OPTIONS (require_partition_filter = TRUE)
AS
SELECT * FROM `{project_id}.{dataset_id}.events_unpartitioned`;
```

### 修改聚簇规范

分两步将新的聚簇规范应用于未分区或分区表。
首先，更新表的聚簇规范以匹配新的聚簇：

```bash
bq update --clustering_fields=customer_id,status \
  {project_id}:{dataset_id}.{table_id}
```

`--clustering_fields` 值为用于聚簇的逗号分隔列名列表。

然后，按新的聚簇规范对所有行进行聚簇，运行
以下 `UPDATE` 语句。向用户建议此语句，而非
直接运行：

```sql
UPDATE `{project_id}.{dataset_id}.{table_id}`
SET customer_id = customer_id
WHERE true;
```

若表设置了 `require_partition_filter = true`，`UPDATE` 必须提供分区过滤条件，
且单个作业不能修改超过
4,000 个分区（每作业修改限制）。对于分区表，
按有界批次重新聚簇：

```sql
UPDATE `{project_id}.{dataset_id}.{table_id}`
SET customer_id = customer_id
WHERE {partition_column} >= '{range_start}'
  AND {partition_column} < '{range_end}';
```

在少于 4,000 个分区的连续范围内重复，直到覆盖
整个表。

告知用户：对处于长期存储的表应用新的聚簇规范会使该表
恢复为
[活跃存储定价](https://cloud.google.com/bigquery/pricing#storage-pricing)。

## 5. 表设计与推荐工作流（给定表或 Schema）

当被要求为特定表或 Schema 推荐分区和聚簇策略时：

1.  **步骤 1：确定分区键与时间粒度**

    *   *审计现有表配置：* 在提出更改前，检查目标数据集的
        `INFORMATION_SCHEMA.COLUMNS`（`is_partitioning_column = 'YES'`、
        `data_type`、`clustering_ordinal_position`）和
        `INFORMATION_SCHEMA.TABLE_OPTIONS`（`option_name =
        'require_partition_filter'`），或运行 `bq show
        --format=prettyjson {project_id}:{dataset_id}.{table_id}`
        （`timePartitioning`、`rangePartitioning`、`clustering`、
        `requirePartitionFilter`）以确认表是否已有
        `DATE`、`TIMESTAMP`、`DATETIME` 或 `INT64` 整数范围分区、
        聚簇列和分区过滤强制。
    *   *扫描列类型：* 查找代表主要事件生成时间的 `TIMESTAMP`、`DATE` 或 `DATETIME` 列。
    *   *检查 10,000 分区配额限制：*
        *   **`DAY` 分区（默认）：** 最适合 1–7 年生命周期
            （约 365 至约 2,555 个分区）。
        *   **`HOUR` 分区：** 仅对数据量大且时间跨度短的
            表选用——通常时间戳值少于六个月。确保
            产生的分区数保持在 10,000 分区限制内。
        *   **`MONTH` 或 `YEAR` 分区：** 对于历史归档（>10
            年），按月分区并按时间戳聚簇。
    *   *无时间列：* 评估对增量 ID 使用整数范围分区
        （`RANGE_BUCKET`），或对仅追加日志流使用摄入时间分区
        （`_PARTITIONTIME`）。

2.  **步骤 2：选择并排序聚簇列（最多 4 列）**

    *   *候选列：* 频繁出现在 `WHERE` 等值过滤、`JOIN ON` 子句或 `GROUP BY`
        聚合中的高基数维度列（如 `customer_id`、`organization_id`、`status`）。
    *   *列排序规则：*
        *   `col1`：最常过滤的列或等值/租户
            标识符（`customer_id`）。
        *   `col2`、`col3`：次要维度或范围过滤属性
            （`event_type`、`created_at`）。
        *   *顺序陷阱：* 在 `col1` 无谓词的情况下过滤 `col2` 的查询
            无法执行高效的块裁剪。

3.  **步骤 3：输出生产 DDL 与防护措施**

    *   生成完整的 `CREATE TABLE` DDL，包括 CTAS 迁移。要
        修改表的聚簇规范，使用第 4 节中的两步序列。
    *   为高数据量生产表添加 `require_partition_filter = true`（`ALTER TABLE
        {dataset_id}.{table_id} SET OPTIONS (require_partition_filter = true)`）
        以防止意外的未裁剪全表扫描。
    *   为暂存、ETL 或滚动日志表添加 `partition_expiration_days = N`
        以自动化数据清理。

## 6. 表设计反模式与架构约束

-   **`ALTER TABLE` 分区不变量：** BigQuery **不支持
    将现有未分区表改为分区表**。必须使用 CTAS 或加载作业创建新的分区表。
    `ALTER TABLE` 不支持设置或更改表的聚簇；使用 `bq update
    --clustering_fields`（第 4 节）。
-   **小分区反模式（每分区 <10 GB）：** 当分区会导致**每分区小于约 10 GB** 时，
    优先使用聚簇而非分区。创建大量小分区会膨胀表元数据，
    并在查询时拖慢元数据访问，且不会带来扫描节省。
    *（另外，按需查询计算费用对每个被引用表和每个查询强制执行 10 MiB 数据处理量下限，
    无论表或分区大小）。*
-   **1,024 字符字符串限制：** 对于 `STRING` 聚簇列，BigQuery
    仅评估**前 1,024 个字符**用于块裁剪。不要
    对长自由文本描述或原始 JSON 载荷聚簇；提取简洁
    标识符。
-   **流式缓冲区压缩（`__UNPARTITIONED__`）：** 最近通过 BigQuery Storage Write API
    流式写入的行最初位于 `__UNPARTITIONED__` 分区，`_PARTITIONTIME` 为 `NULL`。
    在 BigQuery 后台压缩将其提取到永久分区之前，
    这些行无法被分区过滤裁剪。
-   **查询过滤优化：** 要在查询执行中有效利用分区和聚簇裁剪，
    谓词必须使用编译时常量。有关查询级 `WHERE` 子句重写和函数反模式，
    请参阅 `bigquery-optimization` 技能。

## 7. 遥测与数据检索参考

若工作区中有 `bigquery-observability` 技能，请参阅
其共享遥测参考以获取规范查询模板和 Schema
字典。所有必要的表设计工作流、DDL 模板和
修复手段均包含在本指南中。

*   **表分区、聚簇与过滤强制审计：** 参阅
    `bigquery-observability` 技能（`references/storage_footprints.md`，章节
    "Table Partitioning, Clustering & Filter Enforcement Audit"）以查询
    `INFORMATION_SCHEMA.COLUMNS` 和 `INFORMATION_SCHEMA.TABLE_OPTIONS`
    获取未分区表、分区列类型、聚簇列和
    `require_partition_filter`。
*   **高扫描被引用表发现：** 参阅 `bigquery-observability`
    技能（`references/compute_ondemand_billable.md`，章节 "CORE RULE: The
    Golden Base CTE (bytes_billed_cte)"）以按
    `query_info.query_hashes.normalized_literals` 对
    `INFORMATION_SCHEMA.JOBS_BY_PROJECT` 分组，获取精确的作业级
    `total_bytes_billed` 和 `total_bytes_processed` 归因，并展开
    `referenced_tables` 以发现候选表（不要跨多表连接汇总作业级字节数；
    在推荐分区前，通过 `INFORMATION_SCHEMA.TABLE_STORAGE` 中的
    `total_logical_bytes` 交叉核对候选表大小）。