---
title: filter()实现模糊的三种方式
data: 2018/4/26
categories: CSS
---
![2018/4/17 晴天](https://upload-images.jianshu.io/upload_images/8542482-2ef43dd9c47ca960.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

<center> filter:blur(2px) 是实现背景模糊的主要属性，下面来讲述filter实现三种效果的方法。
<center> blur里的参数是设定高斯函数的标准差，或者说是屏幕上以多少像素融在一起，所以值越大，越模糊。

### <center>普通背景模糊</center>
<center> filter属性会使整个div的后代模糊并且还会出现白边，如果想让div里的子元素不模糊，***怎么办呢？***
<center> 可以使用伪元素，既解决了模糊问题，也解决了白边问题。

- 实现思路：
在父容器中设置背景，并且使用相对定位，方便伪元素重叠；在：after中只需要继承背景，并且设置模糊，绝对定位覆盖父元素即可，这样父容器中的子元素便可不受模糊度影响。因为伪元素的模糊度不能被父元素的子代继承。

- 代码如下：

```bush
html
<div class="bg">
  <div class="drag">like window</div>
</div>

css
.bg{
  with:100%;
  hight:100%;
  position:relative;
  background:url("picture") no-repeat fixed;
  padding:1px;
  box-sizing:border-box;
  z-index:1;
}
.bg:after{
  content:"";
  width:100%;
  height:100%;
  position:absolute;
  left:0;
  top:0;
  background:inherit;
  filter:blur(2px);
  z-index:2;
}
.drag{
position:absolute;
left:50%;
top:50%;
transform:translate(-50%,-50%);
width:200px;
height:200px;
text-align:center;
z-index:11;
}
```

***这里需要注意三个地方：***
1 子代元素也需要使用绝对定位
2 需要用z-index确定层级关系
3 保证子代元素的层级最高


### <center> 背景局部模糊
背景局部模糊不需要设置父元素的伪元素为模糊，直接在你想模糊的元素外面套一个div，给该元素的伪元素模糊即可。
```bush
html部分
<div class="bg">
  <div class="drag">
    <div>like window</div>
  </div>
</div>
css部分
.bg{
width:100%;
height:100%;
background:url("picture") no-repeat fixed;
padding:1px;
box-sizing:border-box;
z-index:1;
}
.drag{
margin:100px auto;
width:200px;
height:200px;
background:inherit;
position:relative;
}
.drag >div{
width:100%;
height:100%;
text-align:center;
line-height:200px;
position:absolute;
left:0;
top:0;
z-index:11;
设置子元素的层级高一些，不会被drag覆盖
}
.drag:after{
content:"";
width:100%;
height:100%;
position:absolute;
left:0;
top:0;
background:inherit;
filter:blur(15px);
z-index:2;
}
```

### <center> 背景局部清晰
设置父元素的伪元素的filter，子元素的z-index高于父元素。
```bush
.bg{
width:100%;
height:100%;
position:relative;
background:url(picture) no-repeat fixed;
padding:1px;
box-sizing:border-box;
}
.bg:after{
content:"";
width:100%;
height:100%;
position:absolute;
left:0;
top:0;
background:inherit;
filter:blur(3px);
z-index:1;
}
.drag{
position:absolute;
left:40%;
top:30%;
width:200px;
height:200px;
text-align:center;
background:inherit;
z-index:11;
}
```






