# 存储生命周期管理（保留与 TTL 优化）

静态存储容量调整的决策矩阵与参数。

## 1. 优化决策矩阵

<!-- mdformat off -->
| 目标领域 | 触发条件 | 操作 / 参数 | 预期影响 |
| :--- | :--- | :--- | :--- |
| **分区 TTL** | 暂存数据集或滚动日志 | 设置 `default_partition_expiration_days` 或 `partition_expiration_days` | 窗口期后自动删除过期分区 |
| **时间旅行调优** | `PHYSICAL` 存储上的临时表 | 设置 `max_time_travel_hours = 48`（最短 2 天） | 将 `time_travel_physical_bytes` 降低约 5/7（约 71%） |
| **未分区表** | 成熟表（>90 天）且仅有活跃数据开销 | 按日期或时间戳对表分区 | 历史行可享受 50% 长期折扣 |
| **开发/测试复制** | 完整的 CTAS 暂存表副本 | 改用 `CREATE TABLE CLONE` | 仅对新增/修改的增量块计费 |
| **灾难恢复** | 时间点表备份 | 改用 `CREATE SNAPSHOT TABLE` | 仅对修改/删除的基础行计费 |
| **冷合规** | 每年访问 <1 次的数据 | 导出至 GCS Archive，通过 BigLake 查询 | 比 BigQuery 长期逻辑费率便宜约 8 倍 |
<!-- mdformat on -->

## 2. 节省公式与计算

```text
# 时间旅行缩减节省（仅物理计费）：
estimated_tt_savings = (
                         current_time_travel_gib
                         * (1 - (new_window_hours / current_window_hours))
                       ) * price_active_physical
```

## 3. 注意事项与约束

-   **DML 计时器重置：** 未分区的 `UPDATE`/`MERGE` 会触及所有分区，将其 90 天长期计时器重置为 0。务必加上 `WHERE _PARTITIONDATE >= ...` 过滤条件。
-   **故障安全不变式：** 故障安全窗口固定为 7 天，不可配置；时间旅行结束后仍会继续产生活跃物理存储费用。
-   **逻辑计费适用性：** 在 `LOGICAL` 计费下，时间旅行和故障安全字节免费；调整 `max_time_travel_hours` 在逻辑数据集上节省 \$0 存储费用。
-   **TTL 永久删除：** 通过分区过期删除的数据，一旦时间旅行 + 故障安全窗口结束便无法恢复。