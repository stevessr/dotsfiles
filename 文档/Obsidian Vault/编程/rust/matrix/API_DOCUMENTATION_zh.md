# Vodozemac HTTP API 文档 (中文版)

## 引言

本文档提供了 Vodozemac HTTP API 的人类可读文档。该 API 是 `vodozemac` Rust 库的一个包装器，通过 HTTP 暴露其 Olm、Megolm 和 SAS 加密功能。

该 API 遵循 RESTful 原则，使用标准的 HTTP 方法 (GET, POST) 以及 JSON 作为请求和响应体。密钥、消息和签名等二进制数据通常在 JSON 负载中使用 Base64 编码。

**基础 URL:** (示例) `http://localhost:8000`

**核心概念:**

*   **资源 ID:** 当您创建加密对象（如 Olm 账户或会话）时，服务器会为其分配一个唯一的 ID（例如 `account_id`, `session_id`, `sas_id`）。您可以在 URL 路径中使用这些 ID 与特定资源进行交互。
*   **Base64 编码:** 许多端点期望或返回在 JSON 负载中以 Base64 字符串编码的二进制数据（密钥、消息、签名）。
*   **错误处理:** 错误通过标准的 HTTP 状态码（4xx, 5xx）和一个包含 `error_code`、`message` 和可选 `details` 的 JSON 体来指示。（具体代码请参阅 `openapi.yaml`）。

---

## Olm 账户 (`/olm/accounts`)

本节涉及管理 Olm 身份，包括密钥和会话初始化。

### `POST /olm/accounts`

*   **目的:** 创建一个新的 Olm 账户实例。
*   **请求体:** 无。
*   **成功响应 (201 Created):** 返回新账户的 ID 及其公钥身份密钥。可以选择性地包含初始生成的一次性密钥或后备密钥。
    ```json
    {
      "account_id": "unique-server-generated-id",
      "identity_keys": {
        "curve25519": "base64_public_key",
        "ed25519": "base64_public_key"
      }
      // 可选: "one_time_keys": { ... }, "fallback_key": { ... }
    }
    ```

### `POST /olm/accounts/unpickle`

*   **目的:** 通过解密并加载先前生成的 pickle 中的状态来创建 Olm 账户。
*   **请求体:**
    ```json
    {
      "pickle": "base64_encrypted_pickle_string",
      "pickle_key": "base64_encoded_32_byte_key"
    }
    ```
*   **成功响应 (201 Created):** 返回新账户的 ID 和密钥，反映 unpickle 后的状态。
    ```json
    {
      "account_id": "new-server-generated-id",
      "identity_keys": { ... }
      // ... 从 unpickle 状态加载的其他密钥
    }
    ```

### `GET /olm/accounts/{account_id}`

*   **目的:** 检索特定 Olm 账户的详细信息，包括其公钥。
*   **路径参数:** `account_id` - 要检索的账户 ID。
*   **成功响应 (200 OK):**
    ```json
    {
      "account_id": "unique-server-generated-id",
      "identity_keys": { ... },
      "one_time_keys": { "curve25519": { /* 未发布的密钥映射 */ } },
      "fallback_key": { "curve25519": { /* 未发布的后备密钥，如果有的话 */ } }
    }
    ```

### `POST /olm/accounts/{account_id}/sign`

*   **目的:** 使用账户的 Ed25519 身份密钥对任意消息进行签名。
*   **路径参数:** `account_id` - 签名账户的 ID。
*   **请求体:**
    ```json
    {
      "message": "base64_encoded_message_to_sign"
    }
    ```
*   **成功响应 (200 OK):**
    ```json
    {
      "signature": "base64_encoded_ed25519_signature"
    }
    ```

### `POST /olm/accounts/{account_id}/one-time-keys`

*   **目的:** 为账户生成指定数量的新一次性密钥。这些密钥用于建立新的 Olm 会话。
*   **路径参数:** `account_id` - 账户的 ID。
*   **请求体:**
    ```json
    {
      "count": 50 // 要生成的密钥数量
    }
    ```
