# Google IMA SDK HTML5（Web）集成指南

本指南按广告播放的自然生命周期，介绍 Google IMA SDK for HTML5（客户端）的集成方法。

## 集成流程

### 1. 导入 SDK

在页面级别加载 SDK 脚本。确保 SDK 能访问 `window.top.location.href`。

```html
<script src="https://imasdk.googleapis.com/js/sdkloader/ima3.js"></script>
```

### 2. 初始化

尽早配置 SDK 设置，搭建广告 UI，并初始化 `AdDisplayContainer` 和 `AdsLoader`。

*   **尽早配置设置（最佳实践）：** **必须**在创建 `AdDisplayContainer` 或 `AdsLoader` *之前*，在全局单例 `google.ima.settings` 上设置配置。之后再改不会生效。
*   **广告 UI 搭建：** 始终为 IMA SDK 单独使用一个专用 HTML 元素（如 `<div id="adContainer">`），直接叠放在视频播放器之上。广告播放时隐藏自定义播放器控件。
*   **创建 AdDisplayContainer：** 传入广告容器 HTML 元素和内容视频元素，实例化 `google.ima.AdDisplayContainer`。
*   **创建 AdsLoader：** 传入 `AdDisplayContainer` 实例，实例化 `google.ima.AdsLoader`。该对象负责广告请求的生命周期。

```html
<!-- HTML Structure -->
<div id="playerContainer" style="position:relative; width:640px; height:360px;">
  <video id="contentVideo" style="width:100%; height:100%;"></video>
  <div id="adContainer" style="position:absolute; top:0; left:0; width:100%; height:100%; z-index:10;"></div>
</div>
```

```typescript
// TypeScript Initialization
let adsLoader: google.ima.AdsLoader;
let adDisplayContainer: google.ima.AdDisplayContainer;

function initializeIMA(): void {
    // 1. Configure global settings early
    google.ima.settings.setLocale('en');
    google.ima.settings.setVpaidMode(google.ima.ImaSdkSettings.VpaidMode.ENABLED);

    // 2. Set up Ad UI Container
    const adContainerElement = document.getElementById('adContainer') as HTMLElement;
    const videoElement = document.getElementById('contentVideo') as HTMLVideoElement;

    adDisplayContainer = new google.ima.AdDisplayContainer(adContainerElement, videoElement);

    // 3. Initialize AdsLoader
    adsLoader = new google.ima.AdsLoader(adDisplayContainer);

    // Setup listeners for Ad Load Success/Failure
    setupAdsLoaderListeners();
}
```

### 3. 搭建应用 UI

**必须**建议采用最佳实践：通过用户操作（如按钮点击）启动内容和广告播放。

如果应用需要自动播放广告，必须编写代码检查浏览器是否支持自动播放。非静音自动播放失败时，回退到静音自动播放。若仍失败，回退到通过用户操作（如按钮点击）启动内容和广告播放。

关键点：必须通过 `setAdWillAutoPlay()` 和 `setAdWillPlayMuted()` 将最终状态告知 IMA SDK。这样广告服务器才能返回适配当前播放环境的广告。

```typescript
const request = new google.ima.AdsRequest();
request.adTagUrl = 'YOUR_AD_TAG_URL';

// Inform the SDK of the current playback capabilities
request.setAdWillAutoPlay(isAutoPlay);
request.setAdWillPlayMuted(isMuted);

adsLoader.requestAds(request);
```

### 4. 广告请求

创建 `AdsRequest`，用 `AdsLoader` 加载。

```typescript
function requestAds() {
    adDisplayContainer.initialize();

    const adsRequest = new google.ima.AdsRequest();
    adsRequest.adTagUrl = `YOUR_AD_TAG_URL`;

    adsLoader.requestAds(adsRequest);
}
```

### 5. 广告加载成功/失败

监听 `AdsLoader` 成功加载广告（提供 `AdsManager`）或提前失败（致命加载错误）。

