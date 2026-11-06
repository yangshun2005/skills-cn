# Unity 横幅广告 AI 集成代理说明

## 必需的导入

使用以下导入实现横幅广告：

```c#
using System;
using UnityEngine;
using GoogleMobileAds.Api;
using GoogleMobileAds.Common;
```

## 横幅广告工作流

1.  **定义广告视图**：

    使用 `GoogleMobileAds.Api` 中的 `BannerView` 类定义横幅广告。创建广告视图时设置合适的测试广告单元 ID：

    -   Android 锚定自适应：`ca-app-pub-3940256099942544/9214589741`
    -   iOS 锚定自适应：`ca-app-pub-3940256099942544/2435281174`

2.  **设置广告尺寸**：

    将 `AdSize.Banner` 或合适的自适应广告尺寸连同 `AdPosition`（`AdPosition.Top` 或 `AdPosition.Bottom`）传给 `BannerView` 构造函数。

    对于锚定自适应横幅，除非用户投诉或需要较小高度，否则使用大型锚定自适应广告尺寸。

    ```c#
    int width = MobileAds.Utils.GetDeviceSafeWidth();
    AdSize adSize = AdSize
        .GetCurrentOrientationLargeAnchoredAdaptiveBannerAdSizeWithWidth(width);
    BannerView bannerView =
        new BannerView(_adUnitId, adSize, AdPosition.Bottom);
    ```

3.  **注册广告加载事件**：

    订阅 `BannerView` 对象上的广告事件是可选的，取决于应用需求：

    -   `OnBannerAdLoaded`：订阅以在横幅广告就绪并展示时更新 UI 状态。
    -   `OnBannerAdLoadFailed`：订阅以在横幅广告加载失败时记录错误。
    -   `OnAdPaid`：订阅以记录估算广告收入（`AdValue`），用于展示级广告变现跟踪。
    -   `OnAdImpressionRecorded`：订阅以在内部遥测或第三方分析平台中跟踪广告展示。
    -   `OnAdClicked`：订阅以记录用户对广告的点击。
    -   `OnAdFullScreenContentOpened`：订阅以在横幅点击打开全屏目标内容时暂停游戏音频、计时器或游戏循环。
    -   `OnAdFullScreenContentClosed`：订阅以在全屏目标内容关闭时恢复游戏音频或游戏循环。

    **关键**：无论何时提供 Unity 广告的代码或说明，你都必须明确说明：与 Unity 对象（`UnityEngine`）交互的后台广告回调必须使用 `MobileAdsEventExecutor.ExecuteInUpdate()` 调度到主线程，因为 Unity API 不是线程安全的。

    ```c#
    bannerView.OnBannerAdLoaded += () =>
    {
        MobileAdsEventExecutor.ExecuteInUpdate(() =>
        {
            // Interact with UnityEngine objects on the main thread here.
        });
    };
    ```

4.  **加载横幅广告**：

    在 `BannerView` 对象上调用 `LoadAd()` 方法，传入新的 `AdRequest` 对象（`bannerView.LoadAd(new AdRequest())`）。

    ```c#
    bannerView.LoadAd(new AdRequest());
    ```

5.  **验证实现**：

    运行 `dotnet build` 编译 Unity csproj 文件。解决任何与 GMA SDK 相关的编译错误。

### 链接

附加文档：

-   https://developers.google.com/admob/unity/banner.md.txt