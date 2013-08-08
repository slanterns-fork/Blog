---
layout: default
title: 军哥LNMPA环境降级到LNMP
category: tech
summary: 之前发现LNMP安装包支持升级LNMPA，于是升级了试试看。但是升级完成发现，因为LNMPA的原理是用Nginx反代Apache，导致反代这一层的效率损失过大，因此反而没有用Nginx执行php快了。<br />但是查询军哥lnmp安装包官网得出，LNMPA不能直接降级回LNMP。<br />这下蛋疼了，这坑爹的LNMPA。<br />于是继续翻阅，找到军哥提供的一种解决办法，相当于卸载LNMPA再重新安装LNMP。
---
之前发现LNMP安装包支持升级LNMPA，于是升级了试试看。但是升级完成发现，因为LNMPA的原理是用Nginx反代Apache，导致反代这一层的效率损失过大，因此反而没有用Nginx执行php快了。<br />
但是查询军哥lnmp安装包官网得出，LNMPA不能直接降级回LNMP。<br />
这下蛋疼了，这坑爹的LNMPA。<br />
于是继续翻阅，找到军哥提供的一种解决办法，相当于卸载LNMPA再重新安装LNMP。<br />
1、备份MySQL数据库<br />
2、ssh登录服务器，运行<br />
{% highlight sh %}
/etc/init.d/nginx stop
/etc/init.d/mysql stop
/etc/init.d/php-fpm stop
/etc/init.d/httpd stop
rm -rf /usr/local/php
rm -rf /usr/local/nginx
rm -rf /usr/local/mysql
rm -rf /usr/local/apache
rm -rf /usr/local/zend
rm -f /etc/my.cnf
rm -f /root/vhost.sh
rm -f /root/lnmp
rm -f /root/run.sh
rm -f /etc/init.d/php-fpm
rm -f /etc/init.d/nginx
rm -f /etc/init.d/mysql
{% endhighlight %}
3、进入lnmp安装目录，重新执行./centos.sh（CentOS的运行centos.sh，其他系统运行对应的脚本）<br />
4、还原MySQL数据，重新建立虚拟主机，重新配置伪静态和权限。
