### 什么是ANR和OOM
- ANR：Application Not Responding，应用程序无响应，Android系统对于某些系统服务有超时限制，如果应用程序在规定的时间内没有响应，系统就会弹出ANR对话框，提示用户当前应用程序无响应，并询问用户是否等待应用程序响应，如果用户选择等待，系统会继续等待应用程序响应，如果用户选择取消，系统会杀死应用程序进程。

1. 主线程频繁进行耗时的IO操作：如数据库读写
2. 多线程操作的死锁，主线程被block；
3. 主线程被Binder 对端block；
4. System Server中WatchDog出现ANR；
5. service binder的连接达到上线无法和和System Server通信
6. 系统资源已耗尽（管道、CPU、IO）
7. 用户输入事件没有在5秒内得到处理

- OOM：Out Of Memory，内存溢出，当应用程序在运行过程中，申请的内存超过了系统分配给应用程序的最大内存，就会发生内存溢出，导致应用程序崩溃。

1. 发生内存泄露
2. 系统内存不足
3. 应用程序申请的内存过大

### 如何定位ANR

1. 获取ANR日志 `$adb pull data/anr/traces.txt `
  
   确定ANR类型：从logcat中找到ANR类型，例如InputDispatching Timeout、Broadcast Timeout、Service Timeout和死锁Block Waitting等。

2. 使用Traceview定位耗时操作或者通过Android Profile查看方法执行时间
 
    在代码中插入 Debug.startMethodTracing() 和 Debug.stopMethodTracing()，生成 .trace 文件。
    通过 Android Studio 的 ​​Profiler​​ 查看方法调用耗时。

3. 使用SysTrace定位耗时操作