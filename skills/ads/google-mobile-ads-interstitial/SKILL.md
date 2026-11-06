---
name: google-mobile-ads-interstitial
description: >-
  提供在 Android、iOS 或 Unity 移动应用中实现、集成或配置 Google Mobile Ads
  (GMA) SDK 插页式广告的说明。当任务涉及设置插页式广告时使用。不适用于
  "激励插页式"广告。
metadata:
  version: "1.1.0"
  category: GoogleAds
---
# Google Mobile Ads SDK - 插页式广告

插页式广告为移动应用用户展示全屏广告。插页式广告设计用于放置在内容之间，最适合放在应用自然的过渡点。

## 工作流程

1.  **确定用户平台**：确认项目是 Android、iOS 还是 Unity。如果不明确，先询问再继续。

2.  **阅读平台指南**了解实现细节：
    -   Android：`references/android-interstitial.md`
    -   iOS：`references/ios-interstitial.md`
    -   Unity：`references/unity-interstitial.md`

3.  **按顺序执行以下步骤**：
    -   [ ] 加载广告
    -   [ ] 注册广告事件回调
    -   [ ] 展示广告
    -   [ ] 验证实现