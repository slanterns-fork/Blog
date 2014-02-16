---
layout: default
title: ArchLinux 安装过程小记
category: tech
---
上个星期一，由于突然断电，我的可爱的Ubuntu惨死于升级过程之中。后来，由于希望寻找一个轻量级的系统以便维护、备份和随时重装，我选择了ArchLinux。本文其实主要是对官方英文文档的翻译，融入了我自己的安装经验，希望可以帮到大家，也希望可以借此将K.I.S.S.理念宣传给更多人。

准备
=====
1. 一台电脑  
2. CD光盘/1G U盘  
3. 光驱/USB端口  
4. 至少2Mbps的网络连接  
5. 一个正常的硬盘  
6. 非UEFI模式(因为我也没试过UEFI)

安装
=====
前往<http://archlinux.org>下载最新版本的Live CD，即安装镜像，然后刻录至盘片中，一张CD足以刻录下双版本ArchLinux的安装镜像。

然后，用该镜像引导电脑启动。会出现启动菜单，如果你要安装64位，则选择x86_64，否则选择x86，回车启动进入Live CD环境。

不要看傻眼了，这就是一个命令行，我们将通过它安装整个系统。

第一步，用`parted`命令进行分区。该工具的使用方法在百度上比比皆是，在这里不再赘述，当然你也可以通过其他你喜欢的工具进行分区，只是别忘了将安装分区设为活动。建议分区:

> /boot 400-500M(活动分区)  
> /     1-20G (取决于你的软件包安装多少)  
> /home 余下全部

接着应该对其进行格式化。先执行`lsblk`查看分区好的列表，一般硬盘分区是/dev/sdax(x是数字)。如果你不知道哪个是哪个，可以通过后面显示的大小判断。此时，对home、boot和根分区对应的分区执行`mkfs.ext4 /dev/sdax` (sdax换成你对应的分区名称)，如果有swap分区，则使用`mkswap`命令，并用`swapon`命令开启swap。

格式化好后，先将分出来的根分区用`mount`命令挂载到/mnt，再用`mkdir`命令创建/mnt/home与/mnt/boot，并挂载。`mount`命令使用格式：`mount /dev/sdax /mnt/xxx`

然后，配置网络连接。这里以无线网络为例，因为现在很多人应该都使用无线网。

在命令行中输入`iw dev`，你应该看见类似这样的输出

> Interface xxx  
>     ifindex x  
>     wdev xxx  
>     ........

现在假设那个`Interface`后面的字符串为`xxx`。在命令行中键入

{% highlight sh %}
ip link set xxx up  
wifi-menu
{% endhighlight %}

然后再在显示的WiFi菜单中选择一个。选择后会提示你创建配置文件，这时候随便写一个就行。如果看不见WiFi Interface或看不见扫描结果，那么你就需要自己去折腾驱动了。

连上WiFi以后，我们可以开始安装了。只需执行

{% highlight sh %}
pacstrap -i /mnt base
{% endhighlight %}

然后一路回车，耐心等待命令完成。其实这里可以通过添加国内源来加快速度，但由于我是爪机党，所以查看不了那天用的源。命令完成后，基本系统就安装完成了，下面我们进行基本配置。

配置
=====
首先，生成该系统使用的分区表

{% highlight sh %}
genfstab -U -p /mnt >> /mnt/etc/fstab
{% endhighlight %}

接着就可以使用`arch-chroot`进入该系统了

{% highlight sh %}
arch-chroot /mnt /bin/bash
{% endhighlight %}

现在你已经在该系统中。我们需要先设置区域

{% highlight sh %}
nano /etc/locale.gen
{% endhighlight %}

由于命令行界面不支持中文显示，所以我们先将语言设为英文。找到`en_US.UTF-8`一行，去除前面的井号，然后按`Ctrl+O`并回车保存，按`Ctrl+X`退出。

接着直接执行`locale-gen`命令即可。

然后就可以设置时间了

{% highlight sh %}
ln -s /usr/share/zoneinfo/Asia/Shanghai /etc/localtime  
hwclock --systohc --utc
{% endhighlight %}

该给你的电脑起一个名字了。假设你起的名字是aaa

{% highlight sh %}
echo aaa >> /etc/hostname
{% endhighlight %}

按照官方提供的步骤，接下来该重新配置网络了，但因为系统自带的`netctl`在我的电脑上并不能实现开机自动联网，所以这里我先跳过了。之后在GNOME环境配置的时候我会在教程中教您安装`NetworkManager`来管理网络。

不过，这时候必须安装一些包

{% highlight sh %}
pacman -S iw wpa_supplicant dialog
{% endhighlight %}

然后，可以设置根密码了，使用`passwd`命令即可。

下面安装`syslinux`引导程序

{% highlight sh %}
pacman -S syslinux
syslinux-install_update -i -a -m
{% endhighlight %}

然后，直接使用`reboot`命令重启，并弹出光盘。不出意外，你就可以进入archlinux的基本命令行界面了。

后记
=====
本文仅适用于单系统ArchLinux的安装，多系统的安装请参看官方文档。

本文仅仅是记录了我的安装过程和体会，您的电脑和网络环境可能并不完全适应于本文，请特别注意网络配置部分，如果有必要，参考官网。

本文仅仅安装了ArchLinux的基本系统，有关桌面环境配置和Android开发环境搭建，会在稍后补上，谢谢。


