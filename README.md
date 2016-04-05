# android-accumulation
Accumulating android for myself.

## Draw View 绘制

### onMeasure()
计算view大小
### onLayout()
确定view于content中的位置
### onDraw()
绘制view

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
