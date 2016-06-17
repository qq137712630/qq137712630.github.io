title: Android 夜间模式：Colorful 使用
---

# Android 夜间模式：Colorful 使用

[https://github.com/hehonghui/Colorful](https://github.com/hehonghui/Colorful)
[有妹子的日报 MeiZiNews ](https://github.com/qq137712630/MeiZiNews)

夜间模式最好在项目一开始就决定好，不然在后来的修改会很麻烦

---

## Colorful 夜间模式的实现

其实这项目的方法主要是以切换 theme 来实现夜间模式，但它有个好处就是可以随时切换，而实现这效果的主要方法是在 Colorful 类里：

```Java

	Colorful.java
	
	/**
	 * 设置新的主题
	 *
	 * @param newTheme
	 */
	protected void setTheme(int newTheme) {
	    mActivity.setTheme(newTheme);
	    makeChange(newTheme);
	}
	
	/**
	 * 修改各个视图绑定的属性
	 */
	private void makeChange(int themeId) {
	    Theme curTheme = mActivity.getTheme();
	    for (ViewSetter setter : mElements) {
	        setter.setValue(curTheme, themeId);
	    }
	}

```

它通过 Activity.setTheme 来更改主题然后 通过 makeChange 方法来更改每一个写入的 View 的属性 

相关文章：

 - [张涛：Android夜间模式实现](http://kymjs.com/code/2015/05/26/01)
 - [夜晚的故事(android夜间模式实现)](http://www.jianshu.com/p/60608820bb71)
 - [Android夜间模式最佳实践](http://mp.weixin.qq.com/s?__biz=MzA4MjU5NTY0NA==&mid=401740657&idx=1&sn=8e6727fbe094ea42d5fd80b185a49395#rd)

---

## Colorful 使用

一般的使用方法可以参考官方的，它是自带 textColor 和 background 的了

 - [https://github.com/hehonghui/Colorful](https://github.com/hehonghui/Colorful)

如果你还需修改更多的属性，可以参考 setter 包下的类来编写自己的构造类，通过使用 Colorful.Builder(activity).setter 方法来调用

当然还有比较复杂的情况，下面是我遇到的。

以自己的开源项目 [MeiZiNews](https://github.com/qq137712630/MeiZiNews)  来讲

### Activity + ViewPager + Fragment

有2种方法：

 1. Activity 传值
 2. 使用事件总线框架

而我使用了 RxJava ，所以就选择 RxBus-事件总线：Fragment 通过收到事件，然后直接在事件处理中使用 setTheme

RxBus-事件总线

按照以下顺序阅读的话，大概就理解如何使用了

 1. [翻译：通过 RxJava 实现一个 Event Bus – RxBus](https://drakeet.me/rxbus)
 2. [当EventBus遇上RxJava](http://www.easydone.cn/2016/03/29/)
 3. [RXBUS入门](http://www.james-ye.com/2016-04-07/rxbus%E5%85%A5%E9%97%A8.html)
 4. [做更好的RxBus：用RxJava实现事件总线(Event Bus)](http://www.jianshu.com/p/ca090f6e2fe2/)

 [Subject解读：ReactiveX文档中文翻译](https://mcxiaoke.gitbooks.io/rxdocs/content/Subject.html)

### RecyclerView

虽然官方有 ViewGroupSetter 可以用，但在 ViewPager + Fragment 的 RecyclerView 会有个别的 Item 没有变化：

[https://github.com/hehonghui/Colorful/issues/5](https://github.com/hehonghui/Colorful/issues/5)

所以，我根据上面的问题新建一个 RecyclerViewSetter 类：
	
```Java

	public class RecyclerViewSetter extends ViewGroupSetter {
	
	
	    public RecyclerViewSetter(ViewGroup targetView, int resId) {
	        super(targetView, resId);
	    }
	
	    public RecyclerViewSetter(ViewGroup targetView) {
	        super(targetView);
	    }
	
	    @Override
	    protected void clearRecyclerViewRecyclerBin(View rootView) {
	        super.clearRecyclerViewRecyclerBin(rootView);
	
	        ((RecyclerView) rootView).getRecycledViewPool().clear();
	    }
	
	    @Override
	    public void setValue(Resources.Theme newTheme, int themeId) {
	
	        clearRecyclerViewRecyclerBin(mView);
	
	        // 遍历子元素与要修改的属性,如果相同那么则修改子View的属性
	        for (ViewSetter setter : mItemViewSetters) {
	
	            for (int i = 0; i < ((RecyclerView) mView).getAdapter().getItemCount(); i++) {
	
	                View itemView = ((RecyclerView) mView).getChildAt(i);
	                if (itemView == null) {
	                    continue;
	                }
	
	                boolean isBaseAdapterHelper = ((RecyclerView) mView).getChildViewHolder(itemView) instanceof BaseAdapterHelper;
	
	                if (!isBaseAdapterHelper) {
	                    continue;
	                }
	
	                BaseAdapterHelper baseAdapterHelper = (BaseAdapterHelper) ((RecyclerView) mView).getChildViewHolder(itemView);
	                setter.mView = baseAdapterHelper.getView(setter.mViewId);
	                int itemId = setter.getViewId();
	
	                if (baseAdapterHelper.getView(itemId) == null) {
	                    continue;
	                }
	                setter.setValue(newTheme, themeId);
	            }
	        }
	    }
	
	}

```

---

最后，推荐下自己的开源项目：

[有妹子的日报:https://github.com/qq137712630/MeiZiNews](https://github.com/qq137712630/MeiZiNews)

[APK下载：http://www.wandoujia.com/apps/com.ms.meizinewsapplication](http://www.wandoujia.com/apps/com.ms.meizinewsapplication)

它使用了多个开源框架，并在 ReadMe 中都有标明其框架的使用在哪个方面，遇到了什么奇怪的问题，中文的哦，喜欢的， Star 一下。

---

### 我的公众号

![](img/wx_er_wei_ma.jpg)