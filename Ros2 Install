# 安装ROS2
## 安装docker
```bash
sudo pacman -S docker
sudo systemctl enable docker
sudo systemctl start docker
```

tips:
如果在启动docker daemon前运行了sudo pacman -Syu更新系统，会导致启动失败。需要重启电脑

## 配置docker VPN
```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo nvim /etc/systemd/system/docker.service.d/http-proxy.conf
```
添加如下内容

```bash
[Service]
Environment="HTTP_PROXY=socks5://127,0,0,1:1080/"
Environment="HTTPS_PROXY=socks5://127,0,0,1:1080/"
```
```bash
sudo docker pull osrf/ros:humble-desktop
```

enjoy ros2
