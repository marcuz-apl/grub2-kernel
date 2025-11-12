# Use GRUB2 command line to boot up

If you are using GRUB 2 as your bootloader, you can use GRUB's command line to choose which partition to boot from and which partition to mount as root.

I'm assuming your ultimate goal is to reformat your old drive, preferably without having to use live media. If you are trying to set up a multi-boot system between the cloned disks, this will allow you to boot to one or the other, but I wouldn't recommend it as a permanent solution.

### 1. Show the GRUB menu while booting

If your system wasn't set up to do this, you can do it by:

- Holding Shift while booting via BIOS, or
- Pressing Esc while booting via UEFI

### 2. Open the GRUB Command Prompt

Once the GRUB menu is up, press c. This should take you to the GRUB command line. I also found that in the course of continuously pressing Esc, you may end up in the GRUB command line anyway. You should see something like:

```
Minimal BASH-like line editing is supported. For the first word, 
TAB lists possible command completions. Anywhere else TAB lists 
possible device or file completions. ESC at any time exits.

grub> _
```

### 3. Find your root directory on your new SSD

Use `ls` to list the drives and partitions:

```
grub> ls

(proc) (hd0) (hd0,gpt2) (hd0,gpt1) (hd1) (hd1,gpt1) (hd1,gpt2)
```

This roughly equates to the output of an `lsblk` BASH command, where `(hd0)` would be `/dev/sda`, `(hd0,gpt1)` would be `/dev/sda1`, etc.

You can then use the `ls` command to find your root directory on the new SSD. Be sure to include the trailing `/` or the command will just display some info about the partition and not the files contained therein.

```
grub> ls (hd0,gpt2)/
lost+found/ boot/ dev/ proc/ run/ sys/ bin lib lib32 lib64 libx32 sbin etc/
home/ media/ mnt/ opt/ root/ snap/ srv/ tmp/ usr/ var/
```

### 4. Set the root directory

Once you've found root, you can set it:

```
grub> set root=(hd0,gpt2)
```

where `(hd0,gpt2)` is the absolute path to your root directory.

### 5. Get kernel and initrd image

These files are typically found in your boot directory. You'll need the file names, including kernel version, for the next step.

```
grub> ls /boot/
efi/ initrd.img initrd.img.old vmlinuz vmlinuz.old grub/
config-5.13.0-27-generic vmlinuz-5.13.0-27-generic memtest86+.bin
memtest86+.elf memtest86+_multiboot.bin System.map-5.13.0-27-generic
initrd.img-5.13.0-27-generic
```

In this case, the files I used were `vmlinuz-5.13.0-27-generic` and `initrd.img-5.13.0-27-generic`. You may also be able to use `vmlinuz` and `initrd.img`, but I have not tested that.

### 6. Boot

Boot your system using the new SSD with the following commands:

```
grub> linux /boot/vmlinuz-X.XX.X-XX-generic root=/dev/sd##
grub> initrd /boot/initrd.img-X.XX.X-XX-generic
grub> boot
```

where `vmlinuz-X.XX.X-XX-generic` and `initrd.img-X.XX.X-XX-generic` is the kernel version you found in the previous step and `##` is the letter and number combination of the partition where your root directory is located on the new SSD. I determined this based on `(hd0,gpt2)` roughly equating to `/dev/sda2` for me.

At this point, you should be booted onto your new SSD. In my case, I wanted to erase my old drive to use it as storage, so I was able to reformat it from the new SSD.