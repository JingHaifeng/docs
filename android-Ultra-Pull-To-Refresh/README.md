android-Ultra-Pull-To-Refresh 源码解析
====================================
> 本文为 [Android 开源项目源码解析](https://github.com/android-cn/android-open-project-analysis) 中 android-Ultra-Pull-To-Refresh 部分  
> 项目地址：[android-Ultra-Pull-To-Refresh](https://github.com/liaohuqiu/android-Ultra-Pull-To-Refresh)，分析的版本：[508c632](https://github.com/liaohuqiu/android-Ultra-Pull-To-Refresh/tree/508c63266de51ad8c010ac9912f7592b2f2da8fc)，Demo 地址：[android-Ultra-Pull-To-Refresh Demo](https://github.com/android-cn/android-open-project-demo/tree/master/android-ultra-pull-to-refresh-demo)    
> 分析者：[Grumoon](https://github.com/grumoon)，校对者：，校对状态：未完成   
 

###1. 功能介绍  
下拉刷新，几乎是每个Android应用都会需要的功能。android-Ultra-Pull-To-Refresh（以下简称 UltraPTR ）便是一个强大的Andriod下拉刷新框架。    
主要特点：  
(1).继承于 ViewGroup，Content 可以包含任何View。  
(2).简洁完善的 Header 抽象，方便进行拓展，构建符合需求的头部。
> 对比 [Android-PullToRefresh](https://github.com/chrisbanes/Android-PullToRefresh) 项目，UltraPTR 没有实现 **加载更多** 的功能，但我认为 **下拉刷新** 和 **加载更多** 不是同一层次的功能， **下拉刷新** 有更广泛的需求，可以适用于任何页面。而 **加载更多** 的功能应该交由具体的 Content 自己去实现。这应该是和Google官方推出 SwipeRefreshLayout 是相同的设计思路，但对比 SwipeRefreshLayout，UltraPTR 更灵活，更容易拓展。


###2. 详细设计
###2.1 核心类功能介绍

####2.1.1 PtrHandler.java
下拉刷新功能接口，对下拉刷新功能的抽象。  
```java
public void onRefreshBegin(final PtrFrameLayout frame)
```  
刷新回调函数，用户在这里写自己的刷新功能实现。  

```java
public boolean checkCanDoRefresh(final PtrFrameLayout frame, final View content, final View header)
```
判断是否可以下拉刷新。UltraPTR 的 Content 可以包含任何内容，用户在这里判断决定是否可以下拉。  
例如，如果 Content 是 TextView ，则可以直接返回 true，表示可以下拉刷新。  
如果 Content 是 ListView ，则第一条在顶部时返回 true，表示可以下拉刷新。  
如果 Content 是 ScrollView ，则滑动到顶部时返回 true ，表示可以刷新。  

####2.1.2 PtrDefaultHandler.java
抽象类，实现了 PtrHandler.java 接口，给出了 `checkCanDoRefresh` 默认实现。  
```java
@Override
public boolean checkCanDoRefresh(PtrFrameLayout frame, View content, View header) {
	return checkContentCanBePulledDown(frame, content, header);
}

public static boolean checkContentCanBePulledDown(PtrFrameLayout frame, View content, View header) {
    /**
     * 如果Content不是ViewGroup，返回true,表示可以下拉</br>
     * 例如：TextView，ImageView
     */
    if (!(content instanceof ViewGroup)) {
        return true;
    }

    ViewGroup viewGroup = (ViewGroup) content;

    /**
     * 如果Content没有子View（内容为空）时候，返回true，表示可以下拉
     */
    if (viewGroup.getChildCount() == 0) {
        return true;
    }

    /**
     * 如果Content是AbsListView（ListView，GridView），当第一个item不可见是，返回false，不可以下拉。
     */
    if (viewGroup instanceof AbsListView) {
        AbsListView listView = (AbsListView) viewGroup;
        if (listView.getFirstVisiblePosition() > 0) {
            return false;
        }
    }

    /**
     * 如果SDK版本为14以上，可以用canScrollVertically判断是否能在竖直方向上，向上滑动</br>
     * 不能向上，表示已经滑动到在顶部或者Content不能滑动，返回true，可以下拉</br>
     * 可以向上，返回false，不能下拉
     */
    if (Build.VERSION.SDK_INT >= 14) {
        return !content.canScrollVertically(-1);
    } else {
        /**
         * SDK版本小于14，如果Content是ScrollView或者AbsListView,通过getScrollY判断滑动位置 </br>
         * 如果位置为0，表示在最顶部，返回true，可以下拉
         */
        if (viewGroup instanceof ScrollView || viewGroup instanceof AbsListView) {
            return viewGroup.getScrollY() == 0;
        }
    }

    /**
     * 最终判断，判断第一个子View的top值</br>
     * 如果第一个子View有margin，则当top==子view的marginTop+content的paddingTop时，表示在最顶部，返回true，可以下拉</br>
     * 如果没有margin，则当top==content的paddinTop时，表示在最顶部，返回true，可以下拉
     */
    View child = viewGroup.getChildAt(0);
    ViewGroup.LayoutParams glp = child.getLayoutParams();
    int top = child.getTop();
    if (glp instanceof ViewGroup.MarginLayoutParams) {
        ViewGroup.MarginLayoutParams mlp = (ViewGroup.MarginLayoutParams) glp;
        return top == mlp.topMargin + viewGroup.getPaddingTop();
    } else {
        return top == viewGroup.getPaddingTop();
    }
}

```
这里特别注意一下，以上代码中存在一些小bug。[Issue](https://github.com/liaohuqiu/android-Ultra-Pull-To-Refresh/issues/30)
```java
if (viewGroup instanceof ScrollView || viewGroup instanceof AbsListView) {
    return viewGroup.getScrollY() == 0;
}
```
如果 Content 是AbsListView（ListView，GridView），通过getScrollY（）获取的值一直是0，所以这段代码的判断，无效。
####2.1.3 PtrUIHandler.java
下拉刷新，UI事件接口
####2.1.4 PtrUIHandlerHolder.java
封装了PtrUIHandler.java的链表
####2.1.5 PtrFrameLayout.java
下拉刷新实现类
####2.1.6 PtrClassicFrameLayout.java
继承PtrFrameLayout.java，经典下拉刷新实现类
####2.1.7 PtrClassicDefaultHeader.java
经典下拉刷新实现类的头部实现
####2.1.8 PtrUIHandlerHook.java
####2.1.9 MaterialHeader.java
Material Design风格的头部实现
####2.1.10 MaterialProgressDrawable.java
####2.1.11 StoreHouseHeader.java
StoreHouse风格的头部实现
####2.1.12 StoreHouseBarItem.java
####2.1.13 StoreHousePath.java
####2.1.14 PtrLocalDisplay.java
显示相关工具类  

 

  
###2.2 类关系图
类关系图，类的继承、组合关系图，可是用 StartUML 工具。  

**完成时间**  
- 根据项目大小而定，目前简单根据项目 Java 文件数判断，完成时间大致为：`文件数 * 7 / 10`天，特殊项目具体对待  

###3. 流程图
主要功能流程图  
- 如 Retrofit、Volley 的请求处理流程，Android-Universal-Image-Loader 的图片处理流程图  
- 可使用 StartUML、Visio 或 Google Drawing 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈  

**完成时间**  
- `两天内`完成  

###4. 总体设计
整个库分为哪些模块及模块之间的调用关系。  
- 如大多数图片缓存会分为 Loader 和 Processer 等模块。  
- 可使用 StartUML、Visio 或 Google Drawing 等工具完成，其他工具推荐？？  
- 非所有项目必须，不需要的请先在群里反馈。  

**完成时间**  
- `两天内`完成  

###5. 杂谈
该项目存在的问题、可优化点及类似功能项目对比等，非所有项目必须。  

**完成时间**  
- `两天内`完成  

###6. 修改完善  
在完成了上面 5 个部分后，移动模块顺序，将  
`2. 详细设计` -> `2.1 核心类功能介绍` -> `2.2 类关系图` -> `3. 流程图` -> `4. 总体设计`  
顺序变为  
`2. 总体设计` -> `3. 流程图` -> `4. 详细设计` -> `4.1 类关系图` -> `4.2 核心类功能介绍`  
并自行校验优化一遍，确认无误后，让`校对 Buddy`进行校对，`校对 Buddy`校对完成后将  
`校对状态：未完成`  
变为：  
`校对状态：已完成`  

**完成时间**  
- `两天内`完成  

**到此便大功告成，恭喜大家^_^**  
