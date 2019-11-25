title: 安卓Drawable
author: Jon
tags:
  - 新知识
categories:
  - android
date: 2019-11-25 17:00:00
---

Android Drawable 相信大家都不陌生，本篇我们就来全面深入了解它，Drawable是一种可以在Canvas上进行绘制的抽象的图像，它的子类也相当多，所以在开发中很容易导致我们对不同Drawable的理解产生混乱，因此我们很有必要来全面了解一下Drawable的子类及其使用方式滴，哈~。

## 一、Drawable的简述

  Drawable在我们开发中常被用来作为View的背景图像，一般情况下我们都是通过XML来定义Drawable的，当然我们也可以通过代码创建Drawable，只不过会比较复杂而已。Drawable最大的好处就是可以方便我们做出一些特殊的UI效果，这点比我们自定义View实现的效果来得更容易些。因此深入理解Drawable的用法还是很有必要的，接下来我们来看看Drawable的一些特性：

- 1、Drawable本身表示的只是一种图像的概念，因此Drawable不仅仅是图片，也可以是颜色构造出来的图像效果（后面会说明）。
- 2、Drawable本身是一个抽象类，因此具体的实现都是由子类完成的，比如ShapeDrawable，BitmapDrawable等。
- 3、Drawable的内部宽高可以分别通过getIntrinsicWidth()和getIntrinsicHeight()获取，但并不是所有的Drawable都有内部宽高的属性，比如一个颜色形成的Drawable并没有宽高的概念。在大多数情况下，Drawable并没有大小的概念，因为当Drawable作为View的背景图时，Drawable会被拉伸至View的同等大小。



## 二、千变万化的Drawable

#### 1、BitmapDrawable

  BitmapDrawable 是对bitmap的一种包装，可以设置它包装的bitmap在BitmapDrawable区域内的绘制方式，如平铺填充、拉伸填充或者保持图片原始大小，也可以在BitmapDrawable区域内部使用gravity指定的对齐方式。其语法如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<bitmap
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:src="@[package:]drawable/drawable_resource"
    android:antialias=["true" | "false"]
    android:dither=["true" | "false"]
    android:filter=["true" | "false"]
    android:gravity=["top" | "bottom" | "left" | "right" | "center_vertical" |
                      "fill_vertical" | "center_horizontal" | "fill_horizontal" |
                      "center" | "fill" | "clip_vertical" | "clip_horizontal"]
    android:tileMode=["disabled" | "clamp" | "repeat" | "mirror"] 
     />123456789101112
```

我们分析一下这些属性的含义：
**android:src**
  类型：Drawable resource。必需。 引用一个drawable resource.
**android:antialias**
  类型：Boolean。是否开启抗锯齿。开启后图片会变得更平滑些，因此一般建议开启，设置为true即可。
**android:dither**
  类型：Boolean。是否允许抖动，如果位图与屏幕的像素配置不同时，开启这个选项可以让高质量的图片在低质量的屏幕上保持较好的显示效果（例如：一个位图的像素设置是 ARGB 8888，但屏幕的设置是RGB 565，开启这个选项可以是图片不过于失真）一般建议开启，为true即可。
**android:filter**
  类型：Boolean。是否允许对位图进行滤波。当图片被压缩或者拉伸时，使用滤波可以获得平滑的外观效果。一般建议开启，为true即可
**android:gravity**
  当图片小于容器尺寸时，设置此选项可以对图片经典定位，这个属性比较多，不同选项可以使用‘|’来组合使用。

| 可选项            | 含义                                               |
| ----------------- | -------------------------------------------------- |
| top               | 将图片放在容器顶部，不改变图片大小                 |
| bottom            | 将图片放在容器底部，不改变图片大小                 |
| left              | 将图片放在容器左侧，不改变图片大小                 |
| right             | 将图片放在容器右侧，不改变图片大小                 |
| center_vertical   | 图片竖直居中，不改变图片大小                       |
| fill_vertical     | 图片竖直方向填充容器                               |
| center_horizontal | 图片水平居中，不改变图片大小                       |
| fill_horizontal   | 图片水平方向填充容器                               |
| center            | 使图片在水平方向和竖直方向同时居中，不改变图片大小 |
| fill              | 图片填充容器，默认值                               |
| clip_vertical     | 竖直方向剪切，很少使用                             |
| clip_horizontal   | 水平方向剪切，很少使用                             |

**android:mipMap**
  纹理映射处理技术，不太懂，不过一般也不用，默认为false
**android:tileMode**
  平铺模式。共有以下几个值
  disabled ：默认值，表示不使用平铺
  clamp ：复制边缘色彩
  repeat ：X、Y 轴进行重复图片显示，也就是我们说要说的平铺
  mirror ：在水平和垂直方向上使用交替镜像的方式重复图片的绘制
  三者区别如下图：
![img](%E5%AE%89%E5%8D%93Drawable/20160814223852655.png)
  BitmapDrawable的xml使用方式比较简单，我们这里就不贴案例了哈。接下来我们来看看在代码中如何使用BitmapDrawable。
![img](%E5%AE%89%E5%8D%93Drawable/20160817080954451.png)
  实际上我们从BitmapDrawable的源码可以看出，目前Google建议我们创建BitmapDrawable的构造方法有3种

```
public BitmapDrawable(Resources res, Bitmap bitmap) 

