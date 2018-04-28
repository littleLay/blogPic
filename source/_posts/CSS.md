---
title: CSS知识点
date: 2018/4/14
categories: 小Tips
---
![2018/4/10 晴天](https://imgsa.baidu.com/forum/w%3D580/sign=403828a34dc2d562f208d0e5d71090f3/c6f617fa513d2697a0ee8e9e57fbb2fb4216d8cb.jpg)

### <center> 左右布局：不少于三种方法
圣杯布局和双飞翼布局解决的问题是一样的，就是两边顶宽，中间自适应的三栏布局，中间栏要在放在文档流前面以优先渲染。

圣杯布局和双飞翼布局解决问题的方案在前一半是相同的，也就是三栏全部float浮动，但左右两栏加上负margin让其跟中间栏div并排，以形成三栏布局。

不同在于解决”中间栏div内容不被遮挡“问题的思路不一样：
<!-- more -->
圣杯布局，为了中间div内容不被遮挡，将中间div设置了左右padding-left和padding-right后，将左右两个div用相对布局position: relative并分别配合right和left属性，以便左右两栏div移动后不遮挡中间div。

双飞翼布局，为了中间div内容不被遮挡，直接在中间div内部创建子div用于放置内容，在该子div里用margin-left和margin-right为左右两栏div留出位置。

##### <center> 圣杯模型

<center>两栏和自适应元素都设置同一方向的浮动(如float: left)

<center>父元素设置左右padding为左右边栏的宽度。

<center>自适应元素设置宽度为100%

<center>左边栏margin-left为负100%，再设置relative，最后通过left属性偏移负的自身宽度。

<center>右边栏margin-left为负自身宽度，再设置relative，最后通过right属性偏移负的自身宽度。

##### <center> 双飞翼模式
<center>与圣杯模式相似，只不过少了relative，left，right的步骤和共同父元素，主内容元素多了层父元素，实现思路如下：

<center>main元素设置左右margin值，值为左右两栏的宽度，main父元素设置宽度为100%。
<center>左边栏margin-left为负100%
<center>右边栏margin-left为负自身宽度

##### <center> 两栏布局
利用BFC布局实现：
```
html:
<div class="main">
        <div class="left"></div>
        <div class="right">可爱如我可爱如我可爱如我可爱如我   biubiubiubiubiubiu</div>
</div>

css:
        .main{
            /* float: left; */
            width: 100%;
        }
        .left{
            float: left;
            width: 100px;
            height: 100px;
            background-color: orange;
            opacity: 0.5;
        }
        .right{
            overflow: auto;
            height: 100px;
            background-color: green;
        }
main里注释的那句代码是因为加和不加对这个布局没影响，加的话如果后面还有元素，可能需要视情况考虑清除浮动。
```
该布局利用了float和overflow，overflow在这里的作用是激活BFC布局。

除了BFC，还可以利用float和margin来实现，和上面BFC相比，只是少了一句BFC的激活语句，多了一句  margin-left:100px;

### <center> BFC的功能和特点：

##### <center> 如何激活BFC布局：

```
1.float值不是none
2.position为absolute或者fixed
3.非块级元素的display是其中一个：inline-block，table-cell，table-caption，flex，inline-flex
4.块级元素具有overflow，且值不是visible
```
##### <center> BFC用处
<center>[原文链接](https://segmentfault.com/a/1190000009545742)

###### <center> 清除浮动

```bush
html:
<div class="wrap">
<section>1</section>
<section>2</section>
</div>

css:
.wrap {
  border: 2px solid yellow;
  width: 250px;
}
section {
  background-color: pink;
  float: left;
  width: 100px;
  height: 100px;
}
```
由于子元素都是浮动的，受浮动影响，边框为黄色的父元素的高度坍塌了。
![image](http://upload-images.jianshu.io/upload_images/8542482-098186c529bb638d?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
解决方案：为.wrapper 加上 overflow:hidden，使其形成BFC，根据BFC规则第六条，计算高度时就会计算float的元素的高度，达到清除浮动的效果。
清除后如下：
![image](http://upload-images.jianshu.io/upload_images/8542482-0889c8b286b55825?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
清除浮动原理：触发父div的BFC属性，使下面的子div都处在父div的同一个BFC区域之内，此时已成功清除浮动。

  ###### <center> 自适应两栏布局 
见[CSS + CSS3的问题1](https://www.jianshu.com/p/c1363a5b35fc) 有详细解释
  ###### <center> 防止垂直margin合并
```
html:
<section class="top">1</section>
<section class="bottom">2</section>

css：
section {
  background-color: pink;
  margin-bottom: 100px;
  width: 100px;
  height: 100px;
}
.bottom {
  margin-top: 100px;
}
请注意，这里有一个margin-bottom和margin-top分别都是100px，section的宽高为了好对比也设置的100px
```
![image](http://upload-images.jianshu.io/upload_images/8542482-4ef692e16d388f96?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到，我们实际只能看到两个section之间的高度是100px，这是因为他们的外边距相遇发生了合并。
解决方式：为其中一个元素的外面包裹一层元素，并为这个外层元素设置overflow:hidden，使其形成BFC。因为BFC内部是一个独立的容器，所以不会与外部相互影响，可以防止margin合并。
```
代码如下：
html:
<section class="top">1</section>
<div class="wrap">
<section class="bottom">2</section>
</div>
css多加了.wrap的样式，其他不变。
overflow:hidden
```
![image](http://upload-images.jianshu.io/upload_images/8542482-8237d4573a3db21f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### <center> 对栅格的理解
<center>大部分栅格系统的公式依据：W =（c + g） * N
<center>其中W是Flowline是W，c是column的宽度，g是Gutter（卡槽）的宽度

###### <center> bootsrap的栅格系统：
bootstrap中，必须将列放入row中，而row必须在container中，container类在布局中主要有两个作用：
   1.在不同的宽度区间内（响应式断点）提供宽度限制，当宽度变化时，采用不同的宽度。
  2.提供一个padding，组织内部内容触碰到浏览器边界。

响应式布局概念：为同一个页面设计多种布局形态，分别适配不同屏幕尺寸的设备。

在介绍栅格布局如何实现之前，先介绍一个bootstrap中的栅格原理：
要理解bootstrap的栅格，这其中有两个要点：
 - 容器（container），行（row）和列（column）之间的层级关系，一个正确的写法如下：
```bush
<div class="container">  
    <div class="row">  
        <div class="col-md-6"></div>  
        <div class="col-md-6"></div>  
    </div>  
</div>
```
bootstrap栅格的container，row，column必须保持特定的层级关系，才能正常工作。原因如下：container有15px的水平内边距，row有-15px的水平外边距，column有15px的水平内边距，这些边距是故意的、互相关联的，因此就像齿轮一样，限定了层级结构。这也是bootstrap的精巧之处。
正确的用法如下：
```bush
<div class="container">   
    <div class="row">   
        <div class="col-md-8">   
            <div class="row">   
                <div class="col-md-6"></div>   
                <div class="col-md-6"></div>   
            </div>   
        </div>   
        <div class="col-md-4"></div>   
    </div>   
</div>
```
- 第二个要点，是不同的断点类型的意义及其搭配

bootstrap栅格的column对应的类名形如.col-xx-y，其中y是数字，表示该元素的宽度占据12列中的y列，而xx只有特定的几个值可以选择，分别是xs、sm、md、lg，他们就是断点类型，断点像素值依次增大，其中xs表示极小，即认为视口宽度永远不小于xs断点，column始终水平浮动。
在bootstrap的sass源码中是这样定义栅格的：
```
@include make-grid-columns;   
@include make-grid(xs);   
@media (min-width: $screen-sm-min) {   
  @include make-grid(sm);   
}   
@media (min-width: $screen-md-min) {   
  @include make-grid(md);   
}   
@media (min-width: $screen-lg-min) {   
  @include make-grid(lg);   
}
```
可以看到，用了min-width的写法，而且断点像素值越大的，对应代码越靠后。这就是为什么在实现栅格布局时，只设定最小值，而不设定最大值的原因了。


现在我们可以来了解一下如何实现栅格布局了：
###### <center> 栅格布局如何实现：
- 开始

 实现栅格系统，首要确定两个内容：
  1.我们打算把屏幕分为几类
  2.我们打算支持的列数是多少
我们可以以bootstrap为参照，将屏幕分成四类：超小屏幕（<768px），小屏幕--平板（>=768px），中等屏幕--桌面显示器（>=992px），大屏幕--大屏幕显示器（>=1200px），类前缀分别为：.col-xs-，.col-sm-，.col-md-，.col-lg-，列数也可以定位12。
- 实现

确定好屏幕分类和列数后，就可以开始实现了。
首先给栅格系统的所有class增加浮动 float:left

声明浮动后开始声明宽度，从最小屏幕开始，最小屏幕不需要使用@media来声明。
```
.col-sm-1 {
    width: 8.333333%  // 十二分之一
}
.col-sm-2 {
    width: 16.66667%  // 十二分之二      
}
// ...
.col-sm-12 {
    width: 100%  // 十二分之十二
}
```
其他情况情况也需要声明浮动和宽度，但是需要声明最小屏幕宽度，以中屏举例：
```
注意括号里是 min-width
@media (min-width:992) {
    .col-md-1 {
        width: 8.333333%  // 十二分之一
    }
    .col-md-2 {
        width: 16.66667%  // 十二分之二      
    }
    // ...
    .col-md-12 {
        width: 100%  // 十二分之十二
    }
}
```
其他情况同理，这样我们就能实现一个简单的栅格系统。


### <center> （水平）居中有哪些实现方式
- text-align:center;
将行内元素包裹在一个属性display为block的父层元素中，并且把父层元素添加如下属性即可：
- 块状元素解决方案： margin:0 auto;
- 多个块状元素解决方案
将元素的display属性设置为Inline-block，并且把父元素的text-align属性设置为center。
- 多个块状元素解决方案（使用flexbox布局实现）
使用flexbox布局，只需要把待处理的块状元素的父元素添加属性display：flex及justify-content:center
###### <center> 垂直居中
- 单行的行内元素解决方案
height=line-height

- 多行的行内元素解决方案
组合使用display:table-cell和vertical-align:middle属性来定义需要居中的元素的父容器元素生成效果

- 已知高度的块元素解决方案
top移动50%，之后margin-top移动自身高度的一半

###### <center> 水平垂直居中
- 已知高度和宽度的元素
```bush
.item{
    position:absolute;
    margin:0 auto;
    left:0;
    top:0;
    right:0;
    bottom:0;
}
```
- 已知高度和宽度的元素
```bush
.item{
    position: absolute;
    top: 50%;
    left: 50%;
    margin-top: -75px;  /* 设置margin-left / margin-top 为自身高度的一半 */
    margin-left: -75px;
}
```
- 未知高度和宽度的元素
```bush
.item{
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);  /* 使用css3的transform来实现 */
}
```
- 使用flex布局来实现
```bush
.parent{
    display: flex;
    justify-content:center;
    align-items: center;
    /* 注意这里需要设置高度来查看垂直居中效果 */
    background: #AAA;
    height: 300px;
}
```
### <center> CSS3有哪些新属性？
<br>
 ###### <center>弹性盒子 flex

弹性盒子是CSS3的一种新的布局模式，是一种当页面需要适应不同的屏幕大小以及设备类型时确保元素拥有恰当的行为的布局方式。
弹性盒子由弹性容器和弹性子元素组成，弹性容器通过设置display属性的值为flex或inline-flex将其定义为弹性容易。
弹性容器内包含了一个或多个弹性盒子。
- flex-direction属性
1 row：横向从左到右排列（左对齐），默认的排列方式
2 row-reverse：反转横向排列（右对齐，从后往前排，最后一项排在最前面）
3 column：纵向排列
4 column-reverse：反转纵向排列，从后往前排，最后一项排在最上面

- justify-content属性
1 flex-start：默认值。弹性项目向行头紧挨着填充。
2 flex-end：弹性项目向行尾紧挨着填充
3 center：弹性项目居中紧挨着填充（如果剩余的自由空间是负的，则弹性项目将在两个方向上同时溢出）。
4 space-between：在弹性项目刚好排下的时候该值等同于flex-start，如果弹性项目少的话，则第一个弹性项目的外边距和行的main-start边线对齐，而最后一个弹性项的外边距和行的main-end边线对齐，然后剩余的弹性项分布在该行上，相邻项目的间隔相等。
5 space-round：
如果剩余空间为负或者只有一个弹性项，则该值等同于center。否则，弹性项目沿该行分布，且彼此间隔相等（比如是20px），同时首尾两边和弹性容器之间留有一半的间隔

- align-items属性：设置盒子元素在侧轴（纵轴）方向上的对齐方式。
1 flex-start：弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴起始边界。
2 flex-end：弹性盒子元素的侧轴（纵轴）起始位置的边界紧靠住该行的侧轴结束边界。
3 center：弹性盒子元素在该行的侧轴（纵轴）上居中放置。（如果该行的尺寸小于弹性盒子元素的尺寸，则会向两个方向溢出相同的长度）。
4 baseline：如弹性盒子元素的行内轴与侧轴为同一条，则该值与'flex-start'等效。其它情况下，该值将参与基线对齐。
5 stretch：如果指定侧轴大小的属性值为'auto'，则其值会使项目的边距盒的尺寸尽可能接近所在行的尺寸，但同时会遵照'min/max-width/height'属性的限制。

###### <center> 3D旋转
不同浏览器的动画兼容性都不一样，所以在写旋转属性时考虑多种浏览器的兼容特点。
比如：同样的transform，谷歌是-webkit-transform，火狐是-moz-transfrom。

旋转方法如下：
```
沿X轴旋转
div{
  transform:rotateX(120deg);
  -webkit-transform:rotateX(120deg);
}
沿Y轴旋转
div
{
transform: rotateY(130deg);
-webkit-transform: rotateY(130deg); /* Safari 与 Chrome */
}

另外还可以沿Z轴旋转，这里的x,y,z可以联想一些有趣的内容记住~
比如 正面推倒妹子，就是X，妹子侧躺着欣赏，就是Y，妹子玉体横陈就是Z。
以及定义3D缩放转换scaleX(x)，scaleY(y)，scaleZ(z)
translateX(x)，transformY(y)，transformZ(z)来定义3D转换。
```
##### <center> border-radius属性：
使用该属性可以给任何元素制作“圆角”。

###### <center> @font-face规则：
```
<style>
自定义字体
@font-face
{
font-family: myFirstFont;
src: url(sansation_light.woff);
}
div
{
font-family:myFirstFont;
}
</style>
```
##### <center> 文本效果
文本阴影，text-shadow：5px 5px 5px #ff0000；
盒子阴影，box-shadow：10px 10px 5px #888888；
文本溢出属性，text-overflow
##### <center> 按钮
CSS3的按钮添加了更多颜色或者边框啊，圆润程度的改变，并没有什么太大的亮点。
### <center> margin塌陷
[参考原文](https://juejin.im/entry/56cd377c2e958a69f941f802)
- margin属性，注意几点：
1 如果 margin 的值是百分比，则是相对于父元素的内容盒宽度来计算的，即使 margin-top 和 margin-bottom 也是如此。因此即使父元素的高宽不相等，子元素的 margin 元素指定了相同的百分比值，则子元素各个方向的 margin 计算值都是相等的。
2 margin-top 和 margin-bottom 值对行内非替换元素（non-replaced inline element）是无效的。因此我们可以指定 img 元素的 margin-top 和 margin-bottom，而非替换行内元素（如 i，span 等）设置 margin-top 和 margin-bottom 却不会产生效果。
（非替换元素是指该元素的height和width属性的大小由其内容决定，而不能直接设置）
- 相邻的margin
两个margin符合垂直相连的的四种情况，满足其中之一即可：
1 父元素的 top margin 和第一个子元素的 top margin
2 父元素的bottom margin 和最后一个子元素的 bottom margin
3 元素的 bottom margin 和与这个元素相邻的兄弟元素的 top margin
4  如果一个元素，它没有生成 [BFC](https://link.juejin.im/?target=https%3A%2F%2Fwww.w3.org%2FTR%2FCSS2%2Fvisuren.html%23block-formatting)、没有包含正常流的子元素、`min-height`是0、`height`是0或者 auto，则它的 top margin 和 bottom margin 也是垂直毗连的

如果两个margin满足以下三个条件，我们就说这两个margin是相邻的：
1 这两个margin是垂直相连的，即满足上面四种情况之一
2 margin的两个元素都是正常流的块级元素，并且在同一个BFC中
3 两个margin之间没有行盒、清除浮动后的空隙、padding和border。

- margin折叠
即相邻的margin有可能会被折叠成一个。
比如元素 #a 指定了 margin-bottom 为 10px，而它下方的元素 #b 指定了 margin-top 为 20px，
![image](http://upload-images.jianshu.io/upload_images/8542482-9b2db4dde5f36895.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

元素 #a 的 margin-bottom 和元素 #b 的margin-top 在位置上重叠了，它们之间的距离是 20px，即元素 #b 的 bottom margin 长度，这就是 margin 折叠现象。关于这个现象，可以这么理解：

margin 定义的是它与其他盒子之间的最小间距。其中元素 #a 指定了 margin-bottom 为 10px，就表明它下方的元素 #b 与它至少要有 10px 的距离，它指定的是一个最小值，因此实际的距离可以比这个大。

元素 #a 下方的元素 #b 也设置了 margin-top 为 20px，如果不折叠，则他们之间就有 30px 的距离。如果折叠成了一个 20px 的距离，则对元素 #a 来说，它的 margin-bottom 要求至少要有 10px 的距离，是满足的，而对于元素 #b 来说，它的 margin-top 要求至少要有 20px 的距离，也是满足的。

而 margin 折叠的存在，其实是为了可以在视觉上显得更美观，也更贴近设计师的预期。

-margin的折叠规则
并不是所有的margin都可以折叠，需要满足以下条件：
1 垂直相邻的margin才有可能折叠，水平margin永远不折叠
2 根元素（即html）的margin永远不折叠
3 如果一个元素，它的 top margin 和 bottom margin 是**相邻**的，并且有清除浮动后的空隙（[clearance](https://link.juejin.im/?target=https%3A%2F%2Fwww.w3.org%2FTR%2FCSS2%2Fvisuren.html%23clearance)），这个元素的 margin 可以跟兄弟元素的 margin 折叠，但是折叠后的 margin 不能跟父元素的 bottom margin 折叠。

-折叠后的Margin大小

当两个或者两个以上的 margin 折叠后，margin 的值计算如下：
1 如果 margin 都是正数，则取他们当中的最大值
2 如果 margin 中有正有负，则取最大的正数加上最小的负数（如最大的 margin是 20px，最小的 margin 是 -20px，则他们计算后的值是 0）
3 如果 margin 中都是负数，则取他们当中的最小值

- 如何解决margin塌陷
##### 例1：针对——>“*   两个 margin 之间没有行盒（line box）、清除浮动后的空隙（[clearance](https://link.juejin.im/?target=https%3A%2F%2Fwww.w3.org%2FTR%2FCSS2%2Fvisuren.html%23clearance)）、padding和边框”

如图所示：
![image](http://upload-images.jianshu.io/upload_images/8542482-8bbff93b557f891f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果 #container 没有下边框，则 #container 的 bottom margin 和 #inner 的 bottom margin 是相邻的，因此它们折叠了，并且 #inner 撑开了 #container 元素，所以可以看到 #container 元素的高度变成了 10px，且显示的是 #inner 的红色背景
我们通过增加padding的方式来阻止Margin的折叠：
![image](http://upload-images.jianshu.io/upload_images/8542482-16af8f431f902e8e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
当给 #container 添加一个下边框，两个 margin 之间就边框的阻隔，他们就不相邻了，因此不能折叠。所以可以看到 #container 被撑开成了 20px，其中 10px 是 #inner 的高度，还有 10px 是 #inner 的 bottom margin，并且由于 margin 是透明的，因此 #container 露出了部分蓝色的背景。
##### 例2：针对——>'*   margin 的两个元素都是正常流的块级元素，并且在同一个 [BFC](https://link.juejin.im/?target=https%3A%2F%2Fwww.w3.org%2FTR%2FCSS2%2Fvisuren.html%23block-formatting) 中''

我们通过创建新的BFC来阻止Margin的折叠：

![image](http://upload-images.jianshu.io/upload_images/8542482-5878154dd7007952.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如上图 #container 元素和 #inner 元素同属于一个 [BFC](https://link.juejin.im/?target=https%3A%2F%2Fwww.w3.org%2FTR%2FCSS2%2Fvisuren.html%23block-formatting) 中，#container 的 top margin 和 #inner 的 top margin 折叠，bottom margin 同理。
但如果让 #container 跟 #innter 处在不同的 [BFC](https://link.juejin.im/?target=https%3A%2F%2Fwww.w3.org%2FTR%2FCSS2%2Fvisuren.html%23block-formatting) 中，则 top margin 和 bottom margin 都不会折叠，如：
![image](http://upload-images.jianshu.io/upload_images/8542482-808ef00f2339f4e8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
给 #container 元素增加一个 `overflow: hidden` 属性，让它的内容盒生成一个独立的 [BFC](https://link.juejin.im/?target=https%3A%2F%2Fwww.w3.org%2FTR%2FCSS2%2Fvisuren.html%23block-formatting)，而 #inner 处于这个独立的 [BFC](https://link.juejin.im/?target=https%3A%2F%2Fwww.w3.org%2FTR%2FCSS2%2Fvisuren.html%23block-formatting) 中，因此 #container 和 #inner 就处于两个不同的 [BFC](https://link.juejin.im/?target=https%3A%2F%2Fwww.w3.org%2FTR%2FCSS2%2Fvisuren.html%23block-formatting) 中了，所以他们的 margin 不能折叠。
##### 例3：针对——>"*   如果一个元素，它本身的 top margin 和 bottom margin 是**相邻**的，并且有清除浮动后的空隙（[clearance](https://link.juejin.im/?target=https%3A%2F%2Fwww.w3.org%2FTR%2FCSS2%2Fvisuren.html%23clearance)），这个元素的 margin 可以跟兄弟元素的 margin 折叠，但是折叠后的 margin 不能跟父元素的 bottom margin 折叠"
![image](http://upload-images.jianshu.io/upload_images/8542482-98b84b77a67775dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
给父元素 #container 设置了一个灰色背景，并且没有设置高度，因此高度会随着内容而扩展，margin 设置为 50px。
其中有一个红色的浮动元素 #floated，高宽都设置为 40px。
给 #cleared 设置了 15px 的 margin，并且元素的高度、padding、margin 都为0，因此 #cleared 元素的 top margin 和 bottom margin 是相邻的。这个元素的位置如下图所示：
![image](http://upload-images.jianshu.io/upload_images/8542482-deeeb4d14f094a16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
因为 #cleared 元素清除了左浮动，所以 #cleared 元素下移。
而 #cleared 元素和 #slibling 元素的 margin 折叠了，因此可以看到他们的位置是重叠的。
![image](http://upload-images.jianshu.io/upload_images/8542482-b8ff8e315d9a2e28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
由于这条规则的存在，导致他们折叠后的 margin 不能跟 #container 的 bottom margin 进行折叠，因此 #container 的高度被撑开。

如果没有这条规则，他们还应该跟 #container 的 bottom margin 进行折叠，如：
![image](http://upload-images.jianshu.io/upload_images/8542482-c0cb1715e617542e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
以上这张图，在去掉了 #cleared 元素的 clear 属性之后，就不满足这条规则了，所以可以看到 #container 的高度就只有 40px，即红色的浮动元素的高度，而 #cleared 元素、#sibling 元素、#container 元素的 margin 都折叠成了一个。

[参考源文](http://www.jb51.net/css/362199.html)
[参照源](https://blog.csdn.net/weixin_35955795/article/details/70194764)
[概念源文](http://www.runoob.com/css3/css3-flexbox.html)

作者：知乎用户
链接：https://www.zhihu.com/question/21504052/answer/50053054
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
