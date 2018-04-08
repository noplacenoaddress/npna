## INTRODUCTION TO THE FACTS

![OpenBSD puffy](https://steemitimages.com/0x0/https://www.openbsd.org/images/puffy62.gif)

Hello people, got a good Saturday? Ready to open tons of bottles tomorrow? I'm not...this year specially, i'm not ready. I will return on the street without any doubt, but this the another story.

So in our hands there's the nightmare of the security cannibals, the fish more difficult to take. There's OpenBSD. He (yes, it's a person, it's my best friend) is a pure BSD 4.4 based Unix that never fail to his customer, respecting rules that, i'm my opinion, are fundamental blocks of great secures peaces of software:

1. Clean and clear code.
2. Default install with the necessary, no less no more.
3.  Very easy to use for a new customer.

Yes, the point three will open yours eyes i know, you will be accuse my to be insane....Well for sure... i'm totally insane but OpenBSD it's very clear and simple to manage. The secret it's to follow simple rules very easy to understand for the new and semi-new customer. OpenBSD don't use strange, absurd and loooong explication or vocabulary to explain the things. He will do it in simple way. 

Remember, where you will find tons of code, many times there some fantastic genius that have done it to take you away from the source code. Adding complexity and time to understand basics procedure it's absolutely not a sign of great knowledge, it's a medieval procedure to maintain people away from the real purpose, that in this case will be obtain the keys to an extreme powerful, secure and clear Unix: OpenBSD.

I love Linux, but nowadays something, someone, or whatever the name you will use are totally destroy it. Millions of lines of code and complexity that have grow up a lot at low level code. For low class customer can use it with nice visual effects..

It's not difficult to understand, think about the new systemd, adopted for A LOT of great Linux distribution. Here are some links that will help you thing and OPEN your mind.

- [https://suckless.org/](https://suckless.org/)
- [https://suckless.org/sucks/systemd](https://suckless.org/sucks/systemd)
-   [http://without-systemd.org/wiki/index.php/Arguments_against_systemd](http://without-systemd.org/wiki/index.php/Arguments_against_systemd)


And this only the first thing that i've got in my mind when a speak about the beautiful but terribly fucked Linux, that it's NOT secure by default.

## TOR, EIN GOTT ODER EIN BETRÜGER (ohne das H)

![THOR](https://steemitimages.com/0x0/http://www.ancientpages.com/wp-content/uploads/2017/06/thormidgardsorm11.jpg)

Tor in computer science, that is what i'm speaking about with some Baroque embellishments, is the state of the art to theoretically preserve privacy of a Internet user. Remember very well, it was not designed to act like a security machine but like a privacy ensure. Is an open source software that enable anonymous communication  in Internet. It can be stacked with other technologies, like vpn or i2p, to try conceal our real ip address and geo point of source.

Thor mythological speaking, was son of Odin, dedicated to the protection of mankind and of the fortress of Asgard.

But..¿Was or is T[h]or really dedicated to the protection of the truth or does it complain also with other more obscures functions? A great question with a very difficult reply without a lot of sentiment and passion. I'm trying to do it better that i can but i really appreciate T[h]or. Really.

In real world, ou Jesus..real...in the binary system, Tor also open to the user, the man who navigate the Internet ocean, the access to the subterranean Internet, the (in)famous #deepweb, or Deep Internet. And like all the things that we know, deep can be deeper and deeper and deeper and so on. Many people think that #deepinternet is many times bigger than the Internet, is this true? Nobody, that i know ... ¿you know? know the answer. Like the deepest ocean in nature, i cannot explain to you what is the deepest Internet and who is the owner of that obscure site.

So it's normal that sometimes i was thinking about T[h]or more like Χάρων.

Here you are some URI to try to do some light more on that themes:
	1. [https://www.torproject.org/](https://www.torproject.org/)
	2.  [https://guardianproject.info/apps/orbot/](https://guardianproject.info/apps/orbot/)
	3. [https://www.torproject.org/projects/torbrowser.html.en](https://www.torproject.org/projects/torbrowser.html.en)
	4. [https://ooni.torproject.org/](https://ooni.torproject.org/)
	5. [https://atlas.torproject.org/](https://atlas.torproject.org/)
	6. [https://nyx.torproject.org/](https://nyx.torproject.org/)

## THE FOURTEEN EYES

![The 14 eyes](https://steemitimages.com/0x0/https://restoreprivacy.com/wp-content/uploads/2017/10/5-Eyes-9-Eyes-14-Eyes-2.png)

But let's start together configuring those two fantastic open source project, and remember that Tor is not simple and easy like OpenBSD is.

Simply search and install tor and a good manager in nvcurse and gtk to admin it, we find in the classic port tree of OpenBSD:

```sh
$ pkg_info -Q tor arm
$ doas pkg_add -U tor arm
```
Arm is the the next Nyx.

We will use it like a foreground program, without launch it from rc. In this first and simple configuration it will act like a SOCKS proxy, configure only some kind of nation like good jump in our tor path to the Internet and little more.
We will use Arm with the same preinstaller _tor user that use OpenBSD package. So:
```sh
$ doas chown -R _tor ~/.arm
# cat > ~/.arm/.torrc <<EOF
DataDirectory /home/taglio/.arm/tor_data # location to store runtime data
Log notice file /home/taglio/.arm/tor_log # location to log notices, warnings, and errors
ControlPort 9051 # port controllers can connect to
CookieAuthentication 1 # method for controller authentication
ExitNodes {RO},{CH}
ExcludeNodes {AU},{CA},{US},{NZ},{GB}
EntryNodes {BE},{DE},{IT},{NL}
StrictNodes 1
SocksPort 9900
EOF
```
We're now interested in initialize arm configure and explain what the hell are those `{}`

- [https://en.wikipedia.org/wiki/UKUSA_Agreement](https://en.wikipedia.org/wiki/UKUSA_Agreement)
- [https://www.privacytools.io/#ukusa](https://www.privacytools.io/#ukusa)

Also the acronym is obscure: `UKUSA`.
Read with your eyes what the hell is this democratic right abuse and masochism. Here up in those links.
So in `torrc`  we can indicate to our Tor how to don't pass through one nation that remiss to "The Fourteen Eyes".  
I use to enter in this example Belgium or Deutschland or Italy or Holland, and after an unknown jump i will leave the Tor network passing or for Romania or for Swiss.

No only the correct launch for the same Tor and we can go to sleep :P
```sh
$doas -u _tor tor -f toorrc
$doas -u _tor arm
```
Explore the deepest that you can, but remember the more deep you arrive more pressure and less oxygen you will find.

*good night*. 
*Gute Nacht.*


