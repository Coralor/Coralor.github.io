---
title: background的属性整理
date: 2022-11-21 02:13:56
tags: css
---

##  background-image
语法：background-image: url() | none | inherit;
1.url()：指向图像的路径。
2.none：默认值。不显示背景图像。
3.inherit：规定应该从父元素继承 background-image 属性的设置。
**如果定义了多组背景图，且背景图之间有重叠，写在前面的将会盖在写在后面的图像之上。**

## background-position
语法：background-position:<position> [<position>]，第二个值可选；第一个值规定水平位置，第二个值规定垂直位置

1. 关键词top、bottom、left、right、center：规定图像分别从顶部、底部、左边、右边和中间开始填充，如果只规定了第一个position，那么第二个值为center；
2. percentage：用百分比指定背景图像填充的位置，可以为负值。如果只规定了第一个值，那么第二个值为50%；
3. length：用长度值（可以是任何单位）指定背景图像填充的位置。可以为负值。如果只规定了第一个值，那么第二个值为50%。

在css3中，可以为background-position提供四个参数值。
如果提供四个，每个percentage或length值的前面必须提供关键词（即top、bottom、left、right、center）来指定相对偏移的方向，比如：

```css
    #bg{
        width: 500px;
        height: 300px;
        margin:0 auto;
        padding: 50px; 
        border:10px dashed rgba(255,0,0,.7);
        background: #0ff url('https://p3.music.126.net/ZjlPU7akZZYlscvnLS9NjA==/18668607929720106.jpg') no-repeat;
        background-size: 100px;
        background-position: right 50px bottom 50px;  // 相对右边界向左偏移50px，相对底边界向上偏移50px(距离右边界50px, 下边界50px)
    }
```

如果是用长度值来规定背景图片的位置，那么这个长度值的度量方式是：图片的边缘线到指定偏移方向的距离
如果使用百分比来规定图片背景的位置，那么这个百分比的度量方式为：相对于图片的百分比位置到边界的距离等于相对于这个块元素的百分比距离

例如： background-position: 50% 50%; 
**就是说相对于图片水平和垂直都是50%的位置（即图片的中心点），这个中心点到左边界和上边界的距离都是这个块的宽和高（不包括边框）的50%。**

## background-size
语法：background-position:length|percentage|cover|contain;，可设置两个值，第二个值可选；第一个值设置宽度，第二个值设置高度；
1. length：设置背景图像的高度和宽度。可以是任何单位。如果只设置一个值，则第二个值会被设置为 "auto"；
2. percentage：以父元素的百分比来设置背景图像的宽度和高度。如果只设置一个值，则第二个值会被设置为 "auto"；（即图片的宽和高是容器的宽和高的x%）
3. cover：**将背景图像等比缩放到完全覆盖容器，背景图像有可能超出容器。** （即铺满容器，背景图可能不完整）
4. contain：**将背景图像等比缩放到宽度或高度与容器的宽度或高度相等，背景图像始终被包含在容器内。**（即背景图完整，但容器未必铺满）

## background-repeat
语法：background-position:<repeat-style> [<repeat-style>];，可设置两个值，第二个值可选；第一个用于横向，第二个用于纵向。默认值：repeat
1.repeat-x：背景图像在横向上平铺
2.repeat-y：背景图像在纵向上平铺
3.repeat：背景图像在横向和纵向平铺
4.no-repeat：背景图像不平铺
5.round：背景图像自动缩放直到适应且填充满整个容器。（CSS3）
6.space：背景图像以相同的间距平铺且填充满整个容器或某个方向。（CSS3）


## background-origin
语法：background-origin: padding-box|border-box|content-box; 默认值：padding-box

1.border-box：从border区域（含border）开始显示背景图像。
2.padding-box：从padding区域（含padding）开始显示背景图像。
3.content-box：从content区域开始显示背景图像。


## background-attachment
语法：background-attachment : fixed | scroll | local; 默认值：scroll

1.fixed：背景图像相对于窗体固定。
2.scroll：背景图像相对于元素固定，也就是说当元素内容滚动时背景图像不会跟着滚动，因为背景图像总是要跟着元素本身。但会随元素的祖先元素或窗体一起滚动。
3.local：背景图像相对于元素内容固定，也就是说当元素随元素滚动时背景图像也会跟着滚动，因为背景图像总是要跟着内容。（CSS3）



<!-- https://www.cnblogs.com/zzsdream/p/13583346.html -->