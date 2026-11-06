# Google IMA DAI SDK Cast - StreamManager 指南

使用 `StreamManager` 在 Cast Application Framework（CAF）Web Receiver 中请求并播放 Google 全服务 DAI 的直播流或 VOD 流。

## 目录

*   [导入 IMA DAI SDK](#import-the-ima-dai-sdk)（第 17 行）
*   [SDK 初始化](#sdk-initialization)（第 28 行）
*   [发起流请求](#make-stream-requests)（第 43 行）
*   [流事件与错误处理](#stream-event-and-error-handling)
    （第 95 行）
    *   [DAI 会话事件](#dai-session-events)（第 103 行）
    *   [广告插播事件](#ad-break-events)（第 108 行）
    *   [广告事件](#ad-events)（第 114 行）
    *   [清理](#cleanup)（第 124 行）

## 导入 IMA DAI SDK

在 CAF receiver 框架脚本加载器之后，紧接着添加 CAF 版 IMA DAI SDK 的 script 标签。CAF DAI SDK 为常青版本，无需指定版本号。

```html
<script src="//www.gstatic.com/cast/sdk/libs/caf_receiver/v3/cast_receiver_framework.js"></script>
<script src="//imasdk.googleapis.com/js/sdkloader/cast_dai.js"></script>
```

## SDK 初始化

获取 `CastReceiverContext` 和 `PlayerManager` 实例。

初始化 `StreamManager`，以便 IMA DAI SDK 在播放开始前设置定时元数据事件监听器。

```typescript
const castContext: cast.framework.CastReceiverContext =
    cast.framework.CastReceiverContext.getInstance();
const playerManager: cast.framework.PlayerManager =
    castContext.getPlayerManager();
const streamManager = new google.ima.cast.dai.api.StreamManager();
```

## 发起流请求

根据发送端应用的 cast 请求创建 `google.ima.cast.dai.api.LiveStreamRequest`。

```typescript
let streamRequest = new google.ima.cast.dai.api.LiveStreamRequest();
streamRequest.networkCode = <NETWORK_CODE_PLACEHOLDER>;
streamRequest.assetKey = <ASSET_KEY_PLACEHOLDER>;
```

从 `cast.framework.messages.MediaInformation.customData` 中提取以下参数：

*   **<NETWORK_CODE_PLACEHOLDER>**：Google Ad Manager 网络代码。
*   **<ASSET_KEY_PLACEHOLDER>**：在 Google Ad Manager 中配置的直播流资产键。

对于 VOD，根据发送端应用的 cast 请求创建 `google.ima.cast.dai.api.VODStreamRequest`。

```typescript
let streamRequest = new google.ima.cast.dai.api.VODStreamRequest();
streamRequest.networkCode = <NETWORK_CODE_PLACEHOLDER>;
streamRequest.contentSourceId = <CMS_ID_PLACEHOLDER>;
streamRequest.videoId = <VIDEO_ID_PLACEHOLDER>;
```

从 `cast.framework.messages.MediaInformation.customData` 中提供以下参数：

*   **<NETWORK_CODE_PLACEHOLDER>**：Google Ad Manager 网络代码。
*   **<CMS_ID_PLACEHOLDER>**：Google Ad Manager 中的内容源 ID（CMS ID）。
*   **<VIDEO_ID_PLACEHOLDER>**：CMS 中的视频 ID。

测试时，可使用 https://developers.google.com/ad-manager/dynamic-ad-insertion/streams.md.txt?utm_source=agent-skills&utm_medium=content&utm_campaign=adr-ss-ai&utm_content=ima-dai-sdk 中的 DAI 示例流值。

要立即播放流，创建 load 消息拦截器，通过调用 `streamManager.requestStream()` 发起流请求：

```typescript
playerManager.setMessageInterceptor(
  cast.framework.messages.MessageType.LOAD,
  (loadRequestData: cast.framework.messages.LoadRequestData) => {
    return streamManager.requestStream(loadRequestData, streamRequest);
    }
  );
```

## 流事件与错误处理

在 `google.ima.cast.dai.api.StreamManager` 上为所有事件类型添加监听器：

*   DAI 会话事件与错误
*   广告插播事件
*   单条广告事件

### DAI 会话事件

监听 `google.ima.cast.dai.api.StreamEvent.Type.ERROR` 事件，记录 `event.getStreamData().errorMessage` 字符串并切换到备用流。

### 广告插播事件

监听 `google.ima.cast.dai.api.StreamEvent.Type.AD_PERIOD_STARTED` 和 `google.ima.cast.dai.api.StreamEvent.Type.AD_PERIOD_ENDED` 事件，以禁用和恢复播放控件（如跳转）。

### 广告事件

监听以下单条广告事件并记录日志：

*   `google.ima.cast.dai.api.StreamEvent.Type.STARTED`
*   `google.ima.cast.dai.api.StreamEvent.Type.FIRST_QUARTILE`
*   `google.ima.cast.dai.api.StreamEvent.Type.MIDPOINT`
*   `google.ima.cast.dai.api.StreamEvent.Type.THIRD_QUARTILE`
*   `google.ima.cast.dai.api.StreamEvent.Type.COMPLETE`

### 清理

流结束或发生致命错误时，重置 `StreamManager`。

```typescript
streamManager.reset();
```