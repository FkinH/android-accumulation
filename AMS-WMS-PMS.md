ams AMS统一调度所有应用程序的Activity

1.首先IPC调用AMS方法传入参数启动指定Activity
2.在AMS中首先查询PKMS获取该ActivityInfo，新建ActivityRecord和根据lunchMod创建TaskRecord两个重要变量，并且将ActivityRecord添加到task栈顶作为准备启动的Activity。
3.在正式启动Activity之前首先检查其进程是否启动，为启动时通知AMS启动对应进程。

1.首先就是zygote进程fork新进程，指定新进程执行需要启动的ActivityThread类。
2.在ActivityThread的main()方法中：
首先新建ActivityThread对象。
接着回调AMS接口通知其应用进程已启动成功。
最后开启主线程消息循环，处理消息。

3.其中上步骤第二点在与AMS交互中IPC调用AMS的attachApplication()函数中：
首先根据pid查找对应的ProcessRecord
设置该应用退出时的回调接口
修改ProcessRecord一些参数
最后与应用进程交互，开始创建Application对象

4.AMS与应用进程交互时会发出一个异步消息，该消息内部：
配置运行时信息
安装该Application对应的ContentProvider
生成一个Application对象并且回调onCreate()函数
5.AMS与应用进程交互完接着准备启动目标Activity，在ActivityStackSupervisor.attachApplicationLocked()函数中会首先获取需要启动的ActivityRecord并保存到对应的ProcessRecord，接着IPC调用应用进程启动Activity。

6.在应用进程中也是异步消息处理
首先反射实例化目标Activity对象，回调相应onStart()之前的生命周期函数
接着回调onResume()函数并且添加一个Idler消息对象到队列中。该对象内部方法会调用到AMS中的相应方法处理因此次启动而被暂停的Activity的相关声明周期函数处理。

7.同时在AMS中会将task添加到最近任务列表中，并且发送10s定时等待这个Activity处理结果。
8.此时目标Activity已经被成功启动，接着会处理一些Pending组件，这些组件可能是在系统启动未成功时发起的一些启动指令。

最终一个Activity启动成功。

wms
1. 窗口的添加和删除
2. 窗口的显示和隐藏控制
3. Z-order顺序管理
4. 焦点窗口和焦点应用的管理
5. 输入法窗口管理和墙纸窗口管理
6. 窗口动画管理
7. 系统消息收集和分发

1.每个应用类窗口都对应一个Activity对象，所以创建应用类窗口需要创建Activity对象。当AmS要启动某个Activity时就会通知客户端进程，每个客户端进程都对应一个ActivityThread类，所以需要ActivityThread启动Activity。
启动某个Activity实际是构造一个Activity对象，使用ClassLoader从程序文件中装载指定的Activity对应的Class文件。

2.创建完成Activity对象后调用Activity的attach（）方法，attach（）的作用就是为刚刚创造好的Activity设置内部变量。

3.为该Activity创建Window对象。

4.给Window对象中的mWindowManager变量赋值。

5.然后就需要给该窗口添加真正的View或者ViewGroup。从performLaunchActivity（）调用callActivityOnCreate（）开始，然后经一系列调用到Activity的onCreate（）方法，在onCreate（）方法中调用setContentView（）方法实际是调用了其对应的Window对象的setContentView（）方法。

6.接着会调用到PhoneWindow的setContentView，首先调用installDecor（）为Window类添加窗口装饰，其实就是标题栏，程序中设置的layout.xml界面被包含在窗口装饰中，叫做窗口内容。窗口装饰也是ViewGroup，窗口装饰和它内部的内容加起来就是我们所说的窗口，或者叫做Window界面。

7.把创建的窗口通知WmS，让WmS把窗口显示在屏幕上。当Activity准备好后会通知Ams，然后Ams经过一系列调用到Activity的makeVisible（），该方法将真正完成把窗口添加进Wms中。

8.在makeVisible方法中，首先获得该Activity内部的WindowManager对象，然后调用该对象的addView（）方法。

9.调用WindowManagerImpl的addView（）方法，流程如下：

检查添加的窗口是否已经添加过，不能重复添加。

如果添加的窗口是子窗口类型，找到父窗口并保存在临时变量panelParentView中，该变量作为后面调用ViewRoot的setView（）参数。

创建一个新的ViewRoot

调用ViewRoot的setView（）。

10.完成新建一个ViewRoot对象后，需要把新建的ViewRoot对象添加到mRoots对象中。

11.调用ViewRoot对象的setView方法。流程如下：

给ViewRoot的重要变量赋值。

调用requestLayout（），发出界面重绘请求。

调用sWindowSession.add（），通知Wms添加窗口。

pms 总结起来就是扫描settings, Android系统中的apk,并且建立相应的数据结构去管理Package的信息，四大组件的信息，权限信息等内容。




