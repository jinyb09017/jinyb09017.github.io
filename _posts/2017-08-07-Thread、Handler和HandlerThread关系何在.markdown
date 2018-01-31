---
layout: post
title: Thread、Handler和HandlerThread关系何在
date: 2017-08-07 12:42:24.000000000 +09:00
categories: android
tags: 源码分析|源码
---


HandlerThread看名字，确实比较奇怪。到底是handler还是thread.其实看过源码后，就会非常清楚。

HandlerThread 继承自thread。所以本质上是一个线程，内部有Looper和Handler引用。它和AsyncTask非常像，都是google为了方便开发者，封装的工具类。HandlerThread可以让你不用维护Looper来实现线程的消息通知机制。

这个类非常简单，我们用下源码并可以得知。


```
public class HandlerThread extends Thread {
    int mPriority;
    int mTid = -1;
    Looper mLooper;
    private @Nullable Handler mHandler;

    public HandlerThread(String name) {
        super(name);
        mPriority = Process.THREAD_PRIORITY_DEFAULT;
    }
    ...

}
```
可以清楚的看到，它继承自Thread.所以是一个线程。同时持有Looper和handler的引用。mPriority用于设置线程的优先级。

我们再看下线程的run方法

```
    public void run() {
        mTid = Process.myTid();
        Looper.prepare();
        synchronized (this) {
            mLooper = Looper.myLooper();
            notifyAll();
        }
        Process.setThreadPriority(mPriority);
        onLooperPrepared();
        Looper.loop();
        mTid = -1;
    }
```
我们看到，通过Looper.prepare();的静态方法调用 基于ThreadLocal最终后创建一个线程独有Looper。

同步代码块保证looper创建成功后，唤醒阻塞的队列。最后通过Looper.loop();来开启looper的循环机制。


```
    public Looper getLooper() {
        if (!isAlive()) {
            return null;
        }

        // If the thread has been started, wait until the looper has been created.
        synchronized (this) {
            while (isAlive() && mLooper == null) {
                try {
                    wait();
                } catch (InterruptedException e) {
                }
            }
        }
        return mLooper;
    }
```

getLooper返回Looper对象。其中同步代码块，如果looper为空，则等待唤醒。


```
    public Handler getThreadHandler() {
        if (mHandler == null) {
            mHandler = new Handler(getLooper());
        }
        return mHandler;
    }

```
getThreadHandler返回hander对象。

所以如果我们要自定义一套非主线程的消息通知机制，只需要HandlerThread的帮助就可以轻松实现。而不需要自己去创建looper对象了。

所以调用的时候，只需要开启一个HandlerThread线程。
```
private HandlerThread mHandlerThread;
......
mHandlerThread = new HandlerThread("HandlerThread");
handlerThread.start();
```

由于HandlerThread内部的handler即没有实现Handler.Callback方法，也没有重写handleMessage方法，所以只能通过msg.callback回调来实现后续处理。所以通常我们可以自定义一个自己的Handler来处理消息即可。


```
final Handler handler = new Handler(mHandlerThread.getLooper()){
            @Override
            public void handleMessage(Message msg) {
                System.out.println("收到消息");
            }
        };

```
其中handler一定要传入本线程的looper对象。这样我们就可以在handleMessage处理后续的业务问题。

最后我们可以在任何一个线程中通过handler来发送消息了。

```
handler.sendEmptyMessage(0);
```


最后一定不要忘了在onDestroy释放,避免内存泄漏


```
 @Override
    protected void onDestroy() {
        super.onDestroy();
        mHandlerThread.quit();
    }
```


