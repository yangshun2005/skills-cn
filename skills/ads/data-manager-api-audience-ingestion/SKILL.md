---
name: data-manager-api-audience-ingestion
description: >-
  指导开发者使用 Data Manager API 及其客户端库管理（添加、移除和清空）Google 产品的受众成员。当用户想要上传受众成员、移除特定用户，或为 Customer Match、移动设备 ID 受众或 Data Manager API 支持的任何其他受众用例清空/替换整个受众时，使用此技能。不要用于上传事件或转化（请使用 data-manager-api-event-ingestion 技能）。
metadata:
  version: "1.1.0"
  category: GoogleAds
---

# Data Manager API 受众数据导入

## 实施工作流

### 前提条件

-   **身份验证与库安装**：如需设置 Data Manager API 的访问权限，或安装客户端和工具库，请参阅 `data-manager-api-setup` 技能。
-   **受众创建（如需要）**：如果用户没有现成的受众或需要创建新受众，请使用[创建受众](references/create-audience.md)参考文档。此步骤提供导入或移除请求所需的 `product_destination_id`。

### 第 1 步：确定用例并阅读文档

-   **确定目标账号类型**：[关键] 如果不清楚数据发送到哪里（例如 Google Ads、Display & Video 360 等），在生成任何代码之前，停止并与用户确认。不要默认假设是 Google Ads。这对应 `Destination` 中 `operating_account` 的 `account_type` 字段。
-   **阅读实施指南**：阅读目标平台和用例对应的相关指南。在回答问题或编写代码之前先做这一步，因为每个目标平台都有独特的载荷结构、同意规则和必填字段。

