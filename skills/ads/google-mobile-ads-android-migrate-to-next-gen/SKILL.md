---
name: google-mobile-ads-android-migrate-to-next-gen
description: 将 Android 应用从旧版 Google Mobile Ads (GMA) SDK（com.google.android.gms:play-services-ads）迁移到新版 GMA Next-Gen SDK（com.google.android.libraries.ads.mobile.sdk:ads-mobile-sdk）。提供导入、类和方法签名的完整映射表，帮助确定迁移步骤。适用于将现有 Android 代码库从旧版 GMA SDK 迁移到 GMA Next-Gen SDK。
metadata:
  version: "1.1.0"
  category: GoogleAds
---
# Google Mobile Ads SDK AI 迁移代理说明

## 迁移工作流

使用此清单跟踪迁移进度：

*   **配置 Gradle**：
    -   [ ] 将 `com.google.android.gms:play-services-ads` 替换为最新稳定版的 `com.google.android.libraries.ads.mobile.sdk:ads-mobile-sdk`。如果无法直接访问 Maven，运行以下命令获取最新版本。`latest` 标签返回的版本即为 GMA Next-Gen SDK 的最新版本：

        ```bash
        curl -sS https://dl.google.com/dl/android/maven2/com/google/android/libraries/ads/mobile/sdk/ads-mobile-sdk/maven-metadata.xml | sed -n 's/.*<latest>\(.*\)<\/latest>.*/\1/p'
        ```

    -   [ ] 更新 `minSdk`（24+）和 `compileSdk`（34+）。
    -   [ ] 在应用级构建文件中全局排除所有依赖中的 `play-services-ads` 和 `play-services-ads-lite`，避免符号重复错误。
    -   [ ] 同步 Gradle 后再继续下一步。
*   **逐文件迁移**：
    -   [ ] 按照 API 映射表和方法映射表重构代码库，将导入、类名和方法签名迁移到 GMA Next-Gen SDK。
*   **验证与构建**：
    -   [ ] 运行 `gradle build -x test` 确认构建成功。解决所有 GMA SDK 相关的编译错误。

## 核心迁移规则

*   **App ID 使用**：`InitializationConfig` 中的 `applicationId` **必须**使用 `AndroidManifest.xml` 中 `com.google.android.gms.ads.APPLICATION_ID` meta-data 标签的值。
    *   *约束*：保留清单中的 `<meta-data>` 标签；使用 User Messaging Platform SDK 的发布者仍需要它。
*   **初始化顺序**：
    1.  在后台线程调用 `MobileAds.initialize()`。
    2.  确保 `initialize()` 在**所有其他 SDK 方法之前**调用。
    3.  如果使用 `RequestConfiguration`，将其传入 `InitializationConfig.Builder.setRequestConfiguration()`。**不要**在初始化前调用 `MobileAds.setRequestConfiguration()`。
*   **UI 线程**：**强制要求**：GMA Next-Gen SDK 的回调在后台线程执行。GMA SDK 回调中的**所有 UI 相关操作**（如 Toast、View 更新、Fragment 事务）**必须**包裹在 `runOnUiThread {}` 或 `Dispatchers.Main.launch {}` 中。跳过此步骤将导致应用崩溃。
*   **中介**：实现 `com.google.android.gms.ads.mediation.Adapter` 的类必须继续使用 `com.google.android.gms.ads`。

## 格式细节

### 横幅广告

*   使用 `com.google.android.libraries.ads.mobile.sdk.banner.AdView` 加载 GMA Next-Gen SDK 横幅。
*   以下 API 检查横幅是否可折叠：`adView.getBannerAd().isCollapsible()`。

### 原生广告

