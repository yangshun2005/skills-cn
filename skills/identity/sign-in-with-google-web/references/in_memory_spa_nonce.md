# 模式 C：基于浏览器的临时内存 SPA 与原生 GIS Nonce

本参考文档提供 **模式 C（基于浏览器的 OAuth 客户端 — IETF `draft-ietf-oauth-browser-based-apps` § 6.3）** 的兜底实现方案，严格适用于无后端服务器的无服务静态单页应用（SPA）。

> [!WARNING] **客户端验证的局限：** 在浏览器中通过 `atob` / `TextDecoder` 解码 JWT 载荷，仅用于检查声明以进行 UI 展示和客户端 `nonce` 关联。它**不能**替代后端对 Google 公钥 JWK 的加密签名验证。只要存在后端，就应始终使用模式 A（BFF）或模式 B（TMB）。

--------------------------------------------------------------------------------

## 完整的 React + TypeScript 实现

```tsx
import React, { useEffect, useRef } from 'react';
import { useGoogleIdentityScript } from './useGoogleIdentityScript';

// 内存令牌管理器，支持安全的 UTF-8 Base64URL JWT 解码
export const AuthManager = (() => {
  let _idToken: string | null = null;
  let _user: Record<string, any> | null = null;

  return {
    setCredential: (jwt: string, expectedNonce: string) => {
      const base64Url = jwt.split('.')[1];
      const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
      const binaryStr = atob(base64);
      const bytes = Uint8Array.from(binaryStr, (c) => c.charCodeAt(0));
      const payload = JSON.parse(new TextDecoder().decode(bytes));

      if (payload.nonce !== expectedNonce) {
        throw new Error('Security violation: ID token nonce mismatch!');
      }
      _idToken = jwt;
      _user = payload;
    },
    getIdToken: () => _idToken,
    getUser: () => (_user ? { ..._user } : null),
    clear: () => {
      _idToken = null;
      _user = null;
    },
  };
})();

export const GoogleSignInButton: React.FC<{
  clientId: string;
  onSuccess: (user: Record<string, any>) => void;
}> = ({ clientId, onSuccess }) => {
  const buttonRef = useRef<HTMLDivElement>(null);
  const isGisLoaded = useGoogleIdentityScript();

  useEffect(() => {
    if (!isGisLoaded || !window.google?.accounts?.id) return;

    // 1. 生成 WebCrypto 随机 nonce
    const nonceBytes = window.crypto.getRandomValues(new Uint8Array(16));
    const clientNonce = Array.from(nonceBytes, (b) =>
      b.toString(16).padStart(2, '0')
    ).join('');

    window.google.accounts.id.initialize({
      client_id: clientId,
      nonce: clientNonce, // 直接绑定原生 nonce
      auto_select: true,
      callback: (response: { credential: string }) => {
        if (response.credential) {
          AuthManager.setCredential(response.credential, clientNonce);
          onSuccess(AuthManager.getUser()!);
        }
      },
    });

    if (buttonRef.current) {
      window.google.accounts.id.renderButton(buttonRef.current, {
        type: 'standard',
        theme: 'outline',
        size: 'large',
        text: 'signin_with',
        shape: 'rectangular',
      });
    }
  }, [isGisLoaded, clientId, onSuccess]);

  return <div ref={buttonRef} />;
};
```