# 添加jdk
```shell
sudo apt install openjdk-17-jdk
```
#  添加 android-studio 存储库
```shell
sudo add-apt-repository ppa:maarten-fonville/android-studio
#因为会自动更新apt
```
# 安装Android Studio
```shell
sudo apt install android-studio -y
```
# 安装kvm
```shell
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils virt-manager virtinst
```
## 添加用户组
```shell
sudo adduser $USER libvirt
sudo adduser $USER kvm
```
## 检验安装
```shell
ls -l /dev/kvm
```
# 打开Android Studio 
进行基础环境配置
