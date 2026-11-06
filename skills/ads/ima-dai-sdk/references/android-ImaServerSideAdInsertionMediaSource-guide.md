# Google IMA DAI SDK Android - ExoPlayer IMA 扩展指南

使用 ExoPlayer IMA 扩展（`androidx.media3:media3-exoplayer-ima`）中的 `ImaServerSideAdInsertionMediaSource` 类，从 Google 全服务 DAI 请求直播流或 VOD 流。

## 目录

*   [依赖](#dependencies)（第 21 行）
*   [IMA DAI SDK 所需权限](#ima-dai-sdk-required-permissions)
    （第 57 行）
*   [UI 设置](#ui-setup)（第 66 行）
*   [尽早初始化 SDK](#early-sdk-initialization)（第 89 行）
*   [创建可复用的 AdsLoader](#create-a-reusable-adsloader)（第 104 行）
*   [监听流事件和广告事件](#listen-to-stream-events-and-ad-events)
    （第 123 行）
*   [使用 SSAI MediaSource Factory 初始化 ExoPlayer](#initialize-exoplayer-with-ssai-mediasource-factory)
    （第 155 行）
*   [请求直播流](#request-a-livestream)（第 180 行）
    *   [视频点播（VOD）流](#video-on-demand-vod-streams)（第 202 行）
*   [清理 SDK 资源](#clean-up-sdk-resources)（第 227 行）
*   [参考实现](#reference-implementation)（第 240 行）

## 依赖

在 `build.gradle` 中添加 Media3 ExoPlayer IMA 扩展。该组件会自动导入 IMA DAI SDK。

```groovy
apply plugin: "com.android.application"

android {
    compileOptions {
        // Required by IMA SDK v3.37.0+
        coreLibraryDesugaringEnabled = true
    }

    defaultConfig {
        minSdkVersion(23)
    }
}

repositories {
    google()
    mavenCentral()
}

dependencies {
    implementation("androidx.media3:media3-ui")
    implementation("androidx.media3:media3-exoplayer")
    implementation("androidx.media3:media3-exoplayer-hls")
    implementation("androidx.media3:media3-exoplayer-dash")
    implementation("androidx.media3:media3-exoplayer-ima")
}
```

使用 [Media3](https://developer.android.com/jetpack/androidx/releases/media3.md.txt) 上发布的最新版本。

## IMA DAI SDK 所需权限

确保在 `AndroidManifest.xml` 中声明 IMA DAI SDK 所需的权限：

```xml
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
```

## UI 设置

将 `PlayerView` 放入布局中。例如：

```xml
<androidx.media3.ui.PlayerView android:id="@+id/player_view" />
```

如果使用 Jetpack Compose，创建一个包裹 exoplayer 的 `PlayerView`：

```kotlin
AndroidView(
  factory = { ctx ->
    PlayerView(ctx).apply {
      player = exoPlayer
    }
  },
  update = { playerView ->
    playerView.player = exoPlayer
  }
)
```

## 尽早初始化 SDK

在应用生命周期中尽早初始化 IMA SDK（例如在 `Application` 或主 `Activity` 的 `onCreate()` 中），以缩短流加载时间：

创建 `ImaSdkSettings` 对象来初始化 SDK。保存该对象，以便与 `ImaServerSideAdInsertionMediaSource` 复用。

```kotlin
val imaSdkFactory = ImaSdkFactory.getInstance()
val sharedImaSdkSettings: ImaSdkSettings = imaSdkFactory.createImaSdkSettings()
imaSdkFactory.initialize(this, sharedImaSdkSettings)
```

## 创建可复用的 `AdsLoader`

提供 `androidx.media3.ui.PlayerView` 来构建 `ImaServerSideAdInsertionMediaSource.AdsLoader`。

提供与上一步 `ImaSdkFactory.getInstance().initialize()` 相同的 `ImaSdkSettings`。

```kotlin
val playerView = findViewById(R.id.player_view)
val adsLoaderBuilder =
    ImaServerSideAdInsertionMediaSource.AdsLoader.Builder(this, playerView)

val adsLoader = adsLoaderBuilder
    .setAdEventListener(buildAdEventListener())
    .setImaSdkSettings(sharedImaSdkSettings)
    .build()
```

## 监听流事件和广告事件

通过创建 `AdEventListener` 实现 DAI 流事件处理。在实例化时将此监听器提供给 `AdsLoader` 构建器。

```kotlin
val adEventListener = AdEvent.AdEventListener { event ->
    when (event.type) {
      AdEvent.AdEventType.LOADED,
      AdEvent.AdEventType.CUEPOINTS_CHANGED,
      AdEvent.AdEventType.AD_BREAK_STARTED,
      AdEvent.AdEventType.AD_BREAK_ENDED,
      AdEvent.AdEventType.AD_PERIOD_STARTED,
      AdEvent.AdEventType.AD_PERIOD_ENDED,
      AdEvent.AdEventType.STARTED,
      AdEvent.AdEventType.FIRST_QUARTILE,
      AdEvent.AdEventType.MIDPOINT,
      AdEvent.AdEventType.THIRD_QUARTILE,
      AdEvent.AdEventType.COMPLETED,
      AdEvent.AdEventType.PAUSED,
      AdEvent.AdEventType.RESUMED,
      AdEvent.AdEventType.SKIPPABLE_STATE_CHANGED,
      AdEvent.AdEventType.SKIPPED,
      AdEvent.AdEventType.CLICKED -> Log.i(LOG_TAG, "Ad event: ${event.type}")
      AdEvent.AdEventType.AD_PROGRESS -> {
        // High-frequency event fired periodically during ad playback; ignore to prevent log spam
      }
      else -> Log.i(LOG_TAG, "Unhandled ad event: ${event.type}")
    }
  }
```

## 使用 SSAI MediaSource Factory 初始化 ExoPlayer

创建 `DefaultMediaSourceFactory` 和 `ImaServerSideAdInsertionMediaSource.Factory` 对象来构建 `ExoPlayer` 实例。

确保在使用 `ImaServerSideAdInsertionUri` 调用 `ExoPlayer.setMediaItem` 之前，先调用 `AdsLoader.setPlayer()` 并提供上述 `ExoPlayer` 实例。

```kotlin
val dataSourceFactory: DataSource.Factory = DefaultDataSource.Factory(this)
val mediaSourceFactory = DefaultMediaSourceFactory(dataSourceFactory)

val adsMediaSourceFactory =
    ImaServerSideAdInsertionMediaSource.Factory(adsLoader, mediaSourceFactory)

mediaSourceFactory.setServerSideAdInsertionMediaSourceFactory(adsMediaSourceFactory)

player = ExoPlayer.Builder(this)
    .setMediaSourceFactory(mediaSourceFactory)
    .build()
adsLoader.setPlayer(player)
```

## 请求直播流

使用 `ImaServerSideAdInsertionUriBuilder` 和资产键构建直播流 URI：

```kotlin
val liveStreamUri: Uri = ImaServerSideAdInsertionUriBuilder()
      .setNetworkCode(<NETWORK_CODE_PLACEHOLDER>)
      .setAssetKey(<ASSET_KEY_PLACEHOLDER>)
      .setFormat(androidx.media3.common.C.CONTENT_TYPE_HLS)
      .build()

val liveStreamMediaItem: MediaItem = MediaItem.fromUri(liveStreamUri)
player.setMediaItem(liveStreamMediaItem)
```

提供以下参数：

*   **<NETWORK_CODE_PLACEHOLDER>**：Google Ad Manager 网络代码。
*   **<ASSET_KEY_PLACEHOLDER>**：在 Google Ad Manager 中配置的直播流资产键。

### 视频点播（VOD）流

使用 `ImaServerSideAdInsertionUriBuilder` 和内容源 ID（CMS ID）及视频 ID 构建 VOD 流 URI：

```kotlin
val vodStreamUri: Uri = ImaServerSideAdInsertionUriBuilder()
      .setNetworkCode(<NETWORK_CODE_PLACEHOLDER>)
      .setContentSourceId(<CONTENT_SOURCE_ID_PLACEHOLDER>)
      .setVideoId(<VIDEO_ID_PLACEHOLDER>)
      .setFormat(androidx.media3.common.C.CONTENT_TYPE_HLS)
      .build()
val vodStreamItem: MediaItem = MediaItem.fromUri(vodStreamUri)
player.setMediaItem(vodStreamItem)
```

提供以下参数：

*   **<NETWORK_CODE_PLACEHOLDER>**：Google Ad Manager 网络代码。
*   **<CONTENT_SOURCE_ID_PLACEHOLDER>**：Google Ad Manager 中的 CMS ID。
*   **<VIDEO_ID_PLACEHOLDER>**：CMS 中的视频 ID。

用于测试时，请使用 https://developers.google.com/ad-manager/dynamic-ad-insertion/streams.md.txt?utm_source=agent-skills&utm_medium=content&utm_campaign=adr-ss-ai&utm_content=ima-dai-sdk 中 DAI 示例流的值。

## 清理 SDK 资源

在释放播放器或处理流错误时调用 `ImaServerSideAdInsertionMediaSource.AdsLoader.release()`，以关闭活动的 DAI 会话。

保存返回的 `AdsLoader.State` 对象，以便在屏幕旋转等配置变更期间恢复播放器。

```kotlin
adsLoader?.setPlayer(null)
adsLoaderState = adsLoader?.release()
```

## 参考实现

*   [ExoPlayerExample/app/src/main/res/layout/activity_my.xml](https://raw.githubusercontent.com/googleads/googleads-ima-android-dai/refs/heads/main/ExoPlayerExample/app/src/main/res/layout/activity_my.xml)
*   [ExoPlayerExample/app/src/main/java/com/google/ads/interactivemedia/v3/samples/videoplayerapp/MyActivity.java](https://raw.githubusercontent.com/googleads/googleads-ima-android-dai/refs/heads/main/ExoPlayerExample/app/src/main/java/com/google/ads/interactivemedia/v3/samples/videoplayerapp/MyActivity.java)