public BitmapDrawable(Resources res, String filepath)

public BitmapDrawable(Resources res, java.io.InputStream is)12345
```

  参数比较简单，res就是我们通过getResource()获取到的资源管理对象，bitmap就是我们需要用BitmapDrawable包装的图片对象，filepath，需要包装的图片所在路径，is则是一个图像流，需要转换成 BitmapDrawable。但是在大多数情况下我们还是建议使用xml实现比较好，代码实现我们不打算深究，我们这里直接给出一个代码应用案例：

```java
Bitmap mBitmap = BitmapFactory.decodeResource(getResources(), R.drawable.image1);
BitmapDrawable mBitmapDrawable = new BitmapDrawable(getResources()，mBitmap);
mBitmapDrawable.setTileModeXY(TileMode.MIRROR, TileMode.MIRROR);//平铺方式
mBitmapDrawable.setAntiAlias(true);//抗锯齿
mBitmapDrawable.setDither(true);//防抖动
//设置到imageView上即可
imageView.setImageDrawable(mBitmapDrawable);1234567
```

#### 2、NinePatchDrawable

  NinePatchDrawable表示的是我们熟悉的.9格式的图片，.9图片可以在保证图片不失真的情况下任意进行缩放，在实际的使用中我们也是通过Xml来实现即可：

```xml
<nine-patch xmlns:android="http://schemas.android.com/apk/res/android"   
    android:src="drawable/resource"  
    android:dither="[true|false]"/>  123
```

  属性和BitmapDrawable中属性的含义相同，这里不过多描述。一般情况下不建议代码创建.9图，因为Android虽然可以使用Java代码创建NinePatchDrawable，但是极少情况会那么做，这是因为由于Android SDK会在编译工程时对点九图片进行编译，形成特殊格式的图片。使用代码创建NinePatchDrawable时只能针对编译过的点九图片资源，对于没有编译过的点九图片资源都当做BitmapDrawable对待。还有点需要特别注意的是，点九图只能适用于拉伸的情况，对于压缩的情况并不适用，如果需要适配很多分辨率的屏幕时需要把点九图做的小一点。

#### 3、ShapeDrawable

  ShapeDrawable对于Xml的shape标签，在实际开发中我们经常将其作为背景图片使用，因为ShapeDrawable可以帮助我们通过颜色来构造图片，也可以构造渐变效果的图片，总之，ShapeDrawable足矣满足我们大部分特殊需求下面我们说说其使用方法：

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape=["rectangle" | "oval" | "line" | "ring"] >
    <corners
        android:radius="integer"
        android:topLeftRadius="integer"
        android:topRightRadius="integer"
        android:bottomLeftRadius="integer"
        android:bottomRightRadius="integer" />
    <gradient
        android:angle="integer"
        android:centerX="integer"
        android:centerY="integer"
        android:centerColor="integer"
        android:endColor="color"
        android:gradientRadius="integer"
        android:startColor="color"
        android:type=["linear" | "radial" | "sweep"]
        android:usesLevel=["true" | "false"] />
    <padding
        android:left="integer"
        android:top="integer"
        android:right="integer"
        android:bottom="integer" />
    <size
        android:width="integer"
        android:height="integer" />
    <solid
        android:color="color" />
    <stroke
        android:width="integer"
        android:color="color"
        android:dashWidth="integer"
        android:dashGap="integer" />
</shape>123456789101112131415161718192021222324252627282930313233343536
```

  从代码中我们可以看出Shape的子元素包括、<gradient>、<padding>、<size>、<solid>、<stroke>，我们一个个分析。
