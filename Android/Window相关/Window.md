### 什么是Window和WindowManager
Window是Android中的一种抽象概念，它代表了一个窗口的概念，可以理解为一个透明的容器，可以放置各种视图组件，如按钮、文本框等。WindowManager是用于管理Window的类，它提供了一些方法来创建、更新和删除Window。

### Window和View的关系
Window和View是Android中两个非常重要的概念，它们之间的关系可以理解为Window是View的容器，View是Window的内容。Window可以包含一个或多个View，这些View可以用来显示各种UI组件。WindowManager负责管理这些Window和View，包括它们的创建、更新和删除。

### Window的创建和显示
Window的创建和显示可以通过WindowManager来实现，具体步骤如下：

1. 创建一个WindowManager对象，可以通过Context的getSystemService方法获取WindowManager对象。
2. 创建一个LayoutParams对象，用于设置Window的属性，如宽度、高度、位置等。
3. 创建一个View对象，作为Window的内容。
4. 调用WindowManager的addView方法，将View添加到Window中。

### WindowManager的常用方法
1. updateViewLayout方法，更新Window的属性。
2. removeView方法，删除Window。
3. addView方法，添加View到Window中。

### Window的层级
Window的层级是指Window在屏幕上的显示顺序，可以通过LayoutParams的type属性来设置Window的层级。Window的种类分为3种：
1. 应用Window，用于显示应用程序的UI。层级范围 1~99
2. 子Window，用于显示在应用Window之上的UI。依附于应用Window,例如dialog、popupwindow。层级范围1000~1999
3. 系统Window，用于显示系统级别的UI，如状态栏、导航栏等。层级范围2000~2999，使用时需要声明SYSTEM_ALERT_WINDOW权限

### Window的添加过程
1. WindowManager.addView()
2. WindowManagerImpl.addView()
3. WindowManagerGlobal.addView()
4. ViewRootImpl.setView()
5. WindowSession.addToDisplay() IPC调用WindowSession
6. WindowManagerService.addToDisplay()
7. WindowManagerService通过IPC机制将Window的信息发送给SurfaceFlinger服务。
8. SurfaceFlinger服务根据Window的信息创建一个Surface，并将其添加到SurfaceFlinger服务中。
9. SurfaceFlinger服务将Surface绘制到屏幕上，从而显示Window的内容。

### Window的删除过程
1. WindowManager.removeView()
2. WindowManagerImpl.removeView()
3. WindowManagerGlobal.removeView()
4. ViewRootImpl.doDie()
5. ViewRootImpl.dispatchDetachedFromWindow()
6. ViewRootImpl.destroyHardwareRenderer()
7. ViewRootImpl.destroy()
8. WindowSession.remove() IPC调用WindowSession
9. WindowManagerService.remove()
10. WindowManagerService通过IPC机制将Window的信息发送给SurfaceFlinger服务。
11. SurfaceFlinger服务根据Window的信息删除对应的Surface，并将其从SurfaceFlinger服务中移除。

### Window的更新过程
1. WindowManager.updateViewLayout()
2. WindowManagerImpl.updateViewLayout()
3. WindowManagerGlobal.updateViewLayout()
4. ViewRootImpl.setLayoutParams()
5. ViewRootImpl.scheduleTraversals()
6. ViewRootImpl.performTraversals()
7. ViewRootImpl.relayoutWindow()
8. WindowSession.relayout() IPC调用WindowSession
9. WindowManagerService.relayout()
10. WindowManagerService通过IPC机制将Window的信息发送给SurfaceFlinger服务。
11. SurfaceFlinger服务根据Window的信息更新对应的Surface，并将其绘制到屏幕上。

### 总结
Window的创建是通过WindowManager来实现的，而WindowManager则通过WindowManagerGlobal来实际管理Window，而实际的添加、更新和删除操作是通过ViewRootImpl来完成的。ViewRootImpl通过IPC机制与WindowManagerService进行通信，从而实现Window的添加、更新和删除。