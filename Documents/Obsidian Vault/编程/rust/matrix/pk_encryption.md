# 公钥 (PK) 加密 (Megolm Backup v1)

[返回索引](index.md)

实现用于 Matrix 密钥备份的 `m.megolm_backup.v1.curve25519-aes-sha2` 算法。

**警告:** 此方案存在已知缺陷，不提供密文认证。主要为兼容性而提供。使用受 `insecure-pk-encryption` 功能标志控制。

## `PkDecryption`

*   **创建/恢复:**
    *   `new()`: 使用随机 Curve25519 密钥对创建。
    *   `from_key(secret_key: Curve25519SecretKey)`: 从现有密钥创建。
    *   `from_libolm_pickle(pickle: &str, pickle_key: &[u8])`: 从遗留 libolm pickle 恢复。
*   **密钥访问:**
    *   `secret_key()`: 返回对密钥的引用。
    *   `public_key()`: 返回公钥。
*   **核心操作:**
    *   `decrypt(message: &Message)`: 解密 `Message`。返回 `Result<Vec<u8>, Error>`。
*   **序列化:**
    *   `to_libolm_pickle(pickle_key: &[u8])`: 序列化为遗留 libolm pickle 格式。返回 `Result<String, LibolmPickleError>`。

## `PkEncryption`

*   **创建:**
    *   `from_key(public_key: Curve25519PublicKey)`: 创建一个针对该公钥的加密对象。
*   **核心操作:**
    *   `encrypt(message: &[u8])`: 加密明文。返回一个 `Message`。

## 相关结构体 & 枚举

*   **`Message`**: 持有 `ciphertext`, `mac` (有缺陷) 和 `ephemeral_key`。具有 `from_base64()` 构造函数。
*   **`Error`**: 解密错误 (`InvalidPadding`, `Mac`)。
*   **`MessageDecodeError`**: 从 base64 解码 `Message` 时的错误 (`Base64`, `Key`)。
