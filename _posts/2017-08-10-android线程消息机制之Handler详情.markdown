---
layout: post
title: android线程消息机制之Handler详情
date: 2017-08-10 11:42:24.000000000 +09:00
categories: android
tags: 源码分析|源码|handler|Looper|消息机制
---
android线程消息机制主要由Handler,Looper,Message和MessageQuene四个部分组成。平常在开发中，我们常用来在子线程中通知主线程来更新，其实整个安卓生命周期的驱动都是通过Handler(ActivityThread.H)来实现的。

首先我们先介绍这四个类的作用：

**Handler**：消息的发送者。负责将Message消息发送到MessageQueue中。以及通过Runnable,Callback或者handleMessage()来实现消息的回调处理

**Looper**：是消息的循环处理器，它负责从MessageQueue中取出Message对象进行处理。(Looper含有MessageQueue的引用)

**Message**：是消息载体，通过target来指向handler的引用。通过object来包含业务逻辑数据。其中MessagePool为消息池，用于回收空闲的Message对象的。

**MessageQueue**：消息队列，负责维护待处理的消息对象。


![image](http://img.blog.csdn.net/20150801014511416)

通过上面的图，我们可以比较清楚地知道他们的作用以及关系。接下来，我们从源码角度来分析这种关系是如何建立的。


```
public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
```

hander的其它构造方法可以自己去查看，通过这个构造方法，我们知道，handler持有MessageQueue的引用。所以可以方便地将Message加入到队列中去。

通过源码我们发现，sendMessage->sendMessageDelayed->sendMessageAtTime->enqueueMessage

```
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
```
都是通过enqueueMessage将message将加入到MessageQueue中。

接下来，我们看Message是如何构造的。通过Message的构造方法。


```
public static Message obtain() {
    synchronized (sPoolSync) {
        if (sPool != null) {
            Message m = sPool;
            sPool = m.next;
            m.next = null;
            m.flags = 0; // clear in-use flag
            sPoolSize--;
            return m;
        }
    }
    return new Message();
}
```
我们看到，Message是通过obtain的静态方法从消息池sPool中拿到的。这样可以做到消息的复用。


```
public static Message obtain(Handler h) {
    Message m = obtain();
    m.target = h;

    return m;
}
```
其中有一个重载方法中m.target = h;这段代码非常重要，便于后面找到消息的目标handler进行处理。

接下来，我们来看Looper。我们知道Looper通过过Looper.loop来进入循环的，而循环是通过线程的run方法的驱动的。

首先我们知道，我们在创建Handler的时候，都没有去创建Looper，那么Looper哪里来的呢？




```
public Handler(Callback callback, boolean async) {
        ...
        mLooper = Looper.myLooper();
        if (mLooper == null) {
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
    }
```
再看看Looper.myLooper()


```
    public static @Nullable Looper myLooper() {
        return sThreadLocal.get();
    }
```
ThreadLocal是线程创建线程局部变量的类。表示此变量只属于当前线程。


```
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }
```
我们看到了sThreadLocal.get()的方法实际是取当前线程中的Looper对象。

那么我们主线程的Looper到底在哪里创建的呢？
而我们清楚地知道，如果在子线程中创建handler调用，则需要使用Looper.prepare方法。


```
    private static void prepare(boolean quitAllowed) {
        if (sThreadLocal.get() != null) {
            throw new RuntimeException("Only one Looper may be created per thread");
        }
        sThreadLocal.set(new Looper(quitAllowed));
    }
```
我们看到此方法中，如果此线程中没有Looper对象，则创建一个Looper对象。接下来我们在源码中看到一个熟悉的方法。


```
    public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
    }
```

此方法单独的创建了一个sMainLooper用于主线程的Looper。这个prepareMainLooper到底在哪里调用呢？

高过引用指向发现，我们在ActivityThread.main()方法中发现

```
    public static void main(String[] args) {
        ...
        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```
而ActivityThread.main()是程序的入口方法。这样我们就非常清楚了，主线程的Looper在程序的启动过程中就已经创建并循环。

那么如果在子线程中创建Looper该如何正确调用呢？


```
class LooperThread extends Thread {
      public Handler mHandler;

      public void run() {
          Looper.prepare();

          mHandler = new Handler() {
              public void handleMessage(Message msg) {
                  // process incoming messages here
              }
          };

          Looper.loop();
      }
  }

```

接下来，我们需要看下Looper.loop()的执行方法


```
public static void loop() {
        final Looper me = myLooper();//拿到当前线程的looper
        if (me == null) {
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;//拿到当前looper的消息队列

        // Make sure the identity of this thread is that of the local process,
        // and keep track of what that identity token actually is.
        Binder.clearCallingIdentity();
        final long ident = Binder.clearCallingIdentity();

        for (;;) {//死循环遍历消息体。如果为null，则休眠。
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

            final long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;

            final long traceTag = me.mTraceTag;
            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }
            final long start = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            final long end;
            try {
                msg.target.dispatchMessage(msg);//此处是真正的分发消息。此处的target即是handler对象
                end = (slowDispatchThresholdMs == 0) ? 0 : SystemClock.uptimeMillis();
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (slowDispatchThresholdMs > 0) {
                final long time = end - start;
                if (time > slowDispatchThresholdMs) {
                    Slog.w(TAG, "Dispatch took " + time + "ms on "
                            + Thread.currentThread().getName() + ", h=" +
                            msg.target + " cb=" + msg.callback + " msg=" + msg.what);
                }
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
    }
```
最后我们看下dispatchMessage的处理方法。


```
    public void dispatchMessage(Message msg) {
        if (msg.callback != null) {
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            handleMessage(msg);
        }
    }
```
我们看到，dispatchMessage是优化处理msg.callback,然后就是实现的Callback接口，最后才是handleMessage方法。




**重点说明：**

1、handler在实例化的时候，持有Looper的引用。是通过ThreadLocal<Looper>与Handler进行关联的。

2、Message在实例化的过程中，通过target 持有Handler的引用。

3、通常一个线程对应一个Looper.一个Looper可以属于多个Handler.

