# Entware on stock Asuswrt
Usually Asuswrt-Merlin firmware is used when customizing Asus routers (you really should use it) but if for reason or another you want to stay stock and still use entware, it is possible. There are guides out there which do utilize script_usbmount setting in nvram to run custom script on USB mount, but it is possible to do it without nvram as well. Also according to some sources these nvram variables do longer work with some models and newest firmwares.

This guide taps into out-of-the-box Asuswrt mechanism to install additional features like Download Master and overlays custom entware installation on top of stock Asuswrt. No need for nvram changes and it is possible to plug USB stick to factory resetted router and it just works.

# Compatibility
Tested with RT-AX86U Pro. YMMV.

# Background
Entware must to be installed to /opt. On stock Asuswrt /opt is either symlinkked to /tmp/opt, which does not exist, or subfolders under /opt are symlinkked to /tmp/opt/... which do not exist either. Looks like older models symlink whole /opt and newer models folders under /opt.

When an USB stick is plugged in and mounted then stock firmware checks if asusware.arm/.asusrouter file exists on the stick and if it does, firmware will symlink /tmp/opt to /tmp/mnt/.../asusware.arm so /opt now points to asusware.arm directory on your USB stick. 

Additionally /opt/sbin, /opt/bin, /opt/usr/sbin and /opt/usr/bin will be added to the path. Seems that in the past asusware.arm/.asusrouter file was executed as well but that seems no longer to be the case. 

Stock firmware will then do initialization by running /usr/sbin/app_init_run.sh which executes service specific init files from /opt/etc/init.d if service has been set as enabled in /opt/lib/ipkg/info/service.control file.

This behaviour can be witnessed by installing Download Master from GUI and checking contents of the USB stick afterwards. Like said earlier, exact behaviour seems to differ from model to model or firmware to firmware so adjustments may be necessary .





# Entware installation
We are goi
- Format USB stick to ext3 file system if not already ext3.
- Create asusware.arm/.asusrouter file to USB stick:
```
cd /tmp/mnt/.../
mkdir asusware.arm
touch asusware.arm/.asusrouter
```
Reboot and check that /opt now is symlink to asusware.arm on your USB stick.
- Install entware
```
cd /opt
wget -O - http://bin.entware.net/aarch64-k3.10/installer/generic.sh | sh
```
Check contents of /opt to validate. Reboot and see that entware installation survives the boot.

# Entware initialization
Stock firmware will do init by executing files from /opt/etc/init.d on start by /usr/sbin/app_init_run.sh. Services can be enabled and disabled and based on that start or stop option will be provided for each init file. This is incompatible with entware init so entware must be initialized by executing /opt/etc/init.d/rc.unslung . 

We can piggyback stock init process by creating file /opt/etc/init.d/S99entware without execute flag set. Stock process will then try to stop this service since it is not enabled but we can actually call entware start from there:

```
echo -e "#!/bin/sh\nchmod -x /opt/etc/init.d/S99entware\n/opt/etc/init.d/rc.unslung start" > /opt/etc/init.d/S99entware
```

On the other hand, entware scripts will init only executable files so S99entware won't be called this no infinite loop. Downside is that all entware services are first stopped (though they are not running) by stock process and then started by entware process. 

# No entware initialization
Further research revealed that it is also possible to allow stock init to init entware services as well without doing entware initialization hack above. Stock process will check if file /opt/lib/ipkg/info/service.control exists and if it contains line "Enabled: yes". If so then init file is called with start option. Just create control file for service you want to enable e.g. for lighttpd do: 

```
mkdir /opt/lib/ipkg
mkdir /opt/lib/ipkg/info
echo "Enabled: yes" > /opt/lib/ipkg/info/lighttpd.control
```

This can, of course, break on Asuswrt and entware upgrades.

# Sources and links
- https://github.com/Entware/Entware/wiki



