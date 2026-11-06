# Unity Google Mobile Ads 插件 AI 集成代理说明

## SDK 集成工作流

1.  **添加 SDK 依赖**：

    通过 `Packages/manifest.json` 安装：

    -   运行以下命令获取最新版本。返回的标签（去掉 'v' 前缀）即 Google Mobile Ads SDK 的最新版本：

        ```bash
        curl -sS https://api.github.com/repos/googleads/googleads-mobile-unity/releases/latest | jq -r '.tag_name'
        ```

    -   使用最新版本，在 `Packages/manifest.json` 中添加 "OpenUPM" 作用域注册表。

        ```json
        {
          "scopedRegistries": [
            {
              "name": "OpenUPM",
              "url": "https://package.openupm.com",
              "scopes": [
                "com.google"
              ]
            }
          ],
          "dependencies": {
            // 将 11.2.0 替换为最新稳定版本。
            "com.google.ads.mobile": "11.2.0"
          }
        }
        ```

2.  **设置应用 ID**：

    修改 `Assets/GoogleMobileAds/Resources/GoogleMobileAdsSettings.asset`，将 `adMobAndroidAppId` 和 `adMobIOSAppId` 设为示例 App ID：

    -   Android：`ca-app-pub-3940256099942544~3347511713`
    -   iOS：`ca-app-pub-3940256099942544~1458002511`

    提醒用户在发布前替换为实际的 AdMob App ID。

3.  **初始化 SDK**：

    编写脚本处理初始化。确保与 Unity 对象（`UnityEngine`）交互的后台广告回调通过 `ExecuteInUpdate()` 显式调度到主线程。

    不要将 `MobileAds.RaiseAdEventsOnUnityMainThread` 设为 `true`，该属性已弃用。

    ```c#
    using GoogleMobileAds.Api;
    using GoogleMobileAds.Common;
    using UnityEngine;

    public class GoogleMobileAdsController : MonoBehaviour
    {
        public void Start()
        {
            // 仅初始化 Google Mobile Ads SDK 一次。
            MobileAds.Initialize((InitializationStatus initStatus) =>
            {
                Debug.Log("Google Mobile Ads SDK initialized.");

                MobileAdsEventExecutor.ExecuteInUpdate(() =>
                {
                    // 在此处主线程上与 UnityEngine 对象交互。
                });
            });
        }
    }
    ```

4.  **验证集成**：

    运行 `dotnet build` 编译 Unity csproj 文件。解决所有与 GMA SDK 相关的编译错误。

### 链接

其他文档和指南：

-   快速入门指南：
    https://developers.google.com/admob/unity/quick-start.md.txt
-   全局设置指南：
    https://developers.google.com/admob/unity/global-settings.md.txt
-   OpenUPM 包：https://openupm.com/packages/com.google.ads.mobile/