*   **成功响应 (200 OK):** 返回新生成的公钥一次性密钥。
    ```json
    {
      "one_time_keys": {
        "curve25519": {
          "new_key_id_1": "base64_public_key",
          // ...
        }
      }
    }
    ```

### `GET /olm/accounts/{account_id}/one-time-keys`

*   **目的:** 检索账户当前*未发布*的一次性密钥集。
*   **路径参数:** `account_id` - 账户的 ID。
*   **成功响应 (200 OK):**
    ```json
    {
      "one_time_keys": {
        "curve25519": {
          "key_id_1": "base64_public_key",
          // ... (所有当前未发布的密钥)
        }
      }
    }
    ```

### `POST /olm/accounts/{account_id}/fallback-key`

*   **目的:** 为账户生成一个新的后备密钥。如果接收方用完了一次性密钥，可以使用此密钥。一次只能存在一个未发布的后备密钥。
*   **路径参数:** `account_id` - 账户的 ID。
*   **成功响应 (201 Created / 200 OK):** 返回新生成的公钥后备密钥。
    ```json
    {
      "fallback_key": {
         "curve25519": {
           "key_id": "base64_public_key"
         }
      }
    }
    ```

### `GET /olm/accounts/{account_id}/fallback-key`

*   **目的:** 检索当前*未发布*的后备密钥（如果存在）。
*   **路径参数:** `account_id` - 账户的 ID。
*   **成功响应 (200 OK):** 返回密钥或空对象。
    ```json
    {
      "fallback_key": { // 仅当存在未发布的密钥时出现
         "curve25519": {
           "key_id": "base64_public_key"
         }
      }
    }
    ```
    或者，如果没有未发布的密钥，则返回 `{}`。

### `POST /olm/accounts/{account_id}/publish-keys`

*   **目的:** 将所有当前未发布的一次性密钥和后备密钥（如果有）标记为已发布。这通常在将它们上传到密钥服务器后完成。
*   **路径参数:** `account_id` - 账户的 ID。
*   **成功响应 (204 No Content):** 表示成功。

### `POST /olm/accounts/{account_id}/pickle`

*   **目的:** 检索账户当前状态的加密快照 (pickle)。
*   **路径参数:** `account_id` - 账户的 ID。
*   **请求体:**
    ```json
    {
      "pickle_key": "base64_encoded_32_byte_key" // 用于加密 pickle 的密钥
    }
    ```
*   **成功响应 (200 OK):**
    ```json
    {
      "pickle": "base64_encoded_encrypted_pickle_string"
    }
    ```

### `POST /olm/accounts/{account_id}/sessions/outbound`

*   **目的:** 与接收方发起一个新的出站 Olm 会话。需要接收方的身份密钥和他们的一次性密钥之一。
*   **路径参数:** `account_id` - 发起账户的 ID。
*   **请求体:**
    ```json
    {
      "identity_key": "base64_curve25519_public_key_of_recipient",
      "one_time_key": "base64_curve25519_public_key_of_recipient"
    }
    ```
*   **成功响应 (201 Created):** 返回新会话的 ID 和要发送给接收方的初始 "pre-key" 消息。
    ```json
    {
      "session_id": "server-generated-session-id",
      "initial_prekey_message": "base64_encoded_olm_message" // Type 0 (PRE_KEY)
    }
    ```

### `POST /olm/accounts/{account_id}/sessions/inbound`

*   **目的:** 在收到来自另一方的初始 "pre-key" 消息时建立一个新的入站 Olm 会话。
*   **路径参数:** `account_id` - 接收账户的 ID。
*   **请求体:**
    ```json
    {
      "pre_key_message": "base64_encoded_olm_prekey_message" // Type 0 (PRE_KEY)
    }
    ```
*   **成功响应 (201 Created):** 返回新会话的 ID 和初始消息的解密后明文内容。
    ```json
    {
      "session_id": "server-generated-session-id",
      "plaintext": "base64_encoded_decrypted_plaintext"
    }
    ```

---

## Olm 会话 (`/olm/sessions`)

本节涉及已建立的 1 对 1 Olm 会话。

### `GET /olm/sessions/{session_id}`

