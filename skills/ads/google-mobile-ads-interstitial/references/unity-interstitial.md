# Unity 插页式广告 AI 集成代理说明

## 必需的导入

使用以下导入实现插页式广告：

```c#
using System;
using UnityEngine;
using GoogleMobileAds.Api;
using GoogleMobileAds.Common;
```

## 插页式广告工作流

1.  **加载广告**：

    调用静态方法 `InterstitialAd.Load()`，传入广告单元 ID、`AdRequest` 对象和完成回调（`Action<InterstitialAd, LoadAdError>`）：

    ```c#
    // 测试广告单元 ID。请替换为你自己的广告单元 ID。
    #if UNITY_ANDROID
        string adUnitId = "ca-app-pub-3940256099942544/1033173712";
    #elif UNITY_IPHONE
        string adUnitId = "ca-app-pub-3940256099942544/4411468910";
    #else
        string adUnitId = "unused";
    #endif
    ```

    ```c#
    InterstitialAd.Load(adUnitId, new AdRequest(),
        (InterstitialAd ad, LoadAdError error) =>
        {
            if (error != null || ad == null)
            {
                // 处理错误。
                return;
            }
            interstitialAd = ad;
        });
    ```

    加载新广告前，务必销毁已有的 `InterstitialAd` 对象。

2.  **注册广告事件回调**：

    在 `InterstitialAd` 对象上订阅广告事件是可选的，取决于你的应用需求：

    -   `OnAdPaid`：订阅以记录估算广告收入（`AdValue`），用于展示级广告变现追踪。
    -   `OnAdImpressionRecorded`：订阅以在内部遥测或第三方分析平台中追踪广告展示。
    -   `OnAdClicked`：订阅以记录用户对广告的点击。
    -   `OnAdFullScreenContentOpened`：订阅以在插页式广告覆盖屏幕时暂停游戏音频、计时器或游戏循环。
    -   `OnAdFullScreenContentClosed`：订阅以在广告关闭时恢复游戏音频或游戏循环。销毁 `InterstitialAd` 对象（`interstitialAd.Destroy()`）并重新加载广告（`InterstitialAd.Load()`），以便下一个过渡点有新的广告就绪。
    -   `OnAdFullScreenContentFailed`：订阅以在插页式广告展示失败时记录展示错误。

    确保与 Unity 对象（`UnityEngine`）交互的后台广告回调通过 `MobileAdsEventExecutor.ExecuteInUpdate()` 显式调度到主线程，因为 Unity API 不是线程安全的，必须从主线程调用。

    ```c#
    interstitialAd.OnAdFullScreenContentOpened += () =>
    {
        MobileAdsEventExecutor.ExecuteInUpdate(() =>
        {
            // 在此处从主线程与 UnityEngine 对象交互。
        });
    };
    ```

3.  **展示广告**：

    调用 `Show()` 方法前，先检查 `InterstitialAd` 对象上的 `CanShowAd()`。加入空值检查以避免异常。

    ```c#
    if (interstitialAd != null && interstitialAd.CanShowAd())
    {
        interstitialAd.Show();
    }
    ```

4.  **验证实现**：确保没有编译错误：
    -   [ ] 验证 Unity 控制台没有 C# 脚本编译错误或缺失 GoogleMobileAds 命名空间引用。

### 链接

附加文档：

-   https://developers.google.com/admob/unity/interstitial.md.txt