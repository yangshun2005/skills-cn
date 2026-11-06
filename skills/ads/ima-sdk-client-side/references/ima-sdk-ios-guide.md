# Google IMA SDK iOS 集成指南

本指南按广告播放的生命周期，介绍 Google IMA SDK for iOS（客户端）的集成方法。

## 集成流程

### 1. 导入 SDK

默认使用 Swift Package Manager 添加
https://github.com/googleads/swift-package-manager-google-interactive-media-ads-ios 的主分支。

如果应用必须使用 CocoaPods，请安装 `GoogleAds-IMA-iOS-SDK` pod。

### 2. 初始化

在应用生命周期早期配置 SDK 设置并初始化 `IMAAdsLoader`，以降低延迟，同时设置广告 UI。

*   **尽早初始化：** 创建 `IMAAdsLoader` 实例开销较大，因为会启动底层 WebView（1-2 秒开销）。应在早期（如应用启动时）实例化加载器，并复用该单一实例。
*   **设置不可变：** 在传给加载器之前配置好 `IMASettings`。加载器初始化后，设置将变为只读。
*   **创建 IMAAdsLoader：** 传入已配置的 `IMASettings` 实例来实例化 `IMAAdsLoader`。该对象管理广告请求的生命周期，必须持久保存并复用。
*   **广告 UI 设置：** 为广告创建专用 `UIView`，用 Auto Layout 直接叠加在视频播放器视图之上。广告播放时隐藏自定义播放器控件。

```swift
import UIKit
import GoogleInteractiveMediaAds

// 1. Shared AdsManager to handle early initialization and reuse
class AdsManager: NSObject {
    static let shared = AdsManager()

    var adsLoader: IMAAdsLoader?
    var adsManager: IMAAdsManager?
    private var settings: IMASettings

    private override init() {
        // Configure settings early
        settings = IMASettings()
        settings.language = "en"
        settings.enableDebugMode = true

        super.init()

        // Initialize loader early. Settings are now locked.
        adsLoader = IMAAdsLoader(settings: settings)
    }

    // 2. Ad UI Setup helper
    func setupAdContainer(in viewController: UIViewController, overlaying videoView: UIView) -> UIView {
        let adContainerView = UIView()
        adContainerView.translatesAutoresizingMaskIntoConstraints = false
        viewController.view.addSubview(adContainerView)

        // Align perfectly with the video view
        NSLayoutConstraint.activate([
            adContainerView.leadingAnchor.constraint(equalTo: videoView.leadingAnchor),
            adContainerView.trailingAnchor.constraint(equalTo: videoView.trailingAnchor),
            adContainerView.topAnchor.constraint(equalTo: videoView.topAnchor),
            adContainerView.bottomAnchor.constraint(equalTo: videoView.bottomAnchor)
        ])

        return adContainerView
    }
}
```

### 3. 广告请求

创建 `IMAAdDisplayContainer` 和 `IMAAdsRequest`，然后触发请求。强烈建议在用户手势（如点击播放）时触发。

```swift
extension AdsManager {
    func requestAds(adTagUrl: String, adContainer: UIView, videoDisplay: IMAVideoDisplay, delegate: IMAAdsLoaderDelegate) {
        guard let loader = adsLoader else { return }
        loader.delegate = delegate

        // Create the ad display container
        let displayContainer = IMAAdDisplayContainer(adContainerViewController: delegate as? UIViewController, companionViews: nil)

        // Create the ads request
        let request = IMAAdsRequest(
            adTagUrl: adTagUrl,
            adDisplayContainer: displayContainer,
            contentPlayhead: nil,
            userContext: nil)

        // Request ads
        loader.requestAds(with: request)
    }
}
```

### 4. 广告加载成功/失败

实现 `IMAAdsLoaderDelegate`，处理广告加载成功（返回 `IMAAdsManager`）或早期失败（致命加载错误）。