*   **目的:** 检索特定 Olm 会话的详细信息。
*   **路径参数:** `session_id` - 会话的 ID。
*   **成功响应 (200 OK):**
    ```json
    {
      "session_id": "server-generated-session-id",
      "config": { "version": 1 /* or 2 */ },
      "has_received_message": true // boolean
    }
    ```

### `POST /olm/sessions/{session_id}/encrypt`

*   **目的:** 使用已建立的 Olm 会话加密消息。
*   **路径参数:** `session_id` - 会话的 ID。
*   **请求体:**
    ```json
    {
      "plaintext": "base64_encoded_plaintext"
    }
    ```
*   **成功响应 (200 OK):** 返回加密后的 Olm 消息 (Type 1)。
    ```json
    {
      "message": "base64_encoded_olm_message" // Type 1 (MESSAGE)
    }
    ```

### `POST /olm/sessions/{session_id}/decrypt`

*   **目的:** 使用已建立的会话解密收到的 Olm 消息（Type 0 或 Type 1）。
*   **路径参数:** `session_id` - 会话的 ID。
*   **请求体:**
    ```json
    {
      "message": "base64_encoded_olm_message" // Type 0 or 1
    }
    ```
*   **成功响应 (200 OK):** 返回解密后的明文。
    ```json
    {
      "plaintext": "base64_encoded_decrypted_plaintext"
    }
    ```
*   **错误响应 (400 Bad Request / 409 Conflict):** 如果解密失败（例如，MAC 错误、消息用于不同会话、超出重放窗口的乱序消息）。

### `POST /olm/sessions/{session_id}/pickle`

*   **目的:** 检索会话当前状态的加密快照 (pickle)。
*   **路径参数:** `session_id` - 会话的 ID。
*   **请求体:**
    ```json
    {
      "pickle_key": "base64_encoded_32_byte_key" // 用于加密 pickle 的密钥
    }
    ```
*   **成功响应 (200 OK):**
    ```json
    {
      "pickle": "base64_encoded_encrypted_pickle_string"
    }
    ```

---

## Megolm 出站会话 (`/megolm/outbound-sessions`)

本节涉及创建和管理用于向群组发送消息的会话。

### `POST /megolm/outbound-sessions`

*   **目的:** 创建一个新的出站 Megolm 会话（群组会话棘轮）。
*   **请求体 (可选):** 指定会话配置（默认为版本 1）。
    ```json
    { "config": { "version": 1 /* or 2 */ } }
    ```
*   **成功响应 (201 Created):** 返回新会话的 ID 和初始会话密钥，该密钥必须安全地（例如，通过 Olm）与所有预期参与者共享。
    ```json
    {
      "session_id": "server-generated-session-id",
      "session_key": "base64_encoded_initial_session_key"
    }
    ```

### `POST /megolm/outbound-sessions/unpickle`

*   **目的:** 通过解密并加载 pickle 中的状态来创建出站 Megolm 会话。
*   **请求体:**
    ```json
    {
      "pickle": "base64_encrypted_pickle_string",
      "pickle_key": "base64_encoded_32_byte_key"
    }
    ```
*   **成功响应 (201 Created):** 返回新会话的 ID 和从 unpickle 状态加载的详细信息。
    ```json
    {
      "session_id": "new-server-generated-id",
      "message_index": 456,
      "config": { ... }
    }
    ```

### `GET /megolm/outbound-sessions/{session_id}`

*   **目的:** 检索特定出站 Megolm 会话的详细信息。
*   **路径参数:** `session_id` - 会话的 ID。
*   **成功响应 (200 OK):**
    ```json
    {
      "session_id": "server-generated-session-id",
      "message_index": 123, // 当前加密索引
      "config": { "version": 1 /* or 2 */ }
    }
    ```

### `POST /megolm/outbound-sessions/{session_id}/encrypt`

*   **目的:** 使用出站 Megolm 会话加密消息。会话的内部索引会前进。
*   **路径参数:** `session_id` - 会话的 ID。
*   **请求体:**
    ```json
    {
      "plaintext": "base64_encoded_plaintext"
    }
    ```
*   **成功响应 (200 OK):** 返回加密后的 Megolm 消息，包括其索引。
    ```json
    {
      "message": "base64_encoded_megolm_message"
    }
    ```

