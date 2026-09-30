---
title: Modern Browsers - A Deeper Look
description: Our reading of Mariko Kosaka's Inside look at modern web browser
published: true
date: 2026-09-30T13:35:19.000Z
tags: browser
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/technology/web/inside-look-at-browser.md)

# Preface

This article can be read as a translation of [*Inside look at modern web browser*](https://developers.google.com/web/updates/2018/09/inside-browser-part1) by Chrome developer Mariko Kosaka.
Translations and mash-ups of varying versions and quality already exist online. The author doesn't do mash-ups, and only wants to bring in their own understanding to present a smoother, plainer version, with official links added for some technical terms, for reference and understanding.


> In this 4-part article, we'll explore the Chrome browser from the shallow end to the deep, from its high-level architecture down to the details of the rendering pipeline. If you want to know how a browser turns your code into a working website, or you aren't sure why certain techniques get recommended for improving performance, this series is for you.


# Part 1

Core computing terms, Chrome's multi-process architecture

## At the core of a computer are the CPU and the GPU

### Central processing unit - CPU
The central processing unit, the CPU, can be thought of as the computer's brain.
A CPU core can handle many different tasks one after another as they come in.
In the past, most CPUs were a single chip. A core is like a CPU living in the same chip.

### Graphics processing unit - GPU
The graphics processing unit, the GPU, is different from the CPU: the GPU is good at handling simple tasks, but **across** many cores at the same time.

![computerlevelstruc.png](/tech/web/browser/computerlevelstruc.png)
(The three layers of computer architecture. **Machine hardware** at the bottom, the **operating system** in the middle, **applications** on top.)

## Processes and threads

A process can be described as an application's executing program.
A thread is what lives inside a process and carries out any part of its process's program.

![process&thread.png](/tech/web/browser/process&thread.png)
(The process as a bounding box, with threads as abstract fish swimming inside it.)

![process&threadinmemory.png](/tech/web/browser/process&threadinmemory.png)
When an application starts, a process is created. The program may create threads to help it work, but that's optional.
The operating system gives the process a block of memory, and all of the application's state is kept in that private memory space.
When the application is closed, the process goes away too, and the operating system frees the memory.

![ipc.png](/tech/web/browser/ipc.png)
A process can ask the operating system to start another process to run a different task. When that happens, a different part of memory is allocated to the new process.
If two processes need to talk, they can do it with **inter-process communication (IPC)**.
Many applications work this way, so if a worker process stops responding, it can be restarted without stopping the other processes that run different parts of the application.

## Browser architecture

So how is a web browser built out of processes and threads?
It could be one process with many different threads, or many processes, each with a few threads, communicating over IPC.
The important thing to note here is that these **different architectures are implementation details**. Chrome is the example here.

![browserstruc.png](/tech/web/browser/browserstruc.png)
At the top is the **browser process**, which coordinates with the other processes that take care of different parts of the application.
For the **renderer process**, multiple processes are created and assigned to each tab.
(Until recently, Chrome gave each tab a process whenever it could. Now it tries to give each site its own process, iframes included.)

## What does each process manage?

![kindsofprocesses.png](/tech/web/browser/kindsofprocesses.png)

- **Browser process**
Controls the "chrome" part of the application, including the address bar, bookmarks, and the back and forward buttons. It also handles the invisible, privileged parts of a web browser, such as network requests and file access.

- **Renderer process**
Controls everything inside the tab where a website is displayed.

- **Plugin process**
Controls any plugins the website uses, such as Flash.

- **GPU process**
Handles GPU tasks in isolation from the other processes. Because the GPU handles requests from multiple applications and draws them on the same surface, it's split out into its own process.

## Advantages of Chrome's multi-process design

- **Not affected when another tab freezes**
Say you have 3 tabs open, each run by its own renderer process. If one tab becomes unresponsive, you can close it and move on while the other tabs stay alive.
If all the tabs ran in one process, then when one tab became unresponsive, all of them would.

- **Security and sandboxing** (memory protection, access control)
Since the operating system provides a way to restrict a process's privileges, the browser can keep certain processes away from certain features. For example, Chrome restricts arbitrary file access for processes that handle arbitrary user input, like the renderer process.

- **Memory-saving controls**
Because processes have their own private memory space, they often contain copies of common infrastructure (V8, for example). That means more memory use, since they can't share these the way threads in the same process could.
To save memory, Chrome limits how many processes it can start. The limit depends on the device's memory and CPU power, but once Chrome hits it, it starts running multiple tabs from the same site in the same process.

## Saving more memory - servicification in Chrome

The same approach is applied to the browser process.
Chrome is going through architecture changes to run each part of the browser program as a service, so it can easily be split into different processes or combined into one.

The general idea is that when Chrome runs on powerful hardware, it may split each service into a different process for more stability, but on a resource-constrained device, Chrome consolidates the services into one process to save memory.

## Per-frame renderer processes - site isolation

Site isolation runs a separate renderer process for each cross-site iframe.
The "same-origin policy"[^1] is the core security model of the web. It makes sure one site can't access another site's data without consent. Getting around this policy is a main target of security attacks.
**Process isolation** is the most effective way to separate sites. With Meltdown and Spectre[^2], it became even more obvious that we need processes to separate sites. Since Chrome 67, "site isolation" is on by default on desktop, and every cross-site iframe in a tab gets a separate renderer process.


# Part 2

Core computing terms, Chrome's multi-process architecture

## What happens in navigation

Let's start with a simple web browsing use case: you type a URL into the browser, then the browser fetches data from the internet and displays a page. (The move you can't dodge in an interview.)
This section focuses on the part where the user requests a site and the browser gets ready to render the page (also called navigation).

