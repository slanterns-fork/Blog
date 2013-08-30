---
layout: default
title: 用反向代理加速GITHUB PAGES
category: tech
summary: GITHUB PAGES我之前有说过，这是GITHUB给程序猿提供的极好的免费建站平台，只要你会使用GIT，会使用Markdown标记模板，就可以免费建立博客。<br />而最近，由于GFW的“抽风”，GITHUB PAGES的访问十分不稳定，动不动就无法访问。没有办法，只得使用反向代理加速。<br />试过CloudFlare反代，虽然速度不错，可是CloudFlare本身也非常容易被墙。还剩下唯一一个选择，就是自己进行反代处理。
---
GITHUB PAGES我之前有说过，这是GITHUB给程序猿提供的极好的免费建站平台，只要你会使用GIT，会使用Markdown标记模板，就可以免费建立博客。

而最近，由于GFW的“抽风”，GITHUB PAGES的访问十分不稳定，动不动就无法访问。没有办法，只得使用反向代理加速。

试过CloudFlare反代，虽然速度不错，可是CloudFlare本身也非常容易被墙。还剩下唯一一个选择，就是自己进行反代处理。

在搜索引擎中看见一个很好的反代程序，叫做dynamic，是用PHP写的轻量级反向代理程序。这个程序非常好用，速度很快，但是问题在于它只能反向代理访问其他域名，不能反代自己本身的域名。也就是说，这东西是被设计用来做“小偷”站的。

为了让他改邪归正，我对他进行了小修改，使它反代的域名可以和自己本身的域名相同。这样，就可以用来反代GITHUB了。

修改过的源码包下载地址：<http://typeblog.net/code/php-dynamic.zip>

下载到该源码包以后，只需修改一行代码，即可实现反代GITHUB。

找到如下这行代码：

{% highlight php %}
$mirror = "your.github.pages.domain";
{% endhighlight %}

只需把这行代码中的 __your.github.pages.domain__ 替换成你自己的域名即可，前提是该域名必须已经绑定到GITHUB，具体绑定方法请查阅官方手册。
