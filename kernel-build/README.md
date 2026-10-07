# Build Linux Kernel
## 📦 Install Pre-Requisites
**Compilier Machine:** 🖥️ Ubuntu 64bit x86
```
$ sudo apt install bc binutils bison dwarves flex gcc git gnupg2 gzip libelf-dev libncurses5-dev libssl-dev make openssl pahole perl-base rsync tar xz-utils autoconf gperf autopoint texinfo texi2html gettext gawk bzip2 qemu-system-x86 libtool
```

## 📁 Create Project Directories
Initialize the required directories.
```
$ mkdir edcel-lnx
$ cd edcel-lnx

$ mkdir root

$ cd root

$ mkdir boot
$ mkdir proc
$ mkdir sys
$ mkdir dev
$ mkdir usr

$ cd usr

$ mkdir lib
$ mkdir lib64
$ mkdir bin
$ mkdir sbin

$ cd ..

$ ln -s usr/lib lib
$ ln -s usr/lib64 lib64
$ ln -s usr/bin bin
$ ln -s usr/sbin sbin

#lets check what we have done
$ nautilus ./
#we have dev-proc-and-sys-boot
#we have our links
#and our usr

$ cd ..

$ export LNX="/home/user/edcel-lnx/root"
```

## 📦 Clone Kernel 
Clone inside project folder edcel-lnx.
```
$ git clone --depth 1 https://github.com/torvalds/linux
$ cd linux

$ make tinyconfig
$ make menuconfig
```

## ⚙️ Kernel Configuration
Assited UI Configuration.
```
General setup --->

----Configure standard kernel features --->

--------[*] Enable support for printk

[*] 64-bit kernel

[*] Enable the block layer

Executable file formats --->

----[*] Kernel support for ELF binaries

----[*] Kernel support for scripts starting with #!

Device Drivers --->

----[*] PCI support

----Generic Driver Options --->

--------[*] Maintain a devtmpfs filesystem to mount at /dev

--------[*] Automount devtmpfs at /dev, after the kernel mounted the rootfs

----SCSI device support --->

--------[*] SCSI disk support (this will unlock after serial ATA)

----[*] Serial ATA and Parallel ATA drivers (libata) --->

--------[*] Intel ESB, ICH, PIIX3, PIIX4 PATA/SATA support

----Character devices --->

--------[*] Enable TTY

--------Serial drivers --->

------------[*] 8250/16550 and compatible serial support

------------[*] Console on 8250/16550 and compatible serial port

File systems --->

----[*] The Extended 4 (ext4) filesystem

----Pseudo filesystems --->

--------[*] /proc file system support

--------[*] sysfs file system support
```

## ⚡️ Compile and dump the kernel in the root(boot)
Change `-j8` based on how many cores the complier machine has.
```
$ make -j8

$ mv arch/x86/boot/bzImage ..
$ cd ..
$ mv bzImage root/boot/
```

## 📦 Build Bash
We need a shell in order to interact with our kernel.
```
$ git clone --depth 1 https://git.savannah.gnu.org/git/bash.git

$ mkdir bash-build
$ cd bash-build

#lets show how configure works
$ ../bash/configure --help

$ ../bash/configure --prefix=/usr

$ make -j8
$ make DESTDIR=$LNX install
$ cd ..

$ ln -s bash root/bin/sh

#lets take a look at our bin folder
$ ls root/bin
```

## 📦 Build Coreutils
We need core utilities like cat ,cp, rm... NOTE bootstrap type scripts can be not necessary if you downloaded from a release instead of git.
```
$ git clone --depth 1 https://github.com/coreutils/coreutils

$ mkdir coreutils-build

$ cd coreutils
$ ./bootstrap
$ cd ..

$ cd coreutils-build
$ ../coreutils/configure --without-selinux --disable-libcap --prefix=/usr

$ make -j8
$ make DESTDIR=$LNX install
$ cd ..

#lets take a look at our bin folder
$ ls root/bin
```

## 📦 Build Util-Linux
```
$ git clone --depth 1 https://github.com/util-linux/util-linux

$ mkdir util-build

$ cd util-linux
$ ./autogen.sh
$ cd ..

$ cd util-build
$ ../util-linux/configure --disable-liblastlog2 --prefix=/usr

$ make -j8
$ make DESTDIR=$LNX install
$ cd ..

#lets take a look at our bin folder
4 ls root/bin
```

## 📦 Build Nano
```
$ git clone --depth 1 git://git.savannah.gnu.org/nano.git

$ mkdir nano-build

$ cd nano
$ ./autogen.sh
$ cd ..

$ cd nano-build
$ ../nano/configure --prefix=/usr

$ make -j8
$ make DESTDIR=$LNX install
$ cd ..

#lets take a look at our bin folder
$ ls root/bin
```

