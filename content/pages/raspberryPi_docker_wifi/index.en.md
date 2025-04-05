---
title: "Running OpenWrt on Raspberry Pi 4B via Docker for Wireless Routing"
description: "Running OpenWrt on Raspberry Pi 4B via Docker for Wireless Routing"
weight: 1
draft: false
tags: ["openWrt", "docker", "Raspberry Pi", "Tinkering", "Tutorial"]
---

### Introduction
Although the Raspberry Pi 4B has relatively strong performance, its network interface is not excellent, only suitable as a 100Mbps wireless router. Deploying OpenWrt via Docker allows the Raspberry Pi to function as a router while hosting other services. Note: This setup is more about tinkering than practical use, and using Docker for multiple services might cause instability in the wireless soft router.

### Preparations
1. Raspberry Pi
2. SD Card
3. Raspberry Pi os (make sure your Raspberry Pi os is working properly)

### 1. Enable Ethernet Card Promiscuous Mode
    
```bash
$sudo nano /etc/network/interfaces
# Modify to (add) the following content
# interfaces(5) file used by ifup(8) and ifdown(8)
# Include files from /etc/network/interfaces.d:
source /etc/network/interfaces.d/*

auto eth0
allow-hotplug eth0
iface eth0 inet dhcp
    up ip link set eth0 promisc on
```
    
### 2. Set Up Wireless Hotspot (AP)
Create a hotspot via the graphical interface and set a static IPv4 address to 192.168.0.254.

### 3. Install and Configure Docker
    
```bash
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl enable docker
sudo systemctl start docker
# Add the current user to the Docker user group
sudo gpasswd -a $USER docker
newgrp docker
sudo modprobe veth
```
    
### 4. Create Docker Network

```bash
docker network create -d macvlan --subnet=192.168.0.0/24 --gateway=192.168.0.253 -o parent=wlan0 macwan
docker network create -d macvlan --subnet=x.x.x.x/24 --gateway=x.x.x.x -o parent=eth0 macwan
```

Note: The subnet and gateway for the macwan network must match your router's subnet.
    
### 5. Create Container
    
```bash
docker run --restart always --name openwrt -d --network maclan --privileged --ip 192.168.0.1 "registry.cn-shanghai.aliyuncs.com/suling/openwrt:rpi4" /sbin/init
```
    
- The container image address is for the Chinese version of OpenWrt. You can change it to the official openWrt.

### 6. Modify Docker Network Configuration
    
```bash
# Enter the container
docker exec -it openwrt /bin/sh

#
vi /etc/config/network
```

Modify to the following:

```bash
config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'static'
        option ipaddr '192.168.0.1'
        option netmask '255.255.255.0'
        option ip6assign '60'
```

After modification, restart the network:

```bash
/etc/init.d/network restart
```

### 7. Connect macwan to Container (Host)
    
```bash
docker network connect macwan openwrt
```
    
### 8. Access OpenWrt Console

Connect another device to the Raspberry Pi AP and enter 192.168.0.1 in the browser to access the OpenWrt console. If you cannot access it, check if your device's IP is in the same subnet as the Raspberry Pi AP. If not, enter it manually.

After accessing, add the WAN network interface:

{{< figure
    src="raspberryPi_docker_wifi.png"
    alt="Add Network Interface"
    >}}

Select the interface to include as eth1.

Save and apply the changes, and the devices connected to the Raspberry Pi AP should now be online!