**android:shape**
  这个属性表示图像的形状，可以是rectangle（矩形）、oval（椭圆）、line（横线）、ring（圆环）。默认为rectangle。
这里对于ring值还有几个相关的属性：

| 属性                     | 含义                                                         |
| ------------------------ | ------------------------------------------------------------ |
| android:innerRadius      | 圆环的半径与android:innerRadiusRatio同时存在时，以android:innerRadius 为准 |
| android:innerRadiusRatio | 内半径占整个Drawable宽度的比例，默认值为9.如果为n，那么半径=宽度/n |
| android:thickness        | 圆环的厚度，即外半径减去内半径的大小与android：thicknessRatio同时存在时以android:thickness为准 |
| android：thicknessRatio  | 厚度占整个Drawable宽度比例，默认值为3，如果为n，那么厚度=宽度/n |
| android:useLevel         | 一般都应该使用false，否则可能无法达到预期显示效果，除非它被当做LevelListDrawable来使用。 |

****
  指定边角的半径，数值越大角越圆，数值越小越趋近于直角，参数为：

```xml
<corners
android:radius="integer"
android:topLeftRadius="integer"
android:topRightRadius="integer"
android:bottomLeftRadius="integer"
android:bottomRightRadius="integer" />123456
```

   Android:radius直接指定4个角的半径，另外4个属性可以单独设置4个角的角度.

****
  设置颜色渐变与<solid>为互斥标签，因为solid表示纯色填充，而gradient表示渐变填充。

| 属性                   | 含义                                                         |
| ---------------------- | ------------------------------------------------------------ |
| android:angle          | 渐变的角度，默认为0，其值务必为45°的倍数，0表示从左到右，90表示从下到上。 |
| android:centerX        | 渐变中心点的横坐标                                           |
| android:centerY        | 渐变的中心点的纵坐标，渐变中心点会影响渐变的具体效果。       |
| android:startColor     | 渐变的开始颜色                                               |
| android:centerColor    | 渐变的中间颜色                                               |
| android:endColor       | 渐变的结束颜色                                               |
| android:gradientRadius | 渐变的半径，当android:type=”radial”有效                      |
| android:useLevel       | 一般为false                                                  |
| android:type           | 渐变类别，linear(线性)为默认值，radial（径内渐变），sweep（扫描渐变） |

  angle=0和angle=90的区别(都为线性渐变)：
![img](%E5%AE%89%E5%8D%93Drawable/20160817223030536.png)
  linear(线性)为默认值，radial（径内渐变），sweep（扫描渐变）区别如下：
![img](%E5%AE%89%E5%8D%93Drawable/20160817224001080.png)
  到这里我们利用前面的介绍的知识点来实现一个环形进度圈的案例，我们将shape属性设置为ring（圆环），然后再设置其内半径以及环的厚度，并设置渐变色调，shape_drawable.xml代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:innerRadius="20dp"
    android:shape="ring"
    android:thickness="8dp"
    android:useLevel="false" >

    <gradient android:angle="0"
        android:startColor="@color/normal"
        android:centerColor="#5027844F"
        android:endColor="#fff"
        android:useLevel="false"
        android:type="sweep"
        />
</shape>123456789101112131415
```

效果如下：
![img](%E5%AE%89%E5%8D%93Drawable/20160817215859953.png)
接着，我们将该自定义环形圈设置给一个旋转动画，并利用该旋转动画自定义成一个环形进度圈的style，最后将该自定义的style赋值给Progress组件。代码如下：
自定义旋转动画progress_rotate.xml：

```xml
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/shape_drawable"
    android:pivotX="50%"
    android:pivotY="50%"
    android:fromDegrees="0"
    android:toDegrees="360"
    >
