## Another friend, another animal, a gnu.

![music gnu](https://www.gnu.org/graphics/listen-eighth.jpg)
And yes, this gnu is *special*. First of all is a capital letters **GNU**, he likes [free software](https://www.gnu.org/), and our special one also like **electronic music**. But not only *free software*, he is specialized also in privacy, he can be the **guardian** of our privacy! Because one of the hottest applications in this wonderful *GNU world* is [**GnuPG**](https://www.gnupg.org/) a mature piece of *open source* software:

> GnuPG is a complete and free implementation of the OpenPGP standard as defined by RFC4880 (also known as PGP). GnuPG allows you to encrypt and sign your data and communications; it features a versatile key management system, along with access modules for all kinds of public key directories. GnuPG, also known as GPG, is a command line tool with features for easy integration with other applications. A wealth of frontend applications and libraries are available. GnuPG also provides support for S/MIME and Secure Shell (ssh).
Since its introduction in 1997, GnuPG is Free Software (meaning that it respects your freedom). It can be freely used, modified and distributed under the terms of the GNU General Public License .

## OpenBSD want your pretty good privacy

![privacy is like a pretty woman](https://media.vanityfair.com/photos/59c9789f4f79bc3b7908b511/master/w_960,c_limit/pretty-woman-musical%25202.jpg)
Speaking about *pretty* obviously my first tough is a *wonderful woman*. And doing a simple association my *connected* brain has elaborated this image, a wonderful Julia Roberts singing a song in the set's bathroom of [*pretty woman*](http://www.imdb.com/title/tt0100405/), do you remember it?
But you have to understand that *pretty* have to be also our **privacy** in a **interconnected world**, *the Internet*. Every single man and woman (child i don't think that have to use it) in the world have access to the Internet, but only a little part know how to protect him/her **privacy** from what we've previously named *the privacy cannibals*.  
Today we add another friend to our great guarantees of privacy, a cool dude that follow the **OpenPGP** standard that you can freely study reading the [**RFC 4880**](https://tools.ietf.org/html/rfc4880).
## Create the perfect pgp keypair with OpenBSD
![perfect pgp keypair](https://static.fsf.org/nosvn/enc-dev0/img/en/screenshots/step2a-01-make-keypair.png)
 Ok let's go deep and start the configuration of our *GNU PGP*.
 Install the software from ports repository:
 ```sh
 $ rm -rf .gnupg
 $ mkdir .gnupg
 $ chmod -R go-rwx .gnupg
 $ doas pkg_add -U gnupg-2.1.23
 ```
We've deleted possible previous `gnupg` configuration, recreate the user directory of the program, assign to it the correct permissions and install the version 2 of the `gnupg` collection.
```sh
$ cat << EOF > ~/.gnupg/gpg.conf
use-agent
personal-cipher-preferences AES256 AES192 AES CAST5
personal-digest-preferences SHA512 SHA384 SHA256 SHA224
default-preference-list SHA512 SHA384 SHA256 SHA224 AES256 AES192 AES CAST5 ZLIB BZIP2 ZIP Uncompressed
cert-digest-algo SHA512
s2k-digest-algo SHA512
s2k-cipher-algo AES256
charset utf-8
fixed-list-mode
no-comments
no-emit-version
keyid-format 0xlong
list-options show-uid-validity
verify-options show-uid-validity
with-fingerprint
EOF
```
Create the *bootstrap*  `gpg` configuration file in our previously created directory. 
```sh
$ gpg2 --full-generate-key
gpg (GnuPG) 2.1.23; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 4
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0
Key does not expire at all
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: No Place No Address
Email address: npna@protonmail.ch
Comment: No Place No Address
You selected this USER-ID:
    "No Place No Address (No Place No Address) <npna@protonmail.ch>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: /home/taglio/.gnupg/trustdb.gpg: trustdb created
gpg: key 0xAD8E487FF2F05FDE marked as ultimately trusted
gpg: directory '/home/taglio/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/taglio/.gnupg/openpgp-revocs.d/6ACFBE8E6C24EA903F5B9F49AD8E487FF2F05FDE.rev'
public and secret key created and signed.

Note that this key cannot be used for encryption.  You may want to use
the command "--edit-key" to generate a subkey for this purpose.
pub   rsa4096/0xAD8E487FF2F05FDE 2018-02-23 [SC]
      Key fingerprint = 6ACF BE8E 6C24 EA90 3F5B  9F49 AD8E 487F F2F0 5FDE
uid                              No Place No Address (No Place No Address) <npna@protonmail.ch>
```
We're creating a **no expiring** master key in our home directory, using `RSA` sign only option with the maximum key size that is `4096 bits` . In `gpg` every key is associated with a user ID, in our case is `npna@protonmail.ch`. Remember to use a strong unpredictable password and to anote and store it in a secure place (*¿do you remember what is a pen and what is a paper?*). It's important for the rest of the configuration know the result `keyid`, for our case is `0xAD8E487FF2F05FDE`. Go ahead with the deep configuration:
```
$ gpg2 --expert --edit-key 0xAD8E487FF2F05FDE
gpg (GnuPG) 2.1.23; Copyright (C) 2017 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   1  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 1u
sec  rsa4096/0xAD8E487FF2F05FDE
     created: 2018-02-23  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
[ultimate] (1). No Place No Address (No Place No Address) <npna@protonmail.ch>

gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 6
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  rsa4096/0xAD8E487FF2F05FDE
     created: 2018-02-23  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/0x3C423C42DE438790
     created: 2018-02-23  expires: never       usage: E   
[ultimate] (1). No Place No Address (No Place No Address) <npna@protonmail.ch>

```
Now we've entered in the `gpg` shell! Look at the new prompt `gpg>`. From there we can access to all the possible commands of our **GNU** privacy suite.
```sh
gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
Your selection? 6
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  rsa4096/0x17B9BD907F897DD1
     created: 2018-02-23  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/0xEF0998EA8BB3F32B
     created: 2018-02-23  expires: never       usage: E   
[ultimate] (1). No Place No Address (No Place No Address) <npna@protonmail.ch>
```
We've created a key that depend from our master key, selecting `RSA` for encrypt only. This is our encryption key.
```sh
gpg>addkey
addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 4
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  rsa4096/0xAD8E487FF2F05FDE
     created: 2018-02-23  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/0x3C423C42DE438790
     created: 2018-02-23  expires: never       usage: E   
ssb  rsa4096/0x5DC3DAEF3359F361
     created: 2018-02-23  expires: never       usage: S   
[ultimate] (1). No Place No Address (No Place No Address) <npna@protonmail.ch>
```
Now we've created another sub key. This time to sign only, always with the `RSA` protocol. His size is as usual `4096 bits`.
```sh
addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
   (7) DSA (set your own capabilities)
   (8) RSA (set your own capabilities)
  (10) ECC (sign only)
  (11) ECC (set your own capabilities)
  (12) ECC (encrypt only)
  (13) Existing key
Your selection? 8

Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Sign Encrypt 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? S

Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Encrypt 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? E

Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? A

Possible actions for a RSA key: Sign Encrypt Authenticate 
Current allowed actions: Authenticate 

   (S) Toggle the sign capability
   (E) Toggle the encrypt capability
   (A) Toggle the authenticate capability
   (Q) Finished

Your selection? Q
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 
Key does not expire at all
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  rsa4096/0xAD8E487FF2F05FDE
     created: 2018-02-23  expires: never       usage: SC  
     trust: ultimate      validity: ultimate
ssb  rsa4096/0x3C423C42DE438790
     created: 2018-02-23  expires: never       usage: E   
ssb  rsa4096/0x5DC3DAEF3359F361
     created: 2018-02-23  expires: never       usage: S   
ssb  rsa4096/0x52E923E51A16A5E9
     created: 2018-02-23  expires: never       usage: A   
[ultimate] (1). No Place No Address (No Place No Address) <npna@protonmail.ch>
gpg> save
```
At last we've created a sub key for authentication purpose. Thanks to the `--expert` option that we've used we can edit **like a pro** our last `RSA` key using the `(8)` option in our `gpg` shell. Follow the commands and we've finished! Don't leave the last `save` if you don't launch it you will loose all! Review the created keys with:
```sh
$ gpg2 --list-secret-keys
/home/taglio/.gnupg/pubring.kbx
-------------------------------
sec   rsa4096/0xAD8E487FF2F05FDE 2018-02-23 [SC]
      Key fingerprint = 6ACF BE8E 6C24 EA90 3F5B  9F49 AD8E 487F F2F0 5FDE
uid                   [ultimate] No Place No Address (No Place No Address) <npna@protonmail.ch>
ssb   rsa4096/0x3C423C42DE438790 2018-02-23 [E]
ssb   rsa4096/0x5DC3DAEF3359F361 2018-02-23 [S]
ssb   rsa4096/0x52E923E51A16A5E9 2018-02-23 [A]
```
## Export to an encrypted USB stick

Ok, we've have created our **4**  `gpg2` keys, one master with 3 slaves, *encryption, sign and authentication* .
Let's prepare an encrypted fresh USB stick to backup them!
```sh
$ doas su
#fdisk -yig sd2 
Writing MBR at offset 0.
Writing GPT.
# disklabel -E sd2
Label editor (enter '?' for help at any prompt)
> ?
Available commands:
 ? | h    - show help                 n [part] - set mount point
 A        - auto partition all space  p [unit] - print partitions
 a [part] - add partition             q        - quit & save changes
 b        - set OpenBSD boundaries    R [part] - resize auto allocated partition
 c [part] - change partition size     r        - display free space
 D        - reset label to default    s [path] - save label to file
 d [part] - delete partition          U        - undo all changes
 e        - edit drive parameters     u        - undo last change
 g [d|u]  - [d]isk or [u]ser geometry w        - write label to disk
 i        - modify disklabel UID      X        - toggle expert mode
 l [unit] - print disk label header   x        - exit & lose changes
 M        - disklabel(8) man page     z        - delete all partitions
 m [part] - modify partition

Suffixes can be used to indicate units other than sectors:
 'b' (bytes), 'k' (kilobytes), 'm' (megabytes), 'g' (gigabytes) 't' (terabytes)
 'c' (cylinders), '%' (% of total disk), '&' (% of free space).
Values in non-sector units are truncated to the nearest cylinder boundary.
> l
# /dev/rsd2c:
type: SCSI
disk: SCSI disk
label: Flash Disk      
duid: 0000000000000000
flags:
bytes/sector: 512
sectors/track: 63
tracks/cylinder: 255
sectors/cylinder: 16065
cylinders: 16
total sectors: 257536
boundstart: 64
boundend: 257473
drivedata: 0 
> > a a
offset: [64] 
size: [257409] 
FS type: [4.2BSD] RAID
> w
> q
No label changes.
# 
``` 
We've initialized the USB stick with the **OpenBSD** `fdisk` with a `GPT` partition table and the we add a single `RAID` partition to it with `disklabel`. We use `RAID` because this is the manner that use **OpenBSD** to do it.
```sh
# bioctl -c C -l /dev/sd2a softraid0  
New passphrase: 
Re-type passphrase: 
softraid0: CRYPTO volume attached as sd3
# 
```
We've encrypted the first partition label of our USB stick with the command `bioctl` that result in the creation of the pseudo device `sd3`.
```sh
# newfs sd3c         
newfs: reduced number of fragments per cylinder group from 16048 to 15984 to enlarge last cylinder group
/dev/rsd3c: 125.4MB in 256880 sectors of 512 bytes
5 cylinder groups of 31.22MB, 1998 blocks, 4096 inodes each
super-block backups (for fsck -b #) at:
 32, 63968, 127904, 191840, 255776,
# mkdir /mnt/encrypted_usb && mount dev/sd3c /mnt/encrypted_usb
```
Create an   `ufs2` filesystem in the pseudo device disk label `c`, that is the one that indicate all the disk. Create a partition where to mount it and mount it!
```sh
# gpg2 --homedir /home/taglio/.gnupg --armor --export-secret-keys 0xAD8E487FF2F05FDE > /mnt/encrypted_usb/mastersub.key
# gpg2 --homedir /home/taglio/.gnupg --armor --export-secret-subkeys 0xAD8E487FF2F05FDE > /mnt/encrypted_usb/sub.key
# umount /mnt/encrypted_usb
# 
```
## Greetings

Bye Bye little loves, *i love you*.



