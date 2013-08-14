---
layout: default
title: 给Android wpa_supplicant 添加Ad-Hoc支持
category: tech
summary: 不少人喜欢使用电脑建立虚拟热点给手机分享网络连接，这种热点叫做 <strong>Ad-Hoc</strong> 。但在Android中，该热点形式并不被支持，不能扫描到任何 <strong>Ad-Hoc</strong> 热点<br />一些厂商和ROM开发者使用修改内核或者修改frameworks的方法来支持 <strong>Ad-Hoc</strong> ，但是，内核源码不是所有厂商都开放的，frameworks部分代码修改又太过麻烦，而 <strong>wpa_supplicant</strong> 这种修改方法，又在Android 4.0之后再也无人维护，之前的人又不开源……
---
不少人喜欢使用电脑建立虚拟热点给手机分享网络连接，这种热点叫做 __Ad-Hoc__ 。但在Android中，该热点形式并不被支持，不能扫描到任何 __Ad-Hoc__ 热点

一些厂商和ROM开发者使用修改内核或者修改frameworks的方法来支持 __Ad-Hoc__ ，但是，内核源码不是所有厂商都开放的，frameworks部分代码修改又太过麻烦，而 __wpa\_supplicant__ 这种修改方法，又在Android 4.0之后再也无人维护，之前的人又不开源……

经过我在Google上的一番搜索，我找到了一个在Android 2.3( __wpa\_supplicant\_5__ ) 时代可用的patch，但用在目前的 __wpa\_supplicant\_8__ 上已经无效，无法成功patch，不过至少让我知道了修改支持Ad-Hoc的原理。

首先我们进入wpa\_supplicant_8目录中的wpa\_supplicant也就是源码目录
打开 __ctrl\_interface.c__ , 你会找到类似下面这段代码

{% highlight c %}
	if (bss->caps & IEEE80211_CAP_IBSS) {
		ret = os_snprintf(pos, end - pos, "[IBSS]");
		if (ret < 0 || ret >= end - pos)
			return -1;
		pos += ret;
	}
{% endhighlight %}

这类似的代码有很多段，是为了在Ad-Hoc热点后添加 \[IBSS\] 标记，让上层在扫描的时候忽略这个。  
所以，只需要删除类似的代码，就可以去除Ad-Hoc的特殊标记，不让上层忽略。

而仅仅删除这些还不够，上层还是无法获取到Ad-Hoc的热点，原因在于， __events.c__ 中也有对Ad-Hoc进行过滤的代码：

{% highlight c %}
 		if (!bss_is_ess(bss)) {
 			wpa_dbg(wpa_s, MSG_DEBUG, "   skip - not ESS network");
 			continue;
 		}
{% endhighlight %}

这段代码的作用是忽略掉IBSS，也就是Ad-Hoc热点。将这段代码注释掉，手机可以成功扫描到Ad-Hoc热点。但是，不能连接。这是因为， __wpa\_supplicant__ 有一个特殊的配置，用于连接Ad-Hoc，他就是

{% highlight c %}
wpa_config_set(ssid, "mode", "1", 0)
{% endhighlight %}

要使用此代码来设置Ad-Hoc连接，你首先要为Ad-Hoc类型的热点添加标识，我们之前去除了系统用于忽略Ad-Hoc的标志，因此必须自己重新加一个标志，这里我们使用的是以 __\(\*\)__ 开头来标记Ad-Hoc热点。  
重新切换回 __ctrl\_interface.c__  
首先，定义一些常量

{% highlight c %}
#define ANDROID_IBSS_PREFIX "(*)"
#define ANDROID_IBSS_PREFIX_LEN 3
{% endhighlight %}

然后，在函数 __wpa\_supplicant\_ctrl\_iface\_status__ 中，查找 __ret = os\_snprintf__ ，将其替换为：

{% highlight c %}
	if (bss->caps & IEEE80211_CAP_IBSS) {
		ret = os_snprintf(pos, end - pos, "\t%s%s",
				  ANDROID_IBSS_PREFIX, wpa_ssid_txt(bss->ssid, bss->ssid_len));
	} else {
		ret = os_snprintf(pos, end - pos, "\t%s",
				  wpa_ssid_txt(bss->ssid, bss->ssid_len));
	}
{% endhighlight %}

在函数 __wpa\_supplicant\_ctrl\_iface\_get\_network__ 中，查找 __res = os\_strlcpy__ ，将其替换为：

{% highlight c %}
	if ((os_strcmp(name, "ssid") == 0) && (ssid->mode == IEEE80211_MODE_IBSS)) {
		res = os_snprintf(buf, buflen, "\"%s%s", ANDROID_IBSS_PREFIX, value+1);
	} else {
		res = os_strlcpy(buf, value, buflen);
	}
{% endhighlight %}

在函数 __print\_bss\_info__ 中，查找 __if (mask & WPA\_BSS\_MASK\_SSID) {__ ，将其替换为：

{% highlight c %}
	// ADD IBSS PREFIX
	if (bss->caps & IEEE80211_CAP_IBSS) {
		ret = os_snprintf(pos, end - pos, "ssid=%s%s\n",
				  ANDROID_IBSS_PREFIX, wpa_ssid_txt(bss->ssid, bss->ssid_len));
		if (ret < 0 || ret >= end - pos)
			return 0;
		pos += ret;
	} else if (mask & WPA_BSS_MASK_SSID) {
{% endhighlight %}

以上三段代码的作用就是，在类型为IBSS，即Ad-Hoc的热点名称之前，加上 __\(\*\)__ 标记，用于区分。  
光添加标记没有用，我们要对标记进行处理，在函数 __wpa\_supplicant\_ctrl\_iface\_set\_network__ 中， __if \(wpa\_config\_set(ssid, name, value, 0\) < 0) {__ 这句代码之前，插入：

{% highlight c %}
	if (os_strcmp(name, "ssid") == 0) {
		// check prefix
		if ((value[0] == '"') && (os_strncmp(value+1, ANDROID_IBSS_PREFIX,
			  ANDROID_IBSS_PREFIX_LEN) == 0)) {
			if (wpa_config_set(ssid, "mode", "1", 0) < 0) {
				wpa_printf(MSG_DEBUG, "CTRL_IFACE: failed to set IBSS on '%s'",
					  value);
				return -1;
			}
			value += ANDROID_IBSS_PREFIX_LEN;
			value[0] = '"';
		}
	}
{% endhighlight %}

上边这段代码的作用是，按照热点前缀判断，如果是Ad-Hoc热点，就将 __wpa\_supplicant__ 配置为 __Ad-Hoc__ 模式进行连接。

至此，全部修改结束，保存，编译，是不是已经可以了？注意，Ad-Hoc默认是不支持WPA加密的，所以不要和我说，WPA不能用……

以上全部代码修改，我已经把它们做成一个patch文件，下载地址： <http://typeblog.net/code/wpa_supplicant-adhoc.patch>  
大家下载这个patch之后，只需将这个patch保存在在 __wpa\_supplicant\_8__ 源码根目录下，然后执行

{% highlight sh %}
patch -p1 -N -i wpa_supplicant-adhoc.patch -r - -d ./
{% endhighlight %}

即可将该patch应用到你的源码中，添加支持Ad-Hoc

该功能将出现在下个版本LOSP中。
