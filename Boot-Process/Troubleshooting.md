# Root Password Recovery

Symptoms:
- Forgot root password

Solution:
- Boot with rw init=/sysroot/bin/sh
- chroot /sysroot
- passwd root
- touch /.autorelabel 

# Steps 
# Boot Process

## Reset Root Password (Modern Method)

1. Reboot
2. Press e in GRUB
3. Replace:
   ro
   with:
   rw init=/sysroot/bin/sh
4. Ctrl + x
5. chroot /sysroot
6. passwd root
7. touch /.autorelabel
8. exit
9. reboot -f

# Notes:
- touch /.autorelabel is required when SELinux is enabled.
- chroot changes the root directory to the installed system.