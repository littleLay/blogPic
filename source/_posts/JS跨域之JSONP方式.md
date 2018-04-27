---
title: JS跨域之JSONP方式
date: 2018/4/20
categories: JS
---
![2018/4/16 晴天](https://upload-images.jianshu.io/upload_images/8542482-43a51144a9e6251f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<center>本人小白一枚，网上关于jsonp跨域的博文和回答实在是太多了，看到好多种？？jsonp的方式，并没有一一去尝试，可能原理都相同吧，还没有做深入了解，只是先记录一下我所掌握的jsonp解决跨域的方法，希望和我一样看的头晕眼花的小伙伴能有所收获~

#### <center> 同源策略
要理解跨域，先要了解“同源策略”，所谓同源是指，域名，协议，端口相同。所谓“同源策略”，简单的说就是基于安全考虑，当前域不能访问其他域的东西。
一些常见的是否同源示例可参照下表：
![image](http://upload-images.jianshu.io/upload_images/8542482-636780d70a19dd22?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在同源策略下，在某个服务器下的页面是无法获取到该服务器以外的数据的。例如我们在自己的网站通过ajax去获取豆瓣提供的接口数据。即使是访问本地非JS文件的文件（比如.json文件）.

***不过火狐可以正常读取到***，原因是：虽然允许跨域很不安全，但是如果不跨域的话又带来很多不便，所以火狐是允许跨域的。

为什么***谷歌获取不到？***

这里要提到一点，访问本地计算机中的文件，使用的是***file协议***，file协议主要用来访问本地计算机中的文件，就如同在windows资源管理器中打开文件一样，注意它是针对本地（本机）的，简单来说，file协议是访问你本机的文件资源。
谷歌的报错可以在控制台明显看到：

``
Cross origin requests are only supported for protocol schemes:http，data，chrome，chrome-extension，https
``

实质上就是安全机制认为加载本地其他文件是跨域行为。谷歌浏览器会跨域失败，是因为浏览器安全机制不允许。

***为什么谷歌不支持跨域？***
这是因为浏览器的安全策略，即禁止ajax访问本地的文件，这是不安全也是不允许的，举例的话，就相当于你访问了一个网站，这个网站可以读取到你本地的文件.......

#### <center> 跨域问题
Chrome真的很严格，对于Chrome来说，访问本地非js文件或者非同源地址都属于跨域行为，而火狐可以访问本地文件，但是访问非同源地址时还是跨域的行为。
这类问题统称为跨域问题。

#### <center> JSON
JSON是一种数据存储格式，不懂得朋友可以自行[百度](https://www.baidu.com/)一下~

#### <center> JSONP的原理
JSONP是JSON的一种“使用模式”，可用于解决主流浏览器的跨域数据访问的问题，利用 script 元素的开放策略，网页可以得到从其他来源动态产生的JSON资料，而这种使用模式就是所谓的JSONP。用JSONP抓到的资料并不是JSON，而是任意的JAVAScript，用javascript直译器执行而不是用JSON解析器解析。

动态创建 script 标签，设置其src，回调函数也在src中设置，例如：
```bush
<script src="(http://....其他域地址)/(文件地址)?(可以写查询内容)&(callback=(回调函数的名称))">
                          ||
 <script src="index.json?callback=indexDemo">
或者
<script src="https://api.douban.com/va/book/search?q=javascript&count=1&callback=handleResponse">
```

在页面中，返回的JSON作为参数传入回调函数中，我们通过回调函数来操作数据。
这里需要注意的是，JSON文件中的数据格式如下：

```bush
这里json数据前面的名称要和第一条script标签中的callback里的函数名称一样
indexDemo({
"hello":"bye"
})
```

用过该方法的朋友会发现JSON文件会报错，先别着急否定这个方法，先读一遍下面这段话~（诡异脸）

根据上述过程总结的话，JSONP的原理如下：
- 首先在客户端注册一个callback，然后把callback的名字传给服务器。
- 此时服务器先生成JSON数据
- 然后以JS语法的方式，生成一个function，function名字就是传递上来的参数jsonp
- 最后将json数据直接以入参的方式，放置到function中，这样就生成了一段Js语法的文档，返回给客户端。
- 客户端浏览器，解析script标签，并执行返回的Js文档，此时数据作为参数，传入到了客户端预先定义好的callback函数里。

看到这里，会不会有一种焕然大悟的感觉~原来.json文件中放置的是一个函数，只有是函数，才可以将json数据以入参的方式，放置到function中。

#### <center> JSONP的优缺点

#### 优点
- 比XML轻了很多，没有那么多冗余的东西。
- JSON具有很好的可读性，但是通常返回的都是压缩过后的，不像XML这样的浏览器可以直接显示，浏览器对于JSON的格式化显示就需要借助一些插件了。
- 在JS中处理JSON很简单
- 其他语言，例如PHP对于JSON的支持也不错

#### 缺点
- JSON在服务端语言的支持不像XML那么广泛，不过JSON.org上提供很多语言的库（我并不懂这句话，以后再做研究）
- 如果你使用eval（）来解析的话，会容易出现安全问题（依旧不懂，以后遇到再来补充~）

#### 补充
JSONP是很强大的，但不是所有跨域通信需求的万灵药，还是建议可以学会使用node或其他后端语言来配合完成跨域的方法~（虽然我也还没学会，学会就会更新在博客里）

第一，也是最重要的一点，没有关于JSONP调用的错误处理，如果动态脚本插入有效，就执行调用；如果无效，就静默失败。失败是没有任何提示的，服务器既不会捕捉到404错误，也不会取消或者重新开始请求。

第二，被不信任的服务使用时会很危险，因为JSONP服务返回打包在函数调用中的JSON相应，而函数调用是由浏览器执行的，这使宿主web应用程序更容易受到各类攻击，如果打算使用JSONP服务，了解它能造成的威胁非常重要。

***参考网址***
[轻松搞定JSONP跨域请求](https://blog.csdn.net/u014607184/article/details/52027879)
[AJAX跨域请求—JSONP获取JSON数据](http://justcoding.iteye.com/blog/1366102)