### ViewModel

ViewModel是一个用于管理和视图相关的数据类，当视图（Activity或Fragment）被销毁时，ViewModel会保留数据，以便在视图重建时使用。ViewModel被存放在ViewModelStore中。
ViewModel通过ViewModelProvider类创建的时候会`put`到ViewModelStore中。数据结构是`HashMap<String,ViewModel>`

当屏幕旋转的时候viewModel会保存在`NonConfigurationInstances`中，在`Activity`重建的时候会从`NonConfigurationInstances`中取出，然后`put`到`ViewModelStore`中。


### LiveData是什么

LiveData 是一种可观察的数据持有者类。与常规的可观察者不同，LiveData 具有生命周期感知能力，这意味着它尊重其他应用组件（如 Activity、Fragment 或 Service）的生命周期。这种能力使 LiveData 能够确保界面符合当前的生命周期状态。这有助于避免因界面组件的生命周期比 LiveData 存在时间更长而导致的问题，比如试图在不再存在的 Activity 上更新 UI。通常LiveData和ViewModel一起使用，ViewModel可以持有LiveData，并且LiveData可以通知观察者数据的变化，从而更新UI。

```kotlin
liveData.observe(lifecycleOwner) { data ->
    // update UI
}
```
将LifecycleOwner和observer绑定在一起 `LifecycleBoundObserver`
当数据放生改变的时候会调用`LifecycleBoundObserver`的`shouldBeActive()`方法来判断被绑定的LifecycleOwner是否处于活跃状态，如果处于活跃状态则调用`onChange()`方法，否则将`LifecycleBoundObserver`添加到`mPendingActivations`中，当LifecycleOwner处于活跃状态的时候会调用`mPendingActivations`中的`onChange()`方法。

### LifeCycleOwner是什么，起什么作用呢

LifecycleOwner是Lifecycle的持有者，LifecycleOwner的生命周期回调会调用LifecycleRegistry的`handleLifecycleEvent()`方法，从而改变LifecycleRegistry的`mState`，当`mState`改变的时候会调用`mObserverMap`中的`LifecycleObserver`的`onStateChanged()`方法，从而通知观察者Lifecycle的状态改变。

### LiveData的原理

- `LiveData.value` 只能在主线程更新LiveData的值
- `LiveData.postValue` 可以在非主线程更新LiveData的值，内部会使用Handler切换到主线程更新到LiveData

`LiveData.observer(LifeCycleOwner owner, Observer observer)`，第一个参数是LiveData的持有者的生命周期Owner,第二个参数是LiveData的观察者，当LiveData的值有更新的时候会调用`observer.onChange`方法。

`ComponentActivity`和`androidx.fragment.app.Fragment`默认实现了`LifeCycleOwner`，在`activity`和`fragment`生命周期回调的时候，调用`LifeCycleRegistry`的`handleLifeCycleEvent()`方法。

```java
public enum State {
        /**
         * Destroyed state for a LifecycleOwner. After this event, this Lifecycle will not dispatch
         * any more events. For instance, for an {@link android.app.Activity}, this state is reached
         * <b>right before</b> Activity's {@link android.app.Activity#onDestroy() onDestroy} call.
         */
        DESTROYED,

        /**
         * Initialized state for a LifecycleOwner. For an {@link android.app.Activity}, this is
         * the state when it is constructed but has not received
         * {@link android.app.Activity#onCreate(android.os.Bundle) onCreate} yet.
         */
        INITIALIZED,

        /**
         * Created state for a LifecycleOwner. For an {@link android.app.Activity}, this state
         * is reached in two cases:
         * <ul>
         *     <li>after {@link android.app.Activity#onCreate(android.os.Bundle) onCreate} call;
         *     <li><b>right before</b> {@link android.app.Activity#onStop() onStop} call.
         * </ul>
         */
        CREATED,

        /**
         * Started state for a LifecycleOwner. For an {@link android.app.Activity}, this state
         * is reached in two cases:
         * <ul>
         *     <li>after {@link android.app.Activity#onStart() onStart} call;
         *     <li><b>right before</b> {@link android.app.Activity#onPause() onPause} call.
         * </ul>
         */
        STARTED,

        /**
         * Resumed state for a LifecycleOwner. For an {@link android.app.Activity}, this state
         * is reached after {@link android.app.Activity#onResume() onResume} is called.
         */
        RESUMED;

        /**
         * Compares if this State is greater or equal to the given {@code state}.
         *
         * @param state State to compare with
         * @return true if this State is greater or equal to the given {@code state}
         */
        public boolean isAtLeast(@NonNull State state) {
            return compareTo(state) >= 0;
        }
    }
```

### ViewModel.setValue多次调用会收到多次的值，ViewModel.postValue多次调用只会有一次收到。

`ViewModel.postValue`中如果值还在发送，那么后续的发送都会取消，直到postValue完成。

```java
protected void postValue(T value) {
        boolean postTask;
        synchronized (mDataLock) {
            postTask = mPendingData == NOT_SET;
            mPendingData = value;
        }
        if (!postTask) {
            return;
        }
        ArchTaskExecutor.getInstance().postToMainThread(mPostValueRunnable);
    }
```

`LifeCycleRegistry`管理`LifeCycleOwner`和`LifeCycleObserver`当`FragmentActivity`或者`Fragment`生命周期变化的时候通知`LifeCycleObserver`,通过`LifeCycleOwner`可以获取到当前的`LifeCycle`。
