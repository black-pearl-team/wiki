---
title: JavaScript Memory Leaks - Garbage Collection - Handling in ES6
description: How JavaScript memory reclamation and GC work, and how to deal with them
published: true
date: 2026-09-30T13:35:19.000Z
tags: javascript, gc
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/technology/web/performance.md)

# Memory leaks

A running program needs memory. Whenever the program asks for it, the operating system or the runtime has to supply memory.

For a service process that keeps running (a daemon), memory that's no longer used has to be released in time. Otherwise memory usage keeps climbing, which at best hurts system performance and at worst crashes the process.

Memory that's no longer used and isn't released in time is called a **memory leak**.


## Memory management

The best way to optimize memory usage is to make sure only the necessary data is kept while code runs. Once data is no longer needed, set it to null to release its reference.
1. **Boost performance with const and let declarations**
    Because const and let are both scoped to blocks (not functions), using these two new keywords instead of var may let the garbage collector step in earlier and reclaim memory that should be reclaimed as early as possible.
2. **Hidden classes and the delete operation**
    At runtime, V8 associates the objects it creates with hidden classes to keep track of their property characteristics. Objects that can share the same hidden class perform better, and V8 optimizes for that case, but it can't always manage it.
```js
function Article() {
		this.title = 'Inauguration Ceremony Features Kazoo Band';
}
let a1 = new Article();
let a2 = new Article();
```
V8 configures things behind the scenes so that these two class instances share the same hidden class, because the two instances share the same constructor and prototype. Suppose this line of code is added afterwards:
```js
a2.author = 'Jake';
```
Now the two Article instances map to two different hidden classes. Depending on how often this happens and how big the hidden classes are, it can have a noticeable effect on performance.
The solution, of course, is to avoid JavaScript's "ready-fire-aim" style of dynamic property assignment, and to declare all properties at once in the constructor.

Keep in mind, though, that using the delete keyword produces the same hidden class fragments.
```js
function Article() {
		this.title = 'Inauguration Ceremony Features Kazoo Band';
    this.author = 'Jake';
}
let a1 = new Article();
let a2 = new Article();

delete a1.author;
```
After the code runs, the two instances no longer share a hidden class, even though they use the same constructor. Deleting a property dynamically has the same consequence as adding one dynamically. The best practice is to **set the unwanted property to null**. That keeps the hidden class unchanged and still shared, while also dropping the reference value so the garbage collector can reclaim it.

## Memory leak details

1. Accidentally declaring global variables is the most common memory leak, but also the easiest to fix.
2. Timers can also quietly cause memory leaks. (e.g.: a timer's callback references an outside variable through a closure, and the timer isn't cleared in time)
3. With JavaScript closures it's easy to cause a memory leak without noticing.
4. Referencing DOM nodes for some operation, after which those DOM nodes have already been destroyed on the page.
5. iframe: if the parent window keeps references to variables or methods inside the iframe, or the iframe binds its internal methods to the parent window, then even deleting the iframe won't release all the js heap memory in the iframe or the memory its dom takes up.


# Garbage collection

Most languages provide automatic memory management to take this burden off programmers; this is called the "garbage collector".

To make garbage collection more efficient, V8 splits the heap into two parts, the **young generation** and the **old generation**. The young generation holds short-lived objects (collected often), and the old generation holds long-lived objects (collected less often).

## The young generation

Objects in the young generation are mainly collected with the Scavenge algorithm.
In Scavenge's actual implementation, the Cheney algorithm is what's mainly used.

> The Cheney algorithm is a garbage collection algorithm implemented by **copying**.
> It splits heap memory into two halves, and each half is called a semispace. Of these two semispaces, only one is in use while the other sits idle.
> The semispace in use is called the From space, and the idle one is called the To space. When we allocate objects, they're allocated in the From space first. When garbage collection starts, the live objects in the From space are checked, these live objects are copied into the To space, and the space taken by dead objects is released. Once copying is done, the From space and the To space swap roles.

