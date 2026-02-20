# Vodozemac 协议文档

本文档概述了 Vodozemac 库提供的公共 API 和数据结构。Vodozemac 实现了 Matrix 中使用的加密协议。

## 模块

*   [**通用类型**](common_types.md): 库中使用的基本加密类型和错误类型。
*   [**Olm 协议 (1:1 加密)**](olm.md): 用于 1:1 聊天的双棘轮加密。
*   [**Megolm 协议 (群组加密)**](megolm.md): 用于群聊的加密。
*   [**短认证字符串 (SAS)**](sas.md): 用于带外密钥验证。
*   [**公钥 (PK) 加密 (Megolm Backup v1)**](pk_encryption.md): 用于 Matrix 密钥备份的加密方案（**警告：存在已知安全缺陷**）。
*   [**ECIES (集成加密方案)**](ecies.md): 用于安全通道建立（例如二维码登录）。
*   [**Pickling & 持久化**](pickling.md): 序列化和持久化帐户及会话状态。
