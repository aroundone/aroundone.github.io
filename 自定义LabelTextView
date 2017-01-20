# 自定义LabelTextView

项目需要，实现一个圆角矩形加上标签的自定义View。大概的效果是这样的

<img src="http://ojwjax1r0.bkt.clouddn.com/labelTextView.png" width=360 height=180 />

## 难点
* 标签页的范围需要贴合圆角矩形的范围
* 标签内的字体需要和和线平行
* 以及各种坑

## 遇到问题和解决方式

### 标签文字

#### 问题
按照需求，标签的文字需要斜着展示，而默认的方式只有水平展示

#### 解决方式
标签文字的显示之前走了不少弯路，开始想到使用画布的旋转以及计算角度来实现文字的斜着显示.。
但是效果都不好。
后来发现一个更简单的方法，通过下面方法轻松实现
```
canvas.drawTextOnPath(@NonNull String text, @NonNull Path path, float hOffset,
            float vOffset, @NonNull Paint paint)

```

### 圆角矩形和三角形混合绘制

#### 问题
绘制不规则图形，像是一个圆角矩形和一个三角形的超集在减去三角形不相交的那部分

#### 解决方式


而圆角矩形的绘制，我采取了混合模式。

先通过path绘制一个三角形，再通过混合模式将三角形和圆角矩形重合区域之外给遮盖。
我称之为混合模式其实就是使用了方法。
```
path.op(Path path, Path.Op op)
```

借用一张图展示混合模式

<img src="http://ojwjax1r0.bkt.clouddn.com/0_13219433774KaR.gif" width=360 height=680 />

以及小结

```
         android.graphics.PorterDuff.Mode.SRC:只绘制源图像
         android.graphics.PorterDuff.Mode.DST:只绘制目标图像
         android.graphics.PorterDuff.Mode.DST_OVER:在源图像的顶部绘制目标图像
         android.graphics.PorterDuff.Mode.DST_IN:只在源图像和目标图像相交的地方绘制目标图像
         android.graphics.PorterDuff.Mode.DST_OUT:只在源图像和目标图像不相交的地方绘制目标图像
         android.graphics.PorterDuff.Mode.DST_ATOP:在源图像和目标图像相交的地方绘制目标图像，在不相交的地方绘制源图像
         android.graphics.PorterDuff.Mode.SRC_OVER:在目标图像的顶部绘制源图像
         android.graphics.PorterDuff.Mode.SRC_IN:只在源图像和目标图像相交的地方绘制源图像
         android.graphics.PorterDuff.Mode.SRC_OUT:只在源图像和目标图像不相交的地方绘制源图像
         android.graphics.PorterDuff.Mode.SRC_ATOP:在源图像和目标图像相交的地方绘制源图像，在不相交的地方绘制目标图像
         android.graphics.PorterDuff.Mode.XOR:在源图像和目标图像重叠之外的任何地方绘制他们，而在不重叠的地方不绘制任何内容
         android.graphics.PorterDuff.Mode.LIGHTEN:获得每个位置上两幅图像中最亮的像素并显示
         android.graphics.PorterDuff.Mode.DARKEN:获得每个位置上两幅图像中最暗的像素并显示
         android.graphics.PorterDuff.Mode.MULTIPLY:将每个位置的两个像素相乘，除以255，然后使用该值创建一个新的像素进行显示。结果颜色=顶部颜色*底部颜色/255
         android.graphics.PorterDuff.Mode.SCREEN:反转每个颜色，执行相同的操作（将他们相乘并除以255），然后再次反转。结果颜色=255-(((255-顶部颜色)*(255-底部颜色))/255)
```


但是很可惜的是， ==这个方法必须在API>19的时候才行== ，换言之需要采取别的方式来兼容下面的版本。
这个时候我选择了画布裁剪

```
public boolean clipPath(@NonNull Path path, @NonNull Region.Op op)

```

使用裁剪可以解决这个问题。

but，如果这样写会发现这样有一个很严重的bug，无论采取哪种Op模式，全部都不起作用。
这里涉及到在4.1以及一下版本有一个bug，Path相关的绘制会出现问题，包括不限于drawTextOnPath，
clipPath。

解决方法是：解决方法是在自定义View的构造函数里增加这一句话：
```
this.setLayerType(View.LAYER_TYPE_SOFTWARE, null);
```
这里也是参考前人的遇到的坑，之前搜到这个bug心里留了点心眼，没想到第二天就用上了（窃喜）。


### 不能使用drawable资源
这是一个很严重的需求问题。

现在的需求：是当我要点击item的时候，内部的字体和外面的边框颜色需要改变。
这些都是定义在drawable当中。

所以我现在的设计(继承View)都是不明智的。
当使用者使用会出现这种抉择:我需要使用select或者要通过shape绘制图以及press，这样就需要引用drawable资源
可是需要实现这样的话就需要传递引用，自定义View中还要添加额外的方法。

稳妥起见，只有忍痛割爱，重写整个类，让他继承与TextView。使用一个稳定的系统View总比自己写一个不稳定的好。

然后为了边界显示不冲突，将之前绘制圆角矩形的画笔设为透明


### Label文字显示太粗糙

#### 问题

这个时候的显示如下

<img src="http://ojwjax1r0.bkt.clouddn.com/webwxgetmsgimg.jpg" width=180 height=360 />


Label文字默认是在中间的，即两点中线的位置。对于一个经常是宽度大于高度的Item来说不太友善。
稍微完善它

#### 解决方式

没有什么比转化成视觉展示更加直观的方式了

<img src="http://ojwjax1r0.bkt.clouddn.com/shouhui1.jpg" width=360 height=180 />

（请自动脑补顺时针90度旋转上图）


<img src="http://ojwjax1r0.bkt.clouddn.com/shouhui2.jpg" width=360 height=180 />

所以就转化成为了一个初中三角函数的题目

解：省略N步

最后的公式是：

```
      ps: w = width   h = height

      公式为：
      if(w > h) {
           (w^2 - h^2)/[2 * √(w^2 + h^2)]
      } else if(w < h) {
           (h^2 - w^2)/[2 * √(w^2 + h^2)]
      }

```

用一个方法计算它

```
//允许负值
private float calculationOffset(double width, double height) {
        if (width == height) {
            return 0;
        }

        double molecule = Math.pow(width, 2) - Math.pow(height, 2);

        double denominator = 2 * Math.sqrt(Math.pow(width, 2) + Math.pow(height, 2));

        return double2Float(molecule / denominator);
    }

```

具体代码见GitHub




## 参考

http://blog.csdn.net/jiangwei0910410003/article/details/50935468

https://ghui.me/post/2015/10/android-graphics-path/

http://whuhan2013.github.io/blog/2016/05/14/android-canvas-summary/

http://blog.csdn.net/yanzi1225627/article/details/8583066