### `GET /megolm/outbound-sessions/{session_id}/key`

*   **目的:** 检索此出站会话的*当前*会话密钥。这对于与会话中途加入群组的新参与者共享很有用。
*   **路径参数:** `session_id` - 会话的 ID。
*   **成功响应 (200 OK):**
    ```json
    {
      "session_key": "base64_encoded_current_session_key"
    }
    ```

### `POST /megolm/outbound-sessions/{session_id}/pickle`

*   **目的:** 检索出站会话当前状态的加密快照 (pickle)。
*   **路径参数:** `session_id` - 会话的 ID。
*   **请求体:**
    ```json
    {
      "pickle_key": "base64_encoded_32_byte_key"
    }
    ```
*   **成功响应 (200 OK):**
    ```json
    {
      "pickle": "base64_encoded_encrypted_pickle_string"
    }
    ```

---

## Megolm 入站会话 (`/megolm/inbound-sessions`)

本节涉及创建和管理用于接收和解密群组消息的会话。

### `POST /megolm/inbound-sessions`

*   **目的:** 从群组发送方收到的 `SessionKey`（通常通过 Olm）创建一个新的入站 Megolm 会话。
*   **请求体:**
    ```json
    {
      "session_key": "base64_encoded_session_key",
      "config": { "version": 1 /* or 2 */ } // 可选, 默认为 v1
    }
    ```
*   **成功响应 (201 Created):** 返回新会话的 ID 及其可以解密的第一个消息索引。
    ```json
    {
      "session_id": "server-generated-session-id",
      "first_known_index": 123
    }
    ```

### `POST /megolm/inbound-sessions/import`

*   **目的:** 从 `ExportedSessionKey`（从另一个入站会话实例获取）创建一个新的入站 Megolm 会话。
*   **请求体:**
    ```json
    {
      "exported_key": "base64_encoded_exported_session_key",
      "config": { "version": 1 /* or 2 */ } // 可选, 默认为 v1
    }
    ```
*   **成功响应 (201 Created):** 返回新会话的 ID 及其可以解密的第一个消息索引。
    ```json
    {
      "session_id": "server-generated-session-id",
      "first_known_index": 456
    }
    ```

### `POST /megolm/inbound-sessions/unpickle`

*   **目的:** 通过解密并加载 pickle 中的状态来创建入站 Megolm 会话。
*   **请求体:**
    ```json
    {
      "pickle": "base64_encrypted_pickle_string",
      "pickle_key": "base64_encoded_32_byte_key"
    }
    ```
*   **成功响应 (201 Created):** 返回新会话的 ID 和从 unpickle 状态加载的详细信息。
    ```json
    {
      "session_id": "new-server-generated-id",
      "first_known_index": 789
    }
    ```

### `GET /megolm/inbound-sessions/{session_id}`

*   **目的:** 检索特定入站 Megolm 会话的详细信息。
*   **路径参数:** `session_id` - 会话的 ID。
*   **成功响应 (200 OK):**
    ```json
    {
      "session_id": "server-generated-session-id",
      "first_known_index": 123 // 此会话可以解密的最早索引
    }
    ```

### `POST /megolm/inbound-sessions/{session_id}/decrypt`

*   **目的:** 使用相应的入站会话解密收到的 Megolm 消息。
*   **路径参数:** `session_id` - 会话的 ID。
*   **请求体:**
    ```json
    {
      "message": "base64_encoded_megolm_message"
    }
    ```
*   **成功响应 (200 OK):** 返回解密后的明文和消息的索引。
    ```json
    {
      "plaintext": "base64_encoded_decrypted_plaintext",
      "message_index": 125
    }
    ```
*   **错误响应 (409 Conflict):** 如果解密失败（例如，索引太旧、重复索引、MAC 检查失败）。

### `POST /megolm/inbound-sessions/{session_id}/export`

*   **目的:** 在特定消息索引处导出 会话状态。这允许创建新的入站会话实例（通过 `/import`），该实例可以解密从该索引开始的消息。
*   **路径参数:** `session_id` - 会话的 ID。
*   **请求体:**
    ```json
    {
      "index": 130 // 要导出密钥的消息索引
    }
    ```
