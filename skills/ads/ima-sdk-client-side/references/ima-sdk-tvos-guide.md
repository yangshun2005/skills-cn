# Google IMA SDK tvOS 集成指南

Google IMA SDK for tvOS 与 iOS SDK 高度一致，共用相同的 API 和集成流程。

**继续之前，你必须阅读
[Google IMA SDK iOS 集成指南](ima-sdk-ios-guide.md)，了解完整的
分步生命周期流程（初始化 -> 广告请求 -> 广告加载 -> 播放
-> 清理）。**

本文档列出你必须实现的关键差异和 tvOS 特有要求。

--------------------------------------------------------------------------------

## 与 iOS 的关键差异

### 1. 导入 SDK

默认使用 Swift Package Manager 添加
https://github.com/googleads/swift-package-manager-google-interactive-media-ads-tvos 的 main 分支。

如果应用必须使用 CocoaPods，请安装 `GoogleAds-IMA-tvOS-SDK` pod。

### 2. tvOS 自动播放

在应用中添加 tvOS 平台检查，并以自动播放方式发起广告请求。

### 3. 焦点管理（tvOS 特有）

要处理 Siri Remote，你必须管理焦点：

*   **广告 UI 焦点：** IMA SDK 会在需要时自动管理"跳过"按钮及其他广告 UI 视图的焦点。确保应用的焦点引擎不拦截或覆盖焦点变更，否则用户将无法聚焦并点击跳过按钮。

*   **安全区域：** 确保广告 UI 容器遵循 tvOS 安全区域布局指南，防止广告浮层或跳过按钮被电视过扫描裁切。

### 4. 遥控手势（tvOS 特有）

防止用户交互干扰广告播放（例如快进跳过广告）：

*   **禁用手势：** 广告开始时（`LOADED` 或 `STARTED` 事件），你**必须**禁用应用的自定义遥控手势识别器（如播放/暂停、滑动或菜单按钮覆盖）。
*   **恢复手势：** 仅在广告完成或发生致命错误后，通过 `adsManagerDidRequestContentResume` 委托方法重新启用手势。

--------------------------------------------------------------------------------

## 代码实现差异

主要 `AdsManager` 和 `PlayerViewController` 的实现请参考 [iOS 指南](ima-sdk-ios-guide.md)。按以下方式调整 tvOS 实现：

### 广告 UI 设置（安全区域）

在 `ViewController` 中，将广告容器对齐到安全区域布局指南，防止电视过扫描裁切：

```swift
func setupAdContainer(in viewController: UIViewController, overlaying videoView: UIView) -> UIView {
    let adContainerView = UIView()
    adContainerView.translatesAutoresizingMaskIntoConstraints = false
    viewController.view.addSubview(adContainerView)

    // tvOS Specific: Align with safe area to prevent overscan clipping
    let safeArea = viewController.view.safeAreaLayoutGuide
    NSLayoutConstraint.activate([
        adContainerView.leadingAnchor.constraint(equalTo: safeArea.leadingAnchor),
        adContainerView.trailingAnchor.constraint(equalTo: safeArea.trailingAnchor),
        adContainerView.topAnchor.constraint(equalTo: safeArea.topAnchor),
        adContainerView.bottomAnchor.constraint(equalTo: safeArea.bottomAnchor)
    ])

    return adContainerView
}
```

### 焦点管理（首选焦点环境）

为确保 Siri Remote 能聚焦 SDK 的交互元素（如"跳过"按钮），你必须在 `IMAAdEvent.Type.STARTED` 事件中调用 `setNeedsFocusUpdate()` 函数，并覆盖 `ViewController` 对象的 `preferredFocusEnvironments` 属性。

广告播放结束后（如 `IMAAdEvent.Type.COMPLETE` 或 `IMAAdEvent.Type.SKIPPED` 事件），你必须恢复标准焦点规则。

```swift
import UIKit
import GoogleInteractiveMediaAds

class YourViewController: UIViewController, IMAAdsManagerDelegate {

    // Tracks state to determine who gets focus
    var isAdPlaying: Bool = false
    var adDisplayContainer: IMAAdDisplayContainer?

    // MARK: - Focus Management

    override var preferredFocusEnvironments: [UIFocusEnvironment] {
        // Check if an ad is playing and if the IMA SDK has a valid focus environment
        if isAdPlaying, let adFocusEnvironment = adDisplayContainer?.focusEnvironment {
            // Hand focus routing over to the IMA SDK UI (e.g., "Skip" button)
            return [adFocusEnvironment]
        }

        // Otherwise, use the app's standard focus rules (e.g., video player controls)
        return super.preferredFocusEnvironments
    }

    // MARK: - IMAAdsManagerDelegate Methods

    func adsManager(_ adsManager: IMAAdsManager, didReceive event: IMAAdEvent) {
        switch event.type {
        case .LOADED:
            // tvOS Specific: Disable custom remote gestures during ad
            disableAppGestures()
            adsManager.start()

        case .STARTED:
            // Update state and force tvOS to move focus to the ad
            isAdPlaying = true
            setNeedsFocusUpdate()

        case .COMPLETE, .SKIPPED, .ALL_ADS_COMPLETED:
            // Update state and reclaim focus back to the app
            isAdPlaying = false
            setNeedsFocusUpdate()

        default:
            break
        }
    }

    func adsManagerDidRequestContentResume(_ adsManager: IMAAdsManager) {
        // tvOS Specific: Restore gestures when ad break finishes
        enableAppGestures()
        resumeContent()
    }

    func adsManager(_ adsManager: IMAAdsManager, failedWith error: IMAAdError) {
        // Safety check: Reset focus state just in case an ad fails mid-playback
        isAdPlaying = false
        setNeedsFocusUpdate()

        // tvOS Specific: Restore gestures on failure
        enableAppGestures()
        cleanupAds()
        resumeContent()
    }

    // MARK: - App-Specific Helpers (Implementation depends on your app)

    private func disableAppGestures() { /* ... */ }
    private func enableAppGestures() { /* ... */ }
    private func resumeContent() { /* ... */ }
    private func cleanupAds() { /* ... */ }
}
```