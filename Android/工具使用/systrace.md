### 常见命令参数

#### 采集 10 秒数据，跟踪图形渲染、调度、Activity 管理

android 9 及以下

`python systrace.py --time=10 -o trace.html sched gfx view am res`

Perfetto 替代 Systrace（更强大的新一代工具）

`./record_android_trace -o trace.perfetto-trace -t 10s -b 32mb sched gfx input`

#### 跟踪应用冷启动过程（从点击图标到首帧渲染）

`python systrace.py --time=5 -o startup.html gfx view am res sched -a com.example.app`

#### 定位 ui 卡顿

`python systrace.py --time=5 -o ui.html gfx view sched freq -a com.example.app`

#### 代码添加 Trace 采集时添加 -a 参数过滤应用

```kotlin
fun trace() {
    Trace.beginSection("MyCustomOperation");
    // 需要跟踪的代码
    someCodeFragment();
    Trace.endSection();
}

```

1. 常用参数 ​​

| 参数              | 说明                                                      |
| ----------------- | --------------------------------------------------------- |
| --time=\<SECONDS> | 采集时长（默认 5 秒）                                     |
| -o <FILE>         | 输出文件路径（如 trace.html）                             |
| -a <PACKAGE_NAME> | 仅跟踪指定包名的应用（需应用启用 debuggable）             |
| --buf-size=<SIZE> | 缓冲区大小（默认 20MB，建议调整为 32mb 或 64mb 避免溢出） |
| --list-categories | 查看所有可用的跟踪类别                                    |

2. 常用跟踪类别

| 类别          | 说明                                         |
| ------------- | -------------------------------------------- |
| sched         | CPU 调度信息（核心线程调度分析）             |
| gfx           | 图形渲染（分析掉帧、渲染耗时）               |
| view          | View 系统（布局、测量、绘制）                |
| am            | Activity 管理（启动 Activity、生命周期事件） |
| res           | 资源加载（文件 I/O、锁竞争）                 |
| webview       | WebView 相关性能                             |
| input         | 触摸事件和输入延迟                           |
| binder_driver | Binder IPC 调用分析（跨进程通信瓶颈）        |