</rotate>123456789
```

自定义Progress的style：

```xml
<style name="CustomProgressStyle" >
    <item name="android:indeterminateDrawable">@drawable/progress_rotate</item>
    <item name="android:minWidth">72dp</item>
    <item name="android:maxWidth">72dp</item>
    <item name="android:minHeight">72dp</item>
    <item name="android:maxHeight">72dp</item>
</style>1234567
```

应用到Progress组件

```xml
<ProgressBar
     android:layout_width="100dp"
     android:layout_height="100dp"
     android:layout_centerInParent="true"
     style="@style/CustomProgressStyle"
     android:indeterminateDuration="700"
     />1234567
```

效果如下：
![img](%E5%AE%89%E5%8D%93Drawable/20160817220944354.gif)

****
  表示纯色填充，通过android:color设置颜色即可。
****
描述边框，属性如下：

| 属性              | 含义                                                     |
| ----------------- | -------------------------------------------------------- |
| android:width     | 描述边框的宽度，数值越大，越边框越厚                     |
| android:color     | 边框的颜色                                               |
| android:dashWidth | 组成虚线的线段宽度                                       |
| android:dashGap   | 组成虚线的线段之间的间隔，间隔越大，虚线看起的间隙就越大 |

  有点要明白的是android:dashWidth和android:dashGap有任意一个为0，则虚线无法预期显示。

****
  表示内容或子标签边距，4个属性top、bottom、left、right，需要注意的是这个标签的作用是为内容设置与当前应用此shape的View的边距，而不是设置当前View与父元素的边距。

****
  设置背景大小，width和height俩属性。一般来说这个值不是shape的最终显示大小，因为shape作为背景时会根据View的大小而填充其背景，因此Shape的大小很多时候是View的大小决定的。
  到这里，shapeDrawable的基本属性我们都介绍完了，下面我们来实现一个比较常见的效果，我们在微信朋友圈点赞或者发布评论时总会出现一个红色带数字的小圆圈提示，嗯，我们就来模仿一下这个效果的实现,首先我们必须把shape属性设置为oval，并设置其纯填充颜色为红色，给一个临时大小宽高大小相同（之所以称为临时大小，是因为其最终大小由使用的View决定的），这样一个圆形背景图就出现啦。代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval"
    >
    <solid android:color="#D90E0E" />
    <size android:height="10dp" android:width="10dp" />
</shape>1234567
```

  接着应用到我们的TextView属性中

```xml
<TextView
    android:layout_width="30dp"
    android:layout_height="30dp"
    android:layout_centerInParent="true"
    android:gravity="center"
    android:textColor="#fff"
    android:text="99"
    android:background="@drawable/shape_circle_number"
    />123456789
```

  最终效果如下：
  ![img](%E5%AE%89%E5%8D%93Drawable/20160817231024859.png)
  实际上在开发中我们经常会利用shapeDrawable来自定义出所需要的各种背景图像或者显示图片，同时也有益于减少对美工图片的依赖，另外一个好处通过自定义shapeDrawable图片会比美工图片的size小很多，这样我们就能减少不必要的size，以减轻apk的size，可谓两全其美，因此能用shapeDrawable定义图像时，应该尽量使用它。

#### 4、LayerDrawable

  一个LayerDrawable是一个可以管理一组drawable对象的drawable。在LayerDrawable的drawable资源按照列表的顺序绘制，列表的最后一个drawable绘制在最上层。LayerDrawable对于xml的标签是<layer-list>其语法如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list
xmlns:android="http://schemas.android.com/apk/res/android" >
<item
    android:drawable="@[package:]drawable/drawable_resource"
    android:id="@[+][package:]id/resource_name"
    android:top="dimension"
    android:right="dimension"
    android:bottom="dimension"
    android:left="dimension" />
