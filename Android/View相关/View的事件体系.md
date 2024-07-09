### 触摸事件是如何传递到具体View的?
1. ViewRootImpl.mInputEventReceiver -> WindowInputEventReceiver.onInputEvent()
2. ViewRootImpl.enqueueInputEvent() -> doProcessInputEvents() -> deliverInputEvent()
3. ViewPostImeInputStage.deliver -> onProcess() -> processPointerEvent()
4. DecorView.dispatchPointerEvent() -> dispatchTouchEvent()
5. Activity.dispatchTouchEvent() -> DecorView.superDispatchTouchEvent()
6. ViewGroup.dispatchTouchEvent()

[代码流程](https://www.jianshu.com/p/428ac114d83d)
<img src="https://upload-images.jianshu.io/upload_images/1728460-527fb18a35249e14.png?imageMogr2/auto-orient/strip|imageView2/2/format/webp">

### 事件分发的流程
1. dispatchTouchEvent()
2. onInterceptTouchEvent()
3. onTouchEvent()

### 事件分发结论
1. 同一个事件序列是指从手指接触屏幕的那一刻起，到手指离开屏幕的那一刻结束，在这个过程中所产生的一系列事件，这个事件序列以down事件开始，中间包含若干个move事件，最终以up事件结束。
2. 正常情况下，一个事件序列只能被一个View拦截且消耗。因为一旦一个元素拦截了某个事件，那么同一个事件序列内的所有事件都会直接交给这个元素处理，因此同一个事件序列中的事件不能分别由两个View同时处理，即一旦一个元素拦截了某个事件，那么同一个事件序列中该元素后边的元素将接收不到事件。
3. 某个View一旦开始处理事件，如果它不消耗ACTION_DOWN事件（onTouchEvent返回了false），那么同一事件序列中的其他事件都不会再交给它来处理，并且事件将重新交给它的父元素去处理，即父元素会收到DOWN、MOVE、UP事件。
4. 如果View不消耗除ACTION_DOWN以外的其他事件，那么这个点击事件会消失，此时父元素的onTouchEvent并不会执行，并且当前View可以继续接收到后续的事件，最终这些消失的点击事件会传递给Activity处理。
5. ViewGroup默认不拦截任何事件。Android源码中ViewGroup的onInterceptTouchEvent方法默认返回false。
6. View没有onInterceptTouchEvent方法，一旦有点击事件传递给它，那么它的onTouchEvent方法就会被调用。
7. View的onTouchEvent默认都会消耗事件（返回true），除非它是不可点击的（clickable和longClickable同时为false）。View的longClickable默认为false，clickable需要设置。
8. View的enable属性不影响onTouchEvent的默认返回值。哪怕一个View是disable状态，只要它的clickable或者longClickable有一个为true，那么它的onTouchEvent就会返回true。
9. 事件传递过程是由父控件向子控件传递的，可以通过requestDisallowInterceptTouchEvent方法干预事件传递过程，但是ACTION_DOWN事件一旦传递到某个子控件之后，就再也不能阻止父控件拦截该事件了。
10. 如果View的longClickable为true，并且它长按事件被触发，那么该View的onTouchEvent方法会返回true，后续事件（ACTION_MOVE、ACTION_UP）将不会在传递给父控件，并且父控件的onInterceptTouchEvent方法也不会再被调用。
11. 某个View一旦决定拦截，那么这一个事件序列都只能由它来处理（如果时间序列能够传递给它的话），并且它的onInterceptTouchEvent方法不会再被调用。

### onClickListener和onTouchListener的优先级

1. onTouchListener设置后不会再触发onTouchEvent
2. onTouchEvent处理了点击事件如果onTouchListener不处理点击事件，那么就不会有点击事件了。
<img src="https://upload-images.jianshu.io/upload_images/5811423-685b570505487ce2.png?imageMogr2/auto-orient/strip|imageView2/2/w/823/format/webp">


### 有A、B两个Button，各自设置了OnClickListener，在A上按下，移动到B上抬起，会触发OnClickListener么？为何?UP事件谁接收到了？OnClickListener触发的条件是哪些？

Android 的点击事件识别依赖于 DOWN 和 UP 事件都发生在同一个视图上。这是为了避免误触和提供一致的用户体验。如果仅仅因为用户的手指在移动过程中经过了另一个视图就触发点击事件，那么用户界面的行为将变得难以预测和控制。

OnClickListener触发的条件是哪些？
- DOWN 事件发生在目标视图上。
- 紧接着的 UP 事件也发生在同一目标视图上。
- DOWN 和 UP 事件之间的时间差没有超过系统定义的点击时间阈值（这个值可以通过 ViewConfiguration.getTapTimeout() 获取，但通常不需要开发者直接关心）。
- 没有其他触摸事件（如滑动、长按等）在这个时间窗口内中断 DOWN-UP 序列。

### MotionEvent.ACTION_CANCEL怎么触发
- 父视图拦截事件：
    当父视图（ViewGroup）通过onInterceptTouchEvent()方法返回true时，表示父视图要拦截触摸事件。此时，系统会向被拦截的子视图发送一个ACTION_CANCEL事件，以通知子视图其触摸事件已被取消，子视图将不会再收到后续的事件（如ACTION_MOVE或ACTION_UP）。这是ACTION_CANCEL最常见的一种触发场景。
- 系统需要取消当前手势：
    在某些情况下，系统可能出于内部管理的需要，决定取消当前的触摸手势。这时，也会向相关的视图发送ACTION_CANCEL事件。虽然这种情况不如父视图拦截事件常见，但在设计复杂的用户界面和交互逻辑时，仍然需要考虑这种情况。
- 触摸超出正常UI边界：
    在一些特殊情况下，如用户的触摸操作超出了应用界面的正常边界，系统也可能会触发ACTION_CANCEL事件。这有助于应用及时响应触摸操作的变化，避免不必要的资源消耗和界面混乱。
- 其他系统级因素：
    除了上述几种情况外，还可能存在其他系统级因素导致ACTION_CANCEL事件的触发。这些因素可能涉及到系统的底层实现和性能优化等方面，对于应用开发者来说，通常不需要深入了解这些内部机制。