## It starts with the browser process

From Part 1 we know that everything outside the tab is handled by the **browser process**.

The browser process has the following threads:
- The **UI thread** (draws the browser's buttons and input fields)
- The **network thread** (deals with the network stack to receive data from the internet)
- The **storage thread** (controls access to files), and so on

When you type a URL into the address bar, your input is handled by the browser process's UI thread.
![navigation.png](/tech/web/browser/navigation.png)

## A simple navigation

### Step 1: handling input

When the user starts typing into the address bar, the first thing the **UI thread** asks is: "Is this a search query or a URL?".
In Chrome, the **address bar** is also a **search input field**, so the UI thread needs to parse the input and decide whether to send you to a search engine or to the site you requested.
- Search query: sent to the search engine
- URL: request the site at that URL

### Step 2: starting navigation

When the user hits Enter, the **UI thread** makes a network call to get the site's content. A loading icon shows up in the corner of the tab, and the **network thread** goes through the appropriate protocols for the URL request, doing the lookup (e.g. DNS) and setting up the connection (e.g. TLS).
![startnavi.png](/tech/web/browser/startnavi.png)
(The UI thread talks to the network thread to navigate to mysite.com)

At this point, the network thread may get a redirect status code from the server, such as HTTP 301. In that case, the network thread tells the UI thread that the server is asking for a redirect. Then another URL request is started.

### Step 3: reading the response

#### 3.1 Working out the file's MIME type
Once the response body (the payload) starts coming in, the network thread looks at the first few bytes of the stream if needed. The response's `Content-Type` header should say what type of data it is, but since it can be missing or wrong, [MIME Type sniffing](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types) (in Chinese) is done here.

Every browser does different things in different situations. Because this operation raises some security issues, some MIME types stand for executable content and some for non-executable content. The browser can set `X-Content-Type-Options` through the request header `Content-Type` to block MIME sniffing.

![filemimetype.png](/tech/web/browser/filemimetype.png)

#### 3.2 Handling different MIME files
If the response is an HTML file, the next step is to pass the data to the **renderer process**, but if it's a zip file or some other file, that means it's a download request, so they need to pass the data to the download manager.

#### 3.3 Security checks

- Malicious list check: if the domain and the response data match a list of malicious sites, the network thread raises an alert to show a warning page.
- Cross-origin read check: a Cross Origin Read Blocking[^3] check, to make sure sensitive cross-site data doesn't make it into the renderer process.
![securitycheckbynetworkthread.png](/tech/web/browser/securitycheckbynetworkthread.png)
(The network thread checks whether the response data is HTML from a safe site)

### Step 4: finding a renderer process

Once all the checks are done and the network thread is confident the browser should navigate to the requested site, the network thread tells the UI thread that the data is ready.
The UI thread then finds a **renderer process** to render the web page.
![findrendererprocess.png](/tech/web/browser/findrendererprocess.png)

> **Optimization**
Since a network request can take a few hundred milliseconds to get a response, an optimization is applied to speed this process up. When the UI thread sends the URL request to the network thread in step 2, it already knows which site they're navigating to.
**The UI thread tries to proactively find or start a renderer process in parallel with the network request.** 
This way, if everything goes as expected, a renderer process is already on standby by the time the network thread receives the data. If the navigation redirects cross-site, this standby process might not be used, and in that case a different process may be needed.

### Step 5: committing the navigation

Now that the data and the renderer process are ready, an IPC is sent from the **browser process** to the **renderer process** to commit the navigation.
It also passes along the data stream, so the renderer process can keep receiving HTML data.
Once the browser process hears the commit confirmation from the renderer process, the navigation is complete and the document loading phase begins.

At this point the address bar is updated, and the security indicator and the site settings UI reflect the site information of the new page.
The tab's session history is updated, so the back/forward buttons will step through the sites just navigated to.
To make it easy to restore the tab/session when you close a tab or window, the session history is stored on disk.

![commitnavigation.png](/tech/web/browser/commitnavigation.png)

### Extra step: initial load complete

After the navigation is committed, the renderer process keeps loading resources and renders the page.
Once the renderer process "finishes" rendering, it sends an IPC back to the browser process (this is after all the `onload` events have fired on all the frames in the page and finished running). At this point, the UI thread stops the loading icon on the tab.
("Finishes" here, because client-side JavaScript can still load extra resources and render new views after this.)

![pageloaded.png](/tech/web/browser/pageloaded.png)

## Navigating to a different site

What happens if the user puts a different URL into the address bar again?
Well, the browser process goes through the same steps to navigate to the different site.
But before doing that, it needs to check whether the currently rendered site cares about the `beforeunload` event.

beforeunload can raise a "Leave this site?" alert when you try to navigate away or close the tab. Everything inside the tab, including the developer's JavaScript code, is handled by the renderer process, so when a new navigation request comes in, the browser process has to check with the current renderer process.
![navitodifferentpage.png](/tech/web/browser/navitodifferentpage.png)

If the navigation was started from the **renderer process** (for example, the user clicked a link, or client-side JavaScript ran window.location = "https://newsite.com"), the renderer process first checks its `beforeunload` handlers. Then it goes through the same process as a navigation started by the browser process.
The only difference is that the navigation request is kicked off from the renderer process to the browser process.

When the new navigation is to a different site from the currently rendered one, **a separate renderer process** is called to handle the new navigation, while the current renderer process is kept around to handle events like unload.
For more, see the **overview of page lifecycle states**[^4] and how to hook into events with the **Page Lifecycle API**[^5].

![asyncunload&navi.png](/tech/web/browser/asyncunload&navi.png)


[^1]: [Same-origin_policy - Web | MDN](https://developer.mozilla.org/zh-CN/docs/Web/Security/Same-origin_policy) (in Chinese)

[^2]: Meltdown and Spectre: a process can use these vulnerabilities to read (in the worst case) arbitrary memory, including memory that doesn't belong to that process. [meltdown-spectre | developers.google](https://developers.google.com/web/updates/2018/02/meltdown-spectre)

[^3]: [Cross Origin Read Blocking (CORB)](https://www.chromium.org/Home/chromium-security/corb-for-developers)

[^4]: [Page Lifecycle API  |  Web  |  Google Developers | Overview of Page Lifecycle states and events](https://developers.google.com/web/updates/2018/07/page-lifecycle-api#overview_of_page_lifecycle_states_and_events) 

[^5]: [Page Lifecycle API  |  Web  |  Google Developers](https://developers.google.com/web/updates/2018/07/page-lifecycle-api)