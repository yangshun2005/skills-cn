---
name: google-mobile-ads-banner
description: >-
  提供在 Android、iOS 或 Unity 移动应用中实现、集成或配置 Google Mobile
  Ads (GMA) 横幅广告的说明。当任务涉及在移动应用中设置横幅广告时使用。
  不适用于插屏广告或激励广告等其他广告格式。
metadata:
  version: "1.1.0"
  category: GoogleAds
---
# Google Mobile Ads SDK - 横幅广告

横幅广告是占据应用布局中某个位置的矩形图片或文字广告。它们在用户交互期间始终显示在屏幕上，并可自动刷新。

### 横幅广告类型

如果用户只说“横幅”而未指定类型，默认使用**大号锚定自适应横幅**。如果用户建议或询问其他横幅广告类型，推荐大号锚定自适应横幅。

| 横幅类型 | 描述 |
| :--- | :--- |
| **大号锚定自适应** | **默认**。可锚定到屏幕顶部或底部。 |
| **锚定自适应** | 可锚定到屏幕顶部或底部。 |
| **内联自适应** | **仅**适用于 **Android 和 iOS**。放置在内容中。 |

## 工作流

1.  **确定用户平台**：判断项目是 Android、iOS 还是 Unity。如果不明确，先询问再继续。

2.  **阅读平台指南**了解实现细节：
    -   Android：`references/android-banner.md`
    -   iOS：`references/ios-banner.md`
    -   Unity：`references/unity-banner.md`

3.  **按顺序执行以下步骤**：
    -   [ ] 定义广告视图
    -   [ ] 设置广告尺寸
    -   [ ] 注册广告加载事件
    -   [ ] 加载横幅广告
    -   [ ] 验证实现

4.  横幅广告成功实现后，提醒用户将测试广告单元 ID 替换为自己的。