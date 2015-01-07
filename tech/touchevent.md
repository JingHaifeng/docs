Java 注解 Annotation
----------------
> 本文为 [Android 开源项目实现原理解析](https://github.com/android-cn/android-open-project-analysis) 公共技术点的 事件传递 部分  
> 分析者：[Grumoon](https://github.com/grumoon)    

###事件传递
学习Android的人都知道事件传递的重要性吧，这块内容和View的绘制过程，共同构建了Android自定义控件的基础，相信将这两块内容弄明白以后，自定义控件那都是手到擒来的。View的绘制过程请[点击这里](https://github.com/android-cn/android-open-project-analysis/blob/master/tech/viewdrawflow.md)。   
事件传递对于每个学习Android的人，都是一块很难啃，但是又不得不啃的硬骨头。相信大家也都和我一样，看过无数人写这块的内容，但是发现每个人写的多多少少有点出入，让你觉得不知道谁是对的，又或者写的内容覆盖的不全面，看完一头雾水。这块内容因为涉及到Activity，ViewGroup，View三个类，dispatchTouchEvent，onInterceptTouchEvent，onTouchEvent三个方法，每个方法的true或者false的返回值，ACTION_DOWN，ACTION_MOVE，ACTION_UP...等事件。我相信很难有通过demo例子将其分析完全，不过因为我们是Android，因为我们有源码，我们可以从源代码中找寻答案。