# 短认证字符串 (SAS)

[返回索引](index.md)

提供一种使用简短、人类可比较的字符串（表情符号或十进制数）进行带外密钥验证的机制。

## `Sas` (初始状态)

*   **创建:**
    *   `new()`: 使用随机的临时 Curve25519 密钥对创建一个新的 SAS 对象。
*   **密钥访问:**
    *   `public_key()`: 返回 Curve25519 公钥。
*   **核心操作:**
    *   `diffie_hellman(their_public_key: Curve25519PublicKey)`: 执行 DH 交换。返回 `Result<EstablishedSas, KeyError>`。
    *   `diffie_hellman_with_raw(other_public_key: &str)`: 使用 base64 密钥字符串执行 DH。返回 `Result<EstablishedSas, KeyError>`。

## `EstablishedSas` (DH 之后)

*   **信息访问:**
    *   `our_public_key()`: 返回此方在 DH 中使用的公钥。
    *   `their_public_key()`: 返回对方在 DH 中使用的公钥。
*   **SAS 生成:**
    *   `bytes(info: &str)`: 使用 HKDF 和约定的 `info` 字符串生成 6 字节的 `SasBytes`。
    *   `bytes_raw(info: &str, count: usize)`: 使用 HKDF 生成原始字节。返回 `Result<Vec<u8>, InvalidCount>`。
*   **MAC 计算 & 验证:**
    *   `calculate_mac(input: &str, info: &str)`: 使用派生密钥和 `info` 计算 `input` 的 HMAC-SHA256 MAC。返回 `Mac`。
    *   `verify_mac(input: &str, info: &str, tag: &Mac)`: 验证接收到的 `Mac`。返回 `Result<(), SasError>`。
    *   `calculate_mac_invalid_base64(input: &str, info: &str)` (`libolm-compat`): 计算具有不正确 base64 编码的 MAC（libolm 错误）。

## 相关结构体 & 枚举

*   **`SasBytes`**: 持有用于 SAS 的 6 个字节。提供 `emoji_indices()`, `decimals()`, `as_bytes()`。
*   **`Mac`**: 代表计算出的 MAC 标签。提供 `to_base64()`, `as_bytes()`, `from_slice()`, `from_base64()`。
*   **`SasError`**: MAC 验证期间的错误 (`Mac(MacError)`)。
*   **`InvalidCount`**: 如果 `bytes_raw` 请求过多字节则返回错误。
