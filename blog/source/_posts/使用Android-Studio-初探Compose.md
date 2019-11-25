title: 使用Android Studio 初探Compose
author: Jon
tags:
  - 新知识
categories:
  - Android
    date: 2019-11-24 19:06:00

---

## Compose是啥？

官网上的解释是

**Android’s modern toolkit for building native UI**

### 声明式UI

声明式UI 具体是什么请查看这个：[知乎直达飞机票](https://zhuanlan.zhihu.com/p/68275232)

**Jetpack Compose 是一个用于构建原生Android UI 的现代化工具包，它基于声明式的编程模型，因此你可以简单地描述UI的外观，而Compose则负责其余的工作-当状态发生改变时，你的UI将自动更新。由于Compose基于Kotlin构建，因此可以与Java编程语言完全互操作，并且可以直接访问所有Android和Jetpack API。它与现有的UI工具包也是完全兼容的，因此你可以混合原来的View和现在新的View，并且从一开始就使用Material和动画进行设计。**

## 使用Jetpack Compose的必需配置

### 添加到现有项目

如果你想在现有的项目中使用Jetpack Compose，你需要配置一些必须的设置和依赖：

**（1）gradle 配置**

在app目录下的`build.gradle` 中将app支持的最低API 版本设置为21或更高，同时开启Jetpack Compose `enable`开关，代码如下：

```kotlin
android {
    defaultConfig {
        ...
        minSdkVersion 21
    }

    buildFeatures {
        // Enables Jetpack Compose for this module
        compose true
    }
    ...

    // Set both the Java and Kotlin compilers to target Java 8.

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = "1.8"
    }
}
```

**(2) 使用试验版`Kotlin-Gradle` 插件**

Jetpack Compose 需要试验版的`Kotlin-Gradle`插件,在根目录下的`build.gradle`添加如下代码：

```kotlin
buildscript {
    repositories {
        google()
        jcenter()
        // To download the required version of the Kotlin-Gradle plugin,
        // add the following repository.
        maven { url 'https://dl.bintray.com/kotlin/kotlin-eap' }
    ...
    dependencies {
        classpath 'com.android.tools.build:gradle:4.0.0-alpha01'
        classpath 'org.jetbrains.kotlin:kotlin-gradle-plugin:1.3.60-eap-25'
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        maven { url 'https://dl.bintray.com/kotlin/kotlin-eap' }
    }
}
```

**(3) 添加Jetpack Compose工具包依赖项**

在app目录下的`build.gradle`添加Jetpack Compose 工具包依赖项，代码如下：

```kotlin
dependencies {
    // You also need to include the following Compose toolkit dependencies.
    implementation 'androidx.ui:ui-tooling:0.1.0-dev02'
    implementation 'androidx.ui:ui-layout:0.1.0-dev02'
    implementation 'androidx.ui:ui-material:0.1.0-dev02'
    ...
}
```

到这里就完全完成了，但是还有一种更简单的方式来创建一个Compose应用

### 新建一个支持Compose的应用

* 使用预览版的[android studio](https://developer.android.com/studio/preview) 预览版的android 提供Compose app 的模版
* File -new - new project ，然后选择Empty Compose Activity 就好了
* 然后根据自己的需求配置一下版本就可以成功的创建一个支持Compose 的项目。



## 第一个Compose项目，hello word 

创建一个Compose项目之后会在MainAcitivity中生成这样几行代码

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MaterialTheme{
                Greeting("Android")
            }
        }
    }
}


@Composable
fun Greeting(name: String) {
    Column {
            Text(text = "Hello $name")
    }
}

@Preview
@Composable
fun DefaultPreview() {
    MaterialTheme {
        Greeting("Android")
    }
}
```

* 主要代码的视图代码都在`Greeting`中，其中有个纵向布局Column，然后里面放了一个文本控件，很简单
  * 可以看到`Greeting`和`DefaultPreview`都加了Composable注解，说明