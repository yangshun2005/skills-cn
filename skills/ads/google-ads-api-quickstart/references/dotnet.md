# Google Ads API .NET 配置参考

本指南介绍如何配置、安装 .NET/C# 客户端库，并编写第一个 Google Ads API 调用。

## 前置条件
* .NET SDK：需从[支持的客户端库版本](https://developers.google.com/google-ads/api/docs/client-libs.md.txt#supported_api_versions)动态查询
* 包管理器：NuGet

## 第 1 步：安装
使用 NuGet 安装官方包：
```bash
dotnet add package Google.Ads.GoogleAds
```

## 第 2 步：配置（运行时初始化）
配置 .NET 客户端库的首选方式是在运行时初始化 `GoogleAdsConfig` 对象。准备好以下凭据并写入代码：
* 开发者令牌
* OAuth2 客户端 ID
* OAuth2 客户端密钥
* OAuth2 刷新令牌
* 登录客户 ID（可选；以经理账号身份认证时必填）

### 其他配置方式
如需使用外部配置文件或环境变量，请添加对 `Google.Ads.GoogleAds.Extensions` 包的 NuGet 引用，然后显式将配置加载到 `GoogleAdsConfig` 对象：
* **环境变量：** 调用 `config.LoadFromEnvironmentVariables()`。
* **App.config：** 调用 `config.LoadFromDefaultAppConfigSection()`。
* **settings.json / 自定义 JSON：** 使用 `ConfigurationBuilder`，并调用 `config.LoadFromConfigurationRoot(configRoot)`。

## 第 3 步：编写并运行快速入门脚本
创建 `Program.cs`。**将客户 ID 传给 API 之前，必须先去除连字符（例如把 `123-456-7890` 转为 `1234567890`）**，以确保按标准数字解析：

**重要**

将命名空间和服务调用中的 VXX 替换为前置条件步骤中查到的最新支持版本（例如 V24）。

```csharp
using System;
using Google.Ads.GoogleAds.Config;
using Google.Ads.GoogleAds.Lib;
using Google.Ads.GoogleAds.VXX.Services;
using Google.Ads.GoogleAds.VXX.Errors;

class Program {
    static void Main(string[] args) {
        if (args.Length < 1) {
            Console.WriteLine("Usage: dotnet run <CUSTOMER_ID>");
            return;
        }

        // Clean customer ID by stripping hyphens
        string customerId = args[0].Replace("-", "");
        
        GoogleAdsConfig config = new GoogleAdsConfig()
        {
            DeveloperToken = "INSERT_DEVELOPER_TOKEN_HERE",
            OAuth2Mode = OAuth2Flow.APPLICATION,
            OAuth2ClientId = "INSERT_OAUTH2_CLIENT_ID_HERE",
            OAuth2ClientSecret = "INSERT_OAUTH2_CLIENT_SECRET_HERE",
            OAuth2RefreshToken = "INSERT_OAUTH2_REFRESH_TOKEN_HERE",
            // LoginCustomerId = "INSERT_LOGIN_CUSTOMER_ID_HERE"
        };

        // Optional: If using Google.Ads.GoogleAds.Extensions, you can also load overriding environment variables:
        // config.LoadFromEnvironmentVariables();

        GoogleAdsClient client = new GoogleAdsClient(config);
        GoogleAdsServiceClient service = client.GetService(Services.VXX.GoogleAdsService);
        string query = "SELECT campaign.id, campaign.name, campaign.status FROM campaign ORDER BY campaign.id";

        try {
            service.SearchStream(customerId, query, delegate(SearchGoogleAdsStreamResponse response) {
                foreach (GoogleAdsRow row in response.Results) {
                    Console.WriteLine($"Campaign found: ID = {row.Campaign.Id}, Name = '{row.Campaign.Name}', Status = {row.Campaign.Status}");
                }
            });
        } catch (GoogleAdsException ex) {
            Console.WriteLine($"Request failed: ID {ex.RequestId}. Error: {ex.Message}");
        }
    }
}
```

运行项目（将 `XXXXXXXXXX` 替换为 10 位客户 ID）：
```bash
dotnet run XXXXXXXXXX
```

## 第 4 步：验证

检查运行输出。执行成功时，该客户 ID 关联的广告系列会流式输出到控制台，类似如下：

```text
Campaign found: ID = 987654321, Name = 'Interstate Search Promo', Status = Enabled
Campaign found: ID = 555444333, Name = 'Local Brand Awareness', Status = Paused
```