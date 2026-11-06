# 离线 PWA WebCrypto 加密 IndexedDB 存储架构

本指南详述离线优先渐进式 Web 应用（PWA）的模式 D：断网时用户身份验证凭证需在页面重载后仍可用，且不将原始 ID token 暴露给明文 XSS 窃取。

## 1. 不可导出 WebCrypto 密钥持久化

`window.crypto.subtle.generateKey` 生成 `CryptoKey`。要在无后端的情况下跨浏览器重启保留，`CryptoKey` 必须存入 `IndexedDB`。

> [!IMPORTANT] `IndexedDB` 通过结构化克隆算法原生支持存储 `CryptoKey` 对象，同时严格保持 `extractable: false`。XSS 攻击者可在活动会话中请求浏览器解密现有数据，但**无法**窃取或导出原始密钥材料。

## 2. 完整实现

```typescript
export class EncryptedIdTokenStore {
  private static DB_NAME = 'auth_offline_vault';
  private static STORE_NAME = 'keys_and_tokens';

  private static async getDb(): Promise<IDBDatabase> {
    return new Promise((resolve, reject) => {
      const req = indexedDB.open(this.DB_NAME, 1);
      req.onupgradeneeded = () => {
        req.result.createObjectStore(this.STORE_NAME);
      };
      req.onsuccess = () => resolve(req.result);
      req.onerror = () => reject(req.error);
    });
  }

  // 生成或获取已有的不可导出 AES-GCM 会话密钥
  static async getOrCreateKey(): Promise<CryptoKey> {
    const db = await this.getDb();
    const tx = db.transaction(this.STORE_NAME, 'readonly');
    const store = tx.objectStore(this.STORE_NAME);

    const existingKey = await new Promise<CryptoKey | undefined>((resolve, reject) => {
      const getReq = store.get('session_aes_key');
      getReq.onsuccess = () => resolve(getReq.result);
      getReq.onerror = () => reject(getReq.error);
    });

    if (existingKey) {
      return existingKey;
    }

    // 生成不可导出密钥
    const newKey = await window.crypto.subtle.generateKey(
      { name: 'AES-GCM', length: 256 },
      false, // 不可导出：无法通过 exportKey() 导出
      ['encrypt', 'decrypt']
    );

    const writeTx = db.transaction(this.STORE_NAME, 'readwrite');
    writeTx.objectStore(this.STORE_NAME).put(newKey, 'session_aes_key');
    await new Promise((resolve, reject) => {
      writeTx.oncomplete = resolve;
      writeTx.onerror = () => reject(writeTx.error);
    });

    return newKey;
  }

  static async encryptAndStore(idToken: string): Promise<void> {
    const key = await this.getOrCreateKey();
    const iv = window.crypto.getRandomValues(new Uint8Array(12));
    const encoded = new TextEncoder().encode(idToken);

    const ciphertext = await window.crypto.subtle.encrypt(
      { name: 'AES-GCM', iv },
      key,
      encoded
    );

    const db = await this.getDb();
    const tx = db.transaction(this.STORE_NAME, 'readwrite');
    tx.objectStore(this.STORE_NAME).put({ iv, ciphertext }, 'encrypted_token');
    await new Promise((resolve, reject) => {
      tx.oncomplete = resolve;
      tx.onerror = () => reject(tx.error);
    });
  }

  static async getDecryptedToken(): Promise<string | null> {
    const db = await this.getDb();
    const tx = db.transaction(this.STORE_NAME, 'readonly');
    const store = tx.objectStore(this.STORE_NAME);

    const record = await new Promise<{ iv: Uint8Array; ciphertext: ArrayBuffer } | undefined>(
      (resolve, reject) => {
        const getReq = store.get('encrypted_token');
        getReq.onsuccess = () => resolve(getReq.result);
        getReq.onerror = () => reject(getReq.error);
      }
    );

    if (!record) return null;

    const key = await this.getOrCreateKey();
    const decrypted = await window.crypto.subtle.decrypt(
      { name: 'AES-GCM', iv: record.iv },
      key,
      record.ciphertext
    );

    return new TextDecoder().decode(decrypted);
  }
}
```