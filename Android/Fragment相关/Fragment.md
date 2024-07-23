### Fragment是什么

- Fragment是Android的一个组件，可以添加到View中，区别于普通的View，它有自己独特的生命周期，它也有自己独特的返回栈。
- Fragment可以动态的添加或者移除
- Fragment不能独立存在，必须依附于Activity
- Fragment常用来动态地显示内容，例如弹窗的实现`DialogFragment`、`ViewPager`、`FrameLayout`显示的字内容等等。

### Fragment的生命周期

- onAttach()
- onCreate()
- onCreateView()
- onActivityCreated()
- onStart()
- onResume()
- onPause()
- onStop()
- onDestroyView()
- onDestroy()
- onDetach()

### Framgent使用的注意事项

- 尽量在onViewCreateView中进行相关的View的操作
- 尽量重写无参的构造函数
- 不要重复的添加Fragment否则会排除`IlleagleStateException`
- 避免一次性加载太多的不可见的`Fragment`
- 和`activity`共享`viewModel`

### 什么是Fragment的懒加载

懒加载就是当Fragment可见的时候才去加载Fragment的内容，当Fragment不可见的时候，不加载Fragment的内容，这样就可以避免一次性加载过多的不可见的Fragment，从而避免内存溢出的问题。

```java
class BaseLzFragment extends Fragment {
    protected boolean isPrepared = false;
    protected boolean isVisible = false;
    protected T data = null;

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        isPrepared = true;
        lazyLoad();
    }

    @Override
    public void setUserVisibleHint(boolean isVisibleToUser) {
        super.setUserVisibleHint(isVisibleToUser);
        if (getUserVisibleHint()) {
            isVisible = true;
            onVisible();
        } else {
            isVisible = false;
            onInvisible();
        }
    }

    protected void onVisible() {
        if (isPrepared && isVisible) {
            lazyLoad();
        }
    }

    protected void onInvisible() {
    }

    protected void lazyLoad() {
        /// data =
    }
}
```