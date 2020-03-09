---
title: Android消息机制深入了解
date: 2020-03-08 19:53:47
tags:
- 温故知新
categories:
- Android
---

# 为什么消息机制这么重要呢

其实一个安卓的app也是有自己的**main**方法的<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-2f4c1f2a5eb83e1368364c9a87aa258c_hd-1571843439577.jpg" alt="img" style="zoom:50%;" />，跟一个普通的java程序并没有太大的区别，只不过用户接触不到这个方法，这个方法在**ActivityThread**这个类里面，这个也就是android中所说的**UI线程**。你可能在想一个问题，那么既然是一个有main方法的java程序，那么一整个安卓的app究竟是怎么来运行的呢，如何做到和用户交互并执行这些代码的呢？？（这里涉及到很多东西，本文只讲与**消息机制相关**的，文章**末尾**我会一一解释）<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-70cd1cce150de4d7c1c80c6fea270ec3_hd.jpg" alt="img" style="zoom:50%;" />

# 消息机制重要的三大成员

* **MessageQueue**

* **Looper**

* **Handler**

  > 一个线程只能有一个Looper和MessageQueue，但是可以有多个handler，为什么呢？后面我们慢慢会讲到。



那么这三者是如何配合的呢，简单来说，**MessageQueue**就是一根**运输消息管道**，handler负责不断往里面**放**，**Looper**负责不断从里面**取**出来，又交给发送的这个handler去处理，那么就出现两个问题？

* 那么为什么要这样子设计？这样子设计怎么实现的**跨线程通信**
* 为什么handler放进去最后looper又要交还给发送消息的这个handler去处理

通过后面的源码解析，就能自己领会这两个问题

# 消息机制主要流程详解

如果你要在一个线程中使用消息机制

首先需要`Looper.prepare();`和`Looper.loop();`，然后你就可以使用你的**handler发送消息**了。

> 主线程是不需要自己`Looper.prepare();`和`Looper.loop();`在ActivityThread的main方法里面就已经执行了类似代码`Looper.prepareMainLooper();`和`Looper.loop();`

下面我会逐一从**源码层面**来分析整个消息机制的主要流程<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-9e6c67c00b10fe66487bdac67d1642e5_hd.jpg" alt="img" style="zoom:50%;" />，会不会很难呢，要不要继续学下去，头发 - -。

## Looper.prepare()；

首先我们点进去看看代码是如何实现的

```java
//这个函数在ActivityThread的main方法里面执行,用户创建主线程的Looper对象
public static void prepareMainLooper() {
        prepare(false);
        synchronized (Looper.class) {
            if (sMainLooper != null) {
                throw new IllegalStateException("The main Looper has already been prepared.");
            }
            sMainLooper = myLooper();
        }
}

//通常我们在其他线程如果需要用到消息机制，会执行这个方法
public static void prepare() {
    prepare(true);
}

//函数重载，这个参数用于构造一个可以关闭或者无法关闭的Looper
private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

最重要的方法就是第三个`prepare(boolean quitAllowed)`，我们可以看到代码很短，也很好理解。

首先调用这个这个**Looper类**里面一个**静态变量`sThreadLocal`**

```java
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
```

这是一个**`ThreadLocal<Looper>`类型**的一个变量，而且是直接**`new`出来的**,点击进去看

```java
public ThreadLocal() {
}
```

查看**ThreadLocal源代码里面的构造方法**也没有做任何操作，说明本身这个变量是里是没有我们想要的东西的

那么**关键肯定是在`sThreadLocal.get()`里面**，通过后面的抛出异常

`throw new RuntimeException("Only one Looper may be created per thread");`

我们可以断定这个**判断**肯定**是**用来**判断当前线程是否已经存在了一个Lopper对象**

接下来我们来看`get()`方法内部

```java
//代码不是很多，我们一步步分析
public T get() {
    Thread t = Thread.currentThread();//获得当前的线程对象
    ThreadLocalMap map = getMap(t);//从当前线程拿到一个ThreadLocalMap
    if (map != null) {//如果这个ThreadLocalMap不为空
      	//从里面取得Looper（这里做了一个包装，稍后会讲到）
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {//如果Lopper不为空
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;//直接返回Looper
        }
    }
    return setInitialValue();//如果从当前线程中取得的ThreadLocalMap为空，则进行初始化操作
}
```

可能虽然有注释还是有些疑惑，下面我们一一来讲解一下 Why<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-639605533bdb1ecf1c6562677a430554_hd.jpg" alt="img" style="zoom:50%;" />

首先我们来看看**`ThreadLocalMap map = getMap(t);`中`get(t)`的代码**

```java
ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

