### 什么是Chereographer

    Choreographer是一个用于协调动画、输入事件、绘制等操作的类。它接收来自系统的定时信号(例如垂直同步信号vertical synchronization),然后规划即将显示到屏幕上的下一帧的工作。
    应用层一般不会和Choreographer直接交互，而是通过更高层的动画Framework来间接交互。

> 下面是一些Choreographer常见的用途  

- 使用`android.animation.ValueAniamtor`来进行属性动画
- 使用`View.postOnAniamtion(Runnable)`在下一帧显示前执行Runnable
- 使用`View.postOnAniamtionDelayed(Runnable)`在指定延时计算出下一帧，并在下一帧显示前执行Runnable
- 使用`View.postInvalidateOnAnimation()`在下一次动画时间或者下一次垂直同步信号到来时进行重绘
- 在视图滚动时候为了保证滚动的流畅性，通常不需要额外进行操作，系统会在合适的时间调用`View.onDraw()`
