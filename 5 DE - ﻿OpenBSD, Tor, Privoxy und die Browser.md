## Privoxy, fügt dem Kampf einen neuen Freund hinzu
Also, wir haben jetzt drei Browser, die drei verschiedene Socken-Ports verwenden, um auf das Tor-Netzwerk zuzugreifen, und der letzte unserer Browser baut eine spezielle Doppelschaltung im Tor-Netzwerk, die verschiedene Länder miteinander verbindet (*und das ist überhaupt nicht schlecht*). Denken Sie daran, wir waren [hier](https://steemit.com/openbsd/@npna/openbsd-tor-14-applications-example-part-1).
Aber … die Situation ist so schwierig … wir müssen uns mit mehr Schichten schützen.
Lassen Sie uns die Schicht 7 Firewall **Privoxy** vorstellen.
Was ist eine Schicht 7 Firewall?
Es ist eine Firewall, die auf der letzten Schicht des ISO/OSI-Netzwerkstacks arbeitet, das sprechen auf nicht-technische Art die URI ist (www.facebook.com, steemit.com/@npna) .
## Die Matroschka sind sexy
![Russian Matrioska](https://i.pinimg.com/736x/be/bb/32/bebb32d9f52fc9c1d8f34e6f0c3b1e70.jpg)
*Wir mögen Russland*, und wir mögen матрёшка.
In diesem Fall tun wir dasselbe, indem wir Schicht für Schicht unsere Informationen einkapseln, um sie vor den Sicherheit *Kannibalen zu schützen*.
Unsere Puppen starten vom Navigator, gehen durch Privoxy und werden dann zum Tor-Netzwerk weitergeleitet.
Lassen Sie Privoxy von den vorkompilierten OpenBSD-Paketen mit einem einfachen Befehl installieren:
```sh
$ doas pkg_add -U privoxy
```
Es wird diese Dateien und Binärdateien installieren:
```sh
$ pkg_info -L privoxy
Information for inst:privoxy-3.0.26
Files:
/usr/local/bin/privoxy-log-parser.pl
/usr/local/bin/privoxy-regression-test.pl
/usr/local/bin/uagen.pl
/usr/local/bin/url-pattern-translator.pl
/usr/local/man/man1/privoxy.1
/usr/local/sbin/privoxy
/usr/local/share/doc/privoxy/AUTHORS
/usr/local/share/doc/privoxy/ChangeLog
/usr/local/share/doc/privoxy/LICENSE
/usr/local/share/doc/privoxy/README
/usr/local/share/examples/privoxy/config
/usr/local/share/examples/privoxy/default.action
/usr/local/share/examples/privoxy/default.filter
/usr/local/share/examples/privoxy/match-all.action
/usr/local/share/examples/privoxy/regression-tests.action
/usr/local/share/examples/privoxy/templates/blocked
/usr/local/share/examples/privoxy/templates/cgi-error-404
/usr/local/share/examples/privoxy/templates/cgi-error-bad-param
/usr/local/share/examples/privoxy/templates/cgi-error-disabled
/usr/local/share/examples/privoxy/templates/cgi-error-file
/usr/local/share/examples/privoxy/templates/cgi-error-file-read-only
/usr/local/share/examples/privoxy/templates/cgi-error-modified
/usr/local/share/examples/privoxy/templates/cgi-error-parse
/usr/local/share/examples/privoxy/templates/cgi-style.css
/usr/local/share/examples/privoxy/templates/client-tags
/usr/local/share/examples/privoxy/templates/connect-failed
/usr/local/share/examples/privoxy/templates/connection-timeout
/usr/local/share/examples/privoxy/templates/default
/usr/local/share/examples/privoxy/templates/edit-actions-add-url-form
/usr/local/share/examples/privoxy/templates/edit-actions-for-url
/usr/local/share/examples/privoxy/templates/edit-actions-for-url-filter
/usr/local/share/examples/privoxy/templates/edit-actions-list
/usr/local/share/examples/privoxy/templates/edit-actions-list-button
/usr/local/share/examples/privoxy/templates/edit-actions-list-section
/usr/local/share/examples/privoxy/templates/edit-actions-list-url
/usr/local/share/examples/privoxy/templates/edit-actions-remove-url-form
/usr/local/share/examples/privoxy/templates/edit-actions-url-form
/usr/local/share/examples/privoxy/templates/forwarding-failed
/usr/local/share/examples/privoxy/templates/mod-local-help
/usr/local/share/examples/privoxy/templates/mod-support-and-service
/usr/local/share/examples/privoxy/templates/mod-title
/usr/local/share/examples/privoxy/templates/mod-unstable-warning
/usr/local/share/examples/privoxy/templates/no-server-data
/usr/local/share/examples/privoxy/templates/no-such-domain
/usr/local/share/examples/privoxy/templates/show-request
/usr/local/share/examples/privoxy/templates/show-status
/usr/local/share/examples/privoxy/templates/show-status-file
/usr/local/share/examples/privoxy/templates/show-url-info
/usr/local/share/examples/privoxy/templates/show-version
/usr/local/share/examples/privoxy/templates/toggle
/usr/local/share/examples/privoxy/templates/toggle-mini
/usr/local/share/examples/privoxy/templates/untrusted
/usr/local/share/examples/privoxy/templates/url-info-osd.xml
/usr/local/share/examples/privoxy/user.action
/usr/local/share/examples/privoxy/user.filter
/etc/rc.d/privoxy
```
Wir haben nur ein kleines Problem mit Privoxy in Bezug auf Tor: um drei Privoxy-Ports mit drei Tor-Ports zu verketten, müssen wir drei verschiedene Privoxy-Instanzen starten. Aber mit OpenBSD ist das, dank seiner Klarheit, sehr einfach.
Lass uns **tief** in die Konfiguration gehen:
* ändern Sie das Privoxy-Konfigurationsverzeichnis:
`$ cd /etc/privoxy`
* kopieren Sie die Standardkonfigurationsdatei in drei verschiedene:
`$ doas cp config firefox && doas cp config chrome && doas cp config torbrowser`
* gehen Sie zum OpenBSD rc.d Verzeichnis:
`$ cd /etc/rc.d`
* kopieren Sie das Standard-Privoxy-Initialskript in drei verschiedene:
`$ doas cp privoxy privoxyfirefox && doas cp privoxy privoxychrome && doas cp privoxy privoxytorbrowser`

Ok, alles ist einfach und ohne Komplikationen (*ich liebe OpenBSD*).
Die Pivoxy Hauptkonfigurationsdatei ist voll von Optionen und Sie müssen ein paar Stunden widmen, um alle Stimmen zu verstehen oder sie einfach zu lesen.
Fürs Erste verwenden wir diese Grundkonfiguration (in unseren nächsten Kapiteln werden wir wahrscheinlich einige Änderungen daran vornehmen).
Lassen Sie uns zusammen sehen, welches in der Firefox-Umgebung verwendet wird:
```sh
$ cat /etc/privoxy/firefox
#
#$Id: config,v 1.112 2016/08/26 13:14:18 fabiankeil Exp $
#Copyright (C) 2001-2016 Privoxy Developers https://www.privoxy.org/
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
logfile privoxyfirefox.log
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
listen-address  127.0.0.1:8800
#filter mode
toggle  1
enable-remote-toggle  0
#filter by X-filter http header
enable-remote-http-toggle  0
enable-edit-actions 0
enforce-blocks 1
#      src_addr[:port][/src_masklen] [dst_addr[:port][/dst_masklen]]
permit-access  127.0.0.1
buffer-limit 8192
#enable if there's a parent proxy
enable-proxy-authentication-forwarding 0
forward-socks5 / 127.0.0.1:9900 .
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
```
Ok, ändere `einfachadmin-address` und `hostname` bei Ihnen.
Privoxy sendet in jeder Konfigurationsdatei den  `http proxy port` an einem separeten `socks port`.
So erstellen Sie die anderen zwei, führen Sie diesen Befehl ein:
 ```sh
 # sed s/privoxyfirefox/privoxychrome/g privoxyfirefox | sed s/9900/9901/g > privoxychrome
 # sed s/privoxychrome/privoxytorbrowser/g privoxychrome | sed s/9901/9902/g > privoxytorbrowser    
 ```
 Erstellen Sie nun die fehlenden Protokolldateien mit:
  ```sh
 $ doas touch /var/log/privoxy/privoxyfirefox.log
 $ doas touch /var/log/privoxy/privoxychrome.log
 $ doas touch /var/log/privoxy/privoxytorbrowser.log
 ```
 Und aktivieren Sie alle drei beim Booten:
  ```sh
$ doas rcctl enable privoxyfirefox
$ doas rcctl set privoxyfirefox user _privoxy
$ doas rcctl set privoxyfirefox flags /etc/privoxy/firefox
$ doas rcctl enable privoxychrome
$ doas rcctl set privoxychrome user _privoxy
$ doas rcctl set privoxychrome flags /etc/privoxy/chrome
$ doas rcctl enable privoxytorbrowser
$ doas rcctl set privoxytorbrowser user _privoxy
$ doas rcctl set privoxytorbrowser flags /etc/privoxy/torbrowser
 ```
 Endlich starten die drei Dämonen:
  ```sh
 $ doas rcctl start privoxyfirefox
 $ doas rcctl start privoxychrome
 $ doas rcctl start privoxytorbrowser
 ```
 ## Browserkonfigurationen
 ![browsers configuration with privoxy](http://img3.wikia.nocookie.net/__cb20101231170457/mario/images/a/a7/Bowser_Super_Mario_Galaxy_2.png)
 Jetzt haben wir die richtige Einstellung von Privoxy, aber ohne irgendeine Regel der Schicht 7 Firewall, das werden wir in unserem nächsten Kapitel sehen.
 Wir müssen die Konfiguration der drei Browser ändern, um Privoxy zu verwenden, und nicht direkt die Tor Socken.
 1. Firefox:
![Firefox privoxy](https://lh5.googleusercontent.com/ddHo9wwk1dcYnLeQeWPgMhpGFrBkLanlsxzJo6jY8BXoGsWErsVLarkteptscKp73hb-p1Z82pZ6cLyfMHce=w1366-h650)
2. Chrome: simply change `--proxy-server="socks5://127.0.0.1:9901"`with `-proxy-server="http://127.0.0.1:8801"`
3. Torbrowser:
![Tor browser privoxy](https://lh3.googleusercontent.com/PjC3o3jzjdiJ6l_CFcEHIO9CRCJ-WXtbUxFYw2lhXSOp9J4pqk3MHCJNfETl9pxrug3PipjsvJ34VIUCdbb4=w1366-h650)
## Zubereitung der Hühnersuppe
Ok, das Kochen ist gestartet, aber ein letzter kleiner `hack`, um unsere liebste *Kannibalen* zu stören. Beseitigen das `HTTP header`  **[Refer](https://en.wikipedia.org/wiki/HTTP_referer)**:
```sh
# cat  >> /etc/privoxy/user.filter << EOF
{ +crunch-client-header{Referer:} }
/
EOF
```
Und ja, für heute ist es vorbei.
*Danke*, und #changeNOW bitte.