| 目标平台 | 受众类型 | 接受的数据类型 | 上传指南 | 全部移除/全部替换指南 |
| :--- | :--- | :--- | :--- | :--- |
| **Google Ads** | Customer Match | `composite_data.user_data`（联系信息）、`mobile_data`（设备 ID）、`user_id_data`（用户 ID） | [上传数据](https://developers.google.com/data-manager/api/devguides/audiences/google-ads/customer-match/upload-data.md.txt) | [全部移除/全部替换](https://developers.google.com/data-manager/api/devguides/audiences/google-ads/customer-match/remove-all-members.md.txt) |
| **Display & Video 360** (DV360) | Customer Match | `composite_data.user_data`（联系信息）、`mobile_data`（设备 ID） | [上传数据](https://developers.google.com/data-manager/api/devguides/audiences/display-video/customer-match/upload-data.md.txt) | [全部移除/全部替换](https://developers.google.com/data-manager/api/devguides/audiences/display-video/customer-match/remove-all-members.md.txt) |

### 第 2 步：获取代码示例

> [!IMPORTANT] 编写或更新导入脚本时，务必获取相关代码示例作为参考：

| 语言 | 示例 |
| :--- | :--- |
| **Python** | [`ingest_audience_members.py`](https://github.com/googleads/data-manager-python/blob/main/samples/audiences/ingest_audience_members.py) |
| **Java** | [`IngestAudienceMembers.java`](https://github.com/googleads/data-manager-java/blob/main/data-manager-samples/src/main/java/com/google/ads/datamanager/samples/IngestAudienceMembers.java) |
| **PHP** | [`ingest_audience_members.php`](https://github.com/googleads/data-manager-php/blob/main/samples/audiences/ingest_audience_members.php) |
| **Node** | [`ingest_audience_members.ts`](https://github.com/googleads/data-manager-node/blob/main/samples/audiences/ingest_audience_members.ts) |
| **.NET**| [`IngestAudienceMembers.cs`](https://github.com/googleads/data-manager-dotnet/blob/main/samples/IngestAudienceMembers.cs) |

### 第 3 步：获取迁移指南

> [!IMPORTANT] 如果重构代码以从其他 Google API 升级，务必提取相关字段映射指南的完整内容。

#### Google Ads

*   **Google Ads API Customer Match**：[Google Ads API 到 Customer Match 迁移字段映射](https://developers.google.com/data-manager/api/devguides/audiences/google-ads/customer-match/upgrade/field-mappings.md.txt)

#### Display & Video 360

*   **Display & Video 360 API Customer Match**：[Display & Video 360 API 到 Customer Match 迁移字段映射](https://developers.google.com/data-manager/api/devguides/audiences/display-video/customer-match/upgrade/field-mappings.md.txt)

### 第 4 步：实施

使用以下检查点实施导入逻辑：

-   [ ] **初始化客户端**：实例化 Data Manager 客户端（`IngestionServiceClient`）。
-   [ ] **定义目标**：使用 `product_destination_id` 和相应的账号配置构建 `Destination` 对象：`operating_account`（接收数据的目标账号）、`login_account`（如果使用经理账号或数据合作伙伴账号进行身份验证）和 `linked_account`（如果你是数据合作伙伴，通过合作伙伴链接到经理账号访问该账号）。**强烈建议**：参阅[配置目标和请求头](https://developers.google.com/data-manager/api/devguides/concepts/destinations.md.txt)指南了解目标配置的更多细节。
-   [ ] **格式化用户数据**：如果发送 `IngestAudienceMembersRequest` 或 `RemoveAudienceMembersRequest`，请参阅**[格式化用户数据](references/formatting.md)**，使用工具库正确规范化和哈希用户标识符。
-   [ ] **构建载荷**：根据操作构建相应的请求载荷：
    *   **添加**：`IngestAudienceMembersRequest`
    *   **移除**：`RemoveAudienceMembersRequest`
    *   **全部移除**：`RemoveAllAudienceMembersRequest`
-   [ ] **支持验证**：支持在载荷上发送 `validate_only` 布尔选项，让开发者在不实际应用更改的情况下验证 Schema。
-   [ ] **发送请求**：执行相应方法并记录返回的 `request_id` 以供后续诊断：
    *   **添加**：`ingest_audience_members`
    *   **移除**：`remove_audience_members`
    *   **全部移除**：`remove_all_audience_members`
-   [ ] **检查导入警告**：如果任何非必填字段验证失败，`ingest_audience_members` 的响应还会包含 `field_warnings`，即详细说明问题的 `FieldWarning` 对象列表。
-   [ ] **获取请求状态**：使用诊断功能检查导入请求的状态。由于请求处理是异步的，成功响应（HTTP 200 OK 返回 `request_id`）仅表示载荷已收到。要检查记录是实际成功、部分成功还是处理失败，请使用 `request_id` 查询 `client.retrieve_request_status`。跳过此步骤是常见的用户错误。

## 关键注意事项

*   如果在 `ingest_audience_members` 或 `remove_audience_members` 的 `user_data` 中发送哈希后的用户标识符，必须将 `IngestAudienceMembersRequest` 的 `encoding` 字段设置为 `HEX` 或 `BASE64`。
*   如果*上传*到 Customer Match 受众，`IngestAudienceMembersRequest` 上必须设置 `terms_of_service` 字段，以表明用户已接受相关策略。
*   仅当所有必填字段（`postal_code`、`family_name`、`given_name`、`region_code`）都存在时，才在 `UserIdentifier` 上设置 `address` 字段；不完整的 `address` 字段会导致 API 请求失败。
*   `product_destination_id` 必须是数字字符串，不是资源名称。
*   `ConsentStatus` 的枚举值为 `CONSENT_GRANTED` 和 `CONSENT_DENIED`。不要使用 `GRANTED` 和 `DENIED`。
*   `UserIdentifier` 的字段名为 `email_address` 和 `phone_number`。不要使用 Google Ads API 的字段名 `hashed_email` 和 `hashed_phone_number`。
*   如果 `validate_only` 设置为 `true`，不要调用诊断端点（`retrieve_request_status`）。

## 错误处理与故障排除

### 检查错误载荷与导入警告

> [!IMPORTANT]
> 参阅[了解 API 错误](https://developers.google.com/data-manager/api/devguides/concepts/understand-errors.md.txt)获取关于如何理解 API 返回的错误和警告结构的详细指南。

### 获取请求状态（诊断）

使用指数退避定期轮询状态，至少在发送请求 30 分钟后开始。

1.  使用 `RetrieveRequestStatusRequest(request_id=...)` 调用 `client.retrieve_request_status`。
2.  遍历响应中的 `request_status_per_destination`，检查每个目标的 `request_status`。
3.  如果处理完成且 `request_status` 为 `SUCCESS`、`PARTIAL_SUCCESS` 或 `FAILED`，检查诊断值：
    *   **受众状态**：检查请求对应的状态：
        *   **导入**：检查嵌套在 `audience_members_ingestion_status` 下的数据类型特定状态（例如 `composite_data_ingestion_status`）。
        *   **移除单个成员**：检查嵌套在 `audience_members_removal_status` 下的数据类型特定状态（例如 `composite_data_removal_status`）。
        *   **移除全部成员**：此请求类型没有可检查的嵌套状态字段或记录数。
        *   **记录数**：如适用（导入或移除单个成员），检查 `record_count`（嵌套在数据类型特定状态对象内），包含成功和失败。
        *   **标识符计数**：如适用（导入或移除单个成员），检查嵌套在状态对象内的数据类型特定计数字段（例如上传或移除复合数据时的 `data_type_counts`，或上传或移除移动设备 ID 时的 `mobile_id_count`）。其他计数字段请参阅[诊断指南](https://developers.google.com/data-manager/api/devguides/diagnostics.md.txt)。
        *   **匹配率范围**：对于 `user_data` 和 `composite_data` 的上传，检查嵌套在状态对象内的 `upload_match_rate_range`。
    *   **错误详情**：如果状态为 `FAILED` 或 `PARTIAL_SUCCESS`，检查 `error_info.error_counts` 下每个错误的 `reason` 和 `record_count`。
    *   **警告详情**：检查 `warning_info.warning_counts` 下每个警告的 `reason` 和 `record_count`（即使目标状态为 `SUCCESS`）。

## API 参考

*   [发送受众成员指南](https://developers.google.com/data-manager/api/devguides/audiences/send-audience-members.md.txt)
*   [REST API 参考：导入](https://developers.google.com/data-manager/api/reference/rest/v1/audienceMembers/ingest.md.txt)
*   [REST API 参考：移除](https://developers.google.com/data-manager/api/reference/rest/v1/audienceMembers/remove.md.txt)
*   [REST API 参考：全部移除](https://developers.google.com/data-manager/api/reference/rest/v1/audienceMembers/removeAll.md.txt)