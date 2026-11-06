---
name: google-mobile-ads-rewarded
description: >-
  提供在 Android、iOS 或 Unity 移动应用中实现、集成或配置 Google Mobile Ads
  (GMA) SDK 激励广告的说明。任务涉及设置激励广告时使用。不要用于"激励插页式"广告。
metadata:
  version: "1.1.0"
  category: GoogleAds
---
# Google Mobile Ads SDK - 激励广告

激励广告通过让用户与全屏广告互动来奖励其应用内物品。激励广告仅在用户明确选择观看后才会投放。

## 工作流程

1.  **确定用户平台**：判断项目是 Android、iOS 还是 Unity。如不明确，先询问再继续。

2.  **阅读平台指南**了解实现细节：
    -   Android：`references/android-rewarded.md`
    -   iOS：`references/ios-rewarded.md`
    -   Unity：`references/unity-rewarded.md`

3.  **按顺序执行以下步骤**：
    -   [ ] 加载广告
    -   [ ] 注册广告事件回调
    -   [ ] 添加用于观看广告获取奖励的 UI 元素
    -   [ ] 展示广告
    -   [ ] 验证实现