---
title: JVM-内存管理和垃圾回收
date: 2020-04-01 21:38:41
tags:
- 干货
categories:
- Java
---

## 内存管理

![JVM 内存管理总图 (JVM-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E5%92%8C%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6/JVM%20%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E6%80%BB%E5%9B%BE%20(1)-1585846004384.png)](../../../../../Users/12466/Downloads/JVM%20%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E6%80%BB%E5%9B%BE%20(1).png)

### 方法区 （元空间/持久代）

**线程共享**，存放类加载之后存放类的数据结构，静态常量，JIT(即时编译器)编译后代码也在方法区存放

### 堆区

**线程共享**，对象所在的区域，也是垃圾回收的主要场所

### 虚拟机栈

按照方法执行的顺序，先进后出

#### 栈帧

* 局部变量表
* 操作数栈
* 动态链接
* 方法出口

来看看一段简单的代码，主要分析demo()方法

```java
public class Main {

    public static void main(String[] args){
        demo();
    }

    public static int  demo(){
        int a = 20;
        int b = 30;
        int c = (a + b) * 100;
        return c;
    }
}
```

这里的demo（）方法会有一个局部变量表

![局部变量表 (JVM-%E5%86%85%E5%AD%98%E7%AE%A1%E7%90%86%E5%92%8C%E5%9E%83%E5%9C%BE%E5%9B%9E%E6%94%B6/%E5%B1%80%E9%83%A8%E5%8F%98%E9%87%8F%E8%A1%A8%20(1).png)](../../../../../Users/12466/Downloads/%E5%B1%80%E9%83%A8%E5%8F%98%E9%87%8F%E8%A1%A8%20(1).png)

赋默认值，真正使用的时候才会对其进行初始化。

我们把编译好了的java文件利用javap命令对class文件进行反汇编 javap -c Main.class

```java
Compiled from "Main.java"
public class Main {
  public Main();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public static void main(java.lang.String[]);
    Code:
       0: invokestatic  #2                  // Method demo:()I
       3: pop
       4: return

  public static int demo();
    Code:
       0: bipush        20  //将一个8位带符号整数（20）压入栈
       2: istore_0			//将操作数栈中栈顶int类型的值存入局部变量0
       3: bipush        30	//将一个8位带符号整数（30）压入栈
       5: istore_1			//将操作数栈中栈顶int类型的值存入局部变量1
       6: iload_0			//将局部变量0中的int类型值装载到操作数栈
       7: iload_1			//将局部变量1中的int类型值装载到操作数栈
       8: iadd				//操作数栈中的前两个int弹出并相加，并将结果压入操作数栈顶
       9: bipush        100	//将一个8位带符号整数（100）压入栈
      11: imul				//操作数栈中的前两个int弹出并相乘，并将结果压入操作数栈顶
      12: istore_2			//将操作数栈中栈顶int类型的值存入局部变量2
      13: iload_2			//将局部变量2中的int类型值装载到操作数栈
      14: ireturn			//将操作数栈中的int值返回
}
```

从注释可以很清晰的看出来

### 程序计数器

当前线程执行的字节码的位置指示器

### 本地方法栈 

为JVM提供使用native方法的服务大致结构和虚拟机栈差不多 

## 垃圾回收

STW（stop the world）：停止其他所有工作

Minor GC：轻量级，耗时短，会出现STW

Full GC：重量级，耗时长，会出现STW

### 分代垃圾回收

#### 新生代

对象会被优先被分配到这个地方

#### 老年代

当对象满足一定条件之后会被放到老年代

**注意**：java8之后老年代在Full GC之前是会执行一次Minor GC的，如果内存依然不足，才会执行Full GC

### 算法

* 标记清除

  优点：快

  缺点：会出现内存碎片，导致大对象进来会直接进入老年代

* 标记整理

  解决上面的的内存碎片问题

* 复制

  From（S0）区和To（S1）区来实现这个算法。

  > 这也是为什么需要交换的原因，To（S1）区永远保持空白