可以看到，是直接返回了**当前线程对象**的一个**成员变量`threadLocals`**，那么这个`threadLocals`究竟是个什么东西呢？

### ThreadLocal和ThreadLocalMap

Entry是ThreadLocalMap的静态内部类，ThreadLocalMap是ThreadLocal的静态内部类

首先我们来看`ThreadLocal`，顾名思义，直接翻译过来就是 “线程本地” ？？<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-a636ef3559b5600fdbaaf48cf794f5e4_hd.jpg" alt="img" style="zoom:50%;" />

我们来看看这个类的介绍注释

```java
/**
 * This class provides thread-local variables.  These variables differ from
 * their normal counterparts in that each thread that accesses one (via its
 * {@code get} or {@code set} method) has its own, independently initialized
 * copy of the variable.  {@code ThreadLocal} instances are typically private
 * static fields in classes that wish to associate state with a thread (e.g.,
 * a user ID or Transaction ID).
 */

/**
* 谷歌翻译版本:
* 此类提供线程局部变量
* 这些变量不同于它们的普通副本，
* 因为每个访问一个变量的线程（通过其{@code get}或{@code set}方法）
* 都有自己的、独立的变量副本。
* {@code ThreadLocal}实例通常是类中的私有静态字段，希望将状态与线程关联（例如，*用户ID或事务ID）。
*/
  
```

通过注释我们可以很轻松的理解到，原来这个东西就是为每一个线程创建一个单独的变量副本的呀<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-ee07600d1fc49257fb502f869bb97264_hd.jpg" alt="img" style="zoom:50%;" />。

我们来看看到底怎么使用

```java
public class Main {
    public static void main(String[] args) {
        ThreadLocal<Integer> integerThreadLocal = new ThreadLocal<>(){
            //这个方法专门用来提供开发者复写，提供副本默认值
            @Override
            protected Integer initialValue() {
                return 2333;//默认值我们这设置为
            }
        };
        
        for (int i = 0; i < 5; i++) {
            new Thread(){
                @Override
                public void run() {
                    //为这个ThreadLocal<Integer>在当前线程存一个副本，值为3222
                    //如果不是基本数据类型，注意值传递和引用传递的区别，这里需要new一个新的放进去
                    //例如：ThreadLocal的泛型是Runnable,
                    //那么就需要新建一个独立的Runnable实现类的对象
                    //runnableThreadLocal.set(new Runnable() {
                    //            @Override
                    //            public void run() {
                    //
                    //            }
                    //        };)
                    integerThreadLocal.set(3222);
                    //获取当前线程的副本，如果之前没有set()，那么默认值为2333
                  	//如果之前set()了，就会取得set()的那个对象
                    integerThreadLocal.get();

                    /**
                     * 通过以上代码，在这5个线程里都有了一个与integerThreadLocal对应
                     * 变量副本，这个副本是Integer类型的，每个线程通过integerThreadLocal.get()
                     * 取到的都是线程独有的，与其他线程完全隔离的一个副本
                     */
                }
            }.start();
        }

    }
}
```

