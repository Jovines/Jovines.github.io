---
title: kotlin的let、with、run、apply、also系列方法记录
date: 2020-03-27 01:51:47
tags:
- 记录
categories:
- Kotlin
---

## let

#### 使用场景

- **场景一:** 最常用的场景就是使用let函数处理需要针对一个可null的对象统一做判空处理。
- **场景二:** 然后就是需要去明确一个变量所处特定的作用域范围内可以使用

#### 提醒

在函数块内可以通过 it 指代该对象。返回值为函数块的最后一行或指定return表达式

#### 示例

```kotlin
Any().let { it: Any ->

}
```

## with

#### 提醒

在函数块内可以通过 this 指代该对象。返回值为函数块的最后一行或指定return表达式

#### 示例

```kotlin
with(Any()){
    
}
```

## run

#### 解释

**let和with的结合**，函数块内，可以通过this代指该对象，返回值为函数块的最后一行或指定return表达式

#### 示例

```kotlin
Any().run { 
    
}
```

## apply

**与run类似，唯一的不同是apply始终返回该对象**

#### 示例

```kotlin
Any().apply {

}
```

## also

**结合apply和let**

**函数始终返回该对象，函数类可以通过it访该对象**

#### 示例

```kotlin
Any().also { it: Any ->

}
```