Scavenge's downside is that it can only use half of the heap memory. But because Scavenge only copies live objects, and in short-lived scenarios live objects are only a small share, it performs very well in terms of time. Scavenge is a classic **trade space for time** algorithm; it can't be applied to all garbage collection at scale, but it's a great fit for the young generation.

> So how are objects released?
There's a concept called the **reachability analysis algorithm**: a series of objects called "GC ROOT" serve as starting points, and the search goes downward from these nodes.
The path the search walks is called a **reference chain**. When an object has no reference chain to a GC ROOT, the object is proven unusable.
Of course, among the objects the virtual machine decides to release, even the ones that are unreachable in reachability analysis aren't released right away. If, after reachability analysis, an object turns out to have no reference chain connected to the GC ROOTS, it gets **marked** once and filtered. It's put into a queue and collected in turn. If some object references it again at this point, it won't be collected.

- Promotion

Moving an object from the young generation to the old generation is called promotion.
Live objects in the From space are checked before being copied into the To space; under certain conditions, long-lived objects have to be moved into the old generation, which completes the object's promotion.

There are two main conditions for promotion:

1. Whether the object has already been through one Scavenge collection; if so, it moves to the old generation
2. The To space is already more than 25% used; the To space objects move to the old generation

The reason for the 25% limit is that once this Scavenge collection is done, this To space becomes the From space, and the next memory allocations happen in it; if it's too full, later allocations suffer.

## The old generation

> **Mark-and-sweep**

When a variable enters a context, for example when a variable is declared inside a function, it gets an **in-context mark**. Variables in a context should logically never have their memory released, because as long as code in the context is running, they might be used. When a variable leaves the context, it also gets a **left-context mark**.

