# Android 横幅广告 AI 集成代理说明

## 必需的导入

使用以下导入实现横幅广告：

```
import com.google.android.libraries.ads.mobile.sdk.banner.AdSize
import com.google.android.libraries.ads.mobile.sdk.banner.AdView
import com.google.android.libraries.ads.mobile.sdk.banner.BannerAd
import com.google.android.libraries.ads.mobile.sdk.banner.BannerAdRequest
import com.google.android.libraries.ads.mobile.sdk.common.AdLoadCallback
import com.google.android.libraries.ads.mobile.sdk.common.LoadAdError
```

## 方法签名

使用以下方法签名实现横幅广告：

`BannerAdRequest` 对象接收广告单元 ID 和 `AdSize`：

```
BannerAdRequest.Builder(adUnitId: String, adSize: AdSize).build()
```

从 `AdView` 加载横幅广告的方法签名：

```
fun loadAd(
    adRequest: BannerAdRequest,
    adLoadCallback: AdLoadCallback<BannerAd>
): Unit
```

`AdLoadCallback` 接口定义如下：

```
interface AdLoadCallback<T> {
    fun onAdLoaded(ad: T): Unit
    fun onAdFailedToLoad(adError: LoadAdError): Unit
}
```

## 注意事项

**UI 线程**：**强制要求**：GMA-Next Gen SDK 中的回调在后台线程上调用。**所有 UI 相关操作**（例如 Toast、View 更新、Fragment 事务）**必须**在 GMA SDK 回调内用 `runOnUiThread {}` 或 `Dispatchers.Main.launch {}` 包裹。跳过此步骤将导致应用崩溃。

## 横幅广告工作流程

1.  **定义广告视图**
    -   [ ] 使用 `AdView` 定义 GMA Next-Gen SDK 横幅，而非已弃用的 `BannerAd`。

2.  **设置广告尺寸**：
    -   [ ] 测试时将 `AdSize` 的 `width` 设为 `360`。也可以说明使用设备宽度是常见做法。
    -   [ ] 根据广告类型使用合适的 `AdSize` 方法和测试广告单元 ID
        （`ca-app-pub-3940256099942544/9214589741`）：
        *   对于**大型锚定自适应横幅**：使用
            `AdSize.getLargeAnchoredAdaptiveBannerAdSize(context, width)`。
            *   如果反馈大型锚定自适应横幅过高，使用
                `AdSize.getCurrentOrientationAnchoredAdaptiveBannerAdSize(context, width)`。
        *   对于**内联自适应横幅**：使用
            `AdSize.getCurrentOrientationInlineAdaptiveBannerAdSize(context, width)`。
    -   [ ] 将 `AdSize` 和广告单元 ID 传给 `BannerAdRequest.Builder`，而非直接设在 `AdView` 上。

3.  **注册广告加载事件**：
    -   [ ] 在 `loadAd()` 方法中传入 `AdLoadCallback` 实现，而非直接设在 `AdView` 上。

4.  **加载横幅广告**：

5.  **验证实现**：检查构建以确保没有编译错误：
    -   [ ] 运行 `gradle build -x test` 确认干净构建成功。解决所有 GMA SDK 相关的编译错误。

### 链接

附加文档：

-   https://developers.google.com/admob/android/next-gen/banner.md.txt