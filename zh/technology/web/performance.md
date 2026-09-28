---
title: JavaScript 内存泄漏 - 垃圾回收 - ES6 处理方法
description: JavaScript 内存回收、GC 的原理和处理方法
published: true
date: 2021-01-08T03:47:21.154Z
tags: javascript, gc
editor: markdown
dateCreated: 2020-11-30T15:05:19.322Z
---

# 内存泄漏

程序的运行需要内存。只要程序提出要求，操作系统或者运行时（runtime）就必须供给内存。

对于持续运行的服务进程（daemon），必须及时释放不再用到的内存。否则，内存占用越来越高，轻则影响系统性能，重则导致进程崩溃。

不再用到的内存，没有及时释放，就叫做 **内存泄漏（memory leak）**。


## 内存管理

优化内存占用的最佳手段就是保证在执行代码时只保存必要的数据。如果数据不再必要，那么把它设置为 null，从而释放其引用。
1. **通过 const 和 let 声明提升性能**
    因为 const 和 let 都以块(而非函数)为作用域，所以相比于使用 var，使用这两个新关键字可能会更早地让垃圾回收程序介入，尽早回收应该回收的内存。
2. **隐藏类 和 删除操作**
    运行期间，V8 会将创建的对象与隐藏类关联起来，以跟踪它们的属性特征。能够共享相同隐藏类的对象性能会更好，V8 会针对这种情况进行优化，但不一定总能够做到。
```js
function Article() {
		this.title = 'Inauguration Ceremony Features Kazoo Band';
}
let a1 = new Article();
let a2 = new Article();
```
V8 会在后台配置，让这两个类实例共享相同的隐藏类，因为这两个实例共享同一个构造函数和原型。假设之后又添加了下面这行代码:
```js
a2.author = 'Jake';
```
此时两个 Article 实例就会对应两个不同的隐藏类。根据这种操作的频率和隐藏类的大小，这有可能对性能产生明显影响。
当然，解决方案就是避免 JavaScript 的“先创建再补充”(ready-fire-aim)式的动态属性赋值，并在构造函数中一次性声明所有属性。

不过要记住，使用 delete 关键字会导致生成相同的隐藏类片段。
```js
function Article() {
		this.title = 'Inauguration Ceremony Features Kazoo Band';
    this.author = 'Jake';
}
let a1 = new Article();
let a2 = new Article();

delete a1.author;
```
在代码结束后，即使两个实例使用了同一个构造函数，它们也不再共享一个隐藏类。动态删除属性与动态添加属性导致的后果一样。最佳实践是**把不想要的属性设置为 null**。这样可以保持隐藏类不变 和继续共享，同时也能达到删除引用值供垃圾回收程序回收的效果。

## 内存泄漏细节

1. 意外声明全局变量是最常见但也最容易修复的内存泄漏问题。
2. 定时器也可能会悄悄地导致内存泄漏。（e.g.: 定时器的回调通过闭包引用了外部变量，没有及时清理定时器）
3. 使用 JavaScript 闭包很容易在不知不觉间造成内存泄漏。
4. 引用 DOM 节点做某些操作，之后 DOM 节点在页面上已被销毁。
5. iframe：如果父窗口保留了iframe中的引用变量、方法或者 iframe 将内部方法绑定到父窗口，即使删除 iframe 也不会释放 iframe 中所有的 js 堆内存和 dom 所占用的内存。


# 垃圾回收机制

大多数语言提供自动内存管理，减轻程序员的负担，这被称为"垃圾回收机制"（garbage collector）。

为了提高垃圾回收的效率，V8将堆分为 **新生代** 和 **老生代** 两个部分，其中新生代为存活时间较短的对象（需要经常进行垃圾回收），而老生代为存活时间较长的对象（垃圾回收的频率较低）。

## 新生代

新生代中的对象主要通过 Scavenge 算法进行垃圾回收。
在 Scavenge 的具体实现中，主要采用了 Cheney 算法。

> Cheney 算法是一种采用**复制**的方式实现的垃圾回收算法。
> 它将堆内存一分为二，每一部分空间成为 semispace。在这两个 semispace 空间中，只有一个处于使用中，另一个处于闲置中。
> 处于使用中的 semispace 空间成为 From 空间，处于闲置状态的空间成为 To 空间。当我们分配对象时，先是在 From 空间中进行分配。当开始进行垃圾回收时，会检查 From 空间中的存活对象，这些存活对象将被复制到 To 空间中，而非存活对象占用的空间将被释放。完成复制后， From 空间和 To 空间的角色发生对换。

Scavenge 的缺点是只能使用堆内存的一半，但 Scavenge 由于只复制存活的对象，并且对于生命周期短的场景存活对象只占少部分，所以它在时间效率上表现优异。Scavenge 是典型的 **牺牲空间换取时间** 的算法，无法大规模地应用到所有的垃圾回收中，但非常适合应用在新生代中。

