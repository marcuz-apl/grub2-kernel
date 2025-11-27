# Rescue or Reinstall GRUB - the Manual Way

by Drew Howden Tech



Video: https://www.youtube.com/watch?v=ZhxBJ3yh2OY



Say you have a corrupt ubuntu system or any Linux system, To mimic this, I am going to use a VM to showcase this. 



## Environment mimicking

Login to a Linux system and launch a terminal

```shell
sudo rm -rf /boot/grub
```

As such, the `/boot/grub` folder is gone and it will enter into a console of `Grub Rescue/` once you reboot it.



## Fix the grub issues

Boot the system with a live DVD/ISO of Linux system

```shell
sudo fdisk -l
```

you should have `/sda1`, `/sda2` and `/sda3` as below:

```text
Disk /dev/sda: 200 GiB, 21474836800 bytes, 419430400 sectors
Device		   Start		 End	Sectors	Size	Type	
/dev/sda1		2048	 2000895	1998848	976M	EFI System
/dev/sda2	 2000896   419428351  417427456	199G	Linux filesystem

Disk /dev/sdb: 1.82 TiB, 2000398934016 bytes, 3907029168 sectors
Device		   Start		 End	Sectors	Size	Type	
/dev/sdb1	 	2048	 1050623	1048576	512M	EFI System
/dev/sdb2 	 1050624  1049626623 1048576000 500G	Linux filesystem
/dev/sdb3 1049626624  3907028991 2857402368	1.3T	Microsoft basic data
```

let's get started.

```shell
## Then mount the partitions
sudo mount /dev/sdb2 /mnt
sudo mount /dev/sdb1 /mnt/boot/efi
## Install grub2
sudo grub-install --root-directory=/mnt /dev/sdb
## reboot
reboot
```

Now we got a interface of `grub/`, instead of `grub rescue/`.

```shell
ls
## (hd0) (hd0,gpt2) (hd0,gpt1)
## Or
## (hd0) (hd0,gpt3) (hd0,gpt2) (hd0,gpt1)

## Look at the root partition
ls (hd0,gpt3)/
## set the root partition
set root=(hd0,gpt3)
## take a loot at root directory
ls /

## Specify the vmlinuz and the init
linux /boot/vmlinuz	root=/dev/sda3    ## Use tab key to find options of vmlinux
initrd /boot/initrd.img               ## USe tab key to find options of initrd
## Boot the root system
boot
```

It will boot up your Linux system. And the steps above is a temporary fix at `grub/` console.

Open a terminal in the Linux and fix the issue permanently.

```shell
sudo update-grub
## For Arch Linux:
## sudo grub-mkconfig -o /mnt/boot/grub/grub.cfg
```

Hola! the issue of grub got fixed.



## In case of emergency mode encountered

Sometimes, you entered into an emergency mode with system asking you typr "journalctl -xb" to view system logs, "systemctl reboot" to reboot, "systemctl default"  or "exit" to boot into default mode.

This is most likely due to UUID of the filesystem changed. Then, boot your system with a Live System again.

```shell
lsblk -o uuid
## The output belike:
## DF45-FG4F
## 64F2D68C-124D-4312-7A75-CF344456BB00

# sudo fdisk -l ## to confirm the sda scheme

## then update the EFI partition's UUID at /etc/fstab
sudo mount /dev/sda2 /mnt
sudo nano /mnt/etc/fstab
```



## The End