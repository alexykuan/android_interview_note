### Activity的启动模式和作用简述
#### 1. `standard`

默认的启动模式，无论活动栈中是否有当前`activity`的实例，启动该`activity`都会创建新的实例
#### 2. `singleTop`
栈顶复用模式，如果当前启动`activity`的实例位于活动栈的栈顶，则复用当前`activity`的实例，并调用`onNewIntent()`方法，如果没有则正常启动`activity`放入活动栈栈顶。
#### 3. `singleTask`
栈内复用模式，如果当前活动栈中存在要启动`activity`的实例，则将当前`activity`上面的`activity`出栈，使当前`activity`位于栈顶。并调用`onNewIntent()`方法，如果没有则正常启动`activity`放入活动栈栈顶。
#### 4. `singleInstance`
- 独占一个任务栈，该任务栈中有且仅有该Activity实例
- 整个系统就只有一个实例。
- 设置singleIntance的Activity启动的其他Activity，默认的在包名的任务栈中，如果配合android:taskAffinity，也可以新的任务栈中创建的实例。
- 设置singleIntance的Activity启动的其他Activity，一定不在singleInstance的Activity所在的栈中，其他Activity的实例的创建和使用取决于该Activity设置的launchMode和android:taskAffinity

### taskAffinity的作用
- `taskAffinity`属性用于指定Activity希望归属的任务栈的名字，默认情况下，应用程序中所有Activity的`taskAffinity`属性都为应用程序的包名。
- `taskAffinity`属性主要和`singleTask`和`singleInstance`启动模式相关，它指明了Activity希望归属的任务栈。

### allowTaskReparenting
- `allowTaskReparenting`属性用于指定具有`taskAffinity`属性的Activity在当前应用的任务栈中运行时，是否允许重新分配到具有相同`taskAffinity`属性的任务栈中。

### activity manifest相关flag
- android:allowTaskReparenting:

       用来标记Activity能否从启动的Task移动到有着affinity的Task（当这个Task进入到前台时）,“true”，表示能移动，“false”，表示它必须呆在启动时呆在的那个Task里。 如果这个特性没有被设定，设定到<application>元素上的allowTaskReparenting特性的值会应用到Activity上。默认值为“false”。
        一般来说，当Activity启动后，它就与启动它的Task关联，并且在那里耗尽它的整个生命周期。当当前的Task不再显示时，你可以使用这个特性来强制Activity移动到有着affinity的Task中。典型用法是：把一个应用程序的Activity移到另一个应用程序的主Task中。 
      例如，如果 email中包含一个web页的链接，点击它就会启动一个Activity来显示这个页面。这个Activity是由Browser应用程序定义的，但是，现在它作为email Task的一部分。如果它重新宿主到Browser Task里，当Browser下一次进入到前台时，它就能被看见，并且，当email Task再次进入前台时，就看不到它了。 
       Actvity的affinity是由taskAffinity特性定义的。Task的affinity是通过读取根Activity的affinity决定。因此，根Activity总是位于相同affinity的Task里。由于启动模式为“singleTask”和“singleInstance”的Activity只能位于Task的底部，因此，重新宿主只能限于“standard”和“singleTop”模式。 

- android:alwaysRetainTaskState ：

       用来标记Activity所在的Task的状态是否总是由系统来保持。 “true”，表示总是；“false”，表示在某种情形下允许系统恢复Task到它的初始化状态。默认值是“false”。这个特性只针对Task的根Activity有意义；对其它Activity来说，忽略之。 
      一般来说，特定的情形如当用户从主画面重新选择这个Task时，系统会对这个Task进行清理（从stack中删除位于根Activity之上的所有Activivity）。典型的情况，当用户有一段时间没有访问这个Task时也会这么做，例如30分钟。 然而，当这个特性设为“true”时，用户总是能回到这个Task的最新状态，无论他们是如何启动的。这非常有用，例如，像Browser应用程序，这里有很多的状态（例如多个打开的Tab），用户不想丢失这些状态。

 
