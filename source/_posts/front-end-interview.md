---
title: 前端面试题
date: 2018-11-19 23:43:52
tags:
  - interview
categories:
  - front-end
---

## JavaScript 面试题

### 异步解决方案
#### Callback 回调函数
当多个异步事务多级依赖时，回调函数会形成多级的嵌套，代码变成金字塔型结构。虽然能解决异步问题，但这使得代码得看难懂，更使得调试、重构的过程充满风险。

#### Promise
Promise比传统解决方案更加合理和强大，是更加好的异步解决方案，promise对象有以下四个特点：
1. promise 可能有三种状态：等待（pending）、已完成（resolved）、已拒绝（rejected）。
2. promise 的状态只可能从“等待”转到“完成”态或者“拒绝”态，不能逆向转换，同时“完成”态和“拒绝”态不能相互转换。
3. promise对象必须实现then方法，而且then必须返回一个promise ，同一个 promise 的then可以调用多次（链式），并且回调的执行顺序跟它们被定义时的顺序一致。
4. then方法接受两个参数，第一个参数是成功时的回调，在 promise 由“等待”态转换到“完成”态时调用，另一个是失败时的回调，在 promise 由“等待”态转换到“拒绝”态时调用

代码看起来更加符合逻辑、可读性更强。Promise并没有改变JS异步执行的本质，从写法上甚至还能看到一点点callback的影子。

#### Generator
Generator函数很特殊，理解起来比promise和callback更加难，从语法上将，generator具备以下特质：
1. 定义Generator时，需要使用function*。
2. 使用时生成一个Generator对象。
3. 执行.next()激活暂停状态，开始执行内部代码，直到遇到yield，返回此时执行的结果，并记住此时执行的上下文，暂停。

#### Async/Await
通过进一步的演进，async和await函数应运而生。它们的个性特征如下：
1. async 表示这是一个async函数，而await只能在这个函数里面使用。
2. await 表示在这里等待await后面的操作执行完毕，再执行下一句代码。
3. await 后面紧跟着的最好是一个耗时的操作或者是一个异步操作。
4. await后面必须是一个Promise对象，如果不是会被转化为一个已完成状态的Promise



### 数组去重
``` js
// 方法一
Array.from(new Set([1, '1', 2, 2, 3])) // [1, "1", 2, 3]

// 方法二
[...new Set([1, '1', 2, 2, 3])] // [1, "1", 2, 3]

// 方法三 
function unique (arr) {
  const seen = new Map()
  return arr.filter((a) => !seen.has(a) && seen.set(a, 1))
}
unique([1, '1', 2, 2, 3]) // [1, "1", 2, 3]

// 方法四
function unique(arr) {
    return arr.filter(function(item, index){ //item 表示数组中的每个元素，index 是每个元素的出现位置。
       return arr.indexOf(item) === index; // indexOf 返回第一个索引值
   });
}
unique([1, '1', 2, 2, 3]) // [1, "1", 2, 3]

// 方法5️
function unique(arr) {
    var newArr = [];                  
    arr.sort();
    for(var i = 0; i < arr.length; i++){
        if( arr[i] !== arr[i+1]){
            newArr.push(arr[i]);
        }
    }
    return newArr;
}

unique([1, '1', 2, 2, 3]) // [1, "1", 2, 3]
```

### ajax
#### 概念
异步javascript和XML，是指一种创建交互式网页应用的网页开发技术。通过后台与服务器进行少量数据交换，AJAX可以使网页实现异步更新。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。

#### 工作原理
Ajax的原理简单来说通过XmlHttpRequest对象来向服务器发异步请求，从服务器获得数据，然后用javascript来操作DOM而更新页面。相当于在用户和服务器之间加了—个中间层(AJAX引擎),使用户操作与服务器响应异步化。

#### 简述ajax执行流程
``` js
var xhr =null;//创建对象 
if(window.XMLHttpRequest){
  xhr = new XMLHttpRequest();
}else{
  xhr = new ActiveXObject("Microsoft.XMLHTTP");
}
xhr.open(method,url,asyn);//初始化请求 
xhr.setRequestHeader(“”,””);//设置http头信息 
xhr.onreadystatechange =function(){
  if (xhr.readyState == 4) {
    var head = xhr.getAllResponseHeaders();
    var response = xhr.responseText;
    // 将服务器返回的数据，转换成json
    response = JSON.parse(response);
    if (xhr.status == 200) { // 请求成功
      var Authorization = xhr.getResponseHeader("Authorization");
    }
  }
}//指定回调函数 
xhr.send();//发送请求 
```
执行流程：
1. 创建XMLHttpRequest对象,也就是创建一个异步调用对象
2. 创建一个新的HTTP请求,并指定该HTTP请求的方法、URL及验证信息
3. 设置响应HTTP请求状态变化的函数
4. 发送HTTP请求
5. 获取异步调用返回的数据
6. 使用JavaScript和DOM实现局部刷新

status是XMLHttpRequest对象的一个属性，表示响应的HTTP状态码。
readyState是XMLHttpRequest对象的一个属性，用来标识当前XMLHttpRequest对象处于什么状态。

readystate 0~4：
0：未初始化状态：此时，已经创建了一个XMLHttpRequest对象
1：准备发送状态：此时，已经调用了XMLHttpRequest对象的open方法，并且XMLHttpRequest对象已经准备好将一个请求发送到服务器端
2：已经发送状态：此时，已经通过send方法把一个请求发送到服务器端，但是还没有收到一个响应
3：正在接收状态：此时，已经接收到HTTP响应头部信息，但是消息体部分还没有完全接收到
4：完成响应状态：此时，已经完成了HTTP响应的接收

#### 优点
1. 无需刷新更新数据 
2. 异步与服务器通信 
3. 前端和后端负载平衡 
4. 基于规范被广泛支持
4. 界面与应用分离

#### 缺点
1. ajax不支持浏览器back按钮。
2. 安全问题 AJAX暴露了与服务器交互的细节。 
3. 对搜索引擎的支持比较弱。
4. 破坏了程序的异常机制。
5. 不容易调试

### 平时工作中怎么样进行数据交互?如果后台没有提供数据怎么样进行开发?mock数据与后台返回的格式不同意怎么办?
由后台编写接口文档、提供数据接口实、前台通过ajax访问实现数据交互；
在没有数据的情况下寻找后台提供静态数据或者自己定义mock数据；
返回数据不统一时编写映射文件 对数据进行映射。

### ajax 和 jsonp ？
ajax和jsonp的区别：
相同点：都是请求一个url
不同点：ajax的核心是通过xmlHttpRequest获取内容
jsonp的核心则是动态添加`<script>`标签来调用服务器 提供的js脚本。

