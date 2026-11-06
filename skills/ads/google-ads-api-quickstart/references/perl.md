# Google Ads API Perl 设置参考

本指南介绍如何配置、安装并使用 Perl 编写第一个 Google Ads API 调用。

## 前提条件
* Perl：需从[支持的客户端库版本](https://developers.google.com/google-ads/api/docs/client-libs.md.txt#supported_api_versions)动态查找所需版本
* CPAN 客户端：`cpanm`

## 第 1 步：安装
安装库：
```bash
cpanm Google::Ads::GoogleAds::Client
```

## 第 2 步：配置（`googleads.properties`）
创建名为 `googleads.properties` 的配置文件。

通过 `Client->new()` 实例化客户端时，Perl 客户端库按以下顺序解析配置文件路径：
1. 直接传给 `Client->new({properties_file => '/path/to/googleads.properties'})` 的显式路径。
2. 若已设置 `GOOGLE_ADS_CONFIGURATION_FILE_PATH` 环境变量，则使用该变量指定的位置。
3. 否则使用用户主目录（`$HOME`）下的默认 `googleads.properties` 文件。

对于自包含的项目环境，最佳实践是将此文件放在项目根目录并显式加载。

```properties
# googleads.properties
developer_token=INSERT_DEVELOPER_TOKEN_HERE
client_id=INSERT_OAUTH2_CLIENT_ID_HERE
client_secret=INSERT_OAUTH2_CLIENT_SECRET_HERE
refresh_token=INSERT_OAUTH2_REFRESH_TOKEN_HERE
# login_customer_id=INSERT_LOGIN_CUSTOMER_ID_HERE
```

## 第 3 步：编写并运行快速入门脚本
创建 `get_campaigns.pl`：
```perl
use strict;
use warnings;
use Google::Ads::GoogleAds::Client;
use Google::Ads::GoogleAds::Utils::SearchStreamHandler;

my $customer_id = $ARGV[0];
$customer_id =~ s/-//g;

# Determine configuration file path (prefer local workspace config)
my $local_config = "./googleads.properties";
my $client;

if (-e $local_config) {
    # Load explicitly from local workspace
    $client = Google::Ads::GoogleAds::Client->new({properties_file => $local_config});
} else {
    # Fallback to default search paths (GOOGLE_ADS_CONFIGURATION_FILE_PATH environment variable or $HOME/googleads.properties)
    $client = Google::Ads::GoogleAds::Client->new();
}

# Load any overriding configuration from environment variables
$client->configure_from_environment_variables();

my $query = "SELECT campaign.id, campaign.name, campaign.status FROM campaign ORDER BY campaign.id";

my $search_stream_handler = Google::Ads::GoogleAds::Utils::SearchStreamHandler->new({
    client => $client,
    customer_id => $customer_id,
    query => $query
});

$search_stream_handler->process(sub {
    my $row = shift;
    printf "Campaign found: ID = %s, Name = '%s', Status = %s\n",
        $row->{campaign}{id},
        $row->{campaign}{name},
        $row->{campaign}{status};
});
```

运行脚本（将 `XXXXXXXXXX` 替换为你的 10 位客户 ID）：
```bash
perl get_campaigns.pl XXXXXXXXXX
```