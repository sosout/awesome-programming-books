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