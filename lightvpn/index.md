---
layout: default
title: LightVPN
---

`VPN` 这个东西嘛……咳咳，用来做什么你们都懂的。

不过呢，作为一个Android党，你一定遇到过这事情：Android自带的VPN客户端必须要设置坑爹的锁屏密码才能使用，而你自己又完全不想使用麻烦的锁屏密码！

所以，`LightVPN` 就此诞生了。

__软件简介__  
LightVPN是Android上的一款第三方 `PPTP VPN` 客户端，目的是解除Android上连接VPN必须设置锁屏密码的烦恼。它调用系统自带的 `mtpd` 程序来建立VPN链接，因此程序本身需要的权限非常少，仅有网络权限和ROOT权限要求。因为调用系统自带的链接程序，所以 `LightVPN` 本身并不会在后台保持运行，您完全不必担心耗电等问题。

__FAQ__  
Q: 支持什么协议？  
A: 仅支持PPTP协议  

Q: 连不上/连上了不能上网怎么办?  
A: 尝试勾选 “Mppe 加密”一项。如果仍然不行，请在此提交BUG报告。

__说明__  
本软件需要ROM自带完整的busybox支持，如果没有请用busybox安装器。  
由于本屌丝暂时手头只有一部手机，所以很多问题都测试不到，如果不能用，请不要怪我……我找到更多测试机以后会一个个解决的。

__更新历史__  
v1.0: 第一个版本

__源代码__  
<https://github.com/PeterCxy/LightVPN>

__下载地址__  
v1.0: <http://www.androidfilehost.com/?fid=23501681358562081>  
下载说明:  
1. 点击下载链接进入下载页面  
2. 点击"Begin Download"  
3. 等10秒  
4. 选择"primary download"或其他mirror即可开始下载
