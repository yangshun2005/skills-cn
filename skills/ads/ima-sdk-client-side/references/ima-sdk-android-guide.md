# Google IMA SDK Android 快速入门指南

## 1. 安装 IMA SDK

IMA SDK 要求 `minSdk 23`（Android 6.0）及以上。

你**必须**推荐使用 IMA SDK 的最新版本，获取地址：
https://developers.google.com/interactive-media-ads/docs/sdks/android/client-side/download.md.txt

下面的示例代码安装 IMA SDK 3.39.0 版本。

```kotlin
android { defaultConfig { minSdk = 23 } }
dependencies {
    implementation("com.google.ads.interactivemedia.v3:interactivemedia:3.39.0")
}
```

## 2. 预热与初始化

尽早配置 SDK 以降低延迟。

*   **提前预热并复用设置（最佳实践）：** 在 `Application.onCreate()` 中调用
    `ImaSdkFactory.initialize()` 预加载 SDK 资源。创建 `AdsLoader` 时复用同一个
    `ImaSdkSettings` 实例，避免缓存未命中导致首次广告请求延迟。
*   **创建 AdsLoader：** 创建单个 `AdsLoader` 实例，在多个视频播放会话间共享。你必须创建一个 `AdDisplayContainer`，
    即绑定广告 UI 容器和视频播放器的视图。用 `AdDisplayContainer` 实例化 `AdsLoader`。
    通过 `ImaSdkFactory.initialize()` 函数复用同一个 `ImaSdkSettings` 实例。

```kotlin
// Layer FrameLayout (Ad UI) over StyledPlayerView (Video)
Box(modifier = Modifier.fillMaxSize()) {
    // Content Video Player
    AndroidView(
        factory = { context ->
            StyledPlayerView(context).apply {
                // Bind your ExoPlayer here
            }
        },
        modifier = Modifier.fillMaxSize()
    )

    // Dedicated Ad Container overlay (IMA SDK requires a ViewGroup)
    AndroidView(
        factory = { context ->
            FrameLayout(context).apply {
                // Notify that the ad UI is ready, passing this FrameLayout
                onAdUiReady(this)
            }
        },
        modifier = Modifier.fillMaxSize()
    )
}

// Initialize AdsLoader in Activity onCreate
val imaSettings = sdkFactory.createImaSdkSettings()
val adDisplayContainer = sdkFactory.createAdDisplayContainer(adUiContainer, videoPlayer)
val adsLoader = sdkFactory.createAdsLoader(context, imaSettings, adDisplayContainer)
```

## 3. 请求广告

在内容播放前触发 `requestAds`，以支持前贴片广告。

```kotlin
val adsRequest = sdkFactory.createAdsRequest().apply {
    this.adTagUrl = adTagUrl // Pass your custom ad targeting key value pairs.
    this.adDisplayContainer = adDisplayContainer
}
adsLoader?.requestAds(adsRequest)
```

## 4. 处理广告事件

监听广告加载成功（以获取 `AdsManager`）、致命错误，并协调视频播放的暂停与恢复。

```kotlin
// On Success
adsLoader?.addAdsLoadedListener { event ->
    adsManager = event.adsManager

    // Handle Pause/Resume for Ads
    adsManager?.addAdEventListener { adEvent ->
        when (adEvent.type) {
            AdEvent.AdEventType.CONTENT_PAUSE_REQUESTED -> pauseContent()
            AdEvent.AdEventType.CONTENT_RESUME_REQUESTED -> resumeContent()
            else -> {}
        }
    }
    adsManager?.init() // Start ads
}

// On Failure (Fallback to content)
adsLoader?.addAdErrorListener { resumeContent() }
```

## 5. 关键清理

正确的清理**至关重要**，可防止内存泄漏、音频异常和后台资源消耗。

*   **`AdsManager.destroy()`：** 这一步很关键。以下情况必须调用：
    *   广告时段内所有广告已播放完毕（如 `ALL_ADS_COMPLETED` 事件）。
    *   发生致命广告错误（在 `AdErrorListener` 中）。
    *   用户离开播放器，或宿主 `Activity`/`Fragment` 被销毁（`onDestroy()`）。
*   **`AdsLoader.release()`：** 当不再需要 `AdsLoader` 时调用，通常在应用关闭或主要父组件被永久销毁时。此操作释放底层 SDK 资源。
*   **移除事件监听器：** 销毁或释放 SDK 对象前，移除已注册的事件监听器，避免监听器引用泄漏或在生命周期结束后仍触发回调。

```kotlin
// In your Activity or Fragment
override fun onDestroy() {
    super.onDestroy()
    cleanupAds()

    // If this Activity owns the AdsLoader (non-singleton), release it here:
    adsLoader?.removeAdsLoadedListener(adsLoadedListener)
    adsLoader?.removeAdErrorListener(adsLoaderErrorListener)
    adsLoader?.release()
}

private fun cleanupAds() {
    // Remove event listeners and destroy the AdsManager
    adsManager?.apply {
        removeAdEventListener(adEventListener)
        removeAdErrorListener(adErrorListener)
        destroy()
    }
    adsManager = null
}

// Wherever you manage the AdsLoader (e.g. Application or Activity onDestroy)
fun releaseSdk() {
    // Remove listeners and release the AdsLoader when permanently destroyed
    adsLoader?.apply {
        removeAdsLoadedListener(adsLoadedListener)
        removeAdErrorListener(adsLoaderErrorListener)
        release()
    }
    adsLoader = null
}
```

--------------------------------------------------------------------------------

## 参考实现

*   基础示例 [MyActivity.java](https://raw.githubusercontent.com/googleads/googleads-ima-android/main/basicexample/app/src/main/java/com/google/ads/interactivemedia/v3/samples/videoplayerapp/MyActivity.java)
*   基础示例 [VideoAdPlayerAdapter.java](https://raw.githubusercontent.com/googleads/googleads-ima-android/main/basicexample/app/src/main/java/com/google/ads/interactivemedia/v3/samples/videoplayerapp/VideoAdPlayerAdapter.java)