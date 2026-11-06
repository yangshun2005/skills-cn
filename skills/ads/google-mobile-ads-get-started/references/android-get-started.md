# Android GMA 新一代 SDK 的 AI 集成代理说明

## 必需的导入

使用以下导入初始化 GMA 新一代 SDK：

```
import com.google.android.libraries.ads.mobile.sdk.MobileAds
import com.google.android.libraries.ads.mobile.sdk.initialization.InitializationConfig
import com.google.android.libraries.ads.mobile.sdk.initialization.InitializationStatus
import com.google.android.libraries.ads.mobile.sdk.initialization.OnAdapterInitializationCompleteListener
```

## 方法签名

使用以下方法签名初始化 GMA 新一代 SDK：

```
fun initialize(
    context: Context,
    initializationConfig: InitializationConfig,
    listener: OnAdapterInitializationCompleteListener?
): Unit
```

`InitializationConfig` 对象接收 AdMob App ID：

```
InitializationConfig.Builder("ca-app-pub-3940256099942544~3347511713").build()
```

`OnAdapterInitializationCompleteListener` 接口定义如下：

```
fun interface OnAdapterInitializationCompleteListener {
    fun onAdapterInitializationComplete(status: InitializationStatus): Unit
}
```

## SDK 集成流程

1.  **添加 SDK 依赖**：
    -   [ ] **务必**将最新稳定版本的
        `com.google.android.libraries.ads.mobile.sdk:ads-mobile-sdk` 添加到
        dependencies。如果无法直接访问 Maven，运行以下命令获取最新版本。
        `latest` 标签返回的版本即 Google Mobile Ads SDK 的最新版本：

        ```bash
        curl -sS https://dl.google.com/dl/android/maven2/com/google/android/libraries/ads/mobile/sdk/ads-mobile-sdk/maven-metadata.xml | sed -n 's/.*<latest>\(.*\)<\/latest>.*/\1/p'
        ```

    -   [ ] 配置 `minSdk`（24+）和 `compileSdk`（34+）。
    -   [ ] 在应用级构建文件中全局排除所有依赖中的 `play-services-ads` 和
        `play-services-ads-lite`，避免符号重复错误。
    -   [ ] 同步 Gradle 后再继续下一步。

2.  **设置应用标识符**：
    -   [ ] 在 `InitializationConfig.Builder` 中以编程方式传入 `applicationId`
        参数，使用示例 AdMob App ID
        `ca-app-pub-3940256099942544~3347511713`。提醒用户在发布前替换为实际的
        AdMob App ID。

3.  **初始化 SDK**：
    -   [ ] **务必**在后台线程初始化 SDK。

4.  **验证集成**：
    -   [ ] 运行 `gradle build -x test` 确认构建成功且无错误。
        解决所有 GMA SDK 相关的编译错误。

### 链接

其他文档：

-   https://developers.google.com/admob/android/next-gen/quick-start.md.txt?utm_source=agent-skills&utm_medium=content&utm_campaign=adr-ss-ai&utm_content=google-mobile-ads-get-started