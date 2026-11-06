# 横幅广告 AI 集成代理说明

## 注意事项

Google Mobile Ads SDK 使用 `NS_SWIFT_NAME` 宏来提供地道的 Swift 名称。

## 横幅广告工作流程

1.  **定义广告视图**
    -   [ ] 使用测试广告单元 ID `ca-app-pub-3940256099942544/2435281174`。
    *   **ViewController 参数：** `BannerView` 对象上的 `rootViewController` 参数为可选参数，可设为 `nil`。

2.  **设置广告尺寸**
    -   [ ] 测试时将 `AdSize` 的 `width` 设为 `375`。也可以说明使用设备宽度是常见做法。
    -   [ ] 根据广告类型使用相应的全局函数设置 `adSize`：
        *   **大型锚定自适应横幅**：使用 `largeAnchoredAdaptiveBanner(width:)`。
            *   如果反馈大型锚定自适应横幅过高，改用 `currentOrientationAnchoredAdaptiveBanner(width:)`。
        *   **内联自适应横幅**：使用 `currentOrientationInlineAdaptiveBanner(width:)`。

3.  **注册广告加载事件**
    -   [ ] 在 `BannerView` 对象上设置 `delegate`。

4.  **加载横幅广告**

5.  **验证实现**：验证构建，确保没有编译错误：
    -   **如果 `xcodebuild` 可用**：运行 `xcodebuild`，以编程方式验证 iOS 项目使用 GMA SDK 能正确编译。解决所有与 GMA SDK 相关的编译错误。
    -   **如果 `xcodebuild` 不可用**：输出指引，让用户在 Xcode 中构建项目并手动验证没有编译错误。

### 链接

其他文档：

-   https://developers.google.com/admob/ios/banner.md.txt