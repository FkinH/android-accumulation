### Binder
Binder 跨进程通信机制 模型 基于 `Client - Server` 模式 

一种实现`IPC`的方式，实现了`IBinder`接口

Binder数据拷贝只需要一次，而管道、socket、消息队列需要2次
Binder为进程分配了UID/PID作为身份鉴别
Binder面向对象


加载大图 bitmapregiondecoder lrucache
LruCache的核心思想很好理解，就是要维护一个缓存对象列表，其中对象列表的排列方式是按照访问顺序实现的，即一直没访问的对象，将放在队尾，即将被淘汰。而最近访问的对象将放在队头，最后被淘汰。
LinkedHashMap是由数组+双向链表的数据结构来实现的。其中双向链表的结构可以实现访问顺序和插入顺序，使得LinkedHashMap中的<key,value>对按照一定顺序排列起来。
LruCache中维护了一个集合LinkedHashMap，该LinkedHashMap是以访问顺序排序的。当调用put()方法时，就会在结合中添加元素，并调用trimToSize()判断缓存是否已满，如果满了就用LinkedHashMap的迭代器删除队尾元素，即近期最少访问的元素。当调用get()方法访问缓存对象时，就会调用LinkedHashMap的get()方法获得对应集合元素，同时会更新该元素到队头。


RecyclerView 和 ListView 性能和效果区别
 RecyclerView 就能支持 线性布局、网格布局、瀑布流布局 三种，而且同时还能够控制横向还是纵向滚动
 没有setemptyview，headerview，footerview
 局部刷新粒度notifyItemChanged

Listview
RecycleBin 移出屏幕时回收，进入屏幕后复用getScrapView()
convertView

算法：https://wizardforcel.gitbooks.io/the-art-of-programming-by-july/content/00.01.html

binder http://blog.csdn.net/carson_ho/article/details/73560642

https://hit-alibaba.github.io/interview/Android/Questions.html


### apk、dex、odex

https://www.jianshu.com/p/389911e2cdfb

ART : .dex->.odex(机器码)（AOT　　Ahead-Of-Time）
Dalvik: .dex->.odex(字节码)（JIT　Just-In-Time）
机器码可直接执行，而字节码每次启动都需要执行将优化过的odex字节码再转换成机器码

ART优缺####
系统性能大幅提升
App启动、运行更快
减少每次启动的编译增加电池续航
存储占用更大
安装时间更长