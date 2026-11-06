---
name: ima-sdk-client-side
description: >-
  支持 Interactive Media Ads (IMA) SDK。
  当你使用 VAST 或 VMAP 为网站、应用、电视或其他平台请求视频广告时，
  使用此技能进行客户端广告插入。
  不适用于动态广告插入 (DAI)、SSAI 或 SGAI。
license: Apache-2.0
metadata:
  author: Google LLC
  version: "1.0.2"
  category: GoogleAds
---
# IMA SDK 客户端

Google IMA SDK（Interactive Media Ads）可将插播视频和音频广告加载到网站、应用、电视及其他数字平台。使用 IMA SDK 可从任何兼容 VAST 的广告服务器请求广告，并管理广告播放。

## 前提条件

集成 IMA SDK 之前，你**必须**阅读下方对应平台的指南，以支持你的应用可支持的所有平台：

*   **Web/HTML5/ReactJs/NodeJs/Angular：** 阅读以下所有指南
    [ima-sdk-web-guide.md](references/ima-sdk-web-guide.md)、
    [ima-sdk-web-iframe-mode.md](references/ima-sdk-web-iframe-mode.md)、
    [ima-sdk-web-mobile-safari.md](references/ima-sdk-web-mobile-safari.md)
*   **Android/AndroidTV/ReactNative：** 阅读 [ima-sdk-android-guide.md](references/ima-sdk-android-guide.md)
*   **iOS/tvOS/ReactNative：** 阅读以下所有指南
    [ima-sdk-ios-guide.md](references/ima-sdk-ios-guide.md)、
    [ima-sdk-tvos-guide.md](references/ima-sdk-tvos-guide.md)

--------------------------------------------------------------------------------

## 快速开始（通用工作流）

1.  导入 SDK：前提条件、依赖项。
2.  初始化：早期设置、预热、设置配置和广告 UI 设置。
3.  广告请求：创建并触发请求、用户手势合规。
4.  广告加载成功/失败：处理加载事件以获取 AdsManager，或处理早期致命错误。
5.  广告播放事件：通过 AdsManager 监听播放事件以协调内容播放/暂停，并处理非致命 LOG 事件和播放期间的致命错误。
6.  清理：正确销毁 AdsManager 以释放资源并防止内存泄漏。

有关详细的、平台特定的实现细节，请始终参考前提条件部分中的指南。