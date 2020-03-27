---
title: 关于activity异常销毁重启之后得处理
date: 2020-03-26 12:00:56
tags:
- 新知识
categories:
- Android
---

前段时间处理项目的由于activity异常销毁导致的一系列问题的解决，这里记录一下

## 哪些时候activity会异常销毁

* 切换夜间模式
* 内存不足，activity被回收
* 分屏或者分屏下窗口调整

* ......................................

## 出现了哪些问题？

* 由于项目使用的是MVVM框架，使用ViewModel来处理数据和保存数据，所以activity异常重启之后如果没有注意容易导致重复加载，因为activity配置更改之后viewModel会被自动保留

  > 在配置更改期间会自动保留 [`ViewModel`](https://developer.android.com/reference/androidx/lifecycle/ViewModel?hl=zh-cn) 对象，以便它们存储的数据立即可供下一个 Activity 或 Fragment 实例使用。

* 由于FragmentManager

