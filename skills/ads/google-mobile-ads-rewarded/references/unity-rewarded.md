# Unity 激励广告的 AI 集成代理说明

## 必需的导入

使用以下导入实现激励广告：

```c#
using System;
using UnityEngine;
using GoogleMobileAds.Api;
using GoogleMobileAds.Common;
```

## 激励广告工作流

1.  **加载广告**：

    使用静态方法 `RewardedAd.Load()`，传入广告单元 ID、
    `AdRequest` 对象和完成回调
    （`Action<RewardedAd, LoadAdError>`）：

    ```c#
    // 测试广告单元 ID。请替换为你自己的广告单元 ID。
    #if UNITY_ANDROID
        string adUnitId = "ca-app-pub-3940256099942544/5224354917";
    #elif UNITY_IPHONE
        string adUnitId = "ca-app-pub-3940256099942544/1712485313";
    #else
        string adUnitId = "unused";
    #endif
    ```

    ```c#
    RewardedAd.Load(adUnitId, new AdRequest(),
        (RewardedAd ad, LoadAdError error) =>
        {
            if (error != null || ad == null)
            {
                // 处理错误。
                return;
            }
            rewardedAd = ad;
        });
    ```

    加载新广告前，务必销毁已有的 `RewardedAd` 对象。

2.  **注册广告事件回调**：

    订阅 `RewardedAd` 对象上的广告事件是可选的，取决于
    你的应用需求：

    -   `OnAdPaid`：订阅以记录估算广告收入（`AdValue`），
        用于展示级别的广告变现跟踪。
    -   `OnAdImpressionRecorded`：订阅以在内部遥测或第三方分析平台
        跟踪广告展示。
    -   `OnAdClicked`：订阅以记录用户对广告的点击。
    -   `OnAdFullScreenContentOpened`：订阅以在激励广告覆盖屏幕时
        暂停游戏音频、计时器或游戏循环。
    -   `OnAdFullScreenContentClosed`：订阅以在广告关闭时恢复游戏音频或
        游戏循环。销毁 `RewardedAd` 对象
        （`rewardedAd.Destroy()`）并重新加载另一个广告（`RewardedAd.Load()`），
        以便新广告在下一个过渡点就绪。
    -   `OnAdFullScreenContentFailed`：订阅以在激励广告展示失败时
        记录展示错误。

    确保与 Unity 对象（`UnityEngine`）交互的后台广告回调
    通过 `MobileAdsEventExecutor.ExecuteInUpdate()` 显式调度到主线程，
    因为 Unity API 不是线程安全的，必须从主线程调用。

    ```c#
    rewardedAd.OnAdFullScreenContentOpened += () =>
    {
        MobileAdsEventExecutor.ExecuteInUpdate(() =>
        {
            // 在此处主线程上与 UnityEngine 对象交互。
        });
    };
    ```

3.  **添加用于观看广告获取奖励的 UI 元素**：

    **强制要求**：你必须明确提及或演示添加一个
    交互式 UI 元素（例如 Button）来触发 `ShowRewardedAd()`。
    激励广告绝不能自动展示。

4.  **展示广告**：

    检查 `RewardedAd` 对象上的 `CanShowAd()`，并向
    `Show()` 方法传入 `Action<Reward>` 回调。加入空值检查以
    避免异常。将此方法挂接到交互式 UI Button：

    ```c#
    // 从 UI Button 点击事件调用此方法：
    public void ShowRewardedAd()
    {
        if (rewardedAd != null && rewardedAd.CanShowAd())
        {
            rewardedAd.Show((Reward reward) =>
            {
                // 奖励用户。
                Debug.Log($"Rewarded! Amount: {reward.Amount}, Type: {reward.Type}");
            });
        }
    }
    ```

5.  **验证实现**：确保没有编译错误：
    -   [ ] 验证 Unity Console 中没有 C# 脚本编译错误或
        缺少 GoogleMobileAds 命名空间引用。

### 链接

其他文档：

-   https://developers.google.com/admob/unity/rewarded.md.txt