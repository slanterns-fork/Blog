---
layout: default
title: 在Android手机上编译Android App
category: tech
---
这篇文章介绍的是在Android手机上编译Android App的方法。这不是推广文章，不需要借助某收费的运行在Android上的IDE来完成。

作为一个学生党，开学以后碰电脑的机会就很少了，但是作为不开发会死星人，必须在手机上配置一个Android App编译环境了。注意，这是编译环境。至于开发，我推荐使用 `VimTouch` 加上我之前发布的 `javacomplete` 插件的 `dalvik` 分支。

一开始我是想借助 `Terminal IDE` 这个免费程序提供的软件包里面的命令来编译，但是我很快发现了严重的问题：它提供的 `javac` 命令无法正确解析引入的第三方jar包。我猜想是 `Dalvik` 虚拟机导致的这个问题，因为它根本不支持加载 `.class` 字节码。

失败了？不，我最终还是找到了可以成功编译的方法。

<!--more-->

准备放弃的时候，无意间打开了 `Apktool` 的手机版数据目录，在里面我意外地发现了 `java-7-openjdk-armel` ，天哪，OpenJDK！有了它，还怕不能编译吗？我立即尝试用它运行了一下 `dx.jar` ，成功获得了正确的输出！

所以，我就把这个 `java-7-openjdk-armel` 给提取出来，自己搭建了一个可以成功编译Android App的环境，并且打包上传与大家分享。

#### 准备

