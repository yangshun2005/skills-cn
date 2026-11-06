# 存储计费模式：逻辑与物理优化

评估 BigQuery 数据集在 `LOGICAL` 或 `PHYSICAL` 存储计费模式下哪种更具成本效益的指南。

## 目录

-   [1. 评估工作流](#1-evaluation-workflow)（第 17-81 行）
    -   [步骤 1：收集目标指标](#step-1-gather-target-metrics)（第 22-34 行）
    -   [步骤 2：计算成本对比](#step-2-calculate-cost-comparison)（第 36-61 行）
    -   [步骤 3：评估决策规则与权衡](#step-3-evaluate-decision-rules-trade-offs)（第 63-81 行）
-   [2. 可操作建议](#2-actionable-recommendations)（第 83-107 行）
    -   [如果当前使用逻辑计费：](#if-currently-on-logical-billing)（第 88-97 行）
    -   [如果当前使用物理计费：](#if-currently-on-physical-billing)（第 99-107 行）
-   [3. 关键陷阱](#3-critical-footguns)（第 109-129 行）

## 1. 评估工作流

当用户询问是否切换存储计费模式时，按以下步骤操作：

### 步骤 1：收集目标指标

从目标数据集所在区域的 `INFORMATION_SCHEMA.TABLE_STORAGE` 视图获取存储指标。

-   **范围：** 查询 ``
    `{project_id}`.`region-{location}`.INFORMATION_SCHEMA.TABLE_STORAGE``，
    筛选条件为 `table_schema = '{dataset_id}'`、`table_type = 'BASE TABLE'`
    且存储非零（`total_physical_bytes + fail_safe_physical_bytes > 0`），
    以隔离可计费的原生基础表。
-   **所需输入：** `active_logical_bytes`、`long_term_logical_bytes`、
    `active_physical_bytes`、`long_term_physical_bytes`、
    `time_travel_physical_bytes` 和 `fail_safe_physical_bytes`。

### 步骤 2：计算成本对比

使用区域或合同单价计算两种模式的预测月度成本：

```text
logical_cost  = (active_logical_gib * price_active_logical)
              + (long_term_logical_gib * price_long_term_logical)

physical_cost = (
                  (active_physical_gib + fail_safe_physical_gib)
                  * price_active_physical
                )
              + (long_term_physical_gib * price_long_term_physical)

forecast_total_cost_difference = logical_cost - physical_cost
```

*（注：在 `TABLE_STORAGE` 中，`active_physical_bytes` 已包含基线活跃数据和
`time_travel_physical_bytes`。加上 `fail_safe_physical_bytes` 即构成完整的
活跃物理存储占用。）*

-   **正差值（`> 0`）：** 预测物理计费总成本更低。
-   **负差值（`< 0`）：** 逻辑计费更便宜；切换到物理计费会增加成本。

### 步骤 3：评估决策规则与权衡

1.  **盈亏平衡压缩阈值：**
    -   *基线比率：* 物理费率（美国区活跃 \$0.04/GiB、长期 \$0.02/GiB）约为
        逻辑费率（活跃 \$0.02/GiB、长期 \$0.01/GiB）的 **2 倍**。在美国标准区域，
        数据压缩率必须**优于 2:1（>2x）**（`logical_volume / physical_volume > 2`）
        才能抵消开销。
    -   *区域与合同调整：* 对于非美国区域或企业合同，使用
        [BigQuery 存储定价](https://cloud.google.com/bigquery/pricing#storage-pricing)
        中的当前费率动态计算具体的 `price_physical / price_logical` 比率。
2.  **时间旅行与故障安全开销：**
    -   在**逻辑**计费下，时间旅行和故障安全字节**免费**。
    -   在**物理**计费下，时间旅行和故障安全字节**按物理活跃费率计费**。
    -   高变更工作负载（频繁 `UPDATE`/`DELETE` 或每日全表覆盖）会膨胀物理存储，
        需要更高的盈亏平衡压缩率（通常 2.5x–3x+）。

## 2. 可操作建议

确定数据集当前的存储计费模式（通过元数据或
`INFORMATION_SCHEMA.TABLE_STORAGE`）：

### 如果当前使用逻辑计费：

-   **物理计费更具成本效益（`forecast_total_cost_difference > 0` 且低变更）：**
    建议在数据集设置中将存储计费模式更新为 `PHYSICAL`。
-   **高变更但压缩率高：** 建议在切换到物理计费*之前*缩短数据集的时间旅行窗口
    （例如从 7 天降至 2 天），以最小化开销。
-   **逻辑计费更具成本效益（`forecast_total_cost_difference < 0`）：**
    建议继续使用 `LOGICAL` 计费。

### 如果当前使用物理计费：

-   **逻辑计费更具成本效益（`forecast_total_cost_difference < 0`）：**
    建议在数据集设置中将数据集切换回 `LOGICAL` 计费（确认 14 天锁定冷却期已过）。
-   **物理计费有利但被时间旅行膨胀：** 建议缩短数据集的时间旅行窗口以降低可计费
    物理活跃字节，无需更改计费模式。
-   **物理计费最优：** 建议维持 `PHYSICAL` 计费。

## 3. 关键陷阱

-   **14 天冷却期（硬锁定）：** 数据集的存储计费模式一旦更新，**14 天内不能再次更改**。
    在建议更改之前，请验证完整周期内的压缩率和历史变更情况。
-   **24 小时传播延迟：** 更新数据集的存储计费模式后，更改最多需要 **24 小时**才能生效
    并反映在存储计费计算中。
-   **时间旅行与故障安全按活跃物理费率计费：** 在物理计费下，时间旅行（1–7 天）和
    故障安全（7 天不可配置的灾难恢复）存储均按**活跃物理费率**计费。相比之下，
    逻辑计费下时间旅行和故障安全字节完全免费。对于高变更数据集，时间旅行 + 故障安全
    开销可能超过压缩节省的成本。
-   **免费额度按比例分配与范围：** 每月 10 GiB 免费额度在整个 Cloud Billing 账户中
    共享，并按秒比例分配（例如，存储 100 GiB 1 天约为 3.3 GiB-月，在免费额度范围内）。
-   **物理存储的删除陷阱：** 在逻辑存储下删除表会立即停止计费，但在物理存储下，
    活跃物理费用会继续累积 9–14 天（时间旅行 + 7 天故障安全）。