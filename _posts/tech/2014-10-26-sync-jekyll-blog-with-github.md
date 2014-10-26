---
layout: default
title: 将服务端的Jekyll博客与GitHub Repo同步
category: tech
---

我的这个博客，以前使用的是Github Pages，后来用比尔盖子的VPS作服务端，而博客的版本库仍然存储在GitHub。当时我是写了个 `shell` 脚本定期循环 `git pull` 来实现与GitHub版本库同步，比较暴力。现在我换用自己的DigitalOcean VPS，认为以前的方法不够优雅(特别是定时循环有些浪费资源)，所以我换用另一种方法。

<!--more-->

那么，使用什么方法呢?

我注意到GitHub有个 `Web Hook` 功能，可以在发生某些git事件的时候调用一个远程回调地址。这正是我需要的。

现在思路就来了：我们只需要让GitHub在远程版本库更新的时候向服务器发送请求，服务器接到请求以后就执行 `git pull` 和 `jekyll build`, 不就可以了？而且这样更新才是真正的 `即时更新` !

那么我们只需要实现一个服务端回调就可以了。由于我以前Python用的少，所以我决定用Python练练手。只需安装 `python python-pip spawn-fcgi` 几个软件包并使用 `pip install web.py` 和 `pip install flup` 即可。

需要先实现一个Python脚本。可以参考 <http://webpy.org/cookbook/fastcgi-nginx> 配置好nginx并按照例子写好hello world。以下内容均以此为基础修改。假设你在Nginx绑定的域名为`your.com`

接下来需要加入对GitHub事件的响应。有几个关键的地方

1. 引入 `subprocess`
2. 需要在 `urls` 变量中加入你要设定的hook地址。地址的名称可以随便写，但等会要把它填入GitHub使其生效。假设hook地址名就叫hook，那么urls变量应该是  
{% highlight python %}
urls = (
    "/hook", "github-push",
    "/(.*)", "hello"
)
{% endhighlight %}  
如果有多个网站请添加多条。`github-push` 定义的是收到数据时要调用的类名，必须与下面新建的类保持一致。不同的网站的hook请使用不同的类名。
3. 为了调用 `git` 和 `jekyll`, 我写了个shell脚本，传入要更新的网站名称即可自动执行更新。在python中，应该这样调用  
{% highlight python %}
def runscript(name):
    subprocess.call(['/path/to/your/shell/script', name])
{% endhighlight %}  
这是一个函数，它的作用是启动shell脚本并传入网站名称这个参数(如果你只有一个网站就不需要了)。等会我们将在处理过程中调用。  
shell脚本的实现很简单，我就不赘述了 
4. 建立一个新的class  
{% highlight python %}
class github-hook:
    def POST(self):
        runscript("name-of-your-website")
        web.ctx.status = "200 OK"
        return "OK"
{% endhighlight %}  
这里是处理请求并执行更新脚本。类名请与 `urls` 里面的定义一致。多个网站可以添加多个类。这里只处理了POST，因为GitHub hook是post形式。

现在你已经完成了hook，将它添加到GitHub Repo的 `Webhook` 里面(按照本文的设定，填入的地址应该是`http://your.com/hook`)，然后就可以调试啦。

根据我的经验，还是够折腾的。(对于我这种Python新手而言)。你可能会遇到shell脚本提示找不到jekykll，此时请使用完整路径。

记得把 `spawn-fcgi` 加入开机自启。
