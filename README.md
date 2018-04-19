# android-accumulation
Accumulating android for myself.

## Framework 框架

Application, Application Framework, Libraries + Android Runtime, Linux Kernel

## View 绘制
`Activity.setContentView()` ，逻辑关系。每一个Activity，都包含一个Window对象，它表示的是一个顶级的一整屏幕上面的界面逻辑。
在整个控件树的最顶端，是一个逻辑的树顶，`ViewParent`，在源码中的实现是`ViewRoot`。
它是整个控件树和WindowManager之间的事件信息的翻译者。WindowManager将用户的操作，翻译成为指令，发送给呈现在界面上的各个Window。

`onMeasure()` 计算view大小

`onLayout()` 确定view于content中的位置

`onDraw()` 绘制view

view 绘制 https://github.com/FkinH/interview/blob/master/android/draw.md


## 事件传递

> Note:
> 1. 所有Touch事件都被封装成了`MotionEvent`对象。
> 2. 事件类型分为`ACTION_DOWN`, `ACTION_UP`, `ACTION_MOVE`, `ACTION_POINTER_DOWN`, `ACTION_POINTER_UP`, `ACTION_CANCEL`，以`ACTION_DOWN`开始`ACTION_UP`结束。
> 3. 对事件的处理包括三类，分别为传递—`dispatchTouchEvent()`, 拦截—`onInterceptTouchEvent()`, 消费—`onTouchEvent()`和`OnTouchListener`


`Activity.dispatchTouchEvent()`->`Activity.onUserInteraction()`->`Layout.dispatchTouchEvent()`->`Layout.onIntercepTouchEvent()`

1. 事件从`Activity.dispatchTouchEvent()`开始传递，只要没有被停止或拦截，从最上层的`View(ViewGroup)`开始一直往下传递。子View可以通过onTouchEvent()对事件进行处理。

2. 事件由父View(ViewGroup)传递给子View，ViewGroup可以通过`onInterceptTouchEvent()`对事件做拦截，停止其往下传递。

3. 如果事件从上往下传递过程中一直没有被停止，且最底层子View没有消费事件，事件会反向往上传递，这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到Activity的`onTouchEvent()`函数。

4. 如果View没有对`ACTION_DOWN`进行消费，之后的其他事件不会传递过来。

5. `OnTouchListener`优先于`onTouchEvent()`对事件进行消费。

`dispatchTouchEvent()`
true: 由自身此方法消费，停止向下传递
false: 由上级消费
super: onInterceptTouchEvent

`onInterceptTouchEvent()`
true/super: 拦截并消费
false: 由下级分发

`onTouchEvent()`
true: 消费
false/super: 由上级消费

`OnTouchListener`

http://blog.csdn.net/carson_ho/article/details/54136311
https://www.jianshu.com/p/e99b5e8bd67b

```
public boolean dispatchTouchEvent(MotionEvent ev) {    
    boolean consume = false;    
    if (onInterceptTouchEvent(ev)) {
        consume = onTouchEvent(ev);
    } else {
        consume = child.dispatchTouchEvent(ev);    
    }
    return consume;
}
```

## Multithreading 多线程

### AsyncTask

`SerialExecutor` ：将params 变成 future task
`THREAD_POOL_EXECUTOR`: 线程池

### Handler


http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2012/1025/475.html

一个线程只能有一个Looper，对应一个MessageQueue

Looper.prepare()
Looper.Loop()
MesegeQueue
msg.target

Q:Android中为什么主线程不会因为Looper.loop()里的死循环卡死？

A:ActivityThread实际上并非线程, Activity的生命周期都是依靠主线程的Looper.loop，当收到不同Message时, H.handleMessage(msg)

## Life Cycle 组件生命周期
### Activity
```
onCreate()->onStart()->onRestart()->onResume()->onPause()->onStop()->onDestroy()
```
[Activity生命周期详解一](http://stormzhang.com/android/2014/09/14/activity-lifecycle1/)

[Activity生命周期详解二](http://stormzhang.com/android/2014/09/17/android-lifecycle2/)

standard：默认启动模式，不管有没有已存在的实例都生成新的实例。
singleTop：如果栈顶存在对应的实例则重复利用不生产新的实例，不存在则新建实例。
singleTask：如果栈内存在对于的实例则使此Activity实例之上的其他Activity实例都出栈，使此Activity实例成为栈顶对象显示。
singleInstance：启用一个新栈放入新建Activity实例，并且该栈内只允许存在这一个Activity实例。

FLAG_ACTIVITY_NEW_TASK：将目标Activity放置到新的task中。
FLAG_ACTIVITY_CLEAR_TASK：启动一个Activity时先清除和其有关联的task，并新建Activity实例将其放入新的task中。必须和上面变量一起使用
FLAG_ACTIVITY_CLEAR_TOP：启动一个不处于栈顶的Activity时，清除排在它前面的Activity使其显示出来。


### Service
在Service每一次的开启关闭过程中，只有onStart可被多次调用(通过多次startService调用)

其他onCreate，onBind，onUnbind，onDestory在一个生命周期中只能被调用一次。
#### Start Service
```
onCreate()->onStart()->onDestroy()
```

#### Bind Service
```
onCreate()->onBind()->onDestroy()
```

