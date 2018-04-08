## The problem.
![TO BE OR NOT TU BE](https://lh6.googleusercontent.com/yuqE8Q2LDtwbzfipgUzHSHhh4o01mUbEc_ej-FFdngHMTWAICEn3YqnS2bwNuiw3kd0rgBdo0gJOvgFlkmYR=w1366-h601)
Hello there dear fellows, how are you? I haven't written new articles those days because i was busy on German translations, pass all the released articles in my [**facebook community**](https://www.facebook.com/noplacenoaddress/), real mode assembler applications, and social networks, best results are on [my own twitter](https://twitter.com/taglio). I'm very satisfied but the i've got to work hard and i've got to climb a lot of difficult mountain passes. *Pray for me* and remember that the *wind of changes* is blowing, **join us on #changeNOW**.
Today we're with our first and best friend, his name is **OpenBSD**.
We want to access to the same network, our **WAN** in this case, with to different network interfaces, the ethernet and the wireless (*with or without cable*) depending on where we're going to connect our fantastic **OpenBSD laptop**. And we want like usual that all the operations will be totally transparent to the end user when we will apply our solution. That is right now, here below under your eyes. Use it!
## The solution
![failover with two slaves](https://lh4.googleusercontent.com/v1Ztfzu4c-fuqWgStIuy3F0EN43i_d0syTLunctnPfD_9zhZTES0Hbugws6LiEmGO7c-2t-_K4maQ0isfbe0=w1366-h601)
Technically speaking in computer science the operation that we're going to explain in this rapid article is what we known for **failover**. In **OpenBSD** we can configure it with the **trunk** driver who's explained on the system manual pages at `man 4 trunk`.
 `trunk`accept six different operational modes:
 1. `broadcast`
 2. `failover`
 3. `lacp`
 4. `loadbalance`
 5. `none`
 6. `roundrobin`

Like you can image, we're going to use this time the `failover`one:

> Sends and receives traffic only through the master port.  If
                  the master port becomes unavailable, the next active port is
                  used.  The first interface added is the master port; any
                  interfaces added after that are used as failover devices.

But **let it be** and start with the configuration:
```sh
$ cd /etc
$ doas rm -rf hostname.re0 hostname.iwm0
$ doas su
# cat > hostname.re0 <<EOF
lladdr random
up
EOF
# cat > hostname.iwm0 <<EOF
lladdr random
up
nwid cyberterror
wpakey stopprivacycannibals
# cat > hostname.trunk0 <<EOF
trunkproto failover trunkport re0 trunkport iwm0
EOF
# sh /etc/netstart
```
And like usual let me explain exactly what we're doing, you know...i'm clear and creative:
1. change directory to the system configuration one.
2. remove old interfaces configuration.
3. activate **root power** account.
4. configurate ethernet interface and meanwhile *fuck a little* our privacy cannibals using a random layer 2 `mac address`.
5. configurate wireless interface, go *ahead with the fuck* , connect with our home wi-fi, the mine is `cyberterror` using a `wpa2-aes` key that in the example is `stopprivacycannibal`.
6. configurate the `pseudo` interface trunk0 to the layer network adding two `slaves`, `re0`and `iwm0`. The result is a simple and efficient `failover`.
7. Restart `networking`.

## Greetings
Like usual i'm educated and i use to cheers people. So *bye bye* and **keep in touch**.

*Riccardo Giuntoli*

