# ECIES (集成加密方案)

[返回索引](index.md)

使用 X25519、HKDF-SHA512 和 ChaCha20-Poly1305 实现 ECIES，常用于二维码登录流程。

## `Ecies` (初始状态)

*   **创建:**
    *   `new()`: 使用随机临时密钥创建，使用默认信息前缀 (`MATRIX_QR_CODE_LOGIN`)。
    *   `with_info(info: &str)`: 使用自定义信息前缀创建。
*   **密钥访问:**
    *   `public_key()`: 返回临时公钥。
*   **核心操作:**
    *   `establish_outbound_channel(their_public_key: Curve25519PublicKey, initial_plaintext: &[u8])`: 启动通道，加密初始消息。返回 `Result<OutboundCreationResult, Error>`。
    *   `establish_inbound_channel(message: &InitialMessage)`: 从初始消息建立接收方通道。返回 `Result<InboundCreationResult, Error>`。

## `EstablishedEcies` (建立后)

*   **信息访问:**
    *   `public_key()`: 返回此方的临时公钥。
    *   `check_code()`: 返回用于带外验证的 `CheckCode` 的引用。
*   **核心操作:**
    *   `encrypt(plaintext: &[u8])`: 为通道加密明文。返回 `Message`。
    *   `decrypt(message: &Message)`: 解密接收到的 `Message`。返回 `Result<Vec<u8>, Error>`。

## 相关结构体 & 枚举

*   **`InitialMessage`**: 包含发起者的公钥和初始密文。
*   **`Message`**: 包含建立后的后续密文。具有 `from_base64()` 构造函数。
*   **`CheckCode`**: 用于 OOB 验证的 2 字节代码。具有 `as_bytes()`, `to_digit()` 方法。
*   **`OutboundCreationResult`**: 包含 `EstablishedEcies` 和 `InitialMessage`。
*   **`InboundCreationResult`**: 包含 `EstablishedEcies` 和解密的初始明文 `message`。
*   **`Error`**: ECIES 失败 (`NonContributoryKey`, `Decryption`)。
*   **`MessageDecodeError`**: 解码 `InitialMessage` 或 `Message` 时的错误 (`Base64`, `Key`)。
