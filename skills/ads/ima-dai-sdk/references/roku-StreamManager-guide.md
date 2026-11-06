# Google IMA DAI SDK Roku - StreamManager 指南

对于 Google 全服务 DAI，使用 `CreateLiveStreamRequest` 请求直播流，使用 `CreateVodStreamRequest` 请求 VOD 流。

## 目录

*   [导入 IMA DAI SDK](#import-the-ima-dai-sdk)（第 21 行）
*   [SDK 初始化](#sdk-initialization)（第 43 行）
*   [视频播放器设置](#video-player-setup)（第 54 行）
*   [发起流请求](#make-stream-requests)（第 87 行）
    *   [请求直播流](#request-a-livestream)（第 89 行）
    *   [请求 VOD 流](#request-a-vod-stream)（第 105 行）
    *   [执行流请求](#execute-stream-request)（第 126 行）
*   [开始流播放](#start-stream-playback)（第 153 行）
*   [定时元数据转发](#timed-metadata-forwarding)（第 177 行）
*   [监听广告事件](#listen-to-ad-events)（第 205 行）
    *   [可跳过广告支持](#skippable-ads-support)（第 244 行）
*   [参考实现](#reference-implementation)（第 256 行）

## 导入 IMA DAI SDK

将所需库添加到 `manifest`：

```
bs_libs_required=roku_ads_lib,googleima3
```

创建 Task 组件 `components/Sdk.xml`，用于加载库并在后台线程运行 SDK：

```xml
<component name="imasdk" extends="Task">
  <script type="text/brightscript">
    <![CDATA[
      Library "Roku_Ads.brs"
      Library "IMA3.brs"
    ]]>
  </script>
</component>
```

## SDK 初始化

使用 `New_IMASDK()` 初始化 IMA SDK 实例：

```brightscript
if m.sdk = invalid
  m.sdk = New_IMASDK()
  m.sdk.initSdk()
end if
```

## 视频播放器设置

使用 `sdk.createPlayer()` 创建播放器实例。

```brightscript
m.player = m.sdk.createPlayer()
m.player.top = m.top
```

创建广告插播回调：

```brightscript
m.player.loadUrl = Function(urlData)
  m.top.video.enableTrickPlay = false
  m.top.urlData = urlData
End Function

m.player.adBreakStarted = Function(adBreakInfo as Object)
  m.top.adPlaying = true
  m.top.video.enableTrickPlay = false
End Function

m.player.adBreakEnded = Function(adBreakInfo as Object)
  m.top.adPlaying = false
  m.top.video.enableTrickPlay = true
End Function

m.player.seek = Function(timeSeconds as Double)
  m.top.video.seekMode = "accurate"
  m.top.video.seek = timeSeconds
End Function
```

## 发起流请求

### 请求直播流

```brightscript
streamRequest = m.sdk.CreateLiveStreamRequest(
  <ASSET_KEY_PLACEHOLDER>,
  "", // 如果应用使用 Google DAI API 密钥，请将空字符串替换为该密钥
  <NETWORK_CODE_PLACEHOLDER>
)
```

提供以下参数：

*   **<NETWORK_CODE_PLACEHOLDER>**：Google Ad Manager 网络代码。
*   **<ASSET_KEY_PLACEHOLDER>**：在 Google Ad Manager 中配置的直播流资产键。

### 请求 VOD 流

```brightscript
streamRequest = m.sdk.CreateVodStreamRequest(
  <CONTENT_SOURCE_ID_PLACEHOLDER>,
  <VIDEO_ID_PLACEHOLDER>,
  "", // 如果应用使用 Google DAI API 密钥，请将空字符串替换为该密钥
  <NETWORK_CODE_PLACEHOLDER>
)
```

提供以下参数：

*   **<NETWORK_CODE_PLACEHOLDER>**：Google Ad Manager 网络代码。
*   **<CONTENT_SOURCE_ID_PLACEHOLDER>**：Google Ad Manager 中的内容源 ID（CMS ID）。
*   **<VIDEO_ID_PLACEHOLDER>**：CMS 中的视频 ID。

测试时，请使用 https://developers.google.com/ad-manager/dynamic-ad-insertion/streams.md.txt?utm_source=agent-skills&utm_medium=content&utm_campaign=adr-ss-ai&utm_content=ima-dai-sdk 中的 DAI 示例流值。

### 执行流请求

将播放器对象和视频节点引用（`adUiNode`）传给请求。

```brightscript
streamRequest.player = m.player
streamRequest.adUiNode = m.top.findNode("myVideo")
requestResult = m.sdk.requestStream(streamRequest)
If requestResult <> Invalid
  print "Error requesting stream ";requestResult
Else
  m.streamManager = Invalid
  While m.streamManager = Invalid
    sleep(50)
    m.streamManager = m.sdk.getStreamManager()
  End While
  If m.streamManager = Invalid or (m.streamManager["type"] <> Invalid and m.streamManager["type"] = "error")
    errors = CreateObject("roArray", 1, True)
    print "error ";m.streamManager["info"]
    errors.push(m.streamManager["info"])
    m.top.errors = errors
  Else
    m.streamManager.start()
  End If

```

## 开始流播放

在 `MainScene.xml` 中监听流清单数据，并将其传给 `Video` 节点：

```brightscript
m.sdkTask.observeField("urlData", "urlLoadRequested")
' 将 control 设为 RUN 会启动任务线程。
m.sdkTask.control = "RUN"

Sub urlLoadRequested(message as Object)
  data = message.getData()
  vidContent = createObject("RoSGNode", "ContentNode")
  vidContent.url = data.manifest
  vidContent.title = m.videoTitle
  vidContent.streamformat = data.format
  m.video.content = vidContent
  m.video.setFocus(true)
  m.video.visible = true
  m.video.control = "play"
  m.video.EnableCookies()
End Sub
```

## 定时元数据转发

在流播放期间，将所有定时元数据和视频节点事件转发给 `StreamManager.onMessage(msg)`：

```brightscript
m.top.video.timedMetaDataSelectionKeys = ["*"]

m.port = CreateObject("roMessagePort")
fields = m.top.video.getFields()
for each field in fields
  m.top.video.observeField(field, m.port)
end for

while true
  msg = wait(1000, m.port)
  if m.top.video = invalid
    exit while
  end if

  m.streamManager.onMessage(msg)
  currentTime = m.top.video.position
  if currentTime > 3 and not m.top.adPlaying
    m.top.video.enableTrickPlay = true
  end if
end while
```

## 监听广告事件

在 `m.streamManager` 上注册事件监听器，以监控广告生命周期和错误事件：

```brightscript
m.streamManager.addEventListener(m.sdk.AdEvent.ERROR, errorCallback)
m.streamManager.addEventListener(m.sdk.AdEvent.START, startCallback)
m.streamManager.addEventListener(m.sdk.AdEvent.FIRST_QUARTILE, firstQuartileCallback)
m.streamManager.addEventListener(m.sdk.AdEvent.MIDPOINT, midpointCallback)
m.streamManager.addEventListener(m.sdk.AdEvent.THIRD_QUARTILE, thirdQuartileCallback)
m.streamManager.addEventListener(m.sdk.AdEvent.COMPLETE, completeCallback)

Function startCallback(ad as Object) as Void
  print "Ad event: START"
End Function

Function firstQuartileCallback(ad as Object) as Void
  print "Ad event: FIRST_QUARTILE"
End Function

Function midpointCallback(ad as Object) as Void
  print "Ad event: MIDPOINT"
End Function

Function thirdQuartileCallback(ad as Object) as Void
  print "Ad event: THIRD_QUARTILE"
End Function

Function completeCallback(ad as Object) as Void
  print "Ad event: COMPLETE"
End Function

Function errorCallback(error as Object) as Void
  print "Ad event: ERROR - "; error
  m.errorState = true
End Function
```

### 可跳过广告支持

要支持可跳过广告，请在 `m.player` 上实现 `seek` 回调方法，并确保流请求中的 `streamRequest.adUiNode` 已设置为视频节点：

```brightscript
m.player.seek = Function(timeSeconds as Double)
  m.top.video.seekMode = "accurate"
  m.top.video.seek = timeSeconds
End Function
```

## 参考实现

BasicExample：

*   [Sdk.xml](https://raw.githubusercontent.com/googleads/googleads-ima-roku-dai/refs/heads/main/basic_example/components/Sdk.xml)
*   [MainScene.xml](https://raw.githubusercontent.com/googleads/googleads-ima-roku-dai/refs/heads/main/basic_example/components/MainScene.xml)