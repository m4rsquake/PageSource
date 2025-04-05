---
title: "树莓派4B使用docker运行openwrt实现无线路由"
description: "树莓派4B使用docker运行openwrt实现无线路由"
weight: 1
draft: false
tags: ["openWrt", "docker", "树莓派", "折腾", "教程"]
---

### 前言
虽然树莓派4B的性能相对较强，但其网络接口的性能却并不算优秀，可作为百兆无线路由器。
使用 docker 部署 openWrt，可以在树莓派作为路由器的同时，部署其他服务。注意：该方案折腾大于使用，使用 docker 多合一可能会导致无线软路由不稳定。

### 准备工作
1. 树莓派
2. sd卡
3. 树莓派系统

### 1. 更换树莓派OS软件源
直接根据以下网址换源
[raspbian | 镜像站使用帮助 | 清华大学开源软件镜像站 | Tsinghua Open Source Mirror](https://mirrors.tuna.tsinghua.edu.cn/help/raspbian/)
### 2. 开启以太网网卡混杂模式
    
```bash
$sudo nano /etc/network/interfaces
#修改为（添加）以下内容
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source /etc/network/interfaces.d/*

auto eth0
allow-hotplug eth0
iface eth0 inet dhcp
    up ip link set eth0 promisc on
```
    
### 3. 开启无线热点（AP）
通过图形界面创建一个热点，并配置静态v4地址为 192.168.0.254

### 4. 安装docker并配置
    
```bash
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
# 将当前用户加入docker用户组
sudo gpasswd -a $USER docker
newgrp docker
sudo modprobe veth
```
    
### 5. 创建docker网络

```bash
docker network create -d macvlan --subnet=192.168.0.0/24 --gateway=192.168.0.253 -o parent=wlan0 macwan
docker network create -d macvlan --subnet=x.x.x.x/24 --gateway=x.x.x.x -o parent=eth0 macwan
```

注意，此处的macwan网络的子网与网关需要和你当前的路由器子网相同，原因是docker在创建网络时强制要求
    
### 6. 创建container
    
```bash
docker run --restart always --name openwrt -d --network maclan --privileged --ip 192.168.0.1 "registry.cn-shanghai.aliyuncs.com/suling/openwrt:rpi4" /sbin/init
```
    
- 此处容器镜像地址为openwrt的简中immoralrt，网址为[在树莓派上使用Dockers运行Openwrt并作为主路由器的旁路 - HXSup (ahsup.top)](https://www.ahsup.top/post/linux/openwrt/)

### 7. 修改docker网络配置
    
```bash
# 进入容器
docker exec -it openwrt /bin/sh

#
vi /etc/config/network
```

主要修改成如下所示：

```bash
config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.0.1'
        option netmask '255.255.255.0'
        option ip6assign '60'
```

修改完成后，重启网络

```bash
/etc/init.d/network restart
```

### 8. 连接macwan到container（host）
    
```bash
docker network connect macwan openwrt
```
    
### 9. 进入openwrt控制台
    
使用另一台设备连接树莓派AP，并在浏览器输入192.168.0.1进入openwrt控制台，如果无法进入，检查本机得到的ip是否和树莓派AP的ip地址为同一网段，不是则手动输入

进入后添加网络接口WAN

{{< figure
    src="raspberryPi_docker_wifi.png"
    alt="添加网络接口"
    >}}

并选择包括接口为eth1
    
最后保存并应用，发现连上树莓派AP的设备已经能联网！