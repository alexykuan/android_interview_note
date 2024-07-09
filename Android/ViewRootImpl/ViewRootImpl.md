### 何时创建ViewRootImpl

ViewRootImpl是在ActivityThread的handleResumeActivity方法中创建的。具体来说，当Activity的onResume方法被调用时，handleResumeActivity方法会被调用，其中会创建ViewRootImpl对象。
1. ActivityThread.handleResumeActivity
2. WindowManagerGlobal.addView
3. ViewRootImpl.setView

### ViewRootImpl的作用

视图树的测量、布局和绘制：
当应用需要更新界面时（例如，窗口大小变化、添加或删除视图等），ViewRootImpl 会触发视图树的测量和布局过程，以确保所有视图都正确地放置在屏幕上。
随后，ViewRootImpl 会负责绘制视图树，将视图内容渲染到屏幕上。
事件分发：
ViewRootImpl 接收来自 WindowManager 的输入事件（如按键、触摸等），并将这些事件分发到正确的视图上进行处理。
它还负责处理触摸事件的手势识别（如滑动、缩放等），并可能将这些事件转发给更高级别的处理器（如滑动菜单、滑动视图等）。
窗口管理：
ViewRootImpl 与 WindowManager 紧密合作，管理窗口的创建、更新和销毁。
它负责向 WindowManager 发送请求，以调整窗口的属性（如大小、位置、透明度等）。
同步和动画：
ViewRootImpl 还负责处理视图树的同步和动画效果。它确保在适当的时间更新视图的状态，并在需要时应用动画效果。

### 参考
[工作流程](https://www.jianshu.com/p/1221d7afa806)