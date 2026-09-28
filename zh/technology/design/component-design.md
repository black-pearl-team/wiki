---
title: 组件设计原则
description: 在搭建前端公共组件库时的组件规范化思考与总结
published: true
date: 2021-06-09T02:23:11.089Z
tags: componentization
editor: markdown
dateCreated: 2020-12-20T10:10:38.892Z
---

# 前言

基于组件的开发是高效的：一个复杂的系统是由专门的、易于管理的组件构建的。

此文是笔者在搭建前端公共组件库时的组件规范化思考与经验总结，参考 *7 Architectural Attributes of a Reliable React Component*[^1] 并提取了常用的几条原则，原文更为详尽，可供参考。


# “Single responsibility” 单一职责原则

> 当一个组件只有一个改变的原因时，它有一个单一的职责。
	
单一职责原则（SRP）[^2] 要求组件有一个且只有一个变更的原因。

理由：组件的修改隔离并且受控。限制了组件的大小，使其集中在一件事情上，便于编码、修改、重用和测试。  

*反例*（写的时候很嗨皮）：

- 一个拥有多职责的组件，工作量少

- 无需识别职责并相应地规划结构

- 一个大的组件可以做到一切：不需要为每个职责创建组成部分

- 无拆分-无开销：无需为拆分组件之间的通信创建 props 和 callbacks


上帝组件：多重责任问题的最坏情况（上帝对象的类比）。  
上帝组件倾向于了解并做所有事情。你可能会看到它名为  

- ` <Application>`
- `<Manager>`
- `<Bigcontainer>`
- `<Page>`

很可能代码超过500行。。。。。。

![multiple-responsibilities.jpeg](/tech/componentization/multiple-responsibilities.jpeg =45%x)

不要关闭电灯开关，因为这个开关同样作用于电梯。

# 封装

> 一个封装组件提供 props 控制其行为而不是暴露其内部结构。

耦合是决定组件之间依赖程度的系统特性。根据组件的依赖程度，可区分两种耦合类型：

- 当应用程序组件对其他组件知之甚少或一无所知时，就会发生松耦合。

- 当应用程序组件知道彼此的许多详细信息时，就会发生紧耦合。

松耦合是我们设计应用结构和组件之间关系的目标。

![loosely-coupled.jpeg](/tech/componentization/loosely-coupled.jpeg =40%x)

**松耦合** 会带来以下好处：

- 可以在不影响应用其它部分的情况下对某一块进行修改
- 任何组件都可以替换为另一种实现
- 在整个应用程序中实现组件复用，从而避免重复代码
- 独立组件更容易测试，增加了测试覆盖率

  
相反，紧耦合的系统会失去上面描述的好处。主要缺点是很难修改高度依赖于其他组件的组件。即使是一处修改，也可能导致一系列的依赖组件需要修改。

![tighly-coupled.jpeg](/tech/componentization/tighly-coupled.jpeg =40%x)

**封装** 或 **信息隐藏** 是如何设计组件的基本原则，也是松耦合的关键。

## 信息隐藏

- 设置 ref、拥有 state、使用生命周期方法……
	
## 通信

- props
	
建议 prop 的类型为基本数据（例如，string 、 number 、boolean）：

```jsx
	<Message text="Hello world!" modal={false} />;
```
必要时，使用复杂的数据结构，如对象或数组：

```jsx
	<MoviesList items={['Batman Begins', 'Blade Runner']} />
```

prop 可以是一个事件处理函数和异步函数：

```jsx
	<input type="text" onChange={handleChange} />
```

prop 甚至可以是一个组件构造函数。组件可以处理其他组件的实例化：

```jsx
	function If({ component: Component, condition }) {
	    return condition ? <Component /> : null;
	}
	<If condition={false} component={LazyComponent} />  
```

为了避免破坏封装，请注意通过 props 传递的内容。给子组件设置 props 的父组件不应该暴露其内部结构的任何细节。例如，使用 props 传输整个组件实例或 refs 都是一个不好的做法。

# 组合

> 一个组合式组件是由更小的特定组件组合而成的。
	
组合（composition）是一种通过将各组件联合在一起以创建更大组件的方式。组合是 React 的核心。

把一组小的片段，联合起来，创建一个更大个儿。

![composable.jpeg](/tech/componentization/composable.jpeg =40%x)

## 组合的好处
- 单一责任 (phone -> condition)
- 可重用（根据不同逻辑判断挂载不同模块）
- 灵活（某个模块随便可以用其他方案替代）

![page.jpeg](/tech/componentization/page.jpeg)


# 复用

> 可重用的组件，一次编写多次使用。
	
## 应用内的复用

- 合理封装的组件
- 隐藏了内部实现
- 有明确的 props

当一个可以脱离业务的组件，为了贪快，没有考虑到数据的通用性，写起来一时爽，到时候为了兼容其他的业务模块，回头改就很头疼了，因为改了数据结构或是交互等等，之前用的地方也都要改……

### 案例

- 受控组件 | React[^3]

## 复用第三方库

- antd
- echarts
- react-virtualized or react-window...
- ......

### 确定第三方库是否值得使用的检查清单

- 文档：检查库是否具备有意义的 `README.md` 文件和详细的文档
- 测试过的：可信赖库的一个显著特征就是有高的测试覆盖率
- 维护：看看库作者创建新特性、修改bug及日常维护的频率  

# 富有意义

> 一个有意义的组件很容易让人理解它的作用。
	
代码可读性的重要性：

开发人员大部分时间都在阅读和理解代码，而不是实际编写代码。我们花75％的时间理解代码，花20％的时间修改现有代码，只有5％编写新的代码。  
emm......  

## 组件命名

### Pascal Case 命名法

组件名是由一个或多个Pascal单词（主要是名词）串联起来的，比如：`<DatePicker>、<GridItem>、<Application>、<Header>`。

### 专业化

- `<HeaderMenu>`, `<SidebarMenuItem>` ✔️
- `<SaleTable>`, `<PhoneChart>` ✔️
- `<Item>`, `<Content>` ❌

### 一个单词 - 一个概念

- 渲染项集合： **list** or **table**


## 表现力阶梯

- 读取变量名 和 props
- 阅读文档/注释
- 浏览代码
- 咨询作者...

![expressiveness.jpeg](/tech/componentization/expressiveness.jpeg =50%x)

组件在楼梯上的位置越低，意味着需要更多的努力才能理解。

多阅读借鉴开源项目 ✔️


# 持续改进

有时，在第一次尝试时几乎不可能创建正确的组件结构，因为：  

- 紧迫的项目排期不允许在系统设计上花费足够的时间
- 最初选择的方法是错误的
- 刚找到了一个可以更好地解决问题的开源库
- ~~没睡饱，心情不好~~
- 或任何其他原因

组件越复杂，就越需要验证和重构。  
![improvement.jpeg](/tech/componentization/improvement.jpeg =60%x)

究极解决方案：编写可靠的组件 ✔️ 

# Last but not least, you may need... 
- 组件按需加载（babel-plugin-import）


[^1]: [7 Architectural Attributes of a Reliable React Component
](https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/)
[^2]: [Single responsibility principle | Wikipedia](https://en.wikipedia.org/wiki/Single_responsibility_principle)
[^3]: [受控组件 | React](https://zh-hans.reactjs.org/docs/forms.html#controlled-components)