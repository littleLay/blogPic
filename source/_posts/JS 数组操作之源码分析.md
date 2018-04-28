---
title: JS 数组操作之源码分析
date: 2018/4/17
categories: JS
---
![2018/4/17  晴天](https://upload-images.jianshu.io/upload_images/8542482-2ef43dd9c47ca960.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


<center>想单独拿一篇博客来记录是因为，在我想要了解一些方法操作的效率或性能怎么样时，网上资料很少，所以专门去查看了一下源码，特写此文。希望能帮助到和我一样有疑惑的前端小盆友。


<center>下面进入正题：
<!-- more -->
#### <center> JS Array的特点
- 既可以当作一个普通的数组来使用，即通过下标找到数组的元素，比如：
```
var array = [19,50,99];
console.log(array[0]);
```
- 也可以当作一个栈来使用，并且继承栈的特点，先进后出，push和pop。

- 还可以当作一个哈希表来使用
例如：
```
var map = [];
map["id"] = 1234;
map["name"] = yin;
console.log(map["name"]);
```
#### <center> JS Array的实现
源码里说：JSArray有两种模式，一种是快速的，一种是慢速的，快速的用的是索引直接定位，慢速的使用哈希查找。快速和慢速的讲解见传送门。

#### <center> Push和扩容
- 初始化：举个例子，数组初始化大小为4
```
//number of element slots to pre-allocate for an empty array
static const int kPreallocatedArrayElements = 4;
```

- push操作：执行push的时候会在数组的末尾添加新的元素，而一旦空间不足时，将进行扩容。
***push过程***
源码里面push是用汇编实现的，在c++里面嵌入的汇编，这个应该是考虑到push是一个最常用的操作，所以用汇编实现***提高执行速度***，在汇编的上面封装了一层用C++封装的函数，在编译组装的时候，将把这些C++代码转成汇编代码。

扩容后，计算新容量的函数：
```
Node *code StubAssemBler::CalculateNewElementsCapacity(Node *old_capacity,ParameterMode mode){
Node *half_old_capacity = WordOrSmiShr(old_capacity,old_capacity,mode);
Node *new_capacity = IntPtrOrSmiAdd(half_old_capacity,old_capacity,mode);
Node *padding = IntPtrOrSmiConstant(16,mode);
return IntPtrOrSmiAdd(new _capacity,padding,mode);
}
```

如上代码新容量等于：
```
new _capacity = old_capacity/2 + old_capacity + 16;
```

即旧容量的1.5倍加上16。初始化是4个，当push第五个时，容量会变成 4/2+4+16=22

***push过程***
- 申请内存：申请一块刚刚计算出来的大小的内存，把旧的数据拷过去
```
Node* CodeStubAssembler::GrowElementsCapacity(
Node *object,Node *element,Node *capacity,Node *new_capacity){
Node *new_elements = AllocateFixedArray(new_capacity,mode);
CopyFixedArrayElements(elements,new_elements,capacity,new_capacity);
return new_elements;
}
```
- 复制内容
由于复制是用的memcopy，把整一段内存空间拷贝过去，所以这个操作还是比较快的。
再把新元素放到当前length的位置，再把length++

#### <center> Pop和减容

pop的逻辑使用c++写的，在执行pop的时候，第一步，获取到当前的length，用这个length-1得到要删除的元素，然后调用setLength调整容量，最后返回删除的元素。

```
int new_length = length - 1;
int remove_index = remove_position == AT_START?0:new_length;
Handle<Object> result = Subclass::GetImpl(isolate,*backing_store,remove_index);
Subclass::SetLengthImpl(isolate,receiver,new_length,backing_store);
return result;
```

***减容过程***

```
if(2*length <= capacity){
isolate->heap()->RightTrimFixedArray(*backing_store,capacity-length);
}else{
BackingStore::cast(*backing_store)->FillWithHoles(length,old_length);
}
```

如果容量大于等于length的2倍，则进行容量调整，否则用holes对象填充，第三行的rightTrim函数，会算出需要释放的空间大小，并做标记，并等待GC回收。

```
int bytes_to_trim = elements_to_trim*element_size;
Address old_end = object->address() + object ->Size();
Address new_end = old_end-bytes_to_trim;
CreateFillterObjectAt(new_end,bytes_to_trim,ClearRecorderedSlots::kYes);
```
#### <center> shift和splice数组中间的操作
```
push和pop都是在数组末尾操作，相对比较简单，而shift、unshift和splice是在数组的开始或者中间进行操纵。
```
######(1)shift 出队
即删除并返回数组的第一个元素，shift和pop调用的都是同样的删除函数，只不过shift传的删除的Position是AT_START，源码里面会判断如果是AT_START的话，会把元素进行移动。
```
if(remove_position==AT_START){
Subclass::MoveElements(isolate,receiver,backing_store,0,1,new_length,0,0);
```
#####(2)unshift在数组的开始位置插入元素
- 首先判断容量是否足够存放，如果不够，将容量扩展为老容量的1.5倍加16
- 再把老元素移到新的内存空间偏移为unshift元素个数的位置，也就是说要腾出起始的空间放unshift传进来的元素
- 如果空间够了，则直接执行memmove移动内存空间
- 最后再把unshift传进来的参数copy到开始的位置。
- 更新array的length。

#####(3)splice
splice的操作已经几乎不用去看源码了，通过shift和unshift的操作，就可以想象到它的执行过程是怎样的，只是shift和unshift的操作的Index是0，而splice可以制定index。

#### <center> Join和Sort
```
说join和sort小清新，是因为它们是用JS实现的，然后再用wasm打包成native code
```
- Join
不过join的实现逻辑并不简单，因为array的元素本身具有多样化，可能为慢元素或者快元素，还可能带有循环引用。对于慢元素，需要先排下序：
```
var keys=GetSortedArrayKeys(array,%GetArrayKeys(array,length));
```
预处理完之后，最后创建一个字符串数组，用连接符连起来。
```
var elements = new InternalArray(length);
for(var i = 0; i < length; i++){
elements[i] = ConverToString(use_local,array[i]);
}
if(separator==' '){
return %SrtingBuilderConcat(elements,length,' ');
}else{
return %SrtingBuilderJoin(elements,length,separator);
}
```
- Sort
sort函数是用的快速排序:

```
function ArraySort(comparefn){
CHECK_OBJECT_COERCIBLE(this,"Array.prototype.sort");
%Log("js/array.js execute ArraySort");
var array=TO_BOJECT(this);
var length = TO_LENGTH(array.length);
return InnerArraySort(array,length,comparefn);
}
```

不过当数组元素的个数不超过10个时，排序用的是插入排序。
```
function InnerArraySort(array,length,comparefn){
function QuickSort(a,from,to){
var third_index = 0;
while(true){
if(to-from<=10){
InsertionSort(a,from,to);
return;
}
//other code...
}
}
}
```

快速排序算法有一个关键点就是选择枢纽元素，最简单的是每次都是选取第一个元素，或者中间的元素，sort的源码里是这样选择的：
```
if(to-from>1000){
third_index = GetThirdIndex(a,from,to);
}else{
third_index = from+((to-from)>>1);
}
```

***如果元素个数在1000以内，则使用它们的中间元素，否则要算一下，***这个算法比较有趣；
```
function GetThirdIndex(a,from,to){
var t_array = new InternalArray;
var increment = 200+((to-from)&15);
var j = 0;
from+=1;
to-=1;
for(var i=from;i<to;i+=increment){
t_array[j]=[i,a[i]];
j++;
}
t_array.sort(function(a,b){
return comparefn(a[1],b[1]);
});
var third_index = t_array[t_array.length>>1][0];
return third_index;
}
```

先取一个递增区间200~215之间，再循环取出原元素里面落到这个间距的元素，放到一个新的数组里面（这个数组时C++中的数组），然后排下序，取中间的元素，因为枢纽元素刚好是所有元素的中位数时，排序效果最好，而这里取出少数元素的中位数，类似于***抽样模拟***，缺点是它得再借助另外的排序算法。

#### <center> Array和线性链接（List）的速度

线性链接是一种非连续存储的数据结构，每个元素都有一个指针指向它的下一个元素，所以它删除元素的时候不需要移动其它元素，也不需要考虑扩容的事情，但它的查找比较慢。

我们实现一个简单的List和Array进行比较。
List的每个节点用一个Node表示：
```
class Node{
constructor(value,next){
this.value = value;
this.next=next;
}
}
```

每个List都有一个头指针指向第一个元素，和一个Length记录它的长度。
```
class List{
constructor(){
this.head= null;
this.tail=null;
this.length=0;
}
}
```

然后实现它的push和unshift函数：
```
class List{
unshift(value){
return this.insert(0,value);
}
push(value){
if(this.head===null){
this.head=new Node(value,this,tail);
this.length++;
}else{
this.insert(this.length,value);
}
return this.length;
}
}
```

push和unshift都会调用一个通用的Insert函数：
```
insert(index,value){
var insertPos = this.head;
//找到需要插入的位置的节点
for(var i = 0; i < index-1; i++){
insertPos = insertPos.next;
}
var node = null;
if(index===0){
node = new Node(value,this.head);
}else{
node = new Node(value,insertPos.next);
insertPos.next = node;
}
this,length++;
return value;
}
```

有了这个List之后，就可以初始化一个List和array：
```
var list = new List();
var arr = [];
for(var i = 0;i<100;i++){
list.push(i);
arr.push(i);
}
```

用下面代码可以比较List和Array在数组起始位置插入元素的操作时间:
```
var count = 10000;
console.time("list unshift");
for(var i = 0; i<count; i++){
list.unshift(i);
}
console.timeEnd("list unshift");
console.time("array unshift");
for(var i = 0;i<count;i++){
arr.unshift(i);
}
console.timeEnd("array unshift");
```

在比较从正中间位置插入元素的时间：
```
console.time("list insert middle with index");
for(var i = 0; i < count; i++){
insert.unshift(i);
}
console.timeEnd("list unshift");

console.time("array unshift");
for(var i = 0; i < count; i++){
arr.unshift(i);
}
console.timeEnd("array unshift");
```

运行之后可以得到如下表格：
![运行时间](http://upload-images.jianshu.io/upload_images/8542482-e6501a68862833bb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以由图得到结论：
在队首插入元素，使用线性链接List的时间将会数量级的优先于Array；如果是在中间位置插入的话，由于List的查找花费了很多时间，导致总时间明显高于Array，但是如果在插入的时候，记住上一次的位置i，那么List又会明显快于Array。

***综上：***
Array的实现用了三种语言：汇编，C++和JS，最常用的如push用了汇编实现，比较常用的如Pop/splice等用了C++，较为少用的如join/sort用了JS。

Array为快元素即普通的数组时，增删元素操作需要不断的扩容、减容和调整元素的位置，特别是当不断地在起始位置插入元素时，和链表相比，这种时间效率还是比较低下的。如果使用的场景是要根据Index删除元素，使用Array还是有优势，但是若能很快定位到删除元素的位置，链表毫无疑问还是更合适的。

末尾挂一下[资料原文](https://blog.csdn.net/zdy0_2004/article/details/70198964)
我们要做文明的知识搬运工~