通过上面的注释就足以理解这个东西的用法，这时候我们回到安卓来看看消息机制这里的设计，是不是就一下子恍然大悟了<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-790d03f584a1de8717e02e56547e38ca_hd.jpg" alt="img" style="zoom:50%;" />，我这再梳理一下

* 首先Lopper中是有一个`sThreadLocal`

  ```java
  static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
  ```

  理解了上面小Demo,就能知道因为这个`sThreadLocal`的存在，我们就可以在每一个线程存一个Looper副本

  ```java
  private static void prepare(boolean quitAllowed) {
      if (sThreadLocal.get() != null) {
          throw new RuntimeException("Only one Looper may be created per thread");
      }
      sThreadLocal.set(new Looper(quitAllowed));
  }
  ```

  看，这是不是就很清晰了<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-8312b637ba6407992515ffa42b56391d_hd.jpg" alt="img" style="zoom:50%;" />



然后我们来看看`ThreadLocalMap`，前面我们看到`ThreadLocal.get()`的`get()`方法,这里再把代码贴一下

```java
public T get() {
    Thread t = Thread.currentThread();//获得当前的线程对象
    ThreadLocalMap map = getMap(t);//从当前线程拿到一个ThreadLocalMap
    if (map != null) {//如果这个ThreadLocalMap不为空
      	//从里面取得Looper（这里做了一个包装，稍后会讲到）
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {//如果Lopper不为空
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;//直接返回Looper
        }
    }
    return setInitialValue();//如果从当前线程中取得的ThreadLocalMap为空，则进行初始化操作
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}
```

从上面的代码可以知道，一个线程对象里面是有一个`threadLocals`属性的，它是一个`ThreadLocalMap`类型的变量。就是它用来储存各种存在当前线程的副本。他是一个键值对存储的形式,下面是`ThreadLocalMap`的set（）方法的声明

```java
set(ThreadLocal<?> key, Object value)
```

具体实现是比较复杂的，用到哈希表来储存，这里不展开讲了，有兴趣可以去了解一下，文章后面会有超链接。

然后我们来看看`ThreadLocalMap.Entry e = map.getEntry(this);`这个行代码，肯定很多人都会有疑惑，首先我们需要看一下ThreadLocalMap.Entry这个静态内部类

```java
static class Entry extends WeakReference<ThreadLocal<?>> {
    /** The value associated with this ThreadLocal. */
    Object value;

    Entry(ThreadLocal<?> k, Object v) {
        super(k);
        value = v;
    }
}
```

这里你可以看到这里是直接继承WeakReference弱引用这个类的，那为什么要这么设计呢？<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-5f1a5d4ebb5a0c3d2751ee342dd09d71_hd.jpg" alt="img" style="zoom:50%;" />

这里我大致的说一下，ThreadLocal是为了每个线程中的对象副本服务的，一旦我不需要使用ThreadLocal这个对象了，那么我肯定会将ThreadLocal对象的强引用去掉，以防止内存泄漏，但是这个如果不这么设计，如果有一个线程的生命周期比ThreadLocal长，那么必定会持有ThreadLocal的引用，这样就会导致内存泄漏，ThreadLocal没有被使用但是却无法被GC回收。相比之下，这样设计就不会出现这个问题。

### Looper的构造方法

```java
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);//生成一个MessageQueue，并作为成员保存在Looper中
    mThread = Thread.currentThread();//绑定当前线程
}
```

从上面的代码也印证了Looper和MessageQueue一般都是同时出现的，而且都只能有一个的言论（因为prepare方法在一个线程调用一次，前面代码提到过）。

## Looper.loop