> 对象是如何释放的呢？
有个叫**可达性分析算法**的概念，即通过一系列的称为 “GC ROOT” 的对象作为起始点。从这些节点开始向下搜索。
搜索走过的路径称为**引用链**。当一个对象到GC ROOT没有任何引用链时，则证明此对象是不可用的。
当然在虚拟机判断要被释放的对象里面，即使在可达性分析算法中不可达的对象，也并非是立即释放的。如果对象在进行可达性分析后发现没有与 GC ROOTS 相连接的引用链。将会对它进行一次**标记**，并进行刷选。它会放进一个队列中依次进行回收。如果这时又有对象引用到它，它就不会被回收。

- 晋升

对象从新生代中移动到老生代中的过程称为晋升。
From 空间中的存活对象在复制到 To 空间之前需要进行检查，在一定条件下，需要将存活周期长的对象移动到老生代中，也就是完成对象的晋升。

晋升条件主要有两个：

1. 对象是否经历过一次 Scavenge 回收，是的话，则移动到老生代
2. To 空间已经使用超过 25%，To 空间对象移动到老生代

设置 25% 这个限制值得原因是当这次 Scavenge 回收完成后，这个 To 空间将变成 From 空间，接下来的内存分配将在这个空间中进行，如果占比过高，会影响后续的内存分配。

## 老生代

> **标记清理（mark-and-sweep）**

当变量进入上下文，比如在函数内部声明一个变量时，这个变量会被加上**存在于上下文中的标记**。而在上下文中的变量，逻辑上讲，永远不应该释放它们的内存，因为只要上下文中的代码在运行，就有可能用到他们。当变量离开上下文时，也会被加上**离开上下文的标记**。

给变量加标记的方式有很多种（原理是判断是否**在上下文中**）。

**垃圾回收程序** 运行的时候，会标记内存中存储的所有变量（标记的方法有很多种，标记过程的实现并不重要，关键是策略）。然后，它会将所有在上下文中的变量，以及被在上下文中的变量引用的变量的标记去掉，在此之后再被加上标记的变量就是待删除的了，原因是任何在上下文中的变量都访问不到他们了。随后，垃圾回收程序做一次**内存清理**，销毁带标记的所有值并回收它们的内存。

现代垃圾回收程序会基于对JavaScript运行时环境的探测来决定何时运行。
探测机制应引擎而异，但基本上都是根据**已分配对象的大小和数量**来判断的。
根据 V8 团队 2016 年的一篇博文的说法：在一次完整的垃圾回收之后，V8 的**堆增长策略**会根据**活跃对象的数量外加一些余量**来确定何时再次垃圾回收。

> **"引用计数"（reference counting）**

另一个没那么常使用的方法，语言引擎有一张"引用表"，保存了内存里面所有的资源（通常是各种值）的引用次数。如果一个值的引用次数是 `0`，就表示这个值不再用到了，因此可以将这块内存释放。

如果一个值不再需要了，引用数却不为 `0`，垃圾回收机制无法释放这块内存，从而导致内存泄漏。


# 内存泄漏的识别方法

- 经验法则：
如果连续五次垃圾回收之后，内存占用一次比一次大，就有内存泄漏。这就要求实时查看内存占用。

- 浏览器：
Chrome Devtool -> Performance -> 勾选 Memory -> 录制、操作、Stop -> 查看显示

- 命令行：
`process.memoryUsage()` Node 提供的方法
`heapUsed`
```js
console.log(process.memoryUsage());
// { rss: 27709440,
//  heapTotal: 5685248,
//  heapUsed: 3449392,  <- 判断内存泄漏，以该字段为准。
//  external: 8772 }
```


# ES6 解决方案

最好能有一种方法，在新建引用的时候就声明，哪些引用必须手动清除，哪些引用可以忽略不计，当其他引用消失以后，垃圾回收机制就可以释放内存。这样就能大大减轻程序员的负担，你只要清除主要引用就可以了。

ES6 考虑到了这一点，推出了两种新的数据结构：WeakSet 和 WeakMap。它们对于值的引用都是不计入垃圾回收机制的，所以名字里面才会有一个"Weak"，表示这是弱引用。

## WeakMap[^1]

与 Map 的区别：
- WeakMap 只接受对象作为键名（null除外），不接受其他类型的值作为键名。
- WeakMap的键名所指向的对象，不计入垃圾回收机制。

语法上的区别：
- WeakMap 没有遍历操作（即没有 `keys()`、`values()` 和 `entries()` 方法），也没有 `size` 属性。
- WeakMap 无法清空，即不支持 `clear` 方法。因此，WeakMap 只有四个方法可用：`get()`、`set()`、`has()`、`delete()`。

