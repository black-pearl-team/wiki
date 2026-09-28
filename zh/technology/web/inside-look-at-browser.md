---
title: 现代浏览器 - 深入理解
description: 对 Mariko Kosaka 所著 Inside look at modern web browser 的理解
published: true
date: 2021-05-20T06:26:25.889Z
tags: browser
editor: markdown
dateCreated: 2020-12-07T07:23:09.114Z
---

# 前言

此文可看做 Chrome 开发者 Mariko Kosaka 的[《Inside look at modern web browser》](https://developers.google.com/web/updates/2018/09/inside-browser-part1)的译文。
网络上已存在不同版本、质量的翻译和糅杂。笔者不做糅杂，只愿结合自己的理解，呈现较流畅、平实的语言版本，加入部分专业术语的官方链接，以供参考和理解。


> 在这个由 4 部分组成的文章中，我们将在 Chrome 浏览器中由浅至深地探索从高级架构到渲染管道细节的内容。如果您想知道浏览器如何将你的代码变成运转起来的网站，或者不确定为什么某些特定技术会被推荐用来提高性能，那么本系列适合您。


# 第一部分

核心计算术语、Chrome 多进程结构

## 计算机的核心是 CPU 和 GPU

### 中央处理器-CPU
中央处理器-CPU，可以视为计算机的大脑。
一个CPU内核可以在许多不同的任务传入时一个接一个地处理。
过去，大多数CPU都是单芯片。内核就像生活在同一芯片中的一个CPU。

### 图形处理单元-GPU
图形处理单元-GPU，与CPU不同，GPU擅长处理简单任务，但同时**跨**多个内核。

![computerlevelstruc.png](/tech/web/browser/computerlevelstruc.png)
（计算机体系结构的三层。**机器硬件**在底部，**操作系统**在中间，**应用程序**在顶部。）

## 进程 与 线程

进程可以描述为应用程序的执行程序。
线程是存在于进程内部并由它们执行其进程程序的任何部分。

![process&thread.png](/tech/web/browser/process&thread.png)
（进程作为边界框，线程作为抽象鱼在进程内部游动。）

![process&threadinmemory.png](/tech/web/browser/process&threadinmemory.png)
启动应用程序时，将创建一个进程。该程序可能会创建线程来帮助其工作，但这是可选的。
操作系统为进程提供了一块内存，所有应用程序状态都保留在该专用内存空间中。
当关闭应用程序时，该进程也将消失，并且操作系统会释放内存。

![ipc.png](/tech/web/browser/ipc.png)
一个进程可以要求操作系统启动另一个进程来运行不同的任务。发生这种情况时，将为新进程分配内存的不同部分。
如果两个进程需要通话，则可以使用 **进程间通信（IPC）** 进行通话。
许多应用程序都以这种方式工作，因此，如果工作进程无响应，则可以重新启动它，而无需停止正在运行应用程序不同部分的其他进程。

## 浏览器结构

那么如何使用进程和线程构建 Web 浏览器？
它可以是一个具有许多不同线程的进程，也可以是多个具有几个通过 IPC 进行通信的线程的进程。
这里要注意的重要一点是，这些**不同的体系结构是实现细节**。这里以 Chrome 举例。

![browserstruc.png](/tech/web/browser/browserstruc.png)
顶部是**浏览器进程**，它与负责应用程序不同部分的其他进程进行协调。
对于**渲染器进程**，将创建多个进程并将其分配给每个选项卡。
（直到最近，Chrome才尽可能为每个标签提供了一个进程。现在，它尝试赋予每个网站自己的进程，包括iframe。）

## 哪些进程都是管理些什么呢？

![kindsofprocesses.png](/tech/web/browser/kindsofprocesses.png)

- **浏览器进程**
控制应用程序的“ chrome”部分，包括地址栏，书签，后退和前进按钮。 还处理 Web 浏览器的隐形，特权部分，例如网络请求和文件访问。

- **渲染器进程**
控制显示网站的 tab 页内的所有内容。

- **插件进程**
控制网站使用的所有插件，例如 Flash。

- **GPU进程**
与其他进程隔离地处理 GPU 任务。由于 GPU 处理来自多个应用程序的请求并将它们绘制在同一表面上，因此将其分为不同的进程。

## Chrome 多进程的优势

- **不受其他 tab 页的卡顿影响**
假设您有 3 个标签页处于打开状态，每个标签页均由独立的渲染器进程运行。如果一个选项卡变得无响应，则可以关闭无响应的选项卡并继续运行，同时保持其他选项卡的活动状态。
如果所有选项卡都在一个进程上运行，则当一个选项卡无响应时，所有选项卡将无响应。

- **安全性和沙箱隔离**(内存保护、访问控制)
由于操作系统提供了一种限制进程特权的方法，因此浏览器可以从某些功能中限制某些进程。例如，Chrome 浏览器限制了处理诸如渲染器进程之类的任意用户输入的进程的任意文件访问。

- **节省内存的控制**
由于进程具有自己的私有内存空间，因此它们通常包含通用基础结构的副本（例如 V8）。这意味着更多的内存使用情况，因为如果它们是同一进程中的线程，将无法共享它们。
为了节省内存，Chrome 对可启动的进程数量进行了限制。该限制取决于设备拥有的内存和 CPU 能力，但是 Chrome 达到限制后，它将开始在同一进程中运行同一站点的多个标签页。

## 节省更多内存 - Chrome 中的服务化

将相同的方法应用于浏览器进程。
Chrome 正在进行架构更改，以将浏览器程序的每个部分作为一项服务运行，从而可以轻松拆分为不同的进程或聚合为一个进程。

一般的想法是，当 Chrome 在功能强大的硬件上运行时，它可能会将每个服务拆分为不同的进程，从而提供更高的稳定性，但是如果在资源受限的设备上，Chrome 会将服务整合为一个进程，从而节省了内存。

## 分帧渲染器进程 - 站点隔离

网站隔离可为每个跨网站 iframe 运行单独的渲染器进程。
“同源策略”[^1]是 Web 的核心安全模型。这样可以确保一个站点未经同意就无法访问其他站点的数据。绕过此策略是安全攻击的主要目标。
**进程隔离**是分离站点的最有效方法。借助 Meltdown 和 Spectre[^2]，我们更加明显地需要使用流程来分离站点。自 Chrome 67 起，默认情况下在桌面上启用“网站隔离”，标签中的每个跨网站 iframe 都会获得单独的渲染器进程。


# 第二部分

核心计算术语、Chrome 多进程结构

## 导航栏中会发生什么

从一个简单的网络浏览用例开始：您在浏览器中输入一个 URL，然后浏览器从 Internet 上获取数据并显示一个页面。（面试中躲不掉的招）
这一小节，重点介绍用户请求网站且浏览器准备呈现页面（也称为导航）的部分。

## 从浏览器进程开始

从第一部分可知，选项卡之外的所有内容都由**浏览器进程**处理。

浏览器进程具有以下线程：
- **UI 线程**（用于绘制浏览器的按钮和输入字段）
- **网络线程**（用于处理网络堆栈以从 Internet 接收数据）
- **存储线程**（用于控制对文件的访问）等等

在地址栏中键入 URL 时，输入的内容将由浏览器进程的 UI 线程处理。
![navigation.png](/tech/web/browser/navigation.png)

## 来一个简单的导航过程

### 第一步：处理输入

当用户开始在地址栏中输入内容时，**UI 线程**首先问的是：“这是搜索查询还是 URL？”。
在Chrome浏览器中，**地址栏**也是**搜索输入字段**，因此UI线程需要解析并决定是定位到搜索引擎还是请求的网站。
- 搜索查询：发送到搜索引擎
- URL：请求 URL 的网站

### 第二步：开始导航

当用户按下Enter键时，**UI 线程**会发起网络调用以获取网站内容。Loading 图标会显示在选项卡的角上，并且**网络线程**通过相应的协议为该 URL 请求查找（例如 DNS）和建立连接（例如 TLS）。
![startnavi.png](/tech/web/browser/startnavi.png)
（UI 线程与网络线程通信，以导航到mysite.com）

此时，网络线程可能会收到服务器的重定向状态码，例如 HTTP 301。在这种情况下，网络线程与 UI 线程进行通信，告知请求的服务器正在请求重定向。然后，将启动另一个 URL 请求。

### 第三步：读取响应结果

#### 3.1 确定文件MIME类型
一旦响应体（有效负载）开始进入，网络线程将在必要时查看流的前几个字节。响应的 `Content-Type` 标头应说明数据的类型，但是由于可能丢失或错误，因此在此处进行 [MIME Type 嗅探](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)。

每一个浏览器在不同的情况下会执行不同的操作。因为这个操作会有一些安全问题，有的 MIME 类型表示可执行内容而有些是不可执行内容。浏览器可以通过请求头 `Content-Type` 来设置 `X-Content-Type-Options` 以阻止MIME嗅探。

![filemimetype.png](/tech/web/browser/filemimetype.png)

#### 3.2 处理不同MIME文件
如果响应是 HTML 文件，则下一步是将数据传递到**渲染进程**，但是如果是zip文件或其他文件，则意味着这是下载请求，因此他们需要将数据传递到下载管理器。

#### 3.3 安全检查

- 恶意名单检查：如果域和响应数据在恶意站点名单中，则网络线程将发出警报以显示警告页面。
- 跨域读取检查：跨源读取拦截（Cross Origin Read Blocking）[^3]检查，以确保敏感的跨站点数据不会进入渲染器进程。
![securitycheckbynetworkthread.png](/tech/web/browser/securitycheckbynetworkthread.png)
（网络线程检查响应数据是否为来自安全站点的 HTML）

### 第四步：查找渲染进程

一旦完成所有检查，并且网络线程确信浏览器应导航到请求的站点，则网络线程将告知 UI 线程数据已准备就绪。
然后，UI 线程找到一个**渲染进程**来进行网页渲染。
![findrendererprocess.png](/tech/web/browser/findrendererprocess.png)

> **优化**
由于网络请求可能需要几百毫秒才能获得响应，因此将应用优化来加快此过程。当 UI 线程在步骤 2 向网络线程发送 URL 请求时，它已经知道他们正在导航到哪个站点。
**UI 线程尝试与网络请求并行地主动查找或启动渲染器进程。** 
这样，如果一切按预期进行，则当网络线程接收到数据时，渲染器进程已经处于备用位置。如果导航跨站点重定向，则可能不会使用此备用过程，在这种情况下，可能需要其他过程。

### 第五步：提交导航

现在已经准备好数据和渲染器进程，将 IPC 从**浏览器进程**发送到**渲染进程**以提交导航。
它还会传递数据流，因此渲染器进程可以继续接收HTML数据。
一旦浏览器进程听到渲染进程中的提交确认，导航即完成，文档加载阶段开始。

此时，地址栏已更新，安全指示符和站点设置UI反映了新页面的站点信息。
选项卡的会话历史记录将被更新，因此 后退/前进 按钮将逐步浏览刚刚导航到的站点。
为方便在关闭选项卡或窗口时恢复选项卡/会话，会话历史记录存储在磁盘上。

![commitnavigation.png](/tech/web/browser/commitnavigation.png)

### 额外步骤：初始加载完成

提交导航后，渲染器进程将继续加载资源并渲染页面。
渲染器进程“完成”渲染后，它将 IPC 发送回浏览器进程（这是在页面上所有帧上触发所有 `onload` 事件并完成执行之后）。此时，UI 线程在选项卡上停止加载 Loading 图标。
（此处“完成”是因为客户端 JavaScript 仍然可以在此之后加载其他资源并呈现新视图。）

![pageloaded.png](/tech/web/browser/pageloaded.png)

## 导航到其他站点

如果用户再次将不同的 URL 放入地址栏会发生什么？
好吧，浏览器过程将通过相同的步骤导航到不同的站点。
但是在这样做之前，它需要检查当前渲染的站点是否关心 `beforeunload` 事件。

beforeunload 可以在尝试导航或关闭选项卡时发出“离开此网站？”警报。选项卡内的所有内容（包括开发者的 JavaScript 代码）都由渲染器进程处理，因此，当新的导航请求出现时，浏览器进程必须与当前渲染器进程进行核对。
![navitodifferentpage.png](/tech/web/browser/navitodifferentpage.png)

如果导航是从**渲染进程**启动的（例如用户单击链接或客户端 JavaScript 已运行window.location =“ https://newsite.com”），则渲染进程首先检查 `beforeunload` 处理程序。然后，它经历与浏览器过程启动的导航相同的过程。
唯一的区别是导航请求从渲染进程开始向浏览器进程启动。

当新的导航与当前渲染的站点不在同一个站点上时，将调用**一个单独的渲染过程**来处理新的导航，而当前的渲染过程将保留来处理诸如卸载之类的事件。
有关更多信息，请参见 **页面生命周期状态概**[^4] 述以及如何使用 **页面生命周期 API**[^5] 挂载事件。

![asyncunload&navi.png](/tech/web/browser/asyncunload&navi.png)


[^1]: [Same-origin_policy - Web | MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy)

[^2]: Meltdown and Spectre：一个进程可以使用该漏洞读取（最坏的情况下）任意内存，包括不属于该进程的内存。[meltdown-spectre | developers.google](https://developers.google.com/web/updates/2018/02/meltdown-spectre)

[^3]: [跨源读取拦截 Cross Origin Read Blocking（CORB）](https://www.chromium.org/Home/chromium-security/corb-for-developers)

[^4]: [Page Lifecycle API  |  Web  |  Google Developers | Overview of Page Lifecycle states and events](https://developers.google.com/web/updates/2018/07/page-lifecycle-api#overview_of_page_lifecycle_states_and_events) 

[^5]: [Page Lifecycle API  |  Web  |  Google Developers](https://developers.google.com/web/updates/2018/07/page-lifecycle-api)