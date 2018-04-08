## In the last chapter...
![the terminator](https://images.duckduckgo.com/iu/?u=http%3A%2F%2F2.bp.blogspot.com%2F-ezjdI7-BZFc%2FTc6Ooc4pGVI%2FAAAAAAAAMlQ%2FAZ54weX0YhA%2Fs1600%2FTERMINATOR5.jpg&f=1)
Hello there, nice people. Machines and technology, take care i'm in love with the twos but i command them they cannot own me, today are so so dangerous. Obviously we're not in the post apocalyptic scene *predicted* by **James Cameron** long time ago, in the **1984** with the futuristic (*just now*) film [**The Terminator**](http://www.imdb.com/title/tt0088247/). But i don't think that *we are not so distance from it*. *But this is off-topic and i don't have sufficient information to argue it*. Simply take care. **Take child off technological dispositives**.
So in our [last article](https://steemit.com/openbsd/@npna/openbsd-and-freedos-vs-the-hell-in-earth) we build with **OpenBSD** a **FreeDOS** *raw* image and now we want to write directly to a **usb stick**. I've presented to you OpenBSD like a very *clean and clear* system; and it's absolutely true. But if you came here with a Linux background it could not appear like this to you, and yes...you have got reason. 
One of the biggest wall you have to pass to enter in the *OpenBSD world* is understand how he handle **mass storage devices**, or *disks*. 
**OpenBSD** for x86 and amd64 handle storage with two drivers wich manual pages you could find at:
1. `man 4 wd`, driver compatible with standards `MFM`, `RLL`, `ESDI`, `IDE`, and `EIDE` drives, as well as `Serial ATA` drives, and `PCMCIA/CF` storage media.
2. `man 4 sd`, driver compatible with standards `SCSI` that includes USB disks, SATA disks attached to an ahci(4) interface, and disk arrays attached to a RAID controller.
Devices are number on boot stage in the sequence that they are found, from `0`to `x`. 

Partitions means two different thinks in **OpenBSD**:

1. filesystem partitions created and managed by `disklabel`. We can find more information about it at `man 8 disklabel`.
2. `MBR`, `GPT`, that can be named also like *BIOS partitions*, because they are created using the `BIOS controller`, that are managed by `fdisk`. We can find more information about at `man 8 fdisk`.

## Write an usb stick with OpenBSD
![OpenBSD](https://www.openbsd.org/images/tshirt-2.jpg)
Let's see what happened when we plug an usb stick:
```sh
$ dmesg | tail -n 5
umass0 at uhub0 port 1 configuration 1 interface 0 "SanDisk Cruzer Blade" rev 2.00/2.01 addr 5
umass0: using SCSI over Bulk-Only
scsibus4 at umass0: 2 targets, initiator 0
sd2 at scsibus4 targ 1 lun 0: <SanDisk, Cruzer Blade, 2.01> SCSI4 0/direct fixed serial.888888888888888888
sd2: 3819MB, 512 bytes/sector, 7821312 sectors
``` 
We can see that the device `sd2` is initialized and it is a 4GB USB stick. Do directly write our boot `FAT-16`disk image that we previously created we don't have to take care about any partition of any kind, the virtual disk image have his personal `MBR`and filesystem. So we directly write to the disk with the `dd`utility, we can find more information about it at `man 1 dd`.
```sh
$ doas disklabel sd2
# /dev/rsd2c:
type: SCSI
disk: SCSI disk
label: Cruzer Blade    
duid: 0000000000000000
flags:
bytes/sector: 512
sectors/track: 63
tracks/cylinder: 255
sectors/cylinder: 16065
cylinders: 486
total sectors: 7821312
boundstart: 0
boundend: 7821312
drivedata: 0 

16 partitions:
#                size           offset  fstype [fsize bsize   cpg]
  c:          7821312                0  unused                    
  i:          7821312                0   MSDOS 
```
The `c` partition simply identify the entire disk, so:
```sh
$ doas dd if=~/freedos/freedos.img of=/dev/sd2c bs=4M
$ doas sync
$ doas sync
``` 
Now simply reboot the personal computer, select from `BIOS`the USB stick to boot and enjoy **FreeDOS**.
## Virtualize FreeDOS full with QEMU
![QEMU](https://images.duckduckgo.com/iu/?u=https%3A%2F%2Ftse4.mm.bing.net%2Fth%3Fid%3DOIP.fB91mTfs8H5RQTykFT-ytwHaEj%26pid%3D15.1&f=1)
Just in case you thinking that i'm alone, i've decided to add *another friend* in my personal *list of good guys*. It's another way to handle virtual machines under **OpenBSD**. It's a lot more sophisticated and older than *OpenBSD vmm* but in this case is not *kernel accelerated*.  We will dedicate to it many others POST but for now let's see how to install it under **OpenBSD** and next we will attach a video document about the process of instalation of the latest version of **FreeDOS**, the `1.2`. Simply add to the packages like this:
`$ doas pkg_add -U qemu`
And here you are the video: 
<iframe width="560" height="315" src="https://www.youtube.com/embed/sZlZgt6YSg4" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen></iframe>

## Greetings

As usual have a good night, thank you for spend you time reading me.
Nice regards,

*Riccardo Giuntoli*
