---
layout: post
title: AsyncTask源码解析
date: 2017-08-07 12:42:24.000000000 +09:00
categories: android
tags: 源码分析|源码
---


AsyncTask,是android提供的轻量级的异步类。本质上还是基于Thread和消息机制（handler）的封装。

首先我们先看一下，通常AsyncTask的用法。首先，AsyncTask是一个抽象类，需要实现doInBackground方法。


```
private class MyTask extends AsyncTask<String, Integer, String> {
    //onPreExecute方法用于在执行后台任务前做一些UI操作
    @Override
    protected void onPreExecute() {
        Log.i(TAG, "onPreExecute");
        textView.setText("loading...");
    }

    //doInBackground方法内部执行后台任务,不可在此方法内修改UI（在单独的线程里面处理任务）
    @Override
    protected String doInBackground(String... params) {
        Log.i(TAG, "doInBackground");
        ...
        return "FAIL";
    }

    //onProgressUpdate方法用于更新进度信息(此方法通过在doInBackground内调用publishProgress触发。)
    @Override
    protected void onProgressUpdate(Integer... progresses) {
        Log.i(TAG, "onProgressUpdate");
        progressBar.setProgress(progresses[0]);
        textView.setText("loading..." + progresses[0] + "%");
    }

    //onPostExecute方法用于在执行完后台任务后更新UI,显示结果
    @Override
    protected void onPostExecute(String result) {
        Log.i(TAG, "onPostExecute");
        textView.setText(result);

        execute.setEnabled(true);
        cancel.setEnabled(false);
    }

    //onCancelled方法用于在取消执行中的任务时更改UI
    @Override
    protected void onCancelled() {
        Log.i(TAG, "onCancelled");
        textView.setText("cancelled");
        progressBar.setProgress(0);

        execute.setEnabled(true);
        cancel.setEnabled(false);
    }
}
```


创建并执行AsyncTask的接口如下：
```
mTask = new MyTask();
mTask.execute("http://www.baidu.com");
```
而取消任务

```
mTask.cancel();
```

从源码角度解读，首先我们从new MyTask()构架方法看起


```
    public AsyncTask() {
        //合建一个实现Callable接口的任务。
        mWorker = new WorkerRunnable<Params, Result>() {
            public Result call() throws Exception {
                mTaskInvoked.set(true);
                Result result = null;
                try {
                    Process.setThreadPriority(Process.THREAD_PRIORITY_BACKGROUND);
                    //noinspection unchecked
                    result = doInBackground(mParams);
                    Binder.flushPendingCommands();
                } catch (Throwable tr) {
                    mCancelled.set(true);
                    throw tr;
                } finally {
                    postResult(result);
                }
                return result;
            }
        };
        //传入mWorker参数创建一个Future对象，主要用于了解线程运行的执行情况。
        mFuture = new FutureTask<Result>(mWorker) {
            @Override
            protected void done() {
                try {
                    postResultIfNotInvoked(get());
                } catch (InterruptedException e) {
                    android.util.Log.w(LOG_TAG, e);
                } catch (ExecutionException e) {
                    throw new RuntimeException("An error occurred while executing doInBackground()",
                            e.getCause());
                } catch (CancellationException e) {
                    postResultIfNotInvoked(null);
                }
            }
        };
    }
```
从构造方法中，我们可以看出，首先我们创建一个继承于Callable的任务。此任务通常在线程池中执行。然后通过传入线程任务mWorker创建一个future,主要用于查询线程执行以及获得线程的返回值。

再来看AsyncTask的执行方法excute();


```
    public final AsyncTask<Params, Progress, Result> execute(Params... params) {
        return executeOnExecutor(sDefaultExecutor, params);
    }

```
实质上调用executeOnExecutor（）方法

