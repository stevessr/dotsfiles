---
title: 安装 Niri 与 WayVNC
tags:
  - niri
  - wayvnc
  - arch
  - kde
date: {{date:YYYY-MM-DD}}
---

# 📦 安装 Niri（Wayland 合成器） & WayVNC

> 本指南适用于 **Arch Linux**（或基于 Arch 的发行版），使用 **yay**（AUR 助手）进行软件安装。  
> 若您使用其他发行版，只需将 `yay` 替换为对应的包管理器即可。

## 目录
1. [前置准备](#前置准备)
2. [安装 WayVNC](#安装-wayvnc)
3. [常见兼容性提示](#常见兼容性提示)
4. [后续配置建议](#后续配置建议)
5. [故障排查](#故障排查)

---

## 前置准备

- 确认系统已安装 **yay**（AUR 包管理器）。如果没有：

  ```bash
  sudo pacman -S --needed base-devel git
  git clone https://aur.archlinux.org/yay.git
  cd yay
  makepkg -si
  ```

- 确保已经安装 **Niri**（Wayland 合成器），如果尚未安装：

  ```bash
  yay -S niri
  ```

- 启动 Niri 前，请确保已关闭 Xorg 会话或其他 Wayland 合成器（如 `sway`、`kwin`）。

---

## 安装 WayVNC

```shell
yay -S wayvnc
```

> **Tip**：安装过程会自动拉取所需的依赖，包括 `libvncserver`、`wayland-protocols` 等。

### 验证安装

```bash
wayvnc --version
```

若输出版本号即表示安装成功。

---

## 常见兼容性提示

> [!TIP]  
> **KDE (KWin)** 似乎不支持 WayVNC。若您在 KDE 环境下使用 Wayland，会出现以下现象：
> - VNC 客户端无法连接
> - 启动 `wayvnc` 时报错 “No Wayland compositor found”

**解决方案**  
- 使用 **Niri**、**Sway**、**Weston** 等原生 Wayland 合成器。  
- 若必须在 KDE 环境下使用远程桌面，可考虑 **x11vnc**（基于 Xorg）或 **TigerVNC**（需要 XWayland 支持）。

---

## 后续配置建议

| 项目 | 推荐值 / 操作 | 备注 |
|------|----------------|------|
| **VNC 端口** | `5900`（默认） | 如需自定义，可在 `~/.config/wayvnc/config` 中添加 `port = 5901` |
| **密码** | 使用 `vncpasswd` 生成 | 将生成的 `passwd` 文件放在 `~/.config/wayvnc/` |
| **分辨率** | `1920x1080`（或根据显示器） | 在 `config` 中加入 `geometry = 1920x1080` |
| **自动启动** | `systemd --user` 服务 | ```bash\n[Unit]\nDescription=WayVNC Service\nAfter=graphical-session.target\n\n[Service]\nExecStart=/usr/bin/wayvnc -C ~/.config/wayvnc/config\nRestart=on-failure\n\n[Install]\nWantedBy=default.target\n``` |

创建并启用服务：

```bash
systemctl --user enable --now wayvnc.service
```

---

## 故障排查

| 症状 | 可能原因 | 检查/解决办法 |
|------|----------|--------------|
| **无法连接 VNC** | Wayland 合成器未运行 | `pgrep niri` / `pgrep sway` 确认合成器进程 |
| **提示 “No Wayland compositor found”** | 使用 KDE/KWin | 切换到 Niri 或 Sway，或改用 X11 VNC |
| **VNC 客户端显示黑屏** | 未正确设置 `geometry` | 在 `~/.config/wayvnc/config` 中添加 `geometry = 1920x1080` |
| **密码验证失败** | `passwd` 文件路径错误 | 确认 `~/.config/wayvnc/passwd` 存在且权限为 `600` |

---

### 参考文档

- **WayVNC 官方仓库** – https://github.com/any1/wayvnc  
- **Niri 项目主页** – https://github.com/YaLTeR/niri  
- **Arch Wiki – Wayland** – https://wiki.archlinux.org/title/Wayland  

---

> **小结**  
> 1️⃣ 安装 `wayvnc` 前确保已在 Wayland 环境（推荐 Niri）中运行。  
> 2️⃣ KDE/KWin 对 WayVNC 支持不佳，建议切换合成器或改用 X11 VNC。  
> 3️⃣ 配置好端口、密码、分辨率后，可通过 systemd 用户服务实现开机自启。

祝您使用愉快 🎉
```