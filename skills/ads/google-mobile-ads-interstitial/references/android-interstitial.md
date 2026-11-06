# 插页式广告的 AI 集成代理说明

## 必需的导入

使用以下导入实现插页式广告：

```
import com.google.android.libraries.ads.mobile.sdk.common.AdRequest
import com.google.android.libraries.ads.mobile.sdk.common.AdLoadCallback
import com.google.android.libraries.ads.mobile.sdk.common.FullScreenContentError
import com.google.android.libraries.ads.mobile.sdk.common.LoadAdError
import com.google.android.libraries.ads.mobile.sdk.interstitial.InterstitialAd
import com.google.android.libraries.ads.mobile.sdk.interstitial.InterstitialAdEventCallback
```

## 方法签名

使用以下方法签名实现插页式广告：

`AdRequest` 对象接收广告单元 ID：

```
AdRequest.Builder(adUnitId: String).build()
```

从 `InterstitialAd` 加载广告的方法签名：

```
fun load(
    adRequest: AdRequest,
    adLoadCallback: AdLoadCallback<InterstitialAd>
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

**UI 线程**：**强制要求**：GMA-Next Gen SDK 中的回调在后台线程上调用。在 GMA SDK 回调中，**所有与 UI 相关的操作**（例如 Toast、View 更新、Fragment 事务）**必须**包裹在 `runOnUiThread {}` 或 `Dispatchers.Main.launch {}` 中。跳过此步骤将导致应用崩溃。

## 插页式广告工作流

1.  **加载广告**

2.  **注册广告事件回调**
    -   [ ] 在 `InterstitialAd` 对象上设置 `adEventCallback`。
        *   当广告被关闭或展示失败时，释放对插页式广告的引用。

3.  **展示广告**

4.  **验证实现**：验证构建以确保没有编译错误：
    -   [ ] 运行 `gradle build -x test` 确认构建成功且无错误。解决任何与 GMA SDK 相关的编译错误。

### 链接

附加文档：

-   https://developers.google.com/admob/android/next-gen/interstitial.md.txt