# 中间 Iframe API 集成指南（`gsi/intermediate`）

当 Google One Tap 或 Sign In With Google 必须嵌入跨源 iframe 或嵌入式组件（顶层直接执行脚本受限）时，使用中间 Iframe 辅助模式（`https://developers.google.com/identity/gsi/web/amp/nonamp-reference`）。

## 1. 中间框架标记与脚本（发布方源上的 `intermediate.html`）

```html
<!DOCTYPE html>
<html>
<head>
  <!-- 专用中间辅助库 -->
  <script src="https://accounts.google.com/gsi/intermediate"></script>
</head>
<body>
  <div id="g_id_intermediate_iframe"
       data-src="https://publisher.example.com/onetap_inner.html">
  </div>
</body>
</html>
```

## 2. 中间 JavaScript 初始化与安全 PostMessage 中继

```javascript
// 在中间 iframe 辅助脚本内
window.google?.accounts?.id?.initialize({
  client_id: 'YOUR_CLIENT_ID.apps.googleusercontent.com',
  callback: (response) => {
    if (response.credential) {
      // 仅将凭证安全转发给受信任的顶层父窗口
      const trustedParentOrigin = 'https://publisher.example.com';
      window.parent.postMessage(
        { type: 'GSI_CREDENTIAL', credential: response.credential },
        trustedParentOrigin // 显式指定目标源，防止凭证外泄
      );
    }
  }
});
```

## 3. 顶层父框架事件监听器与严格源断言

```javascript
// 在顶层父窗口上
window.addEventListener('message', (event) => {
  // 1. 严格源校验 - 强制要求
  if (event.origin !== 'https://publisher.example.com') {
    return; // 拒绝不受信任的跨源消息
  }

  if (event.data?.type === 'GSI_CREDENTIAL') {
    const idToken = event.data.credential;
    // 通过模式 A（BFF 后端转发）或模式 C（内存中）处理令牌
    console.log('Received authenticated credential from intermediate iframe');
  }
});
```