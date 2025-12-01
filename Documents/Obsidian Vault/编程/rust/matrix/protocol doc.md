# Vodozemac 协议文档

  

本文档根据 Vodozemac 库的源代码，概述了其提供的公共 API 和数据结构。Vodozemac 实现了 Matrix 中使用的加密协议，包括 Olm、Megolm、SAS、PK 加密和 ECIES。

  

## 1. 引言

  

Vodozemac 是 Olm 和 Megolm 加密棘轮（cryptographic ratchets）以及相关协议（如短认证字符串 (SAS) 验证、用于备份的公钥 (PK) 加密和椭圆曲线集成加密方案 (ECIES)）的 Rust 实现。它旨在为原始 C 库 libolm 提供一个安全且高性能的替代方案。

  

## 2. 通用类型

  

几个基本类型在不同模块中被使用：

  

*   **`Curve25519PublicKey`**: 代表 X25519 Diffie-Hellman 函数的公钥。

*   **`Curve25519SecretKey`**: 代表 X25519 Diffie-Hellman 函数的密钥。

*   **`Ed25519PublicKey`**: 代表 Ed25519 签名方案的公钥。

*   **`Ed25519SecretKey`**: 代表 Ed25519 签名方案的密钥。

*   **`Ed25519Signature`**: 代表一个 Ed25519 签名。

*   **`KeyError`**: 与密钥验证或使用相关的错误类型（例如，非贡献性密钥）。

*   **`SignatureError`**: 与签名验证失败相关的错误类型。

*   **`PickleError`**: 现代（基于 `serde`）pickling/unpickling 期间失败的错误类型。

*   **`LibolmPickleError`**: 遗留（`libolm-compat`）pickling/unpickling 期间失败的错误类型。

  

## 3. Olm 协议 (1:1 加密)

  

Olm 使用双棘轮算法为 1:1 聊天提供端到端加密。

  

### 3.1. `Account`

  

代表用户的 Olm 身份，管理密钥和会话。

  

*   **创建/恢复:**

    *   `new()`: 使用随机密钥创建一个新帐户。

    *   `from_pickle(pickle: AccountPickle)`: 从现代 pickle 格式恢复。

    *   `from_libolm_pickle(pickle: &str, pickle_key: &[u8])` (`libolm-compat`): 从遗留加密 pickle 恢复。

    *   `from_dehydrated_device(pickle: &str, key: &[u8], max_age: Option<Duration>)` (`dehydration`): 从脱水设备备份恢复。

*   **密钥管理:**

    *   `identity_keys()`: 返回帐户的 `IdentityKeys`（Ed25519 和 Curve25519 公钥）。

    *   `one_time_keys()`: 返回当前的 Curve25519 一次性公钥映射。

    *   `generate_one_time_keys(count: usize)`: 生成指定数量的新一次性密钥。

    *   `generate_fallback_key()`: 生成一个后备密钥（如果尚不存在）。

    *   `mark_keys_as_published()`: 将当前的一次性密钥集和后备密钥（如果有）标记为已发布。

    *   `max_number_of_one_time_keys()`: 返回帐户可以容纳的最大一次性密钥数量。

    *   `forget_old_fallback_key()`: 如果存在，则移除后备密钥。

*   **会话初始化:**

    *   `create_outbound_session(their_identity_key: &Ed25519PublicKey, their_one_time_key: &Curve25519PublicKey)`: 使用接收者的身份密钥和一次性密钥创建一个新的出站 `Session`。返回 `Result<(Session, PreKeyMessage), SessionCreationError>`。

    *   `create_inbound_session(message: PreKeyMessage)`: 从接收到的 `PreKeyMessage` 创建一个新的入站 `Session`。返回 `Result<Session, SessionCreationError>`。

    *   `create_inbound_session_from(their_identity_key: &Ed25519PublicKey, message: PreKeyMessage)`: 从 `PreKeyMessage` 创建一个入站 `Session`，并验证其是否与预期的发送者身份密钥匹配。返回 `Result<Session, SessionCreationError>`。