* 为了简化教程，我们默认把开发环境放在 `/sdcard/env` ，工作目录为 `/sdcard/work`
* 创建 `/sdcard/env` 和 `/sdcard/work` 两个目录
* 下载我打包好的环境: [env.zip](http://www.androidfilehost.com/?fid=23578570567720305)
* 把下载好的压缩包内容全部解压到 `/sdcard/env`
* 把你的 `android.jar` 放到 `/sdcard/work` 里面。
* 安装 [Android Terminal Emulator](https://play.google.com/store/apps/details?id=jackpal.androidterm) 或其他终端模拟器。
* 确保你的系统自带 `busybox` 以及ROOT权限

注意：俗话说，不会翻墙的程序猿不是一个好程序猿，所以不要和我说上面的下载地址进不去。

#### 环境介绍

我打包好的环境包含:

* 适用于ARM平台的完整OpenJDK7
* git
* dx
* dropbear ssh
* 几种aapt

#### 基础环境配置

首先，打开 `Android Terminal Emulator` ，打开菜单，选择 `Preferences` ，勾选 `Default UTF-8` ，然后拉到下面，点击 `Command Line` 一项，并填入

{% highlight sh %}
/system/xbin/su -c "/system/xbin/busybox sh"
{% endhighlight %}

然后点击OK保存。再点击 `Initial Command` 一项，填入

{% highlight sh %}
cd /sdcard && source env/.bashrc
{% endhighlight %}

再点击OK，然后退出终端，重新进入。此时系统会询问你是否授权终端获取ROOT权限，选择允许即可。

#### 配置aapt

在终端里键入 `aapt` ，如果输出了很长一段的帮助信息，那您可以跳过本步骤。否则，你需要继续看本步骤。一般来说，如果你的系统包含了 `CM新主题引擎`，那你一定可以跳过本步骤。

用文件管理器之类的软件进入 `/sdcard/env/lix` 目录，你会看到好几个以 `aapt` 开头的可执行文件。选择适合你的一个（我推荐 `aapt4.4`），并把它复制到 `/sdcard/env/tools` 目录里面，改名为 `aapt`

#### 配置git ssh

因为这个环境自带的 `git` 命令不支持 `http` 和 `https` 协议，所以你必须手动生成 `ssh` 证书。

进入终端模拟器，输入下列命令

{% highlight sh %}
mkdir ~/.ssh
dropbearkey -t rsa -f ~/.ssh/id_rsa
dropbearkey -y -f ~/.ssh/id_rsa | grep "^ssh-rsa" >> my_key
{% endhighlight %}

现在你的 `/sdcard` 目录下已经有了一个 `my_key`，这是一个文本文件，你可以用文本编辑器打开，或者传输到电脑上用文本编辑器打开。打开以后，复制里面的内容，粘贴到 `GitHub -> Edit Profile -> SSH Keys -> Add SSH Key` 里面并随便取一个名字保存，这样 `GitHub` 就认识你的设备了。现在你可以尝试在终端里键入命令

{% highlight sh %}
ssh git@github.com
{% endhighlight %}

如果出现提示 `Hi, xxx! You've successfully authenticated, but GitHub does not provide shell access` ，那么就说明你已经成功了。

现在你已经可以使用 `git` 命令来克隆项目了，我们先举个例子，假如你要克隆 <https://github.com/PeterCxy/BlackLight> 项目到你的工作目录，那么请在终端里运行：

{% highlight sh %}
cd work
git clone git@github.com:PeterCxy/BlackLight.git
{% endhighlight %}

这样即可克隆成功。注意，URL格式一定要使用 `git@github.com:Username/Repository`，否则将失败。

接下来你还可以配置 `Git` 的其他参数，比如说用户名、邮箱、Mergetool、编辑器等，我就不一一赘述了，和电脑上的配置是差不多的。

#### 编译APP

现在你已经把你的App项目克隆到工作目录了，并且你已经切换到App项目目录，那么如何编译呢？

我推荐你自己编写一个 `shell` 脚本来编译，下面是简单的编译过程：

* 用 `aapt` 命令编译你的App和扩展库的资源并生成 `R.java`。如果你有扩展库资源（比如说 `SwipeBackLayout`，记得加上参数 `--extra-package=xxx.xxx.xxx` ，把 `xxx.xxx.xxx` 换成扩展库的包名，如果有多个扩展库，用冒号分割它们的名称
* 生成 `BuildConfig.java` ，这是一个类，里面只有一个布尔型常量 `DEBUG` ，值为true代表是调试版本，值为false代表是发布版本。
* 用 `javac` 命令编译你的源码并用 `-d` 参数输出到一个目录里面。注意 `javac` 不能直接在命令行输入多个源码文件，所以你需要使用shell脚本遍历源码目录并把源码文件输出到一个文本文件，用空格或换行符分割每个源码文件。别忘了把前两步生成的 `R.java` 和 `BuildConfig.java` 加入。哦对了，如果你的源码里包含中文，请加上参数 `-encoding utf-8`
* 用 `dx --dex` 命令把 `javac` 编译出来的字节码目录转换为 `class.dex`
* 用 `aapt` 把 `classes.dex` 添加到前面编译资源生成的apk里面
* 用 `jarsigner` 对编译后的apk签名。

以上提及的编译工具在Google上都有详细说明。用 `shell` 脚本可以很方便地实现自动化编译，下面是我的 `BlackLight` 项目的编译脚本 `build.sh`，供大家参考:

{% highlight sh %}
#!/bin/bash
 
function scandir() {
	local cur_dir parent_dir workdir
	workdir=$1
	cd ${workdir}
	if [ ${workdir} = "/" ]
	then
		cur_dir=""
	else
		cur_dir=$(pwd)
	fi
 
	for dirlist in $(ls ${cur_dir})
	do
		if test -d ${dirlist};then
			cd ${dirlist}
			scandir ${cur_dir}/${dirlist}
			cd ..
		else
			echo ${cur_dir}/${dirlist}
			echo "\n"
		fi
	done
}

function usage() {
	echo "usage: sh build.sh [debug|release]"
}

# Verification
if [ $# != "1" ]; then
	usage
	exit 1
fi

# Clean up
echo 'Cleanning up...'
rm -rf build

# Init
echo 'Initializing build environment...'
mkdir build
mkdir build/gen
mkdir build/bin
mkdir build/bin/classes

# Configurations
res="-S res -S libs/SlidingUpPanel/res" # Resources
ext_pkg='com.sothree.slidinguppanel.library' # Package that needs resources
src='src libs/SlidingUpPanel/src libs/SystemBarTint/src build/gen' # Sources
jar="$ANDROID_JAR:libs/android-support-v4.jar:libs/gson-2.2.2.jar:libs/SlidingUpPanel/libs/nineoldandroids-2.4.0.jar" # JARs
manifest='AndroidManifest.xml' # Manifest
assets='assets' # Assets
pkgs='us.shandian.blacklight' # Packages that needs to generate BuildConfig

# Run aapt
echo 'Compiling resources...'
aapt p -m -M $manifest -A $assets -I $ANDROID_JAR $res --extra-packages $ext_pkg --auto-add-overlay -J build/gen -F build/bin/build.apk

# Generate BuildConfig
echo 'Generating BuildConfig...'
if [ ${1} = "debug" ]; then
	flag="true"
elif [ ${1} = "release" ]; then
	flag="false"
fi
for pkg in $pkgs
do
	path="build/gen/${pkg//.//}"
	mkdir -p $path
	echo -e "package $pkg;\npublic class BuildConfig {\n	public static final boolean DEBUG=$flag;\n}" >> "$path/BuildConfig.java"
done

# Get list of sources
echo 'Generating list of sources...'
for dir in $src
do
	echo -e `scandir $dir` >> build/bin/sources.list
done

# Run javac
echo 'Compiling Java sources...'
javac -encoding utf-8 -cp $jar @build/bin/sources.list -d build/bin/classes

# Dex
echo 'Dexing...'
jar=${jar//:/ }
jar=${jar//$ANDROID_JAR/}
dx --dex --no-strict --output=build/bin/classes.dex build/bin/classes $jar

# Merge the dex and the apk
echo 'Merging...'
cd build/bin
aapt a build.apk classes.dex
cd ../..

# Sign
echo 'Signing...'
if [ ${1} = "debug" ]; then
	jarsigner -keystore keystore/debug.keystore -storepass android build/bin/build.apk my_alias
elif [ ${1} = "release" ]; then
	# Jarsigner will ask me for my passwords ^_^
	jarsigner -keystore keystore/publish.keystore build/bin/build.apk peter
fi

# Finished
echo "Apk built for $1: ${PWD}/build/bin/build.apk"
{% endhighlight %}

我的项目托管在 <https://github.com/PeterCxy/BlackLight> ，上面的脚本也在项目里面，可以直接编译成功。你可以关注我的项目的最新更新来获取最新的编译脚本示例。

这个脚本仅仅是一个示例，不能直接用来编译你的项目。你需要修改 `Configurations` 段 以及 `Sign` 段里面的参数才能成功编译。你可以参考我的 `BlackLight` 项目信息并对比你自己的项目信息来修改这个脚本。主要就是引入资源文件、库源码、预编译库、签名文件这四项。如果你修改脚本时遇到问题，欢迎在本文下面留言或通过社交渠道联系我，我会尽力给予帮助。

#### 有图有真相

古人云：没图你说个JJ！

![有图有真相]({{ site.baseurl }}res/buildapk.png)

#### 效率

在我的破米1上，最快的一次编译用了大约3分钟，主要时间都耗在转换dex格式上面了。

#### 潜在错误

在运行 `dx --dex` 的时候可能发生 `OutOfMemoryError` ，此时你只需要编辑 `/sdcard/env/tools/dx` 文件（对，它是文本文件），修改最大内存限制即可。

#### 意义

* 可以让学生党用手机做开发
* 可以帮助你了解Android App的结构及编译过程
* 可以拯救不开发会死星人
* 可以帮助你的手机顺利晋级为暖手宝，居家旅行必备！

#### 鸣谢

* Terminal IDE
* Apktool
* [SSH and GIT on Android with Terminal IDE](http://tinyrobot.co.uk/blog/ssh-and-git-on-android-with-terminal-ide/)
