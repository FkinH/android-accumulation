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

## Life Circle 组件生命周期
### Activity