## 📦 Build Glibc (Lib)
Gnu C library contains a bunch of good things and it is the basic for any linux. it contains the lib loader ld-linux which allows for dynamic library.
```
$ git clone --depth 1 https://sourceware.org/git/glibc

$ mkdir glibc-build

$ cd glibc-build
$ ../glibc/configure --libdir=/lib --prefix=/usr

$ make -j8
$ make DESTDIR=$LNX install
$ cd ..

#lets take a look at our lib64 folder
$ ls root/lib64
```

## 📦 Build Ncurses (Lib)
Ncurses' libs allows bash (and more) to work, we also need to compile the wide character version which has its libs end in 'w'. (get the latest here: https://ftp.gnu.org/gnu/ncurses/)
```
$ wget https://ftp.gnu.org/gnu/ncurses/ncurses-6.5.tar.gz

$ tar -xvzf ncurses-6.5.tar.gz

$ mkdir ncurses-build

$ cd ncurses-build
$ ../ncurses-6.5/configure --with-shared --with-termlib --enable-widec --with-versioned-syms --prefix=/usr

$ make -j8
$ make DESTDIR=$LNX install
$ cd ..

$ cd root
$ ln -s libncursesw.so.6 lib/libncurses.so.6
$ ln -s libtinfow.so.6 lib/libtinfo.so.6

$ nano etc/ld.so.conf

# Content of the file (exit using Ctrl-x and than y, than Enter).
/usr/lib
/usr/lib64

# Update the ld.so.cache .
$ ldconfig -v -r ./

#note that in the lib shown there is no libncurses.so.6, this is because the ld cache produced by ldconfig does not detect symlinks, this is why we needed the --libdir=/lib in the configure of glibc to add a system search path that can be shown using:
bin/ld.so --help
```

## ⚙️ Setup Init
Now lets make the init, it can be placed in /, /bin or /sbin, always need to be called "init" and be executable.
```
$ nano sbin/init

# Content of the file (exit using Ctrl-x and than y, than Enter).
#!/bin/bash

mount -t proc none /proc
mount -t sysfs none /sys

exec /bin/bash

# Make it executable and leave dir.

$ chmod +x sbin/init
$ cd ..
```

## 🗄️ Create Disk
We need to create a disk (in our case 1GB).
```
$ dd if=/dev/zero of=disk.img bs=1M count=1024

$ fdisk disk.img
n - p - 1 (leave default 2048sectors)
a
w
```

## ⚙️ Mounting disk 
Can follow again to mount existing one
Mounting our disk using losetup, it will find the next available /dev/loop# and give us access to the partitions at /dev/loop#p#. it will also tell us which loop# it chose.
```
$ sudo losetup -fP --show disk.img
```

## ⚙️ Creating partition
Only if you have a new disk.
You can now put the FS of your choice, given that it was configured into your kernel. if you followed this tutorial without change, you will be using EXT4.
```
$ sudo mkfs.ext4 /dev/loop#p1
```

## ⚙️ Mounting partition
This will be the last thing you have to do to mount an not-new image. don't stop the tutorial if its new.
```
$ sudo mount /dev/loop#p1 /mnt
```

## ⚙️ Copy all the data
```
$ sudo cp -R root/* /mnt/
```

## ⚙️ Setup Grub
We need a bootloader to tell our system what to do at boot, Grub is a complete, battle-harden bootloader and you will learn to use it. Lets get it onto our drive!
```
sudo grub-install --target=i386-pc --root-directory=/mnt --no-floppy --modules="normal part_msdos ext2 multiboot" /dev/loop#
```

## ⚡️ Boot time!
Now lets create the boot entry.
```
sudo nano /mnt/boot/grub/grub.cfg

# Content of the file (exit using Ctrl-x and than y, than Enter).
menuentry 'LNX' {
        set root='(hd0,1)'
        linux /boot/bzImage root=/dev/sda1 rw
}

```

### 💡 Explanations: you can change LNX for what you want.

We have only one drive so our root drive and partition will be:

`hd0<-` this is the drive, counting from 0.

`hd0,1<-` this is the partition, counting from 1 (yes, death to the one that made that choice).

For the linux line, it need to point to our kernel image and we need to tell it to mount /dev/sda1 (we have 1 disk so makes sense) and don't forget the rw to tell it that it can write to it, by default you can't.

## ⚡️ Unmount / Sync your disk
Before you run your LNX it is important that you sync your data to insure that whatever modification you made will be written and usable.
```
#I like to spam it 3x
$ sync
```

Then you can Run LNX, but if you are done for the day, it is always important to umount both the FS and the loopback device using:
```
$ sudo umount /mnt
$ sudo losetup -d /dev/loop#
```

## 🔥 Run LNX
```
qemu-system-x86_64 disk.img
```
Here you can do ls to see your file or you can nano a file (lol) , sync and if you quit and come back, it would stay there as LFN has an EXT4 fs it can write onto.