</layer-list>1234567891011
```

  一个layer-list可以包含多个item，而每个item则表示一个Drawable。下面我们来说明一下item的一些属性
**android:id**
  资源ID，一个为这个item定义的唯一的资源ID。 使用:”@+id/name”.这样的方式。可以检索或修改这个drawable通过下面的方式：View.findViewById() or Activity.findViewById().
**android:top**
  Integer，Drawable相对于View的顶部的偏移量，单位像素
**android:right**
  Integer，Drawable相对于View的右边的偏移量，单位像素
**android:bottom**
  Integer，Drawable相对于View的底部的偏移量，单位像素
**android:left**
  Integer，Drawable相对于View的左边的偏移量，单位像素
**android:drawable**
  Drawable资源，可以引用已有的drawable资源，也可在item中自定义Drawable。默认情况下，layer-list中的Drawable都会被缩放至View的大小，因此在必要的情况下，我们可以使用android:gravity属性来控制图片的展示效果，防止图片变形或者被过度拉伸。
  下面我们来利用layer-list的叠层效果实现一个文本输入框的底部横线背景。
![img](%E5%AE%89%E5%8D%93Drawable/20160815225834972.png)
xml 代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item>
      <shape android:shape="rectangle">
          <solid android:color="@color/colorAccent" />
      </shape>
    </item>

    <item android:bottom="6dp">
        <shape android:shape="rectangle">
            <solid android:color="#ffffff"/>
        </shape>
    </item>

    <item android:bottom="2dp"
          android:left="2dp"
          android:right="2dp">
        <shape android:shape="rectangle">
            <solid android:color="#ffffff" />
        </shape>
    </item>
</layer-list>12345678910111213141516171819202122
```

  应用到EditText上的代码：

```xml
<EditText                                        
  android:layout_width="200dp"                 
  android:layout_height="wrap_content"         
  android:layout_centerInParent="true"         
  android:background="@drawable/layer_drawable"
  />    123456
```

  上面代码比较简单，我们就不过的分析，接着我们再利用<layer-list>标签来实现一个带阴影的圆角矩形，layer_list_drawable_2.xml代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 灰色阴影 内容距离左边2dp，距离顶部4dp-->
    <item
        android:left="2dp"
        android:top="4dp">
        <shape>
            <solid android:color="@android:color/darker_gray" />
            <corners android:radius="10dp" />
        </shape>
    </item>
    <!-- 白色前景 内容距离底部4dp 右边2dp-->
    <item
        android:bottom="4dp"
        android:right="2dp">
        <shape>
            <solid android:color="#FFFFFF" />
            <corners android:radius="10dp" />
        </shape>
    </item>
</layer-list>123456789101112131415161718192021
```

  效果如下：
  ![img](%E5%AE%89%E5%8D%93Drawable/20160817233846388.png)
当然我们也可以在代码中实现这里仅给出示例，不深究，还是建议采用xml的方式定义

```java
Bitmap bitmap=BitmapFactory.decodeResource(getResources(), R.drawable.image1);
Drawable[] drawables=new Drawable[3];
drawables[0]=new BitmapDrawable(bitmap);
drawables[1]=new BitmapDrawable(bitmap);
drawables[2]=new BitmapDrawable(bitmap);
LayerDrawable layer=new LayerDrawable(drawables);
//设置图层边界距离
layer.setLayerInset(0, 20, 20, 0, 0);
layer.setLayerInset(1, 40, 40, 0, 0);
layer.setLayerInset(2, 60, 60, 0, 0);
ImageView imageView=(ImageView)findViewById(R.id.imgView);
imageView.setImageDrawable(layer);123456789101112
```

  这样我们的带阴影的圆角矩形就出来啦，可以将其作为其他View的背景使用，美工也就不用提供类似的图了，到这里我们应该已经体会带巧用各种Drawable的威力了。

#### 5、StateListDrawable

  StateListDrawable对于xml的<selector>标签，这个标签可以说是我们最常用的标签了，在开发中，有时候我们需要一个View在点击前显示某种状态，而在点击后又切换到另外一种状态，这时我们就需要利用<selector>标签来实现啦。如下案例，我们在点击输入邮件地址前文本框底线是灰色，而在点击后文本框底线就变成蓝色了，这也是<selector>标签的应用之一。
  ![img](%E5%AE%89%E5%8D%93Drawable/20160815231047102.gif)
  StateListDrawable本身也是表示Drawable的集合，每个Drawable就对于View的一种状态，如上面的灰色底线和蓝色底线对应着View的两种V不同时期的状态，因此我们经常使用StateListDrawable来设置View的背景，以便在不同状态下显示不同的效果，从而获得更优的用户体验。其主要语法如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android"
    android:constantSize=["true" | "false"]
    android:dither=["true" | "false"]
    android:variablePadding=["true" | "false"] >
    <item
        android:drawable="@[package:]drawable/drawable_resource"
        android:state_pressed=["true" | "false"]
        android:state_focused=["true" | "false"]
        android:state_hovered=["true" | "false"]
        android:state_selected=["true" | "false"]
        android:state_checkable=["true" | "false"]
        android:state_checked=["true" | "false"]
        android:state_enabled=["true" | "false"]
        android:state_activated=["true" | "false"]
        android:state_window_focused=["true" | "false"] />
</selector>1234567891011121314151617
```