There are many ways to mark variables (the principle is deciding whether they're **in context**).

When the **garbage collector** runs, it marks all the variables stored in memory (there are many ways to mark, and how the marking is implemented doesn't matter; the strategy is the key). Then it removes the marks from all variables in context, and from variables referenced by variables in context. Variables that get marked after this point are the ones to delete, because no variable in context can reach them anymore. The garbage collector then does a **memory cleanup**, destroying all marked values and reclaiming their memory.

Modern garbage collectors decide when to run by probing the JavaScript runtime environment.
The probing differs from engine to engine, but it basically goes by **the size and number of allocated objects**.
According to a 2016 blog post by the V8 team: after a full garbage collection, V8's **heap growth strategy** decides when to collect again based on **the number of live objects plus some margin**.

> **Reference counting**

Another, less commonly used method: the language engine keeps a "reference table" that stores the reference count of every resource in memory (usually various values). If a value's reference count is `0`, the value is no longer used, so that memory can be released.

If a value is no longer needed but its reference count isn't `0`, the garbage collector can't release that memory, which leads to a memory leak.


# How to spot memory leaks

- Rule of thumb:
If memory usage gets bigger after each of five consecutive garbage collections, there's a memory leak. This means watching memory usage in real time.

- Browser:
Chrome Devtool -> Performance -> check Memory -> record, interact, Stop -> look at the display

- Command line:
`process.memoryUsage()`, a method Node provides
`heapUsed`
```js
console.log(process.memoryUsage());
// { rss: 27709440,
//  heapTotal: 5685248,
//  heapUsed: 3449392,  <- judge memory leaks by this field.
//  external: 8772 }
```


# ES6 solutions

It would be best to have a way to declare, when a reference is created, which references must be cleared by hand and which can be ignored, so that when the other references disappear, the garbage collector can release the memory. That would take a lot off the programmer's shoulders: you'd only need to clear the main references.

ES6 took this into account and introduced two new data structures: WeakSet and WeakMap. Their references to values don't count for the garbage collector, which is why their names contain "Weak", meaning weak references.

## WeakMap[^1]

How it differs from Map:
- WeakMap only accepts objects as keys (null excluded), not values of other types.
- The objects that WeakMap keys point to don't count for the garbage collector.

Syntax differences:
- WeakMap has no iteration operations (no `keys()`, `values()` or `entries()` methods), and no `size` property.
- WeakMap can't be emptied, i.e. it doesn't support the `clear` method. So WeakMap has only four methods available: `get()`, `set()`, `has()`, `delete()`.

> Example:
>Sometimes we want to store some data on an object, but that creates a reference to the object.
>```js
> const e1 = document.getElementById('foo');
> const e2 = document.getElementById('bar');
> const arr = [
>   [e1, 'foo element'],
>   [e2, 'bar element'],
> ];
> ```
> In the code above, e1 and e2 are two objects, and we attach some text notes to them through the arr array. That creates references from arr to e1 and e2.
> Once these two objects are no longer needed, we have to delete these references by hand, or the garbage collector won't release the memory e1 and e2 take up. (Deleting the references by hand: `arr[0] = null; arr[1] = null;`)

WeakMap was born to solve exactly this problem: the objects its keys reference are all weak references, i.e. the garbage collector doesn't take those references into account. So as soon as the other references to the referenced object are cleared, the garbage collector releases the memory the object takes up. In other words, once it's no longer needed, the key object in the WeakMap and its key-value pair disappear on their own, with no need to delete the reference by hand.

Use:
- Basically, if you want to add data to an object without interfering with the garbage collector, you can use WeakMap.
A typical use case is adding data to DOM elements on a web page, where a WeakMap structure works well. When the DOM element is removed, its WeakMap record is removed automatically.

```js
const wm = new WeakMap();

const element = document.getElementById('example');

wm.set(element, 'some information');
wm.get(element) // "some information"
```
The reference to element inside the WeakMap is a weak reference and doesn't count for the garbage collector. That is, the reference count of the DOM node object above is 1, not 2. Then, as soon as the reference to the node is removed, the memory it takes up is released by the garbage collector. The key-value pair saved in the WeakMap disappears automatically as well.

In short, **WeakMap's dedicated use case is when the objects its keys refer to may disappear in the future. The WeakMap structure helps prevent memory leaks.**

> Note that WeakMap only holds the key weakly, not the value. The value is still a normal reference.
```js
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};

wm.set(key, obj);
obj = null;
wm.get(key)
// Object {foo: 1}
```
In the code above, the value obj is a normal reference. So even if the reference to obj is removed outside the WeakMap, the reference inside the WeakMap still exists.

Syntax:
- No iteration operations (no keys(), values() and entries() methods), and no size property.
- Can't be emptied, i.e. no support for the clear method.
- WeakMap has only four methods available: get(), set(), has(), delete().

Use cases:
1. DOM nodes as keys
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
2. Deploying private properties
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
In the code above, the two internal properties of the Countdown class, _counter and _action, are weak references to the instance, so if the instance is deleted, they disappear with it and cause no memory leak.

## WeakSet[^2]

How it differs from Set:
- WeakSet's members can only be objects, not values of other types.
- The objects in a WeakSet are all weak references, i.e. the garbage collector doesn't consider the WeakSet's reference to the object.
    (That is, if no other objects reference the object anymore, the garbage collector automatically reclaims the memory it takes up, without considering that the object still exists in the WeakSet.)
    
Use:
- WeakSet is good for temporarily storing a group of objects, and for storing information bound to objects. As soon as these objects disappear from the outside, their references in the WeakSet disappear automatically.
- One use of WeakSet is storing DOM nodes without worrying about memory leaks when these nodes are removed from the document.

Usage:
- WeakSet is a constructor; you can use the `new` command to create a WeakSet data structure.
- Three methods: `add`, `delete`, `has`.
- No `size` property, and no way to iterate over its members; trying to get the `size` and `forEach` properties won't work.

[^1]: [WeakMap - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)
[^2]: [WeakSet - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet)
