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

Server = https://mirrors.aliyun.com/archlinux/$repo/os/$arch

- 更新软件包缓存

```c
pacman -Syyu
```

- 安装必要的软件包

如果zen内核找不到，就把zen关键字去掉，安装普通内核

```c
pacstrap -K /mnt base base-devel linux-zen linux-firmware linux-zen-headers grub git openssh networkmanager neovim efibootmgr
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
### 配置系统字体
```bash
# ~/.config/fontconfig/fonts.conf
# 增加如下配置
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
    <alias>
        <family>monospace</family>
        <prefer>
            <family>FiraCode Nerd Font Mono</family>
        </prefer>
    </alias>
</fontconfig>

fc-cache -fv
fc-match monospace
```
### 配置kitty终端字体
参考https://www.ypplog.cn/%e6%88%91%e5%8f%88%e6%8d%a2%e5%9b%9ekitty%e7%bb%88%e7%ab%af%e4%ba%86%ef%bc%88kitty%e7%bb%88%e7%ab%af%e7%be%8e%e5%8c%96%e9%85%8d%e7%bd%ae%ef%bc%89/

## 安装dwm

- 安装字体

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

- 通过登录管理器进入DWM
  
- 安装补丁
```c
下载alpha, barpadding, autostart, uselessgap

//patch < 补丁文件来安装
//如果有冲突手动解决
```

- 安装picom
```c
sudo pacman -S picom
mkdir ~/.config/picom
cp /etc/xdg/picom.conf ~/.config/picom
nvim ~/.config/picom/picom.conf
//将use-damage = false

//在rules节点增加如下内容，使窗口透明
rules: ({
  match = "class_g = 'Alacritty' && focused";
  opacity = 0.9;
}, {
  match = "class_g = 'Alacritty' && !focused";
  opacity = 0.6;
})
```
- 安装autostart插件的script
```c
//参考配置文件
https://gitee.com/jzz777/dwm

将suckless/dwm/script/autostart.sh 拷贝到~/.dwm/下
赋予可执行权限

sudo pacman -S bc xorg-xsetrootdate
```
-安装yazi
参考yazi官方github

- 安装壁纸
```c
sudo pacman -S nitrogen
nitrogen /path/to/image/directory/
```

## 安装其他应用

- 安装miniforge配置python环境

- 安装梯子

- 安装oh-my-zsh
```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
- 安装lazy-vim
https://www.lazyvim.org/installation

- 安装输入法
https://www.cnblogs.com/klelee/p/archlinux-fcitx5.html

- 安装archlinuxcn仓库
https://wiki.archlinuxcn.org/wiki/Arch_Linux_%E4%B8%AD%E6%96%87%E7%A4%BE%E5%8C%BA%E4%BB%93%E5%BA%93

- 安装maple字体
https://github.com/subframe7536/maple-font

- 安装rofi
```c
sudo pacman -S rofi
github.com/adi1090x/rofi
```

- 安装amixer
```c
sudo pacman -S alsa-utils

nvim ~/.asoundrc
添加如下内容
defaults.pcm.card 1
defaults.pcm.device 0
defaults.ctl.card 1

# 我有两张声卡 card 0:HDMI card 1: analog
# 需要配置成默认使用card 1
保存后重启生效
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

- 安装xrandr
```c
sudo pacman -S xorg-xrandr
xrandr --output eDP-1 --mode 1920x1080 --rate 60
```

## 安装hyprland

```c
sudo pacman -S kitty
yay -S hyprland-git
```
### 创建虚拟桌面
```bash
# https://wiki.hypr.land/Configuring/Using-hyprctl/#output
hyprctl output create headless test
hyprctl output remove test

sudo pacman -S wayvnc
wayvnc -o HDMI-A-1 0.0.0.0 5901
wayvnc -o HEADLESS-2 0.0.0.0 5901
```
