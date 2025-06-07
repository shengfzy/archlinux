# Arch Linux Installation Guide

## 安装镜像

- 下载arch 镜像

[Index of /archlinux/iso/2025.06.01/ | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/archlinux/iso/2025.06.01/)

- 官方安装Guide

[Installation guide - ArchWiki](https://wiki.archlinux.org/title/Installation_guide)

- MacOs虚拟机

Parallel Desktop

## 安装步骤

- 硬盘分区

具体分区步骤参考arch 官方安装向导

```c
cfdisk
```

总共分成三个分区

| 分区   | 类型   | 大小   | 挂在点       |
| ---- | ---- | ---- | --------- |
| boot | efi  | 1GiB | /mnt/boot |
| swap | swap | 8GiB | swapon    |
| root | ext4 | free | /mnt      |

- 添加软件源

编辑/etc/pacman.d/mirrorlist

[archlinux | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/archlinux/)

- 更新软件包缓存

```c
pacman -Syyu
```

- 安装必要的软件包

如果zen内核找不到，就把zen关键字去掉，安装普通内核

```c
pacstrap -K /mnt base base-devel linux-zen linux-zen-firmware linux-zen-headers grub git openssh networkmanager neovim efibootmgr
```

- 配置系统

按照官方文档第三章配置

- 安装bootloader

改步骤需要执行配置系统中的chroot后运行

```c
grub-install --target=arm64-efi --efi-directory=/boot --bootloader-id=ARCH
grub-mkconfig -o /boot/grub/grub.cfg
```

- 创建用户

```c
useradd -m -G wheel matteo
nvim /etc/sudoers
```

## keyring

- 更新key

```c
sudo rm -r /etc/pacman.d/gnupg
sudo pacman-key --init
sudo pacman-key --populate archlinux manjaro manjaro-arm
sudo pacman-key --refresh-keys 
sudo pacman -Sc

// 本地信任某个key
sudo pacman-key -lsign-key "builder@archlinuxarm.org"
```

## 安装openssh
- 下载
```c
sudo pacman -S openssh
sudo systemctl start sshd
sudo systemctl enable sshd
```
## 安装nerd fonts
- 下载
```c
sudo pacman -S fontconfig
mkdir -p ~/.local/share/fonts
mv CascadiaMono/*.tff ~/.local/share/fonts
fc-cache -fv
fc-list | grep -i nerd
```

## 安装dwm

- 安装ziti

```c
sudo pacman -S ttf-dejavu ttf-liberation noto-fonts
```

- 安装dwm

```c
sudo pacman -S xorg-server xorg-xinit libxft libxinerama
sudo pacman -S alacritty
mkdir suckless
cd suckless
git clone https://git.suckless.org/dwm
git clone https://git.suckless.org/dmenu
git clone https://git.suckless.org/st
git clone https://git.suckless.org/slstatus

sudo make clean install
```

- 编辑xinitrc

```c
sudo cp /etc/X11/xinit/xinitrc ~/.xinitrc

nvim ~/.xinitrc
// 注释掉结尾的4行
// 添加exec dwm
// startx启动dwm
```

- 通过登录管理器进入DWM
使用root权限到/usr/share/xsessions/新建一个文件，名称可以为dwm.desktop，内容如下：
```c
# /usr/share/xsessions/dwm.desktop

[Desktop Entry]
Encoding=UTF-8
Name=Dwm
Comment=Dynamic window manager
Exec=dwm
Icon=dwm
Type=XSession
```
- 安装登录管理器
```c
sudo pacman -S sddm
```

```c
使用root权限到/usr/share/xsessions/新建一个文件，名称可以为dwm.desktop，内容如下：

# /usr/share/xsessions/dwm.desktop

[Desktop Entry]
Encoding=UTF-8
Name=Dwm
Comment=Dynamic window manager
Exec=dwm
Icon=dwm
Type=XSession
```

```c
sudo systemctl enable sddm
sudo systemctl start sddm
```
- 安装sddm主题
  
- 安装补丁
```c
下载alpha, barpadding, autostart, uselessgap

//patch < 补丁文件来安装
//如果有冲突手动解决
```

- 安装picom
```c
sudo pacman -S picom
sudo nvim /etc/xdg/picom.conf

//在rules节点增加如下内容，使窗口透明
rules: ({
  match = "class_g = 'URxvt' && focused";
  opacity = 0.9;
}, {
  match = "(class_g = 'URxvt' || class_g = 'Alacritty')"
          " && !focused";
  opacity = 0.6;
})
```
- 安装autostart插件的script
```c
//参考配置文件
https://gitee.com/jzz777/dwm

赋予可执行权限

sudo pacman -S bc xorg-xsetrootdate
```

- 安装输入法
https://www.cnblogs.com/klelee/p/archlinux-fcitx5.html

- 安装壁纸
```c
sudo pacman -S nitrogen
nitrogen /path/to/image/directory/
```

## 安装其他应用
- 安装rofi
```c
sudo pacman -S rofi
github.com/adi1090x/rofi
```

- 安装amixer
```c
sudo pacman -S alsa-utils
```

- 安装yay
```c
sudo pacman -S base-devel git
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```
- 安装chrome
```c
yay -S google-chrome
```
如果部分依赖包下载失败的话，可以去arch官方社区的package下搜索对应的文件安装

用代理启动google-chome
```c
google-chrome-stable --proxy-server="socks://127.0.0.1:1080"
```
