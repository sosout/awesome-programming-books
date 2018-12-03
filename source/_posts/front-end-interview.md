---
title: 前端面试题
date: 2018-11-19 23:43:52
tags:
  - interview
categories:
  - front-end
---

## JavaScript 面试题

### promise
更多参考：[async/await 在chrome 环境和 node 环境的 执行结果不一致，求解？](https://www.zhihu.com/question/268007969)
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