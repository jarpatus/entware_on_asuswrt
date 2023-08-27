# Install entware on stock Asuswrt firmware
Usually Asuswrt-Merlin firmware is used when customizing Asus routers (you really should use it) but if for reason or another you want to stay stock but still use entware, it is possible. There are guides out there which do utilize script_usbmount setting in nvram to run custom script on USB mount but it is possible to do it without nvram so you can plug USB stick to factory resetted router and it just works. Or it is possible to use nvram but simpler scripts than usual.

# Compatibility
Tested with RT-AC86U.

# Background
Entware likes to be installed to /opt. On stock Asuswrt /opt is symlinkked to /tmp/opt, which does not exist. On the other hand, when USB stick is mounted then stock firmware checks if asusware.arm/.asusrouter file exists on the stick and if it does, firmware will symlink /tmp/opt to /tmp/mnt/.../asusware.arm so /opt now points to asusware.arm directory on your USB stick. Additionally /opt/sbin, /opt/bin, /opt/usr/sbin and /opt/usr/bin will be added to the path. Seems that in the past asusware.arm/.asusrouter file was executed as well but that seems no longer to be the case(?).

This behaviour can be witnessed by installing Download Master from GUI and checking contents of the USB stick afterwards.

# Entware installation
- Format USB stick to ext3 file system if not already ext3.
- Create asusware.arm/.asusrouter file to USB stick and chmod to 777:
```
cd /tmp/mnt/.../
mkdir asusware.arm
chmod 777 asusware.arm/
touch asusware.arm/.asusrouter
chmod 777 asusware.arm/.asusrouter
```
Reboot and check that /opt now really is symlink to asusware.arm on your USB stick.
- Install entware
```
cd /opt
wget -O - http://bin.entware.net/aarch64-k3.10/installer/generic.sh | sh
```
Check contents of /opt to validate. Reboot and see that entware installation survives the boot.

# Entware initialization (no nvram)
Stock firmware will do some kind of init by executing files from /opt/etc/init.d on start (by /usr/sbin/app_init_run.sh?) and creates some kind of pseudo PID file(?) for each file named /opt/initfile.1. So in example /opt/etc/init.d/S99debug will be ran and /opt/S99debug.1 will be created. Seems also that apps/services can be enabled and disabled and based on that start or stop will be performed for each init file. This is quite incompatible with entware init so entware must be initialized by executing /opt/etc/init.d/rc.unslung . 

We can piggyback stock init process by creating file /opt/etc/init.d/S99entware without execute flag set. Stock process will then try to "stop" this app/service but we can actually call entware start from there:

```
echo -e "#!/bin/sh\n/opt/etc/init.d/rc.unslung start" > /opt/etc/init.d/S99entware
```

On the other hand, entware scripts will init only executable files so S99entware won't be called this on infinite loop. Downside is that all entware services are first stopped by stock process and then started by entware process. 

# Sources and links
- https://github.com/Entware/Entware/wiki



