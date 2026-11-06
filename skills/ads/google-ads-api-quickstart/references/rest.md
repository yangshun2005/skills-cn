# Google Ads API REST 设置参考

本指南介绍如何在不使用客户端库的情况下，通过 Google Ads API 进行身份验证、配置并执行直接 HTTP REST 请求来获取广告系列。

---

## 第 1 步：获取短期访问令牌

> [!IMPORTANT]
> **代理强制指令（OAuth 授权路径）：**
> 在引导用户完成 REST 快速入门时，你的回复**必须**明确说明两种 OAuth2 授权路径：
> 1. **选项 A：服务账号流程（推荐）：** 说明如何创建服务账号、授予广告账号直接访问权限、生成 JWT 声明集，并将其换取访问令牌。
> 2. **选项 B：用户身份验证流程（备选）：** 说明如何通过 cURL 用 `refresh_token`、`client_id` 和 `client_secret` 换取访问令牌。
>
> 最终输出中不得省略任一流程。

客户端库会自动刷新 OAuth2 令牌，而使用直接 REST 接口时，你必须手动请求访问令牌。

### 选项 A：服务账号流程（推荐）
对于服务器到服务器的 REST 应用，使用服务账号是推荐的最佳实践：
1. **设置云项目：** 在 Google Cloud Console 中创建服务账号，并以 JSON 格式下载私钥。记录服务账号邮箱和密钥。
2. **授予账号访问权限：** 在 Google Ads 界面中授予服务账号邮箱对你 Google Ads 账号的直接访问权限（定位账号访问级别：探索者、基本或标准）。
3. **生成 JWT 声明集：** 按照 [OAuth 2.0 服务器到服务器应用](https://developers.google.com/identity/protocols/oauth2/service-account#authorizingrequests) 指南（选择 HTTP/REST 标签页）创建并签名 JWT 声明集。
   * **范围：** 使用 `https://www.googleapis.com/auth/adwords`。
   * **模拟（`sub`）：** 由于服务账号已直接访问 Google Ads 账号，构建 JWT 声明集时可跳过 `sub` 参数。
4. **用 JWT 换取访问令牌：** 通过 Google OAuth2 令牌端点（`https://oauth2.googleapis.com/token`）将已签名的 JWT 换取短期 `access_token`。

### 选项 B：用户身份验证流程（备选）
如果你以个人用户身份进行身份验证，请用 `refresh_token`、`client_id` 和 `client_secret` 换取短期 `access_token`（通常有效期为 1 小时）：

```bash
curl \
  --data "grant_type=refresh_token" \
  --data "client_id=INSERT_OAUTH2_CLIENT_ID_HERE" \
  --data "client_secret=INSERT_OAUTH2_CLIENT_SECRET_HERE" \
  --data "refresh_token=INSERT_OAUTH2_REFRESH_TOKEN_HERE" \
  https://www.googleapis.com/oauth2/v3/token
```

### 预期 JSON 响应：
```json
{
  "access_token": "ya29.a0AfH6S...",
  "expires_in": 3599,
  "scope": "https://www.googleapis.com/auth/adwords",
  "token_type": "Bearer"
}
```

复制 `"access_token"` 的值，用于后续 API 请求的身份验证。

---

## 第 2 步：发送 HTTP POST 请求以获取广告系列

Google Ads API REST 接口通过 **Google Ads 查询语言（GAQL）** 提供统一的资源获取机制。要获取广告系列，请向 `searchStream` 端点发送 `POST` 请求。

*   **端点 URL：** `https://googleads.googleapis.com/vXX/customers/{customer_id}/googleAds:searchStream`
*   **方法：** `POST`
*   **请求头：**
    *   `Content-Type: application/json`
    *   `developer-token: <INSERT_DEVELOPER_TOKEN_HERE>`
    *   `Authorization: Bearer <INSERT_OAUTH2_ACCESS_TOKEN_HERE>`
    *   `login-customer-id: <INSERT_LOGIN_CUSTOMER_ID_HERE>` *（仅当通过经理账号进行身份验证时需要）*

> [!TIP]
> **捕获请求 ID：** 使用 cURL 时，添加 `--include`（或 `-i`）标志以查看响应 HTTP 头。`request-id` 头唯一标识你的 API 请求，对调试或联系 Google 开发者支持非常有价值。

### 通过 cURL 执行：
```bash
# 设置目标客户 ID（仅 10 位数字，不含连字符）
CUSTOMER_ID="1234567890"

curl --include --request POST "https://googleads.googleapis.com/vXX/customers/${CUSTOMER_ID}/googleAds:searchStream" \
  --header "Content-Type: application/json" \
  --header "developer-token: INSERT_DEVELOPER_TOKEN_HERE" \
  --header "Authorization: Bearer INSERT_OAUTH2_ACCESS_TOKEN_HERE" \
  --header "login-customer-id: INSERT_LOGIN_CUSTOMER_ID_HERE" \
  --data '{
    "query": "SELECT campaign.id, campaign.name, campaign.status FROM campaign ORDER BY campaign.id"
  }'
```

---

## 第 3 步：解析 JSON 响应

与返回单个 JSON 对象的典型 REST 端点不同，`searchStream` 方法以 JSON 数组包装的形式流式返回结果块。每个块包含一组广告系列资源：

```json
[
  {
    "results": [
      {
        "campaign": {
          "resourceName": "customers/1234567890/campaigns/987654321",
          "id": "987654321",
          "name": "Interstate Search Promo",
          "status": "ENABLED"
        }
      },
      {
        "campaign": {
          "resourceName": "customers/1234567890/campaigns/555444333",
          "id": "555444333",
          "name": "Local Brand Awareness",
          "status": "PAUSED"
        }
      }
    ],
    "fieldMask": "campaign.id,campaign.name,campaign.status"
  }
]
```