# OPTEE RPI TUTORIAL

## 1.  Prerequisites

```
$ sudo apt-get install android-tools-adb android-tools-fastboot autoconf \
	automake bc bison build-essential cscope curl device-tree-compiler flex \
	ftp-upload gdisk iasl libattr1-dev libc6:i386 libcap-dev libfdt-dev \
	libftdi-dev libglib2.0-dev libhidapi-dev libncurses5-dev \
	libpixman-1-dev libssl-dev libstdc++6:i386 libtool libz1:i386 make \
	mtools netcat python-crypto python-serial python-wand unzip uuid-dev \
	xdg-utils xterm xz-utils zlib1g-dev

$ sudo apt-get install ccache
```

## 2. Android Repo & Initialization

##### Make sure you have a bin/ directory in your home directory and that it is included in your path:
```
$ mkdir ~/bin
$ PATH=~/bin:$PATH
```

##### Download the Repo tool and ensure that it is executable:
```
$ sudo bash
$ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```
##### Configure git with your real name and email address. To use the Gerrit code-review tool, you will need an email address that is connected with a registered Google account. Make sure this is a live address at which you can receive messages. The name that you provide here will show up in attributions for your code submissions.

```
$ git config --global user.name "Your name"
$ git config --global user.email "you@example.com"
```
## 3. Get the Source Code
##### Choose the manifest corresponding to the platform you intend to use. For example, if you intend to use Raspberry Pi3, then ${TARGET}.xml should be rpi3.xml. The repo sync step will take some time if you aren’t referencing an existing tree
```
$ mkdir -p $HOME/devel/optee
$ cd $HOME/devel/optee
$ repo init -u https://github.com/OP-TEE/manifest.git -m {TARGET}.xml [-b ${BRANCH}]
$ repo sync -j3
```
##### For RPI3
```
$ sudo repo init -u https://github.com/OP-TEE/manifest.git -m rpi3.xml
```
##### Repository Synchronization
```
$ repo sync -j3
```
## 4. Get the toolchains
##### In OP-TEE we’re using different toolchains for different targets (depends on ARMv7-A ARMv8-A 64/32bit solutions). In any case start by downloading the toolchains by: 
```
$ cd build
$ make toolchains -j3
```
## 5. Build 
```
$ make all
$ make update_rootfs
```
## 6. Flash
```
$ make img-help
```
##### Follow the instructions
```
$ fdisk /dev/sdx   # where sdx is the name of your sd-card
   > p         	# prints partition table
   > d         	# repeat until all partitions are deleted
   > n         	# create a new partition
   > p         	# create primary
   > 1         	# make it the first partition
   > <enter>   	# use the default sector
   > +32M      	# create a boot partition with 32MB of space
   > n         	# create rootfs partition
   > p
   > 2
   > <enter>
   > <enter>   	# fill the remaining disk, adjust size to fit your needs
   > t         	# change partition type
   > 1         	# select first partition
   > e         	# use type 'e' (FAT16)
   > a         	# make partition bootable
   > 1         	# select first partition
   > p         	# double check everything looks right
   > w         	# write partition table to disk.

run the following as root
   $ mkfs.vfat -F16 -n BOOT /dev/sdx1
   $ mkdir -p /media/boot
   $ mount /dev/sdx1 /media/boot
   $ cd /media
   $ gunzip -cd /home/trustzone/build/../gen_rootfs/filesystem.cpio.gz | sudo cpio -idmv "boot/*"
   $ umount boot

run the following as root
   $ mkfs.ext4 -L rootfs /dev/sdx2
   $ mkdir -p /media/rootfs
   $ mount /dev/sdx2 /media/rootfs
   $ cd rootfs
   $ gunzip -cd /home/android/rpi3/build/../gen_rootfs/filesystem.cpio.gz | sudo cpio -idmv
   $ rm -rf /media/rootfs/boot/*
   $ cd .. && umount rootfs

```

## 7. Load tee-supplicant & Test
```
$ tee-supplicant &
$ xtest
```
## 8. Source
```
https://www.op-tee.org/build/
```