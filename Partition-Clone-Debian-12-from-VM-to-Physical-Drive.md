# Partition-Clone Debian 12.9 from VM to Physical Drive

by marcuz-apl | 16 November 2025



## Table of Contents

[Case Profile](#case-profile)

[Pre-requisite - System Prep](#pre-requisite---system-prep)

[Prepare the problematic SATA disk](#Prepare-the-problematic-SATA-disk)

[Copy the OS-Data Partition](#Copy-the-OS-Data-Partition)

[Create the GRUB2 bootloader](#Create-the-GRUB2-bootloader)

[Post-migration](#post-migration)

[The End](#the-end)



## Case Profile

I have installed a Debian 12.9 system into VMware VM. Since I have acquired quite some experience of grub2 operations, I would like to clone such Debian 12.9 from the VMware VM to the Physical SATA Disk which is big-sized at 1.8TB and already has a Ubuntu 22.04 on /dev/sdb2 (**root** partition) and /dev/sda1 (**efi** partition).

Generally speaking, the steps are:

1- Attach the physical SATA disk to the working VMware VM, running Debian 12.9.

2- Prepare the physical SATA disk (create new partition for Debian clone).

3- Copy the root partition from the Debian VM partition (/dev/sda2) to the prepared SATA partition (/dev/sdb3).

4- Create the GRUB2 bootloader for the newly-prepared SATA disk/partition.

5- Detach the Physical disk from VM and test it on a real-world machine.



## 1- Pre-requisite - System Prep - Attach the physical disk to the VM

First thing first, disable the **Secure Boot** to achieve a smoother operation. 

Have a Live Debian 12.9 ISO handy.

Run your VMware Workstations as Administrator; then add the Physical Disk (the whole disk) as a hard disk into the VM.

- The working Linux system on VM is ready as **/dev/sda** (200GB)

- Uninstall the drivers of the graphic card. Please run command below in VM.

  ```shell
  sudo apt remove --purge open-vm-tools open-vm-tools-desktop
  ```

- The problematic SATA Disk is listed as **/dev/sdb** (1.8TB)

Boot into the Live Debian 12.9 system.



## 2- Prepare the Physical SATA disk in the VM

```shell
## Install fat32 supporting mtools
sudo apt install mtools parted

## lsblk to get info of the partitions
lsblk
## Output belike:
## ......
## sda		8:0		0	  128G	0	disk
## |-sda1	8:1		0		1G	0	part
## |-sda2	8:2		0	  199G	0	part
## sdb		8:16	0	  1.8T	0	disk
## |-sdb1	8:17		0		1G	0	part
## |-sdb2	8:18		0	  127G	0	part
## sr0	   11:0		0	 1024M	0	rom
```

Here is the snapshot:



Let's get started with `fdisk` tool:

```shell
## launch the tool: parted
sudo fdisk /dev/sdb

## command "m" to list the help
m
## print the partitions
p
```

Start from scratch by removing the last 2 partitions on the Physical SATA Disk (/dev/sdb3, /dev/sdb4):

```shell
## Delete partition #4 and #3
d
## Then type in "4"
d
## Then type in "3"
```

If we print the list of partitions (command: `p`), it belikes:

```text 
Disk /dev/sdb: 1.82 TiB, 2000398934016 bytes, 3907029168 sectors
Disk model: VMware Virtual S
Units: sector of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 2FC69765-CA3C-4BAF-ACD3-319568B3720F

Device			Start		End		Sectors	Size	Type
/dev/sdb1	 	 2048	1050623		1048576	512M	EFI System
/dev/sdb2  	  1050624 630198271	  629147648	300G	Linux filesystem
```

Then make partitions within the env of `fdisk`:

```shell
## make partition #3
n
## Partition Number (3-128, default 3):
3
## First sector (630198272-3907029134, default 630198272):
<enter> or 630198272
## Last sector, +/-sectors or +/-size{K,M,G,T,P} (630198272-3907029134, default 3907028991):
+350G
## partition #3 contains a ext4 signature.
## Do you want to remove the signature? [Y]es/[N]o:
Y

## make partition #4
n
## Partition Number (4-128, default 4):
4
## First sector (1364201472-3907029134, default 1364201472):
<enter> or 1364201472
## Last sector, +/-sectors or +/-size{K,M,G,T,P} (1364201472-3907029134, default 3907028991):
<enter> or 3907028991
## partition #4 contains a ntfs signature.
## Do you want to remove the signature? [Y]es/[N]o:
Y

## Check the partition table:
p
```

The output belike:

```text
Device			Start		 End	  Sectors	Size	Type
/dev/sdb1	 	 2048	 1050623	  1048576	512M	EFI System
/dev/sdb2  	  1050624  630198271	629147648	300G	Linux filesystem
/dev/sdb3	630198272 1364201471	734003200	350G	Linux filesystem
/dev/sdb4  1364201472 3907028991   2542827520	1.2T	Linux filesystemt
```

Then save/write the partition table and quit the env and format the partitions:

```shell
## write the results and Quit the env
w
## List the new partitions on /dev/sdb
lsblk
## NAME	  MAJ:MIN  RM	SIZE RO	TYPE  MOUNTPOINTS
## loop0	7:0		0 984.4M  1 loop /usr/lib/live/mount/rootfs/filesystem.sqiashfs
##									 /run/live/rootfs/filesystem.squashfs
## sda		8:0		0	200G  0	disk
## |-sda1	8:1		0	512M  0	part
## |-sda2	8:2		0	199G  0	part
## sdb		8:16	0	1.8T  0	disk
## |-sdb1	8:17	0	512M  0	part
## |-sdb2	8:18	0	300G  0	part
## |-sdb3	8:19	0	350G  0	part
## |-sdb4	8:20	0	1.2T  0	part
## sr0	   11:0		0	1.4G  0	rom	 /usr/lib/live/mount/medium
##									 /run/live/medium
```

The `/dev/sdb` shall be in good standing now.



Then format the partitions:

```shell
## Format the partitions
sudo mkfs.fat -F32 /dev/sdb1
sudo mkfs.ext4 /dev/sdb2
sudo mkfs.fat -F32 /dev/sdb3    ## This takes a while as it's a 1.3TB partition.
```





## 3- Copy the `root` Partition

> [!IMPORTANT]
>
> 1- To move a root partition to another disk using `dd`, you must boot from a live environment, identify the source and destination partitions, and then use `dd` to copy the data. 
>
> 2- After the copy, update the new partition's `/etc/fstab` file with the new UUIDs and reinstall the bootloader to make the new drive bootable. 
>
> 3- While `dd` can create an exact block-by-block copy, it can be dangerous and is best for partitions of the same size; for different sizes or a safer method, using `rsync` after creating a new filesystem is recommended. 

**3A) Boot the system with a Live DVD/ISO of Debian 12.9**.

Assuming the root partition (`/`) at `/dev/sdb3`. If not on one partition, so the same for other partitions.



#### 3B) Synchronize the root partitions using `rsync` command (Recommended)

This file-level copy method is safer and handles different partition sizes better.

```shell
## Mount the source and destination partitions
sudo mkdir /mnt/{src,dst}
sudo mount /dev/sda2 /mnt/src
sudo mount /dev/sdb2 /mnt/dst
## Copy files while preserving permissions, ownership, and timestamps, and excluding virtual folders
sudo rsync -aAXv --info=progress2 --exclude={/dev/*,/proc/*,/sys/*,/run/*,/tmp/*,/mnt/*,/media/*,/swapfile} /mnt/src /mnt/dst
## Umount and remove all
sudo umount -R /mnt/src
sudo umount -R /mnt/dst
sudo rmdir /mnt/{src,dst}
```



## 4- Post-migration: Create/Update the GRUB2 bootloader

> [!IMPORTANT]
>
> **Boot from a Live Debian 12.9 Environment:** Boot the physical machine using your Debian 12.9 Live USB again to perform system adjustments.

* Mount necessary system directories from the local running `/dev/sdb` to `/mnt`.

  ```shell
  sudo mount /dev/sdb3 /mnt
  sudo mount /dev/sdb1 /mnt/boot/efi
  
  ## If booting with the VM, do the below
  # for i in dev dev/pts proc sys run; do sudo mount -R /$i /mnt/$i; done
  ```

* `Chroot` into the mounted system.

  ```shell
  sudo chroot /mnt
  ```

* **Install/Update Hardware Drivers/Firmware:** Install necessary non-free firmware packages for the new physical hardware (CPU, network, graphics, etc.).

  ```shell
  apt update && apt install firmware-linux firmware-linux-nonfree
  ```

* **For UEFI systems** (optional): Reinstall the GRUB EFI packages as needed (Most likely /boot/efi already gave this:

  ```shell
  ## apt-get install --reinstall grub-efi-amd64 shim-signed   ## Most likely /boot/efi already have this 
  ```

* **Rebuild `initramfs`**

  ```shell
  update-initramfs -u -k all
  ```

  

* **Update `/etc/fstab`** on the root partition

  Grab the UUID of the new disk (`/dev/sdb`):

  ```shell
  lsblk -f
  blkid /dev/sdb1
  blkid /dev/sdb2
  blkid /dev/sdb3
  ```

  The output belike:

  ```shell
  /dev/sdb1: UUID="1573-A4E1" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="efi" PARTUUID="f226aded-..."
  /dev/sdb2: UUID="BA1F6BAC-9A4F-4DBE-AD5B-E1B6704B4155" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="ubuntu" PARTUUID="A30DB10F-F8AA-49DF-8C3B-B5B8386EAC79"
  /dev/sdb3: UUID="D1324AE9-FD9B-42C9-B52A-1E56E6806D42" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="debian" PARTUUID="A8F4156E-6EA5-9D41-8BD0-19A348D51C30"
  ```

  Edit `/etc/fstab` file of `/dev/sdb3`:

  ```shell
  nano /mnt/etc/fstab
  ```

  Update the content of `fstab` :

  ```shell
  # <File system> <mount point>   <type>  <options>     <dump> <pass>
  # / was on /dev/sda2 during curtin installation
  UUID=d1324ae9-fd9b-42c9-b52a-1e56e6806d42 / ext4 defaults 0 1
  # /boot/efi was on /dev/sda1 during curtin installation
  UUID=1573-A4E1 /boot/efi vfat defaults 0 1
  /swapfile	    none	swap	sw	0	0
  ```

  Save the change!

  

* Install/Update GRUB bootloader to the target disk: `/dev/sdb`. 

  ```shell
  grub-install /dev/sdb
  ## If error, specify the parameters as below:
  grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=debian /dev/sdb
  ```

  It shall report:

  ```text
  Installing for x86_64-efi platform.
  Installation finished. No error reported.
  ```

* Update GRUB configuration.

  ```shell
  ## Update grub.cfg
  update-grub
  ```

* Exit the `chroot` environment.

  ```shell
  exit
  ```

* Unmount the filesystems and reboot.

  ```shell
  sudo umount -R -l /mnt/boot/efi
  sudo umount -R -l /mnt
  ## reboot
  sudo reboot
  ```



#### Update UEFI Boot Entries (if necessary)

Use `efibootmgr` to create or modify UEFI boot entries to point to the GRUB bootloader on `/dev/sdb`.

```
sudo efibootmgr -c -d /dev/sdb -p 1 -L "debian12" -l "\EFI\Debian\shimx64.efi" 
```

(Adjust `-p 1` for the correct partition number, `-L` for a descriptive label, and `-l` for the correct bootloader path.)



## The End
