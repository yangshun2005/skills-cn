---
name: ima-dai-sdk
description: >-
  将 Google Interactive Media Ads (IMA) Dynamic Ad Insertion (DAI) SDK 集成到网站、Web 应用、移动应用或电视应用中。
  适用场景：
  - 视频播放器需要在 Web 应用、Android 应用、iOS 应用、tvOS 应用、Cast (CAF) 接收器或 Roku 频道中加载并播放 HLS 或 DASH 流。
  - 应用需要使用 Google DAI 直播活动资产键，或点播内容源 CMS ID、视频 ID。
  不要用此技能加载并播放 VAST 或 VMAP URL。
license: Apache-2.0
metadata:
  author: Google LLC
  version: "1.0.0"
  category: GoogleAds
---

# IMA DAI SDK

使用 IMA DAI SDK 将 HLS 或 DASH 流加载到应用中，用于：

*   在 Google Ad Manager 中配置的**直播活动**。
*   导入 Google Ad Manager 的**点播 (VOD)** 内容。

## 前提条件

查阅目标平台对应的集成指南：

*   **Web/HTML5/ReactJs/NodeJs/Angular：** 阅读
    [StreamManager 指南](references/web-StreamManager-guide.md)，了解如何将 Google 全服务 DAI 的流 URL 加载到 `<video>` 元素。

*   **ChromeCast：** 阅读
    [StreamManager 指南](references/cast-StreamManager-guide.md)，了解如何将 IMA DAI SDK 集成到 ChromeCast Web 接收器。

*   **Android：** 阅读
    [ImaServerSideAdInsertionMediaSource 指南](references/android-ImaServerSideAdInsertionMediaSource-guide.md)，
    了解如何集成 Media3 Exoplayer IMA 扩展。

*   **iOS/tvOS：** 阅读
    [IMAStreamRequest 指南](references/ios-IMAStreamRequest-guide.md)，了解如何用 `AVPlayer` 播放流。

*   **Roku：** 阅读 [StreamManager 指南](references/roku-StreamManager-guide.md)，
    了解如何在 Roku SceneGraph 上实现 DAI。

## 快速开始（通用流程）

1.  导入 SDK
2.  初始化 SDK
3.  添加流事件监听器
4.  设置定时元数据转发
5.  发起流请求
6.  流失败或用户离开流时，清理 SDK 资源。