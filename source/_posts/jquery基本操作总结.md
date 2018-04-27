---
title: jquery基本操作总结
date: 2018/4/26
categories: jquery
---

<center>![2018/4/23 阴天](https://imgsa.baidu.com/forum/w%3D580/sign=4cebdabbd31373f0f53f6f97940d4b8b/4beffffe9925bc31c1c843585cdf8db1ca137077.jpg)

<center>[资料来源](http://www.w3school.com.cn/jquery/jquery_dom_get.asp)
### <center> 《jQuery 选择器》
<center>jquery元素选择器和属性选择器允许您通过标签名、属性名或内容对HTML元素进行选择。
<center>- jquery元素选择器

##### 使用CSS选择器来选取HTML元素
```bush
$("p")选取了p元素
$("#intro")选取了id=”intro“的元素
$(".intro")选取了class="intro"的元素
```
#####  jquery属性选择器

使用XPath表达式来选择带有给定属性的元素。
```bash
$("[href]")选取所有带有href属性的元素。
```
#####  jqueryCSS选择器

可用于改变HTML元素的CSS属性
```bush
$("p").css("background-color","red");
```
### <center> 《jQuery 事件》
$(document).ready(function) 将函数绑定到文档的就绪事件（document是固定的）
$(selector).click(function) 触发或将函数绑定到被选元素的点击事件
$(selector).dblclick(function)触发或将函数绑定到被选元素的双击事件
$(selector).focus(function)触发或将函数绑定到被选元素的获得焦点事件
$(selector).mouseover(function)触发或将函数绑定到被选元素的鼠标悬停事件

### <center> 《jQuery HTML》
##### jQuery获取和设置
jquery中非常重要的部分就是操作DOM的能力。

##### jquery text()和jquery html()方法可以来获得所选择的元素的内容

##### jquery val()方法可以获得输入字段的值
```bush
$("#btn").click(function(){
alert("Value:" + $("#test").val());
})
```
通过上面的方法，可以得到Input框中的value值。

##### jquery attr()方法用来获取属性值
```bush
$("button").click(function(){
alert($("#a").attr("href"));
})
```
通过上面的方法，可以得到a标签中href属性值

以上方法，当参数为空时则是获取功能，当传参时为设置功能。其中attr方法设置的时候，参数是键值对的方式，需要一个属性值和一个内容来对应。

##### jquery添加和删除
- append()，在被选元素的结尾插入内容
- prepend()，在被选元素的开头插入内容
- after()，在被选元素之后插入内容
- before()，在被选元素之前插入内容
