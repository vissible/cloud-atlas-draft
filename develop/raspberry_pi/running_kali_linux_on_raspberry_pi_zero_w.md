[Kali Linux](../../os/linux/kali/README)是专注于安全渗透的Linux发行版，不仅提供了x86_64版本，也提供基于ARM的版本，可以安装在树莓派上。

本文是在小巧的树莓派Zero W上安装运行Kali Linux，并且借助Zero W的USB扩展卡，可以方便地将Kali Linux随身携带，只需要插入主机USB接口就可以工作：

![Raspberry Pi Zero USB连接扩展卡保护壳](../../img/develop/raspberry_pi/zero_usb_2.jpg)

# 下载和镜像

[Offensive Security官方网站提供Kali Linux ARM Images](https://www.offensive-security.com/kali-linux-arm-images/)，提供了不同ARM版本的镜像，其中不仅有 Raspberry Pi 2/3 的版本，也有 Raspberry Pi Zero W版本。

下载[Raspberry Pi Zero W版本kali-linux-2018.1](https://images.offensive-security.com/arm-images/kali-linux-2018.1-rpi0w-nexmon.img.xz)是`.xz`压缩版本（不是ISO），通过一下命令解压缩并写入到U盘（这里实际上是TF卡加了USB套，在linux上显示为`/dev/sdb`）

```
xzcat kali-linux-2018.1-rpi0w-nexmon.img.xz | \
dd bs=4M of=/dev/sdb iflag=fullblock oflag=direct status=progress
```

> 注意：这里使用的`of=/dev/sdb`是因为U盘识别为`sdb`，具体要看TF卡转成U盘插入Linux系统主机识别的设备名。

# 设置USB方式网络通讯

和[树莓派Zero设置USB网络通讯(Ethernet Gadget)](raspberry_pi_zero_ethernet_gadget)相同，设置通过USB通讯

* 挂载U盘：

```
sudo mount /dev/sdb2 /mnt
sudo mount /dev/sdb1 /mnt/boot
```

* 编辑`config.txt`（Kali Linux默认似乎没有这个文件，所以创建），在最后添加一行：

```
dtoverlay=dwc2
```

* 编辑`cmdline.txt`在`rootwait`之后加上一个空格，以及`modules-load=dwc2,g_ether`。完整配置如下：

```
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=792bacf6-02 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait modules-load=dwc2,g_ether
```

> 我修改了Kali Linux的`cmdline.txt`文件中`root=/dev/mmcblk0p`，改成`root=PARTUUID=792bacf6-02`。这是因为我通过USB转接卡后，设备名称会改变，所以通过`PARTUUID`来确保设备识别实际不会变化。

* 编辑`/mnt/etc/network/interfaces`配置如下

```
allow-hotplug usb0
iface usb0 inet static
        address 192.168.7.10
        netmask 255.255.255.0
        network 192.168.7.0
        broadcast 192.168.7.255
        gateway 192.168.7.1
        dns-nameservers 192.168.7.1
```

# 启动

卸载挂载的U盘

```
sudo umount /mnt/boot
sudo umount /mnt
```

然后将TF卡插入树莓派Zero W，通过USB方式插入到一台Linux主机上，此时Linux主机会自动发现新的网卡设备，例如`enp0s29u1u1`

通过在Linux主机上`/etc/network/interfaces`添加如下内容：

```
auto enp0s29u1u1
iface enp0s29u1u1 inet static
        address 192.168.7.1
        netmask 255.255.255.0
        network 192.168.7.0
        broadcast 192.168.7.255
```

然后启动`sudo ifup enp0s29u1u1`就`ping 192.168.7.10`，网络通了之后，就可以`ssh 192.168.7.10`，默认账号`root`的密码是`toor`，请立即修改。

# 参考

* [Hands-On: Kali Linux 2018.1 on the Raspberry Pi Zero W](http://www.zdnet.com/article/hands-on-kali-linux-2018-1-on-the-raspberry-pi-zero-w/)
* [How to customise your Linux desktop: Kali Linux and i3 Window Manager](http://www.zdnet.com/article/how-to-customise-your-linux-desktop-kali-linux-and-i3-window-manager/)