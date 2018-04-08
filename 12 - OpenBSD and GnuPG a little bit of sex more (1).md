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
We download the `CA`certificate of the keyservers pool `sks`
with its **PGP signature** (*.asc file*).  We verify it with the [**openssl suite** ](https://www.openssl.org/)  and then we extract the *"X509v3 Subject Key Identifier"* and check if it is the same with the one that we've [here](https://sks-keyservers.net/verify_tls.php).
```sh
$ host -t a hkps.pool.sks-keyservers.net
hkps.pool.sks-keyservers.net has address 193.164.133.100
$
cat << EOF >> gpg.conf          
keykeyserver hkps://193.164.133.100
hkp-cacert ~/.gnupg/sks-keyservers.netCA.pem
EOF
``` 
Like you can see we've just added two new directive in our `gpg.conf` file after resolving the pool of keyservers `sks` in a single `ipv4`address. 
```sh
$ gpg2 --send-key 0xAD8E487FF2F05FDE
gpg: sending key 0xAD8E487FF2F05FDE to hkp://193.164.133.100
$
```
this is the normal command to transfer our `pub`key who owner is ` [ unknown] No Place No Address (No Place No Address) <npna@protonmail.ch>`; after some minutes we can very that we've send to `public`key in the **OpenPGP** network [here](http://hkps.pool.sks-keyservers.net/).
Like an example now we try to import public key of another **OpenPGP** user in our database and declare it like `trust`.  We utilize [this POST on reddit.com](https://www.reddit.com/r/GPGpractice/comments/7z0qmd/my_new_openphpgpg_4096_public_key/dukn9kb/). We can appreciate that reddit [u/chriscrutch](https://www.reddit.com/user/chriscrutch) give us the `fingerprint` `48CF AAEE 7E80 0E1A A9D0 2C5B 5DBA 09ED 73AB 99E8`: 
```sh
$ gpg2 -- search-key 48CF AAEE 7E80 0E1A A9D0 2C5B 5DBA 09ED 73AB 99E8
$ (1)     Chris Brown <chriscrutch@gmail.com>
          4096 bit RSA key 0x5DBA09ED73AB99E8, created: 2016-06-07, expires: 2018-06-07
Keys 1-1 of 1 for "48CF AAEE 7E80 0E1A A9D0 2C5B 5DBA 09ED 73AB 99E8".  Enter number(s), N)ext, or Q)uit > 1
key 0x5DBA09ED73AB99E8:
2 signatures not checked due to missing keys
gpg: key 0x5DBA09ED73AB99E8: "Chris Brown <chriscrutch@gmail.com>" not changed
gpg: Total number processed: 1
gpg:              unchanged: 1
``` 
Ok so we search by `fingerprint` add to the system and find that in reality these are two keys. 
```sh
gpg2 --edit-key 5DBA09ED73AB99E8
gpg (GnuPG) 2.1.23; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
pub  rsa4096/0x5DBA09ED73AB99E8
     created: 2016-06-07  expires: 2018-06-07  usage: SC  
     trust: full          validity: full
sub  rsa4096/0x6154AC198B6517C6
     created: 2016-06-07  expires: 2018-06-03  usage: E   
[  full  ] (1). Chris Brown <chriscrutch@gmail.com>
gpg> sign
"Chris Brown <chriscrutch@gmail.com>" was already signed by key 0xAD8E487FF2F05FDE
Nothing to sign with key 0xAD8E487FF2F05FDE
gpg> 
gpg2 --edit-key 5DBA09ED73AB99E8
gpg (GnuPG) 2.1.23; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
pub  rsa4096/0x5DBA09ED73AB99E8
     created: 2016-06-07  expires: 2018-06-07  usage: SC  
     trust: full          validity: full
sub  rsa4096/0x6154AC198B6517C6
     created: 2016-06-07  expires: 2018-06-03  usage: E   
[  full  ] (1). Chris Brown <chriscrutch@gmail.com>
gpg> sign
"Chris Brown <chriscrutch@gmail.com>" was already signed by key 0xAD8E487FF2F05FDE
Nothing to sign with key 0xAD8E487FF2F05FDE
gpg> trust
pub  rsa4096/0x5DBA09ED73AB99E8
     created: 2016-06-07  expires: 2018-06-07  usage: SC  
     trust: full          validity: full
sub  rsa4096/0x6154AC198B6517C6
     created: 2016-06-07  expires: 2018-06-03  usage: E   
[  full  ] (1). Chris Brown <chriscrutch@gmail.com>
Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)
  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu
Your decision? 4
pub  rsa4096/0x5DBA09ED73AB99E8
     created: 2016-06-07  expires: 2018-06-07  usage: SC  
     trust: full          validity: full
sub  rsa4096/0x6154AC198B6517C6
     created: 2016-06-07  expires: 2018-06-03  usage: E   
[  full  ] (1). Chris Brown <chriscrutch@gmail.com>
gpg> 
```	
So we start editing the new keys,  we sign them and at last we declare them like `trust`. To be sure:
```sh
$ echo "export GPG_TTY=$(tty)" >> ~/.zshrc
$ source ~/.zshrc
$ echo RELOADAGENT | gpg-connect-agent
$ gpg2 --edit-key 0x5DBA09ED73AB99E8
gpg (GnuPG) 2.1.23; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
pub  rsa4096/0x5DBA09ED73AB99E8
     created: 2016-06-07  expires: 2018-06-07  usage: SC  
     trust: full          validity: full
sub  rsa4096/0x6154AC198B6517C6
     created: 2016-06-07  expires: 2018-06-03  usage: E   
[  full  ] (1). Chris Brown <chriscrutch@gmail.com>
gpg> trust
pub  rsa4096/0x5DBA09ED73AB99E8
     created: 2016-06-07  expires: 2018-06-07  usage: SC  
     trust: full          validity: full
sub  rsa4096/0x6154AC198B6517C6
     created: 2016-06-07  expires: 2018-06-03  usage: E   
[  full  ] (1). Chris Brown <chriscrutch@gmail.com>
Please decide how far you trust this user to correctly verify other users' keys
(by looking at passports, checking fingerprints from different sources, etc.)
  1 = I don't know or won't say
  2 = I do NOT trust
  3 = I trust marginally
  4 = I trust fully
  5 = I trust ultimately
  m = back to the main menu
Your decision? 4
pub  rsa4096/0x5DBA09ED73AB99E8
     created: 2016-06-07  expires: 2018-06-07  usage: SC  
     trust: full          validity: full
sub  rsa4096/0x6154AC198B6517C6
     created: 2016-06-07  expires: 2018-06-03  usage: E   
[  full  ] (1). Chris Brown <chriscrutch@gmail.com>
gpg> save
$
```
we source the `zsh` home config file because we've added one variable, then we restart the `gpg-agent`and another time we edit the same key and recheck it.