item的属性介绍如下：

| 属性                         | 含义                                             |
| ---------------------------- | ------------------------------------------------ |
| android:drawable             | 该状态下要显示的图像，可以是Drawable也可以是图片 |
| android:state_pressed        | 表示是否处于被按下状态                           |
| android:state_focused        | 表示是否已得到焦点状态                           |
| android:state_hovered        | 表示光标是否停留在View的自身大小范围内的状态     |
| android:state_selected       | 表示是否处于被选中状态                           |
| android:state_checkable      | 表示是否处于可勾选状态                           |
| android:state_checked        | 表示是否处于已勾选状态，一般用于CheckBox         |
| android:state_enabled        | 表示是否处于可用状态                             |
| android:state_active         | 表示是否处于激活状态                             |
| android:state_window_focused | 表示是否窗口已得到焦点状态                       |

selector标签的属性含义如下：
**android:constantSize**
  StateListDrawable的固有大小是否随着其状态改变而改变，因为在状态改变后，StateListDrawable会切换到别的Drawable，而不同的Drawable其大小可能不一样。true表示大小不变，这时其固有大小是内容所有Drawable的固有大小的最大值。false则会随着状态改变而改变，默认值为false
**android:variablePadding**
  表示 StateListDrawable的padding是否随状态的改变而改变，默认值为false，一般建议设置为false就行。
**android:dither**
  是否开启抖动效果，开启后可使高质量的图片在低质量的屏幕上仍然有较好的显示效果，一般建议开启，设置为true。
接下来我们来看一个例子，按钮点击前后状态改变。代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!--获取焦点状态-->
    <item
    android:state_focused="true"
    android:drawable="@color/color_state"
    />

    <!--按下状态-->
    <item android:state_pressed="true"
    android:drawable="@color/color_state" />

    <!--默认状态下-->
    <item android:drawable="@color/normal" />
</selector>123456789101112131415
```

接着应用到button上：

```xml
<Button
       android:padding="8dp"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:layout_centerInParent="true"
       android:text="Selector_State"
       android:textColor="#fff"
       android:background="@drawable/selector_drawable"
       />123456789
```

效果如下：
![img](%E5%AE%89%E5%8D%93Drawable/20160816083046426.gif)
  上面是通过颜色定义不同状态下的显示 效果，当然我们也可以利用shapeDrawable定义各种背景图像然后应用到StateListDrawable中，下面我们定义两个不同状态下的圆角矩形，并应用到button上
shape_drawable_for_btn_normal.xml代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle"
    >
    <solid android:color="@color/normal"></solid>
    <corners android:radius="8dp" />
</shape>1234567
```

shape_drawable_for_btn_press.xml代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle"
    >
    <solid android:color="@color/color_state"></solid>
    <corners android:radius="8dp" />
</shape>1234567
```

selector_for_btn.xml代码如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">

    <!--获取焦点状态-->
    <item
        android:state_focused="true"
        android:drawable="@drawable/shape_drawable_for_btn_press"
        />

    <!--按下状态-->
    <item android:state_pressed="true"
        android:drawable="@drawable/shape_drawable_for_btn_press" />

    <!--默认状态下-->
    <item android:drawable="@drawable/shape_drawable_for_btn_normal" />
</selector>12345678910111213141516
```

应用到button上

```xml
<Button
       android:padding="8dp"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:layout_centerInParent="true"
       android:text="Selector_State"
       android:textColor="#fff"
       android:background="@drawable/selector_for_btn"
       />123456789
```

最终效果如下：
![img](%E5%AE%89%E5%8D%93Drawable/20160818081248786.gif)
最后给出一个通过代码实现的案例给大家参考（建议尽量使用xml定义，代码定义比较复杂）：

