### Activitity常见生命周期函数
1. onCreate()：在Activity创建时调用，用于初始化Activity的布局、成员变量等。
2. onStart()：在Activity可见时调用，用于开始Activity的交互。
3. onResume()：在Activity可交互时调用，用于恢复Activity的状态。
4. onPause()：在Activity即将失去焦点时调用，用于保存Activity的状态。
5. onStop()：在Activity不可见时调用，用于停止Activity的交互。
6. onDestroy()：在Activity销毁时调用，用于释放Activity的资源。

### 通常情况下的生命周期函数
有Activity A启动Activity B，Activity A的生命周期函数调用顺序如下：
1. Activity A：onPause()
2. Activity A：onStop()
3. Activity B：onCreate()
4. Activity B：onStart()
5. Activity B：onResume()
6. Activity A：onDestroy()

### 异常情况下的生命周期调用
- 屏幕旋转情况下Activity会重新创建，生命周期函数调用顺序如下：
1. Activity A：onPause()
2. Activity A：onStop() -> onSaveInstanceState() 保存数据
3. Activity A：onDestroy()
4. Activity A：onCreate() -> onRestoreInstanceState() 恢复数据
5. Activity A：onStart()
6. Activity A：onResume()


### 为什么页面停止后10s左右才会调用`onDestroy()`
`onDestroy()`通过IdleHandler调用。当Activity停止后，系统会通过IdleHandler来延迟调用`onDestroy()`，以防止在Activity停止后立即销毁，导致一些操作无法完成。如果Handler中一直存在消息，那么IdleHandler将一直无法调用，因此`onDestroy()`也会一直无法调用。finish调用后有10s的超时时间来确保`onDestroy()`被调用。
> ActivityStackSupervisor
```java
/** How long we wait until giving up on the last activity telling us it is idle. */
private static final int IDLE_TIMEOUT = 10 * 1000;
```
processStoppingAndFinishingActivities -> processStoppingAndFinishingActivities -> ActivityRecord.destroyImmediately