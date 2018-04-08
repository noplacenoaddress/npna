<iframe width="560" height="315" src="https://www.youtube.com/embed/Zf1VTOUIuP8" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

In this catch from [*viaggi di nozze*](https://it.wikipedia.org/wiki/Viaggi_di_nozze) the Italian iconic  actors [*Carlo Verdone*](https://it.wikipedia.org/wiki/Carlo_Verdone) and [*Claudia Gerini*](https://it.wikipedia.org/wiki/Claudia_Gerini) listening to some good *heavy metal* remember the most special situation where they have some good sex.
Like them, or better *i wish it'll be*, our two friends **OpenBSD** and **GnuPG** continue them relation.  [*Last time*](https://steemit.com/openbsd/@npna/openbsd-is-back-now-with-gnupg) we've seen how to install, generate master key and subkeys (*sign, encryption and authentication*) and backup all of them in an encrypted usb stick under **OpenBSD**.  The result of those operation can be resumed and visualized with this command:
```sh
$ gpg2 --list-keys
/home/taglio/.gnupg/pubring.kbx
-------------------------------
pub   rsa4096/0xAD8E487FF2F05FDE 2018-02-23 [SC]
      Key fingerprint = 6ACF BE8E 6C24 EA90 3F5B  9F49 AD8E 487F F2F0 5FDE
uid                   [ultimate] No Place No Address (No Place No Address) <npna@protonmail.ch>
sub   rsa4096/0x3C423C42DE438790 2018-02-23 [E]
sub   rsa4096/0x5DC3DAEF3359F361 2018-02-23 [S]
sub   rsa4096/0x52E923E51A16A5E9 2018-02-23 [A]
$
```
## Export the public key to the Internet
![_Hierarchical trust](http://users.ece.cmu.edu/~adrian/630-f04/PGP-intro_files/fig1-12.gif)
To publish our public key to the Internet allowing others **OpenPGP** users establish a secure channel of communication with us (*i know it's not perfect but combined with other technologies it's the state of the art*) we've to export our **public key**   0xAD8E487FF2F05FDE to one the keyserver of the *pgp network* . Let's start with our *shell sex*:
```sh
$ wget -P ~/.gnupg https://sks-keyservers.net/sks-keyservers.netCA.pem{,.asc}
$ chmod -R 700 .gnupg 
$ openssl verify -trusted sks-keyservers.netCA.pem -check_ss_sig sks-keyservers.netCA.pem
sks-keyservers.netCA.pem: OK
$ openssl x509 -in sks-keyservers.netCA.pem -noout -text | grep "X509v3 Subject Key Identifier" -A1 | tail -n1                E4:C3:2A:09:14:67:D8:4D:52:12:4E:93:3C:13:E8:A0:8D:DA:B6:F3
$
```
We download the `CA`certificate of the keyservers pool `sks-keyservers.net`
with its **PGP signature** (*.asc file*).  We verify it with the [**openssl suite** ](https://www.openssl.org/) and then we extract the *"X509v3 Subject Key Identifier"* and check if it is the same with the one that we've [here](https://sks-keyservers.net/verify_tls.php).
```sh
$ host -t a hkps.pool.sks-keyservers.net
hkps.pool.sks-keyservers.net has address 193.164.133.100
$ cat << EOF >> ~/.gnupg/gpg.conf          
keyserver hkps://193.164.133.100
EOF
$ cat << EOF > ~/.gnupg/dirmngr.conf
keyserver hkps://193.164.133.100
hkp-cacert ~/.gnupg/sks-keyservers.netCA.pem
EOF
```
Like you can see we've just added one new directive in our `gpg.conf` file after resolving the pool of keyservers `sks` in a single `ipv4` address.  Next we've created a new configure file for `dirmngr` that is the daemon resposible of the access to the **OpenPGP** keyservers
```sh
$ gpg2 --send-key 0xAD8E487FF2F05FDE
gpg: sending key 0xAD8E487FF2F05FDE to hkp://193.164.133.100
$ gpg2 --fingerprint 0xAD8E487FF2F05FDE | grep finger | head -n 1
      Key fingerprint = 6ACF BE8E 6C24 EA90 3F5B  9F49 AD8E 487F F2F0 5FDE
```
this is the normal command to transfer our `pub `key who owner is ` [ultimate] No Place No Address (No Place No Address) <npna@protonmail.ch>`; after some minutes we can search our `public` key in the **OpenPGP** network [here](http://hkps.pool.sks-keyservers.net/).

## Encryption using public key is AWESOME

We want to establish a secure communications channel between `npna@protonmail.ch` and `r.giuntoli@protonmail`, without public key encription and signing will give us a lot of headchache. Using it produce this esquema:

![bob and alice](https://www.usna.edu/Users/cs/wcbrown/courses/si110AY13S/lec/l26/asymmetricencryption.png)

In practice never Alice or Bob have to send in an insecure channel, like *Internet*, their private keys. In my example alice got a **Gentoo** workstation, and Bob an **OpenBSD** one. The two folks have uploaded their `public` keys using the **OpenPGP** suite. The two `trust` each one and can verify the fingerprint of the other utilizing a *secure* channel, like they are in the same room creating this example.

```sh
alice$ gpg --search-keys npna
gpg: data source: http://37.191.226.104:11371
(1)	No Place No Address (No Place No Address) <npna@protonmail.ch>
	  4096 bit RSA key 0xAD8E487FF2F05FDE, created: 2018-02-23
Keys 1-1 of 1 for "npna".  Enter number(s), N)ext, or Q)uit > 1
gpg: key 0xAD8E487FF2F05FDE: public key "No Place No Address (No Place No Address) <npna@protonmail.ch>" imported
gpg: Total number processed: 1
gpg:               imported: 1
alice $ gpg --edit-key 0xAD8E487FF2F05FDE
gpg (GnuPG) 2.2.5; Copyright (C) 2018 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

pub  rsa4096/0xAD8E487FF2F05FDE
     created: 2018-02-23  expires: never       usage: SC  
     trust: unknown       validity: unknown
sub  rsa4096/0x3C423C42DE438790
     created: 2018-02-23  expires: never       usage: E   
sub  rsa4096/0x5DC3DAEF3359F361
     created: 2018-02-23  expires: never       usage: S   
sub  rsa4096/0x52E923E51A16A5E9
     created: 2018-02-23  expires: never       usage: A   
[ unknown] (1). No Place No Address (No Place No Address) <npna@protonmail.ch>

gpg> fpr
pub   rsa4096/0xAD8E487FF2F05FDE 2018-02-23 No Place No Address (No Place No Address) <npna@protonmail.ch>
 Primary key fingerprint: 6ACF BE8E 6C24 EA90 3F5B  9F49 AD8E 487F F2F0 5FDE

gpg> sign

pub  rsa4096/0xAD8E487FF2F05FDE
     created: 2018-02-23  expires: never       usage: SC  
     trust: unknown       validity: unknown
 Primary key fingerprint: 6ACF BE8E 6C24 EA90 3F5B  9F49 AD8E 487F F2F0 5FDE

     No Place No Address (No Place No Address) <npna@protonmail.ch>

Are you sure that you want to sign this key with your
key "Riccardo Giuntoli <r.giuntoli@protonmail.ch>" (0xA51D8EF938AF47D0)

Really sign? (y/N) y

gpg> save
alice$
```

So we search the `public` key of Alice in the `keyserver` , in this case we've got only a possible key, we select pressing `1` at the **OpenPGP** shell.  Next we enter in `edid` mode and always at the **OpenPGP** shell and we see that also Bob have created three *subkeys* one for **encryption**, another for **signing** and the last for **authentication**. We print the fingerprint to validate it with Bob to locally sign it with the option `sign` . We save and the returl to the `bash` shell.

We do the same process in the Bob **OpenBSD** machine:

```sh
bob$ gpg2 --search-key r.giuntoli
gpg: data source: http://18.9.60.141:11371
(1)     Riccardo Giuntoli <r.giuntoli@protonmail.ch>
          4096 bit RSA key 0x6DAE5C27DFAF0D6F, created: 2018-02-20
(2)     ANTRAX <r.giuntoli@protonmail.ch>
          3072 bit RSA key 0xAAE9F49A70ED3F09, created: 2017-09-04
(3)     Riccardo Giuntoli (MESWIFI, S.L.) <r.giuntoli@meswifi.com>
          2048 bit RSA key 0x83D1285CD6F38DFA, created: 2014-01-26
Keys 1-3 of 3 for "r.giuntoli".  Enter number(s), N)ext, or Q)uit > 1
gpg: key 0x6DAE5C27DFAF0D6F: public key "Riccardo Giuntoli <r.giuntoli@protonmail.ch>" imported
gpg: Total number processed: 1
gpg:               imported: 1
bob$ gpg2 --edit-key r.giuntoli
gpg (GnuPG) 2.1.23; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.


pub  rsa4096/0x6DAE5C27DFAF0D6F
     created: 2018-02-20  expires: never       usage: SC  
     trust: unknown       validity: full
sub  rsa2048/0xA51D8EF938AF47D0
     created: 2018-02-20  expires: never       usage: S   
sub  rsa2048/0x32772E38B5C56D73
     created: 2018-02-20  expires: never       usage: E   
sub  rsa2048/0xE3E741619E88263B
     created: 2018-02-20  expires: never       usage: A   
[  full  ] (1). Riccardo Giuntoli <r.giuntoli@protonmail.ch>

gpg> fpr
pub   rsa4096/0x6DAE5C27DFAF0D6F 2018-02-20 Riccardo Giuntoli <r.giuntoli@protonmail.ch>
 Primary key fingerprint: 90DC 1D49 FC85 DD2E 38AC  5301 6DAE 5C27 DFAF 0D6F

gpg> sign

pub  rsa4096/0x6DAE5C27DFAF0D6F
     created: 2018-02-20  expires: never       usage: SC  
     trust: unknown       validity: full
 Primary key fingerprint: 90DC 1D49 FC85 DD2E 38AC  5301 6DAE 5C27 DFAF 0D6F

     Riccardo Giuntoli <r.giuntoli@protonmail.ch>

Are you sure that you want to sign this key with your
key "No Place No Address (No Place No Address) <npna@protonmail.ch>" (0xAD8E487FF2F05FDE)

Really sign? (y/N) y

gpg>save
bob$ 
```

In this case the reply at our search are three diferents keys, we install the first. 

```sh
alice$ echo "Я хочу встретиться с тобой" | gpg --armor --sign --encrypt -r npna - 
gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
gpg: 0x3C423C42DE438790: There is no assurance this key belongs to the named user
sub  rsa4096/0x3C423C42DE438790 2018-02-23 No Place No Address (No Place No Address) <npna@protonmail.ch>
 Primary key fingerprint: 6ACF BE8E 6C24 EA90 3F5B  9F49 AD8E 487F F2F0 5FDE
      Subkey fingerprint: 2754 CF0A D8FA 32DD 1123  EF25 3C42 3C42 DE43 8790

It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.

Use this key anyway? (y/N) y
-----BEGIN PGP MESSAGE-----

hQIMAzxCPELeQ4eQAQ/8CzGw9ms0lyHcinfJptV3ICjUIrDxSHEDlj7L5oME1tc0
7BEuURJcqP6ftuU/OK8oQipKw+TBbioDIjNNnWVlpPGhSyHZl8tRnUyOKhQDcGEU
8KWk9wZfwElBVBAbqhfsih5dBIGgmZwvA9iVJrdvXv8uA7OD2qNsiD7SLG1XaemN
un8FnlRptYvwnR4/FEESf4FNeYUki0SwE3PDQb1c1uxTBILSavontFmUwxEhVyKB
2gV8SB2XtiN+WvxJDNlnuEI9NUM46XjNZVew9Iam78PAro/dcXxkD+PkY4Z8aXNB
ugIrym5//4vE7uTskT8qh0m4dlfDgnhuojFWjwSMdedntU09tNy5uMQMbqJkKmSV
8J4qfCVTtp9UtKgA/Ylqmw4TyAsZ+r8TZYgMtbTF3VTOGiXk5fSd5cjNUsAhDPWI
PR0+IrdumfTDoUdXX9SXZrJ8ftlFuVkj8II5Zm7wXbbXO3wTrzAZZTmZL0NoHkca
BKGp11rWk4MgpxwYQKsCI1LZc/q2Ve7PsviN57nmeiDGLGP1AgOqVVJP1LNEPLvs
7xlTDlviALdJXKR3rwww+iwFFLwdE7V7GO4pw5z/Wgxgsx2WlFbOPG6FnJmeW22D
xCQm4NDJvVLnmI6fa8Y2JPzZVFou7bJxEgnGWwoXGdT8b+S10DWxNlAxKKh+svLS
wPEBJYi7+I0iY1gucLhLnKDnTd1XV9POHt/kT2uSb7INZedd221lqLw327aqFpo+
anrVtodShpg3CQM9mC/VX5/tYh/4YqupR+TAmlXvscbjNEy19gjbTjhhO80c1dla
axeLsA6OuDh77arEyk1gzlEFHl2ebUSZjBVNG7nFAMur9hS22TBSOmGHdgEH4w4z
pPCk3dAnqOBdgXCOrt8a9MHDXmg6RqkpclXAK/X/EOtx0JouNQUSXjyakzrKffKF
KtkUGsTyH8wUAlJb2X/enOi89b2VBHgqqy4ss9B3Do1sYpgLo3Ho9oGqpxt0oaiZ
pnIq2jZdns8kdtsEJsQTnN/ucu4L+/MzDGjUAdP7vC94in62gqypnaNhYzpyHrS+
H8p+ddtWoYEZHnp/TuqqdSw3psDzhA9UyWFa1BKTOczI3CBFhoio/NPdK4bvnNdw
Ef9gP4kVaCIDh6QOQV5xJtKgTU53alu76xlyZcyBqP4q6fk94klVxkb1G5lmP60I
HLXQvFuTaGF8BTZqNYsNZlOS0B5plXMYCw1s4hX6n2h+4HgbOg71cHaTTIw5IRls
qk2g
=i1ah
-----END PGP MESSAGE-----
```

Alice encrypt and sign the message *Я хочу встретиться с тобой* for Bob. The process is simple, she piped it to `gpg` that is launched with:

1.  `--armor` the result of the process have to be always text.
2.  `--sign` she sign it with her `subkey` dedicated to the sign operation.
3.  `--ecrypt` she encript it.
4.  `-r Bob` the message will only readable by Bob (*that seems to be russian*).
5.  `-` it indicate to `gpg` that the text in input was piped. 

Now she use a **free past service** to upload it to the *Internet*, the result is there:

[ghostbin free past service,  a message for you darling](https://ghostbin.com/paste/28fqm/raw)

So Bob is very exited and want to rapidly verify and decrypt his *love message*:

```sh
bob$ wget https://ghostbin.com/paste/28fqm/raw -o log -O testmsg
bob$ cat testmsg | gpg2 --decrypt
gpg: encrypted with 4096-bit RSA key, ID 0x3C423C42DE438790, created 2018-02-23
      "No Place No Address (No Place No Address) <npna@protonmail.ch>"
Я хочу встретиться с тобой
gpg: Signature made Tue Mar  6 18:49:14 2018 CET
gpg:                using RSA key E9664F440C5E14C8E661AC6BA51D8EF938AF47D0
gpg: Good signature from "Riccardo Giuntoli <r.giuntoli@protonmail.ch>" [full]
Primary key fingerprint: 90DC 1D49 FC85 DD2E 38AC  5301 6DAE 5C27 DFAF 0D6F
     Subkey fingerprint: E966 4F44 0C5E 14C8 E661  AC6B A51D 8EF9 38AF 47D0
bob$
```

Silently download the message and save in local `testmsg` text file and use `gpg --decrypt` to read his message.

## OpenPGP others applications and a little bit of magic 

![gandalf](/home/taglio/Pictures/gandalf.png)

With the **OpenPGP** suite we can also simply encrypt a file or folder. 

But *Bob* want to be more [**31337** ](https://en.wikipedia.org/wiki/Leet) and want to hide a *secret message* in an image, embedding in it using a technology knowned as [**Steganography**](https://en.wikipedia.org/wiki/Steganography). And with **OpenBSD** is a nice and rapid task. 

```sh
bob$ doas pkg_add -U steghide
doas (taglio@Lutetia.unknown_domain) password: 
quirks-2.367 signed on 2017-10-03T11:21:28Z
steghide-0.5.1p3: ok
bob$ steghide
steghide version 0.5.1

the first argument must be one of the following:
 embed, --embed          embed data
 extract, --extract      extract data
 info, --info            display information about a cover- or stego-file
   info <filename>       display information about <filename>
 encinfo, --encinfo      display a list of supported encryption algorithms
 version, --version      display version information
 license, --license      display steghide's license
 help, --help            display this usage information

embedding options:
 -ef, --embedfile        select file to be embedded
   -ef <filename>        embed the file <filename>
 -cf, --coverfile        select cover-file
   -cf <filename>        embed into the file <filename>
 -p, --passphrase        specify passphrase
   -p <passphrase>       use <passphrase> to embed data
 -sf, --stegofile        select stego file
   -sf <filename>        write result to <filename> instead of cover-file
 -e, --encryption        select encryption parameters
   -e <a>[<m>]|<m>[<a>]  specify an encryption algorithm and/or mode
   -e none               do not encrypt data before embedding
 -z, --compress          compress data before embedding (default)
   -z <l>                 using level <l> (1 best speed...9 best compression)
 -Z, --dontcompress      do not compress data before embedding
 -K, --nochecksum        do not embed crc32 checksum of embedded data
 -N, --dontembedname     do not embed the name of the original file
 -f, --force             overwrite existing files
 -q, --quiet             suppress information messages
 -v, --verbose           display detailed information

extracting options:
 -sf, --stegofile        select stego file
   -sf <filename>        extract data from <filename>
 -p, --passphrase        specify passphrase
   -p <passphrase>       use <passphrase> to extract data
 -xf, --extractfile      select file name for extracted data
   -xf <filename>        write the extracted data to <filename>
 -f, --force             overwrite existing files
 -q, --quiet             suppress information messages
 -v, --verbose           display detailed information

options for the info command:
 -p, --passphrase        specify passphrase
   -p <passphrase>       use <passphrase> to get info about embedded data

To embed emb.txt in cvr.jpg: steghide embed -cf cvr.jpg -ef emb.txt
To extract embedded data from stg.jpg: steghide extract -sf stg.jpg
bob$ wget https://pbs.twimg.com/profile_images/966976157592838145/bzhg-p3s_400x400.jpg -o log 
bob$ cat << EOF > msg
trip lover trust
EOF
bob$ steghide embed -cf bzhg-p3s_400x400.jpg -ef msg
Enter passphrase: 
Re-Enter passphrase: 
embedding "testmsg" in "bzhg-p3s_400x400.jpg"... done
bob$ 
```

To obtain this little *magic*, Bob install **steghide** from the official ports of **OpenBSD**, next he write a text message and embedded protected with a password in the `jpeg` image. 

Now he saves the `jpg` image in his harddisk with `gpg` and `symmetric` encryptions. When he will send to *Alice* he will decrypt it and reuse the `public` method. 

```sh
bob$ gpg2 --symmetric bzhg-p3s_400x400.jpg
```

Nice, i'm in love with this *Bob and Alice* story, i'm in love with my work that is and will be write about security stuff, protecting privacy for who want it.

*love is back, RG*