*   **成功响应 (200 OK):** 返回可导出的会话密钥数据。
    ```json
    {
      "exported_key": "base64_encoded_exported_session_key"
    }
    ```
*   **错误响应 (400 Bad Request):** 如果请求的索引太旧或无效。

### `POST /megolm/inbound-sessions/{session_id}/pickle`

*   **目的:** 检索入站会话当前状态的加密快照 (pickle)。
*   **路径参数:** `session_id` - 会话的 ID。
*   **请求体:**
    ```json
    {
      "pickle_key": "base64_encoded_32_byte_key"
    }
    ```
*   **成功响应 (200 OK):**
    ```json
    {
      "pickle": "base64_encoded_encrypted_pickle_string"
    }
    ```

---

## SAS 验证 (`/sas`)

本节涉及两方之间的短认证字符串 (Short Authentication String) 验证。

### `POST /sas`

*   **目的:** 启动一个新的 SAS 验证过程实例。
*   **成功响应 (201 Created):** 返回此 SAS 实例的 ID 和我们用于 Diffie-Hellman 交换的公钥。
    ```json
    {
      "sas_id": "server-generated-sas-instance-id",
      "public_key": "base64_encoded_curve25519_public_key"
    }
    ```

### `POST /sas/{sas_id}/diffie-hellman`

*   **目的:** 使用对方的公钥执行 Diffie-Hellman 密钥交换。这将建立后续步骤所需的共享密钥。
*   **路径参数:** `sas_id` - SAS 实例的 ID。
*   **请求体:**
    ```json
    {
      "their_key": "base64_encoded_curve25519_public_key" // 对方的公钥
    }
    ```
*   **成功响应 (200 OK):** 表示内部已建立共享密钥。
    ```json
    {
      "established": true
    }
    ```
*   **错误响应 (409 Conflict):** 如果已为此 `sas_id` 执行过 DH。

### `GET /sas/{sas_id}/bytes`

*   **目的:** 在 Diffie-Hellman 交换完成后，生成 SAS 验证字节（可以映射为表情符号或十进制数）。
*   **路径参数:** `sas_id` - SAS 实例的 ID。
*   **查询参数:** `info` (必需) - 上下文字符串 (例如 `"MATRIX_SAS_VERIFICATION"`)。
*   **成功响应 (200 OK):**
    ```json
    {
      "emoji_indices": [10, 5, 63, ...], // 7 个数字 (0-63) 的数组，代表表情符号索引
      "decimals": [1234, 5678, 9012]  // 3 个数字 (1000-9191) 的数组，代表十进制数
    }
    ```
*   **错误响应 (409 Conflict):** 如果尚未建立 Diffie-Hellman。

### `POST /sas/{sas_id}/calculate-mac`

*   **目的:** 使用已建立的共享密钥，为给定的输入字符串和上下文计算消息认证码 (MAC)。
*   **路径参数:** `sas_id` - SAS 实例的 ID。
*   **请求体:**
    ```json
    {
      "input": "string_to_authenticate",
      "info": "mac_context_string" // 此特定 MAC 的上下文
    }
    ```
*   **成功响应 (200 OK):** 返回计算出的 MAC 标签。
    ```json
    {
      "mac": "base64_encoded_mac_tag"
    }
    ```
*   **错误响应 (409 Conflict):** 如果尚未建立 Diffie-Hellman。

### `POST /sas/{sas_id}/verify-mac`

*   **目的:** 使用已建立的共享密钥，根据给定的输入字符串和上下文，验证从对方收到的 MAC 标签。
*   **路径参数:** `sas_id` - SAS 实例的 ID。
*   **请求体:**
    ```json
    {
      "input": "string_that_was_authenticated",
      "info": "mac_context_string", // 必须与发送方使用的上下文匹配
      "tag": "base64_encoded_mac_tag_to_verify" // 从对方收到的要验证的 MAC
    }
    ```
*   **成功响应 (200 OK):** 指示验证是通过还是失败。
    ```json
    {
      "verified": true // 或 false
    }
    ```
*   **错误响应 (409 Conflict):** 如果尚未建立 Diffie-Hellman。
