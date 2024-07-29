### 什么是Chereographer

    Choreographer是一个用于协调动画、输入事件、绘制等操作的类。它接收来自系统的定时信号(例如垂直同步信号vertical synchronization),然后规划即将显示到屏幕上的下一帧的工作。
    应用层一般不会和Choreographer直接交互，而是通过更高层的动画Framework来间接交互。

> 下面是一些Choreographer常见的用途  

- 使用`android.animation.ValueAniamtor`来进行属性动画
- 使用`View.postOnAniamtion(Runnable)`在下一帧显示前执行Runnable
- 使用`View.postOnAniamtionDelayed(Runnable)`在指定延时计算出下一帧，并在下一帧显示前执行Runnable
- 使用`View.postInvalidateOnAnimation()`在下一次动画时间或者下一次垂直同步信号到来时进行重绘
- 在视图滚动时候为了保证滚动的流畅性，通常不需要额外进行操作，系统会在合适的时间调用`View.onDraw()`

> 下面是直接使用Choreographer的例子

如果你的应用需要再不同的线程中显示ui，可能是GL，可能不打算使用动画框架、View。只是为了确保和显示相协调。就使用postFrameCallback
```java
Choreographer.getInstance().postFrameCallback(new Choreographer.FrameCallback() {
    @Override
    public void doFrame(long frameTimeNanos) {
        // 在下一帧显示前执行
    }
});
```
每个Looper线程都有自己的`Chereographer`。其它线程可以post callbacks到chereographer，但是这些callbacks会在chereographer所在的线程上运行。

### Choreographer的原理

    Choreographer通过VSync信号来驱动UI的绘制。在Android中，VSync信号是由硬件提供的，每秒60次。Choreographer会在收到VSync信号后，调用`doFrame()`方法来执行UI的绘制。

### ViewRootImpl和Choreographer的协作
    
    ViewRootImpl作为PhoneWindow和WindowManager沟通的桥梁。在windows显示出来的时候会调用ViewRoootImpl的setView方法，而setView方法中会创建一个Choreographer对象，并且注册一个FrameCallback，这个FrameCallback会在下一帧执行callback的runnable方法，而runnable方法会调用doTraversals()方法，而doTraversals()方法会调用performTraversals()方法，而performTraversals()方法会调用performDraw()方法，而performDraw()方法会调用View的draw方法，从而完成UI的绘制。
