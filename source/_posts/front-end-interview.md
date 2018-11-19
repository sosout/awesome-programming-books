---
title: 前端面试题
date: 2018-11-19 23:43:52
tags:
  - interview
categories:
  - front-end
---

## JavaScript 面试题

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
