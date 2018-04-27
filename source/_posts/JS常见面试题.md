---
title: JS知识点（持续扩展）
date: 2018/4/18
categories: 小Tips
---
![2018/4/18 晴天](https://imgsa.baidu.com/forum/w%3D580/sign=4dfad5bbd31373f0f53f6f97940d4b8b/4beffffe9925bc31c0d94c585cdf8db1ca137000.jpg)

### <center> 图片懒加载
<center>[引用源文](https://www.cnblogs.com/flyromance/p/5042187.html)

##### - 什么叫图片懒加载？
当访问一个页面的时候，先把img元素或是其他元素的背景图路径替换成一张大小为1*1px图片的路径（这样就只需请求一次），只有当图片出现在浏览器的可视区域内，才设置图片真正的路径，让图片显示出来，这就是图片懒加载。

##### - 为什么要使用这个技术？
如果一个页面中有很多图片，一上来就发送那么多请求，页面加载就会很漫长。如果js文件都放在了文档的底部，恰巧页面的头部又依赖这个js文件，那就凉凉了；最重要的是，这么多的请求，服务器可能会吃不消。

##### - 怎么实现？
- 页面中的ig元素，如果没有src属性，浏览器就不会发出请求去下载图片，一旦通过js设置了图片路径，浏览器才会发送请求。（有点按需分配的意思）
- 把真正的路径存在元素的“data-url”（自己取个名字）属性里，要用的时候就取出来，再设置。
- 获取某个元素的尺寸大小、滚动条滚动距离以及偏移位置距离的方法：
     -  屏幕可视窗口大小：
        原生方法：window.innerHeight 标准浏览器及IE9 || document.documentElement.scrollTop标准浏览器及低版本IE标准模式|| document.body.clientHeight 低版本混杂模式
        jQuery方法：$(window).height()
     - 浏览器窗口顶部与文档顶部之间的距离，也就是滚动条滚动的高度
　　 原生方法：window.pagYoffset——IE9+及标准浏览器 || document.documentElement.scrollTop 兼容ie低版本的标准模式 || document.body.scrollTop 兼容混杂模式；
         jQuery方法：$(document).scrollTop(); 

　  - 获取元素的尺寸，左边jquery方法，右边原生方法
```bush
$(o).width() = o.style.width; 

$(o).innerWidth() = o.style.width+o.style.padding;

$(o).outerWidth() = o.offsetWidth = o.style.width+o.style.padding+o.style.border;

$(o).outerWidth(true) = o.style.width+o.style.padding+o.style.border+o.style.margin;
```

　　　　注意：要使用原生的style.xxx方法获取属性，这个元素必须已经有内嵌的样式，如
```bush
<div style="...."></div>
```
　　　　如果原先是通过外部或内部样式表定义css样式，必须使用以下方法来获取样式值：
```bush
o.currentStyle[xxx] || document.defaultView.getComputedStyle(0)[xxx]
```


- 获取元素的位置信息

　　1 返回元素相对于文档document顶部、左边的距离；
```bush
　jQuery：$(o).offset().top元素距离文档顶的距离，$(o).offset().left元素距离文档左边缘的距离
```
　原生：getoffsetTop()返回的元素相对于第一个以定位的父元素的偏移距离，注意与上面偏移距的区别；
```bush
jQuery：position()返回一个对象，$(o).position().left = style.left，$(o).position().top = style.top；
```
- 如何判断某个元素进入或者即将进入可视窗口区域？
就是根据对象的边界与可视窗口边界相对距离的大小来判断。

### <center> 事件委托和事件捕获
<center>[阻止事件委托和事件捕获](https://www.jianshu.com/p/f937db28b007)
<center>[源文来源](https://blog.csdn.net/u013035060/article/details/60770477)

##### <center> 概述
<right>事件委托还有一个名字叫事件代理，利用事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。举个例子更生动：
有三个同事预计会在周一收到快递，为签收快递，有两种方法：一是三个人在公司门口等快递；二是委托给前台MM签收。现实当中，我们大都采用委托的方案（公司也不会容忍那么多员工站在门口就为了等快递）。前台MM收到快递后，她会判断收件人是谁，然后按照收件人的要求签收，甚至代为付款。这种方案还有一个优势，那就是即使公司里来了新员工（不管多少），前台MM也会在收到寄给新员工的快递后核实并代为签收。
这里其实还有2层意思的：
第一，现在委托前台的同事是可以代为签收的，即程序中的现有的dom节点是有事件的；
第二，新员工也是可以被前台MM代为签收的，即程序中新添加的dom节点也是有事件的。
- 为什么要用事件委托：
避免多次DOM操作，减少dom与页面的交互次数，提高性能，节省空间。

##### <center> 事件委托的原理：
<center>利用事件的冒泡，（什么是事件冒泡呢：就是事件从最深的节点开始，然后逐步向上传播事件）

```bush
<ul id="ul1">
    <li>111</li>
    <li>222</li>
    <li>333</li>
    <li>444</li>
</ul>

传统的li实现点击功能：
window.onload = function(){
    var oUl = document.getElementById("ul1");
    var aLi = oUl.getElementsByTagName('li');
    for(var i=0;i<aLi.length;i++){
        aLi[i].onclick = function(){
            alert(123);
        }
    }
}
通过遍历所有的Li绑定好点击事件。

接下来是利用事件委托：
window.onload = function(){
    var oUl = document.getElementById("ul1");
   oUl.onclick = function(){
        alert(123);
    }
}
这里用父级ul做事件处理，当li被点击时，由于冒泡原理，事件就会冒泡到ul上，因为ul上有点击事件，所以事件
就会被触发。如果想让事件代理的效果跟直接给节点的事件效果一样时，我们还有一招：

Event对象提供了一个属性叫target，可以返回事件的目标节点，标准浏览器用event.target，IE浏览器用
event.srcElement，此时只是获取了当前节点的位置，并不知道什么节点名称，这里再用nodeName来获取具体
是什么标签名，返回值是一个大写的字符串。
代码如下：

window.onload = function(){
　　var oUl = document.getElementById("ul1");
　　oUl.onclick = function(ev){
　　　　var ev = ev || window.event;
　　　　var target = ev.target || ev.srcElement;
　　　　if(target.nodeName.toLowerCase() == 'li'){
　 　　　　　　 alert(123);
　　　　　　　  alert(target.innerHTML);
　　　　}
　　}
}
```
### <center> 事件捕获的执行顺序

首先强调事件绑定时的兼容性写法：
IE8及更早版本需要用attachEvent()，正常版本用addEventListener()，对于兼容写法，如下:
```bush
var x = document.getElementById("myBtn");

if (x.addEventListener) {                    //所有主流浏览器，除了 IE 8 及更早 IE版本

    x.addEventListener("click", myFunction);

} else if (x.attachEvent) {                  // IE 8 及更早 IE 版本

    x.attachEvent("onclick", myFunction);

}
其中element.addEventListener(event，function，useCapture)方法中，event不要使用on前缀，function
指定要事件触发时执行的函数；useCapture可选，是布尔值，指定事件是否在捕获或冒泡阶段执行。
```
事件冒泡和事件捕获的过程正好相反，

```bush
<div id="div1">我是div1</div>
<div id="div2">我是div2</div>
<div id="div3">我是div3</div>
<div id="div4">我是div4</div>
var div1=document.getElementById("div1");

var div2=document.getElementById("div2");

var div3=document.getElementById("div3");

var div4=document.getElementById("div4");

div1.addEventListener("click",function(){

alert("我是div1");

})

div2.addEventListener("click",function(){

alert("我是div2");

})

div3.addEventListener("click",function(){

alert("我是div3");

})

div4.addEventListener("click",function(){

alert("我是div4");

})

当所有的Li标签绑定的事件的useCapture都是true，则点击第四个div时，会依次显示1-2-3-4，而如果是冒泡的
话，显示的结果应该是4-3-2-1

如果一个dom元素中，既有冒泡，又有捕获的话，会这么执行呢？W3C规定，任何发生在W3C模型中的事件，首先
进入捕获阶段，直到达到目标元素，再进入冒泡阶段。
```

### <center> JS数组方法
<br>
##### <center> join()
<center>Array.join()是string.split()的逆向操作

##### <center> reverse()
<center>将数组元素全部倒过来

##### <center> sort()
<center>排序，可以在括号里写排序的规则

##### <center> cancat()

```bush
var arr = [1，2，3];
arr.concat(4,5);
得到一个新的数组arr = [1，2，3，4，5]
```
##### <center> slice(start,end)
<center>返回数组片段，不需要变量接收

##### <center> splice()
<center> 删除，插入，替换。
这个方法的功能很多，参数的不同决定功能的不同；

##### 删除的用法，array.splice(starti,n)，其中starti的含义是从哪个位置开始，n指的是需要删除的个数。
```bush
<script>  
    var array=[1,2,3,4,5];  
    array.splice(3,2);  
    console.log(array);  
</script>  
得到的结果是：[1,2,3]，这里被删除的元素其实可以用一个变量来接收
```
##### 插入的用法，array.splice(starti，0，值1，值2......)，其中0表示删除0个元素，值1和值2表示需要插入的值。
##### 替换的用法，array.splice(starti，n，值1，值2)，在需要替换的位置先删除，然后再插入值~
<br>
#### <center> 把数组当栈使用
<center>push()，尾部添加
<center>unshift()，头部添加
<center>pop()，尾部删除
<center>shift()，头部删除
- 遍历
for...in...（除了可以遍历数组，还可以遍历对象的键值对）
- indexof（）从左向右索引，
lastIndexof恰好相反，是从右向左索引。
- 数组去重
数组去重的方法很多，比如利用一个for循环，在里面用indexof的返回值来判断是否有重复，有的话就删除，没有的话继续；或者是两个循环遍历一次。很多方法~

### <center> 实现拖拽功能
<br>
原理：
1 鼠标按下时，开始执行
2 鼠标按下后，鼠标开始移动时，需要拖拽的元素跟着一起移动
3 鼠标松开，然后停止移动
对应的三个事件就是：
1 onmousedown
2 onmousemove
3 onmouseup
在鼠标按下时，记录当前元素的位置newSite，同时设置一个标记，用来表示此时鼠标按下的状态，方便鼠标移动事件执行。
当鼠标松开时，我们要将标记改变，说明此时鼠标松开了，当鼠标移动时就不能再移动元素了。
当鼠标按下并且移动时，我们就要记录鼠标每次移动的位置currentSite，currentSite和newSite的差值就是鼠标移动的距离，同时也是元素应该移动的距离，然后设置元素的位置就可以了。
这里需要注意的是，需要移动的元素的Position属性应该设为absolute或者relative，这样元素才可以移动。
其中目标元素的初始位置都从CSS样式中提取。
### <center> 实现Gulp的功能
<br>
<center>[参考源文](https://blog.csdn.net/qingyafan/article/details/52231383)
前端优化中的一条是，尽量缩小js，html，css和图片等静态文件的尺寸，因为每个文件都将会有一个http请求，在下载的过程中，浏览器会停止所有手头工作，专心下载，直到完成，才会做其他事情。而gulp主要就是自动化压缩静态文件的工具。
- 特点：1 相对于其他构建工具，gulp以“管道和文件流”的处理方式标榜自己。数据流是gulp的基础，读取磁盘文件流，处理文件流，最后将文件流写入磁盘文件，完成任务。管道的思想来源于Unix，举个例子，水在管道中，我把管道接到哪里，水就可以流到哪里，也就是说，我可以利用多根管道，让水流过多个地方，这样多个地方都可以用到水。
2 gulp的另一个特点是它的模块插件，其主张“每个模块插件只应该做一件事”，gulp只提供比较简单的基础功能，或者说“给它的插件提供一个平台”，大部分工作都是其插件在其提供的“平台”之上完成的。
#####- 内容
gulp一共有四个方法或者说API，
- gulp.src(glob[s][,options])，读取磁盘文件，并返回文件流，glob是字符串表示的文件通配符，类似正则表达式，可以是字符串或者是glob数组。

- gulp.dest(path[,options])，将管道中的文件流写入文件，因为我们可以使用多根管道，因此我们当然可以多次调用dest方法，path是写入文件的路径。

- gulp.task(name[,deps],[,fn])，定义具体的任务，name是字符串类型的任务名，如果name是“default”，那么在命令行直接输入gulp，默认会执行这个任务；deps表示依赖的任务，是任务名字符串数组，也就是说deps任务执行完才会执行name任务；fn是执行完name指定的任务后的回调函数。

- gulp.watch(glob[s][,options],tasks)或者gulp.watch(glob[,options,cb])，相当于一个观察者，一旦在其管辖下的文件有所变动，执行一个或多个任务，glob是字符串或者glob数组；watch有两种形式，tasks表示事件触发时要执行的任务数组，cb表示事件触发时的回调函数，传入回调函数的参数是事件对象，返回观察者对象，该对象可以监听change，end，ready，error，ready，nomatch事件。

##### - 常用功能及其插件
1 gulp-jshint，检查js的错误
2 gulp-uglify，压缩js
3 gulp-minify-css，压缩css
- gulpfile配置文件
gulpfile是定义gulp任务的地方，它所遵循的commonjs规范和js文件模块并无二致，在gulpfile中定义的任务，我们可以在命令行中调用，调用语法为 gulp takename，如果不带taskname参数，默认执行名为default的任务。
- gulp自动化实例
在gulpfile中引入需要的模块，如：
```
const gulp = require('gulp');
const jshint = require('gulp-jshint');
const uglify = require('gulp-uglify');
const minifyCss = require('gulp-minify-css');
```
- 检查语法或者其他错误（jshint）
检查 JavaScript 文件是否存在错误或者可能会引起错误的地方，例如一般比较应该使用===/!==代替==/!=，字符串换行拼接应在行末添加连接符+等等。gulp-jshint 并不能独自完成这项艰巨的任务，它依赖于真正的大将 jshint，gulp-jshint 只不过加了一层包裹，使之可以在 gulp 中使用，所以不仅应该安装 gulp-jshint 依赖，还应该添加 jshint 依赖：npm install --save-dev jshint gulp-jshint。
```
// check javascript error or warnning
gulp.task('jshint', function() {
  gulp.src('./scripts/apps/*.js')
      .pipe('jshint')
      .pipe(jshint.reporter('default'));
});
执行 gulp jshint，看疗效如下：
```
![image](http://upload-images.jianshu.io/upload_images/8542482-ccfbae49588870d7?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 压缩JS文件
```
// minify javascript file from apps folder to scripts/dist folder
gulp.task('minifyjs', function() {
  gulp.src('./scripts/apps/*.js')
      .pipe(uglify())
      .pipe(gulp.dest('./scripts/dist'));
});
```
经过上面对 gulp API 的介绍，你应该能理解这段代码，定义一个名为‘minifyjs’的任务，可以在命令行通过 gulp minifyjs进行调用。任务首先读取 ./scripts/apps 文件夹中的所有以 .js 后缀结尾的文件，然后使用管道将流导入 uglify() 加工，最后再通过管道将加工好的流写入到 ./scripts/dist 文件夹中，名称不变，如果 ./scripts/dist 文件夹不存在，那么 gulp 会主动创建，如果 ./scripts/apps 文件夹不存在，那么 gulp 什么也不做，也不报错【笑】！执行效果如下:
![image](http://upload-images.jianshu.io/upload_images/8542482-62fa73ee3afc69cf?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 压缩合并CSS文件
css 压缩任务和 JavaScript 压缩任务几乎相同，就不多加解释了。
```
// minify css file from css folder to css/dist folder
gulp.task('minifycss', function() {
  gulp.src('./styles/*.css')
      .pipe(minifyCss())
      .pipe(gulp.dest('./styles/dist'));
});
```
- 监控代码改变
```
gulp.task('watchchange', function() {
  gulp.watch('./scripts/apps/*.js', ['minifyjs']);
  gulp.watch('./styles/*.css', ['minifycss']);
});
```
### <center> promise的实现原理
[资料源文](https://mengera88.github.io/2017/05/15/promise%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/)
- 什么是promise？
promise是把类似异步处理对象和处理规则进行规范化，并按照采用统一的接口来编写，而采取规定方法之外的写法都会报错。例如：
```
var promise = http.get('/v1/get');
promise.then(function(result) {
    //成功时的处理
}).catch(function(error) {
    //错误时的处理
})
```
我们必须按照接口规定的方法编写处理代码。也就是说，除promise对象规定的方法(这里的 then 或 catch)以外的方法都是不可以使用的， 而不会像回调函数方式那样可以自己自由的定义回调函数的参数，而必须严格遵守固定、统一的编程方式来编写代码。这样，基于Promise的统一接口的做法， 就可以形成基于接口的各种各样的异步处理模式。
但这并不是使用promise的足够理由，promise为异步操作提供了统一的接口，能让代码不至于陷入回调嵌套的死路中，它的强大之处在于它的链式调用（文章后面会有提及）。
- 基本用法
promise的语法：
```
new Promise(function(resolve, reject) {
    //待处理的异步逻辑
    //处理结束后，调用resolve或reject方法
})
```
新建一个promise很简单，只需要new一个promise对象即可。所以promise本质上就是一个函数，它接受一个函数作为参数，并且会返回promise对象，这就给链式调用提供了基础。
其实Promise函数的使命，就是构建出它的实例，并且负责帮我们管理这些实例。而这些实例有以下三种状态：

1 pending: 初始状态，位履行或拒绝
2 fulfilled: 意味着操作成功完成
3 rejected: 意味着操作失败

pending 状态的 Promise对象可能以 fulfilled状态返回了一个值，也可能被某种理由（异常信息）拒绝（reject）了。当其中任一种情况出现时，Promise 对象的 then 方法绑定的处理方法（handlers）就会被调用，then方法分别指定了resolve方法和reject方法的回调函数。
![image](http://upload-images.jianshu.io/upload_images/8542482-8cc5795fed1e4d8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
简单的示例：
```
var promise = new Promise(function(resolve, reject) {
  if (/* 异步操作成功 */){
    resolve(value);
  } else {
    reject(error);
  }
});
promise.then(function(value) {
  // 如果调用了resolve方法，执行此函数
}, function(value) {
  // 如果调用了reject方法，执行此函数
});
```
- promise的链式操作
Promise.prototype.then方法返回的是一个新的Promise对象，因此可以采用链式写法。
```
getJSON("/visa.json").then(function(json) {
  return json.name;
}).then(function(name) {
  // proceed
});
上面的代码使用then方法，依次指定了两个回调函数。第一个回调函数完成以后，会将返回结果作为参数，传入第二个回调函数。
```
如果前一个回调函数返回的是Promise对象，这时后一个回调函数就会等待该Promise对象有了运行结果，才会进一步调用。例如：
```
getJSON("/visa/get.json").then(function(post) {
  return getJSON(post.jobURL);
}).then(function(jobs) {
  // 对jobs进行处理
});
```
这种设计使得嵌套的异步操作，可以被很容易得改写，从回调函数的“横向发展”改为“向下发展”。
### <center> webpack
[资料源文](https://www.webpackjs.com/concepts/)
webpack的功能就如它的官网所示——打包所有的资源、脚本、样式、图片。
![TIM图片20180331222517.png](https://upload-images.jianshu.io/upload_images/8542482-42aad8702ce073e5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
注意下面的两句英文：依赖模型   和  静态资产
- 首先理解四个核心概念
1 入口
2 输出
3 loader
4 插件

#####1 入口
入口起点(entry point)指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。我们看一个 entry 配置的最简单例子：
webpack.config.js
```
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```
##### 2 出口
output 属性告诉 webpack 在哪里输出它所创建的 bundles，以及如何命名这些文件。你可以通过在配置中指定一个 output 字段，来配置这些处理过程：
webpack.config.js
```
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```
- Gulp和Webpack的区别：
gulp是基于流的构建工具：all in one的打包模式，输出一个js文件和一个css文件，优点是减少http请求，万金油方案。
webpack是模块化管理工具，使用webpack可以对模块进行压缩、预处理、打包、按需加载等。
### <center> 浅克隆和深克隆和ES6的Symbol
[资料来源](https://juejin.im/post/5abd8ee1518825558a06b827)
- 浅度克隆
js有六大数据类型：number，string，boolean，undefined，null，object
其中number，string，boolean，undefined，null归为原始值一类，而object属于引用值，具体包括狭义的object，array，function。
浅克隆在克隆对象和数组时会克隆它们的地址，之后改变目标时，克隆得到的结果也会随着操作而改变，因为他们指向的都是同一个空间，通过一个对象在这个空间里面加了东西，另一个对象也必然会受到影响。
而对于函数而言，通过上面这种普通的赋值拷贝，就可以实现，且互不影响，因为函数的克隆会在内存中单独开辟一块空间。

-深度克隆
先整理一下思路：
1 遍历待拷贝的对象
2 判断每个元素是不是原始值，若是，则通过浅度克隆的手段进行拷贝
3 若是引用值，则需要继续判断是对象还是数组
4 再分别建立空数组或空对象用来盛放里面即将拷贝而来的属性值
5 数组和对象里面的若是原始值，则浅拷贝即可实现，若还有引用值，则还需要重复进行上述一系列的判断。

上述思路怎么用代码实现呢？
1 使用for in进行遍历。但需要注意的是，for in方法会把对象原型里的属性也一起遍历了，所以要与hasOwnProperty()方法进行联用，hasOwnProperty()方法可以判断某属性是不是该对象自己的属性，从而过滤掉原型中的属性。
2 用typeof()返回值来进行判断，数组和对象的typeof返回值是'object'
3 用toString（）方法来判断是对象还是数组
4 创建新的{}或者[]
5 重复判断，可以采用递归
```
代码如下：
function deepClone(origin, target) {
	var target = target || {},
		toStr = Object.prototype.toString,
		arrStr = '[object Array]';
	for (var prop in origin) {
		if (origin.hasOwnProperty(prop)) {
			if (typeof (origin[prop]) == 'object' && origin[prop] !== null) {
				if (toStr.call(origin[prop]) == arrStr) {
					target[prop] = [];
				} else {
					target[prop] = {};
				}
				deepClone(origin[prop], target[prop]);
			} else {
				target[prop] = origin[prop];
			}
		}
	}
	return target;
}

作者：Aden_Z
链接：https://juejin.im/post/5abd8ee1518825558a06b827
来源：掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```
注意到origin[prop] !== null这句了么？为啥要加上它，因为typeof(null)也是'object'。

##### - 针对ES6新增的Symbol类型来探讨一下浅拷贝和深拷贝
[资料源文](https://juejin.im/post/5a6f39df6fb9a01cab288733)
- Symbol介绍
Symbol是ES6中引入的原始数据类型。Symbol值通过Symbol函数生成，是独一无二的。同时，ES6中规定了对象的属性名有两种类型，一种是字符串，另一种就是 Symbol 类型。凡是属性名属于 Symbol 类型，就不会与其他属性名产生冲突。
但是，随之而来的问题是，我们的for...in循环不能遍历出该属性。

Symbol 作为属性名，该属性不会出现在for...in、for...of循环中，也不会被Object.keys()、Object.getOwnPropertyNames()、JSON.stringify()返回。但是，它也不是私有属性，有一个Object.getOwnPropertySymbols方法，可以获取指定对象的所有 Symbol 属性名。

有Symbol类型，自然有遍历Symbol类型的方法。Object.getOwnPropertySymbols + for...in的组合起来好像是能满足我们要求的了。
可是等等，还有一个方法——>Reflect.ownKeys，该方法返回一个数组，包含对象自身的所有属性，不管是属性名是Symbol或字符串，也不管是否可以枚举。

总结：
######for...in循环遍历对象自身的和继承的可枚举属性（不含 Symbol 属性）。
######Object.keys()返回一个数组，包括对象自身的（不含继承的）所有可枚举属性（不含 Symbol 属性）的键名。
######Object.getOwnPropertyNames()返回一个数组，包含对象自身的所有属性（不含 Symbol 属性，但是包括不可枚举属性）的键名。
######Object.getOwnPropertySymbols()返回一个数组，包含对象自身的所有 Symbol 属性的键名。
######Reflect.ownKeys()返回一个数组，包含对象自身的所有键名，不管键名是 Symbol 或字符串，也不管是否可枚举。
### <center> 手写parseInt的实现：
[资料原文](http://www.jb51.net/article/124438.htm)
```
function _parseInt(str, radix) {
 let str_type = typeof str;
 let res = 0;
 if (str_type !== 'string' && str_type !== 'number') {
  // 如果类型不是 string 或 number 类型返回NaN
  return NaN
 }

 // 字符串处理
 str = String(str).trim().split('.')[0]
 let length = str.length;
 if (!length) {
  // 如果为空则返回 NaN
  return NaN
 }

 if (!radix) {
  // 如果 radix 为0 null undefined
  // 则转化为 10
  radix = 10;
 }
 if (typeof radix !== 'number' || radix < 2 || radix > 36) {
  return NaN
 }

 for (let i = 0; i < length; i++) {
  let arr = str.split('').reverse().join('');
  res += Math.floor(arr[i]) * Math.pow(radix, i)
 }

 return res;
}
```
### <center> 什么的跨域，为什么会有跨域？
[原文来源](https://www.jianshu.com/p/01654a4da9e5)
- 什么是跨域？
简单地理解就是因为JavaScript同源策略的限制，如 a.com 下的js 无法和b.com 的js通信。这是浏览器为了安全而设定的策略。
只有域名相同才不会有跨域的情况。
以下情况都会出现跨域：
特别注意两点：
第一：如果是协议和端口造成的跨域问题“前台”是无能为力的，只能通过后台代理。
第二：在跨域问题上，域仅仅是通过“URL的首部”来识别而不会去尝试判断相同的ip地址对应着两个域或两个域是否在同一个ip上，“URL的首部”指https://（协议）
- 常见的情况和解决方案
1 .jsonp跨域访问接口
```
// 在a.com下
$.ajax({
    url: 'b.com/getjson'?cb=?$a=1&b=2, // cb=? jquery的jsonp 会把此处的？
    type: 'jsonp',                     //替换为jsonpCallback
    jsonpCallback:'jsonpCallback',  // 发出的请求为b.com/getjson'?cb=jsonpCallback$a=1&b=2
    success:function(data){
        console.log(data)
    }
})
```
jsonp的原理：
虽然js是受到同源策略，但是引用js文件，img，css文件是不会受到限制的，所以原理就是动态创建script，jquery在发送请求前生产了一个jsonpCallback函数，然后向b.com请求了一个js文件。
服务器根据cb的值返回一个jsonpCallback(jsonDate)，即这个js文件被浏览器解析后执行了之前定义的函数。

缺点：
1 需要后端配合提供jsonp端口
2 因为是动态创建script标签，所以只能get，不能Post。安全性要大大低于post，不适合发送机密信息。
