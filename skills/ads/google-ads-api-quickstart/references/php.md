# Google Ads API PHP 设置参考

本指南介绍如何配置、安装并使用 PHP 编写第一个 Google Ads API 调用。

## 前提条件
* PHP：需从[支持的客户端库版本](https://developers.google.com/google-ads/api/docs/client-libs.md.txt#supported_api_versions)动态查询
* 包管理器：Composer

## 第 1 步：安装
安装官方库：
```bash
composer require googleads/google-ads-php
```

## 第 2 步：配置（`google_ads_php.ini`）

> [!IMPORTANT]
> **强制代理指令（配置加载顺序）：**
> 在引导用户完成 PHP 快速入门时，你的回复**必须**明确解释 `fromFile()` 使用的三级配置加载顺序：
> 1. 直接传给 `fromFile('/path/to/google_ads_php.ini')` 的显式路径。
> 2. 环境变量 `GOOGLE_ADS_CONFIGURATION_FILE_PATH` 的值（若已设置）。
> 3. 否则，使用用户主目录（`$HOME`）下的默认 `google_ads_php.ini` 文件。
>
> 最终输出中不得省略此说明。

创建配置文件 `google_ads_php.ini`。对于自包含的项目环境，最佳实践是将此文件放在项目根目录并显式加载。

```ini
; google_ads_php.ini
[GOOGLE_ADS]
developerToken = "INSERT_DEVELOPER_TOKEN_HERE"
; 使用经理账号用户访问客户账号时需要
; loginCustomerId = "INSERT_LOGIN_CUSTOMER_ID_HERE"

[OAUTH2]
clientId = "INSERT_OAUTH2_CLIENT_ID_HERE"
clientSecret = "INSERT_OAUTH2_CLIENT_SECRET_HERE"
refreshToken = "INSERT_OAUTH2_REFRESH_TOKEN_HERE"
```

## 第 3 步：编写并运行快速入门脚本
创建 `get_campaigns.php`。**将客户 ID 传给 API 之前，必须先清理客户 ID，去除连字符和空格。**（例如将 `123-456-7890` 或 `123 456 7890` 转换为 `1234567890`）。下方代码以编程方式实现。

```php
<?php
require __DIR__ . '/vendor/autoload.php';

use Google\Ads\GoogleAds\Lib\VXX\GoogleAdsClientBuilder;
use Google\Ads\GoogleAds\Lib\OAuth2TokenBuilder;
use Google\ApiCore\ApiException;

// 检查命令行参数
if ($argc < 2) {
    printf("Usage: php get_campaigns.php <CUSTOMER_ID>\n");
    exit(1);
}

$rawCustomerId = $argv[1];

// 清理客户 ID，去除连字符和空格
$customerId = str_replace(['-', ' '], '', $rawCustomerId);

// 确定配置文件路径（优先使用本地工作区配置）
$localConfig = __DIR__ . '/google_ads_php.ini';

if (file_exists($localConfig)) {
    // 从显式本地文件构建 OAuth2 凭据
    $oAuth2Token = (new OAuth2TokenBuilder())->fromFile($localConfig)->build();

    // 从显式本地文件构建 Google Ads 客户端
    $googleAdsClient = (new GoogleAdsClientBuilder())
        ->fromFile($localConfig)
        ->withOAuth2Credential($oAuth2Token)
        ->build();
} else {
    // 回退到默认搜索路径（GOOGLE_ADS_CONFIGURATION_FILE_PATH 环境变量或 $HOME/google_ads_php.ini）
    $oAuth2Token = (new OAuth2TokenBuilder())->fromFile()->build();

    $googleAdsClient = (new GoogleAdsClientBuilder())
        ->fromFile()
        ->withOAuth2Credential($oAuth2Token)
        ->build();
}

$googleAdsServiceClient = $googleAdsClient->getGoogleAdsServiceClient();
$query = "SELECT campaign.id, campaign.name, campaign.status FROM campaign ORDER BY campaign.id";

try {
    $stream = $googleAdsServiceClient->searchStream($customerId, $query);
    foreach ($stream->readAll() as $response) {
        foreach ($response->getResults() as $row) {
            printf("Campaign found: ID = %d, Name = '%s', Status = %s\n",
                $row->getCampaign()->getId(),
                $row->getCampaign()->getName(),
                $row->getCampaign()->getStatus()
            );
        }
    }
} catch (ApiException $apiException) {
    printf("Request failed with message: %s\n", $apiException->getMessage());
}
```

运行脚本（将 `XXXXXXXXXX` 替换为你的 10 位客户 ID）：
```bash
php get_campaigns.php XXXXXXXXXX
```