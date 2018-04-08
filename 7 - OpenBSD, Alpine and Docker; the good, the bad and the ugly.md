## The goodfellas

![blue whale](https://images.duckduckgo.com/iu/?u=http%3A%2F%2Fnews.bbcimg.co.uk%2Fmedia%2Fimages%2F50355000%2Fjpg%2F_50355036_blue_whale_1.jpg&f=1)
OpenBSD, [*last time*](https://steemit.com/openbsd/@npna/openbsd-up-on-the-alps-vmm-and-alpine-linux), meet a good friend in the alps, [**Alpine Linux**](https://en.wikipedia.org/wiki/Alpine_Linux), *¿do you remember?*
After **four** days of [*titties & beer*](https://www.last.fm/music/Frank+Zappa/_/Titties+and+Beer) , they decides to meet another friend, *a blue whale*, his name is [**Docker**](https://www.docker.com/). 
## ¿Who is Docker?

> Docker is a software technology providing containers, promoted by the company Docker, Inc. Docker provides an additional layer of abstraction and automation of operating-system-level virtualization on Windows and Linux. Docker uses the resource isolation features of the Linux kernel such as cgroups and kernel namespaces, and a union-capable file system such as OverlayFS and others to allow independent "containers" to run within a single Linux instance, avoiding the overhead of starting and maintaining virtual machines (VMs).

>The Linux kernel's support for namespaces mostly isolates an application's view of the operating environment, including process trees, network, user IDs and mounted file systems, while the kernel's cgroups provide resource limiting, including the CPU, memory, block I/O, and network. Since version 0.9, Docker includes the libcontainer library as its own way to directly use virtualization facilities provided by the Linux kernel, in addition to using abstracted virtualization interfaces via libvirt, LXC (Linux Containers) and systemd-nspawn.

## OpenBSD, Alpine linux and Docker
Always remembering [*russian Matryoshka*](https://steemit.com/openbsd/@npna/openbsd-tor-privoxy-and-the-browsers), we've decided to add another layer of virtualization to our workstation searching how to build [**adblock2privoxy**](http://projects.zubr.me/wiki/adblock2privoxy) in OpenBSD. Retaking our last video tutorial we've got **Alpine linux** correctly installed in a virtual environment in a OpenBSD host. Now we've to adjust some parameters in Alpine to finish the installation:
Add the virtual machine in automatic boot with the host OS:
 ```sh
 # cat >> /etc/vm.conf <<EOF
 vm "screencast" {
        memory 2048M
        disk "/home/taglio/Virtual/alpine-screencast.img"
        interface { switch "local" }
}
EOF
#
 $ doas rcctl restart vmd
 ```
 Connect to the serial console, press `ENTER` to boot the default `initramfs` and `kernel` and customize `syslinux` [bootloader](http://www.syslinux.org/) and enable the community repository of `apk` [packet manager](https://wiki.alpinelinux.org/wiki/Alpine_Linux_package_management):
 ```sh
 alpine# cat /boot/extlinux.conf
 SERIAL 0 115200
DEFAULT virthardened
PROMPT 0
LABEL virthardened
  MENU LABEL Linux virthardened
  LINUX vmlinuz-virthardened
  INITRD initramfs-virthardened
  APPEND root=UUID=ebf73ff9-7df6-463d-915f-0ab84f5e11e9 modules=sd-mod,usb-storage,ext4 quiet rootfstype=ext4
MENU SEPARATOR
alpine# cat /etc/apk/repositories
http://nl.alpinelinux.org/alpine/v3.7/main
http://nl.alpinelinux.org/alpine/v3.7/community
#
 ```
 Doing so Alpine will reboot automatically without the need of press `ENTER` in `syslinux` prompt. 
 Generate `root` OpenBSD ssh keys and add to the `root` `authorized_keys` in the virtual Alpine:
```sh
$ doas su
# ssh-keygen
# ssh 10.1.10.2 mkdir /root/.ssh
# scp ./ssh/id_rsa.pub 10.1.10.2:/root/.ssh/authorized_keys
```
 Upgrade Alpine packages and install our new friend, **Docker**:
 ```sh
 alpine# apk update && apk upgrade
 alpine# apk add docker
 ```
 Configure Alpine `openrc` , the [**Gentoo**](https://gentoo.org/) [antagonist](https://wiki.gentoo.org/wiki/Project:OpenRC) of the *hugly infamous* [systemd](https://suckless.org/sucks/systemd), to boot on start our **Docker**.
 ```sh
 alpine# rc-service docker start
 alpine# rc-update add docker default
 ```
 ## adblock2privoxy Docker configuration
 **Docker** container is a extremely flexible software. Have got thousand of different possible uses and configurations. And a *deep dive* into it will require an entire new series of articles (*asap* we will analyze it). 
 The most important think to understand, he has got a public internet libraries from where you can `pool` a system operative `image` into our local system. After pulling the correct one we can automate a serie of operation every time we `build` one of them into a `container`. I know...there's many new types of concepts here, but with practice it will be simple. Containers are super flexibles because for every command defined in a special file called `Dockerfile` they save a subcontainer in a `overlay` [filesystem](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/tree/Documentation/filesystems/overlayfs.txt).
 This is the `Dockerfile` that we're going to use in this project:
 ```sh
 alpine# mkdir docker
 alpine# cat > Dockerfile << EOF
 FROM debian:8

MAINTAINER Riccardo Giuntoli <r.giuntoli@protonmail.ch>

RUN apt-get update
RUN apt-get -y upgrade

RUN apt-get -y install wget 

WORKDIR /root
RUN mkdir privoxy
RUN mkdir lists

RUN wget https://s3.amazonaws.com/ab2p/adblock2privoxy_1.4.2_amd64.debian8.deb
RUN dpkg -i adblock2privoxy_1.4.2_amd64.debian8.deb

RUN wget -O lists/easyprivacy.txt https://easylist.to/easylist/easyprivacy.txt 
RUN wget -O lists/easylist.txt https://easylist.to/easylist/easylist.txt
RUN wget -O lists/antiadblockfilters.txt https://easylist-downloads.adblockplus.org/antiadblockfilters.txt
RUN wget -O lists/malwaredomains_full.txt https://easylist-downloads.adblockplus.org/malwaredomains_full.txt
RUN wget -O lists/adblock-list.txt https://raw.githubusercontent.com/Dawsey21/Lists/master/adblock-list.txt

RUN adblock2privoxy -p ./privoxy lists/easyprivacy.txt lists/easylist.txt  \
	lists/antiadblockfilters.txt lists/malwaredomains_full.txt lists/adblock-list.txt

RUN tar -cvzf privoxy.tgz privoxy/
 ```
 Let's dive a little bit into this `Dockerfile`:
 1. `FROM`: key to specify base `image` where start to compile the `container` , in this key is a [**debian**](https://debian.org) machine, stable version number **8**, codename [**jessie**](https://www.debian.org/releases/jessie/).
 2. `MANTAINER`: simply the owner of this `Dockerfile`.
 3. `RUN`: `exec` commands in the virtual debian environment. 
 4. `WORKDIR`: change the working directory.
 
In this specific docker application you can see that we download and install our **adblock2privoxy** software, download *bad boys* list maintained buy the guys of [**easylist.to**](https://easylist.to/), give to adblock2proxy and pack the result in a `tar.gz` archive.
## Automatic OpenBSD, Alpine and Docker process
Our goal is the automatize all the process and every week update ours **privoxy** rules. We've got an environment with three distinct system operatives, one OpenBSD and two Linux, it's like an orgy. 
Let's start to create a little `ash` script in our **Alpine**:
```sh
alpine# mkdir bin
alpine# cat > bin/automatic_docker.sh <<EOF
cd docker/ && docker build -t adblock2privoxy-bootstrap:final .
ASDF=`docker run -d -t adblock2privoxy-bootstrap:final /bin/bash`
docker cp $ASDF:/root/privoxy.tgz .
docker system prune -f
docker stop $(docker ps -a -q)
docker rm $(docker ps -a -q)
docker rmi $(docker images -q)
EOF
```    
1. `docker build`: Build an image from a Dockerfile.
2. `docker run`: Run a command in a new container.
3. `docker system prune`: Remove unused data.
4. `docker stop`: Stop one or more running containers.
5. `docker rm`: Remove one or more containers.
6. `docker rmi`: Remove one or more images.
7. `docker ps`: List containers.
8. `docker images`: List images.

Next combine it with a little `ksh` *magic* in our **OpenBSD**:
```sh
#/bin/sh
ssh 10.1.10.2 rm -rf /root/docker/privoxy.tgz
ssh 10.1.10.2 sh /root/bin/automatic_docker.sh
scp 10.1.10.2:/root/docker/privoxy.tgz /etc/privoxy/adblock2privoxy.tgz
cd /etc/privoxy/
tar zxvf adblock2privoxy.tgz
```
Very basic, i know, it simply remove old output, run the Alpine script, copy and `untar` the output in our **privoxy** directory.
Just add a `crontab` weekly script in OpenBSD and indicate to the [*three browsers*](https://steemit.com/openbsd/@npna/openbsd-tor-privoxy-and-the-browsers)  to do the **things work like a charm**.
```sh
# crontab -l | tail -n 1
5       12      *       *       2       /bin/sh /root/bin/automatic_adblock2privoxy.sh
# 
# cat >> /etc/privoxy/firefox <<EOF
actionsfile adblock2privoxy/ab2p.system.action
actionsfile adblock2privoxy/ab2p.action
filterfile adblock2privoxy/ab2p.system.filter
filterfile adblock2privoxy/ab2p.filter
EOF
# cat >> /etc/privoxy/chrome <<EOF
actionsfile adblock2privoxy/ab2p.system.action
actionsfile adblock2privoxy/ab2p.action
filterfile adblock2privoxy/ab2p.system.filter
filterfile adblock2privoxy/ab2p.filter
EOF
# cat >> /etc/privoxy/torbrowser <<EOF
actionsfile adblock2privoxy/ab2p.system.action
actionsfile adblock2privoxy/ab2p.action
filterfile adblock2privoxy/ab2p.system.filter
filterfile adblock2privoxy/ab2p.filter
EOF
```
And with this **triple cat** i remember to you **all** that the wind of changes is blowing. 

*i love you*,
