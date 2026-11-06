# 插页式广告 AI 集成代理说明

## 注意事项

Google Mobile Ads SDK 使用 `NS_SWIFT_NAME` 宏来提供符合 Swift 习惯的名称。

## 插页式广告工作流

1.  **加载广告**
    *   对于 Swift 代码，使用 `async/await` 而非完成处理器。

2.  **注册广告事件回调**
    -   [ ] 在 `InterstitialAd` 对象上设置 `fullScreenContentDelegate`。
        *   广告关闭或展示失败时，释放对插页式广告的引用。

3.  **展示广告**
    *   **ViewController 参数：** `present(from:)` 方法中的 `rootViewController` 参数为可选参数，可设为 `nil`。

4.  **验证实现**：验证构建以确保没有编译错误：
    -   **如果 `xcodebuild` 可用**：运行 `xcodebuild` 以编程方式验证 iOS 项目能否在 GMA SDK 下正确编译。解决所有与 GMA SDK 相关的编译错误。
    -   **如果 `xcodebuild` 不可用**：输出指引，指导用户在 Xcode 中构建项目并手动验证没有编译错误。

### 链接

附加文档：

-   https://developers.google.com/admob/ios/interstitial.md.txt