# Reinstall Debian on an existing LUKS+LVM setup
Posted on May 28, 2015  

The installer of Debian Jessie does not support existing LUKS containers out of the box. Here is what worked for me to achieve a fresh install of Debian on a existing LUKS/LVM setup (tested on a virtual image with virtualbox):  

Use expert install (did not try this with the default install)  
Follow normal install procedure and select the crypto-dm-modules when asked for additional modules  
At the ‘disk partioning’ first switch to another terminal, e.g.  

**< Ctrl > + < Alt > + < F2 >**  

```no-highlight
$ anna-install crypto-dm-modules cryptsetup-udeb  
$ cryptsetup luksOpen /dev/foo foo_crypt  
$ vgscan  
$ lvdisplay 
Find the concerning volume group and activate it with:  
$ vgchange -a y debian-vg  
```

Swith back to installer terminal:  

**< Ctrl > + < Alt > + < F1 >**  

Choose ‘manual’ partitioning  
Configure LVM  
Choose keep existing configuration  
Configure your boot, root, home, swap partitions (fs, mount point, format yes/no, etc)  
Do not forget to specify the boot partition and probably you do not want to format /home  
Continue installing and stop before rebooting  

Switch back to the terminal with:  

**< Ctrl > + < Alt > + < F2 >**  

Prepare for chroot:  

Mount root partition (LVM) on /mnt and boot partition on /mnt/boot in the following order:  

```no-highlight
$ mount /dev/mapper/debian-root /mnt  
$ mount /dev/sda1 /mnt/boot  

$ mount –bind /dev /mnt/dev  
$ mount –bind /sys /mnt/sys  
$ mount –bind /proc /mnt/proc  
```

Chroot into the installed system:  

```no-highlight
$ chroot /mnt  
$ nano /etc/crypttab  
Add “foo_crypt /dev/foo none luks” without the quotes and save  
$ update-initramfs -k all -u  
```

Clean up actions:  

```no-highlight
$ exit   
$ umount /mnt/dev  
$ umount /mnt/sys  
$ umount /mnt/proc  
$ umount /mnt/boot  
$ umount /mnt  
```

Switch back to the terminal with:  

**< Ctrl > + < Alt > + < F1 >**  

Finish the install and reboot.  

After booting into the fresh install I replaced the device name with its UUID in /etc/crypttab and updated initramfs again:  

```no-highlight
$ ls -l /dev/disk/by-uuid  
Find the UUID  
$ sudo nano /etc/crypttab  
Change to “foo_crypt UUID=xxxxxxxxxxxxxx none luks” without the quotes and save  
$ sudo update-initramfs -k all -u  
```

Your all set!  
Improvements are welcome.  

Thanks to:  

https://blog.hartwork.org/?p=1757  
and the comments from Eddy