```swift
class PlayerViewController: UIViewController, IMAAdsLoaderDelegate {
    var videoView: UIView! // Your video player view
    var adContainerView: UIView?

    func startAdFlow() {
        // Set up UI and request ads
        self.adContainerView = AdsManager.shared.setupAdContainer(in: self, overlaying: videoView)

        let videoDisplay = IMAAVPlayerVideoDisplay(avPlayer: self.contentPlayer)
        AdsManager.shared.requestAds(
            adTagUrl: "YOUR_AD_TAG_URL",
            adContainer: self.adContainerView!,
            videoDisplay: videoDisplay,
            delegate: self)
    }

    // MARK: - IMAAdsLoaderDelegate (Success)
    func adsLoader(_ loader: IMAAdsLoader, admitsCompletedWith adsManagerLoadedData: IMAAdsManagerLoadedData) {
        // Ad Load Success: Get the AdsManager
        AdsManager.shared.adsManager = adsManagerLoadedData.adsManager
        AdsManager.shared.adsManager?.delegate = self

        // Initialize the ads manager
        AdsManager.shared.adsManager?.initialize(with: nil)
    }

    // MARK: - IMAAdsLoaderDelegate (Failure / Fatal Load Error)
    func adsLoader(_ loader: IMAAdsLoader, failedWith adErrorData: IMAAdLoadingErrorData) {
        print("IMA SDK Loading Error: \(adErrorData.adError.message ?? "Unknown error")")
        resumeContent() // Fallback to content
    }
}
```

### 5. 广告播放事件

实现 `IMAAdsManagerDelegate`，监听播放事件、协调内容暂停/恢复，并处理播放错误。

*   **暂停内容：** 收到 `pause` 事件时，暂停内容播放器并隐藏自定义控件。
*   **恢复内容：** 收到 `resume` 事件或 `adsManagerDidRequestContentResume` 时，恢复控件并播放内容。
*   **非致命日志：** 监听 `log` 事件，处理静默跟踪或 VPAID 问题，不中断播放。

```swift
extension PlayerViewController: IMAAdsManagerDelegate {

    // MARK: - IMAAdsManagerDelegate (Playback Events)
    func adsManager(_ adsManager: IMAAdsManager, didReceive event: IMAAdEvent) {
        switch event.type {
        case .LOADED:
            // Start ad playback
            adsManager.start()
        case .PAUSE:
            pauseContent() // Pause player, hide custom controls
        case .RESUME:
            resumeContent() // Resume player, restore controls
        case .LOG:
            if let adData = event.adData {
                print("IMA SDK Non-fatal Log: \(adData)")
            }
        default:
            break
        }
    }

    func adsManagerDidRequestContentResume(_ adsManager: IMAAdsManager) {
        resumeContent() // Resume content when ad completes
    }

    // MARK: - IMAAdsManagerDelegate (Playback Failure / Fatal Error)
    func adsManager(_ adsManager: IMAAdsManager, failedWith error: IMAAdError) {
        print("IMA SDK Manager Error: \(error.message ?? "Unknown error")")
        cleanupAds()
        resumeContent()
    }
}
```

### 6. 清理

正确的清理**至关重要**，可防止内存泄漏、音频播放 bug 和后台资源消耗。

*   **`IMAAdsManager.destroy()`：** 这一步很关键。以下情况务必调用，并将引用置为 `nil`：
    *   所有广告已播放完毕。
    *   发生致命广告错误（在 `failedWith` 委托中）。
    *   用户关闭播放器或离开页面（如 `viewWillDisappear` 或 `deinit` 中）。
*   **`IMAAdsLoader` 清理：** `IMAAdsLoader` 设计为长生命周期对象。不要在广告请求之间销毁它。但如果需要彻底拆除广告集成（如应用关闭或主要父组件被释放），请将加载器的 delegate 置为 `nil`，并将引用置空，以便 ARC 释放它。

```swift
// In your PlayerViewController
deinit {
    cleanupAds()
}

override func viewWillDisappear(_ animated: Bool) {
    super.viewWillDisappear(animated)
    if isMovingFromParent {
        cleanupAds()
    }
}

func cleanupAds() {
    // 1. Destroy the AdsManager and nil its delegate
    if let manager = AdsManager.shared.adsManager {
        manager.destroy()
        manager.delegate = nil
        AdsManager.shared.adsManager = nil
    }

    // 2. Clean up UI
    self.adContainerView?.removeFromSuperview()
    self.adContainerView = nil
}

// Call this only when permanently tearing down the SDK integration
func tearDownSDK() {
    AdsManager.shared.adsLoader?.delegate = nil
    AdsManager.shared.adsLoader = nil
}
```

--------------------------------------------------------------------------------

## 参考实现

*   BasicExample [BasicExampleApp.swift](https://raw.githubusercontent.com/googleads/googleads-ima-ios/main/Swift/BasicExample/BasicExample/BasicExampleApp.swift)
*   BasicExample [PlayerContainerViewController.swift](https://raw.githubusercontent.com/googleads/googleads-ima-ios/main/Swift/BasicExample/BasicExample/PlayerContainerViewController.swift)