```java
//因为方便，我只选择了关键代码
public static void loop() {
        final Looper me = myLooper();//取得当前线程的Looper，当然实际上也是通过sThreadLocal.get()
        if (me == null) {//如果没有说明没有调用prepare(),抛出异常
            throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
        }
        final MessageQueue queue = me.mQueue;//从当前的Looper对象里取得对应的MessageQueue

        //do something............

        for (; ; ) {
            Message msg = queue.next(); // 取出消息，这里可能会堵塞，直到重新取到消息
            if (msg == null) {
              	//如果返回为空，则表示MessageQueue正在退出，所以这里也直接退出无限循环
              	//这里是只有MessageQueue调用quit(boolean safe)方法才会退出
              	//下面会有详细解释
                return;
            }

            //do something.............

          	//这个target是发送这个消息的handler，取得消息之后让发送的这个handler去处理
          	//后面我们会讲到，handler是如何把这个消息发送到当前这个MessageQueue中，并且target的赋值
            msg.target.dispatchMessage(msg);

            //do something.............
            msg.recycleUnchecked();//回收这个消息
        }
}
```

通过代码应该很清晰就能展现出关键点，这里就不再赘述

因为这里涉及到Looper.loop退出的情况，下面我们用代码类分析一下，关键就在`queue.next()`，为了便于理解还是只凸显出主要代码，还有很多MessageQueue的消息处理的一些代码就不赘述了，下面代码重点看那个返回值为空的判断语句

```java
    Message next() {

        //do something..........

        for (; ; ) {//next中也是一个无限循环着也就很好的说明为什么next()是一个可堵塞的方法啦

            //do something..........

            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;
                } while (msg != null && !msg.isAsynchronous());
            }
          
            if (msg != null) {
              	//这里是后面handler的sendMessage方法会给message添加一个时间的处理
                if (now < msg.when) {
                    //do something..........
                } else {
                    //do something..........
                    return msg;//如果所有条件都满足，就可以直接返回这个message了
                }
            } else {
                // No more messages.
                nextPollTimeoutMillis = -1;
            }

            // Process the quit message now that all pending messages have been handled.
          	// 前面没有返回，这里如果为true只就直接返回null，前面Looper.loop就退出了
            if (mQuitting) {
                dispose();
                return null;
            }

            //do something..........

            if (pendingIdleHandlerCount <= 0) {
                // No idle handlers to run.  Loop and wait some more.
                mBlocked = true;
                continue;
            }

            //do something..........

        }
    }

```

我们来看看`mQuitting`这个变量是在哪被赋值为true的

```java
void quit(boolean safe) {
  	//mQuitAllowed这个其实在前文多次提到了，Looper和MessageQueue的构造方法都需要
  	//用来设定是否可以被退出
    if (!mQuitAllowed) {
        throw new IllegalStateException("Main thread not allowed to quit.");
    }

    synchronized (this) {
        if (mQuitting) {
            return;
        }
        mQuitting = true;//关键代码，这里赋值为true

        if (safe) {
            removeAllFutureMessagesLocked();
        } else {
            removeAllMessagesLocked();
        }

        // We can assume mPtr != 0 because mQuitting was previously false.
        nativeWake(mPtr);
    }
}
```

从上面分析我们可以知道一旦调用了MessageQueue的quit的方法那么MessageQueue下一个利用next方法去取消息就会返回null，返回null就导致Looper的loop(也就是无限取消息的循环退出了),那么就有一个问题了，Looper也有一个quit方法，怎么实现的呢？

```java
public void quit() {
    mQueue.quit(false);
}
```

恍然大悟！！！！！！！<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-4d2cb53368cc8b322c3075d23d1e0c34_hd-1571843599481.jpg" alt="img" style="zoom:50%;" />

## handler.sendMessage()

看发送消息之前我们需要先看看**Handler的构造方法**

```java
//无参构造函数
public Handler() {
    this(null, false);
}

//有参构造函数
public Handler(@Nullable Callback callback, boolean async) {
  
    //do something............
  
  	//获取当前线程的Looper
    mLooper = Looper.myLooper();
    if (mLooper == null) {//同样如果没有Looper会报错（没有Looper.prepare()）
        throw new RuntimeException(
            "Can't create handler inside thread " + Thread.currentThread()
                    + " that has not called Looper.prepare()");
    }
  
    mQueue = mLooper.mQueue;//绑定当前线程的MessageQueue
  
    //do something............
}
```

