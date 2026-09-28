---
title: AutoSSH
description: 提供反向代理服务并解决 SSH 超时断开连接的问题
published: true
date: 2021-01-14T07:36:26.627Z
tags: network, autossh
editor: markdown
dateCreated: 2020-11-30T15:00:22.007Z
---

# 场景

如果我们在公司内网中构建了一些测试用例和持续集成服务验证开发提交的代码，但是在下班回家后却发现有内容需要修改，这个时候如何才能访问到内网，提交本地修改的代码到公司内网的流水线中呢？

又或者你购买了数台树莓派，在家中的局域网中搭建了很多日常实用的服务，当你去公司上班的时候，工作的积累的经验让你意识到家中某台服务运行的脚本有误，这个时候如何才能访问到家中的局域网中运行的那台服务器呢？

以上两种场景可以借助 autossh[^1] 完成从公网到本地的端口绑定和反向代理，适用于转发某台线上 VPS 流量到任意本地服务的场景。

# 为什么不使用 SSH

SSH 提供了 `-R` 和 `-L` 可以绑定某台线上节点的端口到本地任意端口，但是如果你使用过 SSH 访问远程主机就会发现，长时间无操作会导致会话的超时和连接中断，这对于单次会话场景是没有大碍的，但是作为服务化的端口绑定和反向代理就不稳定了。

而 AutoSSH 解决了这一问题，它会使用远程回显（remote echo service）监控 SSH 会话的流量，一旦流量中止导致 SSH 会话中断，AutoSSH 会重启该会话。

> Automatically restart SSH sessions and tunnels.

# 安装

## APT

在 Ubuntu 操作系统下可以直接使用 `apt` 包管理安装。

```bash
apt-get install -y autossh
```

## 手动安装

在某些不支持 `apt` 或 `snap` 包管理的操作系统中，推荐手动安装。

前往 https://www.harding.motd.ca/autossh/ 完成下载。

```bash
gunzip -c autossh-1.4e.tgz | tar xvf -
cd autossh-1.4e
./configure
make
# copy binary to where you wish it, or "make install" will install it under /usr/local by default.
# examine autossh.host for example wrapper script and options
```

# 使用

`autossh` 命令与 `ssh` 反向代理端口绑定参数规格一致，`-f` 表示后台执行该命令，`-C` 压缩传输数据，`-N` 禁止远程指令。而 `-M` 参数则会额外提供一个端口使得公网主机获取本地机器信息，以便于在 SSH 隧道中断时让远端主机重新建立连接。

```
autossh -M [remote_port] -fCNR [port]:localhost:[port] [user_name]@[ip_address]
```

假定你已经拥有一台远端主机 `foobar.com`，使用上面的格式，我们可以把该远程主机的 3000 端口绑定到本地运行着 Minecraft 服务的 25565 端口。

```bash
autossh -M 3001 -fCNR 3000:localhost:25565 root@foobar.com
```

这样所有访问到 `foobar.com:3000` 的请求都会被转发到内网本地 `localhost:25565` 的 Minecraft 服务器中，并开启远程主机 3001 端口监听隧道情况，实现了反向代理的同时保证了会话流量的稳定转发。

# 守护进程

上一节的服务可以使用进程管理器进行更有效的控制。

## PM2

> 该小节内容需要完善...
{.is-warning}

## init.d

这里我们以端口重定向为例，将公网的某台 VPS 反向代理到本地某个运行中的程序，并将该任务添加到 Linux 守护进程 init.d 中。

`vi /etc/init/autossh.conf`

```bash
#!/bin/bash

while true
do
autossh -M [remote_port] -fCNR [port]:localhost:[port] [user_name]@[ip_address]
done
```

[^1]: [autossh](https://www.harding.motd.ca/autossh/)