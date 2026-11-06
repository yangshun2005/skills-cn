# Google Ads API Ruby 配置参考

本指南介绍如何配置、安装并使用 Ruby 编写第一个 Google Ads API 调用。

## 前置条件
* Ruby：需从[支持的客户端库版本](https://developers.google.com/google-ads/api/docs/client-libs.md.txt#supported_api_versions)动态查询
* 包管理器：RubyGems

## 第 1 步：安装
开始使用客户端库前，**你必须安装 gem：`gem install google-ads-googleads`。** 直接在终端运行：
```bash
gem install google-ads-googleads
```

## 第 2 步：配置（`google_ads_config.rb`）

> [!IMPORTANT]
> **代理强制指令（配置加载顺序）：**
> 指导用户完成 Ruby 快速入门时，你的回复**必须**明确说明 `GoogleAdsClient.new` 使用的三级配置加载顺序：
> 1. 直接传给构造函数的显式路径：`GoogleAdsClient.new('/path/to/google_ads_config.rb')`。
> 2. `GOOGLE_ADS_CONFIGURATION_FILE_PATH` 环境变量指定的位置（若已设置）。
> 3. 否则使用用户主目录（`$HOME`）下的默认 `google_ads_config.rb` 文件。
>
> 最终输出中不得省略此说明。

创建配置文件 `google_ads_config.rb`。对于自包含的项目环境，最佳实践是将该文件放在项目根目录并显式加载。

```ruby
# google_ads_config.rb
Google::Ads::GoogleAds::Config.new do |c|
  c.developer_token = 'INSERT_DEVELOPER_TOKEN_HERE'
  c.client_id = 'INSERT_OAUTH2_CLIENT_ID_HERE'
  c.client_secret = 'INSERT_OAUTH2_CLIENT_SECRET_HERE'
  c.refresh_token = 'INSERT_OAUTH2_REFRESH_TOKEN_HERE'
  # Optional: Required if authenticating with a manager account user to access a client account
  # c.login_customer_id = 'INSERT_LOGIN_CUSTOMER_ID_HERE'
end
```

## 第 3 步：编写并运行快速入门脚本
创建 `get_campaigns.rb`。**将客户 ID 传给 API 前，必须先清理客户 ID，去除连字符（例如将 `123-456-7890` 转为 `1234567890`）。**

以下是清理 ID 并获取广告系列详情的完整、自包含 Ruby 源码：

```ruby
require 'google/ads/google_ads'

if ARGV.empty?
  puts "Usage: ruby get_campaigns.rb <CUSTOMER_ID>"
  exit 1
end

# Clean customer ID by removing hyphens
customer_id = ARGV[0].gsub('-', '')

# Determine configuration file path (prefer local workspace config)
local_config = File.expand_path('google_ads_config.rb', __dir__)

client = if File.exist?(local_config)
           # Load explicitly from local workspace
           Google::Ads::GoogleAds::GoogleAdsClient.new(local_config)
         else
           # Fallback to default search paths (GOOGLE_ADS_CONFIGURATION_FILE_PATH environment variable or $HOME/google_ads_config.rb)
           Google::Ads::GoogleAds::GoogleAdsClient.new
         end

query = "SELECT campaign.id, campaign.name, campaign.status FROM campaign ORDER BY campaign.id"

begin
  response = client.service.google_ads.search_stream(
    customer_id: customer_id,
    query: query,
  )
  response.each do |page|
    page.results.each do |row|
      puts "Campaign found: ID = #{row.campaign.id}, Name = '#{row.campaign.name}', Status = #{row.campaign.status}"
    end
  end
rescue Google::Ads::GoogleAds::Errors::GoogleAdsError => e
  e.failure.errors.each do |error|
    puts "Error: #{error.message}"
  end
end
```

运行脚本（将 `XXXXXXXXXX` 替换为你的 10 位客户 ID）：
```bash
ruby get_campaigns.rb XXXXXXXXXX
```