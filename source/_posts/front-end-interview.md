---
title: 前端面试题
date: 2018-11-19 23:43:52
tags:
  - interview
categories:
  - front-end
---

## JavaScript 面试题

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