> 例子：
>有时我们想在某个对象上面存放一些数据，但是这会形成对于这个对象的引用。
>```js
> const e1 = document.getElementById('foo');
> const e2 = document.getElementById('bar');
> const arr = [
>   [e1, 'foo 元素'],
>   [e2, 'bar 元素'],
> ];
> ```
> 上面代码中，e1 和 e2 是两个对象，我们通过 arr 数组对这两个对象添加一些文字说明。 这就形成了 arr 对 e1 和 e2 的引用。
> 一旦不再需要这两个对象，我们就必须手动删除这个引用，否则垃圾回收机制就不会释放e1和e2占用的内存。（手动删除引用：`arr[0] = null; arr[1] = null;`）

WeakMap 就是为了解决这个问题而诞生的，它的键名所引用的对象都是弱引用，即垃圾回收机制不将该引用考虑在内。因此，只要所引用的对象的其他引用都被清除，垃圾回收机制就会释放该对象所占用的内存。也就是说，一旦不再需要，WeakMap 里面的键名对象和所对应的键值对会自动消失，不用手动删除引用。

作用：
- 基本上，如果你要往对象上添加数据，又不想干扰垃圾回收机制，就可以使用 WeakMap。
一个典型应用场景是，在网页的 DOM 元素上添加数据，就可以使用WeakMap结构。当该 DOM 元素被清除，其所对应的WeakMap记录就会自动被移除。

```js
const wm = new WeakMap();

const element = document.getElementById('example');

wm.set(element, 'some information');
wm.get(element) // "some information"
```
WeakMap 里面对 element 的引用就是弱引用，不会被计入垃圾回收机制。也就是说，上面的 DOM 节点对象的引用计数是 1，而不是 2。这时，一旦消除对该节点的引用，它占用的内存就会被垃圾回收机制释放。Weakmap 保存的这个键值对，也会自动消失。

总之，**WeakMap 的专用场合就是，它的键所对应的对象，可能会在将来消失。WeakMap 结构有助于防止内存泄漏。**

> 注意，WeakMap 弱引用的只是键名，而不是键值。键值依然是正常引用。
```js
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};

wm.set(key, obj);
obj = null;
wm.get(key)
// Object {foo: 1}
```
上面代码中，键值obj是正常引用。所以，即使在 WeakMap 外部消除了obj的引用，WeakMap 内部的引用依然存在。

语法：
- 没有遍历操作（即没有keys()、values()和entries()方法），也没有size属性。
- 无法清空，即不支持clear方法。
- WeakMap只有四个方法可用：get()、set()、has()、delete()。

用途案例：
1. DOM 节点作为键名
```js
let myWeakmap = new WeakMap();

myWeakmap.set(
  document.getElementById('logo'),
  {timesClicked: 0})
;

document.getElementById('logo').addEventListener('click', function() {
  let logoData = myWeakmap.get(document.getElementById('logo'));
  logoData.timesClicked++;
}, false);
```
2. 部署私有属性
```js
const _counter = new WeakMap();
const _action = new WeakMap();

class Countdown {
  constructor(counter, action) {
    _counter.set(this, counter);
    _action.set(this, action);
  }
  dec() {
    let counter = _counter.get(this);
    if (counter < 1) return;
    counter--;
    _counter.set(this, counter);
    if (counter === 0) {
      _action.get(this)();
    }
  }
}

const c = new Countdown(2, () => console.log('DONE'));

c.dec()
c.dec()
// DONE
```
上面代码中，Countdown类的两个内部属性_counter和_action，是实例的弱引用，所以如果删除实例，它们也就随之消失，不会造成内存泄漏。

## WeakSet[^2]

与 Set 的区别：
- WeakSet 的成员只能是对象，而不能是其他类型的值。
- WeakSet 中的对象都是弱引用，即垃圾回收机制不考虑 WeakSet 对该对象的引用。
    （也就是说，如果其他对象都不再引用该对象，那么垃圾回收机制会自动回收该对象所占用的内存，不考虑该对象还存在于 WeakSet 之中。）
    
作用：
- WeakSet 适合临时存放一组对象，以及存放跟对象绑定的信息。只要这些对象在外部消失，它在 WeakSet 里面的引用就会自动消失。
- WeakSet 的一个用处，是储存 DOM 节点，而不用担心这些节点从文档移除时，会引发内存泄漏。

使用：
- WeakSet 是一个构造函数，可以使用 `new` 命令，创建 WeakSet 数据结构。
- 三个方法：`add`, `delete`, `has`。
- 没有 `size` 属性，没有办法遍历它的成员；试图获取 `size` 和 `forEach` 属性，结果都不能成功。

[^1]: [WeakMap - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)
[^2]: [WeakSet - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet)
