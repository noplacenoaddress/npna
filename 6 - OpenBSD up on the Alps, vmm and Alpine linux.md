
## Four days ago...
¡Hello there, dear souls! Days pass rapidly, exactly *four days ago* we're speaking about add another soldier, who's name was **privoxy**,  in our personal battle with the infamous *privacy cannibals*, [¿do you remember?](https://steemit.com/openbsd/@npna/openbsd-tor-privoxy-and-the-browsers)
Our new friend, could act like a **layer 7 firewall**, but in our last article we didn't any firewalling, only we rewrite the **Refer** HTTP header.
You have to know that some fantastic dudes maintained a public list of hosts that use our *personal sensible data* with the scope of monetizing it (¿have i give my permission to this abuse?) . The name of the list is **easylist** and you can navigate to the home site of the project [here](https://easylist.to/). Normally is used with browsers extensions to grant **ad-free** navigation to his users. But we're *deep* users and we want to use the list in ours *privoxy* rules. The problem is that this is not so simple. There is a project on the *fantastic github* that could help us in doing that, but it use a language not installed by default in our **OpenBSD**, the name of the language is [**haskell**](https://www.haskell.org/), a *standardized, general-purpose purely functional programming language*. Here is some links to the project:

1. [GitHub project home page](https://github.com/essandess/adblock2privoxy)
2. [Project home page](https://projects.zubr.me/wiki/adblock2privoxy)

**Haskell** can be installed on **OpenBSD** but **adblock2privoxy** want [*The stack package*](https://hackage.haskell.org/package/stack) that i couldn't correctly compile under OpenBSD. 

That's the reason why i take my OpenBSD and we go to the up of the [Alps](https://en.wikipedia.org/wiki/Alps) to meet with *another good friend*, another security guy, [**Alpine linux**](https://www.alpinelinux.org/).

## Install Alpine linux under OpenBSD vmm

![Alps](https://images.duckduckgo.com/iu/?u=https%3A%2F%2Fwww.rci.com%2Fstatic%2Fimages%2Fcontent%2Findia%2FC5%2FS5%2Fc5-en_IN_all-destinations-Alps.jpg&f=1)
We're lucky, or *the karma* is in love with us (we're also in love with you), because a few months ago **OpenBSD** introduce in his base tree the *virtual machine monitor* [**vmm**](https://man.openbsd.org/vmm). But *karma* help us more than this, because it's just very few months  that under *vmm* we can virtualize *linux*, read [here](https://marc.info/?l=openbsd-misc&m=149329839013688&w=2).
Ok, let's start with prepare the correct *environment* for our new friend:

Download **Alpine** minimal ISO image, to use in virtual environment, and create a virtual disk of 10GB:
```sh
$ wget http://dl-cdn.alpinelinux.org/alpine/v3.7/releases/x86_64/alpine-virt-3.7.0-x86_64.iso
$ vmctl create alpine-virt.img -s 10G
```
  Enable routing in our OpenBSD:
```sh
$ doas sysctl net.inet.ip.forwarding=1
# echo "net.inet.ip.forwarding=1" >> /etc/sysctl.conf
```
Create virtual switch `vether0`:
```sh
# echo "inet 10.1.10.1 255.255.255.0" > /etc/hostname.vether0
# cat > /etc/vm.conf << EOF
switch "local" {
        add vether0
        up
}
EOF
```
Enable **vmd** on boot, enable it and download the virtual BIOS:
```sh
$ doas rcctl enable vmd
$ doas rcctl start vmd
$ doas fw_update
```
Indicate dns service of Tor to listen in the `vether0` interface and restart it:
```sh
# echo 'DNSPort 10.1.10.1:53 IsolateDestPort' >> /etc/tor/torrc
$ doas rcctl restart tor
```
Create a dedicated **privoxy** for our new friend, **Alpine**:
```sh
$ doas cp -p /etc/rc.d/privoxyfirefox /etc/rc.d/privoxyvesta
# cat > /etc/privoxy/vesta << EOF
#        Sample Configuration File for Privoxy 3.0.26
#
# $Id: config,v 1.112 2016/08/26 13:14:18 fabiankeil Exp $
#
# Copyright (C) 2001-2016 Privoxy Developers https://www.privoxy.org/
#

user-manual https://www.privoxy.org/user-manual/
trust-info-url https://learn.canva.com/wp-content/uploads/2015/06/50-Of-The-Most-Creative-404-Pages-On-The-Web-01.png
admin-address r.giuntoli@protonmail.ch
#config guide
#proxy-info-url http://www.example.com/proxy-service.html
confdir /etc/privoxy
templdir /etc/privoxy/templates
logdir /var/log/privoxy
actionsfile match-all.action # Actions that are applied to all sites and maybe overruled later on.
actionsfile default.action   # Main actions file
actionsfile user.action      # User customizations
filterfile default.filter
filterfile user.filter      # User customizations
logfile privoxyvesta.log
#if set all deny but the ones listed on [use ~ like *]
#trustfile trust
#
#        debug     1 # Log the destination for each request Privoxy let through. See also debug 1024
#        debug     2 # show each connection status
#        debug     4 # show I/O status
#        debug     8 # show header parsing
#        debug    16 # log all data written to the network
#        debug    32 # debug force feature
#        debug    64 # debug regular expression filters
#        debug   128 # debug redirects
#        debug   256 # debug GIF de-animation
#        debug   512 # Common Log Format
#        debug  1024 # Log the destination for requests Privoxy didn't let through, and the reason why.
#        debug  2048 # CGI user interface
#        debug  4096 # Startup banner and warnings.
#        debug  8192 # Non-fatal errors
#        debug 32768 # log all data read from the network
#        debug 65536 # Log the applying actions
debug     1 # Log the destination for each request Privoxy let through. See also debug 1024.
#debug  1024 # Actions that are applied to all sites and maybe overruled later on.
#debug  4096 # Startup banner and warnings
#debug  8192 # Non-fatal errors
single-threaded 0
hostname Lutetia.unknown_domain
listen-address  10.1.10.1:8812
#filter mode
toggle  1
enable-remote-toggle  0
#filter by X-filter http header
enable-remote-http-toggle  0
enable-edit-actions 0
enforce-blocks 1
#      src_addr[:port][/src_masklen] [dst_addr[:port][/dst_masklen]]
permit-access  10.1.10.2
buffer-limit 8192
#enable if there's a parent proxy
enable-proxy-authentication-forwarding 0
forward-socks5 / 127.0.0.1:9912 .
forwarded-connect-retries  0
#transparent proxy
accept-intercepted-requests 0
#
allow-cgi-request-crunching 0
split-large-forms 0
# grow up to 300 (if browser hang stop)
keep-alive-timeout 5
# disable if problems
tolerate-pipelining 1
#default-server-timeout 60
connection-sharing 0
# try to reduce to 5 sec
socket-timeout 300
#max-client-connections 256
handle-as-empty-doc-returns-ok 0
#enable-compression 1
#compression-level 3
#client-header-order Host \
#   Accept \
#   Accept-Language \
#   Accept-Encoding \
#   Proxy-Connection \
#   Referer \
#   Cookie \
#   DNT \
#   If-Modified-Since \
#   Cache-Control \
#   Content-Length \
#   Content-Type
#
#client-specific-tag circumvent-blocks Overrule blocks but do not affect other actions
#          disable-content-filters Disable content-filters but do not affect other actions
#
#
#            client-tag-lifetime 180
#            # IP address with a X-Forwarded-For header.
#            trust-x-forwarded-for 1
EOF
$ doas rcctl enable privoxyvesta
$ doas rcctl set privoxyvesta flags=/etc/privoxy/vesta
$ doas rcctl start privoxyvesta
```
And prepare pf for the new services opened on `vether0`:
```sh
# cat >> /etc/pf.conf << EOF
pass in on vether0 proto tcp from 10.1.10.2 to 10.1.10.1 port 8812
pass in on vether0 proto udp from 10.1.10.2 to 10.1.10.1 port 53
EOF
$ doas pfctl -f /etc/pf.conf
```
Notice that we don't NAT connections from *Alpine linux vesta*, it will only arrive in Internet by the use of our *privoxy* dedicated instance.
## Alpine under OpenBSD, the video

Now, using [this guide](https://wiki.alpinelinux.org/wiki/Install_to_disk) we are going to install our **Alpine** under the **OpenBSD** :

[![asciicast](https://asciinema.org/a/14.png)](https://asciinema.org/a/14)

*...to be continued...* and yes **I LOVE YOU**
