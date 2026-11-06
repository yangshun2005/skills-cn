# 格式化用户数据

*   获取 [格式化用户数据](https://developers.google.com/data-manager/api/devguides/concepts/formatting.md.txt) 指南，将其作为格式化和规范化规则的权威依据。

*   使用工具库对用户数据（邮箱、电话号码、地址）进行格式化、哈希和加密。

    **Python 示例：**

    ```python
    from google.ads.datamanager_util import Formatter
    from google.ads.datamanager_util.format import Encoding

    formatter: Formatter = Formatter()

    processed_email: str = formatter.process_email_address(
        email, Encoding.HEX
    )
    ```