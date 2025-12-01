
```fallback
sudo pacman -S archlinuxcn-keyring
pacman -S yay
pacman -S xorg-xinit
pacman -S base-devel
pacman -S pulseaudio
pacman -S sbc bluez
pacman -S xorg-xrdb
pacman -S xfce4 xfce4-goodies
yay -S xorgxrdp
```

```shell
sudo pacman -S lightdm lightdm-gtk-greeter
sudo systemctl enable lightdm.service
sudo systemctl start lightdm.service
```

[Xfce - ArchWiki](https://wiki.archlinux.org/title/Xfce)
# 先开个代理
```shell
export proxy="http://172.22.29.228:7897"
export http_proxy="http://172.22.29.228:7897"
export https_proxy="http://172.22.29.228:7897"
```

```shell
export proxy=""
export http_proxy=""
export https_proxy=""
```

```shell
export proxy="http://127.0.0.1:7897"
export http_proxy="http://127.0.0.1:7897"
export https_proxy="http://127.0.0.1:7897"
```