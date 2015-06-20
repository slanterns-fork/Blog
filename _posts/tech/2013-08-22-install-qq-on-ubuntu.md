---
layout: default
title: 在Ubuntu下安装QQ
category: tech
---
作为一个中国人，安装Ubuntu后，离开了QQ真是各种蛋疼，幸好有Wine这神器。当然，不是所有QQ都能在Wine下运行的，本文章以QQ国际版为例，实际上可以运行QQ2012、QQ国际版、TM2013，但QQ2013不行……

首先，通过软件中心安装Wine，然后删除默认的 __~/.wine__ 目录，重新创建：
<!--more-->
{% highlight sh %}
export WINEARCH=win32
wineboot -u
{% endhighlight %}

以上代码可以使Wine创建一个 __Win32__ 环境，因为我们用于Android开发的一般是 __64__ 位的Ubuntu，所以需要指定创建的WIN环境为 __32__ 位

然后，安装所需的组件
{% highlight sh %}
winetricks riched20 msxml3 ie7 mfc42
{% endhighlight %}

稍等一会，组件就安装完成了，然后你就可以从 <http://im.qq.com> 下载最新的TM或者QQ国际版，右键，打开方式，选择WINE LOADER然后安装了。安装完成后，桌面上会出现快捷方式，点击运行，是不是非常流畅？

