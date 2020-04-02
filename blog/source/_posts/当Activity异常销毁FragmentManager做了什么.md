---
title: 当Activity异常销毁FragmentManager做了什么
date: 2020-03-27 02:36:09
tags:
- 查漏补缺
categories:
- Android
thumbnail: ../当Activity异常销毁FragmentManager做了什么/image-20200327042035621.png
---



最近在做项目的时候因为`Activity`和`FragmentManager`，使项目遇到了一些问题，`Activity`异常销毁重启之后，上次加载进去的`Fragment`也依然存在`FragmentManager`当中，但是在`Activity`中的初始化函数仍然会生成新的`fragment`进去。以此导致了界面出现了重复加载的问题。

下面的分析都是基于`Androidx`的源码，和`support`有些区别

## 问题出现可能的原因

* `FragmentManager`生命周期更长
* 销毁之前恢复机制将这些`fragment`数据进行了保存，重新创建时恢复了

#### `FragmentManager`生命周期更长？？？

`Activity`异常销毁（或者配置更改）时`FragmentManager`会像`ViewModel`一样自动保存下来？以便于下次加载使用吗

**证明一下**

```kotlin
class MainActivity : AppCompatActivity() {

    companion object{
        const val TAG = "MainActivityDemo"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        Log.d(TAG, "${supportFragmentManager}")
    }
}
```

接下来我们开启分屏

![image-20200327042035621](%E5%BD%93Activity%E5%BC%82%E5%B8%B8%E9%94%80%E6%AF%81FragmentManager%E5%81%9A%E4%BA%86%E4%BB%80%E4%B9%88/image-20200327042035621.png)

很明显这里不是同一个对象，这个原因pass。

#### 被恢复了？？

<img src="%E5%BD%93Activity%E5%BC%82%E5%B8%B8%E9%94%80%E6%AF%81FragmentManager%E5%81%9A%E4%BA%86%E4%BB%80%E4%B9%88/v2-308d4b287931127b020ac4cc861a1757_hd-1571843355924.jpg" alt="img" style="zoom:50%;" />我们来看看`FragmentActivity`中`onSaveInstanceState`干了什么

```java
protected void onSaveInstanceState(@NonNull Bundle outState) {
    super.onSaveInstanceState(outState);
    markFragmentsCreated();//标记已经创建的fragment
    mFragmentLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_STOP);
    Parcelable p = mFragments.saveAllState();//保存状态
    if (p != null) {
        outState.putParcelable(FRAGMENTS_TAG, p);//放到Bundle
    }
    // do something..................
}
```

这里的确做了保存，那么我们看看在哪里用到了`FRAGMENTS_TAG`这个标志位

```java
protected void onCreate(@Nullable Bundle savedInstanceState) {
    mFragments.attachHost(null /*parent*/);
    if (savedInstanceState != null) {
        Parcelable p = savedInstanceState.getParcelable(FRAGMENTS_TAG);//获取保存的数据
        mFragments.restoreSaveState(p);//对数据进行了恢复
        // do something............
    }

    // do something............
    
    mFragmentLifecycleRegistry.handleLifecycleEvent(Lifecycle.Event.ON_CREATE);
    mFragments.dispatchCreate();
}



```

具体保存操作是怎么操作的这个，自己去研究，不在本篇文章讨论范围

涉及到`FragmentManager`对`fragment`的`onSaveInstanceState`和`onRestoreInstanceState`的层层调用

我们来看看一个Fragment到底保存了一些什么东西（以下我只保留了关键函数）

```java
final class FragmentState implements Parcelable {
    final String mClassName;
    final String mWho;
    final boolean mFromLayout;
    final int mFragmentId;
    final int mContainerId;
    final String mTag;
    final boolean mRetainInstance;
    final boolean mRemoving;
    final boolean mDetached;
    final Bundle mArguments;
    final boolean mHidden;
    final int mMaxLifecycleState;

    Bundle mSavedFragmentState;

    Fragment mInstance;

    //传入fragment对数据进行构造
    FragmentState(Fragment frag) {
        mClassName = frag.getClass().getName();//类名
        mWho = frag.mWho;//这个fragment对象独有的id
        mFromLayout = frag.mFromLayout;//是否是直接从布局实例化的（boolean）
        mFragmentId = frag.mFragmentId;//若是动态添加的，则为容器id，从视图添加则为布局中的id
        mContainerId = frag.mContainerId;//动态添加时，容器的id
        mTag = frag.mTag;
        //这个属性有些复杂，如果fragment设置这个属性那么
        //这个fragment会被存到FragmentManagerViewModel里，
        //这个viewModel的owner是FragmentManager依附的Activity
        //由于ViewModel的特性，activity配置更改后viewModel会自动保存
        //下一次FragmentManager恢复数据时会优先判断判断FragmentManagerViewModel
        //里是否保存，如果保存了就不调用下面的instantiate创建新的Fragment对象
        mRetainInstance = frag.mRetainInstance;
        mRemoving = frag.mRemoving;//是否从activity中移除
        mDetached = frag.mDetached;//是否被禁用，当被detach时会被设置为false
        mArguments = frag.mArguments;//这个是个Bundle，用来传递东西的
        mHidden = frag.mHidden;//当前Fragment是否时隐藏状态
        mMaxLifecycleState = frag.mMaxState.ordinal();
    }

    //通过保存的信息恢复一个完整的Fragment
    public Fragment instantiate(@NonNull ClassLoader classLoader,
            @NonNull FragmentFactory factory) {
        if (mInstance == null) {
            if (mArguments != null) {
                mArguments.setClassLoader(classLoader);
            }

            mInstance = factory.instantiate(classLoader, mClassName);
            mInstance.setArguments(mArguments);

            if (mSavedFragmentState != null) {
                mSavedFragmentState.setClassLoader(classLoader);
                mInstance.mSavedFragmentState = mSavedFragmentState;
            } else {
                // When restoring a Fragment, always ensure we have a
                // non-null Bundle so that developers have a signal for
                // when the Fragment is being restored
                mInstance.mSavedFragmentState = new Bundle();
            }
            mInstance.mWho = mWho;
            mInstance.mFromLayout = mFromLayout;
            mInstance.mRestored = true;
            mInstance.mFragmentId = mFragmentId;
            mInstance.mContainerId = mContainerId;
            mInstance.mTag = mTag;
            mInstance.mRetainInstance = mRetainInstance;
            mInstance.mRemoving = mRemoving;
            mInstance.mDetached = mDetached;
            mInstance.mHidden = mHidden;
            mInstance.mMaxState = Lifecycle.State.values()[mMaxLifecycleState];

            if (FragmentManagerImpl.DEBUG) {
                Log.v(FragmentManagerImpl.TAG, "Instantiated fragment " + mInstance);
            }
        }
        return mInstance;
    }

}
```

从上面的代码注释就可以看出，保存了哪些东西

看来这个解答时正解，哦耶<img src="%E5%BD%93Activity%E5%BC%82%E5%B8%B8%E9%94%80%E6%AF%81FragmentManager%E5%81%9A%E4%BA%86%E4%BB%80%E4%B9%88/v2-56d8e6cc72c947ee95df5a1a7bff9fc2_hd-1571843375376.jpg" alt="img" style="zoom:50%;" />