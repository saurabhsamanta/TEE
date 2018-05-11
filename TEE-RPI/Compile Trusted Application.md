# Compile Trusted Application
### Project 
```
$ cd /rpi3/
$ mkdir optee_my_world
$ cd optee_my_world
```
Add TA/RA template from /rpi3/optee_examples
### Script to Compile
##### Run this script in folder which is copied in /rpi3/optee_my_world/compile_me.sh

```
#!/bin/bash


# Normal world user space is 64-bit:
export HOST_CROSS_COMPILE=$PWD/../toolchains/aarch64/bin/aarch64-linux-gnu-


# Normal world user space is 32-bit:
#export HOST_CROSS_COMPILE=$PWD/../toolchains/aarch32/bin/arm-linux-gnueabihf-

# TARGET Platform
#export PLATFORM=vexpress
#export PLATFORM_FLAVOR=qemu_virt
export PLATFORM=PLATFORM=rpi3

# Trusted World Compiler
export TA_CROSS_COMPILE=$PWD/../toolchains/aarch32/bin/arm-linux-gnueabihf-

# Path to Client Library
export TEEC_EXPORT=$PWD/../optee_client/out/export

# Path to TA Development KIT
export TA_DEV_KIT_DIR=$PWD/../optee_os/out/arm/export-ta_arm32

# Build Command
make
```
### UUID for Trusted Application
##### Generate UUID Online
```
https://www.uuidgenerator.net/
```
##### Generate UUID Local
```
$ uuidgen
```
##### Change UUID in following files
```
rpi3/optee_my_world/ta/include/my_world_ta.h
rpi3/optee_my_world/ta/Makefile
```
### Compile
```
$ make clean
$ source compile_me.sh
```
### Path to Copy in RootFS
##### TA Application Path
```
/srv/nfs/rpi/lib/optee_armtz
```
##### REE Application Path
```
/srv/nfs/rpi/bin
```
##### Run in RPi