```sh
taglio@cyberdream /mnt/vm-repo/OpenBSD $ wget https://ftp.heanet.ie/pub/OpenBSD/6.2/amd64/cd62.iso
--2018-03-08 15:32:49--  https://ftp.heanet.ie/pub/OpenBSD/6.2/amd64/cd62.iso
Resolving ftp.heanet.ie... 193.1.193.64, 2001:770:18:aa40::c101:c140
Connecting to ftp.heanet.ie|193.1.193.64|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 9953280 (9.5M) [application/octet-stream]
Saving to: ‘cd62.iso’

cd62.iso            100%[===================>]   9.49M  17.6MB/s    in 0.5s    

2018-03-08 15:32:50 (17.6 MB/s) - ‘cd62.iso’ saved [9953280/9953280]

taglio@cyberdream /mnt/vm-repo/OpenBSD $ 
taglio@cyberdream /mnt/vm-repo/OpenBSD $ wget https://ftp.heanet.ie/pub/OpenBSD/6.2/amd64/SHA256{,.sig}
--2018-03-08 15:33:02--  https://ftp.heanet.ie/pub/OpenBSD/6.2/amd64/SHA256
Resolving ftp.heanet.ie... 193.1.193.64, 2001:770:18:aa40::c101:c140
Connecting to ftp.heanet.ie|193.1.193.64|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1989 (1.9K)
Saving to: ‘SHA256’

SHA256              100%[===================>]   1.94K  --.-KB/s    in 0s      

2018-03-08 15:33:03 (256 MB/s) - ‘SHA256’ saved [1989/1989]

--2018-03-08 15:33:03--  https://ftp.heanet.ie/pub/OpenBSD/6.2/amd64/SHA256.sig
Reusing existing connection to ftp.heanet.ie:443.
HTTP request sent, awaiting response... 200 OK
Length: 2152 (2.1K) [application/pgp-signature]
Saving to: ‘SHA256.sig’

SHA256.sig          100%[===================>]   2.10K  --.-KB/s    in 0s      

2018-03-08 15:33:03 (233 MB/s) - ‘SHA256.sig’ saved [2152/2152]

FINISHED --2018-03-08 15:33:03--
Total wall clock time: 0.7s
Downloaded: 2 files, 4.0K in 0s (243 MB/s)
taglio@cyberdream /mnt/vm-repo/OpenBSD $ 
taglio@cyberdream /mnt/vm-repo/OpenBSD $ sha256sum --check SHA256 | grep cd62.iso 
sha256sum: BOOTIA32.EFI: No such file or directory
sha256sum: BOOTX64.EFI: No such file or directory
sha256sum: BUILDINFO: No such file or directory
sha256sum: INSTALL.amd64: No such file or directory
sha256sum: base62.tgz: No such file or directory
sha256sum: bsd: No such file or directory
sha256sum: bsd.mp: No such file or directory
sha256sum: bsd.rd: No such file or directory
sha256sum: cdboot: No such file or directory
sha256sum: cdbr: No such file or directory
sha256sum: comp62.tgz: No such file or directory
sha256sum: floppy62.fs: No such file or directory
cd62.iso: OK
sha256sum: game62.tgz: No such file or directory
sha256sum: install62.fs: No such file or directory
sha256sum: install62.iso: No such file or directory
sha256sum: man62.tgz: No such file or directory
sha256sum: miniroot62.fs: No such file or directory
sha256sum: pxeboot: No such file or directory
sha256sum: xbase62.tgz: No such file or directory
sha256sum: xfont62.tgz: No such file or directory
sha256sum: xserv62.tgz: No such file or directory
sha256sum: xshare62.tgz: No such file or directory
sha256sum: WARNING: 22 listed files could not be read
taglio@cyberdream /mnt/vm-repo/OpenBSD $ 
cyberdream /etc/openvswitch # ovsdb-tool create /etc/openvswitch/conf.db /usr/share/openvswitch/vswitch.ovsschema
# Socket for bringing the server up
DB_SOCKET="/var/run/openvswitch/db.sock"

# Remote sockets are defined in the database by default
REMOTE_DB="db:Open_vSwitch,Open_vSwitch,manager_options"

# All certificates and keys are stored in the database (if any)
PRIVATE_KEY="db:Open_vSwitch,SSL,private_key"
CERTIFICATE="db:Open_vSwitch,SSL,certificate"
BOOTSTRAP_CA_CERT="db:Open_vSwitch,SSL,ca_cert"

# Alternative path for the database (default is /etc/openvswitch/conf.db)
DATABASE="/etc/openvswitch/conf.db"

# Additional options
OPTIONS="--pidfile --detach"
cyberdream /etc/conf.d # ovs-vsctl --no-wait init
	
```

