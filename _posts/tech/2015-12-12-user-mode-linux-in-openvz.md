---
layout: default
title: User Mode Linux in OpenVZ
category: tech
type: pink-color
---
做那些你在OpenVZ VPS中不能做的事情。

### 起因

我为什么突然想到折腾 `OpenVZ` ？那是因为我这两天在物色新的VPS，因为实在用不起阿里云中转+ConoHa，而高富帅 `Linode` 又没有可以买到的好线路，它的新加坡线路在我这里甚至只有通过贵成狗的微林香港中转才能勉强使用。在这个时候我了解到了 `SoftLayer` 机房，并通过其官网的速度测试发现新加坡机房到我这里的延迟很低而下载速度很快。但是其官网的价格 (US$28.5/mo) 实在是令我望尘莫及，而又有人告诉我老鹰主机 ([HawkHost](https://my.hawkhost.com/aff.php?aff=7713)) 出售的VPS使用的是 `SoftLayer` 的线路，价格也比官网低，可用流量也多出一些。正巧这两天他家在打折又能用支付宝，我就毫不犹豫地剁手买了一台他家新加坡节点的VPS。但是买了以后我才发现这玩意是 `OpenVZ` 虚拟化技术。

我倒不是说怕 `OpenVZ` 虚拟化的VPS会超售，毕竟所有虚拟化技术都是有办法超售的。我所不喜欢的是 `OpenVZ` 不能运行自己的自定义内核，这直接导致如 `ArchLinux` 这样的滚动发行版无法很好地运行并保持最新。而且 `OpenVZ` 内的环境也有很多的限制，比如说无法使用 `sit` 设备，无法使用某些新的TCP拥塞算法等。

但是这家出售的VPS在我这里又实在是速度太快了，要我舍弃这样的东西也不大可能，况且还是个优惠价买的东西。是否真的没有解决办法呢？也不是。

### UML

这个时候我想起以前我曾经撇见过的一个东西，叫 `User-Mode-Linux` ，简称 `UML` 。我是在 `Linux Kernel` 的说明文件里面瞥见的

> Linux has also been ported to itself. You can now run the kernel as a userspace application - this is called UserMode Linux (UML).

也就是说 `Linux` 似乎可以编译成一个能在它本身上面运行的可执行程序。我以前瞥见的时候以为这东西只是一个玩具功能，可能只能用来调试而不适合日常使用，可能效率很低或者bug很多……

但是这个时候我似乎只有这一个选择。其他的虚拟化，都不可能在 `OpenVZ` 内再跑起来。而 `qemu` 这种软件虚拟机，虽然可能跑起来，但那效率……估计不卡死也算是不错了。

所以我就尝试编译并运行 `UML` 并在它里面运行一个 `ArchLinux`

### 编译

首先，因为我在VPS上选择的是 `CentOS 7`, 所以必须先安装一堆基本的开发工具，如 `make` `bc` `gcc` 等，我就不赘述了。

接下来当然是下载并解压内核源码。直接从 <https://www.kernel.org> 下载最新的主线内核就可以了，因为 `UML` 已经被合并进主线内核了。然后，我在 `AUR (ArchLinux)` 上找到了这样一个包 [linux-usermode](https://aur.archlinux.org/packages/linux-usermode/), 从它的 `snapahot` 里面可以找到写好的内核配置，这样我也就偷了个小懒。

把那个内核配置放进内核源码根目录改为 `.config` 以后，我就开始编译。要注意的是因为我们不是在做正常的编译，所以必须指定编译目标为 `User-Mode-Linux`

```bash
make ARCH=um vmlinux
```

当然，直接这样编译会报错，会提示找不到 `vde` 的库的头文件。当然，`OpenVZ` 也是不可能支持的。所以，只需要修改配置文件，把有关 `VDE` 的选项关闭后再编译即可。

如果编译完成，那么在内核源码根目录下会生成一个可执行文件 `vmlinux`, 这就是一个可运行的 `UML` 内核了。将它拷贝到一个空的目录里面即可。

### 文件系统

`Linux` 内核需要一个根文件系统才能启动。而 `User-Mode-Linux` 虽然支持从主机的文件系统启动，但不允许对它做任何修改，所以我们只能创建一个磁盘镜像作为根分区来启动内核。

使用 `fallocate` 命令可以快速创建一个大文件

```bash
fallocate -l 25G rootfs.img
```

这样就在当前目录下创建了一个25G的空文件 `rootfs.img`，可以作为磁盘镜像使用。

但是当我尝试格式化它并挂载出来以便 `bootstrap` 的时候，问题出现了。`OpenVZ` 环境下，你无法创建 `loop` 设备，也自然就无法挂载这样的磁盘镜像。怎么办呢？我只好采取了一个 `workaround`。

因为 `UML` 允许部分访问主机的文件系统，所以我在当前目录下建立了一个空目录，名为 `root`，并从 `ArchLinux` 的镜像源里面下载了一个 `bootstrap` 包裹 (可以从 <http://mirror.rackspace.com/archlinux/iso/2015.12.01/> 找到，含 `bootstrap` 文件名的 `tar.gz` 文件就是)，把它解包丢进了这个目录。

做这个目录的目的是在里面配置一个可用的临时系统，用这个临时的系统启动 `UML`, 在里面把刚刚那个磁盘镜像挂载出来，然后再对它进行 `bootstrap` 操作。

所以我们现在需要在这个目录里面配置基本系统。首先当然是把里面的 `etc/pacman.d/mirrorlist` 配置指向一个可用的源，再修改 `resolv.conf` 指向可用的DNS。接下来，我们要挂载一些必要的东西进去 (假设你的这个临时根目录的路径是 `/path/to/root`)

```bash
mount --rbind /proc /path/to/root/proc
mount --rbind /sys /path/to/root/sys
mount --rbind /dev /path/to/root/dev
mount -t tmpfs tmpfs /path/to/root/tmp
mount --rbind /path/to/root /path/to/root
```

然后就可以执行 `chroot /path/to/root /bin/bash` 进去了。这个时候，我们先在里面安装基本系统

```bash
pacman -Sy base
```

然后，为了让它能在 `UML` 下直接操作，我们需要执行

```bash
systemctl enable getty@tty0
systemctl disable getty@tty1
```

这是因为 `UML` 默认的可直接操作的是 `tty0` 而非 `tty1`

然后我们就可以退出这个 `chroot`, 并执行

```bash
umount /path/to/root/{dev,proc,sys,tmp}
```

如果报错，建议直接重启解决。

接下来我们就可以启动一个 `UML` 试试看了。

```bash
./vmlinux root=/dev/root rootfstype=hostfs hostfs=/path/to/root ubd0=rootfs.img
```

你应该已经能以 `root` 身份直接登录进入刚刚配置好的那个环境。这个环境现在是只读的。而 `rootfs.img` 在这个系统里面被映射到了 `/dev/ubda`。

但是你会发现一个大问题，就是没!网!络!

### 网络

所以我们现在必须配置网络。还好，我的这个 `OpenVZ VPS` 支持 `TUN/TAP`, 也就没问题了。如果你的VPS不支持，那就恐怕无法继续进行下去了。

此时，应该退出刚刚启动的UML (shutdown now)，然后在主机配置一个 `TAP` 设备

```bash
ip tuntap add tap0 mode tap
ip addr add 10.0.0.1/24 dev tap0
ip link set tap0 up
```

这样就配置了一个可以作为网关使用的虚拟网卡设备，其网关地址是 `10.0.0.1`, 子系统可用的网段为 `10.0.0.0/24`。但是别急，为了让UML可以访问网络，我们还需要配置NAT

```bash
iptables -P FORWARD ACCEPT
iptables -t nat -A POSTROUTING -o venet0 -j MASQUERADE
```

其中 `venet0` 是你的因特网访问的网卡设备名。记得保存一下 `iptables` 规则。

这个时候就可以再次启动 `UML` 了。在上次启动的参数后面加一个

```bash
eth0=tuntap,tap0
```

启动后，`eth0` 就是可用的网络访问设备了。临时配置一下网络即可

```bash
ip link set eth0 up
ip addr add 10.0.0.2/24 dev eth0
ip route add default via 10.0.0.1 dev eth0
```
其中 `10.0.0.1` 是刚才在外面配置的 `tap0` 设备的地址。`10.0.0.2` 是分配给这个 `UML` 里面的系统的地址。

可以 `ping 8.8.8.8` 测试一下，应该已经可以访问外网了。

### 基本系统

现在我们可以在 `/dev/ubda` 即在外面创建的 `rootfs.img` 上配置基本的 `ArchLinux` 系统了。

首先当然是格式化 `/dev/ubda`。这以后，把它挂载到 `/mnt` 并灌入基本系统

```bash
…. (格式化)
mount /dev/ubda /mnt
mkdir -p /mnt/var/lib/pacman
pacman -Sy base -r /mnt
```

然后和刚才配置临时系统一样，把几个必须分区挂载进去，配置好 `pacman` 的镜像和DNS。记得按照 `ArchLinux` 的手册把 `Locale` 之类的也配置好。然后我们来配置 `systemd-networkd` 让它自动配置网络。

chroot进 `/mnt` 以后，编辑 `/etc/systemd/network/50-static.network`

```ini
[Match]
Name=eth0

[Network]
Address=10.0.0.2/24
Gateway=10.0.0.1
```

然后开启几个服务

```bash
systemctl enable systemd-networkd
systemctl enable getty@tty0
systemctl disable getty@tty1
```

### 后续

其实现在，一个能用的 `UML` 已经配置好了。我们可以这样启动它

```bash
./vmlinux ubd0=rootfs.img root=/dev/ubda eth0=tuntap,tap0 mem=Xm
```

其中 `mem=Xm` 指定的是你要给 `UML` 分配的内存数量。

为了让 `UML` 内部运行的网络服务能被外部直接访问，你需要在主机的 `iptables` 里面加上

```bash
iptables -t nat -A PREROUTING -p tcp --dport 2222 -j RETURN
iptables -t nat -A PREROUTING -i venet0 -j DNAT --to-destination 10.0.0.2
```

这会把除了到 `TCP 2222` 端口以外的所有数据包转发给 `UML` 内的系统。这个 `2222` 端口可以保留用来做主机的 `SSH` 端口。__一定要记得保留这么一个端口给SSH!!!__

然后你大概需要保存 `iptables`, 启用 `iptables` 开机服务，并编写开机服务让 `tap0` 设备自动创建、自动启动 `UML` 等等，我不再一一赘述。

之后我在这样配置出的 `UML` 里面运行了各种服务，性能都和在外面运行没太大区别。更重要的是在这个 `UML` 里面，你可以加载自己的内核模块，可以使用奇怪的文件系统，可以使用 `Tunnelbroker`，可以做任何在全虚拟化环境下能做的事情。而主系统中要做的，只是把数据包都NAT转发进去而已。

现在(2015.12.12), 本博客就运行在这个环境内。

不过有几个问题需要注意

### iptables / sit 等

使用 `iptables` 和 `sit` (tunnelbroker) 之类的需要额外内核模块的东西，往往会报错，这是因为模块没有编译进去。在内核配置里面打开以后重新编译一次 `UML` 就可以了

### Swap 导致 Udev 卡死

如果你在主机又建立了一个虚拟磁盘作为子系统的 `swap`, 你会发现，如果把它加入UML的 `fstab`, 那么UML将无法启动，卡死在等待udev的时候。这时候，你需要对UML内部的映射设备做个软链接，只要链接到 `/dev` 目录之外，然后把fstab指向这个软链接即可。

### UML 经常崩溃

一开始我遇到一个这样的问题，就是每次做大量的 `I/O` 操作时UML都会很快崩溃。后来我找到了原因。这原因就是运行 `UML` 的主机的 `/dev/shm` 小于给UML分配的内存空间

所以只需要扩大一下，比如说扩大到1G

```bash
mount -o remount,size=1G /dev/shm
```

就可以了。你大概也需要把这个加入开机启动服务。

### 80 / 443 等端口的NAT无效

我配置了 `nginx` 以后发现从外网无法访问而 `shadowsocks` 等服务可以，找了半天原因而不知道怎么回事。这也是最奇怪的一个情况，我的解决办法是在主机里面用 `haproxy` 做转发到 `10.0.0.2`。