*   **密钥移除:**

    *   `remove_one_time_keys(session: &Session)`: 移除给定会话使用的一次性密钥。

*   **签名:**

    *   `sign(message: &[u8])`: 使用帐户的 Ed25519 身份密钥对任意数据进行签名。返回 `Ed25519Signature`。

*   **Pickling/序列化:**

    *   `pickle()`: 返回一个可序列化的 `AccountPickle`。

    *   `to_dehydrated_device(key: &[u8], max_keys: usize, max_age: Duration)` (`dehydration`): 创建一个加密的脱水设备备份。返回 `Result<String, DehydratedDeviceError>`。

  

### 3.2. `Session`

  

代表一个已建立的 Olm 1:1 加密通道。

  

*   **创建/恢复:**

    *   *(内部 `new` 和 `new_remote` 方法由 `Account` 使用)*

    *   `from_pickle(pickle: SessionPickle)`: 从现代 pickle 格式恢复。

    *   `from_libolm_pickle(pickle: &str, pickle_key: &[u8])` (`libolm-compat`): 从遗留加密 pickle 恢复。

*   **信息 & 状态:**

    *   `session_id()`: 返回唯一的 base64 编码的会话标识符字符串。

    *   `has_received_message()`: 如果至少成功接收并解密了一条消息，则返回 `true`。

    *   `session_keys()`: 返回与此会话关联的 `SessionKeys`（身份密钥、基础密钥、一次性密钥）。

    *   `session_config()`: 返回此会话使用的 `SessionConfig`（V1 或 V2）。

*   **核心操作:**

    *   `encrypt(plaintext: impl AsRef<[u8]>)`: 加密明文。返回一个 `OlmMessage`（`PreKey` 或 `Normal`）。

    *   `decrypt(message: &OlmMessage)`: 解密一个 `OlmMessage`。返回 `Result<Vec<u8>, DecryptionError>`。

*   **低级 API (`low-level-api`):**

    *   `next_message_key()`: 获取用于加密的下一个 `MessageKey`（高级用法）。

*   **序列化:**

    *   `pickle()`: 返回一个可序列化的 `SessionPickle`。

  

### 3.3. 相关结构体 & 枚举

  

*   **`IdentityKeys`**: 持有帐户的 Ed25519 和 Curve25519 公钥。

*   **`OlmMessage`**: 代表加密消息的枚举（`PreKey` 或 `Normal`）。

*   **`PreKeyMessage`**: 用于建立会话的初始消息类型。

*   **`SessionKeys`**: 持有与会话关联的身份密钥、基础密钥和一次性密钥。

*   **`SessionConfig`**: Olm 会话的配置（当前为 V1 或 V2）。

*   **`AccountPickle`**: `Account` 的可序列化表示。

*   **`SessionPickle`**: `Session` 的可序列化表示。

*   **`SessionCreationError`**: 会话创建期间的错误（例如，`InvalidSignature`, `InvalidOneTimeKey`）。

*   **`DecryptionError`**: 解密期间的错误（例如，`InvalidMAC`, `MissingMessageKey`）。

*   **`DehydratedDeviceError`**: 与脱水设备操作相关的错误。

  

## 4. Megolm 协议 (群组加密)

  

Megolm 为群聊提供加密，其中一个发送者为多个接收者加密。

  

### 4.1. `GroupSession` (出站)

  

代表 Megolm 会话的发送方。

  

*   **创建/恢复:**

    *   `new(config: SessionConfig)`: 使用随机密钥创建一个新的出站会话。

    *   `from_pickle(pickle: GroupSessionPickle)`: 从现代 pickle 格式恢复。

    *   `from_libolm_pickle(pickle: &str, pickle_key: &[u8])` (`libolm-compat`): 从遗留加密 pickle 恢复。

*   **信息 & 状态:**

    *   `session_id()`: 返回唯一的 base64 编码的会话 ID（Ed25519 公钥）。

    *   `message_index()`: 返回当前消息索引（每次加密时递增）。

    *   `session_config()`: 返回 `SessionConfig`（V1 或 V2）。

