---
title: 关于jQuery的CSS原理
date: 2018-04-29 15:50:23
tags: CSS
---
![2018/4/29](https://imgsa.baidu.com/forum/w%3D580/sign=0190f0864b540923aa696376a259d1dc/e15ef4f81a4c510f55d8ecbf6259252dd42aa515.jpg)

```bush
我们都知道在操作DOM元素的CSS样式时，使用jQuery可以很容易地对CSS元素进行操作。
其中jQuery的CSS方法，其底层运作就应用了getComputedStyle和getPropertyValue的方法。
```
下面进入正题，开始一步步分析。

### <center> jQuery获取CSS样式

jQuery对CSS的操作有很多，如下：
```bush
addClass()-向被选元素添加一个或多个类
removeClass()-从被选元素删除一个或多个类
toggleClass()-对被选元素进行添加/删除类的切换操作
css()-设置或返回样式属性
```
### <center> getComputedStyle()的定义

getComputedStyle()是一个可以获取当前元素所有最终使用的CSS属性值，返回的是一个CSS样式声明对象([object CSSStyleDeclaration])，只读不可写。

语法如下：
```bush
var style = window.getComputedStyle("元素","伪类");
```
该方法只有两个参数，第一个参数是所选的元素，第二个参数“伪类”不是必需的（如果不是伪类，设置为null）。

#### <center> getComputedStyle()与style()

在使用element.style来获取元素的CSS样式声明对象时，与getComputedStyle()的结果还是有差异的，差异如下：

- 只读与可写
```bush
之前说过getComputedStyle()是只读的，只能获取样式，不能设置；而element.style能读也能写~
```
- 获取的对象范围
```bush
之前也说过getComputedStyle()获取的是最终应用在元素上的所有CSS属性对象（即使没有CSS代码，也会把默认的都显示出来），而element.style只能获取元素的style属性中的CSS样式。
```

#### <center> getComputedStyle()与defaultView()

如果我们查看jQuery源码，会发现css()实现使用的不是window.getComputedStyle()，而是document.defaultView.getComputedStyle()。
实际上，使用defaultView基本上是没必要的，getComputedStyle()本省就存在window对象之中。但是根据一种说法，使用defaultView是不愿意在window上专门写方法，或者是让API在JAVA中也可以用。

#### <center> getComputedStyle()与currentStyle()

element.currentStyle()返回的是元素当前应用的最终CSS属性值(包括外链CSS文件，页面中嵌入的style属性等)
从作用上说，getComputedStyle与currentStyle属性走的很近，形式上则style与currentStyle走的近，不过currentStyle不支持伪类样式的获取，这是与getComputeStyle方法的差异，也是jQuery css()无法体现的一点。
举例：
```bush
alert((element.currentStyle? element.currentStyle : window.getComputedStyle(element, null)).height);
```
上面的代码在firefox下显示24px，在IE浏览器下则是CSS中的2em属性值。
这两个方法其他具体差异还有很多~之后补充。

### <center> getPropertyValue()的定义

- getPropertyValue()可以获取CSS样式的属性值，如：
```bush
window.getComputeStyle(element,null).getPropertyValue("float");
```
不使用getPropertyValue方法，直接使用键值访问，也可以得到属性值，但是，比如这里的float，如果用键值访问，则不能直接使用getComputedStyle(element,null).float，而应该是cssfloat和stylefloat，自然需要浏览器判断了，比较折腾~

- 使getPropertyValue方法不支持驼峰写法，如：
```bush
style.getPropertyValue("border-top-left-radius");
```

- getPropertyValue()方法IE9+以及现代浏览器都支持

一涉及到兼容性就... 那IE8以下怎么办呢？
这时候就不得不提到 getAttribute()

#### <center> getPropertyValue()和getAttribute()的区别

在老IE浏览器中，getAttribute()提供了与getPropertyValue()类似的功能，可以访问CSS样式对象的属性，用法和getPropertyValue()类似：
```bush
style.getAttribute("float");
```
注意！使用getAttribute()也不需要cssfloat与stylefloat的怪异写法和兼容性处理，但是属性名需要驼峰写法，例如：
```bush
style.getAttribute("backgroundColor");
```

#### <center> getPropertyValue()和getPropertyCSSValue()的区别

- getPropertyCSSValue()返回的是一个CSS最初值对象或CSS值列表对象，这取决于style属性值的类型。在某些特别的style属性下，其返回的是自定义对象。该自定义对象继承于CSSValue对象

- getPropertyCSSValue()的兼容性不好，IE9不支持，Opera也不支持。


[资料来源1](http://www.zhangxinxu.com/wordpress/2012/05/getcomputedstyle-js-getpropertyvalue-currentstyle/)
[资料来源2](http://www.w3school.com.cn/jquery/jquery_css_classes.asp)


