---
title: HTML5有哪些新特性？
date: 2018/4/8
categories: 小Tips
---
### <center> HTML5新增了哪些内容或API？使用过哪些？

  -   document.querySelector()和document.querySelectorAll()方法
    前者返回匹配到的元素或者null，后者返回元素数组或者空数组。

- classList属性，操作多个类很方便。

- FullScreen API是一个新的JS API，方便用户的阅读或观看视频，很多网站实现了全屏功能。通过编程的方式来向用户请求全屏显示，如果交互完成，随时可以退出全屏状态。

  
```
找到支持方法：
function launchFullScreen(element) {  
  
    var element=element||document.documentElement;  
    alert(element.nodeName);  
    if (element.requestFullscreen) {  
      element.requestFullscreen();  
    } else if (element.mozRequestFullScreen) {  
      element.mozRequestFullScreen();  
    } else if (element.webkitRequestFullscreen) {  
      element.webkitRequestFullscreen();  
    } else if (element.msRequestFullscreen) {  
      element.msRequestFullscreen();  
    }  
  }  
//请注意: exitFullscreen 只能通过 document 对象调用 —— 而不是使用普通的 DOM element.  
  function exitFullscreen() {  
    if (document.exitFullscreen) {  
      document.exitFullscreen();  
    } else if (document.mozExitFullScreen) {  
      document.mozExitFullScreen();  
    } else if (document.webkitExitFullscreen) {  
      document.webkitExitFullscreen();  
    }  
  }  
element.webkitRequestFullScreen(Element.ALLOW_KEYBOARD_INPUT);//全屏状态允许键盘输入  
```
- 页面可见性 Page Visibility
该属性指页面是处于显示状态还是隐藏状态，页面可见性对于网站的统计非常有用。

- 预加载
网站优化一直是项目开发中的重点之中，常见的优化方式主要有：图片懒加载，图片sprite，css合并，数据本地存储，数据网络缓存等。

         预加载是一种浏览器机制，利用浏览器空闲时间来预先下载/加载，用户接下来很可能浏览的页面/资源。页面提供给浏览器需要预加载的集合。浏览器载入当前页面完成后，将会在后台下载需要预加载的页面并添加到缓存中，当用户访问某个预加载的链接时，如果从缓存命中，页面就得以快速呈现。

- 二维绘图API，可以用在一个新的画布（Canvas）元素上以呈现图像、游戏图形或者其他运行中的可视图形。
 
### <center> 用一个div模拟textarea的实现
<center>标签定义一个多行的文本输入空间，但不能像div一样随着内容增加自动增加，一言不合出现滚动条。如下：
<center><textarea>你开心就好啦如果给我打赏我也就开心啦大家一起开心哦

为了更好的交互，可以用div来实现textarea的功能：

##### 内容可编辑属性  contenteditable
给div添加  contenteditable=true即可
##### demo
```bush
//css
.textarea {
  height: 200px;
  width: 300px;
  padding: 4px;
  border: 1px solid #888;
  resize: vertical;
  overflow: auto;
}

.textarea:empty:before {
  content: attr(placeholder);
  color: #bbb;
}
 
//html
<div class="textarea" contenteditable="true" placeholder="This is placeholder"></div>
</div>
```
效果如下：
![image](http://upload-images.jianshu.io/upload_images/8542482-97fef9b1f7c132b5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### <center> 行级元素和块级元素的特点和转换方式：
<center>点击蓝色字体跳入我的另一篇文章，那里有详细的解释。
<center>[行级块级元素特点以及转换方式](https://www.jianshu.com/p/f937db28b007)

### <center> position的类型和特点：
![position.png](https://upload-images.jianshu.io/upload_images/8542482-1218d25767a8ede5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


