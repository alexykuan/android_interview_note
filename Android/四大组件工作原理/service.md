### Service启动流程


<img src='https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4877f6b77fa04106b0994ef393d7a1fc~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp'>

1. context.startService()
2. ActivityManagerNative.getDefault().startService()
3. ApplicationThread.scheduleServiceArgs()
4. Handler H handleCreateService()
5. Service.onCreate()

### Service的绑定过程

* 首先应用向AMS发起bindService调用
* AMS检查是否有binder句柄
* 如果有，就会将binder句柄回调给应用
* 如果没有，AMS就会向Service请求句柄，Service会将句柄发布到AMS，然后AMS会将binder句柄回调给应用
* 应用在拿到binder句柄之后，就可以向Service发起binder调用了
* 应用端bindService的时候会带一个ServiceDispatcher IServiceConnection对象
* 这个对象是一个Bindder对象，把这个对象传到AMS之后，AMS就会将它保存起来
* 等到Service绑定成功了，AMS就会调用IserviceConnection的connected函数来主动通知应用端
* 然后IServiceConnection持有了ServiceConnection的引用，这意味着可以调用ServiceConnection的函数

值得注意的是，IServiceConnection和ServiceConnection不一定是一对一的关系

1. context.bindService(ServiceConnection)
2. ActivityManagerNative.getDefault().bindService(ServiceConnect)
3. ServiceDispatcher.dispatchBindService()


<img src='https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f3c1141059d4bf69b60124d2b245364~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp'>

### service的启动方式

1. startService 会调用onStartCommand方法
2. bindService

## 总结
四大组件的启动过程都可能涉及到跨进程通信，因此都需要AMS来协调完成，最后都会回到ApplicationThread来执行。