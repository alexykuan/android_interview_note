### Handler的作用

Handler是Android中用于处理异步消息的类，它允许我们在不同的线程之间传递消息和Runnable对象，从而实现线程间的通信。
Handler通常与Looper和MessageQueue一起使用，其中Looper负责管理MessageQueue，而Handler则负责向MessageQueue发送消息和Runnable对象。

### Handler的使用场景

Handler通常用于以下场景：

1. 在子线程中更新UI：在Android中，只有主线程才能更新UI，因此我们需要使用Handler将更新UI的操作放到主线程中执行。
2. 处理定时任务：Handler可以用来处理定时任务，例如定时发送消息或Runnable对象。
3. 线程间通信：Handler可以用来在不同的线程之间传递消息和Runnable对象，从而实现线程间的通信。

### Looper

#### Looper的作用

在创建Handler的时候，我们可能需要传入一个Looper对象，Looper的作用是管理MessageQueue，一个线程对应一个Looper和MessageQueue。通过`Looper.myLooper()`可以获取到当前线程的Looper对象，通过`Looper.prepare()`可以创建一个Looper对象，通过`Looper.loop()`可以开始处理MessageQueue中的消息。

#### Looper的创建过程

- Looper通过`Looper.prepare()`方法创建，new出来的Looper直接设置到了`sThreadLocal`里面。
- `sThreadLocal`里面是一个`map`数据结构。`key`是`当前Thread`，`value`是`Looper`。

因此`Looper.prepare()`不能多次调用，因为一个线程只允许创建一次Looper对象。后面通过`Looper.myLooper()`直接获取。

由于sThreadLocal持有当前线程的引用，而主线程在运行时一直存在，所以主线程的Looper是一直存在的，Looper也会一直持有`MessageQueue`和`Message`的引用，而`Handler`持有`MessageQueue`引用，而Handler的内部类的创建方式会持有外部类的引用，所以如果在activity中使用内部类的方式创建handler会造成`activity`的内存泄露，在activity需要销毁的时候无法完成销毁。因此在activity销毁的时候需要调用`Handler.removeCallbacksAndMessages()`,`removeCallbacksAndMessages`会移除handler发送的所有`Message`，切断`Handler`和`MessageQueue`和`Looper`的引用关系，从而阻止内存泄露。

```java
private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```

#### Looper的循环过程

Looper通过`Looper.loop()`方法开始循环处理MessageQueue中的消息，它会不断地从MessageQueue中取出消息，然后调用Handler的`dispatchMessage()`方法来处理消息。

```java
for (;;) {
            Message msg = queue.next(); // might block
            if (msg == null) {
                // No message indicates that the message queue is quitting.
                return;
            }

            // This must be in a local variable, in case a UI event sets the logger
            final Printer logging = me.mLogging;
            if (logging != null) {
                logging.println(">>>>> Dispatching to " + msg.target + " " +
                        msg.callback + ": " + msg.what);
            }
            // Make sure the observer won't change while processing a transaction.
            final Observer observer = sObserver;

            final long traceTag = me.mTraceTag;
            long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
            long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
            if (thresholdOverride > 0) {
                slowDispatchThresholdMs = thresholdOverride;
                slowDeliveryThresholdMs = thresholdOverride;
            }
            final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
            final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);

            final boolean needStartTime = logSlowDelivery || logSlowDispatch;
            final boolean needEndTime = logSlowDispatch;

            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }

            final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
            final long dispatchEnd;
            Object token = null;
            if (observer != null) {
                token = observer.messageDispatchStarting();
            }
            long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
            try {
                msg.target.dispatchMessage(msg);
                if (observer != null) {
                    observer.messageDispatched(token, msg);
                }
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } catch (Exception exception) {
                if (observer != null) {
                    observer.dispatchingThrewException(token, msg, exception);
                }
                throw exception;
            } finally {
                ThreadLocalWorkSource.restore(origWorkSource);
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (logSlowDelivery) {
                if (slowDeliveryDetected) {
                    if ((dispatchStart - msg.when) <= 10) {
                        Slog.w(TAG, "Drained");
                        slowDeliveryDetected = false;
                    }
                } else {
                    if (showSlowLog(slowDeliveryThresholdMs, msg.when, dispatchStart, "delivery",
                            msg)) {
                        // Once we write a slow delivery log, suppress until the queue drains.
                        slowDeliveryDetected = true;
                    }
                }
            }
            if (logSlowDispatch) {
                showSlowLog(slowDispatchThresholdMs, dispatchStart, dispatchEnd, "dispatch", msg);
            }

            if (logging != null) {
                logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
            }

            // Make sure that during the course of dispatching the
            // identity of the thread wasn't corrupted.
            final long newIdent = Binder.clearCallingIdentity();
            if (ident != newIdent) {
                Log.wtf(TAG, "Thread identity changed from 0x"
                        + Long.toHexString(ident) + " to 0x"
                        + Long.toHexString(newIdent) + " while dispatching to "
                        + msg.target.getClass().getName() + " "
                        + msg.callback + " what=" + msg.what);
            }

            msg.recycleUnchecked();
        }
```

### MessageQueue

#### `MessageQueue.next()`

`MessageQueue`的`next()`方法用于从MessageQueue中取出下一个待处理的消息，如果没有消息，则阻塞当前线程，直到有消息到来。

```java
Message next() {
        // Return here if the message loop has already quit and been disposed.
        // This can happen if the application tries to restart a looper after quit
        // which is not supported.
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }

            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                    } else {
                        // Got a message.
                        mBlocked = false;
                        if (prevMsg != null) {
                            prevMsg.next = msg.next;
                        } else {
                            mMessages = msg.next;
                        }
                        msg.next = null;
                        if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                        msg.markInUse();
                        return msg;
                    }
                } else {
                    // No more messages.
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```

MeesageQueue.next方法中判断消息发送的时间使用的是`SystemClock.uptimeMillis()`所用的时间，调整手机时间不会影响`handler`的消息触发的时间。

#### MeesageQueue.postSyncBarrier()和MessageQueue.removeSyncBarrier()

`postSyncBarrier`用于向MessageQueue中插入一个同步屏障，当同步屏障存在的时候，同步的消息将不会被处理，只有异步消息不受影响。当`removeSyncBarrier`执行后同步屏障被移除，同步消息将得到执行。

这两个方法应该成对调用，避免同步消息一直得不到处理

#### MeesageQueue.nativePollOnce

`nativePollOnce`是`MessageQueue`的本地方法，用于等待消息的到来，当有消息到来时，该过程是阻塞的，`Looper`也是阻塞的。`nativePollOnce`会唤醒等待的线程，然后`MessageQueue`会调用`next`方法获取消息，并执行消息中的`runnable`或者`callback`。

#### MeesageQueue.nativePollOnce为何没有卡死主线程

`nativePollOnce`会阻塞等待消息的到来，阻塞会释放CPU资源。主线程Ui的处理依赖于Handler对于各种UI消息的处理。