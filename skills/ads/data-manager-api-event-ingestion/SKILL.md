---
name: data-manager-api-event-ingestion
description: >-
  指导开发者使用 Data Manager API 的 /v1/events/ingest 端点及其关联客户端库，向 Google 产品实现事件和转化数据接入。当用户想要上传离线转化、潜在客户增强转化、点击转化、Google Analytics 网站或应用事件，或 Data Manager API 支持的其他事件接入用例时，使用本技能。不要用于上传受众成员（请使用 data-manager-api-audience-ingestion 技能）。
metadata:
  version: "1.2.0"
  category: GoogleAds
---
# Data Manager API 事件接入

## 实施工作流

### 前提条件

-   **身份验证与库安装**：如需设置 Data Manager API 的访问权限，或安装客户端与工具库，请参阅 `data-manager-api-setup` 技能。

### 第 1 步：确定用例并阅读文档

-   **确定目标账号类型**：[关键] 如果无法从用户上下文中确定，请在生成任何代码前先澄清事件要接入到哪个目标。这对应 `Destination` 中 `operating_account` 的 `account_type` 字段，也决定了有效的事件标识符和要求。
-   **识别用户意图**：
    -   **实现接入代码**：按照下方 **实施指南** 列中对应目标和用例的指南操作。这对确保满足字段要求、正确配置目标至关重要。
    -   **检查请求状态或排查错误**：参阅下方 [错误处理与故障排查](#error-handling-troubleshooting) 部分。
    -   **从其他 Google API 迁移**：参阅下方 [第 3 步：获取迁移指南](#step-3-retrieve-migration-guides)，提取相关字段映射指南的完整内容。

| 目标（`operating_account.account_type`） | 用例 | 实施指南 |
| :--- | :--- | :--- |
| **Google Ads**（`GOOGLE_ADS`） | 离线转化、潜在客户增强转化 | [发送事件](https://developers.google.com/data-manager/api/devguides/events/google-ads/offline/send-events.md.txt) |
| **Google Ads**（`GOOGLE_ADS`） | 补充 Google tag 的多来源转化 | [发送事件](https://developers.google.com/data-manager/api/devguides/events/google-ads/online/send-events.md.txt) |
| **Google Ads**（`GOOGLE_ADS`） | 门店销售转化 | [发送事件](https://developers.google.com/data-manager/api/devguides/events/google-ads/store-sales/send-events.md.txt) |
| **Google Analytics**（`GOOGLE_ANALYTICS_PROPERTY`） | 推荐事件和自定义 GA4 事件 | [发送事件](https://developers.google.com/data-manager/api/devguides/events/analytics/recommended-custom-events/send-events.md.txt) |
| **Google Analytics**（`GOOGLE_ANALYTICS_PROPERTY`） | 带交易 ID 的多来源事件 | [发送事件](https://developers.google.com/data-manager/api/devguides/events/analytics/online/send-events.md.txt) |
| **Floodlight**（`FLOODLIGHT_CONFIG`） | Floodlight 离线转化 | [发送事件](https://developers.google.com/data-manager/api/devguides/events/cm360/offline/send-events.md.txt) |
| **Floodlight**（`FLOODLIGHT_CONFIG`） | 补充 Google 或 Floodlight tag 的多来源转化 | [发送事件](https://developers.google.com/data-manager/api/devguides/events/cm360/online/send-events.md.txt) |

如果请求不匹配任何一行，请获取 [Events 概览](https://developers.google.com/data-manager/api/devguides/events.md.txt) 以找到正确的指南，不要猜测。

### 第 2 步：获取代码示例

> [!IMPORTANT]
> 如果编写或更新接入脚本，务必获取相关代码示例作为参考：

| 语言 | 示例 |
| :--- | :--- |
| **Python** | [`ingest_events.py`](https://github.com/googleads/data-manager-python/blob/main/samples/events/ingest_events.py) |
| **Java** | [`IngestEvents.java`](https://github.com/googleads/data-manager-java/blob/main/data-manager-samples/src/main/java/com/google/ads/datamanager/samples/IngestEvents.java) |
| **PHP** | [`ingest_events.php`](https://github.com/googleads/data-manager-php/blob/main/samples/events/ingest_events.php) |
| **Node** | [`ingest_events.ts`](https://github.com/googleads/data-manager-node/blob/main/samples/events/ingest_events.ts) |
| **.NET**| [`IngestEvents.cs`](https://github.com/googleads/data-manager-dotnet/blob/main/samples/IngestEvents.cs) |

### 第 3 步：获取迁移指南

> [!IMPORTANT]
> 如果重构代码以从其他 Google API 升级，务必提取相关字段映射指南的完整内容。

#### Google Ads

*   **离线转化、潜在客户增强转化**（从 Google Ads API 迁移）：
    [Google Ads 离线转化迁移字段映射](https://developers.google.com/data-manager/api/devguides/events/google-ads/offline/upgrade/field-mappings.md.txt)
*   **门店销售转化**（从 Google Ads API 迁移）：
    [Google Ads 门店销售迁移字段映射](https://developers.google.com/data-manager/api/devguides/events/google-ads/store-sales/upgrade/field-mappings.md.txt)

#### Google Analytics

*   **推荐事件和自定义 GA4 事件**（从 Google Analytics Measurement Protocol 迁移）：
    [Google Analytics Measurement Protocol 迁移字段映射](https://developers.google.com/data-manager/api/devguides/events/analytics/measurement-protocol/upgrade/field-mappings.md.txt)

#### Floodlight

*   **Floodlight 离线转化**（从 Campaign Manager 360 API 迁移）：
    [Campaign Manager 360 离线转化迁移字段映射](https://developers.google.com/data-manager/api/devguides/events/cm360/offline/upgrade/field-mappings.md.txt)

### 第 4 步：实施

使用以下检查点实现接入逻辑：

-   [ ] **初始化客户端**：实例化 Data Manager 客户端（`IngestionServiceClient`）。
-   [ ] **定义目标**：使用 `product_destination_id` 和相应的账号配置构建 `Destination` 对象：`operating_account`（接收数据的目标账号）、`login_account`（如果使用经理账号或数据合作伙伴账号进行身份验证）、`linked_account`（如果你是数据合作伙伴，通过合作伙伴链接访问经理账号下的账号）。**强烈建议**：参阅 [配置目标和请求头](https://developers.google.com/data-manager/api/devguides/concepts/destinations.md.txt) 指南了解目标配置的更多细节。
-   [ ] **准备事件数据**：使用工具库辅助方法正确格式化和规范化用户标识符。
-   [ ] **构建载荷**：构建请求载荷（`IngestEventsRequest`），包含目标、事件记录和同意权限。
-   [ ] **支持验证**：支持在 `IngestEventsRequest` 上发送 `validate_only` 布尔选项，让开发者在不实际上传数据的情况下验证 Schema。
-   [ ] **发送请求**：执行 `ingest_events` 并记录返回的 `request_id` 以供后续诊断。
-   [ ] **检查接入警告**：如果任何非必填字段验证失败，`ingest_events` 的响应还会包含 `field_warnings`，即详细说明问题的 `FieldWarning` 对象列表。
-   [ ] **获取请求状态**：通过诊断检查接入请求的状态。由于请求处理是异步的，成功的接入响应（HTTP 200 OK 返回 `request_id`）仅表示载荷已收到。要检查记录是否实际成功、部分成功或处理失败，请使用 `request_id` 查询 `client.retrieve_request_status` 端点。跳过此步骤是常见的用户错误。

## 格式化

*   获取 [格式化用户数据](https://developers.google.com/data-manager/api/devguides/concepts/formatting.md.txt) 指南，将其作为格式化和规范化规则的权威依据。

*   使用工具库格式化、哈希和加密用户数据（邮箱、电话号码、地址）。

    **Python 示例：**

    ```python
    from google.ads.datamanager_util import Formatter
    from google.ads.datamanager_util.format import Encoding

    formatter: Formatter = Formatter()

    processed_email: str = formatter.process_email_address(
        email, Encoding.HEX
    )
    ```

## 关键注意事项

*   `product_destination_id` 格式为数字字符串，不是资源名称路径。
*   `event_timestamp` 严格使用 RFC 3339 格式。如有可用，使用 SDK 的类型化时间戳对象而非原始字符串。
*   将点击标识符（`gclid`、`gbraid`、`wbraid`）嵌套在 `ad_identifiers` 块内，不要直接放在基础事件载荷上。
*   `ConsentStatus` 的枚举值为 `CONSENT_GRANTED` 和 `CONSENT_DENIED`。不要使用 `GRANTED` 和 `DENIED`。
*   注意 `consent` 可以在 `IngestEventsRequest` 上全局设置，也可以在单个 `Event` 上设置。
*   确认 `UserIdentifier` 使用 `email_address` 和 `phone_number`。不要使用 Google Ads API 的 `hashed_email` 和 `hashed_phone_number` 字段。
*   确保事件上的货币字段名为 `currency`，而非 `currency_code`。
*   如果 `validate_only` 设为 `true`，不要调用诊断端点（`retrieve_request_status`）。

## 错误处理与故障排查

### 检查错误载荷与接入警告

> [!IMPORTANT]
> 参阅 [理解 API 错误](https://developers.google.com/data-manager/api/devguides/concepts/understand-errors.md.txt)，了解如何理解 API 返回的错误和警告结构的详细指南。

### 获取请求状态（诊断）

使用指数退避定期轮询状态，至少在发送 `IngestEventsRequest` 后 30 分钟开始。

1.  使用 `RetrieveRequestStatusRequest(request_id=...)` 调用 `client.retrieve_request_status`。
2.  遍历响应中的 `request_status_per_destination`，检查每个目标的 `request_status`。
3.  如果处理完成且 `request_status` 为 `SUCCESS`、`PARTIAL_SUCCESS` 或 `FAILED`，检查诊断值：
    *   **事件记录数**：检查 `events_ingestion_status.record_count`（包含成功和失败）。
    *   **错误详情**：如果状态为 `FAILED` 或 `PARTIAL_SUCCESS`，检查 `error_info.error_counts` 下每个错误的 `reason` 和 `record_count`。
    *   **警告详情**：检查 `warning_info.warning_counts` 下每个警告的 `reason` 和 `record_count`（即使目标状态为 `SUCCESS`）。

## API 参考

*   [REST API 参考](https://developers.google.com/data-manager/api/reference/rest/v1/events/ingest.md.txt)
*   [诊断指南](https://developers.google.com/data-manager/api/devguides/diagnostics.md.txt)