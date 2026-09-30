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

*Much of the Chinese page is compiled from books (Professional JavaScript for Web Developers and Ruan Yifeng's ES6 tutorial). This English page sums up the same points in our own words instead of translating those passages.*

# Memory leaks

Programs need memory, and the operating system or runtime hands it out on request.

A long-running process (a daemon) has to give back memory it no longer uses; otherwise usage keeps growing until performance suffers or the process crashes.

Memory that's no longer needed but never released is a **memory leak**.


## Memory management

Keep only the data you actually need, and set references you're done with to null so they can be collected.
1. **Use const and let**
    Their block scope lets the garbage collector reclaim values sooner than function-scoped var does.
2. **Hidden classes and delete**
    V8 gives objects of the same shape a shared hidden class, which is faster; changing an object's shape later breaks the sharing.
```js
function Article() {
		this.title = 'Inauguration Ceremony Features Kazoo Band';
}
let a1 = new Article();
let a2 = new Article();
```
Both instances start out sharing one hidden class, since they come from the same constructor and prototype. Adding a property afterwards:
```js
a2.author = 'Jake';
```
splits them into two hidden classes, which costs performance if it happens often.
The fix is to declare every property in the constructor rather than bolting them on later.

delete has the same effect.
```js
function Article() {
		this.title = 'Inauguration Ceremony Features Kazoo Band';
    this.author = 'Jake';
}
let a1 = new Article();
let a2 = new Article();

delete a1.author;
```
After the delete, the two instances no longer share a hidden class. Setting an unwanted property to **null** instead keeps the shape intact and still lets the value be collected.

## Memory leak details

1. Accidental globals: the most common leak, and the easiest to fix.
2. Timers: a callback that closes over outside variables keeps them alive until the timer is cleared.
3. Closures can hold on to memory without you noticing.
4. Holding references to DOM nodes that have already been removed from the page.
5. iframes: if the parent keeps references into an iframe (or the iframe attaches its methods to the parent), removing the iframe won't free its JS heap or DOM memory.


# Garbage collection

Most languages manage memory for you; this is the "garbage collector".

V8 splits the heap into a **young generation** for short-lived objects (collected often) and an **old generation** for long-lived ones (collected less often).

## The young generation

The young generation is collected with the Scavenge algorithm.
Scavenge is mainly implemented with the Cheney algorithm.

> Cheney's algorithm collects by **copying**.
> The heap is split into two semispaces, and only one is in use at a time.
> New objects go into the From space; at collection time the live ones are copied to the To space, the rest is freed, and the two spaces swap roles.

Scavenge leaves half the heap unused, but it only copies live objects, which are few among short-lived data, so it's very fast: a classic **trade space for time**. That makes it a poor fit for the whole heap and a good one for the young generation.

> How are objects released?
Reachability analysis starts from a set of **GC ROOT** objects and follows their references.
Those paths are **reference chains**; an object with no chain back to a root is unreachable.
Unreachable objects aren't freed at once: they're **marked**, filtered and queued for collection, and one that gets referenced again in the meantime survives.

- Promotion

Moving an object from the young generation to the old one is called promotion.
Live objects are checked before being copied to the To space, and the long-lived ones get promoted.

Two main conditions trigger it:

1. The object has already survived one Scavenge
2. The To space is more than 25% full

The 25% limit exists because the To space becomes the next From space, and a crowded one would leave little room for new allocations.

## The old generation

> **Mark-and-sweep**

A variable is marked as in context when it's declared (say, inside a function) and as out of context when it leaves; in-context variables must never be freed.

How the marking is done varies (the principle is deciding whether a variable is **in context**).

The **garbage collector** marks everything in memory, removes the marks from whatever is in context or reachable from it, and then does a **memory cleanup** of what's still marked.

Modern collectors decide when to run by watching the runtime.
The heuristics differ by engine but mostly look at **the size and number of allocated objects**.
A 2016 post from the V8 team describes its **heap growth strategy**: after a full collection, the next one is timed from **the number of live objects plus a margin**.

> **Reference counting**

A less common approach: the engine counts references to each value, and a value whose count drops to `0` can be freed.

If a value is no longer needed but its count never reaches `0`, it can't be freed, and that's a memory leak.


# How to spot memory leaks

- Rule of thumb:
If memory usage keeps growing across five garbage collections in a row, there's a leak, so watch memory usage live.

- Browser:
Chrome Devtool -> Performance -> check Memory -> record, interact, Stop -> read the chart

- Command line:
`process.memoryUsage()`, which Node provides
`heapUsed`
```js
console.log(process.memoryUsage());
// { rss: 27709440,
//  heapTotal: 5685248,
//  heapUsed: 3449392,  <- judge memory leaks by this field.
//  external: 8772 }
```


# ES6 solutions

Ideally you could say, when creating a reference, that it shouldn't keep the object alive, so only the main references would need clearing.

ES6 added WeakSet and WeakMap for this: their references don't count for the garbage collector, hence the "Weak".

## WeakMap[^1]

How it differs from Map:
- Its keys must be objects (not null).
- The objects its keys point to aren't kept alive by it.

Syntax differences:
- No iteration (`keys()`, `values()`, `entries()`) and no `size` property.
- No `clear` method, so only four methods: `get()`, `set()`, `has()`, `delete()`.

> Example:
>Storing extra data about an object normally creates a reference to it.
>```js
> const e1 = document.getElementById('foo');
> const e2 = document.getElementById('bar');
> const arr = [
>   [e1, 'foo element'],
>   [e2, 'bar element'],
> ];
> ```
> Here arr references e1 and e2 in order to attach notes to them.
> Once they're no longer needed, those references have to be removed by hand, or e1 and e2 can't be collected. (By hand: `arr[0] = null; arr[1] = null;`)

WeakMap solves this: its key references are weak, so once nothing else refers to the key object, the object is collected and its entry disappears by itself.

Use:
- Use WeakMap to attach data to objects without getting in the garbage collector's way.
A typical case is data attached to DOM elements: when an element goes away, its entry goes with it.

```js
const wm = new WeakMap();

const element = document.getElementById('example');

wm.set(element, 'some information');
wm.get(element) // "some information"
```
The WeakMap's reference to element is weak, so the node's reference count stays at 1, not 2; drop the other reference and the node is collected along with its entry.

In short, **WeakMap is for keys whose objects may go away later, and it helps prevent memory leaks.**

> Note that only the key is held weakly; the value is an ordinary reference.
```js
const wm = new WeakMap();
let key = {};
let obj = {foo: 1};

wm.set(key, obj);
obj = null;
wm.get(key)
// Object {foo: 1}
```
Here the WeakMap holds obj through a normal reference, so clearing obj outside doesn't remove it inside.

Syntax:
- No iteration (no keys(), values() or entries()), and no size property.
- It can't be cleared (no clear method).
- Only four methods: get(), set(), has(), delete().

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
2. Private properties
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
Here _counter and _action are held weakly per instance, so deleting an instance takes them with it, with no leak.

## WeakSet[^2]

How it differs from Set:
- Its members can only be objects.
- Members are held weakly, so the garbage collector ignores the WeakSet's reference.
    (If nothing else refers to an object, it's collected even though it's still in the WeakSet.)
    
Use:
- Good for temporarily holding a group of objects, or information tied to them; entries vanish when the objects do.
- For example, holding DOM nodes without leaking memory when they're removed from the document.

Usage:
- WeakSet is a constructor: create one with `new`.
- Three methods: `add`, `delete`, `has`.
- No `size` and no way to iterate; `size` and `forEach` don't work on it.

[^1]: [WeakMap - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap)
[^2]: [WeakSet - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakSet)
