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

*Parts of the Chinese page follow Dmitri Pavlutin's article on reliable React components; this English page sums those parts up in our own words instead of translating them back into English.*

# Preface

Building from components is efficient: a complex system is put together from small, manageable parts.

These are the author's thoughts and lessons from standardizing components while building a shared front-end component library. They draw on *7 Architectural Attributes of a Reliable React Component*[^1] and pick out the principles we use most; the original goes into much more detail and is worth a read.


# "Single responsibility" principle

> A component should have just one reason to change.

That's what the single responsibility principle (SRP)[^2] asks for.

Why: changes stay contained, and a small, focused component is easier to write, change, reuse and test.  

*Anti-pattern* (it feels great while you're writing it):

- One component doing many jobs seems like less work

- You get to skip thinking about responsibilities and structure

- One big component handles everything, with no parts to split out

- No splitting means no props or callbacks to wire between the parts


God component: the extreme case of too many responsibilities, named after the god object.  
It knows about and does everything. You might see it named  

- `<Application>`
- `<Manager>`
- `<Bigcontainer>`
- `<Page>`

Most likely with more than 500 lines of code......

<img src="/tech/componentization/multiple-responsibilities.jpeg" alt="multiple-responsibilities.jpeg" width="45%">

Don't turn off the light switch, because the same switch also runs the elevator.

# Encapsulation

> An encapsulated component takes props to control its behavior and keeps its internals to itself.

Coupling is how much components depend on one another. It comes in two kinds:

- Loose coupling: components know little or nothing about each other.

- Tight coupling: components know a lot of each other's details.

Loose coupling is what we aim for when we structure an app and its components.

<img src="/tech/componentization/loosely-coupled.jpeg" alt="loosely-coupled.jpeg" width="40%">

**Loose coupling** gives you:

- Changes in one area that don't ripple into the rest
- Components you can swap for another implementation
- Reuse across the app instead of duplicated code
- Independent components that are easier to test

  
Tight coupling loses all of that: a component that leans on many others is hard to change, and a single change can cascade through everything that depends on it.

<img src="/tech/componentization/tighly-coupled.jpeg" alt="tighly-coupled.jpeg" width="40%">

**Encapsulation**, or **information hiding**, is the basic principle of component design and the key to loose coupling.

## Information hiding

- Refs, state, lifecycle methods... stay inside the component.
	
## Communication

- props
	
Prefer primitive values for props (string, number, boolean):

```jsx
	<Message text="Hello world!" modal={false} />;
```
Reach for objects or arrays when you need them:

```jsx
	<MoviesList items={['Batman Begins', 'Blade Runner']} />
```

Props can be event handlers or async functions:

```jsx
	<input type="text" onChange={handleChange} />
```

A prop can even be a component constructor, so a component decides what gets instantiated:

```jsx
	function If({ component: Component, condition }) {
	    return condition ? <Component /> : null;
	}
	<If condition={false} component={LazyComponent} />  
```

Don't leak internals through props: a parent shouldn't hand its component instances or refs down to its children.

# Composition

> A composable component is assembled from smaller, specific components.

Composition means building bigger components out of smaller ones, and it sits at the heart of React.

Take a bunch of small pieces, put them together, and build one bigger fella.

<img src="/tech/componentization/composable.jpeg" alt="composable.jpeg" width="40%">

## Benefits of composition
- Single responsibility (phone -> condition)
- Reusable (mount different modules depending on different logic)
- Flexible (any module can easily be swapped for another solution)

![page.jpeg](/tech/componentization/page.jpeg)


# Reuse

> Write it once, use it many times.
	
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

- Docs: a useful `README.md` file and real documentation
- Tests: good test coverage is a strong sign you can trust it
- Maintenance: how actively features, fixes and upkeep keep coming  

# Meaningful

> A meaningful component makes its purpose obvious.
	
Why readability matters:

We read and puzzle over code far more than we write it; one common estimate splits the time 75% understanding, 20% changing existing code and 5% writing new code.  
emm......  

## Naming components

### Pascal Case

Component names are one or more Pascal-case words (mostly nouns) joined together, for example `<DatePicker>, <GridItem>, <Application>, <Header>`.

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

The further down the staircase you have to go, the harder the component is to understand.

Read open-source projects and borrow from them a lot ✔️


# Continuous improvement

Getting the component structure right on the first try rarely happens, because:  

- The schedule is too tight for proper design
- The first approach turned out to be wrong
- A better open-source library just turned up
- ~~Didn't sleep enough, in a bad mood~~
- Or any other reason

The more complex a component gets, the more it needs checking and refactoring.  
<img src="/tech/componentization/improvement.jpeg" alt="improvement.jpeg" width="60%">

The ultimate solution: write reliable components ✔️ 

# Last but not least, you may need... 
- Load components on demand (babel-plugin-import)


[^1]: [7 Architectural Attributes of a Reliable React Component
](https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/)
[^2]: [Single responsibility principle | Wikipedia](https://en.wikipedia.org/wiki/Single_responsibility_principle)
[^3]: [Controlled Components | React](https://zh-hans.reactjs.org/docs/forms.html#controlled-components) (in Chinese)