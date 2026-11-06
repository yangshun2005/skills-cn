---
name: google-ads-api-account-diagnostics
description: >-
  诊断 Google Ads 账户效果问题，例如转化丢失（价值或数量）、线索量低、以及因广告评级、出价或预算导致的展示份额损失（机会损失）。适用于排查效果突然下滑、分析广告系列展示份额指标、调查线索量低、或查找出价与预算限制。不适用于新建广告系列、直接上传转化事件、或 Google Mobile Ads SDK 集成的一般问题（请改用 gma-android-integrate）。
compatibility: "Google Ads MCP Server（工具：search、list_accessible_customers、get_resource_metadata）；参见 google-ads-api-mcp-setup 技能。"
metadata:
  category: GoogleAds
  author: google-ads-api-team
  version: "1.0.2"
---

# Google Ads API 账户效果诊断技能

本技能说明如何使用 Google Ads MCP 服务器工具诊断常见的账户效果问题。

## 工作流

### 预检工具验证
开始任何诊断任务前，先确认运行时环境中是否有 Google Ads MCP Server 工具（`search`、`list_accessible_customers`、`get_resource_metadata`——这些工具可能带服务器前缀，例如 `customers_list_accessible_customers`）：
- **MCP 工具可用**：进行实时诊断——先[识别活跃客户账户](#identifying-active-client-accounts)，再通过 `search` 工具（参数 `customer_id` 和 `query`）运行 GAQL。
- **MCP 工具缺失**：按[无法使用 MCP 工具时的回退方案](#fallback-when-mcp-tools-are-unavailable)处理。

### 识别活跃客户账户
大多数诊断任务需要向特定客户账户发送 GAQL 查询。如果用户未明确提供客户 ID，必须先调用 `list_accessible_customers`（或 `customers_list_accessible_customers`）工具，获取你有权限访问的客户资源名称/ID。

拿到可访问的客户 ID 列表后，在这些客户账户下用 `search` 工具查询 `customer_client` 资源，找出活跃客户账户。只选择已启用的客户账户，并过滤掉经理账户：
```sql
SELECT
  customer_client.id,
  customer_client.descriptive_name,
  customer_client.status,
  customer_client.manager
FROM customer_client
WHERE customer_client.status = 'ENABLED' AND customer_client.manager = FALSE
```
后续诊断查询只针对此列表中检索到的已启用客户 ID 运行。不要查询已停用账户或经理账户，否则会导致 API 报错。

### 无法使用 MCP 工具时的回退方案
当运行时环境未注册 Google Ads MCP 工具（`search`、`list_accessible_customers`、`get_resource_metadata`——这些工具可能带服务器前缀，例如 `customers_list_accessible_customers`）时：
1. **告知用户**：说明实时账户效果诊断需要 Google Ads MCP Server。
2. **引用安装指南**：引导用户查看 [google-ads-api-mcp-setup](../google-ads-api-mcp-setup/SKILL.md) 技能中的安装说明，为其助手环境安装并配置 Google Ads MCP Server。
3. **提供 GAQL 模板、注意事项与修复指导**：制定并输出完整诊断方案，供用户在 Google Ads 查询构建器中手动执行，或通过 API 集成执行：
   - 提供工作流所需的全部多步 GAQL 查询，包括当前与上一时间窗口对比（例如流量、展示份额、转化细分或变更事件）。
   - 说明所有指标换算与约束注意事项（货币需 `metrics.cost_micros / 1,000,000`；展示份额以小数或 `"< 0.10"` 形式返回；`change_event` 限 30 天窗口且不带 `metrics.*` 时需 `LIMIT 10000`；上传健康度看 `successful_event_count / total_event_count` 及 `status`）。
   - 为每个诊断分支列出具体**可执行的后续步骤**（例如 `search_budget_lost_impression_share` 高时提高广告系列预算，`search_rank_lost_impression_share` 高时改善广告质量或出价）。
4. **不要尝试未授权的本地探测**：**不要**尝试运行 shell 命令、自定义 Python 脚本、本地 SQL/数据库查询或直接 RPC CLI 工具来访问账户数据。这些命令缺少凭据和 API 访问权限，必然失败。

### 1. 转化与转化价值丢失

当转化或转化价值突然下降时，按以下步骤诊断。

**步骤：**
1.  **发现字段**：用 `get_resource_metadata` 查询资源 `campaign` 或 `ad_group`，确认字段名正确。
2.  **查询效果**：用 `search` 获取效果数据。
    *   **资源**：`campaign` 或 `ad_group`
    *   **字段**：包含 `campaign.name`、`metrics.conversions`、`metrics.conversions_value`、`metrics.cost_micros`。
    *   **细分**：为定位丢失来源，可加入 `segments.date`、`segments.device`、`segments.conversion_action` 等细分。
    *   **条件**：将下降期与上一期对比（例如 `segments.date >= '{start_date}'`）。
    *   *注意*：`metrics.cost_micros` 必须除以 1,000,000 才能得到标准货币金额。
    *   *注意（转化操作细分）*：在 GAQL 中，按 `segments.conversion_action` 细分时不能选择 `metrics.cost_micros`。转化操作细分需单独查询，且不带 `metrics.cost_micros`。

    **GAQL 查询示例：**
    1. 按日期和设备查询效果与成本：
    ```sql
    SELECT
      campaign.name,
      metrics.conversions,
      metrics.conversions_value,
      metrics.cost_micros,
      segments.date,
      segments.device
    FROM campaign
    WHERE segments.date >= '{start_date}' AND segments.date <= '{end_date}'
    ```

    2. 按转化操作查询转化细分（不带 `metrics.cost_micros`）：
    ```sql
    SELECT
      campaign.name,
      metrics.conversions,
      metrics.conversions_value,
      segments.date,
      segments.conversion_action
    FROM campaign
    WHERE segments.date >= '{start_date}' AND segments.date <= '{end_date}'
    ```

3.  **分析**：检查丢失是否集中在某些设备（如移动端 vs 桌面端）或特定转化操作。
4.  **检查上传**：若使用离线导入，查询 `offline_conversion_upload_conversion_action_summary` 验证上传流水线健康度。若查询无结果，说明该账户没有离线上传，报告后继续。

    **GAQL 查询示例：**
    检查客户账户 `{customer_id}` 的上传流水线健康度：
    ```sql
    SELECT
      offline_conversion_upload_conversion_action_summary.conversion_action_name,
      offline_conversion_upload_conversion_action_summary.successful_event_count,
      offline_conversion_upload_conversion_action_summary.total_event_count,
      offline_conversion_upload_conversion_action_summary.status
    FROM offline_conversion_upload_conversion_action_summary
    ```

### 2. 机会损失（展示份额）

要找出因广告评级、出价或预算导致的机会损失，分析展示份额指标。

**步骤：**
1.  **查询展示份额**：用 `search` 获取展示份额指标。
    *   **资源**：`campaign`
    *   **字段**：包含 `campaign.name`、`metrics.search_impression_share`、`metrics.search_rank_lost_impression_share`、`metrics.search_budget_lost_impression_share`。
    *   *注意*：API 返回的展示份额为小数（如 0.35 = 35%）或格式化字符串（如 `"< 0.10"`）。

    **GAQL 查询示例：**
    查询客户账户 `{customer_id}` 在 `{start_date}` 至 `{end_date}` 期间的展示份额指标：
    ```sql
    SELECT
      campaign.name,
      metrics.search_impression_share,
      metrics.search_rank_lost_impression_share,
      metrics.search_budget_lost_impression_share
    FROM campaign
    WHERE segments.date >= '{start_date}' AND segments.date <= '{end_date}'
    ```

2.  **分析**：
    *   `search_budget_lost_impression_share` 高：因预算受限损失机会。
    *   `search_rank_lost_impression_share` 高：因广告评级低（出价或质量问题）损失机会。

### 3. 线索量低诊断

当用户问"为什么最近几天线索量低？"时，按以下系统方法排查。

**步骤：**
1.  **确认下降**：按日期细分查询最近几天与上一期的转化。
2.  **定位原因**：
    *   检查**流量**（点击、展示）是否下降。
    *   检查**转化率**（转化/点击）是否下降。

    **GAQL 查询示例（流量与转化趋势）：**
    按日期细分查询各广告系列的流量与转化率指标：
    ```sql
    SELECT
      campaign.id,
      campaign.name,
      segments.date,
      metrics.impressions,
      metrics.clicks,
      metrics.ctr,
      metrics.conversions,
      metrics.conversions_from_interactions_rate
    FROM campaign
    WHERE segments.date >= '{start_date}' AND segments.date <= '{end_date}'
    ORDER BY segments.date DESC
    ```

3.  **流量下降**：检查展示份额指标（见工作流 2），判断是预算或评级问题，还是搜索量整体下降。
4.  **转化率下降**：按 `segments.device` 或 `segments.conversion_action` 细分，找出具体失效环节。
5.  **检查变更**：查询 `change_event` 资源，看下降开始前后是否对出价、预算或定向做过改动。
    *   *注意（change_event 约束）*：查询 `change_event` 资源时：
        *   必须指定 `LIMIT` 子句，且小于等于 10000。
        *   必须按日期（`change_event.change_date_time`）过滤，且在过去 30 天内。
        *   不能选择效果指标（不支持 `metrics.*`；只能选择 `change_event` 属性和允许的资源字段）。

    **GAQL 查询示例：**
    查询客户账户 `{customer_id}` 在 `{start_date}` 至 `{end_date}` 期间的变更事件：
    ```sql
    SELECT
      change_event.change_date_time,
      change_event.change_resource_name,
      change_event.resource_change_operation,
      change_event.changed_fields
    FROM change_event
    WHERE change_event.change_date_time >= '{start_date}' AND change_event.change_date_time <= '{end_date}'
    LIMIT 10000
    ```

### 4. 离线上传流水线诊断

当特定操作（如门店购买）的离线转化上传不再出现或失败时，按以下步骤诊断。

**步骤：**
1.  **获取客户账户**：若未提供 `{customer_id}`，按[识别活跃客户账户](#identifying-active-client-accounts)获取活跃客户账户。

2.  **验证流水线健康度**：查询活跃客户账户的 `offline_conversion_upload_conversion_action_summary`。
    *   **字段**：包含 `offline_conversion_upload_conversion_action_summary.conversion_action_name`、`offline_conversion_upload_conversion_action_summary.successful_event_count`、`offline_conversion_upload_conversion_action_summary.total_event_count`、`offline_conversion_upload_conversion_action_summary.status`。

    **GAQL 查询示例：**
    ```sql
    SELECT
      offline_conversion_upload_conversion_action_summary.conversion_action_name,
      offline_conversion_upload_conversion_action_summary.successful_event_count,
      offline_conversion_upload_conversion_action_summary.total_event_count,
      offline_conversion_upload_conversion_action_summary.status
    FROM offline_conversion_upload_conversion_action_summary
    ```

3.  **分析**：
    *   *注意*：若 `offline_conversion_upload_conversion_action_summary` 查询无结果或为空（说明该客户账户未配置或没有活跃的离线转化上传），立即停止/中断诊断工作流。直接告知用户可访问账户中不存在离线转化上传数据或摘要，不要重试或尝试编写自定义脚本。
    *   若有结果，对比 `successful_event_count` 与 `total_event_count` 验证上传成功率。检查 `status` 字段诊断失败原因。