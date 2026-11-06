# 广告预加载验证检查

-   **按以下步骤操作**：
    -   [ ] 搜索对应平台的 GMA SDK 广告预加载调用：

        | API | Android | iOS | Unity |
        | :--- | :--- | :--- | :--- |
        | 开始预加载 | `startPreload` | `preload` | `Preload` |
        | 停止预加载 | `destroyAll` | `stopPreloadingAndRemoveAllAds` | `DestroyAll` |
        | 轮询广告 | `pollAd` | `ad(withPreloadID:)` | `DequeueAd` |
        | 广告已预加载 | `onAdPreloaded` | `adAvailable(forPreloadID:)` | `OnAdPreloaded` |
        | 广告已耗尽 | `onAdsExhausted` | `adsExhausted(forPreloadID:)` | `OnAdsExhausted` |
        | 广告可用性检查 | `isAdAvailable` | `isAdAvailable(withPreloadID:)` | `IsAdAvailable` |

-   **通过/失败标准**：
    -   **失败**：项目中检测到以下任一不当的广告预加载模式：
        -   在同一方法作用域内轮询广告但未展示。
        -   在广告已预加载或广告已耗尽回调中开始预加载。
        -   在广告已预加载或广告已耗尽回调中停止预加载。
        -   未先停止预加载器，就对同一预加载 ID 多次开始预加载。
        -   使用空字符串或 null 作为预加载 ID。
        -   在 **Unity** 的 `Update` 循环中检查广告可用性。
    -   **警告**：
        -   为同一广告格式预加载多个广告单元 ID。
    -   **不适用**：项目未使用广告预加载 API。