```typescript
let adsManager: google.ima.AdsManager | null = null;

function setupAdsLoaderListeners(): void {
    // Handle Ad Load Success
    adsLoader.addEventListener(
        google.ima.AdsManagerLoadedEvent.Type.ADS_MANAGER_LOADED,
        onAdsManagerLoaded
    );

    // Handle Ad Load Failure (Fatal Error)
    adsLoader.addEventListener(
        google.ima.AdErrorEvent.Type.AD_ERROR,
        onAdError
    );
}

function onAdsManagerLoaded(adsManagerLoadedEvent: google.ima.AdsManagerLoadedEvent): void {
    // Get the AdsManager
    const adsRenderingSettings = new google.ima.AdsRenderingSettings();
    adsManager = adsManagerLoadedEvent.getAdsManager(videoElement, adsRenderingSettings);

    setupAdsManagerListeners(adsManager);

    try {
        // Initialize the manager
        adsManager.init(width, height, google.ima.ViewMode.NORMAL);
        // Start ad playback
        adsManager.start();
    } catch (adError) {
        // Handle initialization errors
        resumeContent();
    }
}

function onAdError(adErrorEvent: google.ima.AdErrorEvent): void {
    console.error("IMA SDK Fatal Error:", adErrorEvent.getError().toString());
    cleanupAds();
    resumeContent(); // Fallback to content
}
```

### 6. 广告播放事件

用 `AdsManager` 监听播放事件、协调内容暂停/恢复，并监控非致命日志。

*   **暂停内容：** 收到 `CONTENT_PAUSE_REQUESTED` 时，暂停内容视频播放器并隐藏自定义控件。
*   **恢复内容：** 收到 `CONTENT_RESUME_REQUESTED` 时，隐藏广告容器、恢复播放器控件并播放内容视频。
*   **非致命日志：** 监听 `LOG` 事件以发现静默失败（如跟踪 ping 失败），不打断用户。

```typescript
function setupAdsManagerListeners(manager: google.ima.AdsManager): void {
    // Playback Events
    manager.addEventListener(
        google.ima.AdEvent.Type.CONTENT_PAUSE_REQUESTED,
        () => { pauseContent(); }, // Pause player, hide custom controls
        false
    );

    manager.addEventListener(
        google.ima.AdEvent.Type.CONTENT_RESUME_REQUESTED,
        () => { resumeContent(); }, // Resume player, restore controls
        false
    );

    // Non-Fatal Logs
    manager.addEventListener(
        google.ima.AdEvent.Type.LOG,
        (adEvent: google.ima.AdEvent) => {
            const logData = adEvent.getAdData();
            if (logData && logData['errorMessage']) {
                console.warn("IMA SDK Non-fatal Log:", logData['errorMessage']);
            }
        },
        false
    );

    // Playback Errors (Fatal errors during active playback)
    manager.addEventListener(
        google.ima.AdErrorEvent.Type.AD_ERROR,
        onAdError,
        false
    );
}
```

### 7. 清理

在 HTML5 中，正确清理**至关重要**，可防止内存泄漏、冗余事件回调和残留 DOM 元素（如 SDK iframe）。

*   **`adsManager.destroy()`：** 这一步很关键。广告结束、发生致命错误或播放器卸载/离开页面时，务必调用。
*   **`adsLoader.contentComplete()`：** 调用它以告知 SDK 内容已结束。SDK 播放片尾广告需要此调用。
*   **`adDisplayContainer.destroy()`：** 销毁播放器时调用。它会移除 SDK 创建的内部 DOM 元素（包括 iframe）。
*   **置空引用：** 将所有引用设为 `null`，以便垃圾回收。

```typescript
// Call this when the player is being destroyed/unmounted (e.g., ngOnDestroy, componentWillUnmount)
function tearDownPlayer(): void {
    cleanupAds();

    // 1. Signal content completion (important for post-rolls)
    if (adsLoader) {
        adsLoader.contentComplete();
    }

    // 2. Destroy the display container to clean up DOM/iframes
    if (adDisplayContainer) {
        adDisplayContainer.destroy();
        adDisplayContainer = null;
    }

    // 3. Nullify the loader if tearing down the page
    adsLoader = null;
}

function cleanupAds(): void {
    // 4. Destroy the AdsManager to release active ad resources
    if (adsManager) {
        adsManager.destroy();
        adsManager = null;
    }
}
```

--------------------------------------------------------------------------------

## 参考实现

*   UI 搭建：[simple/index.html](https://raw.githubusercontent.com/googleads/googleads-ima-html5/main/simple/index.html)
*   应用逻辑：[simple/ads.js](https://raw.githubusercontent.com/googleads/googleads-ima-html5/main/simple/ads.js)
*   可选：为 TypeScript 项目安装类型定义：

```bash
    npm install --save-dev @types/google_interactive_media_ads_types
```