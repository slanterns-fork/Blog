---
layout: gentoo
title: 选择正确的Gentoo安装媒介
---
### 2.a. 硬件要求

#### __简介__

在开始之前，我们先列出成功安装Gentoo的硬件需求。

#### __硬件要求__

<table class="table table-striped table-bordered">
  <thead>
    <tr>
      <th></th>
      <th>最小化启动CD</th>
      <th>启动DVD</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>CPU</th>
      <td colspan="2">任意AMD64或EM64T CPU(Core 2 Duo & Quad 处理器是EM64T的)</td>
    </tr>
    <tr>
      <th>内存</th>
      <td>256MB</td>
      <td>512MB</td>
    </tr>
    <tr>
      <th>硬盘空间</th>
      <td colspan="2">2.5GB(除交换空间)</td>
    </tr>
    <tr>
      <th>交换空间</th>
      <td colspan="2">至少256MB</td>
    </tr>
  </tbody>
</table>

在安装之前，您也应当浏览 [Gentoo AMD64 项目页面](http://www.gentoo.org/proj/en/base/amd64/)

### 2.b. Gentoo 安装光盘

#### __Gentoo 最小化安装光盘__

最小化安装光盘是一个包含自适应的Gentoo最小化环境的启动光盘。您可以通过它启动一个Linux环境。在启动过程中，它将自动检测硬件并加载合适的驱动。这个被Gentoo开发者所维护的光盘可以让您通过因特网来安装Gentoo。

这个光盘的映像文件名为 `install-amd64-minimal-<release>.iso` 并占用大约 `200MB` 空间。（译者注：[下载页面](http://www.gentoo.org/main/en/where.xml)，请选择amd64_multilib一项，如果您想要您的64位计算机支持部分32位程序的话）

#### __Gentoo Linux 启动DVD__

当然，Gentoo Ten 项目制作的特殊DVD也可以被用来安装Gentoo。本手册仅针对使用最小化安装光盘，所以使用这个DVD安装的过程和命令可能有些不同。但无论如何，启动DVD或任何其他可启动的Linux环境都支持使用 `sudo su` 或 `sudo -i` 来获取root权限。

#### __Stage 3 压缩包__

传说中的  `Stage 3 压缩包` 包含一个最小化的Gentoo Linux环境，而本手册接下来所介绍的安装方式都是针对 `Stage 3 压缩包` 的。曾经，曾经……Gentoo安装手册还包含过另外两种压缩包的安装方式。尽管Gentoo仍然提供另外两种(`stage 1` 和 `stage 2`)，但官方的安装方式是 `Stage 3 压缩包`。如果您对另外两种包包有兴趣（译者注：真蛋疼），请访问 [How do I Install Gentoo Using a Stage1 or Stage2 Tarball?](https://wiki.gentoo.org/wiki/FAQ#How_do_I_Install_Gentoo_Using_a_Stage1_or_Stage2_Tarball.3F)

本压缩包可以从 `releases/amd64/autobuilds/current-stage3/` 下载到（译者注：地址其实可以从上面 `最小化安装光盘` 的地址进去，直接就可以下载到启动CD和这个 `Stage 3` 压缩包。不过其实你无需在安装前下载，在安装过程中我们会再提到这个的）。也可以从 [官方Gentoo镜像](http://www.gentoo.org/main/en/mirrors.xml)下载。请注意，压缩包并不包含在可启动的DVD和CD中。

### 2.c. 下载，烧录并从Gentoo安装光盘启动

### __下载并烧录安装光盘__

您已经选择要用安装光盘来安装Gentoo。我们之前也提到过，不过，咋下载咧？

您可以从Gentoo的 [镜像](http://www.gentoo.org/main/en/mirrors.xml) 下载。（译者注：也可以直接从我上一节提供的那个地址下载）安装CD镜像在 `releases/amd64/autobuilds/current-iso/` 目录里面。

在那个目录里面您可以找到一个 `iso` 文件。这就是一个可烧录的完整CD镜像，您可以把它写入一个 `CD-R` 光盘。

如果您要确定您的下载是完整且正确的，您可以检验它的 `SHA-2` 校验码并与我们提供的对比（通常文件名是 `install-amd64-minimal-<release>.iso.DIGESTS`）。您可以在 `Linux/Unix` 下使用 `sha512sum` 工具或在 `Windows` 下使用 [Checksums calculator](http://www.sinf.gr/en/hashcalc.html).

> 注意：工具会尝试检查列表中的校验码，即使校验码是由其他算法提供的。所以，命令行输出可能有成功也有失败。但至少每个文件得有一个成功的校验。

##### __代码列表 3.1: 校验SHA-2__

{% highlight sh %}
$ sha512sum -c <downloaded iso.DIGESTS>
{% endhighlight %}

> 注意：如果提示找不到合适的校验码，请查看DIGESTS文件中提供了哪些校验码。

另一个方法是验证 `GnuPG` 签名（文件名后缀是 `.asc`）。下载这个文件和 [工程项目发布网站](http://www.gentoo.org/proj/en/releng/index.xml) 上提供的公匙。

##### __代码列表 3.2: 生成公匙__

{% highlight sh %}
(... 替换公匙为工程项目发布网站上的 ...)
$ gpg --keyserver subkeys.pgp.net --recv-keys 96D8BF6D 2D182910 17072058
{% endhighlight %}

然后验证：

##### __代码列表 3.3: 验证签名__

{% highlight sh %}
$ gpg --verify <downloaded iso.DIGESTS.asc>
$ sha512sum -c <downloaded iso.DIGESTS.asc>
{% endhighlight %}

要刻录镜像，您需要选择使用 `raw` 方式刻录。这种选择是依赖于不同软件的，在这里提供 `cdrecord` 和 `K3B` 两种软件的刻录方法（译者注：Windows下的NERO什么的你们都懂的啦）

* 用 `cdrecord`，只需输入 `cdrecord dev=/dev/sr0 <downloaded iso file>` (把 `/dev/sr0` 换成你的刻录机的设备路径，把尖括号里面的内容换成下载的iso文件路径）
* 用 `K3B`，选择 `Tools > Burn CD Image`，然后在 `Image to Burn` 中选择ISO文件，然后开始即可。

#### __启动安装光盘__

刻录好以后，当然是时候启动它了。从驱动器中取出所有CD，重启到BIOS，然后设置光盘启动。这一步可以在您的主板说明书或者搜索引擎中找到，译者在这里不想过多赘述了。

然后把安装盘放进驱动器后启动计算机，您会看见一个启动提示。在这里，你可以直接敲回车，用默认启动选项启动，或者输入自己的启动选项来启动。

当启动提示显示的时候，您可以获得可用的内核(`F1`)和可用的启动选项(`F2`)。如果15秒内您没有做出决定，那么安装盘会自动从硬盘启动。这将重启并寻找其他适合的环境。

现在我们提到了选择内核。在安装光盘中，我们提供多种内核。默认的是 `gentoo`，其他内核是给某些特殊的需要使用 `-nofb` 选项来禁用 `framebuffer` 的硬件。（译者注：大部分情况下你不需要自己选择内核和启动选项，只需要敲回车即可）

下面是一个简短的可用内核的介绍：

<table class="table table-striped table-bordered">
  <thead>
    <tr>
      <th>内核</th>
      <th>介绍</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>gentoo</td>
      <td>默认的内核，支持K8 CPU（包括NUMA）和EM64T CPU</td>
    </tr>
    <tr>
      <td>gentoo-nofb</td>
      <td>和gentoo相同，但没有framebuffer支持</td>
    </tr>
    <tr>
      <td>memtest86</td>
      <td>测试本地内存有没有错误</td>
    </tr>
  </tbody>
</table>

您也可以输入内核选项，它们是您可以根据自己的需要打开或关闭的功能。

#### __硬件选项：__

##### __acpi=on__

> 这将加载ACPI支持，并在CD启动时启动acpid守护进程。仅在您的硬件需要ACPI才能正常运行的情况下开启本选项。超线程技术不需要这个选项。

##### __acpi=off__

> 完全关闭ACPI支持。在一些老的系统上，这个选项是有用的，但这个选项也需要启用APM。这将禁用处理器超线程支持。

##### __console=X__

> 设置CD启动环境的命令行串口输出(serial console)。第一个选项(X)就是指串口设备，在x86上多数是ttyS0。在它之后请跟上链接选项，以英文逗号分割。默认是9600,8,n,1

##### __dmraid=X__

> 这将允许传递参数到软RAID系统(device-mapper RAID)。选项应该用英文引号包裹。

##### __doapm__

> 打开APM支持。如果你使用了acpi=off，那么请使用本选项。

##### __dopcmcia__

> 打开PCMCIA和Cardbus硬件支持，并在CD启动时自动启动pcmcia cardmgr。这只在硬件是PCMCIA/Cardbus时需要。

##### __doscsi__

> 这可以打开对大部分SCSI控制器的支持。从大部分USB设备启动都需要这个选项，因为它们使用SCSI系统。

##### __sda=stroke__

> 这将允许你对您的大硬盘分区，即使连BIOS也无法处理那么大的硬盘。这将在部分老机器上起作用。把sda换成你的那块大硬盘的设备名。

##### __ide=nodma__

> 在内核里禁用DMA选项。这在某些IDE芯片和CDROM设备上是必须的。如果系统无法读取CDROM，请试试这个选项。这也会禁用hdparm命令。

##### __noapic__

> 这将禁用某些新主板上的高级可编程中断控制器。目前已知它将在老硬件上导致一些问题。

##### __nodetect__

> 这将禁用CD的自动硬件检测。当启动失败时，本选项可用于调试。

##### __nodhcp__

> 这会禁用DHCP（自动分配网络IP/DNS等地址）。对于静态网络用户来说，这非常有用。

##### __nodmraid__

> 关闭软RAID阵列支持。(device-mapper RAID)

##### __nofirewire__

> 关闭对火线(Firewire)模块的支持。请只在您的火线设备启动错误的情况下使用。

##### __nogpm__

> 这将关闭gpm驱动和命令行下的鼠标支持。

##### __nohotplug__

> 这将关闭对热插拔和冷插拔设备的加载脚本。这在调试时很有用。

##### __nokeymap__

> 关闭启动时选择非美式键盘布局的步骤。

##### __nolapic__

> 在单核上关闭可编程中断控制器。

##### __nosata__

> 关闭SATA支持。如果SATA出错可以试试。

##### __nosmp__

> 关闭对称多处理支持。在测试某些主板和设备的多处理时本选项有用。

##### __nosound__

> 关闭声音支持和音量选项。如果你的声音出问题了，就试试这个吧。

##### __nousb__

> 关闭USB支持。用于调试USB设备。

##### __slowusb__

> 在启动过程中加入更多等待时间来支持缓慢的USB CD驱动器。

#### __卷管理选项：__

##### __dolvm__

> 打开Linux的逻辑卷管理机制(LVM)

#### __其他选项：__

##### __debug__

> 打开调试信息输出，不过可能会一团糟，因为它会输出一大坨数据。

##### __docache__

> 把整个运行环境缓存到内存里，这需要两倍于CD空间大小的内存。

##### __doload=X__

> 启动时读取指定的内核模块或一组内核模块及其依赖。把X换成一个内核模块或者用英文逗号分割的一组内核模块。

##### __dosshd__

> 启动sshd守护进程来支持远程登陆。

##### __passwd=foo__

> 打开sshd后请务必用此选项设置root密码

##### __noload=X__

> 与doload相反，这是让内核不要加载某些模块。

##### __nonfs__

> 禁用 portmap/nfsmount 的自动启动

##### __nox__

> 在带有X桌面环境的CD上禁用X环境，采用命令行。

##### __scandelay__

> 让CD启动进程中加载速度慢的硬件时暂停10秒再继续。

##### __scandelay=X__

> 同上，但把等待的10秒换成X秒。X是你需要等待的秒数。

-------------------------------------------------------

> 注意：CD会先检查“no”开头的选项，再检查“do”选项，所以你可以以这个顺序来覆盖你先前写入的内容。

未完待续……………………
