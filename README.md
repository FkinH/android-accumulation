# android-accumulation
Accumulating android for myself.

## Framework 框架

Application, Application Framework, Libraries + Android Runtime, Linux Kernel

## Draw View 绘制
`Activity.setContentView()` ，逻辑关系。每一个Activity，都包含一个Window对象，它表示的是一个顶级的一整屏幕上面的界面逻辑。
在整个控件树的最顶端，是一个逻辑的树顶，`ViewParent`，在源码中的实现是`ViewRoot`。
它是整个控件树和WindowManager之间的事件信息的翻译者。WindowManager将用户的操作，翻译成为指令，发送给呈现在界面上的各个Window。

`onMeasure()` 计算view大小

`onLayout()` 确定view于content中的位置

`onDraw()` 绘制view

## View Performance 性能优化

### improving overdraw

1. 移除重复背景
2. 使用 @android:color/transparent 代替重复背景色
3. 使用 `<viewstub>` 动态加载不常用布局
4. 使用 viewpager+fragment 取代 setvisibility
5. 使用 `<merge>`、`<include>` 标签，合并/复用布局
6. 使用hierarchy viewer/uiautomatorviewer检查布局

### improving listview

1. 复用convertview减少adapter读取xml时的io操作
2. 使用viewholder管理item
3. 缓存数据
4. 分页加载
5. 简化item布局
6. 使用新的recycleview

## Event Delivery 事件传递

> Note:
> 1. 所有Touch事件都被封装成了`MotionEvent`对象。
> 2. 事件类型分为`ACTION_DOWN`, `ACTION_UP`, `ACTION_MOVE`, `ACTION_POINTER_DOWN`, `ACTION_POINTER_UP`, `ACTION_CANCEL`，以`ACTION_DOWN`开始`ACTION_UP`结束。
> 3. 对事件的处理包括三类，分别为传递—`dispatchTouchEvent()`, 拦截—`onInterceptTouchEvent()`, 消费—`onTouchEvent()`和`OnTouchListener`


[Android Touch事件传递机制](http://www.trinea.cn/android/touch-event-delivery-mechanism/)

### 流程

`Activity.dispatchTouchEvent()`->`Activity.onUserInteraction()`->`Layout.dispatchTouchEvent()`->`Layout.onIntercepTouchEvent()`

1. 事件从`Activity.dispatchTouchEvent()`开始传递，只要没有被停止或拦截，从最上层的`View(ViewGroup)`开始一直往下传递。子View可以通过onTouchEvent()对事件进行处理。

2. 事件由父View(ViewGroup)传递给子View，ViewGroup可以通过`onInterceptTouchEvent()`对事件做拦截，停止其往下传递。

3. 如果事件从上往下传递过程中一直没有被停止，且最底层子View没有消费事件，事件会反向往上传递，这时父View(ViewGroup)可以进行消费，如果还是没有被消费的话，最后会到Activity的`onTouchEvent()`函数。

4. 如果View没有对`ACTION_DOWN`进行消费，之后的其他事件不会传递过来。

5. `OnTouchListener`优先于`onTouchEvent()`对事件进行消费。

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

### Handler

## Life Cycle 组件生命周期
### Activity
```
onCreate()->onStart()->onRestart()->onResume()->onPause()->onStop()->onDestroy()
```
[Activity生命周期详解一](http://stormzhang.com/android/2014/09/14/activity-lifecycle1/)

[Activity生命周期详解二](http://stormzhang.com/android/2014/09/17/android-lifecycle2/)

具体细节以后补充
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
### Fragment

### Broadcast