*   **核心操作:**

    *   `encrypt(plaintext: impl AsRef<[u8]>)`: 加密明文，对其签名，并返回一个 `MegolmMessage`。递增索引。

    *   `session_key()`: 将当前状态（棘轮、公钥、索引）导出到一个签名的 `SessionKey` 中，以便与接收者共享。

*   **序列化:**

    *   `pickle()`: 返回一个可序列化的 `GroupSessionPickle`。

  

### 4.2. `InboundGroupSession` (入站)

  

代表 Megolm 会话的接收方。

  

*   **创建/恢复:**

    *   `new(key: &SessionKey, session_config: SessionConfig)`: 从签名的 `SessionKey` 创建（受信任）。

    *   `import(session_key: &ExportedSessionKey, session_config: SessionConfig)`: 从未签名的 `ExportedSessionKey` 创建（需要外部信任验证）。

    *   `from_pickle(pickle: InboundGroupSessionPickle)`: 从现代 pickle 格式恢复。

    *   `from_libolm_pickle(pickle: &str, pickle_key: &[u8])` (`libolm-compat`): 从遗留加密 pickle 恢复。

*   **信息 & 状态:**

    *   `session_id()`: 返回发送者的会话 ID（Ed25519 公钥）。

    *   `first_known_index()`: 返回此会话可以解密的最早消息索引。

*   **核心操作:**

    *   `decrypt(message: &MegolmMessage)`: 解密 `MegolmMessage`。验证签名和 MAC。返回 `Result<DecryptedMessage, DecryptionError>`。

    *   `advance_to(index: u32)`: 推进棘轮，丢弃索引小于 `index` 的密钥。返回 `bool`。

    *   `export_at(index: u32)`: 将特定索引处的会话状态导出到 `ExportedSessionKey`。返回 `Option<ExportedSessionKey>`。

    *   `export_at_first_known_index()`: `export_at(self.first_known_index())` 的便捷方法。

*   **会话比较 & 合并:**

    *   `connected(other: &mut InboundGroupSession)`: 检查两个会话是否共享相同的来源。返回 `bool`。

    *   `compare(other: &mut InboundGroupSession)`: 如果连接，则根据 `first_known_index` 进行比较。返回 `SessionOrdering`。

    *   `merge(other: &mut InboundGroupSession)`: 合并两个连接的会话，采用最早的索引和组合的信任信息。返回 `Option<InboundGroupSession>`。

*   **低级 API (`low-level-api`):**

    *   `get_cipher_at(message_index: u32)`: 获取特定索引的 `Cipher` 而不推进状态。返回 `Option<Cipher>`。

*   **序列化:**

    *   `pickle()`: 返回一个可序列化的 `InboundGroupSessionPickle`。

  

### 4.3. 相关结构体 & 枚举

  

*   **`SessionKey`**: 用于共享的 `GroupSession` 状态的签名导出。

*   **`ExportedSessionKey`**: `InboundGroupSession` 状态的未签名导出。

*   **`MegolmMessage`**: 一个加密的 Megolm 消息。

*   **`DecryptedMessage`**: 包含解密的 `plaintext` 和 `message_index`。

*   **`SessionConfig`**: Megolm 的配置（V1 或 V2）。

*   **`GroupSessionPickle`**: `GroupSession` 的可序列化表示。

*   **`InboundGroupSessionPickle`**: `InboundGroupSession` 的可序列化表示。

*   **`DecryptionError`**: 解密期间的错误（例如，`Signature`, `InvalidMAC`, `UnknownMessageIndex`）。

*   **`SessionKeyDecodeError`**: 解码 `SessionKey` 或 `ExportedSessionKey` 时的错误。

*   **`SessionOrdering`**: 比较 `InboundGroupSession` 的结果（`Equal`, `Better`, `Worse`, `Unconnected`）。

  

## 5. 短认证字符串 (SAS)

  