*   **NativeAdLoader**：`NativeAdLoader` 是抽象类，无法实例化。以静态方式使用（如 `NativeAdLoader.load(...)`）。
*   以下 API 现在设置在 `NativeAdRequest.Builder` 上：
    *   `.setCustomFormatIds(customFormatIds: List<String>)`
    *   `.disableImageDownloading()`
    *   `.setMediaAspectRatio(mediaAspectRatio: NativeMediaAspectRatio)`
    *   `.setAdChoicesPlacement(adChoicesPlacement: AdChoicesPlacement)`
    *   `.setVideoOptions(videoOptions: VideoOptions)`
*   **移除**：删除所有“Mute This Ad”逻辑；GMA Next-Gen SDK 不支持。
*   **MediaView**：`NativeAd` 不再有直接的 `mediaView` 变量；使用 `registerNativeAd(nativeAd, mediaView)` 将广告与视图关联。

### 广告预加载

*   除非映射表中另有说明，GMA Next-Gen SDK 中的广告预加载方法保留与旧 SDK 相同的 API 签名和参数。

## API 映射

此表涵盖主要类及其 GMA Next-Gen SDK 对应项。

| 功能                    | 旧 SDK 导入 (`com.google.android.gms.ads...`)                                     | GMA Next-Gen SDK 导入 (`com.google.android.libraries.ads.mobile.sdk...` )                                                                                                           |
|:---------------------------|:-------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **核心**                   |                                                                                      |                                                                                                                                                                                       |
| 初始化             | `MobileAds`                                                                          | `MobileAds`, `initialization.InitializationConfig`                                                                                                                                    |
| 初始化监听器    | `initialization.OnInitializationCompleteListener`                                    | `initialization.OnAdapterInitializationCompleteListener`                                                                                                                              |
| 广告请求                 | `AdRequest`                                                                          | *格式特定*（如 `common.AdRequest`、`banner.BannerAdRequest`、`nativead.NativeAdRequest`）（广告单元 ID 在 `Builder` 中声明。Load() 不再接收 activity）           |
| 加载错误                 | `LoadAdError`                                                                        | `common.LoadAdError`（`LoadAdError` 不再有 domain 变量。如发现 domain 变量请移除。）                                                                            |
| 全屏展示错误     | `AdError`（在 `FullScreenContentCallback` 中）                                       | `common.FullScreenContentError`（在格式特定的 `AdEventCallback` 中）                                                                                                            |
| 请求配置      | `RequestConfiguration`                                                               | `common.RequestConfiguration`（RequestConfiguration 的嵌套枚举/常量现在位于 common.RequestConfiguration 下。）                                                            |
| 事件回调            | `FullScreenContentCallback`（全屏格式）、`AdListener`（横幅、原生） | *格式特定*（如 `interstitial.InterstitialAdEventCallback`、`banner.BannerAdEventCallback`、`native.NativeAdEventCallback`）。广告格式上的变量为 `adEventCallback`。 |
| **工具**                  |                                                                                      |                                                                                                                                                                                       |
| 广告检查器               | `MobileAds.openAdInspector()`                                                        | `MobileAds.openAdInspector()`（`openAdInspector` 不再接收 activity）                                                                                                         |
| 广告检查器监听器      | `OnAdInspectorClosedListener`                                                        | `common.OnAdInspectorClosedListener`                                                                                                                                                  |
| **格式**                |                                                                                      |                                                                                                                                                                                       |
| 开屏广告                   | `appopen.AppOpenAd`                                                                  | `appopen.AppOpenAd`                                                                                                                                                                   |
| 开屏广告加载              | `appopen.AppOpenAd.AppOpenAdLoadCallback`                                            | `common.AdLoadCallback<appopen.AppOpenAd>`                                                                                                                                            |
| 横幅                     | `AdView`, `AdSize`                                                                   | `banner.AdView`, `banner.AdSize`（`AdView` 不再有 `pause()`、`resume()`、`setAdSize()`）。`AdSize` 在 `BannerAdRequest` 中声明。                                            |
| 横幅加载                | `AdListener`                                                                         | `common.AdLoadCallback<banner.BannerAd>`                                                                                                                                              |
| 横幅事件              | `AdListener`                                                                         | `banner.BannerAdEventCallback`, `banner.BannerAdRefreshCallback`                                                                                                                      |
| 插页广告               | `interstitial.InterstitialAd`                                                        | `interstitial.InterstitialAd`                                                                                                                                                         |
| 插页广告加载          | `interstitial.InterstitialAd.InterstitialAdLoadCallback`                             | `common.AdLoadCallback<interstitial.InterstitialAd>`                                                                                                                                  |
| 广告加载器                  | `AdLoader`                                                                           | `nativead.NativeAdLoader`                                                                                                                                                             |
| 原生                     | `nativead.NativeAd`                                                                  | `nativead.NativeAd`（不再有 `mediaView` 变量）                                                                                                                            |
| 原生自定义格式广告    | `nativead.NativeCustomFormatAd`                                                      | `nativead.CustomNativeAd`                                                                                                                                                             |
| 原生自定义点击        | `nativead.NativeCustomFormatAd.OnCustomClickListener`                                | `nativead.OnCustomClickListener`（设置在 `CustomNativeAd` 对象上（如 `.onCustomClickListener`）                                                                                 |
| 原生加载                | `nativead.NativeAd.OnNativeAdLoadedListener`                                         | `nativead.NativeAdLoaderCallback`                                                                                                                                                     |
| 原生广告视图             | `nativead.NativeAdView`                                                              | `nativead.NativeAdView`                                                                                                                                                               |
| 媒体内容              | `MediaContent`                                                                       | `nativead.MediaContent`（hasVideoContent 声明为 `val`）                                                                                                                      |
| 媒体宽高比         | `MediaAspectRatio`                                                                   | `nativead.MediaAspectRatio`                                                                                                                                                           |
| 视频选项               | `VideoOptions`                                                                       | `common.VideoOptions`                                                                                                                                                                 |
| 视频控制器           | `VideoController`                                                                    | `common.VideoController`（VideoLifecycleCallbacks 现在是接口，需用 `object : VideoController.VideoLifecycleCallbacks { ... }` 实例化）                                |
| 激励广告                   | `rewarded.RewardedAd`                                                                | `rewarded.RewardedAd`                                                                                                                                                                 |
| 激励广告加载              | `rewarded.RewardedAd.RewardedAdLoadCallback`                                         | `common.AdLoadCallback<rewarded.RewardedAd>`                                                                                                                                          |
| 激励插页广告      | `rewardedinterstitial.RewardedInterstitialAd`                                        | `rewardedinterstitial.RewardedInterstitialAd`                                                                                                                                         |
| 激励插页广告加载 | `rewardedinterstitial.RewardedInterstitialAd.RewardedInterstitialAdLoadCallback`     | `common.AdLoadCallback<rewardedinterstitial.RewardedInterstitialAd>`                                                                                                                  |
| 付费事件监听器        | `OnPaidEventListener`                                                                | `common.AdEventCallback`                                                                                                                                                              |
| 响应信息              | `ResponseInfo`                                                                       | `common.ResponseInfo`                                                                                                                                                                 |
| 适配器响应信息      | `AdapterResponseInfo`                                                                | `common.AdSourceResponseInfo`                                                                                                                                                         |
| **奖励**                |                                                                                      |                                                                                                                                                                                       |
| 奖励监听器            | `OnUserEarnedRewardListener`                                                         | `rewarded.OnUserEarnedRewardListener`                                                                                                                                                 |
| 奖励项                | `rewarded.RewardItem`                                                                | `rewarded.RewardItem`（`RewardedAd` 和 `RewardedInterstitialAd` 上的属性访问现在为 `getRewardItem()`）                                                                         |
| 广告价值                   | `AdValue`                                                                            | `common.AdValue`                                                                                                                                                                      |
| **预加载**             |                                                                                      |                                                                                                                                                                                       |
| 配置              | `preload.PreloadConfiguration`                                                       | `common.PreloadConfiguration`                                                                                                                                                         |
| 回调                   | `preload.PreloadCallbackV2`                                                          | `common.PreloadCallback`（现在是接口而非抽象类）                                                                                                              |
| 插页广告预加载器     | `interstitial.InterstitialPreloader`                                                 | `interstitial.InterstitialAdPreloader`                                                                                                                                                |
| **广告管理器**             |                                                                                      |                                                                                                                                                                                       |
| 广告请求                 | `admanager.AdManagerAdRequest`                                                       | `common.AdRequest`（现在直接在 `AdRequest` 类中实现）                                                                                                                |
| 广告视图                    | `admanager.AdView`                                                                   | `banner.AdView`（无 `AdManagerAdView` 类）                                                                                                                                          |
| 应用事件监听器         | `admanager.AppEventListener`                                                         | `common.OnAppEventListener`                                                                                                                                                           |

