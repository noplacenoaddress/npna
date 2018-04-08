## ﻿OpenBSD, Tor, 14 Anwendungsbeispiele - Erster Teil
Fangen wir an, jeden Schritt zu analysieren, der notwendig ist, um die Verwendung der TCP / IP-Sockets , die wir in unserem letzten Artikel geöffnet haben, zu konfigurieren.
## Firefox
![Firefox tor sockv5 configuration](https://lh6.googleusercontent.com/NQheoWsSqyJsFDMzaCTnnCHlQKu0A27zoxVr1XakuBW7ck_bECi-bMOeb5oWNbm_DNjHUR2Umo_IYbT7E5DS=w1366-h650)
Wie wir auf dem Bild oben sehen können:
1. öffnen `about:preferences#advanced`
2. klicke auf `network` tab
3. klicke auf `Settings`
4. checken `Manual proxy configuration`
5. schreibe auf `SOCKS Host`: die lokale IP `127.0.0.1` und auf `Port` `9900`
6. schreibe auf `No proxy for` `localhost, 127.0.0.1, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16` das sind private Netzwerke als [RFC1918](https://tools.ietf.org/html/rfc1918).

## 2. Chromium
Standardmäßig hat Chrom keine einfache Möglichkeit, einen Proxy-Server anders als den systemweiten Proxy zu setzen. Aber wir haben einen kleinen Hack gemacht.
```sh
$ which chrome
/usr/local/bin/chrome
$ file /usr/local/bin/chrome
/usr/local/bin/chrome: Bourne shell script text executable
```
Also hier können Sie verstehen, dass Chrome in unserer **OpenBSD-Workstation** keine ausführbare Datei ist, sondern ein Shell-Skript, hier ist eine **cat**:
```sh
$ cat /usr/local/bin/chrome
#!/bin/sh 
#$OpenBSD: chrome,v 1.14 2016/06/02 21:03:38 sthen Exp $

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
#Temporary workaround for the case when chromium crashes and leaves
#the SingletonLock, which prevents chromium to start up.
#
if [ -h ${HOME}/.config/chromium/SingletonLock ]; then
       _pid=`readlink ${HOME}/.config/chromium/SingletonLock | cut -d '-' -f 2`
       kill -0 ${_pid} 2>/dev/null
       if [ $? -gt 0 ]; then
               rm ${HOME}/.config/chromium/SingletonLock
       fi
fi

#
#Issue #395446
#https://code.google.com/p/chromium/issues/detail?id=395446
#
[ -z ${LANG} ] && _l=en_US.UTF-8 || _l=${LANG}

LANG=${_l} exec "/usr/local/chrome/chrome" "${@}" "--proxy-server="socks5://127.0.0.1:9901""
```
Wie man in der letzten Zeile des Skripts sehen kann, gibt es die echte exec der Binärdatei, die sich in **OpenBSD** in `/usr/local/chrome/chrome` befindet.
Einfach in die selbe Zeile des var LANG die Zeichenfolge `"--proxy-server =" socks5: //127.0.0.1: 9901"` eingeben und schließe den Satz mit einem `"`, das wir vorher aus dem alten Satz entfernt haben (die letzte die Sie finden werden, wenn Sie eine `cat` machen).
Sie können schätzen, dass wir für `Chrome` socksv5 Port `9901` verwenden.
## Tor browser
Tor-Browser ist eine Open-Source-Fork von Firefox. Es wird von den Torproject-Leuten aufrechterhalten.
Sie können die Binärdateien [hier](https://github.com/TheTorProject/gettorbrowser.git)  in GitHub finden.
Aber denken Sie daran, dass wir OpenBSD verwenden und das Tor-Browser-Bundle in der Ports-Struktur verfügbar ist.
Um die Binärdatei für OpenBSD herunterzuladen, machen folgendes:
 ```sh
 $ doas pkg_add -U tor-browser
 ```
 Aber `tor-browser` haben einen `tor daemon`, der über den `tor button` gesteuert wird, den wir über das Zwiebel-Symbol auf der linken Seite der Navigationsleiste starten können.
Wenn Sie diesen Browser das erste Mal ausführen, akzeptieren Sie einfach die Standardeinstellungen, warten auf `connect` auf dem Fenster, das angezeigt wird. Aber als nächstes:
![Tor browser onion button first window](https://lh3.googleusercontent.com/XGPNeUdWVWcW5rBkQGTl6SJwWKYUKLOwuxbX8elxmpVY3niDWIleBQMaSGmjp4Z-s5LM9LKH9nGYrH48Zvzu=w1366-h650)
Dies ist das erste Unterfenster, wenn wir auf den `onion button` klicken. Angenommen, wir verwenden `tor browser` mit der tiefsten Navigation im Internet. Klicken Sie also auf `security setting` und:
![Tor browser security settings](https://lh4.googleusercontent.com/hSwsMxRqQGTJs1FxN28aZRSJrEhvvUkY7fDQaBHbw_flp4DUrhRKCYfnFb-_wdxDqEUdeGObrlJEpQjkG80Z=w1366-h650)
Nutze die Position `High`, um die besten Schilde für dieses neue Abenteuer zu garantieren.
Als nächstes kehren Sie zum vorherigen Fenster zurück und klicken auf `Tor network settings`:

![Tor browser network settings](https://lh3.googleusercontent.com/LjlF-MkOyxxe2UST5hbGMtYonduQf9ywlJMomEWst5MB0gFZBzjXG0tiCcuAliKSb1wQ8JpM7fB4ZBVOw-Df=w1366-h650)
1. Checken `This computer need to use a local proxy to access the Internet`
2. Wählen für `Proxy Type` die Option `SOCKS 5`
3. Schreiben in `Address` `127.0.0.1` und in Port `9902`

Nun … es gibt einen Beitrag im Internet, der nicht empfiehlt, **Tor über Tor** zu verketten, wie [dieser]((https://trac.torproject.org/projects/tor/wiki/doc/TorifyHOWTO#ToroverTor)); aber es gibt keine genaue Erklärung, also glaube ich das wirklich nicht.
Denken Sie daran, dass wir in unserer letzten Konfiguration alle Fourteen Eyes Länder [verboten](https://steemit.com/openbsd/@npna/openbsd-tor-and-the-streams-isolation) haben.
Ich zitiere die Erklärung von privacytools.io:
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

In der Konfiguration des `tor daemon`, der mit `tor browser` kam, wir haben Tor gezwungen, die Länder der **Nine Eyes Gruppe** zu benutzen, das sind die ersten drei Hops unserer Sprünge im Tor Netz. Dabei werden wir **sechs** Hops verwenden, um im Internet zu navigieren.
Dies sind die Dateien, die mit `tor launcher` (ein Subset von `tor browser)` geliefert werden:
```sh
$ pkg_info -L tor-launcher
Information for inst:tor-launcher-0.2.12.3p0

Files:
/usr/local/lib/tor-browser-7.0.5/browser/extensions/tor-launcher@torproject.org.xpi
/usr/local/share/tor-browser/torrc-defaults
```
Öffnen Sie `torrc-defaults` mit Ihrem bevorzugten Texteditor und fügen Sie hinzu:
```sh
ExcludeNodes {AD},{AE},{AF},{AG},{AI},{AL},{AM},{AO},{AQ},{AR},{AS},{AT},{AU},{AW},{AX},{AZ},{BA},{BB},{BD},{BE},{BF},{BG},{BH},{BI},{BJ},{BL},{BM},{BN},{BO},{BQ},{BR},{BS},{BT},{BV},{BW},{BY},{BZ},{CA},{CC},{CD},{CF},{CG},{CH},{CI},{CK},{CL},{CM},{CN},{CO},{CR},{CU},{CV},{CW},{CX},{CY},{CZ},{DE},{DJ},{DM},{DO},{DZ},{EC},{EE},{EG},{EH},{ER},{ES},{ET},{FI},{FJ},{FK},{FM},{FO},{GA},{GB},{GD},{GE},{GF},{GG},{GH},{GI},{GL},{GM},{GN},{GP},{GQ},{GR},{GS},{GT},{GU},{GW},{GY},{HK},{HM},{HN},{HR},{HT},{HU},{ID},{IE},{IL},{IM},{IN},{IO},{IQ},{IR},{IS},{IT},{JE},{JM},{JO},{JP},{KE},{KG},{KH},{KI},{KM},{KN},{KP},{KR},{KW},{KY},{KZ},{LA},{LB},{LC},{LI},{LK},{LR},{LS},{LT},{LU},{LV},{LY},{MA},{MC},{MD},{ME},{MF},{MG},{MH},{MK},{ML},{MM},{MN},{MO},{MP},{MQ},{MR},{MS},{MT},{MU},{MV},{MW},{MX},{MY},{MZ},{NA},{NC},{NE},{NF},{NG},{NI},{NP},{NR},{NU},{NZ},{OM},{PA},{PE},{PF},{PG},{PH},{PK},{PL},{PM},{PN},{PR},{PS},{PT},{PW},{PY},{QA},{RE},{RO},{RS},{RU},{RW},{SA},{SB},{SC},{SD},{SE},{SG},{SH},{SI},{SJ},{SK},{SL},{SM},{SN},{SO},{SR},{SS},{ST},{SV},{SX},{SY},{SZ},{TC},{TD},{TF},{TG},{TH},{TJ},{TK},{TL},{TM},{TN},{TO},{TR},{TT},{TV},{TW},{TZ},{UA},{UG},{UM},{US},{UY},{UZ},{VA},{VC},{VE},{VG},{VI},{VN},{VU},{WF},{WS},{YE},{YT},{ZA},{ZM},{ZW}
NodeFamily {AD},{AE},{AF},{AG},{AI},{AL},{AM},{AO},{AQ},{AR},{AS},{AT},{AU},{AW},{AX},{AZ},{BA},{BB},{BD},{BE},{BF},{BG},{BH},{BI},{BJ},{BL},{BM},{BN},{BO},{BQ},{BR},{BS},{BT},{BV},{BW},{BY},{BZ},{CA},{CC},{CD},{CF},{CG},{CH},{CI},{CK},{CL},{CM},{CN},{CO},{CR},{CU},{CV},{CW},{CX},{CY},{CZ},{DE},{DJ},{DM},{DO},{DZ},{EC},{EE},{EG},{EH},{ER},{ES},{ET},{FI},{FJ},{FK},{FM},{FO},{GA},{GB},{GD},{GE},{GF},{GG},{GH},{GI},{GL},{GM},{GN},{GP},{GQ},{GR},{GS},{GT},{GU},{GW},{GY},{HK},{HM},{HN},{HR},{HT},{HU},{ID},{IE},{IL},{IM},{IN},{IO},{IQ},{IR},{IS},{IT},{JE},{JM},{JO},{JP},{KE},{KG},{KH},{KI},{KM},{KN},{KP},{KR},{KW},{KY},{KZ},{LA},{LB},{LC},{LI},{LK},{LR},{LS},{LT},{LU},{LV},{LY},{MA},{MC},{MD},{ME},{MF},{MG},{MH},{MK},{ML},{MM},{MN},{MO},{MP},{MQ},{MR},{MS},{MT},{MU},{MV},{MW},{MX},{MY},{MZ},{NA},{NC},{NE},{NF},{NG},{NI},{NP},{NR},{NU},{NZ},{OM},{PA},{PE},{PF},{PG},{PH},{PK},{PL},{PM},{PN},{PR},{PS},{PT},{PW},{PY},{QA},{RE},{RO},{RS},{RU},{RW},{SA},{SB},{SC},{SD},{SE},{SG},{SH},{SI},{SJ},{SK},{SL},{SM},{SN},{SO},{SR},{SS},{ST},{SV},{SX},{SY},{SZ},{TC},{TD},{TF},{TG},{TH},{TJ},{TK},{TL},{TM},{TN},{TO},{TR},{TT},{TV},{TW},{TZ},{UA},{UG},{UM},{US},{UY},{UZ},{VA},{VC},{VE},{VG},{VI},{VN},{VU},{WF},{WS},{YE},{YT},{ZA},{ZM},{ZW}
StrictNodes 1
GeoIPExcludeUnknown 1
```
In diesem [Blatt](https://drive.google.com/open?id=1NBjLS5Mbf0jDYCdkJ_UHJXFeiPDyaUnaNecr9GkuP7w) finden Sie alle Ländercodes nach **ISO 3166**.

*Danke*.
