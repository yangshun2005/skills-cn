---
name: sign-in-with-google-web
metadata:
  category: Identity
  version: "1.0.0"
description: >-
  在各类 Web 架构中，使用 Google Identity Services（GIS / `https://accounts.google.com/gsi/client`）
  实现、配置并保护 Google 登录（SiwG）。适用于：创建 Google 登录按钮、通过 FedCM 实现 Google One Tap、
  在 React/Next.js/Angular/HTML 中集成 GIS、在后端运行时（Python/Node.js/Go/Java）验证 ID Token、
  强制执行 Google Workspace 域名限制（`hd`）、通过 WebCrypto nonce 或加密 IndexedDB 保护客户端 ID Token、
  通过 Intermediate Iframe API（`https://accounts.google.com/gsi/intermediate`）嵌入跨域 iframe、
  配置内容安全策略（CSP）、COOP 和 Permissions Policy（`identity-credentials-get`）响应头，
  或处理登出与授权撤销。不适用于旧版 Google Sign-In（`gapi.auth2`）、Google Cloud IAM 服务账号，
  或 Android/iOS 原生 Credential Manager。
---

# Google 登录（SiwG）集成与安全架构

本技能提供通过 Google Identity Services（GIS）Web SDK（`https://accounts.google.com/gsi/client`）集成 Google 登录的规范性架构指南、安全实现契约和故障预防协议。

## 1. IETF 架构分类与层级（优先推荐 BFF / TMB）

