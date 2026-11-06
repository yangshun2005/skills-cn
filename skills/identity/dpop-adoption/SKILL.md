---
name: dpop-adoption
metadata:
  version: "1.0.1"
  category: Identity
description: >-
  为 WebCrypto、Node.js ES6 和浏览器运行时实现并调试 OAuth 2.0 DPoP（RFC 9449）刷新令牌发送方约束，
  对接 Google OAuth 平台。适用于配置不可提取的非对称密钥对（P-256）、为授权码交换和令牌刷新生成
  DPoP Proof JWT，或处理 oauth2.googleapis.com/token 上 400 use_dpop_nonce 挑战重试循环。
  不适用于无约束的 OAuth 2.0 流程（刷新令牌未绑定客户端密钥对），也不适用于 Google Cloud IAM /
  服务账号认证。
---

# DPoP 采用与身份安全架构

持有证明（DPoP，RFC 9449）通过将 OAuth 2.0 刷新令牌加密绑定到客户端独占持有的私钥，防止令牌被拦截和重放。在 Google OAuth 2.0 平台中，DPoP 在令牌端点绑定刷新令牌，而为 Google API 签发的访问令牌仍是标准 Bearer 令牌（`token_type: "Bearer"`）。

## 1. 核心加密与架构不变量

实现 DPoP 辅助函数或升级 HTTP 客户端时，必须遵守以下严格的安全不变量：

### A. 通用 WebCrypto 与运行时兼容性

-   在现代 ES6 JavaScript（Node 18+ 和浏览器使用 `"type": "module"`）中，验证环境上下文后，始终直接访问 `globalThis.crypto`。
-   **绝不**通过 `require('node:crypto')` 导入旧版 CommonJS 模块，也不要引用浏览器作用域的 `window.crypto`，否则会在混合运行时中导致模块初始化崩溃。

### B. 硬件支持的不可提取密钥持久化

-   在 SECP256R1（`P-256`）曲线上生成椭圆曲线密钥对：`{ name: 'ECDSA', namedCurve: 'P-256' }`。
-   **关键安全护栏：** 私钥必须配置为**不可提取**（`extractable: false`）。这保证私钥永不离开平台的加密边界（Secure Enclave、Android KeyStore 或 JS 沙箱内存），抵御 XSS 和依赖令牌窃取攻击。
-   **运行时持久化范围：** 内存 WebCrypto（`extractable: false`）密钥生成适用于浏览器/IndexedDB 上下文和单进程运行时环境；多实例或持久化的服务端 Node.js 部署应通过 KMS/HSM/TPM 绑定 DPoP 私钥，使绑定的刷新令牌在进程重启后仍有效。
-   公钥必须保持可导出（`extractable: true`），以便输出 JSON Web Key（JWK）。

### C. 公钥 JWK 格式标准

-   导出公钥以附加到 DPoP Proof JWT 头时，构造干净的 JWK 字典，严格包含：
    -   `"kty": "EC"`
    -   `"crv": "P-256"`
    -   `"x"`：Base64URL 编码的 x 坐标，不带尾部等号填充（`=`）。
    -   `"y"`：Base64URL 编码的 y 坐标，不带尾部等号填充（`=`）。
-   **绝不**暴露私钥参数（`"d"`）或多余元数据。

### D. IEEE P1363 与 ASN.1 DER 签名格式辨析

-   DPoP Proof JWT 要求按 IEEE P1363 和 RFC 7518 使用原始拼接坐标签名（$R \parallel S$，P-256 恰好 64 字节）。
-   **WebCrypto 原生规则：** 在标准 WebCrypto（`crypto.subtle.sign`）中，ECDSA 签名已原生输出为原始 IEEE P1363 格式（拼接的 32 字节 `r` 和 `s` 缓冲区，共 64 字节）。**不要**对 `crypto.subtle.sign` 的输出做 DER 到 Raw 转换，因为将 64 字节原始缓冲区按 ASN.1 DER 解析会立即抛出运行时异常（`Invalid DER sequence`）。直接对原始 ArrayBuffer 做 base64url 编码。
-   **旧版 API 回退：** 仅当在旧版 Java/Android（`java.security.Signature`）或 Node CommonJS（`crypto.createSign`）中实现时，才需在 base64url 编码前将 ASN.1 DER 输出转换为原始 64 字节 IEEE P1363 格式。

