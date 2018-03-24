### 综述

作者：BlackSwift
链接：https://www.jianshu.com/p/aad5aacd79bf

支持HTTP2/SPDY黑科技
socket自动选择最好路线，并支持自动重连
拥有自动维护的socket连接池，减少握手次数
拥有队列线程池，轻松写并发
拥有Interceptors轻松处理请求与响应（比如透明GZIP压缩,LOGGING）
基于Headers的缓存策略

主要对象
Connections: 对JDK中的物理socket进行了引用计数封装，用来控制socket连接
Streams: 维护HTTP的流，用来对Requset/Response进行IO操作
Calls: HTTP请求任务封装
StreamAllocation: 用来控制Connections/Streams的资源分配与释放

Dispatcher
当我们用OkHttpClient.newCall(request)进行execute/enenqueue时，实际是将请求Call放到了Dispatcher进行线程分发，它有两种方法，
一个是普通的同步单线程；
另一种是使用了队列进行并发任务的分发(Dispatch)与回调

Socket管理(StreamAllocation)
连接socket链路(RealConnection)

HTTP请求序列化/反序列化
获得HTTP流（httpStream）
```
httpStream = connect();

//source 用于获取response
source = Okio.buffer(Okio.source(rawSocket)); // inputstream
//sink 用于write buffer 到server
sink = Okio.buffer(Okio.sink(rawSocket)); // outputstream
```


### 复用连接池

Okhttp支持5个并发KeepAlive，默认链路生命为5min

Connection自动回收的实现
okhttp是根据`RealConnection`的虚引用`StreamAllocation`引用计数是否为0实现的。

连接池内部维护了一个叫做OkHttp ConnectionPool的ThreadPool，专门用来淘汰末位的socket，当满足以下条件时，就会进行末位淘汰，非常像GC
1. 并发socket空闲连接超过5个
2. 某个socket的keepalive时间大于5分钟
Connection中的StreamAllocation被不断的aquire与release，也就是List<WeakReference<StreamAllocation>>的大小将时刻变化

1. 遍历Deque中所有的RealConnection，标记泄漏的连接
2. 如果被标记的连接满足(空闲socket连接超过5个&&keepalive时间大于5分钟)，就将此连接从Deque中移除，并关闭连接，返回0，也就是将要执行wait(0)，提醒立刻再次扫描
3. 如果(目前还可以塞得下5个连接，但是有可能泄漏的连接(即空闲时间即将达到5分钟))，就返回此连接即将到期的剩余时间，供下次清理
4. 如果(全部都是活跃的连接)，就返回默认的keep-alive时间，也就是5分钟后再执行清理
5. 如果(没有任何连接)，就返回-1,跳出清理的死循环

标记并找到最不活跃的连接
1. 遍历RealConnection连接中的StreamAllocationList，它维护着一个弱引用列表
2. 查看此StreamAllocation是否为空(它是在线程池的put/remove手动控制的)，如果为空，说明已经没有代码引用这个对象了，需要在List中删除
3. 遍历结束，如果List中维护的StreamAllocation删空了，就返回0，表示这个连接已经没有代码引用了，是泄漏的连接;否则返回非0的值，表示这个仍然被引用，是活跃的连接。


### 缓存策略
1. Expires
表示到期时间，一般用在response报文中，当超过此事件后响应将被认为是无效的而需要网络连接，反之而是直接使用缓存

2. Cache-Control
相对值，单位是秒，指定某个文件被续多少秒的时间，从而避免额外的网络请求。比expired更好的选择，它不用要求服务器与客户端的时间同步，也不用服务器时刻同步修改配置Expired中的绝对时间，而且它的优先级比Expires更高。

3. 修订文件名(Reving Filenames)
我们这时可以通过修改url中的文件名版本后缀进行缓存，

4. 条件GET请求(Conditional GET Requests)与304
如缓存果过期或者强制放弃缓存，在此情况下，缓存策略全部交给服务器判断，客户端只用发送条件get请求即可，如果缓存是有效的，则返回304 Not Modifiled，否则直接返回body。


### 拦截器Interceptor

作者：miraclehen
链接：https://www.jianshu.com/p/fc4d4348dc58
來源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

拦截器可以进行链式处理。
OkHttp利用List集合去跟踪并且保存这些拦截器，并且会依次遍历调用。

拦截器可以以`application`或者`network`两种方式注册，分别调用`addInterceptor()`以及`addNetworkInterceptor`方法进行注册。

```
OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(new LoggingInterceptor())
    .build();

Request request = new Request.Builder()
    .url("http://www.publicobject.com/helloworld.txt")
    .header("User-Agent", "OkHttp Example")
    .build();

Response response = client.newCall(request).execute();
response.body().close();
```

Application interceptors
1. 不需要担心中间响应，如重定向和重试。
2. 总是调用一次，即使从缓存提供HTTP响应。
3. 遵守应用程序的原始意图。不注意OkHttp注入的头像If-None-Match。
4. 允许短路和不通话Chain.proceed()。
5. 允许重试并进行多次呼叫Chain.proceed()。

Network Interceptors
1. 能够对重定向和重试等中间响应进行操作。
2. 不调用缓存的响应来短路网络。
3. 观察数据，就像通过网络传输一样。
4. 访问Connection该请求。