```
    public final AsyncTask<Params, Progress, Result> executeOnExecutor(Executor exec,
            Params... params) {
        if (mStatus != Status.PENDING) {
            switch (mStatus) {
                case RUNNING:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task is already running.");
                case FINISHED:
                    throw new IllegalStateException("Cannot execute task:"
                            + " the task has already been executed "
                            + "(a task can be executed only once)");
            }
        }

        mStatus = Status.RUNNING;

        onPreExecute();

        mWorker.mParams = params;
        exec.execute(mFuture);

        return this;
    }
```
我们看到，参数Executor表示一个执行器。Params...表示传入动态参数，那默认的sDefaultExecutor这个执行器又是什么什么呢？

```
public static final Executor SERIAL_EXECUTOR = new SerialExecutor();

private static class SerialExecutor implements Executor {
        final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
        Runnable mActive;

        public synchronized void execute(final Runnable r) {
            mTasks.offer(new Runnable() {
                public void run() {
                    try {
                        r.run();
                    } finally {
                        scheduleNext();
                    }
                }
            });
            if (mActive == null) {
                scheduleNext();
            }
        }

        protected synchronized void scheduleNext() {
            if ((mActive = mTasks.poll()) != null) {
                THREAD_POOL_EXECUTOR.execute(mActive);
            }
        }
    }
```
我们发现这个执行器只是对runnable的一个包装，只是将任务包装，然后将任务交给THREAD_POOL_EXECUTOR执行。


```
    public static final Executor THREAD_POOL_EXECUTOR;

    static {
        ThreadPoolExecutor threadPoolExecutor = new ThreadPoolExecutor(
                CORE_POOL_SIZE, MAXIMUM_POOL_SIZE, KEEP_ALIVE_SECONDS, TimeUnit.SECONDS,
                sPoolWorkQueue, sThreadFactory);
        threadPoolExecutor.allowCoreThreadTimeOut(true);
        THREAD_POOL_EXECUTOR = threadPoolExecutor;
    }
```
而这个THREAD_POOL_EXECUTOR是一个线程池。负责任务真正的执行。

我们再回到executeOnExecutor方法中，因为这个方法是主线程中执行的，所以onPreExecute()方法也在主线程中执行，然后再调用exec.execute(mFuture);在线程中执行mFuture任务。

而前面我们知道，mFuture任务的执行体是mWorker(WorkerRunnable)的call方法。我们再回到call()方法中，发现result = doInBackground(mParams)方法，所以线程中耗时任务是在doInBackground中处理的。耗时任务执行完后，会调用postResult(result)方法。


我们再来看postResult(result)方法


```
    private Result postResult(Result result) {
        @SuppressWarnings("unchecked")
        Message message = getHandler().obtainMessage(MESSAGE_POST_RESULT,
                new AsyncTaskResult<Result>(this, result));
        message.sendToTarget();
        return result;
    }
```
很显示，是通过handler机制把消息发送到主线程中，我们看看handler的处理对象。


```
    private static class InternalHandler extends Handler {
        public InternalHandler() {
            super(Looper.getMainLooper());
        }

        @SuppressWarnings({"unchecked", "RawUseOfParameterizedType"})
        @Override
        public void handleMessage(Message msg) {
            AsyncTaskResult<?> result = (AsyncTaskResult<?>) msg.obj;
            switch (msg.what) {
                case MESSAGE_POST_RESULT:
                    // There is only one result
                    result.mTask.finish(result.mData[0]);
                    break;
                case MESSAGE_POST_PROGRESS:
                    result.mTask.onProgressUpdate(result.mData);
                    break;
            }
        }
    }
```

接收到消息后，执行了result.mTask.finish(result.mData[0]);方法。


```
    private void finish(Result result) {
        if (isCancelled()) {
            onCancelled(result);
        } else {
            onPostExecute(result);
        }
        mStatus = Status.FINISHED;
    }
```
最后如果消息取消，则回调onCancelled，否则回调onPostExecute。此两个方法都在ui线程。

大家可以会疑惑，onProgressUpdate是在哪里执行的呢？众所周知，这个方法是负责更新任务进度的。而任务进度只有在doInBackground方法中可以得知，而doInBackground是在后台线程中，故肯定需要通过Handler通知，所以AsyncTask封装了publishProgress(progress...)来通过标志
MESSAGE_POST_PROGRESS回调onProgressUpdate方法。