可以看出**Handler在那个线程创建**，它就会**直接绑定哪个线程的MessageQueue**，之后它`sendMessage()`就可以**直接放到这个MessageQueue里面**

还是直接来看`sendMessage()`的代码

```java
//点进来第一个方法是这个
public final boolean sendMessage(@NonNull Message msg) {
    return sendMessageDelayed(msg, 0);
}

//发送延迟处理的消息
public final boolean sendMessageDelayed(@NonNull Message msg, long delayMillis) {
   if (delayMillis < 0) {
        delayMillis = 0;
    }
    return sendMessageAtTime(msg, SystemClock.uptimeMillis() + delayMillis);
}

public boolean sendMessageAtTime(@NonNull Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;//拿到创建Handler时绑定的那个MessageQueue
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
   	}
    return enqueueMessage(queue, msg, uptimeMillis);
}

private boolean enqueueMessage(@NonNull MessageQueue queue, @NonNull Message msg,
        long uptimeMillis) {
    msg.target = this;//【发送的消息】绑定【发送消息的这个handler】
    msg.workSourceUid = ThreadLocalWorkSource.getUid();

    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
  	//把消息交给MessageQueue去排序消息，这里是根据uptimeMillis转成具体时刻来排序的
  	//这里要注意，发送的延迟消息，只能保持设置的时间前这个消息不被处理
  	//但是无法保证这个消息具体在设置的时间之后的那个时候被处理
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

### handler.post()

还是直接看代码，如果上面的理解了这个就非常简单了<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-8312b637ba6407992515ffa42b56391d_hd-3736965.jpg" alt="img" style="zoom:50%;" />，只有一些区别

```java
public final boolean post(@NonNull Runnable r) {
   //这里仍然还是调用的发送延时消息的方法，区别是下面那个方法
   return  sendMessageDelayed(getPostMessage(r), 0);
}

//将Runnable包装成Message，那么这个Runnable在哪会被调用呢
private static Message getPostMessage(Runnable r) {
    Message m = Message.obtain();
    m.callback = r;
    return m;
}
```

还记得前面讲到的Looper.loop吗，它会把消息交给与他绑定的Handler去处理来看看这个方法

```java
public void dispatchMessage(@NonNull Message msg) {
    if (msg.callback != null) {//优先优先判断callback是不是空
        handleCallback(msg);//不为空直接调用方法处理callback
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}

private static void handleCallback(Message message) {
    message.callback.run();//这里直接执行了runnable的run方法
}
```



# 结尾

文章开头留下的悬念我想这下已经很清晰了，主线程中的main方法调用了Looper.loop,正是有这个无限循环的存在才会不会想一个简单的java程序一样，执行完了main里面的代码程序就结束了，而`prepareMainLooper	`,正是为主线程（UI线程）准备了一个不可以退出的Looper。而这就可以说明Android 的是由事件驱动的，looper.loop() 不断地接收事件、处理事件，每一个点击触摸或者说Activity的生命周期都是运行在 Looper.loop() 的控制之下，如果它停止了，应用也就停止了。只能是某一个消息或者说对消息的处理阻塞了 Looper.loop()，而不是 Looper.loop() 阻塞它。**也就说我们的代码其实就是在这个循环里面去执行的**，所以你在你的子线程要使用消息机制接受消息的话，最好将Looper.loop()放到你要直接执行的所有代码后面，不然没有执行qiut方法，后面的代码就不会执行。<img src="Android%E6%B6%88%E6%81%AF%E6%9C%BA%E5%88%B6%E6%B7%B1%E5%85%A5%E4%BA%86%E8%A7%A3/v2-ca9bf81c0e81a09cf4f6dfaacd13ce04_hd.jpg" alt="img" style="zoom: 67%;" />真逊，原来这么简单，是吧，看来还是要好好学习下去，知识++。