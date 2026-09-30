---
title: Docker in Practice and Principle
description: Building microservices with containerization
published: true
date: 2026-09-30T13:35:19.000Z
tags: docker
editor: markdown
dateCreated: 2026-09-30T13:35:19.000Z
---

**English** · [中文](/zh/technology/saas/docker.md)

# Using Docker

## What is Docker

Docker is an [open platform for developing, shipping and deploying applications](https://docs.docker.com/get-started/overview/)[^1]. You can use it to run services or a program script, and it can package the environment a program depends on into an image that you save and run again next time.

It can also keep several services isolated for you. Say you run an Nginx server built on top of Ubuntu, or a CentOS system with a Python environment running a Flask service: you can package each of them into its own image and run them as independent services on the same host.

This is a bit like a virtual machine (though at heart they're different): other operating systems run on your host, and some services or scripts run inside those systems.

## Installation

To understand what Docker is for and what it can do more quickly, let's get the installation done first; Docker can be installed on many platforms[^2].

Since containers are mostly used in server environments, we pick Linux (Ubuntu) as the operating system to install Docker on here; for other Linux distributions, you can [pick yours on the official site](https://docs.docker.com/engine/install/#server)[^3].

On Ubuntu we use `apt-get` as the install tool and fetch Docker's binaries directly.

If a command below stops on a permission problem, add `sudo` to raise your privileges and try again.

### Before installing

1. Update the `apt` package index and install the prerequisites.
```bash
$ apt-get update

$ apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

2. Add Docker's official GPG key.
```bash
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```
Check that the fingerprint key is right by searching for `9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88` with the command below and seeing whether it matches.
```bash
$ apt-key fingerprint 0EBFCD88

pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

3. Set up the repository
```bash
$ add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

### Installing Docker

Once the preparation above is done, run the command below to install Docker for real.

```bash
$ apt-get update
$ apt-get install docker-ce docker-ce-cli containerd.io
```

### Checking the version

After a successful install, first check its version (this step doubles as proof that the command line works).

```bash
$ docker --version
Docker version 19.03.13, build 4484c46d9d
```

## Concepts

With Docker installed, before we start using it we need to get to know a few players, and get a rough picture of the tool's structure[^4].

### Docker Engine

Docker uses a client-server architecture, which means it provides a command-line client (**CLI**); most of the time we talk to Docker's background process (the **docker daemon**) by sending network requests (a **RESTful API**) through this CLI. The client, the API and the server program together are called the Docker Engine[^5].

Here's the official diagram of the Docker Engine.

![engine-components-flow.png](/technology/saas/docker/engine-components-flow.png)

You'll notice the diagram also breaks out the concepts of **image**, **container**, **data volumes** and **network**.

If you've used virtual machines before, you've probably downloaded a VM image (an iso file) from somewhere to your own machine (the Host), then loaded it into a VM tool like VMware and run it. In that process you already dealt with several parts that each have their own job, such as virtual network cards and virtual storage, and the concepts above exist in Docker too.

### Docker's architecture

We can understand Docker's architecture through how it's used.

As with virtual machines, imagine you need an Ubuntu operating system environment. Then you need to get an image (**image**), so first you have to download this image from somewhere. In the Docker ecosystem, that somewhere is the **Registry** (a private or public image storage service that holds images, official and community-contributed, built on different operating systems and different environment dependencies).

Next you need to run this image. With a virtual machine that's easy to understand: VMware installs the system from the iso and runs it, and then you're in that system's graphical interface. In Docker, though, this running process becomes purer and simpler: you may only be running one background program (not many), and Docker calls this running state a container (**container**), that is, an instance of an image.

<img src="/technology/saas/docker/architecture.svg" alt="architecture.svg" width="90%">

In the diagram above, `docker pull` and `docker run` correspond to the two steps of downloading an image from the Registry and running the matching operating system environment from the image; `docker build` is the command for making images, and we'll get into its details in the Images section.

## Basic usage

Next we'll do some basic operations with the Docker command-line client.

### Running a container

1. First, run a container from an image called `ubuntu`.
```bash
$ docker run -i -t ubuntu /bin/bash

Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
da7391352a9b: Pull complete
14428a6d4bcd: Pull complete
2c2d948710f2: Pull complete
Digest: sha256:c95a8e48bf88e9849f3e0f723d9f49fa12c5a00cfc6e60d2bc99d87555295e4c
Status: Downloaded newer image for ubuntu:latest
root@8e4765ca5c0e:/#
```
`docker run` is Docker's main command for running containers. The `-i` flag enters interactive command-line mode and keeps STDIN open to listen for the user's further commands, and the `-t` flag makes Docker allocate a pseudo-TTY for the user, used together with `/bin/bash` inside the container.

This step first downloads the image called `ubuntu` to the local machine. Since we didn't specify a version, it defaults to downloading the `ubuntu:latest` image from the official Registry (Docker Hub). Once the download is done, the container runs with the `/bin/bash` command; when it's running, the command prompt changes: our shell environment has switched from the host to the container's Bash environment.

2. List the images downloaded locally.
Open a new Terminal, and with the command below we can see the `ubuntu:latest` image that has been downloaded locally.
```bash
$ docker image ls

REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
ubuntu                                          latest              f643c72bc252        2 weeks ago         72.9MB
```

3. List the running containers.
With the command below we can see the list of containers that currently exist. Docker gives each container it creates its own `CONTAINER ID` as an identifier, which is also the container's `hostname`.
```bash
$ docker ps --all

CONTAINER ID        IMAGE                                 COMMAND                  CREATED             STATUS                     PORTS                                                                                            NAMES
7cc2d9f71570        ubuntu                                "/bin/bash"              3 seconds ago       Up 2 seconds
```
Note that at this point our container's status is `Up`. If we now exit from the first Terminal, the command prompt switches back to the host's shell, and because the interaction ends, the container moves to the `Exited` status as well.

### Other useful commands

For running containers, you can also use the basic commands below.

- Stop a running container.
  ```
  $ docker container stop <container_name|container_id>
  ```
  
- Remove a stopped container
  ```
  $ docker container rm <container_name|container_id>
  ```
  Note that containers usually have to be stopped before they can be removed; if you want to remove one directly, you can force it with `docker container rm -f <container_name|container_id>`.
  
- Send extra commands to a running container
  For example, open a separate interactive `bash` shell in a running container.
  ```
  $ docker exec -it <container_name|container_id> bash
  ```
  Or you can set an extra environment variable for a running container.
  ```
  $ docker exec -it -e VAR=1 <contaienr_name|container_id> bash
  ```

### Containers and virtual machines

So far we've been using the idea of a virtual machine to understand Docker, so what's the difference between the two[^6]?

In short, container technology is more lightweight. Suppose your host is Linux: different containers share the same Linux kernel, and each container runs in its own process, kept isolated from other containers and from the host's processes.

A virtual machine, on the other hand, runs a complete operating system. You can think of it as a user who has been granted all of the operating system's resources, using the resources the host provides to implement an independent operating system. That means programs you run in a virtual machine make the host carry more overhead.

<img src="/technology/saas/docker/container@2x.png" alt="container@2x.png" width="45%">

## Images

In the last section we used Docker to run a container with an Ubuntu-based operating system environment, and with the `/bin/bash` command got an interactive Ubuntu shell inside the container.

But that's far from enough. At work we're sure to run into cases that need customized containers. Say a team wants separate containers based on Express.js and SpringBoot: a plain Ubuntu system certainly won't do, since team members would also have to install the Node.js and Java environments and do a lot of business-specific setup, so customizing the container's image becomes a must.

### Building an image

We use Docker's official example[^7] to show how to build your own image. First, get the official sample repository with Git.

```bash
$ git clone https://github.com/dockersamples/node-bulletin-board
$ cd node-bulletin-board/bulletin-board-app
```

You can see there's a `Dockerfile` in this directory; this file describes how to create a new image.

```docker
$ cat Dockerfile

FROM node:current-slim

WORKDIR /usr/src/app
COPY package.json .
RUN npm install

EXPOSE 8080
CMD [ "npm", "start" ]

COPY . .
```

Let's go through what makes up the image, step by step.

1. `FROM node:current-slim`
  This `Dockerfile` uses the `node:current-slim` image as its base image, which means the image being built is entirely a customization layered on top of `node:current-slim` (so anyone can build on a standard image from the community or the official source, without installing the base environment all over again).

2. `WORKDIR /usr/src/app`
  The image's default working directory is `/usr/src/app`, which means once the image runs as a container, the container automatically goes into this path, so developers can keep some config files in a directory they chose themselves.
  
3. `COPY package.json .`
  Copy the `package.json` file from `/path/to/node-bulletin-board/bulletin-board-app` on the host into `/usr/src/app` in the image.
  
4. `RUN npm install`
  Run NPM, Node.js's package manager, to install all the Node.js project dependencies listed in `package.json`.
  
5. `EXPOSE 8080`
  After the dependencies are installed, expose the image's port 8080, which means once the image runs as a container, the host can talk to the container through port 8080, and so can reach the service running on that port in the container.
  
6. `CMD [ "npm", "start" ]`
  Set the default command for when the image runs as a container. In the last section we used `/bin/bash` as the command after entering the container running the `ubuntu:latest` image. Here, the image we built runs `npm start` by default once it's a container; this command looks for the current Node.js project's start script described in `package.json`, and looking in `package.json` shows that it's `node server.js`.
  
7. `COPY . .`
  This command copies all the Node.js source files under `/path/to/node-bulletin-board/bulletin-board-app` into `/usr/src/app` in the image.
  
Once we understand the contents of the `Dockerfile`, we use the command below to build the image for real.

```bash
$ docker build --tag bulletinboard:1.0 .

Sending build context to Docker daemon  45.57kB
Step 1/7 : FROM node:current-slim
 ---> 9ac9e9f30b2c
Step 2/7 : WORKDIR /usr/src/app
 ---> Running in 266813564f95
Removing intermediate container 266813564f95
 ---> 20d33d49fc42
Step 3/7 : COPY package.json .
 ---> 852ab2e63c38
Step 4/7 : RUN npm install
 ---> Running in d4cfe37404de
...
```
If the build succeeds, `docker image ls` shows an image called `bulletinboard:1.0` in the list.

Finally, create a container with the command below.

```bash
$ docker run --publish 8000:8080 --detach --name bb bulletinboard:1.0
```

`--publish 8000:8080` maps port 8000 on the host to port 8080 in the container, which means that accessing the host's port 8000 over the network, in the form `<protocol>://<ip>:<port>`, is the same as reaching the service on the container's port 8080 directly (in this case, the Node.js service started by the `node server.js` above).

`--detach` means running in the background, rather than staying interactively in the Terminal foreground waiting for user input like the `/bin/bash` above.

`--name bb` names the new container `bb`.

Last comes the name of the image the container depends on, in the format `<image name>:<image version>`.

Once the command above succeeds, a Node.js project based on the `node:current-slim` environment can be reached through port 8000 on the host.

### Pushing an image to Docker Hub

Once your custom image is built, you can share it by uploading it to Docker Hub, so other developers can get your image with `docker pull`; this is a bit like how the open-source community on Github works.

First create a [Docker Hub account](https://hub.docker.com/signup) and fill in your `Docker ID`. After logging in, first use `docker tag` locally to tag the image you're going to upload (because Docker Hub identifies images in different users' namespaces in the format `<Your Docker ID>/<Repository Name>:<tag>`).
```bash
$ docker tag bulletinboard:1.0 <Your Docker ID>/bulletinboard:1.0
```

Then you can push it to Docker Hub.
```bash
$ docker push <Your Docker ID>/bulletinboard:1.0
```

## Volumes

With the steps above you can build images of custom services with Docker and share them on Docker Hub. You've contributed a Node.js-stack image to your team; from now on, when the service gets updated, you only need to change the project source and rebuild the image, and hand it to the ops folks to write a `docker run` script for one-click deployment. Everything looks just fine.

But the other backend folks get a headache when they see this. The services they look after aren't only business services but also storage services, and some slightly more complex business cases also use caches and non-relational databases. If these services can be containerized as images, who manages the most important part, the data?

Docker offers two ways to persist the data services produce: volumes and bind mounts. The former is a bit like external storage (say you plug a portable hard drive into a computer to copy data; even if that computer later has problems, the data on the drive won't be lost). The latter relies on the host's directory structure and only adds a layer of mapping (like making a shortcut to some path).

volumes are what the official docs recommend more[^8], because they're managed entirely by Docker itself, with no need to lean on the host's directories, and volumes are shared between containers, with advantages for data migration, backup, restore, remote cloud storage, data encryption and cross-platform handling.

The biggest reason to recommend them is that volumes are implemented **independently of the container lifecycle**. In other words, a container that uses volumes doesn't get bigger because of them, and destroying and rebuilding the container itself doesn't affect volumes that already exist.

The diagram below briefly describes the difference between volumes and bind mounts.

![types-of-mounts-volume.png](/technology/saas/docker/types-of-mounts-volume.png)

### Using volumes for storage

Since volumes have so many advantages, let's first look at what they do and how to use them.

1. Create volumes
  To create a volume you can simply give it a name; below we create a storage volume called `my-vol`.
  ```bash
  $ docker volume create my-vol
  ```
  
2. List all volumes
  ```bash
  $ docker volume ls

local               my-vol
  ```
  
3. Inspect a volume in detail
  ```bash
  $ docker volume inspect my-vol
[
    {
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/my-vol/_data",
        "Name": "my-vol",
        "Options": {},
        "Scope": "local"
    }
]
  ```
  
4. Remove a volume
  ```bash
  $ docker volume rm my-vol
  ```
  
### Using volumes with containers

Normally, we use volumes together with running containers.

```bash
$ docker run -d \
  --name devtest \
  -v myvol2:/app \
  nginx:latest
```
The command above uses the `nginx:latest` image, creates a container called `devtest` in the background, and creates a volume called `myvol2`, mapped to the `/app` directory in the container.

Note here that the `-v` flag has several meanings[^9]. If the `/app` directory in the container created by the command above has nothing in it to begin with, `myvol2` is mapped onto that path in real time, which means any data produced under `/app` inside the `devtest` container after it's created is synced to `myvol2` in real time.

Of course, you'll think of the other case: if `/app` already has content (some files and directories) when the `devtest` container is created, that content is copied straight into the volume called `myvol2`.

Finally you may wonder: what if `myvol2` already exists rather than being newly created, and already has content? It's actually simple. Docker doesn't throw away the data that already exists in the volume; if `devtest` is a newly created container and there's already content under `/app`, then `-v myvol2:/app` overwrites everything under `/app` in the new `devtest` with the data that already exists in `myvol2`.

### Backup

From using volumes with containers above, we can see that Docker stores volumes independently, so no data is lost when any container associated with them is destroyed.

In production, service data is usually backed up redundantly in several places with separate backup scripts, and data in Docker volumes can likewise be packed up and copied out on its own.

```bash
$ docker run -v /dbdata --name dbstore ubuntu /bin/bash
```
The command above creates an anonymous volume mapped to all the files under the `/dbdata` path in the container `dbstore`; next we'll back up the content of this directory.

```bash
$ docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```
The command above creates a new container; `--rm` removes the container automatically after it exits, instead of keeping it in the `Exited` status. This container shares the same volume from the container called `dbstore`: `--volumes-from` mounts the `dbstore` container's mount point `/dbdata` at the same path in the new container.

`-v $(pwd):/backup` maps the host's current path to the `/backup` directory in the new container. Together with the command run in the new container at the end, `tar cvf /backup/backup.tar /dbdata` packs the data shared into this container's `/dbdata` into the `/backup` directory as `backup.tar`. Because `/backup` is mapped to the host's current directory, the `/backup/backup.tar` archive ends up in the host's current directory, which completes the backup of the volume from the `dbstore` container.

### Restore

Following on from the last subsection, we now restore the backup data `backup.tar` in a new container.

```bash
$ docker run -v /dbdata --name dbstore2 ubuntu /bin/bash
```
The command above creates a container called `dbstore2` from the `ubuntu` image, and maps `dbdata` into a new anonymous volume.

```bash
$ docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```
Next, create a new container that's cleaned up once it exits. This container shares `dbstore2`'s mount point `/dbdata` and maps the host's current path to `/backup` in the container; because of that mapping, the `backup.tar` in the host's current directory shows up directly under `/backup` in this container.

Finally, this container runs `bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"`, which `cd`s into `/dbdata` and then extracts everything in the `/backup/backup.tar` archive. That way the contents of the backup file `backup.tar`, originally in the host's current directory, are restored into this container's `/dbdata` directory, and because this container shares a volume with the container `dbstore2`, everything extracted into this container's `/dbdata` is shared into `dbstore2`'s `/dbdata` directory.

## Networking

In the last section we used volumes to persist what's produced inside containers, and the data storage part of real scenarios seems solved. But in production the actual service and the database are usually deployed in different containers, so letting the business server (such as Flask or Express.js) talk to the container that hosts the database becomes an essential feature.

Docker, of course, provides a complete networking system for communication between containers.

### Network drivers

Docker manages its networking through several different drivers[^10].

- bridge
  The default network driver, usually used for communication between standalone containers.
  
- host
  The container uses the host's network environment directly. Recommended when you don't want the container's network isolated from the host's network, but do want process, storage, memory and other system resources isolated from the host.

- overlay
  Used for communication between container clusters, and also between standalone containers. The overlay driver is usually used for large-scale microservice clusters and communication between multiple docker daemons.

- macvlan
  This driver can assign a MAC address to a container, so the container shows up on the network as a real physical device.

- none
  Disables the container's networking; usually used together with a custom network driver.
  
- Network Plugins[^11]
  You can configure network plugins for containers yourself through Docker Hub or third-party providers.

### Container-to-container communication in practice

Now let's get familiar with common network operations, and set up communication between containers with the default `bridge` driver[^12].

1. List the networks that already exist
```bash
$ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
17e324f45964        bridge              bridge              local
6ed54d316334        host                host                local
7092879f2cc8        none                null                local
```

2. Run two `alpine` containers
```bash
$ docker run -dit --name alpine1 alpine ash

$ docker run -dit --name alpine2 alpine ash
```
`alpine` is a special Linux operating system image customized by the community. Compared with a typical CentOS or Ubuntu, it keeps little more than the kernel and the usable core commands, so it's very lean. `ash` is the shell in `alpine`, similar in function to `bash`.

Running with `-dit` puts the interactive `ash` shell, waiting for input, straight into the background.

3. See which containers are connected to the bridge network
```bash
$ docker network inspect bridge

[
    {
        "Name": "bridge",
        "Id": "17e324f459648a9baaea32b248d3884da102dde19396c25b30ec800068ce6b10",
        "Created": "2017-06-22T20:27:43.826654485Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "602dbf1edc81813304b6cf0a647e65333dc6fe6ee6ed572dc0f686a3307c6a2c": {
                "Name": "alpine2",
                "EndpointID": "03b6aafb7ca4d7e531e292901b43719c0e34cc7eef565b38a6bf84acf50f38cd",
                "MacAddress": "02:42:ac:11:00:03",
                "IPv4Address": "172.17.0.3/16",
                "IPv6Address": ""
            },
            "da33b7aa74b0bf3bda3ebd502d404320ca112a268aafe05b4851d1e3312ed168": {
                "Name": "alpine1",
                "EndpointID": "46c044a645d6afc42ddd7857d19e9dcfb89ad790afb5c239a35ac0af5e8a5bc5",
                "MacAddress": "02:42:ac:11:00:02",
                "IPv4Address": "172.17.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```
The output above includes IP and gateway information; we mainly care about the `Containers` property, which holds exactly the IP addresses assigned to the two `alpine`-based container instances we just created.

The container `alpine1` got the address `172.17.0.2`, and `alpine2` got `172.17.0.3`.

4. Connect to a container
```bash
$ docker attach alpine1

/ #
```
The `attach` command takes us back into a container running in the background; note that the command prompt changes, meaning we're now inside the container `alpine1`.

Next, look at the network information inside the container.

```bash
# ip addr show

1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
27: eth0@if28: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::42:acff:fe11:2/64 scope link
       valid_lft forever preferred_lft forever
```

Note the second network interface: `172.17.0.2` matches what we just saw with `docker network ls`.

5. Test network connectivity
```bash
# ping -c 2 baidu.com

PING google.com (172.217.3.174): 56 data bytes
64 bytes from 172.217.3.174: seq=0 ttl=41 time=9.841 ms
64 bytes from 172.217.3.174: seq=1 ttl=41 time=9.897 ms

--- baidu.com ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 9.841/9.869/9.897 ms
```
We send two packets to `baidu.com` with `ping -c 2`, and see that the container `alpine1` can reach the internet.

6. Test connectivity between containers
```bash
# ping -c 2 alpine2

ping: bad address 'alpine2'
```
Next we send packets to the container host `alpine2`, and it can't be reached. Of course not: we haven't set up anything for the two containers to talk to each other yet.

7. Create a bridge network
First leave the `alpine1` container environment: hold `CTRL` and type `p` then `q`, so the container doesn't exit the way `CTRL` + `d` would, and its running state stays in the background.

Now let's create a bridge network.
```bash
$ docker network create --driver bridge alpine-net
```

After it's created, list the networks again, and the custom bridge network shows up.
```bash
$ docker network ls

NETWORK ID          NAME                DRIVER              SCOPE
e9261a8c9a19        alpine-net          bridge              local
...
```

Right after, look at the details of `alpine-net`.
```bash
$ docker network inspect alpine-net

[
    {
        "Name": "alpine-net",
        "Id": "e9261a8c9a19eabf2bf1488bf5f208b99b1608f330cff585c273d39481c9b0ec",
        "Created": "2017-09-25T21:38:12.620046142Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```
Note the gateway information `IPAM.Config.Gateway` is `172.18.0.1`.

8. Run containers in custom bridge mode
```bash
$ docker run -dit --name alpine3 --network alpine-net alpine ash

$ docker run -dit --name alpine4 --network alpine-net alpine ash

$ docker run -dit --name alpine5 alpine ash

$ docker run -dit --name alpine6 --network alpine-net alpine ash
```

The new containers are all running.
```bash
$ docker container ls

CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
156849ccd902        alpine              "ash"               41 seconds ago       Up 41 seconds                           alpine6
fa1340b8d83e        alpine              "ash"               51 seconds ago       Up 51 seconds                           alpine5
a535d969081e        alpine              "ash"               About a minute ago   Up About a minute                       alpine4
0a02c449a6e9        alpine              "ash"               About a minute ago   Up About a minute                       alpine3
```

9. Look at the `alpine-net` information
```bash
$ docker network inspect alpine-net

[
    {
        "Name": "alpine-net",
        "Id": "e9261a8c9a19eabf2bf1488bf5f208b99b1608f330cff585c273d39481c9b0ec",
        "Created": "2017-09-25T21:38:12.620046142Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "172.18.0.0/16",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Containers": {
            "0a02c449a6e9a15113c51ab2681d72749548fb9f78fae4493e3b2e4e74199c4a": {
                "Name": "alpine3",
                "EndpointID": "c83621678eff9628f4e2d52baf82c49f974c36c05cba152db4c131e8e7a64673",
                "MacAddress": "02:42:ac:12:00:02",
                "IPv4Address": "172.18.0.2/16",
                "IPv6Address": ""
            },
            "156849ccd902b812b7d17f05d2d81532ccebe5bf788c9a79de63e12bb92fc621": {
                "Name": "alpine6",
                "EndpointID": "058bc6a5e9272b532ef9a6ea6d7f3db4c37527ae2625d1cd1421580fd0731954",
                "MacAddress": "02:42:ac:12:00:04",
                "IPv4Address": "172.18.0.4/16",
                "IPv6Address": ""
            },
            "a535d969081e003a149be8917631215616d9401edcb4d35d53f00e75ea1db653": {
                "Name": "alpine4",
                "EndpointID": "198f3141ccf2e7dba67bce358d7b71a07c5488e3867d8b7ad55a4c695ebb8740",
                "MacAddress": "02:42:ac:12:00:03",
                "IPv4Address": "172.18.0.3/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]
```
We can see that the three containers `alpine3`, `alpine4` and `alpine6` are all attached to the `alpine-net` network.

10. Test connectivity between containers again
```bash
$ docker container attach alpine3

# ping -c 2 alpine4

PING alpine2 (172.18.0.3): 56 data bytes
64 bytes from 172.18.0.3: seq=0 ttl=64 time=0.085 ms
64 bytes from 172.18.0.3: seq=1 ttl=64 time=0.090 ms

--- alpine4 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.085/0.087/0.090 ms

# ping -c 2 alpine6

PING alpine4 (172.18.0.4): 56 data bytes
64 bytes from 172.18.0.4: seq=0 ttl=64 time=0.076 ms
64 bytes from 172.18.0.4: seq=1 ttl=64 time=0.091 ms

--- alpine6 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.076/0.083/0.091 ms

# ping -c 2 alpine3

PING alpine1 (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.026 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.054 ms

--- alpine3 ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.026/0.040/0.054 ms

# ping -c 2 alpine5

ping: bad address 'alpine5'
```
This time `alpine3` can reach `alpine4` and `alpine6`, which are in the same network `alpine-net`, but can't reach `alpine5`, as expected.

11. Remove all the containers and the custom bridge network
```
$ docker container stop alpine1 alpine2 alpine3 alpine4 alpine5 alpine6

$ docker container rm alpine1 alpine2 alpine3 alpine4 alpine5 alpine6

$ docker network rm alpine-net
```

### iptables

Docker implements network isolation by changing iptables rules[^13].

> This subsection still needs work
{.is-warning}

### Security

Remember the Node.js service we customized from `node:current-slim` in the Images section? In the `Dockerfile` we ended up exposing the image's port 8080 to the host, then bound the host and container ports with `-p 8000:8080` when running the container, so users can reach the service on the container's port 8080 through the host's IP address and port 8000.

But this approach has a [problem](https://github.com/moby/moby/issues/22054): because Docker changes the iptables rules, iptables front-end firewalls like `ufw` stop working, which means every port you expose from a container to the host is exposed straight to the public internet, even if it isn't bound to a host port.

In real production, not every service needs to be available on the public internet. Many backend data centers and offline services don't need to (and mustn't) be reachable by users directly, and shouldn't be detectable by network scanners like `nmap`, so **hiding the ports containers expose, without breaking Docker's own customization of the iptables rules, becomes a necessary step to make production deployments more secure**.

[@chaifeng](https://github.com/chaifeng)'s open-source [ufw-docker](https://github.com/chaifeng/ufw-docker)[^14] solves this problem. By modifying the chain rules in `/etc/ufw/after.rules`, the author keeps Docker's handling of the routing table while letting UFW manage outside access to containers.

For the details, see [Solving the problem of ufw and docker](https://github.com/chaifeng/ufw-docker#%E8%A7%A3%E5%86%B3-ufw-%E5%92%8C-docker-%E7%9A%84%E9%97%AE%E9%A2%98) (in Chinese).

If you'd rather not edit things by hand, you can install the author's open-source [ufw-docker tool](https://github.com/chaifeng/ufw-docker#ufw-docker-%E5%B7%A5%E5%85%B7) (in Chinese) to manage the rules.

```bash
wget -O /usr/local/bin/ufw-docker \
  https://github.com/chaifeng/ufw-docker/raw/master/ufw-docker
chmod +x /usr/local/bin/ufw-docker
```

Then modify the `/etc/ufw/after.rules` file above directly with a command.
```bash
ufw-docker install
```

After a successful install and a restart, we find the Node.js service's port 8000 can no longer be reached from outside, as expected. Let's use `ufw-docker` to add a rule that lets traffic through to the service container's port.

```bash
ufw-docker allow <container_name> 8080
```
Note that the command above opens the container service's port 8080, i.e. the host's port 8000, which is more sensible than `ufw`'s behavior of opening both the host and container ports.

Finally, you can check the current firewall rules with the commands below.
```bash
ufw-docker list <container_name>
# or
ufw status
```

With that, the problem of safely opening container service ports is solved.

## Orchestration and configuration


## Clustering services

### Docker Swarm

> This subsection still needs work
{.is-warning}

### Kubernetes
> This subsection still needs work
{.is-warning}


# The ecosystem

## Continuous integration

<img src="/technology/saas/docker/inner-outer-loop.png" alt="inner-outer-loop.png" width="90%">

## Private image registry

Docker provides a platform for storing and distributing images called Registry[^15], and the platform itself can run the Docker way, as a container.

Unlike the Docker Hub service, the point of Registry is to build **private** image storage of your own.

### Setting it up and using it

With the official open-source `registry:2` image, we can conveniently run it as a container service.

```bash
docker run -d -p 5000:5000 --name registry registry:2
```

The command above gives us an image hosting service running on port 5000; next let's push an image to it.

First, pull the ubuntu image from Docker Hub to the local machine.

```bash
docker pull ubuntu
```

Then rename the local ubuntu image with the `tag` command.

```bash
docker image tag ubuntu localhost:5000/myfirstimage
```
Note that the command above renames it with the `localhost:5000` prefix because of how Docker names images[^16]: pull and push identifiers always take the form `<domain:port>/<image_owner_name>/<image_name>:<image_version>`.

Take `docker pull ubuntu` as an example: the full command is `docker pull docker.io/library/ubuntu:latest`.

After renaming, you can push it straight to the private registry.
```bash
docker push localhost:5000/myfirstimage
```

Other clients that can reach this service can pull the image in a similar way.
```bash
docker pull localhost:5000/myfirstimage
```

### Supporting HTTPS

The method in the last section of course can't be used in production. First we need outside clients to be able to reach the host, instead of simulating on one machine with `localhost`.

Given Docker's restrictions on production Registry services, to allow outside access the host's domain first needs TLS support[^17]. The official docs recommend using [Let’s Encrypt](https://docs.docker.com/registry/deploying/#support-for-lets-encrypt) (an open CA provider) to generate the host's private key and certificate.

1. Get a host and a domain
  First you need a server with a public IP and an associated domain (assume the host runs Ubuntu and the domain is `foobar.com`).
  
2. Install letsencrypt
  The Let’s Encrypt scripts are written in Python, and recent Ubuntu versions come with Python3 preinstalled, so here we only need to install pip (Python's package manager).
```bash
apt install -y python3-pip
```

3. Generate the private key and certificate
  The `letsencrypt` command asks you for an email account to bind the TLS service to.
```bash
letsencrypt certonly --standalone -d foobar.com
```
On success you'll find the certificate file and the private key file in the two directories below on the system.
```bash
/etc/letsencrypt/live/foobar.com/fullchain.pem  # certificate
/etc/letsencrypt/live/foobar.com/privkey.pem    # private key
```

4. Map the certificate and private key files into the Registry service.
In your project directory, create a new directory for the certificate and private key, and copy the corresponding files into it.
```bash
mkdir certs
cp /etc/letsencrypt/live/foobar.com/fullchain.pem certs/foobar.com.crt
cp /etc/letsencrypt/live/foobar.com/privkey.pem certs/foobar.com.key
```

Then rerun the Registry service with the certs directory mapped into the container.
```bash
$ docker run -d \
  --restart=always \
  --name registry \
  -v "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_ADDR=0.0.0.0:443 \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/foobar.com.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/foobar.com.key \
  -p 443:443 \
  registry:2
```
The command above uses environment variables to redefine the service's port as 443 (instead of the earlier 5000), and gives the container the certificate and private key files in the `certs` directory.

5. Test the Registry private image service with TLS support
Here we pull an `ubuntu:16.04` image from Docker Hub as a test and push it to the private registry.
```bash
$ docker pull ubuntu:16.04
$ docker tag ubuntu:16.04 foobar.com/my-ubuntu
$ docker push foobar.com/my-ubuntu
```

After the push succeeds, pull it again from another client.
```bash
$ docker pull foobar.com/my-ubuntu
```

With that, we've dealt with the security restrictions on outside access.

### User authentication

As of the last section, we can reach the private registry from outside, but that means anyone can push images to the 443 service on the `foobar.com` domain. In this section we fix that with the officially provided [native authentication](https://docs.docker.com/registry/deploying/#native-basic-auth).

Docker uses the Apache tool `htpasswd` to generate an account and password for simple access control.

1. Generate the account and password file
  Create an `auth` directory in the project and run the `htpasswd` command to generate the password and the associated user file.
```bash
mkdir auth
htpasswd -Bbn testuser testpassword > auth/htpasswd
```

2. Redeploy the Registry service
  To make the next authentication step easier to follow, we start the container on port 5000 instead.
```bash
$ docker container stop registry

$ docker run -d \
  --restart=always \
  --name registry \
  -p 5000:5000 \
  -v "$(pwd)"/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  -e REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd \
  -v "$(pwd)"/certs:/certs \
  -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/foobar.com.crt \
  -e REGISTRY_HTTP_TLS_KEY=/certs/foobar.com.key \
  registry:2
```

3. Log in
```bash
$ docker login foobar.com:5000
```
After that you can push and pull private images just as usual.

4. Manage the deployment with a Compose file
  We can turn step 3 into configuration in the form of a `docker-compose.yml` file.
  ```yaml
  registry:
    restart: always
    image: registry:2
    ports:
      - 5000:5000
    environment:
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/foobar.com.crt
      REGISTRY_HTTP_TLS_KEY: /certs/foobar.com.key
      REGISTRY_AUTH: htpasswd
      REGISTRY_AUTH_HTPASSWD_PATH: /auth/htpasswd
      REGISTRY_AUTH_HTPASSWD_REALM: Registry Realm
    volumes:
      - /path/data:/var/lib/registry
      - /path/certs:/certs
      - /path/auth:/auth
  ```
  
  Then deploy with the `docker-compose` command.
  ```bash
  docker-compose up -d
  ```

### RESTful API

> This subsection still needs work
{.is-warning}

## GUI services

## Monitoring

> This subsection still needs work
{.is-warning}


# How Docker works

After using Docker in the last chapter for things like continuous integration, service deployment and private image hosting, let's try to think: if it were up to us, how would we build containers on top of what the Linux operating system provides?

## Resource isolation

Docker implements resource isolation with Linux namespaces[^18]; you can read about this feature in the Linux manual.

```
$ man namespaces
```

The resources here can be thought of as the complete operating system environment we depend on once services run in containers, such as separate processes, separate network interfaces and separate mount points.

Unlike the other processes running on the host, container processes get a separate, isolated operating system environment through namespaces. With the `clone` system call, flags control exactly what gets isolated, and the resources to isolate can be chosen along the following dimensions.

- Host UTS (Unix Time-sharing System)
  Pass `CLONE_NEWUTS` to isolate the hostname and domain name
- Process ID PID (Process ID)
  Pass `CLONE_NEWPID` to isolate process IDs
- Inter-process communication IPC (Inter-Process Communication)
  Pass `CLONE_NEWIPC` to isolate semaphores, message queues, shared memory, sockets and streams
- Network
  Pass `CLONE_NEWNET` to isolate network devices and ports
- Mount points (Mount)
  Pass `CLONE_NEWNS` to isolate file system mount points

Following [ Let's write one in Go from scratch](https://www.youtube.com/watch?v=HPuvDm8IC-4)[^19], published by [GopherCon UK](https://www.youtube.com/channel/UC9ZNrGdT2aAdrNbX78lbNlQ), we can understand from the key logic how namespaces actually achieve basic resource isolation.

### UTS (Unix Time-sharing System)

The [first version of the logic](https://youtu.be/HPuvDm8IC-4?t=609) below, written in Go, calls the system interface `clone` with the `CLONE_NEWUTS` flag to isolate the hostname.

```go
package main

import (
    "fmt"
    "os"
    "os/exec"
    "syscall"
)

func main() {
    switch os.Args[1] {
    case "run":
        run()
    default:
        panic("what?")
    }
}

func run() {
    fmt.Printf("running %v\n", os.Args[2:])
    cmd := exec.Command(os.Args[2], os.Args[3:]...)
    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr

    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWUTS,
    }

    must(cmd.Run())
}

func must(err error) {
    if err != nil {
        panic(err)
    }
}
```

Save the logic above as `main.go` and run the command below to compile and run the Go code, using `/bin/bash` as the interactive shell of the new process opened by `clone`. Change the `hostname` in the new process's Bash environment, then exit the session: you'll find the changed hostname doesn't affect the host's `hostname`, so the hostname is isolated.

```bash
$ go run main.go run /bin/bash
running [/bin/bash]
# hostname
jovi.archer
# hostname foobar
# hostname
foobar
# exit
$ hostname
jovi.archer
```

### PID

Before isolating process IDs, we need to know a few things about processes in Linux.

If you've used the `ps -ef` command to look at the processes running on your system, you'll have noticed the two special processes below.

```bash
$ ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 22:05 ?        00:00:01 /sbin/init
root         2     0  0 22:05 ?        00:00:00 [kthreadd]
```

One is the `/sbin/init` process with PID 1, which runs the system's initialization tasks; the other, `kthreadd`, schedules the other kernel processes.

If you've also used the `pstree` command, you'll have seen a very vivid process tree printed in the terminal, roughly like this.

```bash
systemd─┬─accounts-daemon─┬─{gdbus}
        │                 └─{gmain}
        ├─acpid
        ├─agetty
        ├─atd
        ├─containerd─┬─containerd-shim─┬─v2ray───7*[{v2ray}]
        │            │                 └─9*[{containerd-shim}]
        │            └─8*[{containerd}]
        ├─cron
        ├─dbus-daemon
        ├─dockerd───7*[{dockerd}]
        ├─2*[iscsid]
        ├─lvmetad
        ├─mdadm
        ├─polkitd─┬─{gdbus}
        │         └─{gmain}
        ├─rsyslogd─┬─{in:imklog}
        │          ├─{in:imuxsock}
        │          └─{rs:main Q:Reg}
        ├─snapd───8*[{snapd}]
        ├─sshd───sshd───bash
        ├─systemd───(sd-pam)
        ├─systemd-journal
        ├─systemd-logind
        ├─systemd-timesyn───{sd-resolve}
        └─systemd-udevd
```
You can see that every process is started by the system service manager systemd; containerd is the name of the process Docker creates, and every container instance is managed by a process called containerd-shim, isolated from the host's processes.

So how does Docker isolate processes from the host? We add to the Go code above the [`CLONE_NEWPID`](https://youtu.be/HPuvDm8IC-4?t=670) flag of the `clone` system call, which the process isolation in namespaces relies on, plus a new Child process to make the result easy to see.

```go
package main

import (
    "fmt"
    "os"
    "os/exec"
    "syscall"
)

func main() {
    switch os.Args[1] {
    case "run":
        run()
    case "child":
        child()
    default:
        panic("what?")
    }
}

func run() {
    fmt.Printf("running %v\n", os.Args[2:])
    cmd := exec.Command("/proc/self/exe", append([]string{"child"}, os.Args[2:]...)...)
    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr

    cmd.SysProcAttr = &syscall.SysProcAttr{
        Cloneflags: syscall.CLONE_NEWUTS | syscall.CLONE_NEWPID,
    }

    must(cmd.Run())
}

func must(err error) {
    if err != nil {
        panic(err)
    }
}

func child() {
    fmt.Printf("running %v as pid: %d\n", os.Args[2:], os.Getpid())
    cmd := exec.Command(os.Args[2], os.Args[3:]...)
    cmd.Stdin = os.Stdin
    cmd.Stdout = os.Stdout
    cmd.Stderr = os.Stderr
    
    must(cmd.Run())
}
```
Now run `go run main.go run /bin/bash` again to get into the process's interactive shell, and type `ps`: you'll see that the host's other processes can no longer be listed from inside this process.

### IPC

### Network

### Mount points

## cgroup

## Storage drivers

### Union FS 

[^1]: [Docker Overview | Docker Documentation](https://docs.docker.com/get-started/overview/)
[^2]: [Get Docker | Docker Documentation](https://docs.docker.com/get-docker/)
[^3]: [Install Docker Engine | Docker Documentation](https://docs.docker.com/engine/install/#server)
[^4]: [Docker overview # Docker architecture | Docker Documentation](https://docs.docker.com/get-started/overview/#docker-architecture)
[^5]: [Docker overview # Docker Engine | Docker Documentation](https://docs.docker.com/get-started/overview/#docker-engine)
[^6]: [Orientation and setup # Containers and virtual machines | Docker Documentation](https://docs.docker.com/get-started/#containers-and-virtual-machines)
[^7]: [Build and run your image | Docker Documentation](https://docs.docker.com/get-started/part2/#set-up)
[^8]: [Use volumes | Docker Documentation](https://docs.docker.com/storage/volumes/)
[^9]: [Use volumes # choose-the--v-or---mount-flag | Docker Documentation](https://docs.docker.com/storage/volumes/#choose-the--v-or---mount-flag)
[^10]: [Networking overview | Docker Documentation](https://docs.docker.com/network/)
[^11]: [Plugins and Services | Docker Documentation](https://docs.docker.com/engine/extend/plugins_services/)
[^12]: [Networking with standalone containers | Docker Documentation](https://docs.docker.com/network/network-tutorial-standalone/)
[^13]: [Docker and iptables | Docker Documentation](https://docs.docker.com/network/iptables/)
[^14]: [chaifeng/ufw-docker: To fix the Docker and UFW security flaw without disabling iptables](https://github.com/chaifeng/ufw-docker)
[^15]: [Docker Registry | Docker Documentation](https://docs.docker.com/registry/)
[^16]: [About Registry | Docker Documentation](https://docs.docker.com/registry/introduction/#understanding-image-naming)
[^17]: [Deploy a registry server | Docker Documentation](https://docs.docker.com/registry/deploying/#run-an-externally-accessible-registry)
[^18]: [namespaces(7) - Linux manual page](https://man7.org/linux/man-pages/man7/namespaces.7.html)
[^19]: [Golang UK Conf. 2016 - Liz Rice - What is a container, really? Let's write one in Go from scratch](https://www.youtube.com/watch?v=HPuvDm8IC-4)