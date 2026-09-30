---
title: Using Chrome DevTools
description: A little guide to the browser's developer debugging tools
published: true
date: 2026-09-30T13:35:19.000Z
tags: browser, chrome
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/technology/web/chrome-devtools.md)

# Preface

This article briefly lists some of the features of Chrome DevTools. It looks like a lot, but you'll remember them once you've used them enough. (Performance isn't covered; I plan to write a separate article on that part.)
Best enjoyed together with Google's official DevTools docs[^1].


# General

## Start shortcuts

### Open DevTools
	Ctrl + Shift + I (Windows) or Cmd + Opt + I (Mac)

### Open DevTools & inspect an element
	Ctrl + Shift + C (Windows) or Cmd + Opt + C (Mac)

## Panels

- Elements panel
- Console panel
- Sources panel
- Network panel
- Performance panel
- Memory panel
- Application panel
- Security panel

## copying & saving

- copy(...)
	`location --> copy(location) --> paste`
	`copy($0)`
- Store as global
	`console --> right-click -->  "Store as global variable"`
- Save the stack trace
	`right-click × --> Save as ...`
- Copy HTML directly 
 	`right-click --> save element`

## Shortcuts and general tips

### Switch the DevTools window layout

- Spot: ctrl + shift + D (⌘ + shift + D Mac) 
- Mobile: ctrl + shift + M (⌘ + shift + M Mac) 

### Switch between DevTools panels

- Switch to the left/right panel: ctrl + \[ and ctrl + \] (⌘ on Mac)
- ctrl + 1 to ctrl + 9 jump straight to panels 1...9 (ctrl + 1 goes to the Elements panel, ctrl + 4 to the Network panel, and so on) P.S.: disabled by default, DevTools --> Settings --> Preferences --> *Appearance*

### Increment/decrement

Adjusting styles:
	The up / down arrow keys, alone or with a modifier key, nudge numeric values by 0.1, 1 or 10  
![addsub.png](/tech/web/chrome-devtools/addsub.png)

### Searching in elements, logs, sources & network

The first 4 main panels in DevTools all take [ctrl] + [f], each with its own kind of query:

- In the Elements panel: search by string, selector or XPath
- In the Console, Network and Source panels: search by case-sensitive text, or by text read as an expression
![find.png](/tech/web/chrome-devtools/find.png)

## Using Command

The Command menu is the quick way to reach features that are tucked away

- With Chrome's DevTools open, press Ctrl + Shift + P ( Mac: ⌘ + Shift + P )
- Use the Run Command option under the DevTools dropdown button

![command.png](/tech/web/chrome-devtools/command.png)

### Screenshots
- Node: Command --> screen --> node...
- Full page: Command --> screen --> full size...

### Switch the panel layout
- Command --> layout   three options

### Switch the theme
- Command --> theme

## Using snippets

### Save a snippet
- `Source` --> `>>` --> `Snippets` --> `new & save` --> `Ctrl + Enter` ( Mac: `⌘  + Enter`)

### Use snippets anywhere
- Command --> type !

# console

## **About `$`**
### `$0`
- $0 is a reference to the currently selected html node
- $1 is a reference to the node selected before that
- ... and so on up to $4

You can try some related operations (for example: $1.appendChild($0))

### `$ and $$`

If the project hasn't defined `$`

- `$ == querySelector`
- `$$ == querySelectorAll`

### `$_`

- `$_`: a reference to the result of the last execution

### `$i`

To use third-party libraries in the console tab of this tool, first install the *Console Importer*[^2] extension, then import them like this:

- `$i('lodash')`
- `$i('moment')`
- ...

## Conditional breakpoints

- Right-click the line number and choose Add conditional breakpoint...
- Once it's set: Edit breakpoint

## BreakPoints Section

- Right-click --> disable all ...

## console.?

1. console.assert

2. Make logs easier to read: console.log({ var1, var2 })

3. console.table(): works for arrays, array-likes and objects; pass the columns you want to see as the second argument

4. console.dir(): see the real js object tied to a DOM node  
div = $('div'): creates a DOM element

5. Time an execution  
`console.time()` — start a timer  
`console.timeEnd()` — stop the timer and print the result in the console
You can pass in a label

6. The "eye" symbol, to define any JavaScript expression: e.g.: location.href
7. Add timestamps to logs: Command --> timestamps
8. Add CSS styles to console.log: console.log('%cWhoops...','font-size: 50px; color: red;')

# Network

1. Request initiator shows the call stack  
Makes it clear which line of which script triggered the request  
Shows the last step in the call stack that triggered the request  
Points into some lower-level libraries

2. Filter: type a string or a regular expression to filter requests; Ctrl + Space shows every possible keyword  
domain
method
...

3. Request table: right-click the header row to add columns (for example, Method is one we often add)
initiator column: shows the call stack, i.e. which line of which script triggered the request  
Response Headers: controls which response headers are shown

4. Resend an XHR request (only for requests whose Type in the table is xhr)

5. XHR/fetch breakpoints
![xhr_breakpoints.png](/tech/web/chrome-devtools/xhr_breakpoints.png)


# Elements panel

## Tips

### `h`
  Hide an element with 'h'

### Drag & drop elements
  In Elements

### Move elements with control (the key)
  `[ctrl]` + `[⬆]` / `[ctrl]` + `[⬇]`
  \(`[⌘]` + `[⬆]` / `[⌘]` + `[⬇]` on Mac)

### Basic-editor-style operations in the Elements panel

#### Edit, undo:

- `[ctrl]` + `[z]`
- \(`[⌘]` + `[z]` on Mac)

### Shadow editor

Styles panel --> box-shadow / text-shadow property --> the square shadow symbol

### Timing function editor

The curve symbol next to it (if the timing function's value isn't set in this shorthand form, the symbol won't show up)

### Buttons for inserting style rules

Hover at the end of a style selector's area and buttons appear for adding CSS properties quickly through the Color and Shadow editors:

- text-shadow
- box-shadow
- color
- background-color

### Expand every child node in the Elements panel
The expand recursively command when you right-click a node

### DOM breakpoints

Track changes to the DOM  

- Choose subtree modifications: listens for any node inside it being removed or added
- Choose attribute modifications: listens for any attribute of the currently selected node being added, removed or changed
- Choose node removal: listens for the selected element being removed
![dom_breakpoints.png](/tech/web/chrome-devtools/dom_breakpoints.png)

The breakpoint list

![breakpoints_hint.png](/tech/web/chrome-devtools/breakpoints_hint.png)

## Color picker

![color_selector.png](/tech/web/chrome-devtools/color_selector.png)

### Pick only the colors you're using

Part of what the color picker offers:

- Switch to a Material palette with shade variations
- Custom, where you can add and remove colors
- Pick, from CSS Variables, a color that exists in a stylesheet your current page uses
- Or all the colors you use in the page's CSS

![color_palettes.png](/tech/web/chrome-devtools/color_palettes.png)

### Pick your colors visually

The text's color picker (color property) --> Contrast ratio:
how **the text's color** stands against **the background DevTools assumes for this text**

- A "🚫" next to the number means the contrast is too low
- A "✅" means the color meets the AA level of *Web Content Accessibility Guidelines (WCAG) 2.0*[^3], which means a contrast ratio of at least 3
- "✅ ✅" means it meets the AAA level

# Drawer

Under the main window sits a second row of tabs.
That row is the `Drawer`, which keeps the console and other tools at hand whichever panel you're in.

### How to open the Drawer
While in DevTools (any tab), press `[esc]` to show it, and press `[esc]` again to hide it

### What's actually in the Drawer
- Click the `⋮` icon in front of the Drawer's console panel on the main page to open the full list of options
- Command --> Drawer

One more look at all the options:

- Animations
- Changes
- Console
- Coverage
- Network conditions
- Performance monitor
- Quick source
- Remote devices
- Rendering
- Request blocking
- Search
- Sensors
- What’s new

### Control the sensors
Drawer --> Sensors

### Simulate network conditions
Drawer --> Network conditions

### Get the source
Drawer --> Quick Source

### Check code coverage

Drawer --> Coverage
![coverage.png](/tech/web/chrome-devtools/coverage.png)


[^1]: [Google's official DevTools docs](https://developers.google.com/web/tools/chrome-devtools/)
[^2]: [Console Importer](https://chrome.google.com/webstore/detail/console-importer/hgajpakhafplebkdljleajgbpdmplhie/related)
[^3]: [Web Content Accessibility Guidelines (WCAG) 2.0](https://www.w3.org/TR/UNDERSTANDING-WCAG20/conformance.html)
