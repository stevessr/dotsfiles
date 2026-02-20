# 通用类型

[返回索引](index.md)

本文档描述了 Vodozemac 库中跨多个模块使用的基本加密类型和错误类型。

*   **`Curve25519PublicKey`**: 代表 X25519 Diffie-Hellman 函数的公钥。
*   **`Curve25519SecretKey`**: 代表 X25519 Diffie-Hellman 函数的密钥。
*   **`Ed25519PublicKey`**: 代表 Ed25519 签名方案的公钥。
*   **`Ed25519SecretKey`**: 代表 Ed25519 签名方案的密钥。
*   **`Ed25519Signature`**: 代表一个 Ed25519 签名。
*   **`KeyError`**: 与密钥验证或使用相关的错误类型（例如，非贡献性密钥）。
*   **`SignatureError`**: 与签名验证失败相关的错误类型。
*   **`PickleError`**: 现代（基于 `serde`）pickling/unpickling 期间失败的错误类型。
*   **`LibolmPickleError`**: 遗留（`libolm-compat`）pickling/unpickling 期间失败的错误类型。
