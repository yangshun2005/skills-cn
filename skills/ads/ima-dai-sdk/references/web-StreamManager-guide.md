# Google IMA DAI SDK HTML5（Web）集成指南

本指南介绍如何在桌面浏览器或移动浏览器中集成 Google IMA DAI（动态广告插入）SDK 以运行 Web 应用。

## 目录

*   [集成流程](#integration-flow)（第 24 行）
    *   [导入 SDK](#import-the-sdk)（第 26 行）
    *   [初始化](#initialization)（第 35 行）
*   [请求 DAI 流](#request-dai-streams)（第 55 行）
    *   [请求直播流](#request-a-livestream)（第 57 行）
    *   [请求 VOD 流](#request-a-vod-stream)（第 86 行）
*   [定时元数据转发](#timed-metadata-forwarding)（第 121 行）
    *   [传递 HLS 清单中的原始 ID3 帧](#passing-raw-id3-frames-from-hls-manifest)
        （第 126 行）
    *   [传递 DASH 清单中的自定义事件数据](#passing-custom-event-data-from-dash-manifest)
        （第 145 行）
*   [流事件与错误处理](#stream-event-and-error-handling)
    （第 165 行）
    *   [DAI 会话事件](#dai-session-events)（第 173 行）
    *   [广告插播事件](#ad-break-events)（第 181 行）
    *   [广告事件](#ad-events)（第 187 行）
    *   [清理](#cleanup)（第 201 行）
*   [参考实现](#reference-implementation)（第 209 行）

## 集成流程

### 导入 SDK

在页面级别加载 IMA DAI SDK 脚本加载器。确保 SDK 能访问 `window.top.location.href`。

```html
<script src="https://imasdk.googleapis.com/js/sdkloader/ima3_dai.js"></script>
```

### 初始化

尽早实例化 `StreamManager`。它需要 HTML video 元素，以及一个覆盖在 video 元素上、用于承载广告 UI 元素的 HTML `div` 元素。

```html
<div id="player-container">
  <video id="video-element" controls></video>
  <div id="ad-ui-element"></div>
</div>
```

```typescript
const videoElement = document.getElementById('video-element') as HTMLVideoElement;
const adUiElement = document.getElementById('ad-ui-element') as HTMLElement;

// Instantiate the StreamManager
const streamManager = new google.ima.dai.api.StreamManager(videoElement, adUiElement);
```

## 请求 DAI 流

### 请求直播流

实例化 `google.ima.dai.api.LiveStreamRequest`，以默认 HLS M3U8 格式请求 Google DAI 线性流。

```typescript
const hlsLiveStreamRequest = new google.ima.dai.api.LiveStreamRequest();
hlsLiveStreamRequest.assetKey = <ASSET_KEY_PLACEHOLDER>;
hlsLiveStreamRequest.networkCode = <NETWORK_CODE_PLACEHOLDER>;
streamManager.requestStream(hlsLiveStreamRequest);
```

要以 DASH MPD 格式请求 Google DAI 线性流，需显式设置 `LiveStreamRequest.format` 属性：

```typescript
const dashLiveStreamRequest = new google.ima.dai.api.LiveStreamRequest();
dashLiveStreamRequest.assetKey = <ASSET_KEY_PLACEHOLDER>;
dashLiveStreamRequest.networkCode = <NETWORK_CODE_PLACEHOLDER>;
dashLiveStreamRequest.format = 'dash';
streamManager.requestStream(dashLiveStreamRequest);
```

提供以下参数：

*   **<NETWORK_CODE_PLACEHOLDER>**：Google Ad Manager 网络代码。
*   **<ASSET_KEY_PLACEHOLDER>**：在 Google Ad Manager 中配置的直播流资产键。

### 请求 VOD 流

实例化 `google.ima.dai.api.VODStreamRequest`，以默认 HLS M3U8 格式请求 Google DAI VOD 流。

```typescript
const hlsVodStreamRequest = new google.ima.dai.api.VODStreamRequest();
hlsVodStreamRequest.networkCode = <NETWORK_CODE_PLACEHOLDER>;
hlsVodStreamRequest.contentSourceId = <CONTENT_SOURCE_ID_PLACEHOLDER>;
hlsVodStreamRequest.videoId = <VIDEO_ID_PLACEHOLDER>;
streamManager.requestStream(hlsVodStreamRequest);
```

要以 DASH MPD 格式请求 Google DAI VOD 流，设置 `VODStreamRequest.format` 属性：

```typescript
const dashVodStreamRequest = new google.ima.dai.api.VODStreamRequest();
dashVodStreamRequest.networkCode = <NETWORK_CODE_PLACEHOLDER>;
dashVodStreamRequest.contentSourceId = <CONTENT_SOURCE_ID_PLACEHOLDER>;
dashVodStreamRequest.videoId = <VIDEO_ID_PLACEHOLDER>;
dashVodStreamRequest.format = 'dash';
streamManager.requestStream(dashVodStreamRequest);
```

提供以下必需参数：

*   **<NETWORK_CODE_PLACEHOLDER>**：Google Ad Manager 网络代码。
*   **<CONTENT_SOURCE_ID_PLACEHOLDER>**：Google Ad Manager 中的内容源 ID（CMS ID）。
*   **<VIDEO_ID_PLACEHOLDER>**：CMS 中的视频 ID。

测试时，可使用 https://developers.google.com/ad-manager/dynamic-ad-insertion/streams.md.txt?utm_source=agent-skills&utm_medium=content&utm_campaign=adr-ss-ai&utm_content=ima-dai-sdk 中的 DAI 示例流值。

## 定时元数据转发

要让 SDK 触发广告事件，需监听视频播放器事件，提取定时元数据并传给 SDK 立即处理。

### 传递 HLS 清单中的原始 ID3 帧

捕获嵌入的 ID3 元数据并传给 SDK 处理。

如果使用 HLS.js，监听 `FRAG_PARSING_METADATA` 事件。

```typescript
import Hls, { FragParsingMetadataData } from 'hls.js';

hls.on(Hls.Events.FRAG_PARSING_METADATA, (event: Events.FRAG_PARSING_METADATA, data: FragParsingMetadataData): void => {
  // Iterate over each parsed metadata sample
  data.samples.forEach((sample) => {
    // sample.data: Uint8Array containing raw ID3 data
    // sample.pts: Presentation timestamp for precise synchronization
    streamManager.processMetadata('ID3', sample.data, sample.pts);
  });
});
```

### 传递 DASH 清单中的自定义事件数据

如果使用 DASH.js，监听由 Google DAI scheme ID 标识的 DASH 自定义事件，并将事件载荷数据传给 SDK 处理。

```typescript
interface DashEventPayload {
  event: {
    messageData: Uint8Array;
    calculatedPresentationTime: number;
  };
}

dashPlayer.on('urn:google:dai:2018', (payload: DashEventPayload): void => {
    const mediaId = payload.event.messageData;
    const pts = payload.event.calculatedPresentationTime;
    streamManager.processMetadata('urn:google:dai:2018', mediaId, pts);
  });
```

## 流事件与错误处理

在 `google.ima.dai.api.StreamManager` 上为所有事件类型添加监听器：

*   DAI 会话事件与错误
*   广告插播事件
*   单个广告事件

### DAI 会话事件

监听 `google.ima.dai.api.StreamEvent.Type.LOADED` 事件，获取 `event.getStreamData().url` 字符串并传给视频播放器。

监听 `google.ima.dai.api.StreamEvent.Type.ERROR` 事件，记录 `event.getStreamData().errorMessage` 字符串并切换到备用流。

### 广告插播事件

监听 `google.ima.dai.api.StreamEvent.Type.AD_PERIOD_STARTED` 和 `google.ima.dai.api.StreamEvent.Type.AD_PERIOD_ENDED` 事件，以禁用和恢复播放控件（如跳转）。

### 广告事件

监听以下单个广告事件并记录日志：

*   `google.ima.dai.api.StreamEvent.Type.STARTED`
*   `google.ima.dai.api.StreamEvent.Type.FIRST_QUARTILE`
*   `google.ima.dai.api.StreamEvent.Type.MIDPOINT`
*   `google.ima.dai.api.StreamEvent.Type.THIRD_QUARTILE`
*   `google.ima.dai.api.StreamEvent.Type.COMPLETE`
*   `google.ima.dai.api.StreamEvent.Type.CLICK`：此点击跳转事件发生时，SDK 会暂停广告播放并将用户带到广告落地页。确保应用 UI 已准备好，以便用户从广告落地页返回后恢复广告播放。

### 清理

流结束或发生致命错误时，重置 `StreamManager`。

```typescript
streamManager.reset();
```

## 参考实现

HLS.js 集成：

*   [HTML 页面](https://raw.githubusercontent.com/googleads/googleads-ima-html5-dai/refs/heads/main/hls_js/simple/dai.html)
*   [应用逻辑](https://raw.githubusercontent.com/googleads/googleads-ima-html5-dai/refs/heads/main/hls_js/simple/dai.js)

DASH.js 集成：

*   [HTML 页面](https://raw.githubusercontent.com/googleads/googleads-ima-html5-dai/refs/heads/main/dash_js/simple/dai.html)
*   [应用逻辑](https://raw.githubusercontent.com/googleads/googleads-ima-html5-dai/refs/heads/main/dash_js/simple/dai.js)