### new 命令
具体参考：[实例对象与 new 命令](https://wangdoc.com/javascript/oop/new.html)

new命令的作用，就是执行构造函数，返回一个实例对象。

为了保证构造函数必须与new命令一起使用，一个解决办法是，构造函数内部使用严格模式，即第一行加上use strict。这样的话，一旦忘了使用new命令，直接调用构造函数就会报错。
``` js
function Fubar(foo, bar) {
  'use strict';
  this._foo = foo;
  this._bar = bar;
}

Fubar()
// TypeError: Cannot set property '_foo' of undefined
```

另一个解决办法，构造函数内部判断是否使用new命令，如果发现没有使用，则直接返回一个实例对象。
``` js
function Fubar(foo, bar) {
  if (!(this instanceof Fubar)) {
    return new Fubar(foo, bar);
  }

  this._foo = foo;
  this._bar = bar;
}

Fubar(1, 2)._foo // 1
(new Fubar(1, 2))._foo // 1
```

**new 命令的原理**
使用new命令时，它后面的函数依次执行下面的步骤。
1. 创建一个空对象，作为将要返回的对象实例。
2. 将这个空对象的隐式原型，指向构造函数的prototype属性。
3. 将这个空对象赋值给函数内部的this关键字。
4. 开始执行构造函数内部的代码，如果无返回值或者返回一个非对象值，则将新创建的空对象作为将要返回的对象；如果返回值是一个新对象的话那么直接返回该对象。

也就是说，构造函数内部，this指的是一个新生成的空对象，所有针对this的操作，都会发生在这个空对象上。构造函数之所以叫“构造函数”，就是说这个函数的目的，就是操作一个空对象（即this对象），将其“构造”为需要的样子。

new命令简化的内部流程，可以用下面的代码表示。
```js
function _new(/* 构造函数 */ constructor, /* 构造函数参数 */ params) {
  // 将 arguments 对象转为数组
  var args = [].slice.call(arguments);
  // 取出构造函数
  var constructor = args.shift();
  // 创建一个空对象，继承构造函数的 prototype 属性
  var context = Object.create(constructor.prototype);
  // 执行构造函数
  var result = constructor.apply(context, args);
  // 如果返回结果是对象，就直接返回，否则返回 context 对象
  return (typeof result === 'object' && result != null) ? result : context;
}

// 实例
var actor = _new(Person, '张三', 28);
```

### this、apply、call、bind
参考：
[this、apply、call、bind](https://juejin.im/post/59bfe84351882531b730bac2)
[前端面试之手写一个bind方法](https://zhuanlan.zhihu.com/p/45992705)

相同之处：改变函数体内 this 的指向。
不同之处：

call、apply的区别：接受参数的方式不一样。
bind：不立即执行。而apply、call 立即执行。

### JavaScript 数组打乱顺序
参考：
* [如何将一个 JavaScript 数组打乱顺序？](https://www.zhihu.com/question/68330851/answer/266506621)
``` js
// sort 算法
let arr = [0, 1, 2, 3, 4];
arr.sort(() => Math.random() > .5);

// 最佳的打乱算法是Fisher-Yates算法
const arr = [0, 1, 2, 3, 4];
for (let i = 1; i < arr.length; i++) {
  const random = Math.floor(Math.random() * (i + 1));
  [arr[i], arr[random]] = [arr[random], arr[i]];
}
```

### 如果我们使用JavaScript的"关联数组"，我们怎么计算"关联数组"的长度？
```js
var counterArray = {
  A : 3,
  B : 4
};
counterArray["C"] = 1;
```
其实答案很简单，直接计算key的数量就可以了。
``` js
Object.keys(counterArray).length // Output 3
```

### 记忆化斐波那契函数（Memoization）
题目：斐波那契数列指的是类似于以下的数列：
``` js
1, 1, 2, 3, 5, 8, 13, ....
```

### promise
更多参考：[async/await 在chrome 环境和 node 环境的 执行结果不一致，求解？](https://www.zhihu.com/question/268007969)

#### 什么是Promise？
所谓Promise，简单说就是一个容器，里面保存着某个未来才会结束的事件的结果。从语法上说，Promise 是一个对象，从它可以获取异步操作的消息。Promise 提供统一的 API，各种异步操作都可以用同样的方法进行处理，让开发者不用再关注于时序和底层的结果。Promise的状态具有不受外界影响和不可逆两个特点。

#### Promise中的异步模式有哪些？有什么区别？
因为ES6中的Promise中只有这两个模式all和race，其他的如first、any、last等都是其他Promise库提供的。

all会将传入的数组中的所有promise全部决议以后，将决议值以数组的形式传入到观察回调中，任何一个promise决议为拒绝，那么就会调用拒绝回调。

race会将传入的数组中的所有promise中第一个决议的决议值传递给观察回调，即使决议结果是拒绝。

#### 如果向Promise.all()和Promise.race()传递空数组，运行结果会有什么不同？
all会立即决议，决议结果是fullfilled，值是undefined

race会永远都不决议，程序卡死……

#### 题目：红灯三秒亮一次，绿灯一秒亮一次，黄灯2秒亮一次；如何让三个灯不断交替重复亮灯？（用Promse实现）
思路：先用promise控制三种灯的执行顺序，然后用递归实现循环亮灯
``` js
function red() {
  console.log('red');
}

function green() {
  console.log('green');
}

function yellow() {
  console.log('yellow');
}

let light = (fn, timer) => new Promise(resolve => {
  setTimeout(function() {
    fn()
    resolve()
  }, timer)
})

// times为交替次数
function start(times) {
  if (!times) {
    return
  }

  times--
  Promise.resolve()
    .then(() => light(red, 3000))
    .then(() => light(green, 1000))
    .then(() => light(yellow, 2000))
    .then(() => start(times))

}
start(3)
```
#### Promise 执行顺序
``` js
new Promise(resolve => {
  console.log(1);
  resolve(3);
}).then(num => {
  console.log(num)
});
console.log(2)
```
这道题的输出是123。
``` js
new Promise(resolve => {
  console.log(1);
  resolve(3);
  Promise.resolve().then(()=> console.log(4))
}).then(num => {
  console.log(num)
});
console.log(2)
```
这道题的输出是1243。

#### Promise的队列与setTimeout的队列
```js
setTimeout(function() {
	console.log(4)
},0); 
new Promise(function(resolve){ 
	console.log(1) 
	for( var i=0 ; i<10000 ; i++ ){ 
		i==9999 && resolve() 
	} console.log(2) 
}).then(function(){ 
	console.log(5) 
}); 
console.log(3);
```
这道题的输出是1,2,3,5,4。
**解析：**
整个script代码，放在了macrotask queue中，setTimeout也放入macrotask queue。但是，promise.then放到了另一个任务队列microtask queue中。
这两个任务队列执行顺序如下，取1个macrotask queue中的task，执行之。然后把所有microtask queue顺序执行完，再取macrotask queue中的下一个任务。
代码开始执行时，所有这些代码在macrotaskqueue中，取出来执行之。后面遇到了setTimeout，又加入到macrotask queue中，然后，遇到了promise.then，放入到了另一个队列microtask queue。等整个execution context stack（执行上下文栈）执行完后，下一步该取的是microtask queue中的任务了。因此promise.then的回调比setTimeout先执行。

一个浏览器环境（unit of related similar-origin browsing contexts.）只能有一个事件循环（Event loop），而一个事件循环可以多个任务队列（Task queue）（目的想必是方便调整优先级），每个任务都有一个任务源（Task source）。
相同任务源的任务，只能放到一个任务队列中。
不同任务源的任务，可以放到不同任务队列中。

它指出，单独的任务队列（Job queue）中的任务总是按先进先出的顺序执行，但是不保证多个任务队列中的任务优先级，具体实现可能会交叉执行。每一个任务队列是有名字的，至于有多少个任务队列，取决于实现。每一个实现至少应该包含以上两个任务队列。

#### setImmediate 和 process.nextTick
```js
setImmediate(function(){ 
	console.log(1); 
},0); 
setTimeout(function(){ 
	console.log(2); 
},0); 
new Promise(function(resolve){ 
	console.log(3); 
	resolve(); 
	console.log(4); 
}).then(function(){ 
	console.log(5); 
}); 
console.log(6); 
process.nextTick(function(){ console.log(7); }); 
console.log(8);
```
这道题的输出是：3 4 6 8 7 5 2 1

事件的注册顺序如下：
setImmediate - setTimeout - promise.then - process.nextTick

因此，我们得到了优先级关系如下：
process.nextTick > promise.then > setTimeout > setImmediate

#### process.nextTick也会放入microtask quque，为什么优先级比promise.then高呢？
“process.nextTick 永远大于 promise.then，原因其实很简单。。。在Node中，_tickCallback在每一次执行完TaskQueue中的一个任务后被调用，而这个_tickCallback中实质上干了两件事：
* nextTickQueue中所有任务执行掉(长度最大1e4，Node版本v6.9.1)
* 第一步执行完后执行_runMicrotasks函数，执行microtask中的部分(promise.then注册的回调)，所以很明显process.nextTick > promise.then”

#### 到底setTimeout有没有一个依赖实现的最小延迟？4ms？
4ms已经标准化了。

### JavaScript 事件委托详解
#### 基本概念
事件委托，通俗地来讲，就是把一个元素响应事件（click、keydown......）的函数委托到另一个元素；
一般来讲，会把一个或者一组元素的事件委托到它的父层或者更外层元素上，真正绑定事件的是外层元素，当事件响应到需要绑定的元素上时，会通过事件冒泡机制从而触发它的外层元素的绑定事件上，然后在外层元素上去执行函数。
举个例子，比如一个宿舍的同学同时快递到了，一种方法就是他们都傻傻地一个个去领取，还有一种方法就是把这件事情委托给宿舍长，让一个人出去拿好所有快递，然后再根据收件人一一分发给每个宿舍同学；
在这里，取快递就是一个事件，每个同学指的是需要响应事件的 DOM 元素，而出去统一领取快递的宿舍长就是代理的元素，所以真正绑定事件的是这个元素，按照收件人分发快递的过程就是在事件执行中，需要判断当前响应的事件应该匹配到被代理元素中的哪一个或者哪几个。

#### 事件冒泡
前面提到 DOM 中事件委托的实现是利用事件冒泡的机制，那么事件冒泡是什么呢？
在 document.addEventListener 的时候我们可以设置事件模型：事件冒泡、事件捕获，一般来说都是用事件冒泡的模型；
![event-delegate1.jpg](/images/front-end-interview/event-delegate1.jpg)
如上图所示，事件模型是指分为三个阶段：
捕获阶段：在事件冒泡的模型中，捕获阶段不会响应任何事件；
目标阶段：目标阶段就是指事件响应到触发事件的最底层元素上；
冒泡阶段：冒泡阶段就是事件的触发响应会从最底层目标一层层地向外到最外层（根节点），事件代理即是利用事件冒泡的机制把里层所需要响应的事件绑定到外层；

#### 事件委托的优点

**1.**减少内存消耗
试想一下，如果我们有一个列表，列表之中有大量的列表项，我们需要在点击列表项的时候响应一个事件；
``` html
<ul id="list">
  <li>item 1</li>
  <li>item 2</li>
  <li>item 3</li>
  ......
  <li>item n</li>
</ul>
// ...... 代表中间还有未知数个 li
```
如果给每个列表项一一都绑定一个函数，那对于内存消耗是非常大的，效率上需要消耗很多性能；因此，比较好的方法就是把这个点击事件绑定到他的父层，也就是 `ul` 上，然后在执行事件的时候再去匹配判断目标元素，所以事件委托可以减少大量的内存消耗，节约效率。

**2.**动态绑定事件
比如上述的例子中列表项就几个，我们给每个列表项都绑定了事件；在很多时候，我们需要通过 AJAX 或者用户操作动态的增加或者去除列表项元素，那么在每一次改变的时候都需要重新给新增的元素绑定事件，给即将删去的元素解绑事件；如果用了事件委托就没有这种麻烦了，因为事件是绑定在父层的，和目标元素的增减是没有关系的，执行到目标元素是在真正响应执行事件函数的过程中去匹配的，所以使用事件在动态绑定事件的情况下是可以减少很多重复工作的。

#### jQuery 中的事件委托
jQuery 中的事件委托相信很多人都用过，它主要这几种方法来实现：
* $.on: 基本用法: $('.parent').on('click', 'a', function () { console.log('click event on tag a'); })，它是 .parent 元素之下的 a 元素的事件代理到 $('.parent') 之上，只要在这个元素上有点击事件，就会自动寻找到 .parent 元素下的 a 元素，然后响应事件；
* $.delegate: 基本用法: $('.parent').delegate('a', 'click', function () { console.log('click event on tag a'); })，同上，并且还有相对应的 $.delegate 来删除代理的事件；
* $.live: 基本使用方法: $('a', $('.parent')).live('click', function () { console.log('click event on tag a'); })，同上，然而如果没有传入父层元素 $(.parent)，那事件会默认委托到 $(document) 上；(已废除)

#### 实现功能
##### 基本实现
比如我们有这样的一个 HTML 片段：
```html
<ul id="list">
  <li>item 1</li>
  <li>item 2</li>
  <li>item 3</li>
  ......
  <li>item n</li>
</ul>
// ...... 代表中间还有未知数个 li
```
我们来实现把 #list 下的 li 元素的事件代理委托到它的父层元素也就是 #list 上：
``` js
// 给父层元素绑定事件
document.getElementById('list').addEventListener('click', function (e) {
  // 兼容性处理
  var event = e || window.event;
  var target = event.target || event.srcElement;
  // 判断是否匹配目标元素
  if (target.nodeName.toLocaleLowerCase === 'li') {
    console.log('the content is: ', target.innerHTML);
  }
});
```
在上述代码中， target 元素则是在 #list 元素之下具体被点击的元素，然后通过判断 target 的一些属性（比如：nodeName，id 等等）可以更精确地匹配到某一类 #list li 元素之上；

### prototype 对象

1、`JavaScript` 继承机制的设计思想就是，原型对象的所有属性和方法，都能被实例对象共享。也就是说，如果属性和方法定义在原型上，那么所有实例对象就能共享，不仅节省了内存，还体现了实例对象之间的联系。

2、`JavaScript` 规定，每个函数都有一个 `prototype` 属性，指向一个对象(就是 JavaScript 的原型对象（prototype）)。

```js
function f() {}
typeof f.prototype // "object"
```

上面代码中，函数 `f` 默认具有 `prototype` 属性，指向一个对象。

3、对于普通函数来说，`prototype` 属性基本无用。但是，对于构造函数来说，生成实例的时候，该属性会自动成为实例对象的原型。

```js
function Animal(name) {
  this.name = name;
}
Animal.prototype.color = 'white';

var cat1 = new Animal('大毛');
var cat2 = new Animal('二毛');

cat1.color // 'white'
cat2.color // 'white'
```

上面代码中，构造函数 `Animal` 的 `prototype` 属性，就是实例对象 `cat1` 和 `cat2` 的原型对象。原型对象上添加一个color属性，结果，实例对象都共享了该属性。

总结一下，原型对象的作用，就是定义所有实例对象共享的属性和方法。这也是它被称为原型对象的原因，而实例对象可以视作从原型对象衍生出来的子对象。

#### 参考：
1. [《JavaScript 标准参考教程（alpha）》，by 阮一峰 prototype 对象](http://javascript.ruanyifeng.com/oop/prototype.html#toc0)

### 原型链

`JavaScript` 规定，所有对象都有自己的原型对象（ `prototype` ）。一方面，任何一个对象，都可以充当其他对象的原型；另一方面，由于原型对象也是对象，所以它也有自己的原型。因此，就会形成一个“原型链”（ `prototype chain` ）：对象到原型，再到原型的原型……

如果一层层地上溯，所有对象的原型最终都可以上溯到Object.prototype，即Object构造函数的prototype属性。也就是说，所有对象都继承了Object.prototype的属性。这就是所有对象都有valueOf和toString方法的原因，因为这是从Object.prototype继承的。

那么，Object.prototype对象有没有它的原型呢？回答是Object.prototype的原型是null。null没有任何属性和方法，也没有自己的原型。因此，原型链的尽头就是null。

#### 参考：
1. [《JavaScript 标准参考教程（alpha）》，by 阮一峰 prototype 对象 - 原型链](http://javascript.ruanyifeng.com/oop/prototype.html#toc3)

### js中__proto__和prototype的区别和关系？

#### 问题描述：

js中__proto__和prototype的区别和关系？

#### 答案：

![](/images/front-end-interview/prototype1.jpg)

`__proto__` （隐式原型）与 `prototype` （显式原型）

**1.是什么？**

* 显式原型 explicit prototype property：

每一个函数在创建之后都会拥有一个名为 `prototype` 的属性，这个属性指向函数的原型对象。

Note：通过 `Function.prototype.bind` 方法构造出来的函数是个例外，它没有 `prototype` 属性。

* 隐式原型 implicit prototype link：

`JavaScript` 中任意对象都有一个内置属性[[ `prototype` ]]，在 `ES5` 之前没有标准的方法访问这个内置属性，但是大多数浏览器都支持通过 `__proto__` 来访问。`ES5` 中有了对于这个内置属性标准的 `Get` 方法 `Object.getPrototypeOf()`。

Note:`Object.prototype` 这个对象是个例外，它的 `__proto__` 值为 `null`.

* 二者的关系：

隐式原型指向创建这个对象的函数( `constructor` )的 `prototype`.

**2. 作用是什么**

* 显式原型的作用：用来实现基于原型的继承与属性的共享。

* 隐式原型的作用：构成原型链，同样用于实现基于原型的继承。举个例子，当我们访问 `obj` 这个对象中的 `x` 属性时，如果在 `obj` 中找不到，那么就会沿着 `__proto__` 依次查找。

**3. __proto__的指向**

`__proto__` 的指向到底如何判断呢？根据ECMA定义 `'to the value of its constructor’s "prototype" '` ----指向创建这个对象的函数的显式原型。所以关键的点在于找到创建这个对象的构造函数，接下来就来看一下JS中对象被创建的方式，一眼看过去似乎有三种方式：（1）`对象字面量的方式` （2）`new 的方式` （3）`ES5中的Object.create()`， 但是我认为本质上只有一种方式，也就是通过 `new` 来创建。为什么这么说呢，首先字面量的方式是一种为了开发人员更方便创建对象的一个语法糖，本质就是 `var o = new Object(); o.xx = xx;o.yy=yy;` 再来看看 `Object.create()`,这是 `ES5` 中新增的方法，在这之前这被称为原型式继承.

::: warning
道格拉斯在2006年写了一篇文章，题为 Prototypal Inheritance In JavaScript。在这篇文章中，他介绍了一种实现继承的方法，这种方法并没有使用严格意义上的构造函数。他的想法是借助原型可以基于已有的对象创建新对象，同时还不比因此创建自定义类型，为了达到这个目的，他给出了如下函数:
:::

```js
function object(o){
  function F(){}
  F.prototype = o;
  return new F()
}
```

所以从实现代码 `return new F()` 中我们可以看到，这依然是通过 `new` 来创建的。不同之处在于由 ` Object.create()` 创建出来的对象没有构造函数，看到这里你是不是要问，没有构造函数我怎么知道它的 `__proto__` 指向哪里呢，其实这里说它没有构造函数是指在 `Object.create()` 函数外部我们不能访问到它的构造函数，然而在函数内部实现中是有的，它短暂地存在了那么一会儿。假设我们现在就在函数内部，可以看到对象的构造函数是 `F`, 现在

```js
//以下是用于验证的伪代码
var f = new F(); 
//于是有
f.__proto__ === F.prototype //true
//又因为
F.prototype === o;//true
//所以
f.__proto__ === o;
```

因此由 `Object.create(o)` 创建出来的对象它的隐式原型指向 `o`。好了，对象的创建方式分析完了，现在你应该能够判断一个对象的 `__proto__` 指向谁了。

好吧，还是举一些一眼看过去比较疑惑的例子来巩固一下。

* 构造函数的显示原型的隐式原型：

内建对象(built-in object)：比如 `Array()`，`Array.prototype.__proto__` 指向什么？ `Array.prototype` 也是一个对象，对象就是由 `Object()` 这个构造函数创建的，因此 `Array.prototype.__proto__ === Object.prototype //true`，或者也可以这么理解，所有的内建对象都是由 `Object()` 创建而来。

* 自定义对象:

1. 默认情况下：

```js
function Foo(){}
var foo = new Foo()
Foo.prototype.__proto__ === Object.prototype //true 理由同上
```

2. 其他情况： 

(1)、Foo继承Bar
```js
function Bar(){}
//这时我们想让Foo继承Bar
Foo.prototype = new Bar()
Foo.prototype.__proto__ === Bar.prototype //true
```

(2)、Foo不继承
```js
//我们不想让Foo继承谁，但是我们要自己重新定义Foo.prototype
Foo.prototype = {
  a:10,
  b:-10
}
//这种方式就是用了对象字面量的方式来创建一个对象，根据前文所述 
Foo.prototype.__proto__ === Object.prototype
```

注： 以上两种情况都等于完全重写了 `Foo.prototype`，所以 `Foo.prototype.constructor` 也跟着改变了，于是乎 `constructor` 这个属性和原来的构造函数 `Foo（）` 也就切断了联系。

* 构造函数的隐式原型

既然是构造函数那么它就是 `Function（）` 的实例，因此也就指向 `Function.prototype`,比如 `Object.__proto__ === Function.prototype`

**4. instanceof**

`instanceof` 操作符的内部实现机制和隐式原型、显式原型有直接的关系。`instanceof` 的左值一般是一个对象，右值一般是一个构造函数，用来判断左值是否是右值的实例。它的内部实现原理是这样的： 

```js
//设 L instanceof R 
//通过判断
 L.__proto__.__proto__ ..... === R.prototype ？
//最终返回true or false
```

也就是沿着 `L` 的 `__proto__` 一直寻找到原型链末端，直到等于 `R.prototype` 为止。知道了这个也就知道为什么以下这些奇怪的表达式为什么会得到相应的值了.

```js
Function instanceof Object // true 
Object instanceof Function // true 
Function instanceof Function //true
Object instanceof Object // true
Number instanceof Number //false
```

#### 参考：
1. [js中__proto__和prototype的区别和关系？](https://www.zhihu.com/question/34183746/answer/59043879)

### JavaScript 如何工作：对引擎、运行时、调用堆栈的概述

#### JavaScript 引擎

JavaScript 引擎说起来最流行的当然是谷歌的 V8 引擎了， V8 引擎使用在 Chrome 以及 Node 中，这个引擎主要由两部分组成:

* 内存堆：这是内存分配发生的地方
* 调用栈：这是你的代码执行时的地方

#### 运行时

有些浏览器的 API 经常被使用到(比如说：setTimeout)，但是，这些 API 却不是引擎提供的。我们把这些引擎之外的 API称为浏览器提供的 Web API，比如说 DOM、AJAX、setTimeout等等。

#### 调用栈工作机制

JavaScript 是一门单线程的语言，这意味着它只有一个调用栈，因此，它同一时间只能做一件事。

调用栈是一种数据结构，它记录了我们在程序中的位置。如果我们运行到一个函数，它就会将其放置到栈顶。当从这个函数返回的时候，就会将这个函数从栈顶弹出，这就是调用栈做的事情。简单的说：**函数被调用时，就会被加入到调用栈顶部，执行结束之后，就会从调用栈顶部移除该函数**，这种数据结构的关键在于后进先出，即大家所熟知的 LIFO。比如，当我们在函数 y 内部调用函数 x 的时候，调用栈从下往上的顺序就是 y -> x 。

我们举个代码实例：

```js
function c() {
    console.log('c');
}

function b() {
    console.log('b');
    c();
}

function a() {
    console.log('a');
    b();
}

a();
```

这段代码运行时，首先 a 会被加入到调用栈的顶部，然后，因为 a 内部调用了 b，紧接着 b 被加入到调用栈的顶部，当 b 内部调用 c 的时候也是类似的。在调用 c的时候，我们的调用栈从下往上会是这样的顺序：a -> b -> c。在 c 执行完毕之后，c 被从调用栈中移除，控制流回到 b 上，调用栈会变成：a -> b，然后 b 执行完之后，调用栈会变成：a，当 a 执行完，也会被从调用栈移除。

当程序开始执行的时候，调用栈是空的，每一个进入调用栈的都称为__调用帧__。

为了更好的说明调用栈的工作机制，我们对上面的代码稍作改动，使用 console.trace 来把当前的调用栈输出到 console 中，你可以认为console.trace 打印出来的调用栈的每一行出现的原因是它下面的那行调用而引起的。

```js
function c() {
    console.log('c');
    console.trace();
}

function b() {
    console.log('b');
    c();
}

function a() {
    console.log('a');
    b();
}

a();
```

当我们在 [Node.js 的 REPL](http://link.zhihu.com/?target=https%3A//nodejs.org/api/repl.html) 中运行这段代码，会得到如下的结果：

```text
Trace
    at c (repl:3:9)
    at b (repl:3:1)
    at a (repl:3:1)
    at repl:1:1 // <-- 从这行往下的内容可以忽略，因为这些都是 Node 内部的东西
    at realRunInThisContextScript (vm.js:22:35)
    at sigintHandlersWrap (vm.js:98:12)
    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
    at REPLServer.defaultEval (repl.js:313:29)
    at bound (domain.js:280:14)
    at REPLServer.runBound [as eval] (domain.js:293:12)
```

显而易见，当我们在 c 内部调用 console.trace 的时候，调用栈从下往上的结构是：a -> b -> c。如果把代码再稍作改动，在 b 中 c 执行完之后调用，如下：

```js
function c() {
    console.log('c');
}

function b() {
    console.log('b');
    c();
    console.trace();
}

function a() {
    console.log('a');
    b();
}

a();
```

通过输出结果可以看到，此时打印的调用栈从下往上是：a -> b，已经没有 c 了，因为 c 执行完之后就从调用栈移除了。

```text
Trace
    at b (repl:4:9)
    at a (repl:3:1)
    at repl:1:1  // <-- 从这行往下的内容可以忽略，因为这些都是 Node 内部的东西
    at realRunInThisContextScript (vm.js:22:35)
    at sigintHandlersWrap (vm.js:98:12)
    at ContextifyScript.Script.runInThisContext (vm.js:24:12)
    at REPLServer.defaultEval (repl.js:313:29)
    at bound (domain.js:280:14)
    at REPLServer.runBound [as eval] (domain.js:293:12)
    at REPLServer.onLine (repl.js:513:10)
```

再总结下调用栈的工作机制：**调用函数的时候，会被推到调用栈的顶部，而执行完毕之后，就会从调用栈移除。**

"**堆栈溢出**"，当你达到调用栈最大的大小的时候就会发生这种情况，而且这相当容易发生，特别是在你写递归的时候却没有全方位的测试它。我们来看看下面的代码：

```js
function foo() {
	foo();
}
foo();
```

当我们的引擎开始执行这段代码的时候，它从 foo 函数开始。然后这是个递归的函数，并且在没有任何的终止条件的情况下开始调用自己。因此，每执行一步，就会把这个相同的函数一次又一次地添加到调用堆栈中。

然后，在某一时刻，调用栈中的函数调用的数量超过了调用栈的实际大小，浏览器决定干掉它，抛出一个错误。

## React 面试题
### 调用 setState 之后发生了什么？
在代码中调用setState函数之后，React 会将传入的参数对象与组件当前的状态进行合并，然后触发所谓的调和过程(Reconciliation)。经过调和过程，React 会以相对高效的方式根据新的状态构建 React 元素树并且着手重新渲染整个UI界面。在 React 得到元素树之后，React 会自动计算出新元素树与旧元素树的节点差异，然后根据差异对界面进行最小化更新。在diff算法中，React 能够相对精确地知道哪些位置发生了改变以及应该如何改变，这就保证了按需更新，而并非全部更新。

### React Component vs React Element
参考：
[React 内部机制探秘 - React Component 和 Element](https://segmentfault.com/a/1190000011413614)
[React Component vs React Element](https://segmentfault.com/a/1190000008447693)

简单而言，React Element 是描述屏幕上所见内容的数据结构，是对于 UI 的对象表述。典型的 React Element 就是利用 JSX 构建的声明式代码片然后被转化为createElement的调用组合。而 React Component 则是可以接收参数输入并且返回某个 React Element 的函数或者类。

React Element 存在的意义：
* JavaScript 对象很轻量。用对象来作为 React Element，那么 React 可以轻松的创建或销毁这些元素，而不必去太担心操作成本；
* React 具有分析这些对象的能力，进一步，也具有分析虚拟 Dom 的能力。当改变出现时，（相比于真实 Dom）更新虚拟 Dom 的性能优势非常明显。

有这样的一个问题：
```js
// 方法定义
function add(x, y) {
    return x + y
}

// 方法调用
add(1, 2)

// 组件定义
class Icon extends Component {}

// 组件调用？？？？？？
<Icon />
```
最后的一句`<Icon />`用专业的词概括是什么操作，组件调用还是什么？

有答“组件声明”的，有答“组件调用的”，有“组件初始化”的，还有“使用一个组件”的。没有一个统一的称呼。造成这样局面的原因是很多时候我们都没有去详细的了解过JSX和React实际操作之间的抽象层。现在我们就深入研究一下这部分知识。

我们来看看最基础的问题，什么是React？React就是一个用来写界面的库。不管React和React的生态有多复杂，最核心的功能就是用来写界面的。那么我们来看看`Element`，很简单，但是一个React element描述的就是你想要在界面上看到的。再深入一点，一个React element就是一个代表了DOM节点的对象。注意，一个React element并不是在界面上实际绘制的东西，而是这些内容的代表。由于JavaScript对象是轻量级的，React可以任意的创建和销毁这些element对象，而且不用担心太大的消耗。另外，React可以分析这些对象，把当前的对象和之前的对象对比，找出发生的改变，然后根据实际发生的改变来更新实际的DOM。

为了创建一个DOM节点的代表对象（也就是React element），我们可以使用React的`createElement`方法。
``` js
const element = React.createElement(
    'div',
    {id: 'login-btn'},
    'Login'
)
```
`createElement`方法传入了三个参数。第一个是标签名称字符串（div、span等），第二个是给element设置的属性，第三个是内容或者是子的React element。本例中的“Login”就是element的内容。上面的`createElement`方法调用后会返回一个这样的对象：
```js
{
  type: 'div’，
  props: {
    children: 'Login',
    id: 'login-btn'
  }
}
```
当这个对象绘制为DOM（使用`ReactDOM.render`方法）的时候，我们就会有一个新的DOM节点：
```js
<div id='login-btn'>Login</div>
```
有一个很有意思的地方，我们在学React的时候首先注意到的就是component（组件）。“Components（组件）是React的构建块”。注意，我们是以element开始本文的。而且你一旦理解了element，理解component也就是水到渠成的事了。一个component就是一个方法或者一个类，可以接受一定的输入，之后返回一个React element。
```js
function Button({onLogin}) {
    return React.createElement(
        'div',
        {id: 'login-btn', onClick: onLogin},
        'Login'
    )
}
```
在上面的定义中，我们有一个`Button`组件（component）。接收一个`onLogin`输入并返回一个React element。注意，`Button`组件接收的`onLogin`方法是它的`prop`。然后把这个方法通过`createElement`方法的第二个参数传入到了实际的DOM里。

目前，我们只接触到了使用HTML元素来创建React element，比如“div”、“span”等。其实，你也可以把其他的React component（组件）作为第一个参数传入`createElement`方法。
```js
const element = React.createElement(
    User,
    {name: 'Uncle Charlie'},
    null
)
```
然而，不同于一般的HTML标签名称，React如果发现第一个参数是class或者function类型的话，它就会检查传入的参数要绘制的是一个什么element，传入必要的props。之后React会一直检查，直到没有方法或者类作为第一个参数传入createElement。我们来看看下面的例子：
```js
function Button({addFriend}) {
    return React.createElement(
        'button',
        {onClick: addFriend},
        'Add Friend'
    )
}

function User({name, addFriend}) {
    return React.createElement(
        'div',
        null,
        React.createElement(
            'p',
            null,
            name
        ),
        React.createElement(Button, {addFriend})
    )
}
```
上面的例子里有两个component（组件）。一个Button，一个User。User“代表”了一个div，div里面有两个子节点：一个包含用户名的“p”和一个Button组件。现在我们看看上面的例子的具体的调用过程。
```js
function Button({addFriend}) {
    return {
        type: 'button',
        props: {
            onClick: addFriend,
            children: 'Add Friend'
        }
    }
}

function User({name, addFreind}) {
    return {
        type: 'div',
        props: {
            children: [
                {
                    type: 'p',
                    props: {
                        children: name
                    }
                },
                {
                    type: Button,
                    props: {
                        addFriend
                    }
                }
            ]
        }
    }
}
```
在上面的代码里你会看到四种不同的属性：“button”，“div”，“p”和Button。当React看到一个element是function和类类型的话，它就会检查element会返回什么element，并传入对应的props。在这个过程结束之后，React就拥有了一个代表DOM树的对象的数。上面的例子最后的结构是这样的：
```js
{
  type: 'div',
  props: {
    children: [
      {
        type: 'p',
        props: {
          children: 'Tyler McGinnis'
        }
      },
      {
        type: 'button',
        props: {
          onClick: addFriend,
          children: 'Add Friend'
        }
      }
    ]
  }
}
```
上面叙述的整个过程叫做Reconciliation（这个不知道怎么翻译，应该叫和谐？）。在React里，每次调用setState方法或ReactDOM.render方法被调用的时候都会触发这个过程。

那么我们来看看最开始的问题：
```js
// 方法定义
function add(x, y) {
    return x + y
}

// 方法调用
add(1, 2)

// 组件定义
class Icon extends Component {}

// 组件调用？？？？？？
<Icon />
```
现在我们已经有了回答这个问题的全部知识，除了一点点以外。有个地方，你可能觉得奇怪在使用React的时候，从来没有用过createElement方法来创建element。你是用了JSX。我（作者）最开始的时候说：“主要原因是从来没有去详细的了解过JSX和React实际操作之间的抽象层”。这个抽象层就是JSX会被Babel转码为React.createElement方法的调用。
看看我们前面的例子：
```js
function Button({addFriend}) {
    return React.createElement(
        'button',
        {onClick: addFriend},
        'Add Friend'
    )
}

function User({ name, addFriend }) {
  return React.createElement(
    "div",
    null,
    React.createElement(
      "p",
      null,
      name
    ),
    React.createElement(Button, { addFriend })
  )
}
```
写成JSX的样子是这样的：
```js
function Button({addFriend}) {
    return (
        <button onClick={addFriend}>Add Friend</button>
    )
}

function User({name, addFriend}) {
    return (
        <div>
            <p>{name}</p>
            <Button addFriend={addFriend} />
        </div>
    )
}
```
所以，最后我们应该怎么回答前面的问题呢？<Icon />叫做什么？

应该叫做“创建element”，应为JSX最后会转码为createElement方法的调用：
```js
React.createElement(Icon, null)
```
前面的例子都是“创建一个React element”。
```js
React.createElement(
  'div',
  { className: 'container' },
  'Hello!'
)

<div className='container'>Hello!</div>

<Hello />
```

### 在什么情况下你会优先选择使用 Class Component 而不是 Functional Component？
在组件需要包含内部状态或者使用到生命周期函数的时候使用 Class Component ，否则使用函数式组件。

### React 中 refs 的作用是什么？
Refs 是 React 提供给我们的安全访问 DOM 元素或者某个组件实例的句柄。

具体参考：[React ref 的前世今生](https://zhuanlan.zhihu.com/p/40462264)

### React 中 keys 的作用是什么？
Keys 是 React 用于追踪哪些列表中元素被修改、被添加或者被移除的辅助标识
```js
render () {
  return (
    <ul>
      {this.state.todoItems.map(({task, uid}) => {
        return <li key={uid}>{task}</li>
      })}
    </ul>
  )
}
```
在开发过程中，我们需要保证某个元素的 key 在其同级元素中具有唯一性。在 React Diff 算法中 React 会借助元素的 Key 值来判断该元素是新近创建的还是被移动而来的元素，从而减少不必要的元素重渲染。此外，React 还需要借助 Key 值来判断元素与本地状态的关联关系，因此我们绝不可忽视转换函数中 Key 的重要性。

具体参考：[谈谈 react 中的 key](https://juejin.im/post/5a7c04746fb9a063461fe700)

### 如果你创建了类似于下面的Twitter元素，那么它相关的类定义是啥样子的？
```js
<Twitter username='tylermcginnis33'>
  {(user) => user === null
    ? <Loading />
    : <Badge info={user} />}
</Twitter>
```
``` js
import React, { Component, PropTypes } from 'react'
import fetchUser from 'twitter'
// fetchUser take in a username returns a promise
// which will resolve with that username's data.
class Twitter extends Component {
  // finish this
}
```
如果你还不熟悉回调渲染模式（Render Callback Pattern），这个代码可能看起来有点怪。这种模式中，组件会接收某个函数作为其子组件，然后在渲染函数中以props.children进行调用：
``` js
import React, { Component, PropTypes } from 'react'
import fetchUser from 'twitter'
class Twitter extends Component {
  state = {
    user: null,
  }
  static propTypes = {
    username: PropTypes.string.isRequired,
  }
  componentDidMount () {
    fetchUser(this.props.username)
      .then((user) => this.setState({user}))
  }
  render () {
    return this.props.children(this.state.user)
  }
}
```
这种模式的优势在于将父组件与子组件解耦和，父组件可以直接访问子组件的内部状态而不需要再通过Props传递，这样父组件能够更为方便地控制子组件展示的UI界面。譬如产品经理让我们将原本展示的Badge替换为Profile，我们可以轻易地修改下回调函数即可：
``` js
<Twitter username='tylermcginnis33'>
  {(user) => user === null
    ? <Loading />
    : <Profile info={user} />}
</Twitter>
```
更多可参考：[React 中的 Render Props](https://zhuanlan.zhihu.com/p/31267131)

### 受控组件 与 非受控组件
React 的核心组成之一就是能够维持内部状态的自治组件，不过当我们引入原生的HTML表单元素时（input,select,textarea 等），我们是否应该将所有的数据托管到 React 组件中还是将其仍然保留在 DOM 元素中呢？这个问题的答案就是受控组件与非受控组件的定义分割。受控组件（Controlled Component）代指那些交由 React 控制并且所有的表单数据统一存放的组件。譬如下面这段代码中username变量值并没有存放到DOM元素中，而是存放在组件状态数据中。任何时候我们需要改变username变量值时，我们应当调用setState函数进行修改。
``` js
class ControlledForm extends Component {
  state = {
    username: ''
  }
  updateUsername = (e) => {
    this.setState({
      username: e.target.value,
    })
  }
  handleSubmit = () => {}
  render () {
    return (
      <form onSubmit={this.handleSubmit}>
        <input
          type='text'
          value={this.state.username}
          onChange={this.updateUsername} />
        <button type='submit'>Submit</button>
      </form>
    )
  }
}
```
而非受控组件（Uncontrolled Component）则是由DOM存放表单数据，并非存放在 React 组件中。我们可以使用 refs 来操控DOM元素：
``` js
class UnControlledForm extends Component {
  handleSubmit = () => {
    console.log("Input Value: ", this.input.value)
  }
  render () {
    return (
      <form onSubmit={this.handleSubmit}>
        <input
          type='text'
          ref={(input) => this.input = input} />
        <button type='submit'>Submit</button>
      </form>
    )
  }
}
```
竟然非受控组件看上去更好实现，我们可以直接从 DOM 中抓取数据，而不需要添加额外的代码。不过实际开发中我们并不提倡使用非受控组件，因为实际情况下我们需要更多的考虑表单验证、选择性的开启或者关闭按钮点击、强制输入格式等功能支持，而此时我们将数据托管到 React 中有助于我们更好地以声明式的方式完成这些功能。引入 React 或者其他 MVVM 框架最初的原因就是为了将我们从繁重的直接操作 DOM 中解放出来。

### 在生命周期中的哪一步你应该发起 AJAX 请求？
我们应当将AJAX 请求放到 componentDidMount 函数中执行，主要原因有下：
* React 下一代调和算法 Fiber 会通过开始或停止渲染的方式优化应用性能，其会影响到 componentWillMount 的触发次数。对于 componentWillMount 这个生命周期函数的调用次数会变得不确定，React 可能会多次频繁调用 componentWillMount。如果我们将 AJAX 请求放到 componentWillMount 函数中，那么显而易见其会被触发多次，自然也就不是好的选择。

* 如果我们将 AJAX 请求放置在生命周期的其他函数中，我们并不能保证请求仅在组件挂载完毕后才会要求响应。如果我们的数据请求在组件挂载之前就完成，并且调用了setState函数将数据添加到组件状态中，对于未挂载的组件则会报错。而在 componentDidMount 函数中进行 AJAX 请求则能有效避免这个问题。

### shouldComponentUpdate 的作用是啥以及为何它这么重要？
shouldComponentUpdate 允许我们手动地判断是否要进行组件更新，根据组件的应用场景设置函数的合理返回值能够帮我们避免不必要的更新。

### 如何告诉 React 它应该编译生产环境版本？
通常情况下我们会使用 Webpack 的 DefinePlugin 方法来将 NODE_ENV 变量值设置为 production。编译版本中 React 会忽略 propType 验证以及其他的告警信息，同时还会降低代码库的大小，React 使用了 Uglify 插件来移除生产环境下不必要的注释等信息。

### 为什么我们需要使用 React 提供的 Children API 而不是 JavaScript 的 map？
props.children并不一定是数组类型，譬如下面这个元素：
``` js
<Parent>
  <h1>Welcome.</h1>
</Parent>
```
如果我们使用props.children.map函数来遍历时会受到异常提示，因为在这种情况下props.children是对象（object）而不是数组（array）。React 当且仅当超过一个子元素的情况下会将props.children设置为数组，就像下面这个代码片：
```js
<Parent>
  <h1>Welcome.</h1>
  <h2>props.children will now be an array</h2>
</Parent>
```
这也就是我们优先选择使用React.Children.map函数的原因，其已经将props.children不同类型的情况考虑在内了。

### 概述下 React 中的事件处理逻辑
为了解决跨浏览器兼容性问题，React 会将浏览器原生事件（Browser Native Event）封装为合成事件（SyntheticEvent）传入设置的事件处理器中。这里的合成事件提供了与原生事件相同的接口，不过它们屏蔽了底层浏览器的细节差异，保证了行为的一致性。另外有意思的是，React 并没有直接将事件附着到子元素上，而是以单一事件监听器的方式将所有的事件发送到顶层进行处理。这样 React 在更新 DOM 的时候就不需要考虑如何去处理附着在 DOM 上的事件监听器，最终达到优化性能的目的。

### createElement 与 cloneElement 的区别是什么？
createElement 函数是 JSX 编译之后使用的创建 React Element 的函数，而 cloneElement 则是用于复制某个元素并传入新的 Props。

### 传入 setState 函数的第二个参数的作用是什么？
该函数会在setState函数调用完成并且组件开始重渲染的时候被调用，我们可以用该函数来监听渲染是否完成：
```js
this.setState(
  { username: 'tylermcginnis33' },
  () => console.log('setState has finished and the component has re-rendered.')
)
```
下述代码有错吗？
``` js
this.setState((prevState, props) => {
  return {
    streak: prevState.streak + props.count
  }
})
```
这段代码没啥问题，不过只是不太常用罢了。
## vue 面试题
### vue 3.0 变化
**vue3.0的改进思路**
vue最主要的特点就是响应式机制、模板、以及对象式的组件声明语法，而3.0对这三部分都做了更改。
1. 响应式
2.x的响应式是基于Object.defineProperty实现的代理，兼容主流浏览器和ie9以上的ie浏览器，能够监听数据对象的变化，但是监听不到对象属性的增删、数组元素和长度的变化，同时会在vue初始化的时候把所有的Observer都建立好，才能观察到数据对象属性的变化。

针对上面的问题，3.0进行了革命性的变更，采用了ES2015的Proxy来代替Object.defineProperty，可以做到监听对象属性的增删和数组元素和长度的修改，还可以监听Map、Set、WeakSet、WeakMap，同时还实现了惰性的监听，不会在初始化的时候创建所有的Observer，而是会在用到的时候才去监听。但是，虽然主流的浏览器都支持Proxy，ie系列却还是不兼容，所以针对ie11，vue3.0决定做单独的适配，暴露出来的api一样，但是底层实现还是Object.defineProperty，这样导致了ie11还是有2.x的问题。但是绝大部分情况下，3.0带来的好处已经能够体验到了。

响应式方面，vue3.0做了实现机制的变更，采用ES2015的Proxy，不但解决了vue2.x中的问题，还是得性能有了进一步提升。虽然有一些兼容问题，但是通过适配的方式解决掉了。此外，还暴露了observable的api来创建响应式对象，可以替代掉event bus，来做一些跨组件的通信。

2. 模板
模板方面没有大的变更，只改了作用域插槽，2.x的机制导致作用域插槽变了，父组件会重新渲染，而3.0把作用于插槽改成了函数的方式，这样只会影响子组件的重新渲染，提升了渲染的性能。

同时，对于render函数的方面，vue3.0也会进行一系列更改来方便习惯直接使用api来生成vdom的开发者。

3. 对象式的组件声明方式
vue2.x中的组件是通过声明的方式传入一系列option，和TypeScript的结合需要通过一些装饰器的方式来做，虽然能实现功能，但是比较麻烦。

3.0修改了组件的声明方式，改成了类式的写法，这样使得和TypeScript的结合变得很容易。

此外，vue的源码也改用了TypeScript来写。其实当代码的功能复杂之后，必须有一个静态类型系统来做一些辅助管理，如React使用的Flow，Angular使用的TypeScript。现在vue3.0也全面改用TypeScript来重写了，更是使得对外暴露的api更容易结合TypeScript。静态类型系统对于复杂代码的维护确实很有必要。

**总结**
vue3.0对vue的主要3个特点：响应式、模板、对象式的组件声明方式，进行了全面的更改，底层的实现和上层的api都有了明显的变化，基于Proxy重新实现了响应式，基于treeshaking内置了更多功能，提供了类式的组件声明方式。而且源码全部用typescript重写。以及进行了一系列的性能优化。

### Vue slot
slot 实际上是一个抽象元素，有点类似template，设计思想有点类似面向对象中的多态，用于组件中某一项需要单独定义，那么就应该使用solt。核心概念是：组件当中某一项，可能是一个元素，也可能只是一个文本。。。。

举例说明下： 项目中需要一个模态框，包括成功和失败两种情况，其中该模态框有文案和背景图片差异，那么模态框可以看作一个组件，而文案和背景图片就可以用slot。

### 数据绑定的实现方法
* 数据劫持(vue)：通过Object.defineProperty() 去劫持数据每个属性对应的getter和setter
* 脏值检测(angular)：通过特定事件比如input，change，xhr请求等进行脏值检测。
* 发布-订阅模式(backbone)：通过发布消息，订阅消息进行数据和视图的绑定监听。

### MVVM
参考：
[合格前端系列第三弹-实现一个属于我们自己的简易MVVM库](https://zhuanlan.zhihu.com/p/27028242)
[剖析Vue实现原理 - 如何实现双向绑定mvvm](https://github.com/DMQ/mvvm)

![mvvm1.jpg](/images/front-end-interview/mvvm1.jpg)

如上图所示，我们可以看到，整体实现分为四步

1、实现一个Observer，对数据进行劫持，通知数据的变化
2、实现一个Compile，对指令进行解析，初始化视图，并且订阅数据的变更，绑定好更新函数
3、实现一个Watcher，将其作为以上两者的一个中介点，在接收数据变更的同时，让Dep添加当前Watcher，并及时通知视图进行update
4、实现MVVM，整合以上三者，作为一个入口函数

### 谈Vue的依赖追踪系统 ——搞懂methods watch和compute的区别和联系
* methods里面定义的函数，是需要主动调用的，而和watch和computed相关的函数，会自动调用,完成我们希望完成的作用。watch、computed是响应式的，methods并非响应式。都是希望在依赖数据发生改变的时候，被依赖的数据根据预先定义好的函数，发生“自动”的变化。但watch和computed也有明显不同的地方：

watch和computed各自处理的数据关系场景不同

watch擅长处理的场景：一个数据影响多个数据
computed擅长处理的场景：一个数据受多个数据影响

计算属性在依赖数据项的初始化时就会执行，但watch在watch对象初始化时并不会执行。

* methods里面定义的是函数，你显然需要像"fuc()"这样去调用它（假设函数为fuc）computed是计算属性，事实上和和data对象里的数据属性是同一类的（使用上）,watch:类似于监听机制+事件机制：例如：
``` js
watch: {
   firstName: function (val) {  this.fullName = val + this.lastName }
}
```
firstName的改变是这个特殊“事件”被触发的条件，而firstName对应的函数就相当于监听到事件发生后执行的方法

* computed是带缓存的，只有其引用的响应式属性发生改变时才会重新计算，而methods里的函数在每次调用时都要执行。
* computed中的成员可以只定义一个函数作为只读属性，也可以定义get/set变成可读写属性，这点是methods中的成员做不到的
* computed是以对象的属性方式存在的，在视图层直接调用就可以得到值，而methods必须以函数形式调用,例如：
``` html
<!-- computed -->
<div>{{msg}}</div>
<!-- methods -->
<div>{{msg()}}</div>
```
可见，computed直接以对象属性方式调用，而methods必须要函数执行才可以得到结果。

### 生命周期
[Vue2.0 探索之路——生命周期和钩子函数的一些理解](https://segmentfault.com/a/1190000008010666?utm_source=tag-newest)

## CSS 面试题

### opacity:0,visibility:hidden,display:none的区别

#### opacity=0

该元素隐藏起来了，但不会改变页面布局，并且，如果该元素已经绑定一些事件，如click事件，那么点击该区域，也能触发点击事件的。

#### visibility=hidden

该元素隐藏起来了，但不会改变页面布局，但是不会触发该元素已经绑定的事件。

#### display=none

把元素隐藏起来，并且会改变页面布局，可以理解成在页面中把该元素删除掉一样。

### 外边距折叠

#### 定义

外边距折叠就是块的顶部外边距和底部外边距有时被组合(折叠)为单个外边距，其大小是组合到其中的最大外边距，这种行为称为外边距塌陷(margin collapsing)

#### 什么情况下会触发外边距折叠

**两个或多个毗邻的普通流中的块元素垂直方向上的 margin 会折叠** 。

##### 一、**两个或多个** 

说明其数量必须是大于一个，又说明，折叠是元素与元素间相互的行为，不存在 A 和 B 折叠，B 没有和 A 折叠的现象。 

##### 二、**毗邻** 

是指没有被非空内容、padding、border 或 clear 分隔开，说明其位置关系。
**注意：**

* 在没有被分隔开的情况下，一个元素的 margin-top 会和它普通流中的第一个子元素(*非浮动元素等*)的 margin-top 相邻；             
* 只有在一个元素的 height 是 "auto" 的情况下，它的 margin-bottom 才会和它普通流中的最后一个子元素(*非浮动元素等*)的 margin-bottom 相邻。

##### 三、垂直方向 

是指具体的方位，只有垂直方向的 margin 才会折叠，也就是说，水平方向的 margin 不会发生折叠的现象。 

#### 外边距折叠的计算

外边距折叠的计算一般是以下三种方法 ：

* 如果两个外边距都为正数，那么取其中较大的数 。
* 如果一个为正数一个为负数，那么取它们的代数和 。
* 如果两个都为负数，那么取它们其中绝对值大的数。

#### 如何解决外边距折叠

**一、浮动元素、inline-block 元素、绝对定位元素的 margin 不会和垂直方向上其他元素的 margin 折叠（注意这里指的是上下相邻的元素）** 

**二、创建了块级格式化上下文的元素，不和它的子元素发生 margin 折叠（注意这里指的是创建了BFC的元素和它的子元素不会发生折叠）** 

我们都知道触发BFC的因素是**float（除了none）、overflow（除了visible）、display（table-cell/table-caption/inline-block）、position（除了static/relative）**  

**很明显大家可以看出来相邻元素不发生折叠的因素是触发BFC因素的子集**，也就是说**如果我为上下相邻的元素设置了overflow:hidden，虽然触发了BFC，但是上下元素的上下margin还是会发生折叠** 。

### css 清除浮动
**总括：**在非IE浏览器（如Firefox）下，当容器的高度为auto，且容器的内容中有浮动（float为left或right）的元素，在这种情况下，容器的高度不能自动伸长以适应内容的高度，使得内容溢出到容器外面而影响（甚至破坏）布局的现象。这个现象叫浮动溢出，为了防止这个现象的出现而进行的CSS处理，就叫CSS清除浮动。
引用W3C的例子，news容器没有包围浮动的元素：
``` js
.news {
  background-color: gray;
  border: solid 1px black;
  }
.news img {
  float: left;
  }
.news p {
  float: right;
  }
<div class="news">
<img src="news-pic.jpg" />
<p>some text</p>
</div>
```

#### 使用带clear属性的空元素
在浮动元素后使用一个空元素如：
``` html
<div class="clear"></div>
```
，并在CSS中赋予:
``` css
.clear{clear:both;}
```
属性即可清理浮动。亦可使用
``` html
<br class="clear" />或<hr class="clear" />
```
来进行清理。
``` js
.news {
  background-color: gray;
  border: solid 1px black;
  }
.news img {
  float: left;
  }
.news p {
  float: right;
  }
.clear {
  clear: both;
  }
<div class="news">
<img src="news-pic.jpg" />
<p>some text</p>
<div class="clear"></div>
</div>
```
优点：简单，代码少，浏览器兼容性好。
缺点：需要添加大量无语义的html元素，代码不够优雅，后期不容易维护。

#### 使用CSS的overflow属性
给浮动元素的容器添加:
```js
 overflow:hidden;
```
或
``` js
overflow:auto;
```
可以清除浮动，另外在 IE6 中还需要触发 hasLayout ，例如为父元素设置容器宽高或设置 zoom:1。在添加overflow属性后，浮动元素又回到了容器层，把容器高度撑起，达到了清理浮动的效果。
``` js
.news {
  background-color: gray;
  border: solid 1px black;
  overflow: hidden;
  *zoom: 1;
  }
.news img {
  float: left;
  }
.news p {
  float: right;
  }
<div class="news">
<img src="news-pic.jpg" />
<p>some text</p>
</div>
```
#### 给浮动的元素的容器添加浮动
给浮动元素的容器也添加上浮动属性即可清除内部浮动，但是这样会使其整体浮动，影响布局，不推荐使用。

#### 使用邻接元素处理
什么都不做，给浮动元素后面的元素添加clear属性。
``` js
.news {
  background-color: gray;
  border: solid 1px black;
  }
.news img {
  float: left;
  }
.news p {
  float: right;
  }
.content{
  clear:both;
  }
<div class="news">
<img src="news-pic.jpg" />
<p>some text</p>
<div class="content">***</div>
</div>
```
注意这里的div.content有内容。

#### 使用CSS的:after伪元素
结合 :after 伪元素（注意这不是伪类，而是伪元素，代表一个元素之后最近的元素）和 IEhack ，可以完美兼容当前主流的各大浏览器，这里的 IEhack 指的是触发 hasLayout。
给浮动元素的容器添加一个clearfix的class，然后给这个class添加一个:after伪元素实现元素末尾添加一个看不见的块元素（Block element）清理浮动。
``` js
.news {
  background-color: gray;
  border: solid 1px black;
  }
.news img {
  float: left;
  }
.news p {
  float: right;
  }
.clearfix:after{
  content: "020";
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;  
  }
.clearfix {
  /* 触发 hasLayout */
  zoom: 1;
  }
<div class="news clearfix">
<img src="news-pic.jpg" />
<p>some text</p>
</div>
```
通过CSS伪元素在容器的内部元素最后添加了一个看不见的空格”020”或点”.”，并且赋予clear属性来清除浮动。需要注意的是为了IE6和IE7浏览器，要给clearfix这个class添加一条zoom:1;触发haslayout。

#### 总结
通过上面的例子，我们不难发现清除浮动的方法可以分成两类：
* 一是利用 clear 属性，包括在浮动元素末尾添加一个带有 clear: both 属性的空 div 来闭合元素，其实利用 :after 伪元素的方法也是在元素末尾添加一个内容为一个点并带有 clear: both 属性的元素实现的。

* 二是触发浮动元素父元素的 BFC (Block Formatting Contexts, 块级格式化上下文)，使到该父元素可以包含浮动元素，关于这一点。

在网页主要布局时使用:after伪元素方法并作为主要清理浮动方式；在小模块如ul里使用overflow:hidden;（留意可能产生的隐藏溢出元素问题）；如果本身就是浮动元素则可自动清除内部浮动，无需格外处理；正文中使用邻接元素清理之前的浮动。

### CSS实现水平垂直居中
仅居中元素定宽高适用：
* absolute + 负margin
* absolute + margin auto
* absolute + calc

居中元素不定宽高：
* absolute + transform
* lineheight
* writing-mode
* table
* css-table
* flex
* grid

#### absolute + 负margin
为了实现上面的效果先来做些准备工作，假设HTML代码如下，总共两个元素，父元素和子元素：
``` html
<div class="wp">
  <div class="box size">123123</div>
</div>
```
wp是父元素的类名，box是子元素的类名，因为有定宽和不定宽的区别，size用来表示指定宽度，下面是所有效果都要用到的公共代码，主要是设置颜色和宽高：
**注意：后面不在重复这段公共代码，只会给出相应提示**
``` css
/* 公共代码 */
.wp {
    border: 1px solid red;
    width: 300px;
    height: 300px;
}

.box {
    background: green;    
}

.box.size{
    width: 100px;
    height: 100px;
}
/* 公共代码 */
```
绝对定位的百分比是相对于父元素的宽高，通过这个特性可以让子元素的居中显示，但绝对定位是基于子元素的左上角，期望的效果是子元素的中心居中显示。
为了修正这个问题，可以借助外边距的负值，负的外边距可以让元素向相反方向定位，通过指定子元素的外边距为子元素宽度一半的负值，就可以让子元素居中了，css代码如下：
``` css
/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
    position: relative;
}
.box {
    position: absolute;;
    top: 50%;
    left: 50%;
    margin-left: -50px;
    margin-top: -50px;
}
```
这是我比较常用的方式，这种方式比较好理解，兼容性也很好，缺点是需要知道子元素的宽高。

#### absolute + margin auto
这种方式也要求居中元素的宽高必须固定，HTML代码如下：
``` html
<div class="wp">
  <div class="box size">123123</div>
</div>
```
这种方式通过设置各个方向的距离都是0，此时再讲margin设为auto，就可以在各个方向上居中了：
``` css
/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
    position: relative;
}
.box {
    position: absolute;;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    margin: auto;
}
```
#### absolute + calc
这种方式也要求居中元素的宽高必须固定，所以我们为box增加size类，HTML代码如下：
``` html
<div class="wp">
    <div class="box size">123123</div>
</div>
```
感谢css3带来了计算属性，既然top的百分比是基于元素的左上角，那么在减去宽度的一半就好了，代码如下:
``` css
/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
  position: relative;
}
.box {
  position: absolute;;
  top: calc(50% - 50px);
  left: calc(50% - 50px);
}
```
这种方法兼容性依赖calc的兼容性，缺点是需要知道子元素的宽高。
#### absolute + transform
还是绝对定位，但这个方法不需要子元素固定宽高，所以不再需要size类了，HTML代码如下：
``` html
<div class="wp">
  <div class="box">123123</div>
</div>
```
修复绝对定位的问题，还可以使用css3新增的transform，transform的translate属性也可以设置百分比，其是相对于自身的宽和高，所以可以讲translate设置为-50%，就可以做到居中了，代码如下：
``` css
/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
  position: relative;
}
.box {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}
```
这种方法兼容性依赖translate2d的兼容性。

#### lineheight
利用行内元素居中属性也可以做到水平垂直居中，HTML代码如下：
``` html
<div class="wp">
    <div class="box">123123</div>
</div>
```
把box设置为行内元素，通过text-align就可以做到水平居中，但很多同学可能不知道通过通过vertical-align也可以在垂直方向做到居中，代码如下：
``` css
/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
    line-height: 300px;
    text-align: center;
    font-size: 0px;
}
.box {
    font-size: 16px;
    display: inline-block;
    vertical-align: middle;
    line-height: initial;
    text-align: left; /* 修正文字 */
}
```
这种方法需要在子元素中将文字显示重置为想要的效果。

#### writing-mode
很多同学一定和我一样不知道writing-mode属性，感谢@张鑫旭老师的反馈，简单来说writing-mode可以改变文字的显示方向，比如可以通过writing-mode让文字的显示变为垂直方向：
``` html
<div class="div1">水平方向</div>
<div class="div2">垂直方向</div>
```
``` css
.div2 {
    writing-mode: vertical-lr;
}
```
显示效果如下：
``` js
水平方向
垂
直
方
向
```
更神奇的是所有水平方向上的css属性，都会变为垂直方向上的属性，比如text-align，通过writing-mode和text-align就可以做到水平和垂直方向的居中了，只不过要稍微麻烦一点:
``` html
<div class="wp">
    <div class="wp-inner">
        <div class="box">123123</div>
    </div>
</div>
```
``` css
/* 此处引用上面的公共代码 */
/* 此处引用上面的公共代码 */

/* 定位代码 */
.wp {
    writing-mode: vertical-lr;
    text-align: center;
}
.wp-inner {
    writing-mode: horizontal-tb;
    display: inline-block;
    text-align: center;
    width: 100%;
}
.box {
    display: inline-block;
    margin: auto;
    text-align: left;
}
```
这种方法实现起来和理解起来都稍微有些复杂。

#### table
曾经table被用来做页面布局，现在没人这么做了，但table也能够实现水平垂直居中，但是会增加很多冗余代码：
``` html
<table>
    <tbody>
        <tr>
            <td class="wp">
                <div class="box">123123</div>
            </td>
        </tr>
    </tbody>
</table>
```
tabel单元格中的内容天然就是垂直居中的，只要添加一个水平居中属性就好了:
``` css
.wp {
    text-align: center;
}
.box {
    display: inline-block;
}
```
这种方法就是代码太冗余，而且也不是table的正确用法。

#### css-table
css新增的table属性，可以让我们把普通元素，变为table元素的现实效果，通过这个特性也可以实现水平垂直居中
``` html
<div class="wp">
    <div class="box">123123</div>
</div>
```
下面通过css属性，可以让div显示的和table一样
``` css
.wp {
    display: table-cell;
    text-align: center;
    vertical-align: middle;
}
.box {
    display: inline-block;
}
```
这种方法和table一样的原理，但却没有那么多冗余代码，兼容性也还不错。

#### flex
flex作为现代的布局方案，颠覆了过去的经验，只需几行代码就可以优雅的做到水平垂直居中
``` html
<div class="wp">
    <div class="box">123123</div>
</div>
```
``` css
.wp {
    display: flex;
    justify-content: center;
    align-items: center;
}
```
目前在移动端已经完全可以使用flex了，PC端需要看自己业务的兼容性情况。

#### grid
感谢@一丝姐 反馈的这个方案，css新出的网格布局，由于兼容性不太好，一直没太关注，通过grid也可以实现水平垂直居中
``` html
<div class="wp">
    <div class="box">123123</div>
</div>
```
``` css
.wp {
    display: grid;
}
.box {
    align-self: center;
    justify-self: center;
}
```
代码量也很少，但兼容性不如flex，不推荐使用。

#### 总结
下面对比下各个方式的优缺点，肯定又双叒叕该有同学说回字的写法了，简单总结下
* PC端有兼容性要求，宽高固定，推荐absolute + 负margin
* PC端有兼容要求，宽高不固定，推荐css-table
* PC端无兼容性要求，推荐flex
* 移动端推荐使用flex
| 方法      | 居中元素定宽高固定        | PC兼容性     | 移动端兼容性        | 
| ------     | ------     | ------     | ------        |
| absolute + 负margin   | 是 | ie6+, chrome4+, firefox2+        |安卓2.3+, iOS6+|
| absolute + margin auto     | 是 |ie6+, chrome4+, firefox2+     | 安卓2.3+, iOS6+        |
| absolute + calc     | 是 |ie9+, chrome19+, firefox4+     | 安卓4.4+, iOS6+      |
| absolute + transform     |否 |ie9+, chrome4+, firefox3.5+     | 安卓3+, iOS6+      |
| writing-mode     | 否 |ie6+, chrome4+, firefox3.5+     | 安卓2.3+, iOS5.1+    |
| lineheight     | 否 |ie6+, chrome4+, firefox2+   | 安卓2.3+, iOS6+      |
| table     | 否 |ie6+, chrome4+, firefox2+     | 安卓2.3+, iOS6+     |
| css-table     | 否 |ie8+, chrome4+, firefox2+     | 安卓2.3+, iOS6+    |
| flex     | 否 |ie10+, chrome4+, firefox2+    | 安卓2.3+, iOS6+     |
| grid     | 否 |ie10+, chrome57+, firefox52+    | 安卓6+, iOS10.3+     |

### BFC
BFC(block formatting context)翻译为“块级格式化上下文”，它会生成一个独立的渲染区域(不影响外面的元素，同时也不受外面元素的影响)，它的生成有以下规则：
* 内部的box会在垂直方向上一个接一个的放置
* 内部box在垂直方向上的距离由margin决定，同属一个BFC内的相邻box会发生margin重叠
* 每一个内部box的左边，与BFC的的左边相接触，即使存在浮动也是一样
* BFC的区域不会与float box发生重叠
* 计算BFC的高度时，浮动元素也参与计算(上面清除浮动的问题就是这个原理)

触发BFC的条件：
* 根元素
* float属性不为none
* position为absolute或者fixed
* display为inline-block、table-cell、table-caption、flex、inline-flex
* overflow不为visible

### 实现自适应两列布局
#### 右边元素触发BFC
``` html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>未知宽高元素水平垂直居中</title>
</head>
<style>
    .father {
        background-color: lightblue;
    }

    .left {
        float: left;
        width: 100px;
        height: 200px;
        background-color: red;
    }

    .right {
        overflow: auto;
        height: 500px;
        background-color: lightseagreen
    }
</style>

<body>
    <div class="father">
        <div class='left'>left</div>
        <div class='right'>
            right
        </div>
    </div>
</body>

</html>
```
#### margin-left实现
局限性：这种方法必须知道左侧的宽度。
``` html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>未知宽高元素水平垂直居中</title>
</head>
<style>
    .father {
        background-color: lightblue;
    }

    .left {
        width: 100px;
        float: left;
        background-color: red;
    }

    .right {
        margin-left: 100px;
        background-color: lightseagreen
    }
</style>

<body>
    <div class="father">
        <div class='left'>left</div>
        <div class='right'>
            right
        </div>
    </div>
</body>

</html>
```
### 三列布局
#### flex
优点：方便 
缺点：还是flex兼容性
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>未知宽高元素水平垂直居中</title>
</head>
<style>
    .father {
        display: flex;
        height: 100%;
    }

    .left,
    .right {
        flex: 0 1 100px;
        background-color: red;
    }

    .middle {
        flex: 1;
        height: 100%;
        background-color: green;
    }
</style>
<body>
    <div class="father">
        <div class='left'>left</div>
        <div class='middle'>middle</div>
        <div class='right'>center</div>
    </div>
</body>
</html>
```
#### 负margin布局(双飞翼)
优点：市面上使用最多的一个
缺点：麻烦，这是多年前淘宝的老技术了
``` html
<!DOCTYPE>
<html>

<head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8" />
    <title>圣杯布局/双飞翼布局</title>
    <style>
        .mainWrap {
            width: 100%;
            float: left;
        }

        .main {
            margin: 0 120px;
        }

        .left,
        .right {
            float: left;
            width: 100px;
        }

        .left {
            margin-left: -100%;
        }

        .right {
            margin-left: -100px;
        }
    </style>
</head>

<div class="parent" id="parent" style="background-color: lightgrey;">
    <div class="mainWrap">
        <div class="main" style="background-color: lightcoral;">
            main
        </div>
    </div>
    <div class="left" style="background-color: orange;">
        left
    </div>
    <div class="right" style="background-color: lightsalmon;">
        right
    </div>
</div>

</html>
```
### 列举HTML5新特性
* 语意化标签(nav、aside、dialog、header、footer等)
* canvas
* 拖放相关api
* Audio、Video
* 获取地理位置
* 更好的input校验
* web存储(localStorage、sessionStorage)
* webWorkers(类似于多线程并发)
* webSocket

### 列举Css3新特性
1、颜色：新增RGBA、HSLA模式
2、文字阴影(text-shadow)
3、边框：圆角（border-radius）边框阴影：box-shadow
4、盒子模型：box-sizing
5、背景：background-size设置背景图片的尺寸，background-origin设置背景图片的原点，background-clip设置背景图片的裁剪区域，以“，”分隔可以设置多背景，用于自适应布局
6、渐变：linear-gradient、radial-gradient
7、过渡：transition可实现动画
8、自定义动画
9、在CSS3中唯一引入的伪元素是::selection
10、多媒体查询、多栏布局
11、border-image
12、2D转换：transform:translate(x,y)rotate(x,y)skew(x,y)scale(x,y)
13、3D转换

### transition和animation的区别是什么？
过渡属性transition可以在一定的事件内实现元素的状态过渡为最终状态，用于模拟一种过渡动画效果，但是功能有限，只能用于制作简单的动画效果；

动画属性animation可以制作类似Flash动画，通过关键帧控制动画的每一步，控制更为精确，从而可以制作更为复杂的动画。

### 伪类与伪元素
具体参考：[CSS中伪类与伪元素](https://zhuanlan.zhihu.com/p/46909886)
先说一说为什么css要引入伪元素和伪类，以下是css2.1 Selectors章节中对伪类与伪元素的描述：
CSS introduces the concepts of pseudo-elements and pseudo-classes to permit formatting based on information that lies outside the document tree.

直译过来就是：css引入伪类和伪元素概念是为了格式化文档树以外的信息。也就是说，伪类和伪元素是用来修饰不在文档树中的部分，比如，一句话中的第一个字母，或者是列表中的第一个元素。下面分别对伪类和伪元素进行解释。

伪类用于当已有元素处于的某个状态时，为其添加对应的样式，这个状态是根据用户行为而动态变化的。比如说，当用户悬停在指定的元素时，我们可以通过:hover来描述这个元素的状态。虽然它和普通的css类相似，可以为已有的元素添加样式，但是它只有处于dom树无法描述的状态下才能为元素添加样式，所以将其称为伪类。

伪元素用于创建一些不在文档树中的元素，并为其添加样式。比如说，我们可以通过:before来在一个元素前增加一些文本，并为这些文本添加样式。虽然用户可以看到这些文本，但是这些文本实际上不在文档树中。

#### 伪类与伪元素的区别
伪类与伪元素的区别在于：有没有创建一个文档树之外的元素。

### css加载会造成阻塞吗？
具体参考：[css加载会造成阻塞吗？](https://zhuanlan.zhihu.com/p/43282197)

### css 回流与重绘
具体参考：[重构与回流](https://www.cnblogs.com/jiayuexuan/p/7490140.html)

### css 盒子模型
具体参考：[从CSS盒子模型说起](https://zhuanlan.zhihu.com/p/27839418)

标准盒子模型：宽度=内容的宽度（content）+ border + padding + margin

低版本IE盒子模型：宽度=内容宽度（content+border+padding）+ margin

### Flex
具体参考：
* [Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html?utm_source=tuicool))
* [Flex 布局教程：实例篇](http://www.ruanyifeng.com/blog/2015/07/flex-examples.html)

### box-sizing属性
用来控制元素的盒子模型的解析模式，默认为content-box

context-box：W3C的标准盒子模型，设置元素的 height/width 属性指的是content部分的高/宽

border-box：IE传统盒子模型。设置元素的height/width属性指的是border + padding + content部分的高/宽

### CSS选择器有哪些？哪些属性可以继承？
具体参考：[css选择器](http://www.sosout.com/2018/08/01/css-selector.html)

#### 哪些属性可以自动继承呢
1、字体系列属性
font：组合字体
font-family：规定元素的字体系列
font-weight：设置字体的粗细
font-size：设置字体的尺寸
font-style：定义字体的风格
font-variant：设置小型大写字母的字体显示文本，这意味着所有的小写字母
均会被转换为大写，但是所有使用小型大写字体的字母与其余文本
相比，其字体尺寸更小。
font-stretch：允许你使文字变宽或变窄。所有主流浏览器都不支持。
font-size-adjust：为某个元素规定一个 aspect 值，字体的小写字母 "x"
的高度与"font-size" 高度之间的比率被称为一个字体的 aspect 值。
这样就可以保持首选字体的 x-height

2、文本系列属性
text-indent：文本缩进
text-align：文本水平对齐
text-shadow：设置文本阴影
line-height：行高
word-spacing：增加或减少单词间的空白（即字间隔）
letter-spacing：增加或减少字符间的空白（字符间距）
text-transform：控制文本大小写
direction：规定文本的书写方向
color：文本颜色
3、元素可见性：visibility
4、表格布局属性：caption-side
border-collapse
empty-cells
5、列表属性：list-style-type
list-style-image
list-stye-position、list-style
6、设置嵌套引用的引号类型：quotes
7、光标属性：cursor

#### 不可继承的属性
1、display
2、文本属性：vertical-align
text-decoration
3、盒子模型的属性:宽度、高度、内外边距、边框等
4、背景属性：背景图片、颜色、位置等
5、定位属性：浮动、清除浮动、定位position等
6、生成内容属性:content、counter-reset、counter-increment
7、轮廓样式属性:outline-style、outline-width、outline-color、outline
8、页面样式属性:size、page-break-before、page-break-after

**注意**
继承中比较特殊的几点
* a 标签的字体颜色不能被继承
* `<h1>-<h6>`标签字体的大下也是不能被继承的
因为它们都有一个默认值

### display有哪些值？说明他们的作用?
block 块类型。默认宽度为父元素宽度，可设置宽高，换行显示。
none 缺省值。像行内元素类型一样显示。
inline 行内元素类型。默认宽度为内容宽度，不可设置宽高，同行显示。
inline-block 默认宽度为内容宽度，可以设置宽高，同行显示。
list-item 像块类型元素一样显示，并添加样式列表标记。
table 此元素会作为块级表格来显示。
inherit 规定应该从父元素继承display属性的值。

### position的值
具体参考：
* [position的四个属性值： relative ，absolute ，fixed，static](http://www.cnblogs.com/chinafine/articles/1765967.html)

static（默认）：按照正常文档流进行排列；

relative（相对定位）：不脱离文档流，参考自身静态位置通过 top, bottom, left, right 定位；

absolute(绝对定位)：参考距其最近一个不为static的父级元素通过top, bottom, left, right 定位；

fixed(固定定位)：所固定的参照对像是可视窗口。

### 如何解决使用inline-block引起的空白间隙的问题
具体参考：
* [前端面试题（一）：如何解决使用inline-block引起的空白间隙的问题](https://blog.csdn.net/lizhengxv/article/details/80385623)
* [有哪些好方法能处理 display: inline-block 元素之间出现的空格？](https://www.zhihu.com/question/21468450/answer/18342657)
相邻的 inline-block 元素之间有换行或空格分隔的情况下会产生间距
非 inline-block 水平元素设置为 inline-block 也会有水平间距
可以借助 vertical-align:top; 消除垂直间隙
可以在父级加 font-size：0; 在子元素里设置需要的字体大小，消除垂直间隙
把 li 标签写到同一行可以消除垂直间隙，但代码可读性差

### font-style 属性 oblique 是什么意思？
ont-style: oblique; 使没有 italic 属性的文字实现倾斜

### line-height 三种赋值方式有何区别？（带单位、纯数字、百分比）
带单位：px 是固定值，而 em 会参考父元素 font-size 值计算自身的行高
纯数字：会把比例传递给后代。例如，父级行高为 1.5，子元素字体为 18px，则子元素行高为 1.5 * 18 = 27px
百分比：将计算后的值传递给后代

### 在 CSS 中，用 float 和 position 的区别是什么？
具体参考：
* [在 CSS 中，用 float 和 position 的区别是什么？](https://www.zhihu.com/question/19588854/answer/12309368)

### 如何在页面引用 CSS
* 外部引用@import引用样式；
* 外部样式表。引入一个外部CSS文件；
* 内部样式表。将CSS代码放在<head>标签内部；
* 内联样式，将CSS样式直接定义在HTML元素内部；

**注意：link标签与@import有区别：**
* link属于XHTML标签，而@import完全是CSS提供的一种方式。
link标签除了可以加载CSS外，还可以做很多其它的事情，比如定义RSS，定义rel连接属性等，@import就只能加载CSS了。
* 加载顺序的差别。当一个页面被加载的时候，link引用的CSS会同时被加 载，而@import引用的CSS会等到页面全部被下载完再被加载。所以有时候浏览@import加载CSS的页面时开始会没有样式（就是闪烁）。
* 兼容性的差别。由于@import是CSS2.1提出的所以老的浏览器不支持，@import只有在IE5以上的才能识别，而link标签无此问题。
* @import只能css中再次引入其他样式表，比如可以创建一个主样式表，在主样式表中再引入其他的样式表，这样做有一个缺点，会对网站服务器产生较多的HTTP请求

### 行内元素和块状元素的区别？行内快元素的兼容性使用？（IE8以下）

块级元素（block）特性：
总是独占一行，表现为另起一行开始，而且其后的元素也必须另起一行显示。
width、height、padding（内边距）、margin(外边距)都可控制。
内联元素（inline）特性：
宽度、高度、内边距的padding-top/padding-bottom和外边距的margin-top、margin-bottom都不可改变（也就是padding和margin的left和right是可以设置的）。
这里还有其他问题。浏览器还有默认的天生inline-block元素（拥有内在尺寸，可设置高宽，但不会自动换行），有哪些元素是天生inline-block元素？
它们是`<input>、<img>、<button>、<textare>、<label>` 。

兼容性：display:inline-block;display:inline;zoom:1;

### 文本text
text-align：文本对其方式 left、center、right、justify
text-indent：文案第一行缩进距离
text-decoration： none、underline、line-through、overline
color：文字颜色
text-transform：改变文字大小写none、uppercase、lowercase、capitalize
word-spacing：可以改变字（单词）之间的标准间隔
letter-spacing：字母间隔修改的是字符或字母之间的间隔

#### 单行文本溢出加 ...如何实现?
``` css
.card {
  white-space:nowrap;/* 文本不会换行，文本会在在同一行上继续 */
  overflow: hidden;
  text-overflow: ellipsis;/* 显示省略符号来代表被修剪的文本。 */  
}
```
### 超链接访问过后hover样式就不出现的问题时什么？如何解决？
被点击访问过的超链接样式不再具有hover和active了，解决方式是改变CSS属性的排列顺序：L-V-H-A（linked, visited, hover, active）。L-V-H-A love hate 用喜欢和讨厌两个词来方便记忆。

### 什么是视差滚动效果，如何给每页做不同的动画？
视差滚动是指多层背景以不同的速度移动，形成立体的运动效果，具有非常出色的视觉体验

一般把网页解剖为：背景层、内容层和悬浮层。当滚动鼠标滚轮时，各图层以不同速度移动，形成视差的

实现原理

以 “页面滚动条” 作为 “视差动画进度条”
以 “滚轮刻度” 当作 “动画帧度” 去播放动画的
监听 mousewheel 事件，事件被触发即播放动画，实现“翻页”效果

### rgba()和opacity的透明效果有什么区别？
rgba()和opacity都能实现透明效果，但最大的不同是opacity作用于元素，以及元素内的所有内容的透明度，而rgba()只作用于元素的颜色或起背景色。设置rgba透明的元素的子元素不会继承透明效果。

### CSS中可以让文字在垂直和水平方向重叠的两个属性分别是什么？
垂直方向：line-height。设置成比字体高度还小就可以让两行重叠
水平方向：letter-spacing。设置为负值即可实现重叠。

### 阐述一下CSS Sprites
将一个页面涉及到的所有图片都包含到一张大图中去，然后利用CSS的 background-image，background- repeat，background-position 的组合进行背景定位。利用CSS Sprites能很好地减少网页的http请求，从而大大的提高页面的性能；CSS Sprites能减少图片的字节。

### style标签写在body后与body前有什么区别？
页面加载自上而下 当然是先加载样式。

写在body标签后由于浏览器以逐行方式对HTML文档进行解析，当解析到写在尾部的样式表（外联或写在style标签）会导致浏览器停止之前的渲染，等待加载且解析样式表完成之后重新渲染，在windows的IE下可能会出现FOUC现象（即样式失效导致的页面闪烁问题）

### png、jpg、gif 这些图片格式解释一下，分别什么时候用。有没有了解过webp？
* png是便携式网络图片（Portable Network Graphics）是一种无损数据压缩位图文件格式.优点是：压缩比高，色彩好。 大多数地方都可以用。

* jpg是一种针对相片使用的一种失真压缩方法，是一种破坏性的压缩，在色调及颜色平滑变化做的不错。在www上，被用来储存和传输照片的格式。

* gif是一种位图文件格式，以8位色重现真色彩的图像。可以实现动画效果.

* webp格式是谷歌在2010年推出的图片格式，压缩率只有jpg的2/3，大小比png小了45%。缺点是压缩的时间更久了，兼容性不好，目前谷歌和opera支持。

### li与li之间有看不见的空白间隔是什么原因引起的？有什么解决办法？
行框的排列会受到中间空白（回车空格）等的影响，因为空格也属于字符,这些空白也会被应用样式，占据空间，所以会有间隔，把字符大小设为0，就没有空格了。

解决方法：

* 可以将`<li>`代码全部写在一排
* 浮动li中float：left
* 在ul中用font-size：0（谷歌不支持）；可以使用letter-space：-3px

### 如果需要手动写动画，你认为最小时间间隔是多久，为什么？
多数显示器默认频率是60Hz，即1秒刷新60次，所以理论上最小间隔为1/60＊1000ms ＝ 16.7ms。

### 让页面里的字体变清晰，变细用CSS怎么做？
-webkit-font-smoothing在window系统下没有起作用，但是在IOS设备上起作用-webkit-font-smoothing：antialiased是最佳的，灰度平滑。

### 怎么让Chrome支持小于12px 的文字？
p {
  font-size:10px;
  -webkit-transform:scale(0.8);
}
// 0.8是缩放比例

### 你对line-height是如何理解的？
line-height 指一行字的高度，包含了字间距，实际上是下一行基线到上一行基线距离
如果一个标签没有定义 height 属性，那么其最终表现的高度是由 line-height 决定的
一个容器没有设置高度，那么撑开容器高度的是 line-height 而不是容器内的文字内容
把 line-height 值设置为 height 一样大小的值可以实现单行文字的垂直居中
line-height 和 height 都能撑开一个高度，height 会触发 haslayout，而 line-height 不会

### 网站图片文件，如何点击下载？而非点击预览？
``` html
<a href="logo.jpg" download>下载</a> <a href="logo.jpg" download="网站LOGO" >下载</a>
```
### input [type=search] 搜索框右侧小图标如何美化？
``` css
input[type="search"]::-webkit-search-cancel-button{
  -webkit-appearance: none;
  height: 15px;
  width: 15px;
  border-radius: 8px;
  background:url("images/searchicon.png") no-repeat 0 0;
  background-size: 15px 15px;
}
```
### 如何修改Chrome记住密码后自动填充表单的黄色背景？
产生原因：由于Chrome默认会给自动填充的input表单加上 input:-webkit-autofill 私有属性造成的
解决方案1：在form标签上直接关闭了表单的自动填充：autocomplete="off"
解决方案2：input:-webkit-autofill { background-color: transparent; }

### ::before 和 :after中双冒号和单冒号有什么区别？解释一下这2个伪元素的作用
* 单冒号(:)用于CSS3伪类，双冒号(::)用于CSS3伪元素。

* ::before就是以一个子元素的存在，定义在元素主体内容之前的一个伪元素。并不存在于dom之中，只存在在页面之中。

:before 和 :after 这两个伪元素，是在CSS2.1里新出现的。起初，伪元素的前缀使用的是单冒号语法，但随着Web的进化，在CSS3的规范里，伪元素的语法被修改成使用双冒号，成为::before ::after

### 什么是响应式设计？响应式设计的基本原理是什么？
响应式网站设计(Responsive Web design)是一个网站能够兼容多个终端，而不是为每一个终端做一个特定的版本。

基本原理是通过媒体查询检测不同的设备屏幕尺寸做处理。

### 全屏滚动的原理是什么？用到了CSS的哪些属性？
* 原理：有点类似于轮播，整体的元素一直排列下去，假设有5个需要展示的全屏页面，那么高度是500%，只是展示100%，剩下的可以通过transform进行y轴定位，也可以通过margin-top实现

* overflow:hidden; transform:translate(100%, 100%); display:none; overflow：hidden；transition：all 1000ms ease；

### CSS优化、提高性能的方法有哪些？
多个css合并，尽量减少HTTP请求
将css文件放在页面最上面
移除空的css规则
避免使用CSS表达式
选择器优化嵌套，尽量避免层级过深
充分利用css继承属性，减少代码量
抽象提取公共样式，减少代码量
属性值为0时，不加单位
属性值为小于1的小数时，省略小数点前面的0
css雪碧图

### 什么是 FOUC(Flash of Unstyled Content)？ 如何来避免 FOUC？
当使用 @import 导入 CSS 时，会导致某些页面在 IE 出现奇怪的现象： 没有样式的页面内容显示瞬间闪烁，这种现象称为“文档样式短暂失效”，简称为FOUC
产生原因：当样式表晚于结构性html加载时，加载到此样式表时，页面将停止之前的渲染。
等待此样式表被下载和解析后，再重新渲染页面，期间导致短暂的花屏现象。
解决方法：使用 link 标签将样式表放在文档 head

### 清除浮动最佳实践（after伪元素闭合浮动）：
``` css
clearfix:after{
    content: "\200B";
    display: table; 
    height: 0;
    clear: both;
  }
  .clearfix{
    *zoom: 1;
  }
```

### 解释下什么是浮动和它的工作原理？
非IE浏览器下，容器不设高度且子元素浮动时，容器高度不能被内容撑开。 此时，内容会溢出到容器外面而影响布局。这种现象被称为浮动（溢出）。
工作原理：
浮动元素脱离文档流，不占据空间（引起“高度塌陷”现象）
浮动元素碰到包含它的边框或者其他浮动元素的边框停留

### 浮动元素引起的问题？

父元素的高度无法被撑开，影响与父元素同级的元素
与浮动元素同级的非浮动元素会跟随其后

### css hack原理及常用hack
原理：利用不同浏览器对CSS的支持和解析结果不一样编写针对特定浏览器样式。
常见的hack有
属性hack：比如IE6能识别下划线“_”和星号“*”，IE7能识别星号“*”，但不能识别下划线”_ ”，而firefox两个都不能认识。
选择器hack：比如IE6能识别*html .class{}，IE7能识别*+html .class{}或者*:first-child+html .class{}。
IE条件注释

### 一个满屏 品 字布局 如何设计?
简单的方式：
上面的div宽100%，
下面的两个div分别宽50%，
然后用float或者inline使其不换行即可

### 用纯CSS创建一个三角形的原理是什么？
``` css
// 把上、左、右三条边隐藏掉（颜色设为 transparent）
#demo {
  width: 0;
  height: 0;
  border-width: 20px;
  border-style: solid;
  border-color: transparent transparent red transparent;
}
```

### hasLayout
hasLayout可以简单看作是IE5.5/6/7中的BFC(Block Formatting Context)。也就是一个元素要么自己对自身内容进行组织和尺寸计算(即可通过width/height来设置自身的宽高)，要么由其containing block来组织和尺寸计算。而IFC（即没有拥有布局）而言，则是元素无法对自身内容进行组织和尺寸计算，而是由自身内容来决定其尺寸（即仅能通过line-height设置内容行距，通过行距来支撑元素的高度；也无法通过width设置元素宽度，仅能由内容来决定而已）
当hasLayout为true时(就是所谓的"拥有布局")，相当于元素产生新BFC，元素自己对自身内容进行组织和尺寸计算;
当hasLayout为false时(就是所谓的"不拥有布局")，相当于元素不产生新BFC，元素由其所属的containing block进行组织和尺寸计算.

### 元素竖向的百分比设定是相对于容器的高度吗？
当按百分比设定一个元素的宽度时，它是相对于父容器的宽度计算的，但是，对于一些表示竖向距离的属性，例如 padding-top , padding-bottom , margin-top , margin-bottom 等，当按百分比设定它们时，依据的也是父容器的宽度，而不是高度。

### margin和padding分别适合什么场景使用？
何时使用margin：

➤需要在border外侧添加空白

➤空白处不需要背景色

➤上下相连的两个盒子之间的空白，需要相互抵消时。

何时使用padding：

➤需要在border内侧添加空白

➤空白处需要背景颜色

➤上下相连的两个盒子的空白，希望为两者之和。

兼容性的问题：在IE5 IE6中，为float的盒子指定margin时，左侧的margin可能会变成两倍的宽度。通过改变padding或者指定盒子的display：inline解决。

### 在网页中的应该使用奇数还是偶数的字体？为什么呢？
使用偶数字体。偶数字号相对更容易和 web 设计的其他部分构成比例关系。Windows 自带的点阵宋体（中易宋体）从 Vista 开始只提供 12、14、16 px 这三个大小的点阵，而 13、15、17 px时用的是小一号的点。（即每个字占的空间大了 1 px，但点阵没变），于是略显稀疏。

### 浏览器是怎样解析CSS选择器的？
CSS选择器的解析是从右向左解析的。若从左向右的匹配，发现不符合规则，需要进行回溯，会损失很多性能。

若从右向左匹配，先找到所有的最右节点，对于每一个节点，向上寻找其父节点直到找到根元素或满足条件的匹配规则，则结束这个分支的遍历。

两种匹配规则的性能差别很大，是因为从右向左的匹配在第一步就筛选掉了大量的不符合条件的最右节点（叶子节点），而从左向右的匹配规则的性能都浪费在了失败的查找上面。

而在 CSS 解析完毕后，需要将解析的结果与 DOM Tree 的内容一起进行分析建立一棵 Render Tree，最终用来进行绘图。

在建立 Render Tree 时（WebKit 中的「Attachment」过程），浏览器就要为每个 DOM Tree 中的元素根据 CSS 的解析结果（Style Rules）来确定生成怎样的 Render Tree。

### CSS优化、提高性能的方法有哪些？
➤避免过度约束

➤避免后代选择符

➤避免链式选择符

➤使用紧凑的语法

➤避免不必要的命名空间

➤避免不必要的重复

➤最好使用表示语义的名字。一个好的类名应该是描述他是什么而不是像什么

➤避免！important，可以选择其他选择器

➤尽可能的精简规则，你可以合并不同类里的重复规则

### 移动端的布局用过媒体查询吗？
通过媒体查询可以为不同大小和尺寸的媒体定义不同的css，适应相应的设备的显示。

➤`<head>`里边

`<link rel="stylesheet" type="text/css" href="xxx.css" media="only screen and (max-device-width:480px)">`

➤CSS : @media only screen and (max-device-width:480px) {/css样式/}

### 设置元素浮动后，该元素的display值是多少？
自动变成display:block

### position跟display、overflow、float这些特性相互叠加后会怎么样？
display属性规定元素应该生成的框的类型；position属性规定元素的定位类型；float属性是一种布局方式，定义元素在哪个方向浮动。

类似于优先级机制：position：absolute/fixed优先级最高，有他们在时，float不起作用，display值需要调整。float 或者absolute定位的元素，只能是块元素或表格。

### 为什么要初始化CSS样式
因为浏览器的兼容问题，不同浏览器对有些标签的默认值是不同的，如果没对CSS初始化往往会出现浏览器之间的页面显示差异。

### absolute的containing block计算方式跟正常流有什么不同？
无论属于哪种，都要先找到其祖先元素中最近的 position 值不为 static 的元素，然后再判断：

➤若此元素为 inline 元素，则 containing block 为能够包含这个元素生成的第一个和最后一个 inline box 的 padding box (除 margin, border 外的区域) 的最小矩形；

➤否则,则由这个祖先元素的 padding box 构成。

如果都找不到，则为 initial containing block。

补充：

➤static(默认的)/relative：简单说就是它的父元素的内容框（即去掉padding的部分）

➤absolute: 向上找最近的定位为absolute/relative的元素

➤fixed: 它的containing block一律为根元素(html/body)

### 前端页面有哪三层构成，分别是什么？作用是什么？
结构层、表示层、行为层
结构层（structural layer）：由HTML或XHTML之类的标记语言负责创建。
表示层（presentation layer）:由css负责创建。
行为层（behaviorlayer）:负责回答“内容应该如何对事件做出反应”这一问题。这是 Javascript 语言和 DOM主宰的领域。

### 简述对Web语义化的理解？
就是让浏览器更好的读懂你写的代码，在进行HTML结构、表现、行为设计时，尽量使用语义化的标签，使程序代码简洁明了，易于进行web操作和网站SEO，方便团队的一种标准，以图实现一种“无障碍”的web开发。

### 合理的页面布局中常听过结构与表现分离，那么结构是什么？表现是什么？
结构是html,表现是css

### 写出video标签中预加载视频用到的属性是什么
preload

### 写一下audio标签中，如何实现音乐的暂停和播放
play()开始,pause()暂停

### box-sizing、transition、translate分别是什么？
1、box-sizing:用来指定模型的大小的计算方式。主要分为border-box(从边框固定盒子大小)、content-box(从内容固定盒子大小)两种计算方式。
2、transition:当前元素只要有"属性"发生变化时，可以平滑的进行过渡。通过transition-propety设置过渡属性；transition-duration设置过渡时间；transition-timing-function设置过渡速度；transition-delay设置过渡延时。
3、translate：通过移动改变元素的位置；有x,y,z三个属性

### iframe的作用？
用法
1、iframe是用来在网页中插入第三方页面，早期的页面使用iframe主要是用于导航栏这种很多页面都相同的部分，这样在切换页面的时候避免重复下载。

优点
1、便于修改，模拟分离，像一些信息管理系统会用到。
2、但现在基本不推荐使用。除非特殊需要，一般不推荐使用。

缺点
1、iframe的创建比一般的DOM元素慢了1-2个数量级
2、iframe标签会阻塞页面的的加载，如果页面的onload事件不能及时触发，会让用户觉得网页加载很慢，用户体验不好，在Safari和Chrome中可以通过js动态设置iframe的src属性来避免阻塞。
3、iframe对于SEO不友好，替换方案一般就是动态语言的Incude机制和ajax动态填充内容等。

### 介绍以下你对浏览器内核的理解？
1、主要分成两部分：渲染引擎（layout engineer或Rendering Engine）和JS引擎。
2、渲染引擎：负责取得网页的内容（HTML、XML、图像等等）、整理讯息（例如加入CSS等）、以及计算网页的显示方式、然后会输出至显示器或打印机。浏览器的内核的不同对于网页的语法解释会有不同、所以渲染的效果也不相同。所有网页浏览器、电子邮件客户端以及其他需要编辑、显示网络内容的应用程序都需要内核
3、JS引擎则：解析和执行javascript来实现网页的动态效果。
4、最开始渲染引擎和JS引擎并没有区分得很明确，后来JS引擎越来越独立，内核九倾向于只指渲染引擎。

### css盒模型有哪些及区别content-box border-box padding-box
IE盒子模型box-sizing:border-box;（怪异模式）

W3C标准盒子模型 box-sizing:content-box;（标准模式）默认模式

content-box:这是默认样式指定CSS标准。测量width和height属性只包括的内容，但不是border, margin, 或者 padding。

padding-box:width和height属性包括padding的大小，不包括border和margin

border-box:width和height属性包括padding和border，但不是margin。这是盒模型的文档时，Internet Explorer使用Quirks模式。

content-box不包含padding，border-box包含padding。所以如果你设置的大小是一样的，content-box看起来，会比border-box大

### html5哪些操作可以SEO优化

title标签；meta标签；header标签；footer标签
元标签（meta标签）；导航标签（nav标签）；文章标签（article标签）；左或右侧标签（aside标签）

### html document是干嘛的？
HTML即：超文本标记语言，标准通用标记语言的一个应用，“超文本”就是指页面内可以包含图片、链接、甚至音乐、程序等非文字元素。
HTML Document即：HTML Document对象，每个载入浏览器的HTML文档都会成为Document对象
由于Document对象是window对象的一部分，所以可通过window.document属性对其进行访问。

### HTML5为什么只需要写<!DOCTYPE HTML>？
HTML5不基于SGML，因此不需要对DTD进行引用，但是需要doctype来规范浏览器的行为（让浏览器按照它们应该的方式来进行）
而HTML4.01基于SGML，所以需要对DTD进行引用，才能告知浏览器文档所使用的文档类型。

### Doctype？严格模式与混杂模式如何触发这两种模式，区分它们有何意义？
用于声明文档使用哪种规范（html/Xhtml）一般为严格过渡基于框架的html文档。
加入XML声明可触发，解析方式更改为IE5.5拥有IE5.5的Bug。

### Doctype作用？标准模式与兼容模式各种什么区别？
！Doctype声明位于HTML文档的第一行，处于html标签之前。告知浏览器的解析器用什么文档标准解析这个文档。doctype不存在或格式不正确会导致文档以兼容模式呈现。
标准模式的排版和JS运作模式都是该浏览器支持的最高标准运行。在兼容模式中，页面以宽松的向后兼容的方式来显示，模拟老式浏览器的行为以防止站点无法工作。

### HTML5为什么只需要写！DOCTYPE HTML？
HTMl5不基于SGML,因此不需要对DTD进行引用，但是需要doctype来规范浏览器的行为(让浏览器按照它们应该的方式来运行)；而HTMl4.01基于SGMA，所以需要对DTD进行引用，才能告知浏览器文档所使用的文档类型。

### 如何实现浏览器内多个标签页之间的通信？

调用localstorage,cookies等本地存储方式
WebSocket、SharedWorker
localstroge另一个浏览器上下文被添加、删除或修改时，它都会触发一个事件，我们通过监听事件，控制它的值来进行页面信息通信。
注意quirks:Safari在无痕迹模式下设置localstorge值抛出QuotExceededError的异常。

### 请描述一下cookies、sessionStorage和localStorage区别？

cookie是网站为了标示用户身份而储存在用户本地终端（Client Side）上的数据（通常经过加密）。
cookie数据始终在同源的http请求中携带（即使不需要），即会在浏览器和服务器间来回传递。
sessionStorage和localStorage不会自动把数据发给服务器，仅在本地保存
存储大小：
cookie数据大小不能超过4K。
sessionStorage和localStorage虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大。
有期时间：
localStorage：存储持久数据，浏览器关闭后数据不丢失除非主动删除数据；
sessionStorage：数据在当前浏览器窗口关闭后自动删除
cookie：设置的cookie过期时间之前一直有效，即使窗口或浏览器关闭

### 请描述一下cookies,sessionStorage（会话存储）和localStrorage（本地存储）的区别？
cookie在浏览器和服务器间来回传递。sessionStorage和localStorage不会；
sessionStorage和localStorage的存储空间更大；
sessionStorage和localStorage有更多丰富易用的接口；
sessionStorage和localStorage各自独立的存储空间；

### 常见的浏览器内核有哪些？
Trident内核：IE,MaxThon,TT,The Word,360,搜狗浏览器等。[又称为MSHTML]
Gecko内核：Netscape6及以上版本，FF,MozillaSuite/SeaMonkey等；
Presto内核：Opera7及以上。[Opera内核原为：Presto，现为：Blink]
Webkit内核：Safari,Chrome等。[Chrome的:Blink(Webkit的分支)]

### 简述一下你对HTML语义化的理解？
1、用正确的标签做正确的事情。
2、html语义化让页面的内容结构化，结构更清晰，便于对浏览器，搜索引擎解析；
3、即使在没有样式CSS情况下也以一种文档格式显示，并且是容易阅读的；
4、搜索引擎的爬虫也依赖于HTML标记确定上下文和各个关键字的权重，利用SEO;
5、使阅读源代码的人对网站更容易将网站分块，便于阅读维护理解。

### 什么是语义化的HTML？
直观的认识标签对于搜索引擎的抓取有好处，用正确的标签做正确的事情！
HTML语义化就是让页面的内容结构化，便于对浏览器，搜索引擎解析；
在没有样式css情况下也以一种文档格式显示，并且是容易阅读。搜索引擎的爬虫依赖于标记来确定上下文和各个关键字的权重，利于SEO。
在阅读源代码的人对网站更容易将网站分块，便于阅读维护理解。

### XHTML和HTML有什么区别
HTML是一种基本的WEB网页设计语言，XHTML是一个基于XMl的置标语言
最主要的不同
XHTML元素必须被正确地嵌套。
XHTML元素必须被关闭
标签名必须用小写字母
XHTMl文档必须拥有根元素

### 一个高度自适应的div，里面有两个div，一个高度100px，希望另一个填满剩下的高度
方案1： .sub { height: calc(100%-100px); }
方案2： .container { position:relative; } .sub { position: absolute; top: 100px; bottom: 0; }
方案3： .container { display:flex; flex-direction:column; } .sub { flex:1; }

## Node 面试题
### npm install的实现原理
输入 npm install 命令并敲下回车后，会经历如下几个阶段（以 npm 5.5.1 为例）：

#### 执行工程自身 preinstall
当前 npm 工程如果定义了 preinstall 钩子此时会被执行。

#### 确定首层依赖模块
首先需要做的是确定工程中的首层依赖，也就是 dependencies 和 devDependencies 属性中直接指定的模块（假设此时没有添加 npm install 参数）。
工程本身是整棵依赖树的根节点，每个首层依赖模块都是根节点下面的一棵子树，npm 会开启多进程从每个首层依赖模块开始逐步寻找更深层级的节点。

#### 获取模块
获取模块是一个递归的过程，分为以下几步：
一、获取模块信息。在下载一个模块之前，首先要确定其版本，这是因为 package.json 中往往是 semantic version（semver，语义化版本）。此时如果版本描述文件（npm-shrinkwrap.json 或 package-lock.json）中有该模块信息直接拿即可，如果没有则从仓库获取。如 packaeg.json 中某个包的版本是 ^1.1.0，npm 就会去仓库中获取符合 1.x.x 形式的最新版本。
二、获取模块实体。上一步会获取到模块的压缩包地址（resolved 字段），npm 会用此地址检查本地缓存，缓存中有就直接拿，如果没有则从仓库下载。
三、查找该模块依赖，如果有依赖则回到第1步，如果没有则停止。

#### 模块扁平化（dedupe）
上一步获取到的是一棵完整的依赖树，其中可能包含大量重复模块。比如 A 模块依赖于 loadsh，B 模块同样依赖于 lodash。在 npm3 以前会严格按照依赖树的结构进行安装，因此会造成模块冗余。
从 npm3 开始默认加入了一个 dedupe 的过程。它会遍历所有节点，逐个将模块放在根节点下面，也就是 node-modules 的第一层。当发现有重复模块时，则将其丢弃。
这里需要对重复模块进行一个定义，它指的是模块名相同且 semver 兼容。每个 semver 都对应一段版本允许范围，如果两个模块的版本允许范围存在交集，那么就可以得到一个兼容版本，而不必版本号完全一致，这可以使更多冗余模块在 dedupe 过程中被去掉。
比如 node-modules 下 foo 模块依赖 lodash@^1.0.0，bar 模块依赖 lodash@^1.1.0，则 ^1.1.0 为兼容版本。
而当 foo 依赖 lodash@^2.0.0，bar 依赖 lodash@^1.1.0，则依据 semver 的规则，二者不存在兼容版本。会将一个版本放在 node_modules 中，另一个仍保留在依赖树里。
举个例子，假设一个依赖树原本是这样：
``` js
node_modules
-- foo
---- lodash@version1
-- bar
---- lodash@version2
```
假设 version1 和 version2 是兼容版本，则经过 dedupe 会成为下面的形式：
```js
node_modules
-- foo
-- bar
-- lodash（保留的版本为兼容版本）
```
假设 version1 和 version2 为非兼容版本，则后面的版本保留在依赖树中：
``` js
node_modules
-- foo
-- lodash@version1
-- bar
---- lodash@version2
```
#### 安装模块
这一步将会更新工程中的 node_modules，并执行模块中的生命周期函数（按照 preinstall、install、postinstall 的顺序）。
#### 执行工程自身生命周期
当前 npm 工程如果定义了钩子此时会被执行（按照 install、postinstall、prepublish、prepare 的顺序）。最后一步是生成或更新版本描述文件，npm install 过程完成。

### node 定时器
[Node 定时器详解](http://www.ruanyifeng.com/blog/2018/02/node-event-loop.html)


### 描述一下所做过的项目中遇到最困难的技术问题是什么，最后怎么解决？
首先我先说一下项目中在写法上我优化的点：
1、**require.context：**创建自己的（模块）上下文，这个方法有 3 个参数：要搜索的文件夹目录，是否还应该搜索它的子目录，以及一个匹配文件的正则表达式。只要重复性的引入文件，我都回使用该方法，比较典型的就是vuex 自动注册store，根据webpack的require.context及store的registerModule方法来自动注册store的modules，多人协作开发不需要担心代码冲突，不需要每个store.js都要import引入。

### Http
参考：
[程序员必备基础知识：通信协议——Http、TCP、UDP](https://zhuanlan.zhihu.com/p/31059450)

### koa 中间件机制
参考：
[理解 Koa 的中间件机制](https://github.com/frontend9/fe9-library/issues/104)