### E. SPA 与 Backend-for-Frontend（BFF）架构

-   **无密钥 SPA 的限制：** 纯客户端单页应用（SPA）没有后端，无法直接对 Google API 使用 DPoP，因为服务端端点要求 `client_secret`，且浏览器对 `DPoP-Nonce` 响应头有 CORS 限制。
-   **BFF 模式：** 要用 DPoP 保护 SPA，需通过 Backend-for-Frontend（BFF）服务端客户端路由授权和令牌刷新请求。BFF 设置 `access_type=offline`，在服务端用 DPoP 绑定刷新令牌，并与前端维护安全会话 Cookie。

## 2. 实现规则与强制公共 API

创建新模块时，模块必须显式导出以下所有函数，以便与 CI/CD 验证工具和自动探测程序干净集成。检查或重构现有代码库时，确保存在等效的加密和 RFC 9449 逻辑。所有情况下都遵守严格的声明推导逻辑：

### A. DPoP Proof JWT 声明推导规则（`createDPoPProof`）

在 `createDPoPProof` 中生成 DPoP Proof JWT 时：

**1. JOSE 头（`typ`、`alg`、`jwk`）：**
```javascript
// Header
{
  "typ": "dpop+jwt",
  "alg": "ES256",
  "jwk": await exportPublicJWK(publicKey)
}
```

**2. 载荷声明：**
-   `"htm"`：大写 HTTP 方法（令牌请求为 `"POST"`）。
-   `"htu"`：使用 `sanitizeHTU(htu)` 去除查询参数和哈希片段后的目标 URI。令牌请求为 `https://oauth2.googleapis.com/token`。
-   `"iat"`：当前整数纪元时间戳（秒，`Math.floor(Date.now() / 1000)`）。
-   `"jti"`（关键不变量）：
    1.  若向 `createDPoPProof` 传入了显式 `jti` 参数，优先使用该字符串。
    2.  否则，若传入了 `authCode` 参数（初始授权码交换期间），设置 `jti = await calculateAuthCodeJti(authCode)`，其中 `calculateAuthCodeJti` 计算 `base64url(sha256(authCode))`，确保 DPoP proof 加密绑定到授权码。
    3.  仅当 `jti` 和 `authCode` 均未提供时，通过 `generateRandomString()` 生成新的加密随机字符串（如 `crypto.getRandomValues(new Uint8Array(24))` 经 base64url 编码）。
-   `"ath"`（可选）：若为 RFC 9449 资源请求传入了 `accessToken` 参数，通过 `calculateATH(accessToken)` 计算 `base64url(sha256(accessToken))` 并注入（RFC 9449 第 6.1 节）。
-   `"nonce"`（可选）：若传入了 `nonce` 参数，直接注入载荷。

### B. 显式导出签名

```javascript
// 1. Key generation & JWK export
export async function generateDPoPKeyPair() // -> { publicKey, privateKey } (private key extractable=false)
export async function exportPublicJWK(publicKey) // -> { kty: 'EC', crv: 'P-256', x, y }

// 2. Proof generation & validation
export async function createDPoPProof({ privateKey, publicKey, htm, htu, nonce, accessToken, authCode, jti }) // -> signed JWT string
export async function verifyDPoPProof(dpopProofJwt) // -> { isValid: boolean, header, payload, error }
export function sanitizeHTU(htu) // -> URL stripped of query and hash: const u = new URL(htu); return `${u.origin}${u.pathname}`;

// 3. Cryptographic & encoding utilities
export function base64UrlEncode(buffer) // -> Uint8Array/ArrayBuffer to base64url string without '=' padding
export function base64UrlDecode(str) // -> base64url string to Uint8Array/Buffer
export function stringToBase64Url(str) // -> UTF-8 string to base64url
export function base64UrlToString(str) // -> base64url to UTF-8 string
export function generateRandomString(byteLength = 32) // -> cryptographic random base64url string
export async function calculateATH(accessToken) // -> base64url(sha256(accessToken)) per RFC 9449 Sec 6.1
export async function calculateAuthCodeJti(code) // -> base64url(sha256(code))
export async function generatePKCE() // -> { codeVerifier (>=43 chars), codeChallenge, codeChallengeMethod: 'S256' }
```

