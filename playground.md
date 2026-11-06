---
title: Playground
description: Formats we often use when editing this site
published: true
date: 2026-09-30T13:35:19.000Z
tags: playground
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/playground.md)

Use this page when editing an article: compare the source with its rendered result, then copy the example you need. The sections cover ordinary Markdown and the Wiki.js formatting used on this site. Keep the `{.tabset}`, `{.links-list}`, and callout markers with their examples; a GitHub preview may display them differently from the site.

# Markdown

## Code blocks

```js
const p = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('miao')
    resolve(1)
  }, 3000)
})
```

## Quotes

> First of all, let there be light.
  —— Hydra Meow-Meow

## Lists

- HiJack
  - coffee
  - music
- Hydra
  - game
  - anime

## Tables

| Tables   |      Are      |  Cool |
|----------|:-------------:|------:|
| col 1 is |  left-aligned | $1600 |
| col 2 is |    centered   |   $12 |
| col 3 is | right-aligned |    $1 |


# Emoji

`:apple:` will produce :apple:
`:dog:` will produce :dog:
`:leaves:` will produce :leaves:

# Tab groups

## Tabset {.tabset}

### Tab1
MiaoMiaoMiao

### Tab2
BulaBulaBula

# Links

- [Google *google.com*](https://www.google.com)
- [Github *github.com*](https://github.com)
{.links-list}

# Callouts

> Please say I love you :)
{.is-info}

> This module is only available from version **2.4 and up**.
{.is-warning}

> Oh no :(
{.is-danger}
