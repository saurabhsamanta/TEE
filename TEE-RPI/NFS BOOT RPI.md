# NFS BOOT RPI
## 1. TFTPD
```
$ sudo apt-get install atftpd
$ sudo gedit /etc/default/atftpd
```
##### Replace with following 
```
USE_INETD=false
OPTIONS="--tftpd-timeout 300 --retry-timeout 5 --mcast-port 1758 --mcast-addr 239.239.239.0-255 --mcast-ttl 1 --maxthread 100 --verbose=5 /tftpboot"
```
##### Create the tftpboot folder and change the permissions
```
$ sudo mkdir /tftpboot
$ sudo chmod -R 777 /tftpboot
$ sudo chown -R nobody /tftpboot
```
##### Add & Update Firewall
```
$ sudo iptables -A INPUT -p tcp --dport 69 -j ACCEPT
$ sudo iptables -A INPUT -p udp --dport 69 -j ACCEPT
$ sudo iptables -L
```

```
$ sudo ufw allow tftp
```
##### Restart the daemon
```
$ sudo /etc/init.d/atftpd restart
```
##### Test TPTF Server
```
$ sudo tftp localhost
tftp> status
Connected to localhost.
Mode: netascii Verbose: off Tracing: off
Rexmt-interval: 5 seconds, Max-timeout: 25 seconds
tftp> get Image
Received 10254866 bytes in 0.7 seconds
tftp> quit
```


## 2. NFS 
##### Install 
```
sudo apt-get install nfs-kernel-server
```
##### Edit Configuration
```
$ sudo gedit /etc/exports
```
##### Modify accodingly ip range <192.168.1.0>
```
/srv/nfs/rpi 192.168.1.0/24(rw,sync,no_root_squash,no_subtree_check)
```
##### Create folder
```
$ sudo mkdir /srv/nfs/rpi
```
##### Restart Service
```
$ service nfs-kernel-server restart
```
## 3. Share & Add Files 

##### Create links in /tftpboot
```
$ cd /tftpboot
$ ln -s /rpi3/linux/arch/arm64/boot/Image
$ ln -s /rpi3/arm-trusted-firmware/build/rpi3/debug/optee.bin 
$ ln -s /rpi3/linux/arch/arm64/boot/dts/broadcom/bcm2710-rpi-3-b.dtb 
```
##### Copying Root FS
```
$ cd /srv/nfs/rpi
$ gunzip -cd /rpi3/build/../gen_rootfs/filesystem.cpio.gz | sudo cpio -idmv
$ sudo rm -rf /srv/nfs/rpi/boot/*
```
## 4. Update uboot.env
##### Connect UART
```
$ sudo apt-get install minicom
Configure 
$ sudo minicom -s
$ sudo minicom
```
##### Power up the Raspberry Pi and almost immediately hit any key and you should see the U-Boot> prompt.
```
U-Boot> setenv serverip '192.168.1.3' 
U-Boot> setenv gatewayip '192.168.1.1'
U-Boot> setenv netmask '255.255.255.0'
U-Boot> setenv rootpath '/srv/nfs/rpi'
U-Boot> setenv tftpserverip '192.168.1.3'
U-Boot> saveenv                       
Saving Environment to FAT...
writing uboot.env
FAT: Misaligned buffer address (000000003ab31b30)
done
U-Boot> printenv
U-Boot> run nfsboot
```


## 5. Source
```
https://www.op-tee.org/docs/rpi3/
```