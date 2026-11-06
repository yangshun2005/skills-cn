# 创建受众（用户列表）

如果用户需要在摄取数据前创建新受众，请按以下步骤操作。

## 参考

[关键] 请遵循相关指南，了解示例请求以及如何使用 Data Manager API 创建受众的更多信息：

*   **Google Ads Customer Match**：[创建 Customer Match 受众](https://developers.google.com/data-manager/api/devguides/audiences/google-ads/customer-match/create-audience.md.txt)
*   **Display & Video 360 Customer Match**：[创建 Customer Match 受众](https://developers.google.com/data-manager/api/devguides/audiences/display-video/customer-match/create-audience.md.txt)

## 常见错误与注意事项

以下示例使用 Python，但概念适用于所有客户端库。

1.  正确的客户端是 `UserListServiceClient`，而不是 `UserListsClient`。

```python
from google.ads import datamanager_v1

user_list_client = datamanager_v1.UserListServiceClient()
```

2.  不要传入 Google Ads 专用请求头，如 `login-customer-id` 或 `developer-token`。如有需要，在资源管理请求中设置格式为资源名称的 `login-account` 或 `linked-account` 请求头（例如 `accountTypes/{account_type}/accounts/{id}`）。

    *   如果使用经理账号或数据合作伙伴账号进行身份验证，请设置 `login-account`。
    *   如果你是数据合作伙伴，通过合作伙伴链接访问经理账号下的账号，请设置 `linked-account`。
    *   请参阅[配置目标与请求头](https://developers.google.com/data-manager/api/devguides/concepts/destinations.md.txt)文档，了解不同资源管理请求访问场景下的示例请求头。

```python
headers = []
account_type = "GOOGLE_ADS"
login_account_id = "1234567890"

if login_account_id:
    headers.append(
        (
            "login-account",
            f"accountTypes/{account_type}/accounts/{login_account_id}",
        )
    )

response = user_list_client.create_user_list(
    parent=parent_account, user_list=user_list_data, metadata=headers
)
```