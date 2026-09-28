---
title: AutoSSH
description: Reverse proxying, and a fix for SSH connections that time out and drop
published: true
date: 2026-09-28T11:19:16.000Z
tags: network, autossh
editor: markdown
dateCreated: 2026-09-28T11:19:16.000Z
---

**English** · [中文](/zh/technology/network/autossh.md)

# Scenarios

Say we've built some test cases and CI services on the company intranet to verify the code developers commit, but after getting home we find something that needs changing. How do we reach the intranet then, and push our local changes into the company's pipeline?

Or say you bought a few Raspberry Pis and set up lots of handy everyday services on your home LAN. Then, while you're at work, the experience you've built up tells you a script running on one of the machines at home is wrong. How do you get into your home LAN and reach that server?

Both scenarios can be handled with autossh[^1], which binds ports from the public internet to local ones and works as a reverse proxy. It fits any case where traffic from an online VPS should be forwarded to some local service.

# Why not just use SSH

SSH provides `-R` and `-L` to bind a port on an online node to any local port. But if you've ever used SSH to reach a remote host, you'll have noticed that a long idle stretch makes the session time out and the connection drop. For a one-off session that's no big deal, but for port binding and reverse proxying run as a service, it just isn't stable.

AutoSSH solves this. It uses a remote echo service to monitor the traffic of the SSH session, and once the traffic stops and the SSH session breaks, AutoSSH restarts that session.

> Automatically restart SSH sessions and tunnels.

# Installation

## APT

On Ubuntu you can install it straight from the `apt` package manager.

```bash
apt-get install -y autossh
```

## Manual installation

On operating systems without `apt` or `snap` package management, manual installation is recommended.

Head over to https://www.harding.motd.ca/autossh/ to download it.

```bash
gunzip -c autossh-1.4e.tgz | tar xvf -
cd autossh-1.4e
./configure
make
# copy binary to where you wish it, or "make install" will install it under /usr/local by default.
# examine autossh.host for example wrapper script and options
```

# Usage

The `autossh` command takes the same arguments as `ssh` does for reverse-proxy port binding: `-f` runs the command in the background, `-C` compresses the transferred data, and `-N` disables remote commands. The `-M` flag additionally provides a port that lets the public host get information about the local machine, so that the remote host can re-establish the connection when the SSH tunnel breaks.

```
autossh -M [remote_port] -fCNR [port]:localhost:[port] [user_name]@[ip_address]
```

Assuming you already have a remote host `foobar.com`, with the format above we can bind port 3000 on that remote host to port 25565 on the local machine, where a Minecraft server is running.

```bash
autossh -M 3001 -fCNR 3000:localhost:25565 root@foobar.com
```

Now every request that reaches `foobar.com:3000` is forwarded to the Minecraft server at `localhost:25565` inside the LAN, and port 3001 on the remote host is opened to watch the tunnel. That gives us a reverse proxy while keeping the session traffic forwarded steadily.

# Running as a daemon

The service from the previous section can be controlled more effectively with a process manager.

## PM2

> This section still needs work...
{.is-warning}

## init.d

Here we take port redirection as the example: reverse-proxy a public VPS to a program running locally, and add the job to init.d, the Linux daemon system.

`vi /etc/init/autossh.conf`

```bash
#!/bin/bash

while true
do
autossh -M [remote_port] -fCNR [port]:localhost:[port] [user_name]@[ip_address]
done
```

[^1]: [autossh](https://www.harding.motd.ca/autossh/)