[tof]
# 因为tf卡炸了，再来安装一次
# # Raspberry Pi Connect (Beta)
## Connect Lite
```bash
sudo apt update
sudo apt full-upgrade
sudo apt install rpi-connect-lite
rpi-connect on
rpi-connect signin
```
# 使用清华大学的镜像
## Raspbian 软件仓库
### /etc/apt/sources.list.d/raspi.list
```list
deb https://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ bookworm main non-free contrib rpi
# deb-src https://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/ bookworm main non-free contrib rpi

deb [arch=arm64] https://mirrors.tuna.tsinghua.edu.cn/raspbian/multiarch/ bookworm main
```
### /etc/apt/sources.list
```list
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

# 以下安全更新软件源包含了官方源与镜像站配置，如有需要可自行修改注释切换
deb https://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware
# deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware
```
## 添加公钥！！！
```shell
gpg --keyserver  keyserver.ubuntu.com --recv-keys 9165938D90FDDD2E
gpg --export --armor  9165938D90FDDD2E | sudo apt-key add -
gpg --keyserver  keyserver.ubuntu.com --recv-keys E77FC0EC34276B4B
gpg --export --armor  E77FC0EC34276B4B | sudo apt-key add -
```

# bottom

```shell
wget https://github.com/ClementTsang/bottom/releases/download/0.10.2/bottom_0.10.2-1_arm64.deb
#如果网络不行，请自行寻找镜像站进行下载
sudo dpkg -i bottom_0.10.2-1_arm64.deb
```
# pipx
```bash
sudo apt update 
sudo apt install pipx 
pipx ensurepath
eval "$(register-python-argcomplete pipx)"
```
# git
```bash
sudo apt update
sudo apt install git -y
```
# zsh
```bash
sudo apt install zsh
```
## oh my zsh
```bash
sh -c "$(curl -fsSL https://install.ohmyz.sh/)"
# or 
sh -c "$(wget -O- https://install.ohmyz.sh/)"
# or
sh -c "$(fetch -o - https://install.ohmyz.sh/)"
```
# alist
[一键脚本 | AList文档](https://alist.nn.ci/zh/guide/install/script.html)
## 脚本
### **安装**

```shell
curl -fsSL "https://alist.nn.ci/v3.sh" | sudo bash -s install
```

### **更新**

```shell
curl -fsSL "https://alist.nn.ci/v3.sh" | sudo bash -s update
```

### **卸载**

```shell
curl -fsSL "https://alist.nn.ci/v3.sh" | sudo bash -s uninstall
```

## **自定义路径**

默认安装在 `/opt/alist` 中。 自定义安装路径，将安装路径作为第二个参数添加，必须是绝对路径（如果路径以 alist 结尾，则直接安装到给定路径，否则会安装在给定路径 alist 目录下），如 安装到 `/root`：

### 正式版

```
# Install
curl -fsSL "https://alist.nn.ci/v3.sh" | sudo bash -s install /root
# update
curl -fsSL "https://alist.nn.ci/v3.sh" | sudo bash -s update /root
# Uninstall
curl -fsSL "https://alist.nn.ci/v3.sh" | sudo bash -s uninstall /root
```

- 启动: `systemctl start alist`
- 关闭: `systemctl stop alist`
- 状态: `systemctl status alist`
- 重启: `systemctl restart alist`
# cloudflared
```shell
curl -L --output cloudflared.deb https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-arm64.deb && 

sudo dpkg -i cloudflared.deb && 

sudo cloudflared service install eyJhIjoiOTAyYzU2MWFkMDFmM2M1OTZmYTdiMTIyZGEzODM4ZDAiLCJ0IjoiNWY4Y2E0NmUtYjE4YS00NTIxLWJlMzItNGJlMDFlOWNhNWNlIiwicyI6Ik5HUXpOelJrTWpVdFlUWmhaaTAwTXpGakxUaGpZbUl0TWpVNFpHUmpabU01WlRZMCJ9
```