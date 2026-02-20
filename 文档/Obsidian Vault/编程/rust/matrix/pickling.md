# Pickling & 持久化

[返回索引](index.md)

Vodozemac 支持序列化帐户和会话状态以实现持久化。

*   **现代 Pickling (`serde`)**:
    *   使用 `serde` 进行序列化/反序列化。
    *   对象提供 `pickle()` 方法，返回可序列化的 `*Pickle` 结构体（例如，`AccountPickle`, `SessionPickle`, `GroupSessionPickle`, `InboundGroupSessionPickle`）。
    *   `*Pickle` 结构体通常具有 `encrypt()` 和 `from_encrypted()` 方法，用于使用用户提供的密钥进行安全存储。
    *   错误类型: `PickleError`。
*   **遗留 Pickling (`libolm-compat`)**:
    *   受 `libolm-compat` 功能标志控制。
    *   在主要对象（`Account`, `Session` 等）上提供 `from_libolm_pickle()` 方法，以加载由 libolm pickling 的数据。
    *   可能提供 `to_libolm_pickle()` 方法用于导出（例如，`PkDecryption`）。
    *   需要 `pickle_key` 进行解密/加密。
    *   错误类型: `LibolmPickleError`。
*   **脱水设备 (`dehydration`)**:
    *   受 `dehydration` 功能标志控制。
    *   `Account::to_dehydrated_device()`: 创建适合服务器存储的加密备份字符串。
    *   `Account::from_dehydrated_device()`: 从脱水备份恢复帐户。
    *   错误类型: `DehydratedDeviceError`。
