

## Stream Isolation?
![isolation](https://steemit-production-imageproxy-thumbnail.s3.amazonaws.com/U5dsbLPPV474ToZVayhHNhTLmarVTrF_1680x8400)
Wir sind also angekommen, um unseren Tor-Daemon auf eine Art und Weise zu konfigurieren, die alle TCP-Verbindungen unserer Workstation transparent “*torifiziert*”, mit einem pf wo es mit der Firewall von OpenBSD “*spricht*”.
*Erinnern Sie sich*? [Es war hier](https://steemit.com/openbsd/@npna/de-openbsd-tor-transparenter-lokaler-proxy), in unserem [Steemit](https://steemit.com/@npna).
Offensichtlich ist unsere Arbeit bisher als abgeschlossen anzusehen, und in der einen oder anderen Sache denke ich ernsthaft, dass wir daran arbeiten werden, die Privatsphäre der Internetnutzer den Rest unseres Lebens zu schützen, aber das ist eine *andere Geschichte*.
**Stream Isolation**… nicht viele Leute sprechen von *Isolation*, der menschliche Verstand hält es für eines der schlimmsten Dinge; und aufrichtig haben sie einen Grund, Isolation ist nicht amüsant.
Aber in der Datenprivatsphäre ist die Isolation der verschiedenen Datenströme **der Schlüssel**, es ist eine der vierzehn oder eine der bösen [Jungs Organisation](https://www.privacytools.io/#ukusa) (*gibt es Unterschiede*? … Nein, ich glaube wirklich nicht … **ich weiß es**).
Unser guter Freund **Tor**, *du bist ein Freund von mir, mein Lieber*, in einem etwas weniger dokumentierten Feature oder besser weniger publizierten Feature kann TCP / IP Streams **auf vier verschiedene Arten isolieren**. Wir wissen, dass eine Verbindung gekennzeichnet ist durch:
1.	source ip
2.	destination ip
3.	source port
4.	destination port

Wenn einige von euch, *liebe Freunde*, nicht wissen wie das TCP / IP-Protokoll funktioniert, ich kann Ihnen empfehlen, hier eine kleine Einführung in die berühmte Wikipedia [zu lesen](https://en.wikipedia.org/wiki/Transmission_Control_Protocol).
In der Praxis wird Tor für jede Verbindung, jede IP und jede Anwendung eine andere Route im Tor-Netzwerk zuweisen, abhängig von unserer Konfiguration in der `torrc` Datei.
## Hacking over torrc
Wir werden also die Konfiguration unseres `torrc `ändern, um die oben beschriebene **Stream Isolation** zu ermöglichen.
Hier finden Sie die neue Version, wie immer unter `/etc/tor/` gelegen:
```sh
User _tor
RunAsDaemon 1
AvoidDiskWrites 1
Log notice syslog
Log notice file /var/log/tor_log

DataDirectory /var/tor

ControlSocket /var/tor/control GroupWritable RelaxDirModeCheck
ControlSocketsGroupWritable 1
SocksPort unix:/var/tor/socks WorldWritable

ControlPort 127.0.0.1:9051 
CookieAuthentication 1
CookieAuthFileGroupReadable 1
CookieAuthFile /var/tor/control.authcookie

GeoIPFile /usr/local/share/tor/geoip
GeoIPv6File /usr/local/share/tor/geoip6

VirtualAddrNetwork 10.192.0.0/10
AutomapHostsOnResolve 1

ExcludeNodes {AU}, {CA}, {US}, {NZ}, {GB}, {DK}, {FR}, {NL}, {NO}, {BE}, {DE}, {IT}, {ES}, {SE}
NodeFamily {AU}, {CA}, {US}, {NZ}, {GB}, {DK}, {FR}, {NL}, {NO}, {BE}, {DE}, {IT}, {ES}, {SE}
StrictNodes 1
GeoIPExcludeUnknown 1
PathsNeededToBuildCircuits 0.95

DNSPort 127.0.0.1:53 IsolateDestPort

#GENERIC
TransPort 127.0.0.1:9040

#FIREFOX
SocksPort 127.0.0.1:9900
#CHROMIUM
SocksPort 127.0.0.1:9901
#TOR BROWSER
SocksPort 127.0.0.1:9902
#XCHAT
SocksPort 127.0.0.1:9903 IsolateDestAddr IsolateDestPort
#THUNDERBIRD + TORBIRDY
SocksPort 127.0.0.1:9904 IsolateDestAddr IsolateDestPort
#IM
SocksPort 127.0.0.1:9905 IsolateDestAddr IsolateDestPort
#PKG_ADD
SocksPort 127.0.0.1:9906
#KEYBASE
SocksPort 127.0.0.1:9907 IsolateDestAddr IsolateDestPort
#SSH
SocksPort 127.0.0.1:9908 IsolateDestAddr IsolateDestPort
#WGET
SocksPort 127.0.0.1:9909 IsolateDestAddr IsolateDestPort
#BITCOIN
SocksPort 127.0.0.1:9910
#PRIVOXY
SocksPort 127.0.0.1:9911
#POLIPO
SocksPort 127.0.0.1:9912
#GNOME wide proxy
SocksPort 127.0.0.1:9913
```
Prima, ausgehend von unserer letzten Konfiguration von `torrc` kann man viele Süchte sehen. Zuallererst müssen Sie erstellen  `/var/tor/control.authcookie` :
```sh
$ doas -u _tor touch /var/tor/control.authcookie
```
Zweitens können Sie verstehen, dass wir eine Serie von `SocksPort` mit verschiedenen Optionen für jede Anwendung deklarieren, die wir normalerweise auf unserer Workstation verwenden (sie sind die Mine, Sie können andere deklarieren).
Aber was genau machen wir?
Wir betrachten den transparenten Proxy-Port (`TransPort 127.0.0.1:9040`) wie unsere *letzten* Ressourcen, tatsächlich werden alle Anwendungen, die wir nicht an den anderen Ports deklarieren, im Tor-Netzwerk mit diesem Port verbunden.
Die anderen Anwendungen, die wir deklarieren, werden nach und nach konfiguriert, wenn sie Proxying mit Socks-Technologie unterstützen, oder für ein drittes Programm gekapselt werden. In unserem Beispiel werden wir **torsocks**, **provoxy** oder **polipo** verwenden.
Es gibt einen weiteren Unterschied, einige Anwendungen verwenden `IsolateDestAddr` andere `IsolateDestPort`.
Diese beiden Optionen, zwei der vierten die für uns bindend sind, erzeugen verschiedene Schaltungen für die gleiche Anwendung. In der Praxis werden wir mit der gleichen Anwendung an verschiedene IP-Adressen binden.
Mit diesen Tricks in nur einer Arbeitssitzung verwenden wir viele verschiedene Identifizierungen zur Außenwelt. Es ist offensichtlich, **dass wir auf diese Weise unsere Präsenz im Internet sehr gut verbergen**.
## Besonderen Dank
Ich möchte diesen Beitrag schließen, indem ich ein einfaches Dankeschön an die [Whonix-Crew](https://www.whonix.org/) sage, die viel gearbeitet und gute Tutorials über **Stream Isolation** veröffentlicht hat, die es auf eine gute und einfache Weise erklären.
*Danke*.