重点说明：

1、executeOnExecutor()首先会对任务的状态进行处理。任务共三种姿态

- PENDING: 挂起状态。当AsyncTask被创建时，就进入了PENDING状态。
- RUNNING: 运行状态。当AsyncTask被执行时，就进入了RUNNING状态。
- FINISHED: 完成状态。当AsyncTask完成(被客户cancel()或正常运行完毕)时，就进入了FINISHED状态。


当任务是RUNNING或PENDING状态时，会抛出异常。这就决定了，一个AsyncTask只能被执行一次，即只能对一个AsyncTask调用一次execute()；如果要重新执行任务，则需要新建AsyncTask后再调用execute()。


SerialExecutor是一个顺序执行器，那么这个执行器到底是如何做到的，我们再贴一下代码。


```
private static class SerialExecutor implements Executor {
    final ArrayDeque<Runnable> mTasks = new ArrayDeque<Runnable>();
    Runnable mActive;

    public synchronized void execute(final Runnable r) {
        mTasks.offer(new Runnable() {
            public void run() {
                try {
                    r.run();
                } finally {
                    scheduleNext();
                }
            }
        });
        if (mActive == null) {
            scheduleNext();
        }
    }

    protected synchronized void scheduleNext() {
        if ((mActive = mTasks.poll()) != null) {
            THREAD_POOL_EXECUTOR.execute(mActive);
        }
    }
}
```
可以看到，SerialExecutor是使用ArrayDeque这个队列来管理Runnable对象的，如果我们一次性启动了很多个任务，首先在第一次运行execute()方法的时候，会调用ArrayDeque的offer()方法将传入的Runnable对象添加到队列的尾部，然后判断mActive对象是不是等于null，第一次运行当然是等于null了，于是会调用scheduleNext()方法。在这个方法中会从队列的头部取值，并赋值给mActive对象，然后调用THREAD_POOL_EXECUTOR去执行取出的取出的Runnable对象。之后如何又有新的任务被执行，同样还会调用offer()方法将传入的Runnable添加到队列的尾部，但是再去给mActive对象做非空检查的时候就会发现mActive对象已经不再是null了，于是就不会再调用scheduleNext()方法。

那么后面添加的任务岂不是永远得不到处理了？当然不是，看一看offer()方法里传入的Runnable匿名类，这里使用了一个try finally代码块，并在finally中调用了scheduleNext()方法，保证无论发生什么情况，这个方法都会被调用。也就是说，每次当一个任务执行完毕后，下一个任务才会得到执行，SerialExecutor模仿的是单一线程池的效果，如果我们快速地启动了很多任务，同一时刻只会有一个线程正在执行，其余的均处于等待状态。


也许你不知道，在android3.0之前并没有这个SerialExecutor类。个时候是直接在AsyncTask中构建了一个sExecutor常量，并对线程池总大小，同一时刻能够运行的线程数做了规定，代码如下所示：


```
private static final int CORE_POOL_SIZE = 5;
private static final int MAXIMUM_POOL_SIZE = 128;
private static final int KEEP_ALIVE = 10;
……
private static final ThreadPoolExecutor sExecutor = new ThreadPoolExecutor(CORE_POOL_SIZE,
        MAXIMUM_POOL_SIZE, KEEP_ALIVE, TimeUnit.SECONDS, sWorkQueue, sThreadFactory);
```
表示同一个时刻，可以运行的线程数据为5个，最后可以同时存在128个线程。所以在3.0之前的版本，如果5个任务执行的时候，可以同时执行，而在3.0之后的版本中，5个任务是串行执行的。

当然在新版本中，我们也可以直接调用executeOnExecutor方法，传入指定的执行器（或者线程也）即可。

例如起用调用task.executeOnExecutor(THREAD_POOL_EXECUTOR,params)即可。