- android:clearTaskOnLaunch ：

        用来标记是否从Task中清除所有的Activity，除了根Activity外（每当从主画面重新启动时）“true”，表示总是清除至它的根Activity，“false”表示不。默认值是“false”。这个特性只对启动一个新的Task的Activity（根Activity）有意义；对Task中其它的Activity忽略。 
        当这个值为“true”，每次用户重新启动这个Task时，都会进入到它的根Activity中，不管这个Task最后在做些什么，也不管用户是使用BACK还是HOME离开的。当这个值为“false”时，可能会在一些情形下（参考alwaysRetainTaskState特性）清除Task的Activity，但不总是。 
       假设，某人从主画面启动了Activity P，并从那里迁移至Activity Q。接下来用户按下HOME，然后返回Activity P。一般，用户可能见到的是Activity Q，因为它是P的Task中最后工作的内容。然而，如果P设定这个特性为“true”，当用户按下HOME并使这个Task再次进入前台时，其上的所有的Activity(在这里是Q)都将被清除。因此，当返回到这个Task时，用户只能看到P。 
     如果这个特性和allowTaskReparenting都设定为“true”，那些能重新宿主的Activity会移动到共享affinity的Task中；剩下的Activity都将被抛弃，如上所述。

- android:finishOnTaskLaunch： 

     用来标记当用户再次启动它的Task（在主画面选择这个Task）时已经存在的Activity实例是否要关闭（结束）“true”，表示应该关闭，“false”表示不关闭。默认值是“false”。 如果这个特性和allowTaskReparenting都设定为“true”，这个特性胜出。Activity的affinity忽略。这个Activity不会重新宿主，但是会销毁。 

- android:noHistory： 

      用于标记当用户从Activity上离开并且它在屏幕上不再可见时Activity是否从Activity stack中清除并结束（调用finish()方法）——“true”，表示它应该关闭，“false”，表示不需要。默认值是“false”。“true”值意味着Activity不会留下历史痕迹。因为它不会在Activity stack的Task中保留，因此，用户不能返回它。比如启用界面的就可以借用这个。

### `Intent.FLAG_ACTIVITY_NEW_TASK`的作用

- 在非`activity`中启动`activity`

     需要添加该`flag`才能正常启动,否则会抛出异常导致应用闪退。因为在非`activity`中启动`activity`时，系统会认为这是一个新的任务栈(因为无法确定何时在哪个页面启动，所以重新开启一个任务栈)，而默认情况下，`activity`只能存在于任务栈中，所以需要添加`FLAG_ACTIVITY_NEW_TASK`标志来创建一个新的任务栈。

- 和`launchMode`的关系

|**启动模式** | **开启flag**|
| ---------------------- | ------------------------- |
|standard|启动新的任务和实例|
|singleTop|如果复用则忽略flag。否则启动新的任务和实例|
|singleTask|如果复用则忽略flag。否则启动新的任务和实例|
|singleInstance|flag无影响|

### Activity启动流程

1. Activity.startActivityForResult()
2. Instrumentation.execStartActivity
3. ActivityTaskManagerService.startActivity() -> startActivityAsUser()
4. ActivityStarter.execute() -> executeRequest() -> startActivityUnchecked() -> startActivityInner()
5. Task.startActivityLocked()
6. RootWindowContainer.resumeFocusedTasksTopActivities() -> resumeTopActivityInnerLocked()
7. TaskFragment.resumeTopActivity()
8. ActivityTaskSupervisor.startSpecificActivity() -> realStartActivityLocked()
9. ActivityThread.scheduleLaunchActivity()
10. ActivityThread.handleLaunchActivity()
11. ActivityThread.performLaunchActivity()
12. Activity.attach() -> attachBaseContext() -> ContextWrapper.attachBaseContext() -> ContextThemeWrapper.attachBaseContext()
13. Activity.onCreate()

> 参考流程图
<img src="https://img-blog.csdnimg.cn/375fec304aee4af1ba6a8a69f2be68be.png#pic_center">
<img src="https://img-blog.csdnimg.cn/b49c570a2a6144e6868538b05c0940d1.png#pic_center">

### Activity传输数据有什么限制

- 传输的数据不能超过1M，否则会抛出异常
- 传输的数据必须是基本数据类型或者string或者Parcelable对象，不能是自定义的对象

### Activity数据为什么不设置为setData


### Activity是什么时候显示出来的。

- ActivityThread.handleResumeActivity()
- ActivityThread.performResumeActivity()
- Activity.performResume()
- Instrumentation.callActivityOnResume()
- Activity.onResume()
- ViewRootImpl.setView()
- ViewRootImpl.requestLayout()
- Activity.makeVisible()