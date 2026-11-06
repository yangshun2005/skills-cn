# Google IMA DAI SDK iOS/tvOS - IMAStreamRequest 指南

Google 全服务 DAI 中，用 `IMALiveStreamRequest` 请求直播流，用 `IMAVODStreamRequest` 请求 VOD 流。

## 目录

*   [依赖](#dependencies)（第 19 行）
    *   [使用 Swift Package Manager](#use-swift-package-manager)（第 21 行）
*   [SDK 初始化](#sdk-initialization)（第 35 行）
*   [UI 设置与视频播放器](#ui-setup--video-player)（第 51 行）
*   [发起流请求](#make-stream-requests)（第 68 行）
    *   [请求直播流](#request-a-livestream)（第 72 行）
    *   [请求 VOD 流](#request-a-vod-stream)（第 92 行）
*   [监听流加载事件](#listen-to-stream-load-events)（第 116 行）
*   [监听流事件与广告事件](#listen-to-stream-and-ad-events)（第 119 行）
*   [参考实现](#reference-implementation)（第 182 行）

## 依赖

### 使用 Swift Package Manager

iOS 添加此包：

```
https://github.com/googleads/swift-package-manager-google-interactive-media-ads-ios
```

tvOS 添加此包：

```
https://github.com/googleads/swift-package-manager-google-interactive-media-ads-tvos
```

## SDK 初始化

尽早初始化 `IMAAdsLoader`，例如在 `ViewController.viewDidLoad` 事件中。

```swift
import GoogleInteractiveMediaAds

class ViewController {
  override func viewDidLoad() {
    super.viewDidLoad()
    adsLoader = IMAAdsLoader(settings: IMASettings())
  }
}
```

## UI 设置与视频播放器

用包含 `AVPlayer` 的 `videoView` 创建 `IMAAdDisplayContainer` 对象。广告 UI 组件将渲染在 `videoView` 内、播放器之上。

```swift
adDisplayContainer = IMAAdDisplayContainer(adContainer: videoView)
```

用 `AVPlayer` 实例创建 `IMAAVPlayerVideoDisplay`，以便 IMA DAI SDK 在广告时段监听视频播放器的定时元数据。

```swift
imaVideoDisplay = IMAAVPlayerVideoDisplay(avPlayer: videoPlayer)
```

## 发起流请求

复用同一个 `AdsLoader` 实例发起流请求。

### 请求直播流

用资产键和网络代码实例化 `IMALiveStreamRequest`：

```swift
let request = IMALiveStreamRequest(
  assetKey: <ASSET_KEY_PLACEHOLDER>,
  networkCode: <NETWORK_CODE_PLACEHOLDER>,
  adDisplayContainer: adDisplayContainer,
  videoDisplay: imaVideoDisplay,
  userContext: nil)
adsLoader.requestStream(with: request)
```

提供以下参数：

*   **<NETWORK_CODE_PLACEHOLDER>**：Google Ad Manager 网络代码。
*   **<ASSET_KEY_PLACEHOLDER>**：在 Google Ad Manager 中配置的直播流资产键。

### 请求 VOD 流

用内容源 ID（CMS ID）和视频 ID 实例化 `IMAVODStreamRequest`：

```swift
let request = IMAVODStreamRequest(
  contentSourceID: <CONTENT_SOURCE_ID_PLACEHOLDER>,
  videoID: <VIDEO_ID_PLACEHOLDER>,
  networkCode: <NETWORK_CODE_PLACEHOLDER>,
  adDisplayContainer: adDisplayContainer,
  videoDisplay: imaVideoDisplay,
  userContext: nil)
adsLoader.requestStream(with: request)
```

提供以下参数：

*   **<NETWORK_CODE_PLACEHOLDER>**：Google Ad Manager 网络代码。
*   **<CONTENT_SOURCE_ID_PLACEHOLDER>**：Google Ad Manager 中的 CMS ID。
*   **<VIDEO_ID_PLACEHOLDER>**：CMS 中的视频 ID。

测试时，可使用 https://developers.google.com/ad-manager/dynamic-ad-insertion/streams.md.txt?utm_source=agent-skills&utm_medium=content&utm_campaign=adr-ss-ai&utm_content=ima-dai-sdk 中的 DAI 示例流值。

## 监听流加载事件

实现 `IMAAdsLoaderDelegate` 处理流初始化或失败：

```swift
func adsLoader(_ loader: IMAAdsLoader, adsLoadedWith adsLoadedData: IMAAdsLoadedData) {
  print("DAI session ID: \(adsLoadedData.streamManager!.streamId!)")
  adsLoadedData.streamManager.initialize(with: nil)
}

func adsLoader(_ loader: IMAAdsLoader, failedWith adErrorData: IMAAdLoadingErrorData) {
  print("Failed to load DAI stream. Error: \(adErrorData.adError.message ?? "")")
  playBackupStream()
}
```

## 监听流事件与广告事件

实现 `IMAStreamManagerDelegate` 接收流事件、广告生命周期事件和错误：

```swift
func streamManager(_ streamManager: IMAStreamManager, didReceive event: IMAAdEvent) {
  print("Ad event: \(event.typeString)")
  switch event.type {
  case IMAAdEventType.STARTED:
    if let ad = event.ad {
      let extendedAdPodInfo = String(
        format: "Showing ad %zd/%zd, bumper: %@, title: %@, "
          +   "description: %@, contentType:%@, pod index: %zd, "
          +   "time offset: %lf, max duration: %lf.",
        ad.adPodInfo.adPosition,
        ad.adPodInfo.totalAds,
        ad.adPodInfo.isBumper ? "YES" : "NO",
        ad.adTitle,
        ad.adDescription,
        ad.contentType,
        ad.adPodInfo.podIndex,
        ad.adPodInfo.timeOffset,
        ad.adPodInfo.maxDuration)
      print("\(extendedAdPodInfo)")
    }
    break
  case IMAAdEventType.AD_BREAK_STARTED:
    print("Ad break started.")
    break
  case IMAAdEventType.AD_BREAK_ENDED:
    print("Ad break ended.")
    break
  case IMAAdEventType.AD_PERIOD_STARTED:
    print("Ad period started.")
    break
  case IMAAdEventType.AD_PERIOD_ENDED:
    print("Ad period ended.")
    break
  default:
    break
  }
}

func streamManager(_ streamManager: IMAStreamManager, didReceive error: IMAAdError) {
  print("Failed to play DAI stream. Error: \(error.message ?? "Unknown Error")")
  playBackupStream()
}
```

## 参考实现

BasicExample：

*   [iPhone.storyboard](https://raw.githubusercontent.com/googleads/googleads-ima-ios-dai/refs/heads/main/Swift/BasicExample/app/iPhone.storyboard)
*   [ViewController.swift](https://raw.githubusercontent.com/googleads/googleads-ima-ios-dai/refs/heads/main/Swift/BasicExample/app/ViewController.swift)