---
title: 去抖动（Debounce）
description: 用 JavaScript 实现 Debounce 功能
published: true
date: 2021-01-30T04:45:56.856Z
tags: javascript, tools, debounce
editor: markdown
dateCreated: 2021-01-05T08:10:04.873Z
---

# 什么是去抖动

去抖动（Debounce）是为了性能和交互体验考量，将高频的多个连续调用序列分组提取为单次调用的功能[^1]。

> The Debounce technique allow us to “group” multiple sequential calls in a single one.

可以在 CodePen 提供的[交互用例](https://codepen.io/dcorb/embed/KVxGqN?height=391&theme-id=1&slug-hash=KVxGqN&default-tab=result&user=dcorb&name=cp_embed_2)中体验，用关键帧记录动作调用的轨迹会呈现下图中的效果。

![debounce.png](/technology/tools/libs/debounce/debounce.png)

该功能在不同端均有实用场景，比如大家经常使用到的搜索引擎中界面输入框的实时下拉列表联想词展示。

![google-search.png](/technology/tools/libs/debounce/google-search.png =65%x)

# 实现核心功能

从功能分析，我们可以用一个计时器延迟触发调用，当有更多后续调用触发的时候，重置这个计时器。这样从结果上看，很多冗余的调用被忽略掉了，我们不断重置的计时器只会允许时间间隔完成计时阈值的调用触发。

那么这个**重置计时器延迟调用**行为可以用 JavaScript 实现成下面的样子。

```js
function debounce(fn, wait = 50) {
  let timer = 0
  
  return function(...args) {
    if (timer) {
    	clearTimeout(timer)
    }
    timer = setTimeout(() => {
    	fn.apply(this, args)
    }, wait)
  }
}
```

我们默认延迟时间为 50ms，上面的实现返回一个未被执行的匿名函数，因此 `timer` 变量被闭包缓存起来不会被销毁。每次该匿名函数执行的时候，我们都可以判断当前是否存在一次未等到真正触发的调用，如果有，那就重设延迟器到 50ms，调用行为越频繁的触发，就会被计时器不断的延迟触发。

具体使用时，下面的 `input` 事件绑定的回调函数是 `debounce(userAction)`，也就是 `debounce` 返回而未被执行的匿名函数，当 `input` 事件被浏览器感知并触发的时候，匿名函数被调用，延迟计时器重置成功。

```js
function userAction() {
	console.log('被扼制的疯狂输入')
}
    
const input = document.getElementById('#input')
input.addEventListener('input', debounce(userAction))
```

# 完善细节

上一节实现的逻辑，其实只不过是不断重设延迟时间，让调用行为不断等待，直到用户停止操作的间隔超过 50ms 的时候触发。

## 现存问题

我们试想下面几个问题。

- 第一次调用，也需要等待 50ms 才能触发，用户如果只键入了一个字符，想要马上看到结果想必并不是过分的要求 :)
- 如果这个 `wait` 并不是 50ms，而是 500ms，用户每次频繁的交互，都会让下一次反馈等待 0.5s 才能被感知到，有没有办法可以照顾到用户的体验，在频繁的交互流程中，让下次调用的时间可以根据上次触发的时间计算用户真正需要等待的间隔，而不是不断的延后固定的 0.5s 呢？
- 参考第一节的调用序列帧图，我们发现每一段序列都有**开始**和**结束**这两个明确的边界点（试想下你打字搜索内容的时候，大多数场景都是输入几个字，停下来然后再输入几个字），那么这个 `debounce` 功能是否可以明确指定触发的时间点在序列开始或结束的时候呢？（目前我们实现的延迟器都是在结束的时候才触发调用）
- 函数对于输入没有任何校验。

只是随便想了想，我们就发现了很多问题，目前的 `debounce` 是不折不扣的玩具代码，实际运行起来，怕是自己都看不下去。

不过，玩具代码正是抽象思维输出的关键逻辑，我们只需要更进一步，把事情做完整即可。

