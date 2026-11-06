# 激励广告 AI 集成代理说明

## 必需的导入

使用以下导入实现激励广告：

```
import com.google.android.libraries.ads.mobile.sdk.common.AdRequest
import com.google.android.libraries.ads.mobile.sdk.common.AdLoadCallback
import com.google.android.libraries.ads.mobile.sdk.common.FullScreenContentError
import com.google.android.libraries.ads.mobile.sdk.common.LoadAdError
import com.google.android.libraries.ads.mobile.sdk.rewarded.OnUserEarnedRewardListener
import com.google.android.libraries.ads.mobile.sdk.rewarded.RewardedAd
import com.google.android.libraries.ads.mobile.sdk.rewarded.RewardedAdEventCallback
```

## 方法签名

使用以下方法签名实现激励广告：

`AdRequest` 对象接收广告单元 ID：

```
AdRequest.Builder(adUnitId: String).build()
```

从 `RewardedAd` 加载广告的方法签名：

```
fun load(
    adRequest: AdRequest,
    adLoadCallback: AdLoadCallback<RewardedAd>
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

**UI 线程**：**强制要求**：GMA-Next Gen SDK 中的回调在后台线程调用。**所有 UI 相关操作**（例如 Toast、View 更新、Fragment 事务）**必须**在 GMA SDK 回调内用 `runOnUiThread {}` 或 `Dispatchers.Main.launch {}` 包裹。跳过此步骤将导致应用崩溃。

## 激励广告工作流

1.  **加载广告**

2.  **注册广告事件回调**
    -   [ ] 在 `RewardedAd` 对象上设置 `adEventCallback`。
        *   广告关闭或展示失败时，释放对激励广告的引用。

3.  **添加用于观看广告获取奖励的 UI 元素**
    *   激励广告**绝不能自动展示**。用户必须明确选择观看。
    *   添加一个 UI 元素（例如按钮），在用户交互时展示广告。

4.  **展示广告**
    *   `show()` 方法需要一个 `OnUserEarnedRewardListener`。

5.  **验证实现**：验证构建以确保没有编译错误：
    -   [ ] 运行 `gradle build -x test` 确认干净构建成功。解决所有 GMA SDK 相关的编译错误。

### 链接

附加文档：

-   https://developers.google.com/admob/android/next-gen/rewarded.md.txt