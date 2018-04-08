## Hintergrund
![The tor project](https://steemit-production-imageproxy-thumbnail.s3.amazonaws.com/U5dspQfZY1SDxDd11MFJysVBehiJk1V_1680x8400)
Nur für den Fall dass Sie unseren letzten Artikel verpasst haben, wir sprechen über OpenBSD, das sicherste Open-Source-System im der Welt, und Tor, eine kostenlose Software für anonyme Kommunikation.
In unserem [ersten Post](https://steemit.com/openbsd/@npna/de-openbsd-tor-und-the-fourteen-eyes) wir haben eine Einführung und eine erste Konfiguration gemacht, die einen Tor-Service mit nur einem lokalen Socks-Port für eine statische Benutzerkonfiguration erstellt.
Wir haben auch das Konzept von [FVEY](https://en.wikipedia.org/wiki/Five_Eyes) eingeführt, das wir so schnell wie möglich analysieren werden.
Was wir beim letzten Mal nicht berührten ist die wichtige Tatsache dass diese zwei fabelhaften Softwareprogramme eine merkwürdige schwierige Geschichte der Interoperabilität zwischen ihnen haben. OpenBSD ist in unserem Leben von Mitte der Neunzehn und Tor, das Zwiebelrouterprojekt, ab September 2002.
Aber erst mit der Geburt des [torbsd Projekt](https://torbsd.github.io/) ist die Konfiguration des Anonymisierungs-Daemon so einfach und einfach wie es sein muss. Wir können lesen und studieren die Arbeit der Torbsd Gefährten bei [github](https://github.com/torbsd/).
## Tor Daemon in einem OpenBSD System
Ausgehend von einer sauberen OpenBSD Installation, wir werden Tor aus Paketen installieren:
```sh
$ doas pkg_add -U tor
$ doas pkg_add -U arm # ncurse control tool
```
Um zu sehen welche Dateien mit der Installation dieser beiden Pakete zu Ihrem Betriebssystem hinzugefügt wurden, verwerten Sie einfach (Paket Tor in dem Beispiel):
```sh
$ pkg_info -L tor
Information for inst:tor-0.3.0.10
Files:
/usr/local/bin/tor
/usr/local/bin/tor-gencert
/usr/local/bin/tor-resolve
/usr/local/man/man1/tor-gencert.1
/usr/local/man/man1/tor-resolve.1
/usr/local/man/man1/tor.1
/usr/local/share/doc/tor/tor-gencert.html
/usr/local/share/doc/tor/tor-resolve.html
/usr/local/share/doc/tor/tor.html
/usr/local/share/examples/tor/torrc.sample
/usr/local/share/tor/geoip
/usr/local/share/tor/geoip6
/etc/rc.d/tor
```
## Die torrc Datei

![torrc](https://steemit-production-imageproxy-thumbnail.s3.amazonaws.com/U5dr3fEGyxzzBTGcJVfT4mPb3ZsiopV_1680x8400)
```sh
# cat > /etc/tor/torrc <<EOF
User _tor
RunAsDaemon 1
AvoidDiskWrites 1
GeoIPFile /usr/local/share/tor/geoip
GeoIPv6File /usr/local/share/tor/geoip6
VirtualAddrNetwork 10.192.0.0/10
AutomapHostsOnResolve 1
TransPort 127.0.0.1:9040
DNSPort 127.0.0.1:53
DataDirectory /var/tor 
Log notice file /var/log/tor_log 
ControlPort 127.0.0.1:9051           
CookieAuthentication 1        
ExcludeNodes {AU}, {CA}, {US}, {NZ}, {GB}, {DK}, {FR}, {NL}, {NO}, {BE}, {DE}, {IT}, {ES}, {SE}
NodeFamily {AU}, {CA}, {US}, {NZ}, {GB}, {DK}, {FR}, {NL}, {NO}, {BE}, {DE}, {IT}, {ES}, {SE}
StrictNodes 1
GeoIPExcludeUnknown 1
SocksPort 127.0.0.1:9900
PathsNeededToBuildCircuits 0.95
EOF
```
Bereite die Umwelt vor:
```sh
$ doas mkdir /var/tor
$ doas chown -R _tor:_tor /var/tor
$ doas chown _tor:_tor /dev/pf
$ doas touch /var/log/tor_log
$ doas chown _tor:_tor /var/log/tor_log
```
Lassen Sie uns jede Option im torrc erklären:
1. **User**: nach dem Öffnen der Sockets arbeitet der Daemon unter der UID.
2. **RunAsDaemon**: um den Daemon im Hintergrund zu starten oder nicht.
3. **AvoidDiskWrites**: zu versuchen, weniger häufig auf die Festplatte zu schreiben.
4. **GeoIpFile**: wo ist in der fs Struktur.
5. **GeoIpv6File**: wo ist in der fs Struktur.
6. **VirtualAddrNetwork**: *später zu erklären*.
7. **AutomapHostsOnResolve**: es steuert VirtualAddrNetwork.
8. **TransPort**: transparenter Proxy-Port, an dem tor mit pf kommuniziert.
9. **DnsPort**: Port, an dem tor dns Resolver Anfragen annimmt.
10. **DataDirectory**: wo tor seine Session-Sachen ablegt.
11. **Log notice file**: Protokolldatei (*Ich weiß nicht, warum hier Leerzeichen akzeptiert werden*).
12. **ControlPort**: Port, an dem sich ARM oder andere, um Tor controllieren, verbinden müssen.
13. **CookieAuthentication**: bool, um den Authentifizierungsmodus im Steuerport anzugeben.
14. **ExcludeNodes**: wo wir, in unsere Kreisläufe mit ISO 3166 Ländercode, gehen wollen nicht.
15. **NodeFamily**: baue eine einzigartige Familie mit diesen Codes.
16. **StrictNodes**: beachten Sie unbedingt unsere ExcludeNodes Liste.
17. **GeoIPExcludeUnknown**: wenn Sie nicht wissen, wo das Tor relais ist, benutzen Sie es einfach nicht.
18. **SocksPort**: static port socks 4/5 listener (*wird später vertieft*).
19. **PathsNeededToBuildCircuits**: Tor wird keine Schaltkreise aufbauen, bis es genug Deskriptoren oder Mikrodeskriptoren hat, um diesen Bruchteil möglicher Wege zu konstruieren.

Jetzt müssen wir sicher sein, dass `dhclient` nicht umgeschrieben wird `/etc/resolv.conf`. In OpenBSD müssen wir dies zu `/etc/dhcpclient.conf` hinzufügen:
```sh
# cat > /etc/dhclient.conf <<EOF
supersede domain-name-servers 127.0.0.1;
EOF
$ doas sh /etc/netstart
```
## Die Datei pf.conf

![OpenBSD PF](https://steemit-production-imageproxy-thumbnail.s3.amazonaws.com/U5drwKeqXcRmMG6dc3NLBP26SF4WCN7_1680x8400)
OpenBSD war das erste System, das eine der leistungsfähigsten Firewalls verwendet, pf.
In anderen POSTs werden wir besser analysieren, wie man dieses Monster richtig benutzt, aber im Moment einfach diese `pf.conf` benutzen Sie, um eine transparente Firewall in einem OpenBSD System zu erstellen, das neuer als die Version 4.7.
Wir erstellen eine weitere Loopback-Schnittstelle in unserem System, um ein wenig mit dem internen Routing zu spielen:
```sh
$ doas ifconfig lo1 create up 127.0.0.2
# cat > /etc/hostname.lo1 <<EOF
inet 127.0.0.2 
EOF
```
Und verwende diese Direktiven in unserer `pf.conf`:
```sh
# cat > /etc/pf.conf <<EOF
# destinations you don't want routed through Tor
non_tor = "{ 10.0.0.0/8 172.16.0.0/12 192.168.0.0/16 }"
# Tor's TransPort
trans_port = "9040"
match in all scrub (no-df random-id reassemble tcp) antispoof for egress inet block return log on egress all
pass in quick on lo1 inet proto tcp all flags S/SA modulate state rdr-to 127.0.0.1 port $trans_port
pass in quick on lo1 inet proto udp to port domain rdr-to 127.0.0.1 port domain
pass quick on { lo0 lo1 }
pass out quick inet proto tcp user _tor flags S/SA modulate state
pass out quick inet proto udp to port domain route-to lo1
pass out quick inet to $non_tor
pass out inet proto tcp all route-to lo1
EOF
```
Hier sind einige Beispiele zur Verwendung der PF-Firewall:
- `pfctl -e` (aktivieren)
- `pfctl -d` (deaktivieren)
- `pfctl -f /etc/pf.conf` (lade Regeln)
- `pfctl -nf /etc/pf.conf` (Datei analysieren, nicht laden)
- `pfctl -sr` (zeige den aktuellen Regelsatz)
- `pfctl -ss` (zeige aktuelle Zustandstabelle an)
- `pfctl -si` (zeige Filterstatistiken und Zähler an)
- `pftcl -sa` (zeige alles)
- `pfctl -t table -T flush` (Tisch zu spülen)
- `pfctl -k 192.168.1.80` (lösche Verbindungen für host 80)
- `pfctl -t -T expire 86400 -flush table` (Einträge in den letzten 24 Stunden hinzugefügt)
- füge `-vv` für weitere Informationen hinzu

## Start Tor beim Booten

Die letzte Konfiguration besteht darin, Tor beim Start zu aktivieren.
*Verwenden Sie diesen Daemon, um Ihre Privatsphäre zu schützen oder um auf das #deepinternet zuzugreifen?*
```sh
$ doas rcctl enable tor
$ doas reboot
```
Bleib dran für mehr OpenBSD-, Tor-, und Deepinternet-Posts, ich *liebe dich*.
