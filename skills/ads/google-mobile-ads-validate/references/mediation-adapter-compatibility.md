# 中介适配器兼容性

-   **请按以下步骤操作**：
    -   [ ] 分析项目构建文件中的依赖。
    -   [ ] 确定 GMA SDK 版本的声明方式：
        -   **显式声明**：
            -   **Android**：直接依赖
                `com.google.android.libraries.ads.mobile.sdk:ads-mobile-sdk`。
            -   **iOS**：直接依赖 `Google-Mobile-Ads-SDK`
                （CocoaPods）或 `GoogleMobileAds`（Swift Package Manager）。
            -   **Unity**：已安装 GMA Unity 插件，并在
                `GoogleMobileAdsDependencies.xml` 中或通过 UPM 包声明了
                Android/iOS GMA SDK 依赖。
        -   **传递依赖**：GMA SDK 由中介适配器依赖引入。
    -   [ ] 确定项目中所有中介适配器所需的 Android 和 iOS GMA SDK 版本。
    -   [ ] 将所有中介适配器要求的 GMA SDK 主版本和次版本，与项目中显式声明的 GMA SDK 版本进行对比。
-   **通过/失败标准**：
    -   **失败**：
        -   任何中介适配器要求的 GMA SDK 主版本高于或低于项目中的核心 GMA SDK 版本。
            -   *示例*：项目显式要求 GMA SDK 24.3.0，但某个适配器要求 GMA SDK 25.0.0（或 23.0.0）。
        -   各中介适配器之间要求的 GMA SDK 主版本不一致。
    -   **警告**：
        -   所有适配器依赖相同的 GMA SDK 主版本，但项目未显式声明 GMA SDK 版本，而是依赖传递依赖。
        -   任何中介适配器要求的 GMA SDK 次版本高于项目中声明的 GMA SDK 版本，但主版本相同。
            -   *示例*：项目中的 GMA SDK 为 13.6.0，但某个适配器要求 13.7.0。
    -   **不适用**：项目未使用任何中介适配器。