```java
/** 设置Selector。 */
    public static StateListDrawable newSelector(Context context, int idNormal, int idPressed, int idFocused,
                                                int idUnable) {
        //相当于<selector>标签
        StateListDrawable bg = new StateListDrawable();
        Drawable normal = context.getResources().getDrawable(R.drawable.shape_drawable_for_btn_normal);
        Drawable pressed = context.getResources().getDrawable(R.drawable.shape_drawable_for_btn_press);
        Drawable focused =context.getResources().getDrawable(R.drawable.shape_drawable_for_btn_press);
        Drawable unable = context.getResources().getDrawable(R.drawable.shape_drawable_for_btn_unable);
        //设置每种状态下的Drawable显示
        // View.PRESSED_ENABLED_STATE_SET
        bg.addState(new int[] { android.R.attr.state_pressed, android.R.attr.state_enabled }, pressed);
        // View.ENABLED_FOCUSED_STATE_SET
        bg.addState(new int[] { android.R.attr.state_enabled, android.R.attr.state_focused }, focused);
        // View.ENABLED_STATE_SET
        bg.addState(new int[] { android.R.attr.state_enabled }, normal);
        // View.FOCUSED_STATE_SET
        bg.addState(new int[] { android.R.attr.state_focused }, focused);
        // View.WINDOW_FOCUSED_STATE_SET
        bg.addState(new int[] { android.R.attr.state_window_focused }, unable);
        // View.EMPTY_STATE_SET
        bg.addState(new int[] {}, normal);
        return bg;
    }123456789101112131415161718192021222324
```

#### 6、LevelListDrawable

  LevelListDrawable对应于<level-list>标签，也表示一个Drawable的集合，但集合中的每个Drawable都一个等级。根据不同等级，LevelListDrawable会切换到相应的Drawable。语法如下：

```xml
<?xml version="1.0" encoding="utf-8"?>
<level-list
    xmlns:android="http://schemas.android.com/apk/res/android" >
    <item
        android:drawable="@drawable/drawable_resource"
        android:maxLevel="integer"
        android:minLevel="integer" />
</level-list>12345678
```

属性说明如下：

| 属性             | 含义                   |
| ---------------- | ---------------------- |
| android:drawable | 该等级下需要展示的图片 |
| android:maxLevel | 该项所允许的最大level  |
| android:minLevel | 该项所允许的最小level  |

  实际上我们也很容易知道<level-list>标签中的每个Item各表示一个Drawable，并有与之对应的等级，而等级则是由android:maxLevel和android:minLevel所决定的，其等级范围是0-10000，最小为0，默认值，最大则为10000，还是一样的做法，先来看一个案例：

```xml
<?xml version="1.0" encoding="utf-8"?>
<level-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/image4"
        android:maxLevel="0"
        />

    <item android:drawable="@drawable/image1"
          android:maxLevel="1"
        />

    <item android:drawable="@drawable/image2"
        android:maxLevel="2"
        />

    <item android:drawable="@drawable/image3"
        android:maxLevel="3"
        />
</level-list>123456789101112131415161718
```

  我们定义了4个item，等级分别为0，1，2，3，它们都有与之对应的Drawable，然后我们在java代码中实现一个效果，每过2秒更好一个不同等级的图片，代码如下：

```java
package com.zejian.drawble;

import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.widget.ImageView;

public class MainActivity extends AppCompatActivity {
    private static ImageView imageView;

    static Handler handler = new Handler(){
        @Override
        public void handleMessage(Message msg) {
            if(msg.what==1){
                imageView.getDrawable().setLevel(1);
            }else if(msg.what==2){
                imageView.getDrawable().setLevel(2);
            }else if(msg.what==3){
                imageView.getDrawable().setLevel(3);
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageView=(ImageView)findViewById(R.id.image);
        imageView.setImageResource(R.drawable.level_list_drawable);
        imageView.setImageLevel(0);

        for (int i=1;i<4;i++){
            handler.sendEmptyMessageDelayed(i,i*2000);
        }
    }
}12345678910111213141516171819202122232425262728293031323334353637
```

实现效果如下：
![img](%E5%AE%89%E5%8D%93Drawable/20160816085811829.gif)
  实际上我们还可以设置等级范围，当等级在某个范围内时去显示对应范围内的图片，这也是可以的。这个比较简单，这里就不演示了哈。