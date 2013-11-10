---
layout: default
title: SlidingPaneLayout的使用
category: tech
summary: 想必各位都体验过最新版环聊，最新版环聊采用的是一种左右滑动的抽屉界面，当点击actionbar图标或手动滑动的时候，界面就会切换，如图：
---
想必各位都体验过最新版环聊，最新版环聊采用的是一种左右滑动的抽屉界面，当点击actionbar图标或手动滑动的时候，界面就会切换，如图：

![screenshot](/code/img/slidingpanlayout_1.png)

大爱这种界面，正好我最近在写FlashBak备份软件，因此一直想着如何实现。  
从Android Design的类查询里找到，这东西叫SlidingPaneLayout，在android-support-v4里面，功能是将自己的两个子view分成两层，可以通过滑动来切换。  
但是，只使用它自己的功能是远远不够的，因为他只能切换view。  
所以，第一个想法就是，通过Fragment来实现类似环聊的界面。  
首先把原来的子Activity全部改造成Fragment，这个有点功底的人都会。然后需要在主activity的布局里，将原先的Layout整个用android.support.v4.widget.SlidingPaneLayout包裹起来，再在原来的layout之后，SlidingPane之内，增加一个有Id的FrameLayout，用来容纳Fragment。  
接下来就要对主Activity开刀了。  
因为SlidingPaneLayout默认是close。状态，这意味着右侧面板将默认覆盖左侧面板，很明显这和环聊的风格不一样，因此，需要调用 openPane() 这个方法将其默认打开。  
首次启动应用时，必须调用FragmentManager，将你希望默认显示的Fragment用replace方法替换到FrameLayout里面。  
如果你此时运行，会发现主程序的actionbar菜单和fragment的重叠了，所以必须写一个监听，自动显示和隐藏。  
如果你通过actionbar菜单激活Fragment，那么注意一个问题，就是不要每次滑动或者点击Action按钮切换都刷新，那样会导致效率低下，我的建议是添加一个bool变量来判断是否需要刷新。  
以上几点可以参考我这两个函数(Fragment里面的)，在相应事件监听到的时候调用。  
{% highlight java %}
	public void pause() {
		((Activity) mContext).setTitle(((MainBackupListActivity) mContext).FlashBakTitle);
		mMenu.findItem(R.id.invert_select).setVisible(false);
		mMenu.findItem(R.id.confirm_backup).setVisible(false);
		getActivity().getActionBar().setDisplayHomeAsUpEnabled(false);
	}
	
	public void resume() {
		((Activity) mContext).setTitle(R.string.title_new_backup);
		mMenu.findItem(R.id.invert_select).setVisible(true);
		mMenu.findItem(R.id.confirm_backup).setVisible(true);
		getActivity().getActionBar().setDisplayHomeAsUpEnabled(true);
		if (mNeedReload) {
			initDisplay();
			mNeedReload = false;
		}
	}
{% endhighlight %}

就像这样，在切换时自动显示隐藏菜单和返回键，按照变量里决定是否刷新显示。  
onBackPressed函数也是一个开刀的对象。  
{% highlight java %}
	@Override
	public void onBackPressed() {
		if (mPane.isOpen()) {
			finish();
		} else {
			mPane.openPane();
		}
	}
	
{% endhighlight %}

这段代码的作用是，如果程序的右侧面板被打开，就返回到主左侧面板，否则退出程序。  
做到以上几点以后，效果和环聊还是有很大差异的。  
第一个差异是在边缘上。默认没有阴影，这个好办，从环聊里提取了并代码调用SlidingPaneLayout.setShadowDrawable(R.draw able.xxxx)就可以了  
第二个差异是，环聊里，滑动抽屉时，两层之间会有一种连带运动，这就需要通过重写PanelSlideListener中的onPaneSlide函数来实现。  
{% highlight java %}
			@Override
			public void onPanelSlide(View v, float f) {
				mLayout.setPadding((int) -((1 - f) * 72), 0, (int) ((1 - f) * 72), 0);
			}
{% endhighlight %}

以上代码里面的mLayout为显示在主界面左边(SlidingPaneLayout的左侧面板)的LinearLayout，这个自己看着办就可以。  

以上是我开发FlashBak的UI时的一些经过，就当笔记吧。  
如果上面说的不是很清楚，你可以直接查看FlashBak的UI这块的源代码：<https://github.com/PeterCxy/FlashBak>
