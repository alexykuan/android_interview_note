### 应用启动过程

应用进程是什么时候启动的

启动应用组件，如Activity、Service会先判断应用所在进程是否启动，如果没有启动，就会先启动进程


进程启动由谁发起

由AMS向zygote发起请求，通过socket进行通信


zygote fork出应用进程，执行ActivityThread的入口main函数，入口函数的java类名是AMS通过socket发给zygote的
进程启动之后向AMS报告，注册Application Thread整个启动才算结束，只有向AMS报告，进程才是可用的

### 应用启动流程

1. 点击桌面图标，启动一个桌面应用
2. Launcher进程通过Binder IPC向AMS发起startActivity请求
3. AMS接收到请求后，向zygote进程发起创建进程的请求
4. zygote进程fork出新的应用进程
5. 新的应用进程，通过Binder IPC向AMS报告进程已经创建完成
6. AMS向新进程发送要启动的Activity信息
7. 应用进程的binder线程收到AMS的请求后，通过Handler将启动Activity的信息发送给主线程

<img src='https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/93345dc5ba4a465c97922c16f1a6210a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp' />

#### getApplication和getApplicationContext有什么区别？

* 都是返回Applicaiton对象
* getApplicationContext是Context中的一个抽象函数，getApplication是Activity和Service特有的，在其他地方如广播中是不能使用的

#### Activity里的this和getBaseContext有什么区别？
* this返回Activity指向自己
* getBaseContext返回ContextWrapper中的mBase

#### 一个进程是怎么启用binder机制的

* 打开binder驱动
* 映射内存，分配缓冲区
* 注册binder线程
* 进入binder loop

[参考引用](https://juejin.cn/post/6936585356791775263#heading-3)
