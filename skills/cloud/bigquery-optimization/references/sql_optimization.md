# BigQuery SQL 优化

通过应用一系列优化规则来优化用户的 SQL。将下列规则应用于提供的 SQL。若没有适用的规则，告知用户其查询已是最优。仅查阅本文件各小节中的规则列表。

> [!TIP] 始终包含一个 **"优化摘要"** 小节，仅列出已应用的优化。

## 目录

-   [减少 slot 时间的 SQL 优化规则](#减少-slot-时间的-sql-优化规则)（第 33-569 行）
    -   [用临时表物化 CTE](#用临时表物化-cte)（第 35-75 行）
    -   [按引用顺序排列多个临时表或变量](#按引用顺序排列多个临时表或变量)（第 77-142 行）
    -   [为半连接子查询添加 DISTINCT](#为半连接子查询添加-distinct)（第 144-190 行）
    -   [用 LIKE 替换 REGEXP_CONTAINS](#用-like-替换-regexp_contains)（第 192-210 行）
    -   [在带搜索索引的列上使用 SEARCH](#在带搜索索引的列上使用-search)（第 212-260 行）
    -   [在 WHERE 子句中用 BETWEEN 表达式代替 EXTRACT 函数](#在-where-子句中用-between-表达式代替-extract-函数)（第 262-295 行）
    -   [为 ORDER BY 子句添加 LIMIT 子句](#为-order-by-子句添加-limit-子句)（第 297-327 行）
    -   [从 CREATE TABLE 语句中移除 ORDER BY 子句](#从-create-table-语句中移除-order-by-子句)（第 329-359 行）
    -   [将日期列与 DATE 字面量比较，而非字符串转换](#将日期列与-date-字面量比较而非字符串转换)（第 361-391 行）
    -   [为 NOT IN 子查询添加空值过滤](#为-not-in-子查询添加空值过滤)（第 393-422 行）
    -   [用 LEFT JOIN 替换 WHERE IN OR IN 子句](#用-left-join-替换-where-in-or-in-子句)（第 424-460 行）
    -   [用近似聚合替换精确聚合](#用近似聚合替换精确聚合)（第 462-491 行）
    -   [连接前预聚合连接输入](#连接前预聚合连接输入)（第 493-569 行）
-   [减少数据量的 SQL 优化规则](#减少数据量的-sql-优化规则)（第 571-818 行）
    -   [用具体列清单替换 `SELECT *`](#用具体列清单替换-select-)（第 573-610 行）
    -   [检查查询对分区列的过滤条件](#检查查询对分区列的过滤条件)（第 612-818 行）
        -   [从 BigQuery 获取表 Schema](#从-bigquery-获取表-schema)（第 774-802 行）
        -   [用试运行验证重写](#用试运行验证重写)（第 804-818 行）

## 减少 slot 时间的 SQL 优化规则

### 用临时表物化 CTE

若存在被多次引用的 CTE 子句，建议将其物化为临时表：用 `CREATE TEMP TABLE AS` 语句包裹该 CTE 子句。

理由：CTE 子句在每次被查询引用时都会求值；物化为临时表后只求值一次。

以下示例中多个 CTE 被多次引用：

```sql
WITH t1 AS (SELECT a0 FROM a),
     t2 AS (SELECT x.a0 FROM t1, t1 x),
     t3 AS (SELECT x.a0 FROM t2, t2 x),
     t4 AS (SELECT x.a0 FROM t3, t3 x)
SELECT
  a0,
  COUNT(*)
FROM
  t4
GROUP BY
  1;
```

重写后的查询应如下所示：

```sql
CREATE TEMP TABLE t1 AS (SELECT a0 FROM a);
CREATE TEMP TABLE t2 AS (SELECT x.a0 FROM t1, t1 x);
CREATE TEMP TABLE t3 AS (SELECT x.a0 FROM t2, t2 x);
CREATE TEMP TABLE t4 AS (SELECT x.a0 FROM t3, t3 x);
SELECT
  a0,
  COUNT(*)
FROM
  t4
GROUP BY
  1;
```

### 按引用顺序排列多个临时表或变量

应用将 CTE 物化为临时表的规则（见"用临时表物化 CTE"小节）或声明变量的规则（见"检查查询对分区列的过滤条件"小节中的规则 3.b）时，按它们在查询中被引用的顺序排列。临时表和变量必须位于引用它们的其他临时表或变量之前。

以下示例中临时表与变量的顺序有误：

```sql
DECLARE last_order_date DEFAULT (
  SELECT
    max(date)
  FROM
    quantity_sold_per_day
);

CREATE TEMP TABLE quantity_sold_per_day AS (
  SELECT
    EXTRACT(DATE from timestamp) AS date,
    SUM(quantity) as total_quantity
  FROM
    orders
  GROUP BY 1
);

SELECT
  timestamp,
  order_id,
  quantity
FROM
  orders
WHERE
  timestamp >= last_order_date;
```

以下是修正后的临时表与变量顺序：

```sql
CREATE TEMP TABLE quantity_sold_per_day AS (
  SELECT
    EXTRACT(DATE from timestamp) AS date,
    SUM(quantity) as total_quantity
  FROM
    orders
  GROUP BY 1
);

DECLARE last_order_date DEFAULT (
  SELECT
    max(date)
  FROM
    quantity_sold_per_day
);

SELECT
  timestamp,
  order_id,
  quantity
FROM
  orders
WHERE
  timestamp >= last_order_date;
```

### 为半连接子查询添加 DISTINCT

若查询包含半连接，仅当子查询没有 GROUP BY 子句时，才将其重写为带 DISTINCT 子句。若子查询有 GROUP BY 子句，则保持原样。若子查询没有 GROUP BY 子句，则重写为带 DISTINCT 子句。

理由：半连接子查询中的 `DISTINCT` 可减少 `IN` 子句需要检查的数据量。

以下示例中半连接的子查询没有 GROUP BY 子句：

```sql
SELECT
  t1.a0
FROM
  test.a t1
WHERE
  t1.a2 IN (SELECT t2.a2 FROM test.a t2);
```

以下是重写为带 DISTINCT 子句的查询：

```sql
SELECT
  t1.a0
FROM
  test.a AS t1
WHERE
  t1.a2 IN (SELECT DISTINCT t2.a2 FROM test.a t2);
```

以下示例中半连接的子查询有 GROUP BY 子句，不应重写：

```sql
SELECT
  t1.a0
FROM
  test.a t1
WHERE
  t1.a2 IN (SELECT t2.a2, count(*) FROM test.a t2 group by 1);
```

该查询不应重写，因为子查询包含聚合。

### 用 LIKE 替换 REGEXP_CONTAINS

若查询中的 `REGEXP_CONTAINS` 函数使用通配符正则表达式，将其重写为 `LIKE` 运算符。

理由：`LIKE` 的性能优于 `REGEXP_CONTAINS`。

有效重写示例：

*   `REGEXP_CONTAINS(a, '.*abc.*')` 应重写为 `a LIKE '%abc%'`
*   `REGEXP_CONTAINS(a, '^abc.*')` 应重写为 `a LIKE 'abc%'`
*   `REGEXP_CONTAINS(a, '.*abc$')` 应重写为 `a LIKE '%abc'`
*   `REGEXP_CONTAINS(a, '.*abc.*def.*')` 应重写为 `a LIKE
    '%abc%def%'`

不应重写的示例：

*   `REGEXP_CONTAINS(a, 'abc')` 不应重写为 `a LIKE '%abc%'`，因为查询不含通配符。

### 在带搜索索引的列上使用 SEARCH

若查询用 `REGEXP_CONTAINS` 或 `LIKE '%term%'` 过滤某列，且该列被一个 `ACTIVE` 搜索索引覆盖、`coverage_percentage` 大于 0（用户确认，或 `INFORMATION_SCHEMA.SEARCH_INDEXES` 的 `ddl` 中显示该列或 `ALL COLUMNS`），建议将过滤条件重写为 `SEARCH` 函数。若不存在索引，仅在同时提出索引建议时提议重写，待覆盖率大于 0 后再应用。不要重写与字符串字面量的 `=`、`IN`、`LIKE 'prefix%'`、`STARTS_WITH` 或 `ENDS_WITH` 比较；BigQuery 已能对它们使用搜索索引。

`SEARCH` 匹配的是词元而非子串。默认 `LOG_ANALYZER` 不区分大小写，`SEARCH(a, '192.0.2.1')` 会匹配任何包含词元 `192`、`0`、`2`、`1`（顺序不限）的行；将词项用反引号包裹可精确匹配。若索引使用非默认分析器，向 `SEARCH` 传入相同分析器。由于结果可能与原过滤条件不同，提议后等待用户确认再应用；不要将其纳入其他自动重写。关于何时创建索引，见 [search_indexes.md](search_indexes.md)。

理由：搜索索引让 BigQuery 跳过不含搜索词元的基表数据，减少处理字节数和 slot 时间。匹配行占表比例越小时收益越大。

以下示例用 `LIKE` 过滤带索引的列：

```sql
SELECT
  Level,
  Source,
  Message
FROM
  my_dataset.Logs
WHERE
  Message LIKE '%94.60.64.181%';
```

以下是使用 `SEARCH` 重写后的查询：

```sql
SELECT
  Level,
  Source,
  Message
FROM
  my_dataset.Logs
WHERE
  SEARCH(Message, '`94.60.64.181`');
```

### 在 WHERE 子句中用 BETWEEN 表达式代替 EXTRACT 函数

若列为 `DATE` 类型：将 `EXTRACT(YEAR FROM date_column) = 2024` 形式的过滤表达式重写为 `date_column BETWEEN '2024-01-01' AND '2024-12-31'`。

若列为 `TIMESTAMP` 类型：将 `EXTRACT(YEAR FROM timestamp_column) = 2024` 形式的过滤表达式重写为 `timestamp_column >= '2024-01-01 00:00:00' AND timestamp_column < '2025-01-01 00:00:00'`。

理由：BigQuery 可静态求值 `BETWEEN` 表达式；无法静态求值作用于列的函数。

以下示例含日期过滤表达式：

```sql
SELECT
  t1.a0
FROM
  test.a AS t1
WHERE
  EXTRACT(YEAR FROM t1.a2) = 2024;
```

以下是重写后的日期过滤表达式：

```sql
SELECT
  t1.a0
FROM
  test.a AS t1
WHERE
  t1.a2 BETWEEN '2024-01-01' AND '2024-12-31';
```

### 为 ORDER BY 子句添加 LIMIT 子句

若查询有 `ORDER BY` 子句但没有 `LIMIT` 子句，建议添加 `LIMIT` 子句。

理由：绝大多数情况下用户不需要 `ORDER BY` 的完整结果；`LIMIT` 可提升 `ORDER BY` 的性能。

以下示例有 `ORDER BY` 子句但没有 `LIMIT` 子句：

```sql
SELECT
  t1.a0
FROM
  test.a AS t1
ORDER BY
  t1.a2;
```

以下是重写为带 `LIMIT` 子句的示例查询：

```sql
SELECT
  t1.a0
FROM
  test.a AS t1
ORDER BY
  t1.a2
LIMIT
  1000;
```

### 从 CREATE TABLE 语句中移除 ORDER BY 子句

若查询是 `CREATE TABLE` 或 `CREATE OR REPLACE TABLE` 语句且含 `ORDER BY` 子句，将其移除。

理由：写入表时 BigQuery 会按需排序数据，`ORDER BY` 是多余的。

以下示例为带 `ORDER BY` 子句的 `CREATE OR REPLACE TABLE` 语句：

```sql
CREATE OR REPLACE TABLE test.a AS (
  SELECT
    t1.a0
  FROM
    test.b AS t1
  ORDER BY t1.a2
);
```

以下是移除 `ORDER BY` 子句后的查询：

```sql
CREATE OR REPLACE TABLE test.a AS (
  SELECT
    t1.a0
  FROM
    test.b AS t1
);
```

### 将日期列与 DATE 字面量比较，而非字符串转换

将 `CAST(date_column AS STRING) = '2025-01-15'` 形式的过滤表达式重写为 `date_column = DATE('2025-01-15')`。

理由：除非列上应用了函数，否则 BigQuery 可静态求值针对该列的比较。

以下示例的日期过滤表达式将 `DATE` 列转换为字符串：

```sql
SELECT
  t1.a0
FROM
  test.a AS t1
WHERE
  CAST(DATE(date_column) AS STRING) = '2025-01-15';
```

以下是重写后的日期过滤表达式：不转换 `DATE` 列，而是将字符串字面量转换为 `DATE`：

```sql
SELECT
  t1.a0
FROM
  test.a AS t1
WHERE
  date_column = DATE('2025-01-15');
```

### 为 NOT IN 子查询添加空值过滤

重写 `NOT IN` 子句中的子查询，添加排除空值的 `WHERE` 子句。

理由：这样 BigQuery 可使用性能更好的反哈希连接，而非较慢的左外哈希连接。

以下示例的 `NOT IN` 子句中含子查询：

```sql
SELECT
  t1.a0
FROM
  test.a AS t1
WHERE
  t1.a2 NOT IN (SELECT t2.a2 FROM test.b AS t2);
```

以下是重写后的查询，在 `NOT IN` 子查询中添加 `WHERE` 子句排除空值：

```sql
SELECT
  t1.a0
FROM
  test.a AS t1
WHERE
  t1.a2 NOT IN (SELECT t2.a2 FROM test.b AS t2 WHERE t2.a2 IS NOT NULL);
```

### 用 LEFT JOIN 替换 WHERE IN OR IN 子句

若查询含 `WHERE ... IN (...) OR ... IN (...)` 子句，即形如 `columnA IN (subqueryX) OR columnB in (subqueryY)` 的表达式，将其重写为使用 `LEFT JOIN` 子句。

理由：`WHERE ... IN (...) OR ... IN (...)` 使用交叉乘积连接，开销可能远高于 `LEFT JOIN`。

以下示例的 `WHERE` 子句含两个 `IN` 子句：

```sql
SELECT
  t1.a0,
  t1.a1
FROM
  test.a AS t1
WHERE
  t1.a2 IN (SELECT t2.a2 FROM test.b AS t2)
  OR t1.a3 IN (SELECT t3.a3 FROM test.c AS t3);
```

以下是使用 `LEFT JOIN` 子句重写后的查询：

```sql
SELECT
  t1.a0,
  t1.a1
FROM
  test.a AS t1
LEFT JOIN (SELECT DISTINCT a2 FROM test.b) AS t2
  ON t1.a2 = t2.a2
LEFT JOIN (SELECT DISTINCT a3 FROM test.c) AS t3
  ON t1.a3 = t3.a3
WHERE
  t2.a2 IS NOT NULL OR t3.a3 IS NOT NULL;
```

### 用近似聚合替换精确聚合

若查询含 `COUNT(DISTINCT ...)` 表达式，建议重写为 `APPROX_COUNT_DISTINCT(...)`。

这会将精确计数变为近似计数。提议后等待用户确认再应用；不要将其纳入其他自动重写。

理由：近似聚合比精确聚合高效得多，但牺牲精度。一般经验：大规模聚合应使用近似聚合，小规模聚合应使用精确聚合。

以下示例含 `COUNT(DISTINCT ...)` 表达式：

```sql
SELECT
  COUNT(DISTINCT t1.a0)
FROM
  test.a AS t1;
```

以下是使用 `APPROX_COUNT_DISTINCT` 重写后的查询：

```sql
SELECT
  APPROX_COUNT_DISTINCT(t1.a0)
FROM
  test.a AS t1;
```

### 连接前预聚合连接输入

若 `WITH` CTE 将明细表连接到父表时未聚合，且外层查询立即按连接键对该 CTE 的行做 `GROUP BY` 聚合，则重写该 CTE：在连接父表之前先按连接键预聚合明细表，并将最大的表放在 `JOIN` 子句最前（最左）。仅当查询含执行未聚合 `JOIN` 且外层有 `GROUP BY` 的 `WITH` CTE 时应用此规则；不要应用于没有连接 CTE 的扁平查询，或已在半连接中聚合的查询（`WHERE ... IN (SELECT ... GROUP BY ...)`）。

理由：在 `JOIN` 前用 `GROUP BY` 减少数据，可避免连接阶段打乱和放大明细行（`job_stages` 中 `records_written >> records_read` 且 `shuffle_output_bytes_spilled > 0`），同时降低 slot 时间和 shuffle 内存占用。诊断笛卡尔连接爆炸时，还应检查 `JOIN` 子句是否有意外的 `CROSS JOIN`、缺失的 `ON` 条件或非选择性连接谓词（两侧键重复），并在连接前过滤掉 `NULL` 或倾斜的哑键（如 `WHERE customer_id IS NOT NULL AND customer_id != 'GUEST'`）。

以下示例在聚合前连接 `comments` 和 `users`：

```sql
WITH
  users
```