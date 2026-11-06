# IMA SDK HTML5 iframe 模式

本指南介绍在只能在 iframe 内使用 Google IMA SDK for HTML5 时的最佳实践。

## 优先使用顶层页面而非 iframe

应尽量在顶层页面添加 IMA SDK 脚本。优先在页面层级使用，而非在 iframe 内使用 IMA SDK。

### 同源 iframe 回退方案

如果检测到 IMA SDK 脚本位于 iframe 内，请确保包含 IMA SDK 脚本的 iframe、容器页面和视频播放器都位于同一域名下。

例如：在 iframe 内检查能否访问顶层窗口的 location。

```typescript
function hasAccessToTopWindow(): boolean {
  try {
    // 1. Check if the current context is already the top window.
    if (window === window.top) {
      return true;
    }

    // 2. Attempt a same-origin read on the top window location.
    // If cross-origin limits apply, this read will throw a DOMException.
    const topUrl = window.top.location.href;
    return true;
  } catch (error) {
    // 3. A SecurityError DOMException was thrown.
    // When this error happens, the SDK is isolated in a cross-origin iframe
    // and does not have direct access to the top-level window.
    const errorMessage = error instanceof Error ? error.message : String(error);
    console.error("Access to top-level window is blocked: ", errorMessage);
    return false;
  }
}

if (hasAccessToTopWindow()) {
  console.log("IMA SDK has access to the top-level window.");
} else {
  console.log("IMA SDK is isolated in a cross-origin iframe sandbox.");
}
```

SDK 需要这种同源设置才能直接访问 `window.top` 层级和视频播放器元素。

### 跨源 iframe 变通方案

如果检测到 IMA SDK 加载在 iframe 内，且该 iframe 的域名与包含视频播放器的主页面域名不同，必须通过设置 `adsRequest.pageUrl` 属性向 SDK 提供主页面的 URL。

示例代码：

```typescript
const adsRequest: google.ima.AdsRequest = new google.ima.AdsRequest();
adsRequest.adTagUrl = 'YOUR_AD_TAG_URL';

// Manually set the page URL to the parent page's URL
adsRequest.pageUrl = 'https://<your_domain>/<path_to_your_page>';
```

检查加载 IMA SDK 的 iframe 的 `sandbox` 属性，授予以下必需能力：

*   添加 `allow-scripts` 以执行 SDK JavaScript。缺少此项，IMA SDK 将无法加载和初始化。
*   添加 `allow-same-origin` 以允许加载的页面读取自身的 cookie。
*   添加 `allow-popups`、`allow-popups-to-escape-sandbox`、`allow-top-navigation-by-user-activation` 以允许 SDK 打开广告主的落地页。

```
<iframe sandbox="allow-scripts allow-same-origin allow-popups allow-popups-to-escape-sandbox allow-top-navigation-by-user-activation" src="PAGE_CONTAINING_IMA_SDK_SCRIPT_TAG">
</iframe>
```