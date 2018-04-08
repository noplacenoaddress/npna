## Fourteen application, the first three are browsers
First of all sorry for the delay, but i was busy with real life and i cannot go on with our series.
Let's start to analyze every step necessary to configure use of the TCP/IP socks sockets that we opened in our [last article](https://steemit.com/openbsd/@npna/openbsd-tor-and-the-streams-isolation).

## Firefox

![Firefox tor sockv5 configuration](https://lh6.googleusercontent.com/NQheoWsSqyJsFDMzaCTnnCHlQKu0A27zoxVr1XakuBW7ck_bECi-bMOeb5oWNbm_DNjHUR2Umo_IYbT7E5DS=w1366-h650)

Like we can see in the image:
1. open `about:preferences#advanced`
2. click on `network` tab
3. click on `Settings`
4. check `Manual proxy configuration`
5. write on `SOCKS Host`the local ip `127.0.0.1` and `Port` `9900`
6. write on `No proxy for` `localhost, 127.0.0.1, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16` that are privates network as [RFC1918](https://tools.ietf.org/html/rfc1918).
## Chromium
As default chromium has no simple option to set a proxy server different from the system wide proxy. But we've done a little *hack*.
```sh
$ which chrome
/usr/local/bin/chrome
$ file /usr/local/bin/chrome
/usr/local/bin/chrome: Bourne shell script text executable
```
So here we can appreciate that chrome in our **OpenBSD workstation** is not an executable but is a shell script, here is a cat:
```sh
·$ cat /usr/local/bin/chrome
#!/bin/sh
# $OpenBSD: chrome,v 1.14 2016/06/02 21:03:38 sthen Exp $

DATASIZE="716800"
OPENFILES="400"

xm_log() {
        echo -n "$@\nDo you want to run Chromium anyway?\n\
(If you don't increase these limits, Chromium might fail to work properly.)" | \
                /usr/X11R6/bin/xmessage -file - -center -buttons yes:0,no:1 -default no
}

if [ $(ulimit -Sd) -lt ${DATASIZE} ]; then
        ulimit -Sd ${DATASIZE} || \
                xm_log "Cannot increase datasize-cur to at least ${DATASIZE}"
                [ $? -eq 0 ] || exit
fi

if [ $(ulimit -Sn) -lt ${OPENFILES} ]; then
        ulimit -Sn ${OPENFILES} || \
                xm_log "Cannot increase openfiles-cur to at least ${OPENFILES}"
                [ $? -eq 0 ] || exit
fi

if ! mount | grep `df -h /usr/local | tail -1 | awk '{print $6}'` | 
        grep -q wxallowed; then 
        echo "Filesystem containing /usr/local must have the 'wxallowed' flag" | 
            /usr/X11R6/bin/xmessage -file - -center -buttons exit:0 -default exit
        exit
fi

#
# Temporary workaround for the case when chromium crashes and leaves
# the SingletonLock, which prevents chromium to start up.
#
if [ -h ${HOME}/.config/chromium/SingletonLock ]; then
        _pid=`readlink ${HOME}/.config/chromium/SingletonLock | cut -d '-' -f 2`
        kill -0 ${_pid} 2>/dev/null
        if [ $? -gt 0 ]; then
                rm ${HOME}/.config/chromium/SingletonLock
        fi
fi

#
# Issue #395446
# https://code.google.com/p/chromium/issues/detail?id=395446
#
[ -z ${LANG} ] && _l=en_US.UTF-8 || _l=${LANG}

LANG=${_l} exec "/usr/local/chrome/chrome" "${@}" "--proxy-server="socks5://127.0.0.1:9901""
```  
How you can see int last line of the script there is the real `exec`of the binary that in OpenBSD is located in `/usr/local/chrome/chrome` .
Simply *concat* in the same line of the var LANG the string `"--proxy-server="socks5://127.0.0.1:9901"`and close the sentence with a `"`that we've previously remove from the old sentence (the last one that you will find doing a `cat`). You can appreciate that for `chrome`we use socksv5 port `9901`.
 ## Tor browser
 Tor browser is an open source fork of Firefox. It's maintained by the torproject folks. You can find the binaries [here](https://github.com/TheTorProject/gettorbrowser.git) in github. But remember that we're using OpenBSD and the Tor browser bundle is available in the ports tree. To download the binary for OpenBSD do:
 ```sh
 $ doas pkg_add -U tor-browser
 ```
 But `tor-browser`have got a `tor daemon` included that control via the `tor button` that we can launch via the onion icon on the left of the navigation bar. The first time that we execute this browser simply accept the default settings waiting to `connect`on the window that appear. But next:
 
![Tor browser onion button first window](https://lh3.googleusercontent.com/XGPNeUdWVWcW5rBkQGTl6SJwWKYUKLOwuxbX8elxmpVY3niDWIleBQMaSGmjp4Z-s5LM9LKH9nGYrH48Zvzu=w1366-h650)

This is the first subwindow if we click on the `onion button`. Let's assume that we will use `tor browser`with the most deep navigation in the web. So click on `security settings` and:

![Tor browser security settings](https://lh4.googleusercontent.com/hSwsMxRqQGTJs1FxN28aZRSJrEhvvUkY7fDQaBHbw_flp4DUrhRKCYfnFb-_wdxDqEUdeGObrlJEpQjkG80Z=w1366-h650)

Use the `High` position to garantized the best shields for this *new adventure* .
Next return to the previous window and click on `Tor network settings`

![Tor browser network settings](https://lh3.googleusercontent.com/LjlF-MkOyxxe2UST5hbGMtYonduQf9ywlJMomEWst5MB0gFZBzjXG0tiCcuAliKSb1wQ8JpM7fB4ZBVOw-Df=w1366-h650)

1. Check `This computer need to use a local proxy to access the Internet`
2. Select for `Proxy Type`the option `SOCKS 5`
3. Write in `Address` `127.0.0.1` and in `Port` `9902`

Now...there's some post on the web that does not recommend concatenate **Tor over Tor**, like [this](https://trac.torproject.org/projects/tor/wiki/doc/TorifyHOWTO#ToroverTor); but there is no exact explication so i really don't think so.

Remember that we have banned **ALL** the fourteens eyes country in our [last](https://steemit.com/openbsd/@npna/openbsd-tor-and-the-streams-isolation) configuration. I quoted the explication from [privacytools.io](https://www.privacytools.io/) :

> The UKUSA Agreement is an agreement between the United Kingdom, United
> States, Australia, Canada, and New Zealand to cooperatively collect,
> analyze, and share intelligence. Members of this group, known as the
> Five Eyes, focus on gathering and analyzing intelligence from
> different parts of the world. While Five Eyes countries have agreed to
> not spy on each other as adversaries, leaks by Snowden have revealed
> that some Five Eyes members monitor each other’s citizens and share
> intelligence to avoid breaking domestic laws that prohibit them from
> spying on their own citizens. The Five Eyes alliance also cooperates
> with groups of third party countries to share intelligence (forming
> the Nine Eyes and Fourteen Eyes), however Five Eyes and third party
> countries can and do spy on each other.

In the configuration of the `tor daemon`that came with `tor browser`we force tor to use the countries of the **nine eyes group** that will be the first three hops of ours jumps in the tor network. Doing so we will use **six** hops to navigate the internet.
This are the files shipped with `tor launcher` (a subpaquet of `tor browser`):
```sh
$ pkg_info -L tor-launcher
Information for inst:tor-launcher-0.2.12.3p0

Files:
/usr/local/lib/tor-browser-7.0.5/browser/extensions/tor-launcher@torproject.org.xpi
/usr/local/share/tor-browser/torrc-defaults
```
Open `torrc-defaults` with your favorite text editor and add:
```sh
ExcludeNodes {AD},{AE},{AF},{AG},{AI},{AL},{AM},{AO},{AQ},{AR},{AS},{AT},{AU},{AW},{AX},{AZ},{BA},{BB},{BD},{BE},{BF},{BG},{BH},{BI},{BJ},{BL},{BM},{BN},{BO},{BQ},{BR},{BS},{BT},{BV},{BW},{BY},{BZ},{CA},{CC},{CD},{CF},{CG},{CH},{CI},{CK},{CL},{CM},{CN},{CO},{CR},{CU},{CV},{CW},{CX},{CY},{CZ},{DE},{DJ},{DM},{DO},{DZ},{EC},{EE},{EG},{EH},{ER},{ES},{ET},{FI},{FJ},{FK},{FM},{FO},{GA},{GB},{GD},{GE},{GF},{GG},{GH},{GI},{GL},{GM},{GN},{GP},{GQ},{GR},{GS},{GT},{GU},{GW},{GY},{HK},{HM},{HN},{HR},{HT},{HU},{ID},{IE},{IL},{IM},{IN},{IO},{IQ},{IR},{IS},{IT},{JE},{JM},{JO},{JP},{KE},{KG},{KH},{KI},{KM},{KN},{KP},{KR},{KW},{KY},{KZ},{LA},{LB},{LC},{LI},{LK},{LR},{LS},{LT},{LU},{LV},{LY},{MA},{MC},{MD},{ME},{MF},{MG},{MH},{MK},{ML},{MM},{MN},{MO},{MP},{MQ},{MR},{MS},{MT},{MU},{MV},{MW},{MX},{MY},{MZ},{NA},{NC},{NE},{NF},{NG},{NI},{NP},{NR},{NU},{NZ},{OM},{PA},{PE},{PF},{PG},{PH},{PK},{PL},{PM},{PN},{PR},{PS},{PT},{PW},{PY},{QA},{RE},{RO},{RS},{RU},{RW},{SA},{SB},{SC},{SD},{SE},{SG},{SH},{SI},{SJ},{SK},{SL},{SM},{SN},{SO},{SR},{SS},{ST},{SV},{SX},{SY},{SZ},{TC},{TD},{TF},{TG},{TH},{TJ},{TK},{TL},{TM},{TN},{TO},{TR},{TT},{TV},{TW},{TZ},{UA},{UG},{UM},{US},{UY},{UZ},{VA},{VC},{VE},{VG},{VI},{VN},{VU},{WF},{WS},{YE},{YT},{ZA},{ZM},{ZW}
NodeFamily {AD},{AE},{AF},{AG},{AI},{AL},{AM},{AO},{AQ},{AR},{AS},{AT},{AU},{AW},{AX},{AZ},{BA},{BB},{BD},{BE},{BF},{BG},{BH},{BI},{BJ},{BL},{BM},{BN},{BO},{BQ},{BR},{BS},{BT},{BV},{BW},{BY},{BZ},{CA},{CC},{CD},{CF},{CG},{CH},{CI},{CK},{CL},{CM},{CN},{CO},{CR},{CU},{CV},{CW},{CX},{CY},{CZ},{DE},{DJ},{DM},{DO},{DZ},{EC},{EE},{EG},{EH},{ER},{ES},{ET},{FI},{FJ},{FK},{FM},{FO},{GA},{GB},{GD},{GE},{GF},{GG},{GH},{GI},{GL},{GM},{GN},{GP},{GQ},{GR},{GS},{GT},{GU},{GW},{GY},{HK},{HM},{HN},{HR},{HT},{HU},{ID},{IE},{IL},{IM},{IN},{IO},{IQ},{IR},{IS},{IT},{JE},{JM},{JO},{JP},{KE},{KG},{KH},{KI},{KM},{KN},{KP},{KR},{KW},{KY},{KZ},{LA},{LB},{LC},{LI},{LK},{LR},{LS},{LT},{LU},{LV},{LY},{MA},{MC},{MD},{ME},{MF},{MG},{MH},{MK},{ML},{MM},{MN},{MO},{MP},{MQ},{MR},{MS},{MT},{MU},{MV},{MW},{MX},{MY},{MZ},{NA},{NC},{NE},{NF},{NG},{NI},{NP},{NR},{NU},{NZ},{OM},{PA},{PE},{PF},{PG},{PH},{PK},{PL},{PM},{PN},{PR},{PS},{PT},{PW},{PY},{QA},{RE},{RO},{RS},{RU},{RW},{SA},{SB},{SC},{SD},{SE},{SG},{SH},{SI},{SJ},{SK},{SL},{SM},{SN},{SO},{SR},{SS},{ST},{SV},{SX},{SY},{SZ},{TC},{TD},{TF},{TG},{TH},{TJ},{TK},{TL},{TM},{TN},{TO},{TR},{TT},{TV},{TW},{TZ},{UA},{UG},{UM},{US},{UY},{UZ},{VA},{VC},{VE},{VG},{VI},{VN},{VU},{WF},{WS},{YE},{YT},{ZA},{ZM},{ZW}
StrictNodes 1
GeoIPExcludeUnknown 1
```
You find all the country codes **ISO 3166** in this [sheet](https://drive.google.com/open?id=1NBjLS5Mbf0jDYCdkJ_UHJXFeiPDyaUnaNecr9GkuP7w)

*thank you*
