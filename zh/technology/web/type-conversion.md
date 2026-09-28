---
title: JavaScript 类型转换
description: 其实也可以看成玄学
published: true
date: 2021-01-08T03:45:51.048Z
tags: javascript, type-system
editor: markdown
dateCreated: 2020-12-03T10:46:33.305Z
---

### 在 JS 中类型转换只有三种情况，分别是：
- 转换为布尔值
- 转换为字符串
- 转换为数字

![primitivetypesconvert](/technology/web/type-conversion/primitivetypesconvert.jpg =80%x)

### 对象转原始类型

对象在转换类型的时候，会调用内置的 `[[ToPrimitive]]` 函数，对于该函数来说，算法逻辑一般来说如下：
* 如果已经是原始类型了，那就不需要转换了
* 调用 x.valueOf()，如果转换为基础类型，就返回转换的值
* 调用 x.toString()，如果转换为基础类型，就返回转换的值
* 如果都没有返回原始类型，就会报错

#### valueOf()
方法返回指定对象的原始值。如果对象没有原始值，则valueOf将返回对象本身。
![valueof](/technology/web/type-conversion/valueof.jpg =65%x)

#### toString()
返回一个表示该对象的字符串。
如果此方法在自定义对象中未被覆盖，toString() 返回 "[object type]"，其中 type 是对象的类型。

### 四则运算符

##### 加法
- 运算中其中一方为字符串，那么就会把另一方也转换为字符串（若是对象，调用 `[[ToPrimitive]]` ）
- 如果一方不是字符串或者数字，那么会将它转换为数字或者字符串（若是对象，调用 `[[ToPrimitive]]` ）
- `+a` 一个加号后跟非数字类型，直接转成数字（优先级最高））

```js
1 + '1'     // '11'
true + true     // 2
4 + [1,2,3] // "41,2,3"
'a' + + 'b' // -> "aNaN"
('b' + 'a' + + 'a' + 'a').toLowerCase()   // 'banana'
```

##### 除加法外
- 只要其中一方是数字，那么另一方就会被转为数字

```js
4 * '3' // 12
4 * [] // 0
4 * [1, 2] // NaN
```

### 比较运算符
- 如果是对象，就通过 toPrimitive 转换对象
- 如果是字符串，就通过 unicode 字符索引来比较

### ==
- 首先会判断两者类型是否相同：相同的话就是比大小；类型不相同就进行类型转换
- 会先判断是否在对比 null 和 undefined，是的话就会返回 true
- 如果值为 true 或 false，则转成 1 或 0 来继续比较
- 判断两者类型是否为 string 和 number，是的话就会将字符串转换为 number
- 判断其中一方是否为 object 且另一方为 string、number 或者 symbol，是的话就会把 object 转为原始类型再进行判断

```js
[] == ![] // true
"" == ![] // true
1 == ![] // false
0 == ![] // true
```
第一个式子计算过程：
```js
[] == ![]
[] == !true  // ! 操作符的优先级高于 == ，所以先执行 ! 操作
[] == false  // !true 得到的是 false
[] == 0  // 比较规则1：如果值为true或false，则转成1或0来继续比较
[] == 0  // 执行左侧的 [] 的 valueOf 方法，而 [] 是对象，所以 [].valueOf() 返回本身 []
"" == 0 // 执行左侧的 [] 的 toString 方法，[].toString() 返回 ""
0 == 0  // 比较规则2：如果一个值是数字，一个值是字符串，则把字符串转换为数字，再进行比较，"" 转成数字是 0
最终是执行 0 == 0 ，结果为 true
```

![convertprocess](/technology/web/type-conversion/convertprocess.jpg)

### 自己的总结

- 计算时，都会先将对象类型转为基本类型`[[ToPrimitive]]`
- 加法计算**字符串**优先级最高，一侧有 string，另一侧转 string；没有 string 或 number，就转为**数字**或**字符串**（先数字再字符串）
- 减乘除都是有数字转数字
- `!` 后跟 x，x 直接转布尔
- == 两边先**对象转值**，然后**布尔转数字**，然后**一侧数字一侧字符串就转数字**
