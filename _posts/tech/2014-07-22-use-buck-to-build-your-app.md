---
layout: default
title: 使用buck构建你的Android App
category: tech
---
`buck` 是Facebook推出的一款高效率的Android App/Java项目构建工具，目前仅支持 `Unix/Linux` 平台。因为它使用多线程编译方式，所以相对于其他的构建工具而言编译时间可以缩短50%甚至更多。

`buck` 提供了非常灵活的、可自定义的构建脚本。通过脚本，你可以为一个app的不同版本，如调试版、正式发布版编写互不一样的构建规则，而不需要开多个分支或者多个版本库。

好了不瞎扯介绍了，开始进入正题。这篇文章主要是推荐buck，并介绍buck的部署，以及如何用buck构建已有的app项目(主要是如何编写构建脚本)。

<!--more-->

### Buck的部署

#### 安装watchman

`watchman` 是 `buck` 的一个必须依赖，它被用来检测文件的变化，以便确定哪些文件需要重新编译。

`watchman` 需要 `automake` 这个依赖库。

{% highlight sh %}
# Ubuntu / Debian
$ sudo apt-get install automake
# ArchLinux
$ sudo pacman -S automake
# Gentoo
$ sudo emerge automake
{% endhighlight %}

然后克隆并编译安装 `watchman`

{% highlight sh %}
$ git clone https://github.com/facebook/watchman.git
$ cd watchman
$ ./autogen.sh
$ ./configure
$ make
$ sudo make install
{% endhighlight %}

#### 安装buck

首先确认你已经配置好 `$ANDROID_HOME` 这个环境变量指向您的sdk，并在sdk管理器中下载好您需要的api版本。

