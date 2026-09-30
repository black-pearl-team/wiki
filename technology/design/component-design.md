---
title: Principles of Component Design
description: Thoughts and takeaways on standardizing components while building a shared front-end component library
published: true
date: 2026-09-30T13:35:19.000Z
tags: componentization
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/technology/design/component-design.md)

# Preface

Component-based development is efficient: a complex system is built out of specialized, easy-to-manage components.

This article is the author's thinking and experience on standardizing components while building a shared front-end component library. It draws on *7 Architectural Attributes of a Reliable React Component*[^1] and picks out a few principles we use often; the original goes into more detail and is worth a read.


# "Single responsibility" principle

> A component has a single responsibility when it has only one reason to change.

The single responsibility principle (SRP)[^2] asks that a component have one, and only one, reason to change.

The reason: changes to the component stay isolated and under control. It limits the component's size and keeps it focused on one thing, which makes it easier to code, change, reuse and test.  

*Anti-pattern* (it feels great while you're writing it):

- One component with many responsibilities means less work

- No need to identify the responsibilities and plan the structure around them

- One big component can do everything: no need to create a part for each responsibility

- No splitting, no overhead: no need to create props and callbacks for the split-up components to talk to each other


God component: the worst case of the multiple-responsibility problem (by analogy with the god object).  
A god component tends to know about and do everything. You might see it named  

- `<Application>`
- `<Manager>`
- `<Bigcontainer>`
- `<Page>`

Most likely with more than 500 lines of code......

<img src="/tech/componentization/multiple-responsibilities.jpeg" alt="multiple-responsibilities.jpeg" width="45%">

Don't turn off the light switch, because the same switch also runs the elevator.

# Encapsulation

> An encapsulated component provides props to control its behavior instead of exposing its internal structure.

Coupling is the property of a system that decides how much its components depend on each other. By the degree of that dependency, two kinds of coupling can be told apart:

- Loose coupling happens when an application's components know little or nothing about other components.

- Tight coupling happens when an application's components know lots of details about each other.

Loose coupling is our goal when we design an application's structure and the relationships between its components.

<img src="/tech/componentization/loosely-coupled.jpeg" alt="loosely-coupled.jpeg" width="40%">

**Loose coupling** brings the following benefits:

- One part can be changed without affecting the rest of the application
- Any component can be replaced with another implementation
- Components can be reused across the whole application, which avoids duplicated code
- Independent components are easier to test, which raises test coverage


A tightly coupled system, on the other hand, loses the benefits described above. The main drawback is that a component that depends heavily on other components is hard to change. Even a single change can force a whole chain of dependent components to change too.

<img src="/tech/componentization/tighly-coupled.jpeg" alt="tighly-coupled.jpeg" width="40%">

**Encapsulation**, or **information hiding**, is a basic principle of how to design components, and the key to loose coupling.

## Information hiding

- Setting refs, owning state, using lifecycle methods...

## Communication

- props

Props are best kept to primitive data (for example string, number, boolean):

```jsx
	<Message text="Hello world!" modal={false} />;
```
When needed, use complex data structures such as objects or arrays:

```jsx
	<MoviesList items={['Batman Begins', 'Blade Runner']} />
```

A prop can be an event handler or an async function:

```jsx
	<input type="text" onChange={handleChange} />
```

A prop can even be a component constructor. The component can then take care of instantiating other components:

```jsx
	function If({ component: Component, condition }) {
	    return condition ? <Component /> : null;
	}
	<If condition={false} component={LazyComponent} />  
```

To avoid breaking encapsulation, watch what gets passed through props. A parent component that sets props on its children shouldn't expose any details of its internal structure. Passing whole component instances or refs through props, for example, is bad practice.

# Composition

> A composable component is built from smaller, specialized components.

Composition is a way of making bigger components by putting components together. Composition is at the heart of React.

Take a bunch of small pieces, put them together, and build one bigger fella.

<img src="/tech/componentization/composable.jpeg" alt="composable.jpeg" width="40%">

## Benefits of composition
- Single responsibility (phone -> condition)
- Reusable (mount different modules depending on different logic)
- Flexible (any module can easily be swapped for another solution)

![page.jpeg](/tech/componentization/page.jpeg)


# Reuse

> A reusable component is written once and used many times.

## Reuse within the application

- Properly encapsulated components
- Internal implementation hidden
- Clear props

When a component that could live apart from the business logic gets written in a hurry, without thinking about how general its data should be, it's a blast to write at the time. Later, when it has to fit other business modules, going back to change it is a real headache, because once the data structure or the interaction changes, every place that already uses it has to change as well...

### Examples

- Controlled Components | React[^3]

## Reusing third-party libraries

- antd
- echarts
- react-virtualized or react-window...
- ......

### A checklist for whether a third-party library is worth using

- Documentation: check whether the library has a meaningful `README.md` file and detailed docs
- Tested: one clear sign of a trustworthy library is high test coverage
- Maintenance: look at how often the author adds new features, fixes bugs and does routine upkeep  

# Meaningful

> A meaningful component makes it easy to understand what it does.

Why code readability matters:

Developers spend most of their time reading and understanding code, not actually writing it. We spend 75% of our time understanding code, 20% changing existing code, and only 5% writing new code.  
emm......  

## Naming components

### Pascal Case

A component name is one or more Pascal-case words (mostly nouns) strung together, for example `<DatePicker>, <GridItem>, <Application>, <Header>`.

### Be specific

- `<HeaderMenu>`, `<SidebarMenuItem>` ✔️
- `<SaleTable>`, `<PhoneChart>` ✔️
- `<Item>`, `<Content>` ❌

### One word, one concept

- Rendering a collection of items: **list** or **table**


## The expressiveness staircase

- Reading variable names and props
- Reading docs/comments
- Browsing the code
- Asking the author...

<img src="/tech/componentization/expressiveness.jpeg" alt="expressiveness.jpeg" width="50%">

The lower a component sits on the staircase, the more effort it takes to understand.

Read open-source projects and borrow from them a lot ✔️


# Continuous improvement

Sometimes it's almost impossible to get the component structure right on the first try, because:  

- A tight project schedule doesn't leave enough time for system design
- The approach picked at the start was wrong
- You just found an open-source library that solves the problem better
- ~~Didn't sleep enough, in a bad mood~~
- Or any other reason

The more complex a component is, the more it needs checking and refactoring.  
<img src="/tech/componentization/improvement.jpeg" alt="improvement.jpeg" width="60%">

The ultimate solution: write reliable components ✔️ 

# Last but not least, you may need... 
- Load components on demand (babel-plugin-import)


[^1]: [7 Architectural Attributes of a Reliable React Component
](https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/)
[^2]: [Single responsibility principle | Wikipedia](https://en.wikipedia.org/wiki/Single_responsibility_principle)
[^3]: [Controlled Components | React](https://zh-hans.reactjs.org/docs/forms.html#controlled-components) (in Chinese)