## 方法映射

此表涵盖主要方法及其 GMA Next-Gen SDK 对应项。

| 功能                                  | 旧 SDK 方法签名                                                                                                                     | GMA Next-Gen SDK 方法签名                                                                                                                                                                                                                                                          |
|:-----------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **核心**                                 |                                                                                                                                              |                                                                                                                                                                                                                                                                                            |
| MobileAds 初始化                 | `MobileAds.initialize(Context context, OnInitializationCompleteListener listener)`                                                           | `MobileAds.initialize(Context context, InitializationConfig config, OnInitializationCompleteListener listener)`                                                                                                                                                                            |
| InitializationConfig Builder             | 不适用                                                                                                                                          | `InitializationConfig.Builder(String applicationId)`                                                                                                                                                                                                                                       |
| 广告请求 Builder                       | `AdRequest.Builder().build()`                                                                                                                | `AdRequest.Builder(String adUnitId).build()`（用于开屏、插页、激励、激励插页）**横幅：** `BannerAdRequest.Builder(String adUnitId, AdSize adSize).build()` **原生：** `NativeAdRequest.Builder(String adUnitId, nativeAdTypes: List<NativeAdType>).build()` |
| 添加网络额外参数 (AdMobAdapter)        | `AdRequest.Builder().addNetworkExtrasBundle(Class<MediationExtrasReceiver>, Bundle networkExtras)`                                           | `AdRequest.Builder(String adUnitId).setGoogleExtrasBundle(Bundle extraBundle)`                                                                                                                                                                                                             |
| 添加网络额外参数 (广告源适配器)   | `AdRequest.Builder().addNetworkExtrasBundle(Class<MediationExtrasReceiver>, Bundle networkExtras)`                                           | `AdRequest.Builder(String adUnitId).putAdSourceExtrasBundle(Class<MediationExtrasReceiver> adapterClass, Bundle adSourceExtras)`                                                                                                                                                           |
| 自定义定向                         | `AdRequest.Builder().setCustomTargeting(String key, String value)`                                                                           | `AdRequest.Builder(String adUnitId).putCustomTargeting(String key, String value)`                                                                                                                                                                                                          |
| **格式**                              |                                                                                                                                              |                                                                                                                                                                                                                                                                                            |
| 开屏广告                                 | `AppOpenAd.load(Context context, String adUnitId, AdRequest request, AppOpenAdLoadCallback loadCallback)`                                    | `AppOpenAd.load(AdRequest request, AdLoadCallback<AppOpenAd> loadCallback)`                                                                                                                                                                                                                |
| 横幅                                   | `AdView.loadAd(AdRequest request)`                                                                                                           | `AdView.loadAd(BannerAdRequest request, AdLoadCallback<BannerAd> loadCallback)`                                                                                                                                                                                                            |
| 插页广告                             | `InterstitialAd.load(Context context, String adUnitId, AdRequest request, InterstitialAdLoadCallback loadCallback)`                          | `InterstitialAd.load(AdRequest request, AdLoadCallback<InterstitialAd> loadCallback)`                                                                                                                                                                                                      |
| 激励广告                                 | `RewardedAd.load(Context context, String adUnitId, AdRequest request, RewardedAdLoadCallback loadCallback)`                                  | `RewardedAd.load(AdRequest request, AdLoadCallback<RewardedAd> loadCallback)`                                                                                                                                                                                                              |
| 激励插页广告                    | `RewardedInterstitialAd.load(Context context, String adUnitId, AdRequest request, RewardedInterstitialAdLoadCallback loadCallback)`          | `RewardedInterstitialAd.load(AdRequest request, AdLoadCallback<RewardedInterstitialAd> loadCallback)`                                                                                                                                                                                      |
| 原生 Builder                           | `AdLoader.Builder(Context context, String adUnitId).forNativeAd(NativeAd.OnNativeAdLoadedListener onNativeAdLoadedListener)`                 | `NativeAdRequest.Builder(String adUnitId, nativeAdTypes: List<NativeAdType>)`（在 `nativeAdTypes` 中包含 `NativeAd.NativeAdType.NATIVE`）                                                                                                                                                  |
| 原生加载                              | `AdLoader.Builder(...).build().loadAd(AdRequest request)`                                                                                    | `NativeAdLoader.load(NativeAdRequest request, NativeAdLoaderCallback callback)`                                                                                                                                                                                                            |
| 原生广告注册                       | `NativeAdView.setNativeAd(NativeAd nativeAd)`                                                                                                | `NativeAdView.registerNativeAd(NativeAd nativeAd, mediaView: MediaView?)`                                                                                                                                                                                                                  |
| 设置应用事件监听器（横幅）       | `AdManagerAdView.appEventListener`                                                                                                           | `BannerAd.adEventCallback`（`onAppEvent(name: String, data: String?)` 现在是 `BannerAdEventCallback` 的一部分）                                                                                                                                                                          |
| 设置应用事件监听器（插页） | `AdManagerInterstitialAd.appEventListener`                                                                                                   | `InterstitialAd.adEventCallback`（`onAppEvent(name: String, data: String?)` 现在是 `InterstitialAdEventCallback` 的一部分）                                                                                                                                                              |
| **回调**                            |                                                                                                                                              |                                                                                                                                                                                                                                                                                            |
| onAdOpened                               | `AdListener.onAdOpened()`                                                                                                                    | `AdEventCallback.onAdShowedFullScreenContent()`                                                                                                                                                                                                                                            |
| onAdClosed                               | `AdListener.onAdClosed()`                                                                                                                    | `AdEventCallback.onAdDismissedFullScreenContent()`                                                                                                                                                                                                                                         |
| onFailedToShowFullScreenContent          | `onAdFailedToShowFullScreenContent(adError: AdError)`                                                                                        | `onAdFailedToShowFullScreenContent(fullScreenContentError: FullScreenContentError)`                                                                                                                                                                                                        |
| onAdLoaded                               | **AdLoadCallback**：`onAdLoaded(ad: T)`（如 `InterstitialAdLoadCallback`、`RewardedAdLoadCallback`、`RewardedInterstitialAdLoadCallback`） | 参数名始终为 `ad` **格式特定**：`onAdLoaded(ad: InterstitialAd)`、`onAdLoaded(ad: RewardedAd)`、`onAdLoaded(ad: RewardedInterstitialAd)`                                                                                                                                |
| onAdFailedToLoad                         | `onAdFailedToLoad(loadAdError: LoadAdError)`                                                                                                 | `onAdFailedToLoad(adError: LoadAdError)`                                                                                                                                                                                                                                                   |
| onCustomFormatAdLoaded                   | `OnCustomFormatAdLoadedListener.onCustomFormatAd