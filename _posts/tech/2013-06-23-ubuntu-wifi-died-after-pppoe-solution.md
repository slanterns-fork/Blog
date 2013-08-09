---
layout: default
title: Ubuntu下通过WIFI连接PPPOE导致wifi不可用的解决办法
category: tech
summary: 电信的ChinaNet开头的ADSL热点，很蛋疼地需要使用PPPOE拨号，于是在Ubuntu下连接wifi后，用pppoeconf命令进行拨号。<br />刚开始，一切正常，但过了一段时间以后，突然所有网站都上不去，提示DNS错误，一开始以为是DNS问题，于是修改DNS以后，依然不可用。重启以后，WIFI提示“设备未托管”。<br />无奈之下用手机查阅资料，终于找到了原因。
---
电信的ChinaNet开头的ADSL热点，很蛋疼地需要使用PPPOE拨号，于是在Ubuntu下连接wifi后，用pppoeconf命令进行拨号。  
刚开始，一切正常，但过了一段时间以后，突然所有网站都上不去，提示DNS错误，一开始以为是DNS问题，于是修改DNS以后，依然不可用。重启以后，WIFI提示“设备未托管”。  
无奈之下用手机查阅资料，终于找到了原因。  
Ubuntu下有两套网络管理器，一套是NetworkManager，就是图形界面的管理器，也就是用来连接WIFI的那个；另一套是Linux标准命令行管理器。默认情况下，Linux标准命令行管理器是禁用的，但如果使用了pppoeconf命令配置pppoe，就相当于启用了这个Linux标准命令行管理器，这就导致NetworkManager被禁用，于是就出现了“设备未托管”的错误。  
那么这样就容易解决了。  
终端输入代码  
{% highlight sh %}
sudo gedit /etc/NetworkManager/NetworkManager.conf
{% endhighlight %}
在弹出的窗口中，把 managed=false改为managed=true，保存，重启，再设置pppoe，ok。