提供一种使用简短、人类可比较的字符串（表情符号或十进制数）进行带外密钥验证的机制。

  

### 5.1. `Sas` (初始状态)

  

*   **创建:**

    *   `new()`: 使用随机的临时 Curve25519 密钥对创建一个新的 SAS 对象。

*   **密钥访问:**

    *   `public_key()`: 返回 Curve25519 公钥。

*   **核心操作:**

    *   `diffie_hellman(their_public_key: Curve25519PublicKey)`: 执行 DH 交换。返回 `Result<EstablishedSas, KeyError>`。

    *   `diffie_hellman_with_raw(other_public_key: &str)`: 使用 base64 密钥字符串执行 DH。返回 `Result<EstablishedSas, KeyError>`。

  

### 5.2. `EstablishedSas` (DH 之后)

  

*   **信息访问:**

    *   `our_public_key()`: 返回此方在 DH 中使用的公钥。

    *   `their_public_key()`: 返回对方在 DH 中使用的公钥。

*   **SAS 生成:**

    *   `bytes(info: &str)`: 使用 HKDF 和约定的 `info` 字符串生成 6 字节的 `SasBytes`。

    *   `bytes_raw(info: &str, count: usize)`: 使用 HKDF 生成原始字节。返回 `Result<Vec<u8>, InvalidCount>`。

*   **MAC 计算 & 验证:**

    *   `calculate_mac(input: &str, info: &str)`: 使用派生密钥和 `info` 计算 `input` 的 HMAC-SHA256 MAC。返回 `Mac`。

    *   `verify_mac(input: &str, info: &str, tag: &Mac)`: 验证接收到的 `Mac`。返回 `Result<(), SasError>`。

    *   `calculate_mac_invalid_base64(input: &str, info: &str)` (`libolm-compat`): 计算具有不正确 base64 编码的 MAC（libolm 错误）。

  

### 5.3. 相关结构体 & 枚举

  

*   **`SasBytes`**: 持有用于 SAS 的 6 个字节。提供 `emoji_indices()`, `decimals()`, `as_bytes()`。

*   **`Mac`**: 代表计算出的 MAC 标签。提供 `to_base64()`, `as_bytes()`, `from_slice()`, `from_base64()`。

*   **`SasError`**: MAC 验证期间的错误 (`Mac(MacError)`)。

*   **`InvalidCount`**: 如果 `bytes_raw` 请求过多字节则返回错误。

  

## 6. 公钥 (PK) 加密 (Megolm Backup v1)

  

实现用于 Matrix 密钥备份的 `m.megolm_backup.v1.curve25519-aes-sha2` 算法。

  

**警告:** 此方案存在已知缺陷，不提供密文认证。主要为兼容性而提供。使用受 `insecure-pk-encryption` 功能标志控制。

  

### 6.1. `PkDecryption`

  

*   **创建/恢复:**

    *   `new()`: 使用随机 Curve25519 密钥对创建。

    *   `from_key(secret_key: Curve25519SecretKey)`: 从现有密钥创建。

    *   `from_libolm_pickle(pickle: &str, pickle_key: &[u8])`: 从遗留 libolm pickle 恢复。

*   **密钥访问:**

    *   `secret_key()`: 返回对密钥的引用。

    *   `public_key()`: 返回公钥。

*   **核心操作:**

    *   `decrypt(message: &Message)`: 解密 `Message`。返回 `Result<Vec<u8>, Error>`。

*   **序列化:**

    *   `to_libolm_pickle(pickle_key: &[u8])`: 序列化为遗留 libolm pickle 格式。返回 `Result<String, LibolmPickleError>`。

  

### 6.2. `PkEncryption`

  

*   **创建:**

    *   `from_key(public_key: Curve25519PublicKey)`: 创建一个针对该公钥的加密对象。

*   **核心操作:**

    *   `encrypt(message: &[u8])`: 加密明文。返回一个 `Message`。

  

### 6.3. 相关结构体 & 枚举

  

