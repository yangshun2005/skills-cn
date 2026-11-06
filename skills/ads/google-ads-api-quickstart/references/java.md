# Google Ads API Java 设置参考

本指南介绍如何配置、安装并使用 Java 编写第一个 Google Ads API 调用。

## 前提条件

*   **Java 开发工具包（JDK）：** 版本 1.8 或更高。建议从[支持的客户端库版本](https://developers.google.com/google-ads/api/docs/client-libs.md.txt#supported_api_versions)动态查询最低所需版本。
*   **构建工具：** Maven 或 Gradle。

---

## 第 1 步：安装与依赖管理

必须将 Google Ads 客户端库添加到项目中。

> [!IMPORTANT]
> **需要执行的操作：** 添加依赖前，必须从 [Google Ads Java 库 GitHub Releases](https://github.com/googleads/google-ads-java/releases) 查询最新发布版本，并将下方的 `LATEST_LIBRARY_VERSION` 替换为实际版本号（例如 `33.0.0`）。

### 选项 A：Maven（`pom.xml`）
在 `<dependencies>` 部分添加以下内容：
```xml
<dependency>
  <groupId>com.google.api-ads</groupId>
  <artifactId>google-ads</artifactId>
  <version>LATEST_LIBRARY_VERSION</version>
</dependency>
```

### 选项 B：Gradle（`build.gradle`）
在 `dependencies` 块中添加以下内容：
```groovy
implementation 'com.google.api-ads:google-ads:LATEST_LIBRARY_VERSION'
```

---

## 第 2 步：配置（`ads.properties`）

> [!IMPORTANT]
> **强制代理指令（配置加载机制）：**
> 在指导用户完成 Java 快速入门时，你的回复**必须**明确解释 `fromPropertiesFile()` 使用的配置加载机制和查找层级：
> 1. 直接传给 `fromPropertiesFile(PATH_TO_CONFIG_FILE)` 的显式路径。
> 2. `GOOGLE_ADS_CONFIGURATION_FILE_PATH` 环境变量指定的位置（若已设置）。
> 3. 否则，使用用户主目录（`System.getProperty("user.home") + "/ads.properties"`）或类路径下的默认 `ads.properties` 文件。
> 
> 最终输出中不得省略此说明。

创建名为 `ads.properties` 的配置文件。

也可以通过 `fromEnvironment()` 使用环境变量（如 `GOOGLE_ADS_DEVELOPER_TOKEN`、`GOOGLE_ADS_CLIENT_ID`）配置客户端库，或两种方式结合使用。

> [!TIP]
> **文件位置：** 对于 Maven/Gradle 项目，最佳位置是资源目录：`src/main/resources/ads.properties`。这样可确保文件自动包含在类路径中。

在文件中填入凭据：
```properties
api.googleads.developerToken=INSERT_DEVELOPER_TOKEN_HERE
api.googleads.clientId=INSERT_OAUTH2_CLIENT_ID_HERE
api.googleads.clientSecret=INSERT_OAUTH2_CLIENT_SECRET_HERE
api.googleads.refreshToken=INSERT_OAUTH2_REFRESH_TOKEN_HERE
# Optional: Un-comment if you are accessing a client account through a manager account
# api.googleads.loginCustomerId=INSERT_LOGIN_CUSTOMER_ID_HERE
```

---

## 第 3 步：编写快速入门脚本

创建名为 `GetCampaigns.java` 的文件。

> [!IMPORTANT]
> **规则 1：规范化客户 ID**
> 将客户 ID 传给 API 之前，**必须**去除所有连字符进行规范化（例如将 `123-456-7890` 转换为 `1234567890`）。否则会导致 API 校验错误。
>
> **规则 2：解析包版本（`vXX`）**
> 下方导入语句包含占位符 `vXX`。**必须**将 `vXX` 替换为目标 API 版本（与第 1 步安装的库版本对应）。例如，若使用库 v33.0.0，API 版本通常为 `v24`（包名为 `com.google.ads.googleads.v24.services...`）。

```java
import com.google.ads.googleads.lib.GoogleAdsClient;
// IMPORTANT: Replace vXX with the active API version (e.g., v24)
import com.google.ads.googleads.vXX.services.GoogleAdsServiceClient;
import com.google.ads.googleads.vXX.services.SearchGoogleAdsStreamRequest;
import com.google.ads.googleads.vXX.services.SearchGoogleAdsStreamResponse;
import com.google.ads.googleads.vXX.errors.GoogleAdsException;
import java.io.IOException;

public class GetCampaigns {
    public static void main(String[] args) {
        if (args.length < 1) {
            System.err.println("Usage: java GetCampaigns <CUSTOMER_ID>");
            System.exit(1);
        }

        // Normalize customer ID by removing hyphens
        String customerId = args[0].replace("-", "");
        
        GoogleAdsClient googleAdsClient;
        try {
            // Combines environment variables (fromEnvironment) and classpath/home configuration (fromPropertiesFile)
            googleAdsClient = GoogleAdsClient.newBuilder()
                    .fromEnvironment()
                    .fromPropertiesFile()
                    .build();
        } catch (IOException e) {
            System.err.println("Failed to load configuration. Ensure ads.properties is in src/main/resources/ or environment variables are set.");
            e.printStackTrace();
            System.exit(1);
            return;
        }

        // The try-with-resources block ensures the gRPC client channel is closed automatically
        try (GoogleAdsServiceClient googleAdsServiceClient = googleAdsClient.getLatestVersion().createGoogleAdsServiceClient()) {
            
            String query = "SELECT campaign.id, campaign.name, campaign.status FROM campaign ORDER BY campaign.id";
            
            SearchGoogleAdsStreamRequest request = SearchGoogleAdsStreamRequest.newBuilder()
                    .setCustomerId(customerId)
                    .setQuery(query)
                    .build();

            System.out.println("Querying Google Ads API...");
            
            // Execute the stream request
            googleAdsServiceClient.searchStreamCallable().call(request).forEach(response -> {
                response.getResultsList().forEach(row -> {
                    System.out.printf("Campaign found: ID = %d, Name = '%s', Status = %s%n",
                            row.getCampaign().getId(),
                            row.getCampaign().getName(),
                            row.getCampaign().getStatus().name());
                });
            });
        } catch (GoogleAdsException gae) {
            System.err.printf("Request ID %s failed.%n", gae.getRequestId());
            System.err.printf("Error details: %s%n", gae.getMessage());
            System.exit(1);
        }
    }
}
```

---

## 第 4 步：编译与运行

如果文件依赖 Maven/Gradle 依赖，不要单独运行。使用构建工具编译并执行应用，以确保类路径配置正确。

### 使用 Maven
在项目根目录运行以下命令：
```bash
mvn compile exec:java -Dexec.mainClass="GetCampaigns" -Dexec.args="XXXXXXXXXX"
```

### 使用 Gradle
如果使用 Gradle Application 插件，运行：
```bash
./gradlew run --args="XXXXXXXXXX"
```
*（将 `XXXXXXXXXX` 替换为 10 位客户 ID，带或不带连字符均可）。*

---

## 第 5 步：验证与故障排查

### 预期成功输出
成功执行后，应看到类似以下输出：
```text
Querying Google Ads API...
Campaign found: ID = 123456789, Name = 'Search - Brand - US', Status = ENABLED
Campaign found: ID = 987654321, Name = 'Display - Remarketing', Status = PAUSED
```

### 常见错误

| 错误症状 / 代码 | 根本原因 | 解决方案 |
| :--- | :--- | :--- |
| `FileNotFoundException: ads.properties` | 库找不到配置文件。 | 确保 `ads.properties` 位于 `src/main/resources/`（Maven/Gradle），或通过 `api.googleads.configurationFilePath` 系统属性显式设置路径。 |
| `DEVELOPER_TOKEN_NOT_APPROVED` | 对生产账户使用未批准（"待审核"）的开发者令牌（生产调用需要 Explorer、Basic 或 Standard 访问权限）。 | 使用**测试账户**（允许待审核令牌），或等待令牌批准。 |
| `NOT_ADS_USER` | 用于生成刷新令牌的 OAuth2 用户凭据无权访问指定的 `CUSTOMER_ID`。 | 使用有权访问目标 Ads 账户的 Google 账户重新进行 OAuth2 认证。 |
| `INVALID_CUSTOMER_ID` | 传入的客户 ID 不是数字（例如仍包含连字符，或长度错误）。 | 确认规范化代码 `args[0].replace("-", "")` 已成功执行。 |