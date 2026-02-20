---
title: Audio Setup on Arch Linux
tags: 
  - audio
  - arch
  - alsa
date: {{date:YYYY-MM-DD}}
---

# 🎧 Arch Linux 音频配置指南

本指南帮助您在 Arch Linux 上快速配置 **ALSA**（Advanced Linux Sound Architecture），解决常见的混音、静音以及驱动缺失问题。

## 1️⃣ 安装必要的软件包

```bash
# 安装 alsa-utils（提供 alsamixer、aplay、arecord 等工具）
sudo pacman -S alsa-utils

# 安装软固件（SOF – Sound Open Firmware），多数现代声卡需要它
sudo pacman -S sof-firmware
```

> **提示**：安装完 `sof-firmware` 后请 **重启** 系统，使固件生效。

## 2️⃣ 调整混音器设置

1. 打开混音器界面  
   ```bash
   alsamixer
   ```
2. 使用方向键将所有通道的音量调到 **最大**（或您需要的水平）。  
3. 若出现 **M**（已静音）标记，按 **`m`** 取消静音。  

> **小技巧**：在 `alsamixer` 中可以按 **F6** 切换声卡，确保对正确的设备进行调节。

## 3️⃣ 保存设置

```bash
sudo alsactl store
```

此命令会将当前的混音器状态写入 `/var/lib/alsa/asound.state`，系统重启后会自动恢复。

## 4️⃣ 验证音频输出

```bash
# 播放测试音频（需要安装 aplay 或者使用 speaker-test）
speaker-test -t wav -c 2
```

如果听到左右声道交替的噪音，说明音频已正常工作。

---

### 📌 常见问题排查

| 问题 | 可能原因 | 解决方案 |
|------|----------|----------|
| 没有声音 | 声卡未被识别 | 确认已安装 `sof-firmware`，并检查 `dmesg` 中的固件加载日志 |
| 部分通道静音 | `alsamixer` 中未取消静音 | 在 `alsamixer` 中按 `m` 取消所有 **M** 标记 |
| 重启后音量恢复为默认 | 未执行 `alsactl store` | 再次运行 `sudo alsactl store` 并确认 `/var/lib/alsa/asound.state` 已更新 |

---

### 📎 参考链接

- Arch Wiki – [ALSA](https://wiki.archlinux.org/title/Advanced_Linux_Sound_Architecture)
- SOF 项目主页 – https://www.sofproject.org/

---

> **备注**：如果您在其他发行版（如 Ubuntu、Fedora）上使用相同步骤，只需将 `pacman` 替换为对应的包管理器（`apt`、`dnf`）即可。祝您使用愉快！