接下来我们可以克隆并构建 `buck`. `buck` 本身是使用 `ant` 构建的，具体原因请参阅 [FAQ](http://facebook.github.io/buck/concept/faq.html)

{% highlight sh %}
$ git clone https://github.com/facebook/buck.git
$ cd buck
$ ant
$ sudo ln -s /path-to-buck/bin/buck /usr/bin/buck
{% endhighlight %}

### 使用Buck编译你的App

本来这里应该还有一个 `quickstart` 向导，但我们就跳过吧。

这一节教程，我们将从配置库引用开始，到编译整个程序结束。我们的例子App拥有类似于Eclipse生成的目录结构。如果是其他类型的目录结构也不要担心，把下面对应的路径换一下就可以啦。

#### 配置库引用(如果有的话)

切换到你的app目录，首先建立一个名叫 `BUCK` 的空文件，并用文本编辑器打开。

首先我们引用预编译库(jar)，一般app至少都会引用 `android-support-v4.jar`，我们以此为例。在 `BUCK` 文件中加入:

{% highlight sh %}
prebuilt_jar(
  name = 'android-support-v4',
  binary_jar = './libs/android-support-v4.jar',
  visibility = [ 'PUBLIC' ],
)
{% endhighlight %}

解释一下，`prebuilt_jar` 表示定义一个预编译库，`name` 是其名称，这里我们用了全名，你要是嫌烦也可以用缩写之类的，只要你自己记住就可以。`binary_jar` 指向的是这个库文件的路径(`./` 表示的是Linux下的当前目录)。`visibility` 一行表示全局可见。

如果你有多个jar，那么就需要为每一个jar添加类似于这一段的内容。

接下来我们添加非预编译的库。我们假设你的库项目路径在 `./libs/MyLibrary`，资源目录为 `./libs/MyLibrary/res`,源码目录为 `./libs/MyLibrary/src` , 包名为 `com.my.library`, 并且需要 `android-support-v4` 才能编译。

{% highlight sh %}
android_resource(
  name = 'mylibrary-res',
  res = './libs/MyLibrary/res',
  package = 'com.my.library',
  visibility = [ 'PUBLIC' ],
)

android_library(
  name = 'mylibrary-src',
  srcs = glob(['./libs/MyLibrary/src/**/*.java']),
  deps = [
    ':mylibrary-res',
    ':android-support-v4',
  ],
  visibility = [ 'PUBLIC' ],
)
{% endhighlight %}

以上两段中的 `name` 都可以自己取，只不过其他对应的地方也要改过来。

第一段 `android_resource` 定义的是库的资源目录，由 `res` 一项配置指定路径，由 `package` 指定库的包名，被下一段 `android_library` 中的 `deps` 引用后即可生效。如果你的库没有资源目录，那么请不要写 `android_resource` 一段，同时将 `android_library` 一段中对资源的引用 `:mylibrary-res` 删除。

第二段 `android_library` 定义的是这个库。`srcs` 一项是引用了它的 `src` 目录下的全部 `java` 文件作为源码。 `deps` 是该库源码的依赖，本例子中它只依赖自己的资源 `:mylibrary-res` 和支持库 `:android-support-v4` ，这些都是我们已经定义好的名称，冒号是指本库与其它被引用的内容在同一项目中。

如果你引用了多个库，请为其每一个都添加这样的内容。

#### 配置签名文件

我们假设你的签名文件位于 `./keystore/debug.keystore`. 首先在同目录创建一个 `debug.keystore.properties` 文件，该文件的模板：

{% highlight sh %}
# 传递给keytool的 -alias 参数
key.alias=my_alias

# keystore文件的密码
key.store.password=store_password

# my_alias的密码
key.alias.password=alias_password
{% endhighlight %}

创建好以后，继续编辑 `BUCK` 文件。

{% highlight sh %}
keystore(
  name = 'debug_keystore',
  store = './keystore/debug.keystore',
  properties = './keystore/debug.keystore.properties',
)
{% endhighlight %}

同样，名字可以自己取。

如果你有多个签名文件，在这里也可以添加多个。当然，接下来的步骤中你也需要添加多个app编译规则才行。

#### 配置app编译规则

继续编辑 `BUCK` 文件。

{% highlight sh %}
android_binary(
  name = 'debug',
  package_type = 'DEBUG',
  manifest = './AndroidManifest.xml',
  target = 'android-19',
  keystore = ':debug_keystore',
  deps = [
    ':res',
    ':src',
    ':mylibrary-res',
  ],
)

android_resource(
  name = 'res',
  res = './res',
  assets = './assets',
  package = 'com.mycompany.myapp',
  deps = [ ':mylibrary-res' ],
  visibility = [ 'PUBLIC' ],
)

android_library(
  name = 'src',
  srcs = glob(['./src/**/*.java']),
  deps = [
    ':build_config',
    ':res',
    ':mylibrary-src',
    ':android-support-v4',
  ],
)

android_build_config(
  name = 'build_config',
  package = 'com.mycompany.myapp',
)
{% endhighlight %}

其中，`name` 是用于内部的名称，这个可以自己取。`package_type` 是指该程序的类型，我们这里选的是 `DEBUG` ，也就是调试版。另一个可选的值是 `RELEASE` ，对这个值的使用我们将在下面讨论到。

所有 `package` 一项的值都应当是你的app的包名。`keystore` 是签名，而 `deps` 是依赖。对于本部分的 `android_library` 一条而言，它的依赖应当是自己的资源 `res` ，编译配置 `build_config`，引用的库文件的源码 `mylibrary-src` ，以及引用的jar包 `android-support-v4`，这些都是先前配置中为它们取的名称，所以请务必确保和上面的配置对应。同时别忘记冒号。

对于 `android_resource` 一条而言，其依赖应当是（其实我也不确定）所有依赖库的资源，如果不添加依赖可能会出错。

`android_binary` 定义的是对可执行文件的编译，其依赖 `deps` 应当是自己的源码 `src` 资源 `res` 和其他所有依赖库的资源。

接下来我们需要创建一个新的文件，名称为 `.buckconfig` ,打开并编辑

{% highlight sh %}
[alias]
    debug = //:debug
[java]
    src_roots = /src
[project]
    # IntelliJ requires that every Android module have an
    # AndroidManifest.xml file associated with it. In practice,
    # most of this is unnecessary boilerplate, so we create one
    # "shared" AndroidManifest.xml file that can be used as a default.
    default_android_manifest = /AndroidManifest.xml 
{% endhighlight %}

这里我相信不用做过多解释。唯一要解释的是 `[alias]` 配置段，它是给 `//:debug` ，即我们之前定义的app编译规则，创建一个叫做 `debug` 的别名。这将在编译时用到。

接下来我们可以编译了。

{% highlight sh %}
$ buck build debug
{% endhighlight %}

如果不出错，它将会输出编译结果及编译好的apk文件名。

#### Proguard的使用

本来教程应该到这里就结束了，但我想我必须提一下 `Proguard`。这个东西大家都不陌生。在buck中如何使用呢？

很简单，我们只需要回到上面的 `android_binary` 一条（你也可以创建一条新的 `android_binary` 规则，但名字不能和上面的那条一样），并将 `DEBUG` 改为 `RELEASE` ，然后新加一行 `proguard_config = './proguard.cfg',` 再编译即可。

如果你添加了多条 `android_binary` 规则，请记得去 `.buckconfig` 里面在 `[alias]` 配置段中添加你新增规则的名称以便编译。此时编译命令将变成 `buck build 名称` 。

### 实例

[BlackLight](https://github.com/PeterCxy/BlackLight) 我的配置中包含了预编译jar、非预编译的有资源的库和无资源的库、还有两个编译规则：调试版和正式发布版。
[ChatterBox](https://gitcafe.com/neo4026/ChatterBox)

### 鸣谢

[但丁不淡定](http://neo4026.gitcafe.com/build-android-app-with-buck/)
