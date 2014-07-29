---
layout: default
title: vim的Android开发自动完成插件
category: tech
---
`vim` 是一款众所周知的强大的文本编辑器。

哦不，它不只是一款强大的文本编辑器。由于它自身提供的 `vim脚本扩展` 功能，因此各路大神已经为它开发了无数强大的插件，使它甚至可以作为IDE使用。据说某些大神一直是使用 `vim` 这类的编辑器来编程的。

代码自动完成，这是IDE的最基本功能，而vim上的此类插件也并不少。`javacomplete` 就是这样一款支持java项目的自动完成插件。但是，它本身提供的功能很有限，不能自动引入 `android.jar`，也不支持读取项目引入的第三方库。

<!--more-->

#### 简介

so，作为一个极(zuo)客(si)型Android开发者，我自己对 `javacomplete` 插件进行了一番修改。修改后的插件地址：<https://github.com/PeterCxy/javacomplete>

我修改的内容就是添加支持对 `android.jar` 的引入以及对第三方jar包的调用。当然，要使用我修改后的插件，你需要对它进行配置。

#### 安装

你可以使用 `Pathogen` 来安装本插件。安装方法在各大搜索引擎上均能找到，你只需要把git地址换成我修改后的地址就可以了。

#### 配置.vimrc

安装好以后当然不能立刻使用。此时，你需要编辑你的 `~/.vimrc` (如果你用的是 `gvim` ，那么就是 `~/.gvimrc`），添加如下内容

{% highlight vim %}
filetype plugin indent on
autocmd Filetype java set omnifunc=javacomplete#Complete
autocmd Filetype java set completefunc=javacomplete#CompleteParamsInf
autocmd FileType java,javascript,jsp inoremap <buffer> . .<C-X><C-O><C-P>

" android.jar
let $ANDROID_JAR = '/path/to/your/android.jar'

" cache
let $JAVACOMPLETE_CACHE = '~/.jcc'
{% endhighlight %}

解释一下，前三行代码是对java文件开启javacomplete自动完成插件，第四行的功能是把触发自动完成的快捷键设为 `.`，方便使用。`$ANDROID_JAR` 一行的作用是设置 `android.jar` 的路径，请将其修改为指向您的sdk中的 `android.jar`, `$JAVACOMPLETE_CACHE` 一行的作用是设置缓存路径为 `~/.jcc`. 如果你在写代码时发现自动完成有问题，可以尝试删除缓存目录再试。

#### 配置你的项目

如果你的项目没有引入第三方jar（我才不信），那你可以跳过这一步。

如果你的项目引入了第三方jar，那么现在请切换到你的项目根目录，创建文件 `.javacomplete` ，然后在其中输入你引用的jar包的相对路径，一行一个。下面是一个示例，在这个示例中我们假定你的项目引用了 `android-support-v4.jar` 和 `gson-2.2.2.jar` 这两个包且都位于项目目录中的 `libs` 目录内。

{% highlight sh %}
libs/android-support-v4.jar
libs/gson-2.2.2.jar
{% endhighlight %}

#### 完成

你已经完成了配置，打开一个源码文件，看看效果吧。截图中我使用是 `freya` 配色方案。

![demo]({{ site.baseurl }}res/javacomplete.png)

#### 不足

这个插件仍然不支持直接引入源码项目的第三方库。

这个插件在某包下只有一个类的时候无法自动完成。

这个插件的速度还有提升空间。

等等等等。

但作为日常编程使用还是比较方便的，也够用了。
