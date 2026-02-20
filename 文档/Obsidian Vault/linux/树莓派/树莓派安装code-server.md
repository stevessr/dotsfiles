## 安装code-server
### 官网一键安装
```shell
curl -L https://coder.com/install.sh | sh
```
### 手动安装
```shell
wget https://github.com/coder/coder/releases/download/v2.18.2/coder_2.18.2_linux_arm64.deb
sudo dpkg -i coder_2.18.2_linux_arm64.deb
```
## 安装个node js
```shell
wget https://nodejs.org/dist/v22.13.0/node-v22.13.0-linux-arm64.tar.xz
tar -xvf node-v22.13.0-linux-arm64.tar.xz
cd node-v22.13.0-linux-arm64
sudo cp -R * /usr/local/
```
## 启动 code-server
```shell
coder --url 0.0.0.0 server
```