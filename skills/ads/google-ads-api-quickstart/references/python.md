# Google Ads API Python 配置参考

本指南介绍如何配置、安装并使用 Python 编写第一个 Google Ads API 调用。

## 前提条件

> [!IMPORTANT]
> **需要执行：动态解析 Python 版本**
> 在向用户展示这些前提条件之前，你**必须**动态解析最低要求的 Python 版本，以确保符合当前的支持周期：
> 1. 访问 [支持的客户端库版本](https://developers.google.com/google-ads/api/docs/client-libs.md.txt#supported_api_versions) 文档页面。
> 2. 在页面/表格中查找 **Python** 客户端库，确定最低支持的 Python 运行时版本（例如 Python 3.8+）。
> 3. 在最终输出给用户时，将下方 `MINIMUM_PYTHON_VERSION` 占位符替换为动态获取的版本。

*   **Python：** 版本 `MINIMUM_PYTHON_VERSION` 或更高。
*   **包管理器：** `pip`

---

## 第 1 步：环境与安装

> [!TIP]
> **最佳实践：** 始终在虚拟环境（`venv`）中安装客户端库，以避免与其他系统包的依赖冲突。对于在共享工作区中运行的自动化代理，这一点尤为重要。

### 1. 创建并激活虚拟环境
在项目根目录运行以下命令：
```bash
python3 -m venv .venv
source .venv/bin/activate
```

### 2. 安装库
安装官方 Google Ads Python 客户端库：
```bash
python -m pip install google-ads
```

---

## 第 2 步：配置（`google-ads.yaml` 及替代方法）

创建名为 `google-ads.yaml` 的文件。

Python 客户端库支持多种初始化 `GoogleAdsClient` 的方法：
1. **YAML 文件（`load_from_storage`）：** 按以下顺序解析：(1) 传给 `load_from_storage('/path/to/google-ads.yaml')` 的显式路径，(2) `GOOGLE_ADS_CONFIGURATION_FILE_PATH` 环境变量，或 (3) 默认的 `$HOME/google-ads.yaml`。
2. **环境变量（`load_from_env`）：** 读取大写 `GOOGLE_ADS_` 前缀的变量（例如 `GOOGLE_ADS_DEVELOPER_TOKEN`、`GOOGLE_ADS_CLIENT_ID`）。*（注意：如果设置了 `GOOGLE_ADS_CONFIGURATION_FILE_PATH`，`load_from_env` 将从该 YAML 文件加载）。*
3. **字典（`load_from_dict`）：** 直接接受包含凭据的 Python 字典。
4. **YAML 字符串（`load_from_string`）：** 接受内存中的原始 YAML 字符串。

> [!IMPORTANT]
> **封闭工作区规则：** 对于自包含的项目环境，最佳实践是将 `google-ads.yaml` 放在项目根目录并显式加载。

用你的凭据填充该文件：
```yaml
# google-ads.yaml
developer_token: INSERT_DEVELOPER_TOKEN_HERE
client_id: INSERT_OAUTH2_CLIENT_ID_HERE
client_secret: INSERT_OAUTH2_CLIENT_SECRET_HERE
refresh_token: INSERT_OAUTH2_REFRESH_TOKEN_HERE
# Optional: Un-comment if you are accessing a client account through a manager account
# login_customer_id: INSERT_LOGIN_CUSTOMER_ID_HERE
use_proto_plus: true
```

---

## 第 3 步：编写快速入门脚本

创建名为 `get_campaigns.py` 的文件。

> [!IMPORTANT]
> **规则 1：规范化客户 ID**
> 在将客户 ID 传给 API 之前，你**必须**去除所有连字符进行规范化（例如将 `123-456-7890` 转换为 `1234567890`）。下方脚本在入口点自动处理此操作。
>
> **规则 2：配置路径**
> 下方脚本配置为首先在**当前工作目录**中查找 `google-ads.yaml`，找不到则回退到环境变量或默认搜索路径。

```python
import argparse
import os
import sys
from google.ads.googleads.client import GoogleAdsClient
from google.ads.googleads.errors import GoogleAdsException

def main(client, customer_id):
    # Initialize the Google Ads Service
    googleads_service = client.get_service("GoogleAdsService")
    
    # Define the GAQL query
    query = "SELECT campaign.id, campaign.name, campaign.status FROM campaign ORDER BY campaign.id"
    
    print("Querying Google Ads API...")
    try:
        # Execute the search stream request
        stream = googleads_service.search_stream(customer_id=customer_id, query=query)
        for response in stream:
            for row in response.results:
                print(f"Campaign found: ID = {row.campaign.id}, Name = '{row.campaign.name}', Status = {row.campaign.status.name}")
                
    except GoogleAdsException as ex:
        print(f"Request ID '{ex.request_id}' failed with status '{ex.error.code().name}':")
        for error in ex.failure.errors:
            print(f"\tError: {error.message}")
        sys.exit(1)

if __name__ == '__main__':
    # Determine configuration file path (prefer local workspace config)
    local_config = os.path.join(os.getcwd(), "google-ads.yaml")
    
    if os.path.exists(local_config):
        # Load explicitly from local workspace
        googleads_client = GoogleAdsClient.load_from_storage(local_config)
    elif "GOOGLE_ADS_DEVELOPER_TOKEN" in os.environ:
        # Load from environment variables
        googleads_client = GoogleAdsClient.load_from_env()
    else:
        # Fallback to default search paths (GOOGLE_ADS_CONFIGURATION_FILE_PATH or $HOME/google-ads.yaml)
        googleads_client = GoogleAdsClient.load_from_storage()

    parser = argparse.ArgumentParser(description="Lists campaigns for a specified customer ID.")
    parser.add_argument("-c", "--customer_id", required=True, help="10-digit customer ID.")
    
    args = parser.parse_args()
    
    # Normalize customer ID by removing hyphens before passing to main
    normalized_customer_id = args.customer_id.replace("-", "")
    
    main(googleads_client, normalized_customer_id)
```

---

## 第 4 步：运行脚本

在终端中执行脚本，将客户 ID 作为参数传入。

> [!NOTE]
> 运行前确保虚拟环境已激活（`source .venv/bin/activate`）。

```bash
python get_campaigns.py -c XXXXXXXXXX
```
*（将 `XXXXXXXXXX` 替换为你的 10 位客户 ID，带或不带连字符均可）。*

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
| `FileNotFoundException` / `File not found` | 库找不到 `google-ads.yaml`。 | 确保文件名完全为 `google-ads.yaml`，并放在运行脚本的目录或 `$HOME` 目录中。 |
| `DEVELOPER_TOKEN_NOT_APPROVED` | 你正在对生产账号使用未批准（"Pending"）的开发者令牌（生产调用需要 Explorer、Basic 或 Standard 访问权限）。 | 使用**测试账号**（允许待批准的令牌）或等待令牌批准。 |
| `NOT_ADS_USER` | 用于生成刷新令牌的 OAuth2 用户凭据无权访问指定的 `-c/--customer_id`。 | 使用有权访问目标 Ads 账号的 Google 账号重新进行 OAuth2 认证。 |
| `ModuleNotFoundError: No module named 'google'` | Python 找不到已安装的库。 | 你可能在虚拟环境中安装了库，但使用全局 Python 解释器运行脚本。确保先运行 `source .venv/bin/activate`。 |