# 计算容量规划（版本迁移与合理配置）

面向组织管理员和账单负责人，指导评估按需计费到版本（Edition）的迁移，以及合理配置活跃的版本预留和承诺。

## 目录

-   [框架与护栏](#framing-guardrails)（第 17-35 行）
-   [1. 统一 Slot 估算器引擎](#1-unified-slot-estimator-engine)（第 37-55 行）
-   [2. 按需计费到版本迁移](#2-on-demand-to-editions-migration)（第 57-94 行）
    -   [Slot 估算器迁移深层链接](#slot-estimator-migration-deep-links)（第 63-77 行）
    -   [导航与工作负载分析](#navigation-workload-profiling)（第 79-94 行）
-   [3. 现有版本容量合理配置](#3-existing-edition-capacity-rightsizing)（第 96-138 行）
    -   [Slot 估算器合理配置深层链接](#slot-estimator-rightsizing-deep-links)（第 102-114 行）
    -   [版本级合理配置决策手册](#edition-level-rightsizing-decision-playbook)（第 116-138 行）

## 框架与护栏

讨论成本或容量优化时，遵循以下核心准则：

-   **术语与框架：** 绝不承诺或保证"降低成本"。始终使用术语 **"优化您的账单"** 或 **"提升成本效益"**。
-   **不做手动 ROI 计算：** 不要尝试用人工数学公式或电子表格式 ROI 计算来估算版本成本。BigQuery 内置的 **Slot 估算器** UI 能以细粒度精度模拟历史工作负载。始终引导用户使用 UI 深层链接和诊断查询。
-   **直接深层链接与参数格式：** 始终将具体的 `project_id` 和 `region` 参数直接填入 Cloud Console 深层链接。在 Cloud Console URL 中，使用裸区域标识符（例如 `region=us`、`region=europe-west1`），省略任何 `region-` 前缀。
-   **不自主购买：** 引导用户在 Cloud Console 中手动执行承诺和预留变更。绝不输出可执行的购买脚本（`gcloud`、`bq`）。

## 1. 统一 Slot 估算器引擎

Slot 估算器的底层数学机制在按需计费迁移和现有版本合理配置中**完全相同**：

-   **30 天工作负载摄取：** 引擎分析最多 30 天的历史工作负载遥测数据。对于按需计费，它将计费和处理的字节数建模为等效 Slot 需求；对于版本，它读取每个预留的历史 Slot 利用率。
-   **性能连续性：** 推荐器假设**无性能降级**。它将组合推荐容量（基线承诺 + 自动扩缩上限）设置为覆盖 30 天期间观测到的 **P99 Slot 需求**。
-   **自动扩缩器模拟：** 引擎在每个离散时间单元上模拟自动扩缩器，测试基线上限和自动扩缩上限之和等于该 P99 用量的所有组合。
-   **成本优化：** 在所有满足 P99 性能目标的可行情景中，UI 筛选并呈现单一最低成本的承诺和自动扩缩配置。

## 2. 按需计费到版本迁移

当项目当前使用按需计费，并希望评估迁移到版本（Standard、Enterprise、Enterprise Plus）是否能提升成本效益时使用。

### Slot 估算器迁移深层链接

提供带 `source=EDITION_UNSPECIFIED` 的直接深层链接：

-   **项目级按需计费推荐：**

    ```text
    https://console.cloud.google.com/bigquery/admin/reservations;region={region}/slot-estimator;source=EDITION_UNSPECIFIED;resourceType=PROJECT;resource={project_id}?project={project_id}
    ```

-   **组织级按需计费推荐：**

    ```text
    https://console.cloud.google.com/bigquery/admin/reservations;region={region}/slot-estimator;source=EDITION_UNSPECIFIED;resourceType=ORGANIZATION;resource={project_id}?project={project_id}
    ```

### 导航与工作负载分析

1.  引导用户导航到容量管理 UI 中的 **Slot 估算器**标签页。
2.  从选择面板中选择 **按需计费** 作为来源。
3.  查看 **成本优化推荐** 和 **Slot 使用图表**，比较按量付费与 1 年期和 3 年期容量承诺。
4.  **工作负载分析策略：**
    -   *稳定基线工作负载：* 全天 Slot 需求平稳、连续的工作负载（例如持续使用 400 个 Slot）。这些是版本承诺（1 年期或 3 年期 CUD）的首要候选，可最大化成本效益（约 40% 折扣）。
    -   *尖峰工作负载：* 大多数时间需要零个或少量 Slot，但有急剧突发的工作负载。这些受益于低基线分配和动态自动扩缩，但需要保守的自动扩缩上限以避免意外的突发成本。

## 3. 现有版本容量合理配置

当组织已在 BigQuery 版本上运行，并希望审计活跃的版本容量、合理配置承诺购买（CUD）并消除未覆盖的基线 PAYG 浪费时使用。

### Slot 估算器合理配置深层链接

提供针对特定版本的直接深层链接：

-   **版本合理配置推荐：**

    ```text
    https://console.cloud.google.com/bigquery/admin/reservations;region={region}/slot-estimator;source={edition}?project={admin_project_id}
    ```

    *（其中 `{edition}` 为 `ENTERPRISE` 或 `ENTERPRISE_PLUS`；Standard 版本不支持承诺，也不会在 Slot 估算器中生成推荐）*

### 版本级合理配置决策手册

1.  **消除未覆盖的基线 PAYG Slot：**
    -   *问题：* 版本中跨预留配置的总基线 Slot 超过活跃容量承诺（`total_baseline_slots > committed_slots`）。未覆盖的基线 Slot 按标准按量付费（PAYG）费率 24/7 计费。
    -   *操作：* 在版本级别购买 1 年期或 3 年期容量承诺，以折扣费率（约 40% 折扣）覆盖基线容量缺口。
2.  **将版本总承诺合理配置到稳定基线：**
    -   *问题：* 承诺在版本级别购买和管理，但版本中所有预留的聚合稳态 Slot 需求可能已发生变化。
    -   *操作：* 使用 Slot 估算器将 1 年期/3 年期承诺与版本的聚合连续基线下限对齐，允许动态自动扩缩吸收该下限以上的可变查询需求。
3.  **审计超额承诺过度配置：**
    -   *问题：* 购买超过版本聚合 Slot 利用率的承诺会导致为未使用的承诺容量付费。
    -   *操作：* 查看 Slot 估算器中版本的 30 天历史使用图表，确保承诺续订和购买不超过真实的稳态需求下限。