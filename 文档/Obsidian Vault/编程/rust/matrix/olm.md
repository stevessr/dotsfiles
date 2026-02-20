# Olm 协议 (1:1 加密)

[返回索引](index.md)

Olm 使用双棘轮算法为 1:1 聊天提供端到端加密。

## `Account`

代表用户的 Olm 身份，管理密钥和会话。

*   **创建/恢复:**
    *   `new()`: 使用随机密钥创建一个新帐户。
    *   `from_pickle(pickle: AccountPickle)`: 从现代 pickle 格式恢复。
    *   `from_libolm_pickle(pickle: &str, pickle_key: &[u8])` (`libolm-compat`): 从遗留加密 pickle 恢复。
    *   `from_dehydrated_device(pickle: &str, key: &[u8], max_age: Option<Duration>)` (`dehydration`): 从脱水设备备份恢复。
*   **密钥管理:**
    *   `identity_keys()`: 返回帐户的 `IdentityKeys`（Ed25519 和 Curve25519 公钥）。
    *   `one_time_keys()`: 返回当前的 Curve25519 一次性公钥映射。
    *   `generate_one_time_keys(count: usize)`: 生成指定数量的新一次性密钥。
    *   `generate_fallback_key()`: 生成一个后备密钥（如果尚不存在）。
    *   `mark_keys_as_published()`: 将当前的一次性密钥集和后备密钥（如果有）标记为已发布。
    *   `max_number_of_one_time_keys()`: 返回帐户可以容纳的最大一次性密钥数量。
    *   `forget_old_fallback_key()`: 如果存在，则移除后备密钥。
*   **会话初始化:**
    *   `create_outbound_session(their_identity_key: &Ed25519PublicKey, their_one_time_key: &Curve25519PublicKey)`: 使用接收者的身份密钥和一次性密钥创建一个新的出站 `Session`。返回 `Result<(Session, PreKeyMessage), SessionCreationError>`。
    *   `create_inbound_session(message: PreKeyMessage)`: 从接收到的 `PreKeyMessage` 创建一个新的入站 `Session`。返回 `Result<Session, SessionCreationError>`。
    *   `create_inbound_session_from(their_identity_key: &Ed25519PublicKey, message: PreKeyMessage)`: 从 `PreKeyMessage` 创建一个入站 `Session`，并验证其是否与预期的发送者身份密钥匹配。返回 `Result<Session, SessionCreationError>`。
*   **密钥移除:**
    *   `remove_one_time_keys(session: &Session)`: 移除给定会话使用的一次性密钥。
*   **签名:**
    *   `sign(message: &[u8])`: 使用帐户的 Ed25519 身份密钥对任意数据进行签名。返回 `Ed25519Signature`。
*   **Pickling/序列化:**
    *   `pickle()`: 返回一个可序列化的 `AccountPickle`。
    *   `to_dehydrated_device(key: &[u8], max_keys: usize, max_age: Duration)` (`dehydration`): 创建一个加密的脱水设备备份。返回 `Result<String, DehydratedDeviceError>`。

## `Session`

代表一个已建立的 Olm 1:1 加密通道。

*   **创建/恢复:**
    *   *(内部 `new` 和 `new_remote` 方法由 `Account` 使用)*
    *   `from_pickle(pickle: SessionPickle)`: 从现代 pickle 格式恢复。
    *   `from_libolm_pickle(pickle: &str, pickle_key: &[u8])` (`libolm-compat`): 从遗留加密 pickle 恢复。
*   **信息 & 状态:**
    *   `session_id()`: 返回唯一的 base64 编码的会话标识符字符串。
    *   `has_received_message()`: 如果至少成功接收并解密了一条消息，则返回 `true`。
    *   `session_keys()`: 返回与此会话关联的 `SessionKeys`（身份密钥、基础密钥、一次性密钥）。
    *   `session_config()`: 返回此会话使用的 `SessionConfig`（V1 或 V2）。
*   **核心操作:**
    *   `encrypt(plaintext: impl AsRef<[u8]>)`: 加密明文。返回一个 `OlmMessage`（`PreKey` 或 `Normal`）。
    *   `decrypt(message: &OlmMessage)`: 解密一个 `OlmMessage`。返回 `Result<Vec<u8>, DecryptionError>`。
*   **低级 API (`low-level-api`):**
    *   `next_message_key()`: 获取用于加密的下一个 `MessageKey`（高级用法）。
*   **序列化:**
    *   `pickle()`: 返回一个可序列化的 `SessionPickle`。

## 相关结构体 & 枚举

*   **`IdentityKeys`**: 持有帐户的 Ed25519 和 Curve25519 公钥。
*   **`OlmMessage`**: 代表加密消息的枚举（`PreKey` 或 `Normal`）。
*   **`PreKeyMessage`**: 用于建立会话的初始消息类型。
*   **`SessionKeys`**: 持有与会话关联的身份密钥、基础密钥和一次性密钥。
*   **`SessionConfig`**: Olm 会话的配置（当前为 V1 或 V2）。
*   **`AccountPickle`**: `Account` 的可序列化表示。
*   **`SessionPickle`**: `Session` 的可序列化表示。
*   **`SessionCreationError`**: 会话创建期间的错误（例如，`InvalidSignature`, `InvalidOneTimeKey`）。
*   **`DecryptionError`**: 解密期间的错误（例如，`InvalidMAC`, `MissingMessageKey`）。
*   **`DehydratedDeviceError`**: 与脱水设备操作相关的错误。