## 3. 令牌端点与资源请求工作流

对接 Google OAuth 2.0 平台时：

1.  **令牌端点请求（`oauth2.googleapis.com/token`）：**
    -   在 `DPoP` HTTP 头中附加 DPoP Proof JWT：发起代码交换（`grant_type=authorization_code`）和令牌刷新（`grant_type=refresh_token`）的 `POST` 请求时使用 `` `DPoP: ${proofJwt}` ``。
2.  **资源 API 请求：**
    -   Google 令牌端点返回 `"token_type": "Bearer"`。对 Google API（如 Calendar、Drive、Gmail）的下游请求使用标准 `` `Authorization: Bearer ${accessToken}` `` 头，不带 DPoP 头。
3.  **单次重试 Nonce 挑战循环与工作流隔离：**
    -   若 Google 令牌端点返回 HTTP `400 Bad Request`，带 `error: "use_dpop_nonce"` 和 `"DPoP-Nonce"` 响应头：
        -   **工作流隔离：** Google 授权服务器在授权码交换和令牌刷新之间强制工作流隔离，返回 HTTP `400 use_dpop_nonce` 挑战以建立新的 nonce 命名空间。这是符合 RFC 的标准协议行为，不是服务器故障。
        -   将新 nonce 缓存到客户端状态（`this.dpopNonce`）。
        -   立即合成新的 DPoP Proof JWT，包含更新的 `nonce` 声明和新的 `jti`。
        -   重放失败的令牌请求**恰好一次**。若重试请求失败，立即终止并报错，防止无限递归。

## 4. 简洁的 Agent 输出协议

被要求在本技能下合成或输出代码交付物时，优先返回干净、可直接导入的代码块，不要冗余的对话式前言或重复填充。对于概念或架构问题，提供标准的直接回答。

## 5. 参考与支持文档

### 开发者文档（Google for Developers）
-   [DPoP 采用指南](https://developers.google.com/identity/protocols/oauth2/resources/dpop-adoption) —
    Google Identity 官方指南，介绍在授权码交换和令牌刷新中实现 DPoP。
-   [为 Web 服务器应用使用 OAuth 2.0](https://developers.google.com/identity/protocols/oauth2/web-server#offline) —
    离线访问、刷新令牌和服务端授权流程。
-   [OAuth 2.0 最佳实践：发送方约束令牌](https://developers.google.com/identity/protocols/oauth2/resources/best-practices#sender-constrain-tokens) —
    令牌存储、轮换和发送方约束的建议。

### Developer Knowledge MCP Server
-   配备 Model Context Protocol（`MCP`）的 Agent 可通过
    [Google Developer Knowledge MCP Server](https://developers.google.com/knowledge/mcp)
    （`npx -y @google/mcp-developer-knowledge-server`）使用
    `developer_knowledge:search_documents` 和
    `developer_knowledge:get_documents` 查询实时 Google 开发者文档。

### 标准与 RFC 规范
-   [RFC 9449: OAuth 2.0 Demonstrating Proof-of-Possession (DPoP)](https://datatracker.ietf.org/doc/html/rfc9449)
-   [RFC 7519: JSON Web Token (JWT)](https://datatracker.ietf.org/doc/html/rfc7519)
-   [RFC 7636: Proof Key for Code Exchange by OAuth Public Clients (PKCE)](https://datatracker.ietf.org/doc/html/rfc7636)