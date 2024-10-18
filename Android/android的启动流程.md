### Android系统的启动流程

1. 引导程序（Bootloader）启动
2. 加载Linux内核 加载硬件驱动
3. 寻找init.rc文件
4. 启动init进程
5. 启动Zygote进程 启动虚拟机 注册JNI方法 预加载资源
6. 启动SystemServer进程
7. 启动Launcher进程

#### 系统服务是怎么启动的

系统服务怎么发布，让应用程序可见？

把系统服务binder注册到ServiceManager中


#### 系统服务跑在什么线程？

DisplayThread 负责显示
FgThread 前台任务
loThread IO任务
UiThread 值得一提的是，这个是子线程
binder线程


问题：

为什么系统服务不都跑在binder线程里呢？
为什么系统服务不都跑在自己私有的工作线程里呢？
跑在binder线程和跑在工作线程，如何取舍？


#### 怎么解决系统服务启动的相互依赖

分批启动

先启动基础服务，AMS PMS PKMS
后启动上层的一些服务


分阶段启动


#### 桌面的启动

AMS就绪之后，会调用SystemReady函数，而其内部又调用了STARThomeActivity函数
PMS查询已安装的应用，并显示在桌面上
点击应用之后就会启动应用的LauncherActivity


### 系统服务和bind的应用服务有什么区别?

1. 系统服务大多数运行在SystemServer中，如AMS WMS PMS
2. 应用服务：最终会调到AMS中，AMS负责Service的管理和调度，servcie启动和加载由应用端负责

注册方式上的区别

系统服务：binder实体对象注册到ServiceManager中，只有系统服务才能注册到ServiceManager中
应用服务：与系统服务不同，应用服务需要应用bindService，而且需要AMS去请求它

<img src='https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d5a09a8c55724996922950623bb1cf57~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp'/>



本文参考 [系统服务及应用进程相关](https://juejin.cn/post/6936585356791775263)