## The family is rapidly growing
![GENOVA antifascista](https://scontent-otp1-1.xx.fbcdn.net/v/t1.0-9/26992485_10215637286587786_5182180295980038263_n.jpg?oh=7fd752bfb4977878a635cda33d79d682&oe=5B150DBE)
Yes sir, yes. Our family, composed until now by [OpenBSD](https://www.openbsd.org/), [Alpine Linux](https://www.alpinelinux.org/) and [Docker](https://www.docker.com/) is rapidly growing. And yes, sir. Yes. All together we're fighting against your best friends, the infamous, the ugliest, the worst...the dudes called *the privacy cannibals*. Do you know what i mean, sure? 
We're working hard, no matter what time is it, no matter in what part in the world we are, no matter if we've **no money**. We perfectly know that you cannot do nothing against **the true**. And we're doing our best to expand our true, our doors are opened to all the *good guys*, there's a lot here but their brain was fucked by your shit tv, your fake news, your laws, etc etc etc. We're alive, we're **here to fight against you**.
Tonight, yes *it's a Friday night and we're working*, we're ready to welcome with open arms an *old guy*,  his experience will give us more power. Welcome to:

[**FreeDOS**](http://www.freedos.org/)

> FreeDOS is a complete, free, DOS-compatible operating system that you can use to play classic DOS games, run legacy business software, or develop embedded systems. Any program that works on MS-DOS should also run on FreeDOS.
It doesn’t cost anything to download and use FreeDOS. You can also share FreeDOS for others to enjoy! And you can view and edit our source code, because all FreeDOS programs are distributed under the GNU General Public License or a similar open source software license.
## UEFI
![The hell in earth](https://scontent-frx5-1.xx.fbcdn.net/v/t1.0-9/22886001_10214894127769280_7559827501261378744_n.jpg?oh=3f015f59bfb1b5b3d7fa53c0cfe848a4&oe=5B1A8A6A)

But why we want to build a bootable usb stick with **FreeDOS** under our strong **OpenBSD**? The answer is as usual to fight against the privacy cannibals! 
More than one decade ago the old [**BIOS**](https://en.wikipedia.org/wiki/BIOS) was silently replaced by the more capable and advanced [**UEFI**](http://www.uefi.org/), this is absolutely normal because of the pass of the years and exponencial grow of the power of our personal computers. [**UEFI**](https://en.wikipedia.org/wiki/Unified_Extensible_Firmware_Interface) is a complex system, it's like a standalone system operative with direct access to every component of our (*yes, it's our not your!*) machine. But...wait a moment...do you know how to use it? Do you ever know that it exist? And one more thing, *it's secure*? The answer to this question is totally insane, no, *it's not secure*. The idea is good, the company that started  in theory is one of the most important in IT, it's [**Intel**](https://www.intel.co.uk/content/www/uk/en/homepage.html?_ga=2.49642342.1336926968.1517604803-1923177664.1517604803).
The history is very large and obviously *we're going to go very deep in it*, but trust me **UEFI** and the various friend of him, like [**ME**](https://en.wikipedia.org/wiki/Intel_Management_Engine), [**TPM**](https://en.wikipedia.org/wiki/Trusted_Platform_Module) are insecure and closed source! Like **the hell in earth**.
## Lenovo UEFI BIOS without Windows?
![chaos](https://lh4.googleusercontent.com/ywq-n7n21H-S1x6vfI39fyRfgqJNtrzEd57IMNpMrofO6kr90wqUBDds7bpBvbmAsg3BC5ONqCV8nePRIj_o=w1366-h601)

There's no simple method to upgrade our personal computer if we've uninstall Windows...ou it's the first time that i write this name, shit! And it's basic for the users have one, because *UEFI* is living under the hood, in the first ring after the hardware. *It's the most important think to keep secure and clean!* There is no clean information, there is nothing, only there's [Χάος](https://en.wikipedia.org/wiki/Chaos_(mythology)).
Remember well people, *they love Χάος*.

## A FreeDOS bootable usb image under OpenBSD

But let's start preparing our **OpenBSD** to put order in this chaos:
```sh
$ mkdir -p freedos/stuff
$ cd freedos/stuff
$ wget https://www.ibiblio.org/pub/micro/pc-stuff/freedos/files/distributions/1.0/fdboot.img
$ wget https://www.ibiblio.org/pub/micro/pc-stuff/freedos/files/dos/sys/sys-freedos-linux/sys-freedos-linux.zip
$ wget https://download.lenovo.com/consumer/desktop/o35jy19usa_y900.exe
$ wget http://145.130.102.57/domoticx/software/amiflasher/AFUDOS%20Flasher%205.05.04.7z
```
Explanation in clear language as usual: create two directory, download the minimal boot disc image of **FreeDOS**, download [Syslinux](http://www.syslinux.org/wiki/index.php?title=The_Syslinux_Project) assembler [**MBR**](https://en.wikipedia.org/wiki/Master_boot_record)  bootloaders, download the last *Windows only* UEFI update from [Lenovo](https://pcsupport.lenovo.com) and download the relative *unknown* utility from [**AMI**](https://ami.com/en/) to flash our  motherboard UEFI chipset. Go ahead:
```sh
$ doas pkg_add -U nasm unzip dosfstools cabextract p7zip
```
1.	`nasm` the Netwide Assembler, a portable 80x86 assembler.
2.	`unzip` list, test and extract compressed files in a ZIP archive.
3.	`dosfstools`a collections of utilities to manipulate `MS-DOS`fs.
4.	`cabextract`  program to extract files from cabinet.
5.	`p7zip`collection of utilities to manipulate `7zip` archives.

```sh
$ mkdir sys-freedos-linux && cd sys-freedos-linux
$ unzip ../../sys-freedos-linux.zip
$ cd ~/freedos && mkdir old new
$ dd if=/dev/null of=freedos.img bs=1024 seek=20480
$ mkfs.fat freedos.img
```
Create another working directory, cd into it, unzip the archive that we've downloaded, return to the *working root* and create another twos directories.
`dd`is one of the most important utilities in the *unix* world to manipulate at *byte level* **input** and **output**:

> The dd utility copies the standard input to the standard output, applying
any specified conversions.  Input data is read and written in 512-byte
blocks.  If input reads are short, input from multiple reads are
aggregated to form the output block.  When finished, dd displays the
number of complete and partial input and output blocks and truncated
input records to the standard error output.

We're creating here a *virtual* disk with `bs=1024` we're setting both input and output block to `1024`bytes; with `seek=20480` we require `20480`bytes. This is the result:
`-rw-r--r--   1 taglio  taglio    20971520 Feb  3 00:11 freedos.img`. 
Next we format the virtual disk using the `MS-DOS` filesystem. Go ahead:
```sh
$ doas su
# perl stuff/sys-freedos-linux/sys-freedos.pl --drive=freedos.img
# vnconfig vnd0 stuff/fdboot.img
# vnconfig vnd1 freedos.img
# mount -t msdos /dev/vnd0c old/
# mount -t msdos /dev/vnd1c new/
```
We use the `perl` utility from `syslinux` to write the MBR of our virtual disk `freedos.img`. Next we create to `loop` virtual node using the **OpenBSD** utility `vnconfig`. Take care here because *it is quite different from Linux*, but as usual is *clear and simple*. The virtual nodes are associated to the downloaded `fdboot.img` and the newly created `freedos.img`. Next we mount the two virtual nodes `c`partitions; in **OpenBSD** `c`partition describes the entire physical disk. *Quite different from Linux, take care*.
```sh
# cp -ar old/* new/
# cd stuff
$ mkdir o35jy19usa
$ cabextract -d o35jy19usa o35jy19usa_y900.exe
$ doas su
# cp -ar o35jy19usa/* ~/freedos/new/
$ mkdir afudos && cd afudos
$ 7z e ../AFUDOS*
$ doas su
# cp AFUDOS.exe ~/freedos/new/
# umount ~/freedos/old/ && umount ~/freedos/new/
```
Copy all files and directories in the new virtual node partition, extract the Lenovo cabinet in a new directory, copy the result in our new image, extract the `afudos` utility and like the others copy it. Umount the partitions.
## Greetings
Ok dears, i love to share my knowledge. **Knowledge is for all, knowledge is for human kind**. *Bye bye*

*Riccardo Giuntoli*