*   **`Message`**: 持有 `ciphertext`, `mac` (有缺陷) 和 `ephemeral_key`。具有 `from_base64()` 构造函数。

*   **`Error`**: 解密错误 (`InvalidPadding`, `Mac`)。

*   **`MessageDecodeError`**: 从 base64 解码 `Message` 时的错误 (`Base64`, `Key`)。

  

## 7. ECIES (集成加密方案)

  

使用 X25519、HKDF-SHA512 和 ChaCha20-Poly1305 实现 ECIES，常用于二维码登录流程。

  

### 7.1. `Ecies` (初始状态)

  

*   **创建:**

    *   `new()`: 使用随机临时密钥创建，使用默认信息前缀。

    *   `with_info(info: &str)`: 使用自定义信息前缀创建。

*   **密钥访问:**

    *   `public_key()`: 返回临时公钥。

*   **核心操作:**

    *   `establish_outbound_channel(their_public_key: Curve25519PublicKey, initial_plaintext: &[u8])`: 启动通道，加密初始消息。返回 `Result<OutboundCreationResult, Error>`。

    *   `establish_inbound_channel(message: &InitialMessage)`: 从初始消息建立接收方通道。返回 `Result<InboundCreationResult, Error>`。

  

### 7.2. `EstablishedEcies` (建立后)

  

*   **信息访问:**

    *   `public_key()`: 返回此方的临时公钥。

    *   `check_code()`: 返回用于带外验证的 `CheckCode` 的引用。

*   **核心操作:**

    *   `encrypt(plaintext: &[u8])`: 为通道加密明文。返回 `Message`。

    *   `decrypt(message: &Message)`: 解密接收到的 `Message`。返回 `Result<Vec<u8>, Error>`。

  

### 7.3. 相关结构体 & 枚举

  

*   **`InitialMessage`**: 包含发起者的公钥和初始密文。

*   **`Message`**: 包含建立后的后续密文。具有 `from_base64()` 构造函数。

*   **`CheckCode`**: 用于 OOB 验证的 2 字节代码。具有 `as_bytes()`, `to_digit()` 方法。

*   **`OutboundCreationResult`**: 包含 `EstablishedEcies` 和 `InitialMessage`。

*   **`InboundCreationResult`**: 包含 `EstablishedEcies` 和解密的初始明文 `message`。

*   **`Error`**: ECIES 失败 (`NonContributoryKey`, `Decryption`)。

*   **`MessageDecodeError`**: 解码 `InitialMessage` 或 `Message` 时的错误 (`Base64`, `Key`)。

  

## 8. Pickling & 持久化

  

Vodozemac 支持序列化帐户和会话状态以实现持久化。

  

*   **现代 Pickling (`serde`)**:

    *   使用 `serde` 进行序列化/反序列化。

    *   对象提供 `pickle()` 方法，返回可序列化的 `*Pickle` 结构体（例如，`AccountPickle`, `SessionPickle`, `GroupSessionPickle`, `InboundGroupSessionPickle`）。

    *   `*Pickle` 结构体通常具有 `encrypt()` 和 `from_encrypted()` 方法，用于使用用户提供的密钥进行安全存储。

    *   错误类型: `PickleError`。

*   **遗留 Pickling (`libolm-compat`)**:

    *   受 `libolm-compat` 功能标志控制。

    *   在主要对象（`Account`, `Session` 等）上提供 `from_libolm_pickle()` 方法，以加载由 libolm pickling 的数据。

    *   可能提供 `to_libolm_pickle()` 方法用于导出（例如，`PkDecryption`）。

    *   需要 `pickle_key` 进行解密/加密。

    *   错误类型: `LibolmPickleError`。

*   **脱水设备 (`dehydration`)**:

    *   受 `dehydration` 功能标志控制。

    *   `Account::to_dehydrated_device()`: 创建适合服务器存储的加密备份字符串。

    *   `Account::from_dehydrated_device()`: 从脱水备份恢复帐户。

    *   错误类型: `DehydratedDeviceError`。