所有 Google 登录实现必须遵循 [IETF OAuth 2.0 for Browser-Based Applications（`draft-ietf-oauth-browser-based-apps`）](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps#name-application-architecture-pa) 分类标准。

> [!IMPORTANT] **首要建议：** 只要应用有后端服务器，始终推荐后端中介模式（**模式 A：Backend For Frontend** 或 **模式 B：Token 中介后端 / 重定向模式**）作为最安全、最健壮、符合行业标准的架构。仅在受限于无服务器静态托管或离线优先 PWA 需求时，才回退到纯客户端浏览器模式（模式 C 和 D）。

<!-- mdformat off(reason: preserve ASCII architecture diagram width) -->
```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                          IETF 架构层级与选择                                             │
├──────────────────────────┬──────────────────────────┬───────────────────────────────────┤
│ 模式 A（推荐）：          │ 模式 B（推荐）：          │ 模式 C 和 D（回退方案）：          │
│ Backend For Frontend     │ Token 中介后端            │ 基于浏览器的 OAuth 客户端          │
│ （BFF - IETF § 6.1）     │ （TMB / 重定向 § 6.2）    │ （纯 JavaScript - IETF § 6.3）    │
├──────────────────────────┼──────────────────────────┼───────────────────────────────────┤
│ 🏆 黄金标准              │ 🚀 原生表单重定向         │ ⚡ 静态 SPA / 💾 离线 PWA         │
│ • 全栈 SPA + API         │ • 服务端渲染应用          │ • 纯客户端或离线 PWA              │
│ • 客户端 JS fetch 调用 API│ • GIS HTTP POST login_uri│ • C：临时内存闭包                 │
│ • HttpOnly 会话 Cookie   │ • 直接服务端重定向        │ • D：WebCrypto 加密 IndexedDB     │
│ • Token 永不进入浏览器    │ • Token 永不进入 JS       │ • 严格 WebCrypto nonce / 密钥     │
└──────────────────────────┴──────────────────────────┴───────────────────────────────────┘
```
<!-- mdformat on -->

### 🏆 模式 A：Backend For Frontend（BFF - IETF § 6.1）— 强烈推荐

*   **为何是黄金标准**：ID Token（JWT）和访问令牌在页面刷新后永远不会暴露给浏览器 JavaScript。这提供了对 XSS Token 窃取的完整架构免疫。

*   **架构**：浏览器前端加载 GIS SDK，在 JavaScript 回调（`callback: handleCredentialResponse`）中捕获凭据，并立即通过 `fetch('/api/auth/google', { method: 'POST' })` 将 ID Token（JWT）转发给后端。

*   **后端职责**：后端使用 Google 公钥 JWK（`google.oauth2.id_token.verify_oauth2_token`）验证 JWT 加密签名，对照服务端会话状态检查 `aud`、`hd` 和 `nonce` 声明，创建服务端会话，并签发第一方 `HttpOnly; Secure; SameSite=Lax` 会话 Cookie。

*   完整实现见 [references/bff_fastapi_verification.md](references/bff_fastapi_verification.md)。

### 🚀 模式 B：通过 `login_uri` 重定向的 Token 中介后端（IETF § 6.2）— 服务端渲染与表单 POST

*   **为何高度安全**：通过指示 GIS 直接向后端的 `login_uri` 执行 HTTP POST，完全绕过客户端 JavaScript 凭据处理。

*   **架构**：通过 `google.accounts.id.initialize({ client_id, login_uri: "https://example.com/api/auth/callback", ux_mode: "redirect" })` 或 HTML 属性（`data-login_uri="https://example.com/api/auth/callback"` 和 `data-ux_mode="redirect"`）配置。

*   **后端职责**：后端以表单 POST 体（`credential` 参数）接收 ID Token，对照 `g_csrf_token` POST 体字段验证双重提交的 `g_csrf_token` Cookie，在服务端加密验证 ID Token，建立会话 Cookie，并返回标准 HTTP 302/303 重定向。

*   完整实现见 [references/tmb_login_uri_redirect.md](references/tmb_login_uri_redirect.md)。

### 模式 C：基于浏览器的 OAuth 客户端 — 临时内存（IETF § 6.3）— 静态 SPA 回退方案

*   **适用范围**：仅在部署严格为无服务器/静态（如 GitHub Pages、Firebase 静态托管）且无后端组件时使用。

*   **存储不变式**：ID Token 在活动标签页会话期间严格保存在私有 JavaScript 内存/闭包中。**切勿将原始 Token 存入 `localStorage` 或 `sessionStorage`**。

*   **会话续期**：使用 GIS One Tap / FedCM 自动选择（`auto_select: true`），在页面刷新时透明地将新的 ID Token 重新获取到内存中，无需持久化浏览器存储。

*   **XSS 与重放保护**：通过 `window.crypto.getRandomValues()` 生成加密 `nonce`，并直接传给 `google.accounts.id.initialize({ client_id, nonce: clientNonce })`。在信任内存中的声明之前，验证 `payload.nonce === clientNonce`。

*   完整实现见 [references/in_memory_spa_nonce.md](references/in_memory_spa_nonce.md)。

### 模式 D：基于浏览器的 OAuth 客户端 — WebCrypto 加密 IndexedDB（IETF § 6.3）— 离线 PWA 回退方案

*   **适用范围**：仅用于离线优先的渐进式 Web 应用（PWA），且用户认证凭证必须在断网时跨标签页刷新存活。

*   **存储不变式**：**切勿将原始明文 ID Token 存入 `localStorage`**。使用 WebCrypto `AES-GCM` 和不可导出会话密钥（`extractable: false`）加密 ID Token 载荷，然后将密文和 IV 写入 `IndexedDB`。

*   完整实现见 [references/offline_pwa_encryption.md](references/offline_pwa_encryption.md)。

--------------------------------------------------------------------------------

## 2. 强制安全指令与浏览器策略

为避免多轮修复循环和浏览器安全异常，在初始交付物中提供所有必需的 CSP、COOP 和用户激活指令：

### A. 内容安全策略（CSP）指令

```http
Content-Security-Policy:
  script-src 'self' https://accounts.google.com/gsi/client;
  frame-src https://accounts.google.com/gsi/;
  connect-src https://accounts.google.com/gsi/;
```

如果使用严格的基于 nonce 的 CSP，将 nonce 附加到 GIS script 标签：`<script src="https://accounts.google.com/gsi/client" async defer nonce="{{NONCE}}"></script>`。

### B. 跨域打开者策略（COOP）

为允许 GIS 弹窗对话框通过 `postMessage` 将凭据传回父窗口：

```http
Cross-Origin-Opener-Policy: same-origin-allow-popups
```

*（使用 `same-origin` 而不加 `allow-popups` 会破坏 GIS 弹窗，导致静默失败）。*

### C. 自动选择、FedCM 与登出生命周期（`disableAutoSelect`）

启用 Google One Tap 自动零点击返回登录时（GIS 默认启用 FedCM）：

*   在 `google.accounts.id.initialize({...})` 中设置 `auto_select: true`（不要传入已弃用的 `use_fedcm_for_prompt` 参数）。

*   **已弃用库警告**：切勿将已弃用的 `gapi.auth2`（`gapi.auth2.init`、`gapi.auth2.getAuthInstance().signOut()`）与 Google Identity Services（GIS）混用。`gapi.auth2` 已完全退役；仅使用 GIS 方法。

*   **登出协议**：当用户显式登出应用时，**必须**调用 `google.accounts.id.disableAutoSelect()`：

    ```javascript
    function handleUserLogout() {
      // 1. 禁用自动 One Tap / FedCM 重新认证
      google.accounts.id.disableAutoSelect();

      // 2. 清除应用会话状态 / 服务端 Cookie
      fetch('/api/auth/logout', { method: 'POST' }).then(() => {
        window.location.href = '/login';
      });
    }
    ```

*   *（未调用 `disableAutoSelect()` 会导致用户主动登出后，下次访问页面时立即自动重新登录循环）。*

### D. 可靠的动态脚本加载与框架生命周期（React、Next.js、SPA）

在单页应用（React、Next.js、Vue、Angular）中挂载 Google 登录时：

*   现代 SPA 异步执行组件生命周期。在 `<script src="https://accounts.google.com/gsi/client">` 加载完成前访问 `window.google.accounts.id` 会导致 `ReferenceError: google is not defined`。

*   **切勿使用 `gapi` 或轮询（`setInterval`）**：使用确定性的动态脚本加载，配合 `onload` / `addEventListener('load')` 事件监听器。

*   **动态脚本加载器模式**：

    ```tsx
    import React, { useEffect, useState } from 'react';

    export function useGoogleIdentityScript() {
      const [isLoaded, setIsLoaded] = useState(false);

      useEffect(() => {
        if (window.google?.accounts?.id) {
          setIsLoaded(true);
          return;
        }

        const existingScript = document.querySelector(
          'script[src="https://accounts.google.com/gsi/client"]'
        );
        if (existingScript) {
          existingScript.addEventListener('load', () => setIsLoaded(true));
          return;
        }

        const script = document.createElement('script');
        script.src = 'https://accounts.google.com/gsi/client';
        script.async = true;
        script.defer = true;
        script.onload = () => setIsLoaded(true);
        document.head.appendChild(script);
      }, []);

      return isLoaded;
    }
    ```

*   *（始终使用确定性的 `onload` 事件监听器或框架的 `<Script strategy="afterInteractive">`，而非脆弱的 `setInterval` 轮询）。*

### E. 嵌入式 Iframe 与 Permissions Policy（`identity-credentials-get`）

在跨域 `<iframe>` 元素或小组件集成中嵌入 Google 登录或 One Tap 时，现代浏览器安全模型和 FedCM 要求容器框架显式委托 Permissions Policy：

```html
<iframe
  src="https://example.com/embed"
  allow="identity-credentials-get 'src' https://accounts.google.com">
</iframe>
```

*（省略 `allow="identity-credentials-get"` 会阻止嵌入式或跨域上下文中的 FedCM / One Tap 初始化）。* 完整集成指南见 [references/intermediate_iframe.md](references/intermediate_iframe.md)。

--------------------------------------------------------------------------------

## 3. 实现契约与反模式禁令

### A. 严格禁止旧版 GAPI（`gapi.auth2`）

*   **切勿**导入或引用 `gapi.auth2`、`gapi.auth2.init`、`gapi.auth2.getAuthInstance()` 或 `gapi.signin2`。始终警告 `gapi.auth2` 已弃用并已停用。

*   GIS（`google.accounts.id.*` 和 `google.accounts.oauth2.*`）完全无状态。它替代所有旧版 GAPI 认证库。

--------------------------------------------------------------------------------

## 4. 实现配方（渐进式披露）

生成实现代码时，加载与目标架构匹配的特定参考文件：

*   **配方 1 — 模式 A（Python / FastAPI 的 Backend For Frontend）[推荐]**：加载 [references/bff_fastapi_verification.md](references/bff_fastapi_verification.md)，了解自定义 UI 按钮提示、`google.oauth2.id_token.verify_oauth2_token` 加密验证、`nonce` 重放检查、Google Workspace `hd` 域名强制以及 `HttpOnly; Secure; SameSite=Lax` 会话 Cookie 签发。

*   **配方 2 — 模式 B（通过 `login_uri` 表单 POST 重定向的 Token 中介后端）[推荐]**：加载 [references/tmb_login_uri_redirect.md](references/tmb_login_uri_redirect.md)，了解 `data-login_uri` / `ux_mode: 'redirect'` 前端设置以及 FastAPI 双重提交 `g_csrf_token` Cookie 与表单体验证。

*   **配方 3 — 模式 C（使用原生 GIS Nonce 的基于浏览器临时内存 SPA）[回退]**：加载 [references/in_memory_spa_nonce.md](references/in_memory_spa_nonce.md)，了解 WebCrypto `nonce` 生成（`google.accounts.id.initialize({ client_id, nonce: clientNonce })`）、UTF-8 安全的 Base64URL JWT 解码以及私有闭包 Token 管理。

*   **配方 4 — 模式 D（用于离线 PWA 的基于浏览器 WebCrypto 加密 IndexedDB）[回退]**：加载 [references/offline_pwa_encryption.md](references/offline_pwa_encryption.md)，了解不可导出（`extractable: false`）`AES-GCM` `CryptoKey` 生成和加密 `IndexedDB` 存储。

*   **配方 5 — 嵌入式跨域上下文（`gsi/intermediate`）**：加载 [references/intermediate_iframe.md](references/intermediate_iframe.md)，了解 `<div id="g_id_intermediate_iframe">`、`allow="identity-credentials-get"` 以及严格的 `window.postMessage` 来源验证。

--------------------------------------------------------------------------------

## 5. 所有模式下的登出与账号授权撤销

```javascript
export function handleSignOut(userEmail) {
  // 1. 禁用客户端 One Tap 自动选择
  google.accounts.id.disableAutoSelect();

  // 2. 如果用户断开账号连接，撤销 OAuth 授权
  if (userEmail) {
    google.accounts.id.revoke(userEmail, () => {
      console.log('User grant revoked');
    });
  }

  // 3. 清除存储 / 会话
  AuthManager.clear(); // 模式 C
  fetch('/api/auth/logout', { method: 'POST' }); // 模式 A
}
```

--------------------------------------------------------------------------------

## 6. 参考与规范标准

*   [RFC 7519: JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519) — 标准声明（`iss`、`sub`、`aud`、`exp`、`iat`、`nonce`）、格式和处理规则。

*   [RFC 7515: JSON Web Signature (JWS)](https://datatracker.ietf.org/doc/html/rfc7515) — 对照 Google 公钥 JWK 证书（`https://www.googleapis.com/oauth2/v3/certs`）的加密签名验证。

*   [RFC 4648 § 5: Base64url 编码](https://datatracker.ietf.org/doc/html/rfc4648#section-5) — JWT 各段使用的无填充 URL 安全 Base64 编码。

*   [IETF OAuth 2.0 基于浏览器的应用](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-browser-based-apps) — 规范性架构标准（§ 6.1 BFF、§ 6.2 TMB、§ 6.3 客户端）。

*   [Google Identity Services Web 参考](https://developers.google.com/identity/gsi/web/reference/js-reference) — 官方 GIS API 参考。

*   [Intermediate Iframe API 参考](https://developers.google.com/identity/gsi/web/amp/nonamp-reference) — 跨域 iframe 嵌入指南。

*   [服务端验证 ID Token](https://developers.google.com/identity/gsi/web/guides/verify-google-id-token) — 服务端 Token 验证协议。

*   [W3C 联邦凭据管理 API（FedCM）](https://www.w3.org/TR/fedcm-1/) — 浏览器身份凭据中介标准。

*   [W3C Web 加密 API](https://www.w3.org/TR/WebCryptoAPI/) — 不可导出密钥生成（`extractable: false`）和 AES-GCM 标准。