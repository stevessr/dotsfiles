# Megolm 协议 (群组加密)

[返回索引](index.md)

Megolm 为群聊提供加密，其中一个发送者为多个接收者加密。

## `GroupSession` (出站)

代表 Megolm 会话的发送方。

*   **创建/恢复:**
    *   `new(config: SessionConfig)`: 使用随机密钥创建一个新的出站会话。
    *   `from_pickle(pickle: GroupSessionPickle)`: 从现代 pickle 格式恢复。
    *   `from_libolm_pickle(pickle: &str, pickle_key: &[u8])` (`libolm-compat`): 从遗留加密 pickle 恢复。
*   **信息 & 状态:**
    *   `session_id()`: 返回唯一的 base64 编码的会话 ID（Ed25519 公钥）。
    *   `message_index()`: 返回当前消息索引（每次加密时递增）。
    *   `session_config()`: 返回 `SessionConfig`（V1 或 V2）。
*   **核心操作:**
    *   `encrypt(plaintext: impl AsRef<[u8]>)`: 加密明文，对其签名，并返回一个 `MegolmMessage`。递增索引。
    *   `session_key()`: 将当前状态（棘轮、公钥、索引）导出到一个签名的 `SessionKey` 中，以便与接收者共享。
*   **序列化:**
    *   `pickle()`: 返回一个可序列化的 `GroupSessionPickle`。

## `InboundGroupSession` (入站)

代表 Megolm 会话的接收方。

*   **创建/恢复:**
    *   `new(key: &SessionKey, session_config: SessionConfig)`: 从签名的 `SessionKey` 创建（受信任）。
    *   `import(session_key: &ExportedSessionKey, session_config: SessionConfig)`: 从未签名的 `ExportedSessionKey` 创建（需要外部信任验证）。
    *   `from_pickle(pickle: InboundGroupSessionPickle)`: 从现代 pickle 格式恢复。
    *   `from_libolm_pickle(pickle: &str, pickle_key: &[u8])` (`libolm-compat`): 从遗留加密 pickle 恢复。
*   **信息 & 状态:**
    *   `session_id()`: 返回发送者的会话 ID（Ed25519 公钥）。
    *   `first_known_index()`: 返回此会话可以解密的最早消息索引。
*   **核心操作:**
    *   `decrypt(message: &MegolmMessage)`: 解密 `MegolmMessage`。验证签名和 MAC。返回 `Result<DecryptedMessage, DecryptionError>`。
    *   `advance_to(index: u32)`: 推进棘轮，丢弃索引小于 `index` 的密钥。返回 `bool`。
    *   `export_at(index: u32)`: 将特定索引处的会话状态导出到 `ExportedSessionKey`。返回 `Option<ExportedSessionKey>`。
    *   `export_at_first_known_index()`: `export_at(self.first_known_index())` 的便捷方法。
*   **会话比较 & 合并:**
    *   `connected(other: &mut InboundGroupSession)`: 检查两个会话是否共享相同的来源。返回 `bool`。
    *   `compare(other: &mut InboundGroupSession)`: 如果连接，则根据 `first_known_index` 进行比较。返回 `SessionOrdering`。
    *   `merge(other: &mut InboundGroupSession)`: 合并两个连接的会话，采用最早的索引和组合的信任信息。返回 `Option<InboundGroupSession>`。
*   **低级 API (`low-level-api`):**
    *   `get_cipher_at(message_index: u32)`: 获取特定索引的 `Cipher` 而不推进状态。返回 `Option<Cipher>`。
*   **序列化:**
    *   `pickle()`: 返回一个可序列化的 `InboundGroupSessionPickle`。

## 相关结构体 & 枚举

*   **`SessionKey`**: 用于共享的 `GroupSession` 状态的签名导出。
*   **`ExportedSessionKey`**: `InboundGroupSession` 状态的未签名导出。
*   **`MegolmMessage`**: 一个加密的 Megolm 消息。
*   **`DecryptedMessage`**: 包含解密的 `plaintext` 和 `message_index`。
*   **`SessionConfig`**: Megolm 的配置（V1 或 V2）。
*   **`GroupSessionPickle`**: `GroupSession` 的可序列化表示。
*   **`InboundGroupSessionPickle`**: `InboundGroupSession` 的可序列化表示。
*   **`DecryptionError`**: 解密期间的错误（例如，`Signature`, `InvalidMAC`, `UnknownMessageIndex`）。
*   **`SessionKeyDecodeError`**: 解码 `SessionKey` 或 `ExportedSessionKey` 时的错误。
*   **`SessionOrdering`**: 比较 `InboundGroupSession` 的结果（`Equal`, `Better`, `Worse`, `Unconnected`）。
