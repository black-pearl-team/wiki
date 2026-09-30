---
title: Debounce
description: Implementing debounce in JavaScript
published: true
date: 2026-09-30T13:35:19.000Z
tags: javascript, tools, debounce
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/technology/tools/libs/debounce.md)

# What is debounce

Debounce is a technique that, for the sake of performance and interaction experience, groups a sequence of many high-frequency consecutive calls and pulls it out as a single call[^1].

> The Debounce technique allow us to “group” multiple sequential calls in a single one.

You can try it in the [interactive demo](https://codepen.io/dcorb/embed/KVxGqN?height=391&theme-id=1&slug-hash=KVxGqN&default-tab=result&user=dcorb&name=cp_embed_2) on CodePen; recording the trail of the calls as keyframes gives the effect in the picture below.

![debounce.png](/technology/tools/libs/debounce/debounce.png)

It has practical uses on every platform, for example the live dropdown of suggested words under the input box of the search engines we all use.

<img src="/technology/tools/libs/debounce/google-search.png" alt="google-search.png" width="65%">

# Implementing the core

Looking at the feature, we can delay the call with a timer, and reset that timer whenever more calls come in. As a result, lots of redundant calls get ignored, and the timer we keep resetting only lets a call fire once the interval has run past the timing threshold.

So this **reset the timer, delay the call** behavior can be implemented in JavaScript like this.

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

Our default delay is 50ms. The implementation above returns an anonymous function that hasn't run yet, so the `timer` variable is cached by the closure and isn't destroyed. Every time the anonymous function runs, we can check whether there's a call still waiting to really fire; if there is, we reset the delay to 50ms. The more often the call fires, the more the timer keeps delaying it.

In actual use, the callback bound to the `input` event below is `debounce(userAction)`, that is, the anonymous function `debounce` returns without running it. When the browser picks up and fires the `input` event, the anonymous function is called and the delay timer is reset.

```js
function userAction() {
	console.log('frantic typing, reined in')
}
    
const input = document.getElementById('#input')
input.addEventListener('input', debounce(userAction))
```

# Filling in the details

The logic from the previous section really just keeps resetting the delay, making the call wait and wait, until the user pauses for longer than 50ms and it fires.

## Problems so far

Let's think about the following problems.

- The first call also has to wait 50ms before it fires. If the user types just one character, wanting to see the result right away is surely not too much to ask :)
- If this `wait` isn't 50ms but 500ms, every burst of frequent interaction makes the next feedback wait 0.5s before the user can notice it. Is there a way to look after the user's experience, so that during frequent interaction the next call works out, from when the last one fired, how long the user really needs to wait, instead of pushing it back by a fixed 0.5s every time?
- Looking back at the frame diagram of the call sequence in the first section, we notice that each sequence has two clear boundary points, a **start** and an **end** (think of typing a search: most of the time you type a few characters, stop, then type a few more). So could this `debounce` let us say explicitly whether it fires at the start or at the end of a sequence? (The delay we've built so far only fires at the end.)
- The function doesn't validate its input at all.

Just from a quick think, we've found plenty of problems. The current `debounce` is toy code through and through; if we actually ran it, I doubt we could even stand to watch it ourselves.

But toy code is exactly where the key logic of abstract thinking comes out; we just need to go one step further and finish the job.

## Conditions for allowing a call

We refer to the open-source project [Lodash](https://github.com/lodash/lodash)[^2] and the debounce feature it implements[^3], and flesh out our logic starting from its entry point[^4].

First, to solve the problem from the last subsection, that the first call can't run right away, we can check at the entry point whether a call is allowed. To work out how much time has passed since the last call, we take a timestamp before the check and pass it in as an argument.

```js
  function debounced(...args) {
    const time = Date.now()
    const isInvoking = shouldInvoke(time)
    
    ...
  }
```

This `shouldInvoke` function returns a flag saying whether the call may go ahead. Let's write the check that allows the first call.

```js
  function shouldInvoke(time) {
    return (lastCallTime === undefined)
  }
```

Of course, before the first call no `lastCallTime` exists yet, so it's `undefined`.

Next, the normal case: when the time since the last call exceeds the default interval we set, the next call should be allowed to run as well, so we extend the condition.

```js
  function shouldInvoke(time) {
    const timeSinceLastCall = time - lastCallTime

    return (lastCallTime === undefined || (timeSinceLastCall >= wait))
  }
```

At this point the conditions for allowing a call can handle most scenarios, but lodash also covers the abnormal case of the system timestamp going backwards.

```js
  function shouldInvoke(time) {
    const timeSinceLastCall = time - lastCallTime

    return (lastCallTime === undefined || (timeSinceLastCall >= wait) || (timeSinceLastCall < 0))
  }
```

So at the entry point, we now control whether the current call is allowed.

## Handling the edges of a call sequence

Next we think about how to pick the start or the end edge of a call sequence. The `setTimeout` we rely on can only fire the call after the delay has finished (that is, at the end of the sequence). To call right away when the timing starts, we need to add a flag to check: if the user wants the action to fire as soon as the sequence starts, they pass this argument in explicitly.

Lodash also provides an argument for specifying when the call happens, `options.leading`[^5].

> `_.debounce(func, [wait=0], [options={}])`
  `[options={}] (Object)`: The options object.
  `[options.leading=false] (boolean)`: Specify invoking on the leading edge of the timeout.
  
For the implementation, we change the structure of the code a little, so that the logic from the last section sits closer to the "toy code" structure we started this article with.

In the code below, let's ignore some of the new variables for now; understanding what they mean up front wouldn't help at all.

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

Once a call gets past the `shouldInvoke` limit, we need to check whether a timed call is already waiting, which matches the `if (timer)` from our very first implementation. Then, back to our problem, we implement a `leadingEdge` function to handle running right away at the **start or end of a call sequence**.

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

By checking the `leading` flag, we handle the case where the **call happens at the start of the sequence**: fire the call directly and record when it fired. If the **call happens at the end of the sequence**, we handle it as usual and start a delay timer to fire the call later.

## Correcting the wait between calls

So far our `debounce` keeps getting better, but the user-experience problem of **every interaction having to wait a fixed time** still isn't solved.

Picture it: every time the user makes their last move, they have to wait the fixed `wait` time, and once that value is set long enough for the user to notice, it straight away creates the illusion that "whatever I do makes this page lag".

So let's implement the `timerExpired` function, which showed up in the last section's logic without being explained, to solve the problem of working out the real wait.

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

In `timerExpired` we start a delay timer to handle calls that aren't at the start of a sequence; such a call can happen **at some moment in the middle of the sequence**, or **at the end, after the delay has finished**.

Let's first take the case of a call at some moment in the middle. To work out the exact delay, we first need to know how long it's been since the last call; then the fixed delay we set, `wait - the time that has already passed`, naturally gives us **how much longer we still have to wait**.

# Extra features

By now our `debounce` is fairly complete. Not only does it handle the firing points of a call sequence correctly, it can also run the first call immediately, and on top of that it can dynamically work out, for the user's sake, how much longer each interaction still has to wait. Everything looks complete, but Lodash still adds some external features for more precise control over debouncing.

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

The main logic above adds an external `cancel` feature that can cancel the debounce behavior right away, and a `flush` feature that resets it. The former immediately cancels all current delay timers and puts no limits on anything the user does; the latter overwrites the time of the last call with the current timestamp, meaning to end the delay as soon as possible and so fire the call as soon as possible.

With that, a fairly complete debounce feature is done. This article aims to find and solve the key problems through thinking, and doesn't explain every detail of the source code one by one; for the full implementation, bring your own understanding along and refer to the official [Lodash debounce source on GitHub](https://github.com/lodash/lodash/blob/master/debounce.js).

[^1]: [Debouncing and Throttling Explained Through Examples | CSS-Tricks  ](https://css-tricks.com/debouncing-throttling-explained-examples/)
[^2]: [lodash/lodash: A modern JavaScript utility library delivering modularity, performance, & extras.](https://github.com/lodash/lodash)
[^3]: [lodash/debounce.js at master · lodash/lodash](https://github.com/lodash/lodash/blob/master/debounce.js)
[^4]: [lodash/debounce.js at master · lodash/lodash#L184](https://github.com/lodash/lodash/blob/master/debounce.js#L184)
[^5]: [Lodash Documentation](https://lodash.com/docs/4.17.15#debounce)