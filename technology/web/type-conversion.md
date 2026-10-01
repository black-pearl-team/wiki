---
title: JavaScript Type Conversion
description: Honestly, you could also call it black magic
published: true
date: 2026-09-30T13:35:19.000Z
tags: javascript, type-system
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/technology/web/type-conversion.md)

### In JS there are only three kinds of type conversion:
- to a boolean
- to a string
- to a number

<img src="/technology/web/type-conversion/primitivetypesconvert.jpg" alt="primitivetypesconvert" width="80%">

### Object to primitive

When an object is converted, the built-in `[[ToPrimitive]]` function is called, and its algorithm generally goes like this:
* If it's already a primitive, no conversion is needed
* Call x.valueOf(); if that converts to a primitive, return the converted value
* Call x.toString(); if that converts to a primitive, return the converted value
* If neither returns a primitive, an error is thrown

#### valueOf()
The method returns the primitive value of the given object. If the object has no primitive value, valueOf returns the object itself.
<img src="/technology/web/type-conversion/valueof.jpg" alt="valueof" width="65%">

#### toString()
Returns a string that represents the object.
If this method isn't overridden in a custom object, toString() returns "[object type]", where type is the object's type.

### Arithmetic operators

##### Addition
- If either side of the operation is a string, the other side is converted to a string as well (if it's an object, `[[ToPrimitive]]` is called)
- If one side is neither a string nor a number, it's converted to a number or a string (if it's an object, `[[ToPrimitive]]` is called)
- `+a`, a plus sign followed by a non-number type, converts it straight to a number (highest precedence)

```js
1 + '1'     // '11'
true + true     // 2
4 + [1,2,3] // "41,2,3"
'a' + + 'b' // -> "aNaN"
('b' + 'a' + + 'a' + 'a').toLowerCase()   // 'banana'
```

##### Everything except addition
- As long as one side is a number, the other side is converted to a number

```js
4 * '3' // 12
4 * [] // 0
4 * [1, 2] // NaN
```

### Comparison operators
- If it's an object, the object is converted with toPrimitive
- If it's a string, the comparison goes by unicode character index

### ==
- First it checks whether the two types are the same: if they are, it compares the values; if not, it converts types
- It checks whether null and undefined are being compared; if so, it returns true
- If a value is true or false, it's turned into 1 or 0 and the comparison goes on
- It checks whether the two types are string and number; if so, the string is converted to a number
- It checks whether one side is an object and the other a string, number or symbol; if so, the object is converted to a primitive before comparing

```js
[] == ![] // true
"" == ![] // true
1 == ![] // false
0 == ![] // true
```
How the first expression is worked out:
```js
[] == ![]
[] == !true  // the ! operator has higher precedence than ==, so ! runs first
[] == false  // !true gives false
[] == 0  // comparison rule 1: if a value is true or false, turn it into 1 or 0 and keep comparing
[] == 0  // call valueOf on the [] on the left; [] is an object, so [].valueOf() returns [] itself
"" == 0 // call toString on the [] on the left; [].toString() returns ""
0 == 0  // comparison rule 2: if one value is a number and the other a string, convert the string to a number and then compare; "" becomes 0
In the end it evaluates 0 == 0, and the result is true
```

![convertprocess](/technology/web/type-conversion/convertprocess.jpg)

### My own summary

- In a calculation, object types are always turned into primitives first, via `[[ToPrimitive]]`
- In addition, **strings** have the highest priority: if one side is a string, the other side becomes a string; with no string or number, it's converted to a **number** or a **string** (number first, then string)
- For subtraction, multiplication and division, if there's a number, it all goes to number
- `!` followed by x: x is converted straight to a boolean
- For ==, both sides first go **object to value**, then **boolean to number**, then **number on one side and string on the other means converting to number**