## 允许调用条件

我们参考开源项目 [Lodash](https://github.com/lodash/lodash)[^2] 中实现的 debounce 功能[^3]，从入口处[^4]开始完善我们的逻辑。

首先为解决上一小节发现的第一次调用不能马上执行问题，可以在入口处做是否可以调用的判断，为了能够计算从上一次调用到现在已经流逝的时间，判断前做时间戳打点作为参数传入。

```js
  function debounced(...args) {
    const time = Date.now()
    const isInvoking = shouldInvoke(time)
    
    ...
  }
```

这个 `shouldInvoke` 功能返回是否可以调用的标识，我们来完成是否允许第一次调用的判断。

```js
  function shouldInvoke(time) {
    return (lastCallTime === undefined)
  }
```

当然第一次调用前并没有任何 `lastCallTime` 产生，所以是 `undefined`。

接着考虑正常情况，当距离产生上一次调用发生的时间段超出了我们设置的默认间隔，也应该让下一次调用得以执行，因此继续完善该条件。

```js
  function shouldInvoke(time) {
    const timeSinceLastCall = time - lastCallTime

    return (lastCallTime === undefined || (timeSinceLastCall >= wait))
  }
```

到这里为止，允许调用的条件已经可以应付大部分场景了，不过 lodash 多考虑了系统获取时间戳倒退的异常行为。

```js
  function shouldInvoke(time) {
    const timeSinceLastCall = time - lastCallTime

    return (lastCallTime === undefined || (timeSinceLastCall >= wait) || (timeSinceLastCall < 0))
  }
```

这样从入口，我们限制了是否可以允许当前调用的行为。

## 处理调用序列边界

我们接着思考调用序列中如何选择起点或终点的边界问题，目前依赖的 `setTimeout` 只能在延迟器完成计时后触发调用（即在调用序列终点处调用），想要实现在计时开始的时候就立即调用，我们需要增加一个标志位进行判断，如果用户想在调用序列一开始的时候就触发动作，就显式的传入这个参数。

Lodash 同样提供了用于指定调用产生的时间点参数 `options.leading`[^5]。

> `_.debounce(func, [wait=0], [options={}])`
  `[options={}] (Object)`: The options object.
  `[options.leading=false] (boolean)`: Specify invoking on the leading edge of the timeout.
  
逻辑实现上，我们稍微更改下代码的结构，让上一节的逻辑更加贴近我们本篇内容开始实现的 “玩具代码” 结构。

下面的代码，让我们先忽略掉新增的一些变量，提前搞懂这些变量的含义并没有任何帮助。

```js
function debounce(func, wait, options) {
  let timerId,
      lastCallTime
      result
    
  let lastInvokeTime = 0
  let leading = false
  
  function debounced(...args) {
    const time = Date.now()
    const isInvoking = shouldInvoke(time)

    lastCallTime = time

    if (isInvoking) {
      if (timerId === undefined) {
        return leadingEdge(lastCallTime)
      }
    }
    
    ...
  }

  return debounced
}
```

当通过了是否调用的限制 `shouldInvoke` 后，我们需要通过判断是否存在一个已经在等待的调用计时，这跟我们最早实现的 `if (timer)` 保持一致，接着回到我们的问题中来，实现一个 `leadingEdge` 功能，用以处理**调用序列开始或结束**的时候立即执行的情况。

```js
  function invokeFunc(time) {
    lastInvokeTime = time
    result = func.apply(thisArg, args)
    return result
  }
  
  function leadingEdge(time) {
    lastInvokeTime = time
    return leading ? invokeFunc(time) : setTimeout(timerExpired, wait)
  }
```

我们通过对标志位 `leading` 的判断，处理在**调用序列起点产生调用**的场景，直接触发调用并记录这次触发的时间点。如果是在**调用序列终点调用**，我们就照常处理，开启一个延迟计时器，延迟触发调用。

## 修正调用间需要等待的时间

到目前为止，我们的 `debounce` 函数功能不断完善，但是涉及到用户体验的**每次交互都需要等待固定时长**的问题仍未得到解决。

试想下，每次用户触发最后一次操作，都需要等待固定的 `wait` 时间，一旦这个值设置的能够被用户感知到，就会马上产生 “我做任何动作都会导致当前页面变卡” 的错觉。

那我们就来实现上一节逻辑中出现但没有解释的 `timerExpired` 函数，用以解决计算真实等待时间的问题。

```js
  function remainingWait(time) {
    const timeSinceLastCall = time - lastCallTime
    const timeWaiting = wait - timeSinceLastCall
    return timeWaiting
  }
  
  function timerExpired() {
    const time = Date.now()
    timerId = setTimeout(timerExpired, remainingWait(time))
  }
```

在 `timerExpired` 中我们开启一个延迟计时器处理非调用序列起点的调用场景，这个调用的时间点可以是**调用序列中间的某个时刻**，也可以是**延迟计时结束后的终点处**。

我们先考虑中间某个时刻调用场景，要计算出准确的延迟时间，就需要先知道距离上一次调用已经过去了多久，然后用设置的固定延迟时间 `wait - 已经流逝掉的时间`，自然就能得到**还需要继续等待的时间**。

# 额外功能

来到这里，我们的 `debounce` 已经相对完善，不仅可以正确的处理调用序列的触发时间点，而且能够处理第一次调用的立即执行，最后还可以从体验上动态的计算用户每次触发交互还需要继续等待的延迟时间，看起来一切都已经很完备了，不过 Lodash 依然添加了一些外部功能以便于更加精确的控制去抖动场景。

```js
function debounce(func, wait, options) {
  let timerId,
      lastCallTime
      result
    
  let lastInvokeTime = 0
  let leading = false
  
  ...
  
  function trailingEdge(time) {
    timerId = undefined

    if (trailing) {
      return invokeFunc(time)
    }
    return result
  }
  
  function cancel() {
    if (timerId !== undefined) {
      cancelTimer(timerId)
    }
    lastInvokeTime = 0
    lastCallTime = timerId = undefined
  }
  
  function flush() {
    return timerId === undefined ? result : trailingEdge(Date.now())
  }
  
  function debounced(...args) {
    const time = Date.now()
    const isInvoking = shouldInvoke(time)

    lastCallTime = time

    if (isInvoking) {
      if (timerId === undefined) {
        return leadingEdge(lastCallTime)
      }
    }

		return result
  }
  
  debounced.cancel = cancel
  debounced.flush = flush
  return debounced
}
```

上面的主逻辑实现了能够马上取消 debounce 行为的外部功能 `cancel` 和重置 debounce 行为的 `flush` 功能，前者可以立即取消当前的所有延迟计时器，对用户的任何操作都不做限制，后者则将当前时间戳覆盖上一次调用的时间点，意图使延迟计时尽快结束，从而尽快触发调用。

至此，一个相对完整的去抖动功能得以实现，本文旨在从思考中发现并解决关键问题，并没有将所有源码细节一一详尽解释，完整的实现请带着自己的理解参考官方 [Github Lodash Debounce 源代码](https://github.com/lodash/lodash/blob/master/debounce.js)。

[^1]: [Debouncing and Throttling Explained Through Examples | CSS-Tricks  ](https://css-tricks.com/debouncing-throttling-explained-examples/)
[^2]: [lodash/lodash: A modern JavaScript utility library delivering modularity, performance, & extras.](https://github.com/lodash/lodash)
[^3]: [lodash/debounce.js at master · lodash/lodash](https://github.com/lodash/lodash/blob/master/debounce.js)
[^4]: [lodash/debounce.js at master · lodash/lodash#L184](https://github.com/lodash/lodash/blob/master/debounce.js#L184)
[^5]: [Lodash Documentation](https://lodash.com/docs/4.17.15#debounce)