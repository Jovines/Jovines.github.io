---
title: fitsSystemWindows相关总结
date: 2020-03-18 23:50:07
tags:
- 归纳总结
categories:
- Android
---

## 先上结论

```xml
android:fitsSystemWindows="true"
```

* 默认情况下，设置它的view会添加一些内边距，用来留出一些系统的window的区域，例如状态栏或者底部的虚拟键盘

  > 日常我们最常见的就只有这两个，但不排除其他或者后面会出现的一些新的系统window要留出，所以一旦设置这个属性之后你自己设置的**所有的padding都会失效，上下左右全部，记住是全部**

  另外在xml文件里，配置了这个属性为true的，**哪个控件写在前面谁的就生效**，而且**只能生效一个**，在它之后的后不会生效。

* View是可以对`fitsSystemWindows`进行个性化的，也就是说可以替换默认的添加Padding预留系统视窗的操作，最好的例子我们在下面会讲

网络上有些博客简直在喷粪，CV神通自己都没有试一下就乱说