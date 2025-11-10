# Rebuild the Grub2 Bootloader

by marcuz-apl | 24 October 2025



## Table of Contents

[Case Study: Migrate a Legacy BIOS Linux system to UEFI System](#case-study-migrate-a-legacy-bios-linux-system-to-uefi-system)

[Pre-requisite - Prepare the new SATA disk](#pre-requisite---prepare-the-new-sata-disk)

[Solution 1 - I have no Live USB, but a working Ubuntu System](#solution-1---i-have-no-live-usb-but-a-working-ubuntu-system)

[Solution 2 - I do have a Live USB - Better Method](#solution-2---I-do-have-a-live-usb---better-method)

[Post-migration](#post-migration)

[The End](#the-end)



## Case Study: Migrate a Legacy BIOS Linux system to UEFI System

I have a Ubuntu 25.10 system, which I've customized quite a bit and engaged quite some apps and projects going on there, but it's running in a Legacy BIOS mode. Shame on me, using Legacy BIOS in 2025. Now I decided to migrate such to a modern UEFI based system on another disk without re-installing the entire OS and Apps and even those projects.

Generally speaking, the steps are:

1- Attach the Ubuntu 25.10 (Legacy BIOS) disk and a brand new disk to the working machine (also running Ubuntu 25.10).

2- Copy the content of OS, apps and data (in one partition) from the Legacy BIOS SATA disk to a new UEFI SATA disk.

3- Create / configure a bootloader for the new UEFI SATA disk.

4- Test the migrated disk with UEFI System.



## Pre-requisite - Prepare the new SATA disk

Better have a working Ubuntu 25.10 system handy:

- Have a working Linux system (it's a Ubuntu 25.10) ready - **/dev/sda** (200GB)

- Have a new empty SATA Disk as **/dev/sdb** (128GB), will be formatted once booting up.

- Plug in the Legacy BIOS Ubuntu disk as **/dev/sdc** (80 GB)

  

Important Considerations:

- **Secure Boot:** If Secure Boot is enabled, the new bootloader might not be recognized without additional steps or disabling Secure Boot temporarily.



Details can be as below:

#### a) Prepare the new SATA disk: /dev/sdb

```shell
## Install fat32 supporting mtools and partition app: gparted
sudo apt install mtools dosfstools gparted

## lsblk to get info of the partitions
lsblk
## Output belike:
## ......
## sda		8:0		0	  200G	0	disk
## |-sda1	8:1		0		1G	0	part /boot/efi
## |-sda2	8:2		0	198.9G	0	part /
## sdb		8:12	0	  128G  0	disk
## sdc		8:16	0	   80G	0	disk
## |-sdc1	8:17	0		1M	0	part
## |-sdc2	8:18	0	   80G	0	part

## launch the tool: parted
sudo parted /dev/sdb

## label the partition types: aix, amiga, bsd, dvh, gpt, loop, mac, msdos (for MBR), pc98, sun
mklabel gpt

## make partitions in the env of (parted):
mkpart "efi" fat32 1MiB 1024MiB
set 1 esp on
mkpart "system" ext4 1024MiB 100%
quit

## Format the partitions
sudo mkfs.fat -F 32 /dev/sdc1
sudo mkfs.ext4 /dev/sdc2
```

#### b) Copy the OS+Data on /dev/sdb2 to /dev/sdc2

```shell 
## Ensure the disks and partitions
lsblk
## Copy OS+App+Data partition from the legacy BIOS disk /dev/sdc2 to /dev/sdb2
sudo dd if=/dev/sdc2 of=/dev/sdb2 bs=4M
## run fsck
sudo fsck.ext4 -f /dev/sdb2
```

#### c) Detach the Legacy BIOS based HDD: /dev/sdb

In order to make easier, let remove the Legacy BIOS based **/dev/sdc** since it has nothing (No `esp` partition at all) to do with next operations. As such, the new SATA disk `/dev/sdb` stays same.



## Solution 1 - I have no Live USB, but a working Ubuntu System

To create a GRUB bootloader on `/dev/sdb` using the configuration from `/dev/sda`, follow these steps. This assumes you are working within a live Linux environment or have access to a functional Linux installation.

- Identify the root partition (that's `/`) on `/dev/sda`: Determine which partition on `/dev/sda` contains your Linux root filesystem (e.g., `/dev/sda1`, `/dev/sda2`).

  ```shell
  lsblk
  ```

  The output belike:

  ```text
  loop12 ...
  sda		8:0		0	  200G	0	disk
  |-sda1	8:1		0		1G	0	part /boot/efi
  |-sda2	8:2		0	198.9G	0	part /
  sdb		8:16	0	  128G	0	disk
  |-sdb1	8:17	0		1G	0	part
  |-sdb2	8:18	0	  127G	0	part
  sr0		11:0	1	 1024M	0	rom
  ```

  grab the UUID of the 2 disks:

  ```shell
  sudo blkid
  ```

  The output belike:

  ```shell
  /dev/sda1: UUID="9DAD-448F" BLOCK_SIZE="512" TYPE="vfat" PARTUUID="d547aa30-fba7-4b38-a1f0-2036193d72f4"
  /dev/sda2: UUID="5ef58875-6789-48d2-aa0a-23846d5ecc08" BLOCK_SIZE="4096" TYPE="ext4" PARTUUID="ca505717-9fd4-4476-8e2a-7f29b26dd1a6"
  
  /dev/sdb1: UUID="DE5B-E306" BLOCK_SIZE="512" TYPE="vfat" PARTLABEL="efi" PARTUUID="44683ae3-1ff3-42d4-a059-01884a92de8a"
  /dev/sdb2: UUID="bf044976-c0c6-4fcf-822f-085c98009ead" BLOCK_SIZE="4096" TYPE="ext4" PARTLABEL="system" PARTUUID="4a84ae9b-f248-4eac-90e9-df1ab4a78f39"
  ```

  

- Mount the data and root partition from `/dev/sdb` to `/mnt`:

  ```shell
  ## Mounting
  sudo mount /dev/sdb2 /mnt
  sudo mount /dev/sdb1 /mnt/boot/efi
  
  ## Empty folder /boot/efi so far
  ls /mnt/boot/efi 
  ```

  

* Edit `/etc/fstab` file of `/dev/sdb2`:

  ```shell
  blkid
  ## UUID of /dev/sdb1: DE5B-E306
  ## UUID of /dev/sdb2: bf044976-c0c6-4fcf-822f-085c98009ead
  
  sudo nano /mnt/etc/fstab
  ```
  
  The content of `/mnt/etc/fstab` belikes:
  
  ```shell
  # <File system> <mount point>   <type>  <options>     <dump> <pass>
  # / was on /dev/sda2 during curtin installation
  /dev/disk/by-uuid/bf044976-c0c6-4fcf-822f-085c98009ead / ext4 defaults 0 1
  # /boot/efi was on /dev/sda1 during curtin installation
  /dev/disk/by-uuid/DE5B-E306 /boot/efi vfat defaults 0 1
  /swap.img	    none	swap	sw	0	0
  ```
  
  Save the change!
  



* Bind mount necessary system directories from the local running `/dev/sda` to `/mnt`.

  ```shell
  sudo mount --rbind /dev /mnt/dev
  sudo mount --rbind /proc /mnt/proc
  sudo mount --rbind /run /mnt/run
  sudo mount --rbind /sys /mnt/sys
  ```

* Chroot into the mounted system.

  ```shell
  sudo chroot /mnt
  ```

* If error: `grub-install: warning: EFI variables cannot be set on this system` , please refer to Section "bbbbbb", but I would run the commands below prior to next step:

  ```shell
  ## exit from chroot env
  exit
  modprobe efivarfs
  mount -t efivarfs efivarfs /sys/firmware/efi/efivars
  ## Re-enter into chroot env
  sudo chroot /mnt
  ```

  

* **For UEFI systems**: Reinstall the GRUB EFI packages:

  ```shell
  apt-get install --reinstall grub-efi-amd64 shim-signed
  ```

  

* Install GRUB to the target disk: `/dev/sdb`. 

  ```shell
  grub-install /dev/sdb
  ```

  * If using **UEFI**, the command might be:


  ```shell
  ## grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Ubuntu25 /dev/sdb
  ```

* Update GRUB configuration.

  ```shell
  update-grub
  ```

* Exit the chroot environment.

  ```shell
  exit
  ```

  

* (Option) Check the folder of `/mnt/boot/efi`, which shall have a `EFI` folder now. And the `EFI` folder shall embrace 2 sub-folders: `BOOT` and `ubuntu`.

  ```shell
  ls /mnt/boot/efi
  ## EFI
  ls /mnt/boot/efi/EFI
  ## BOOT ubuntu
  ```

  

* Unmount the filesystems.

  ```shell
  sudo umount -R /mnt
  ```



* In case of error: "`cannot umount: /mnt/sys: target is busy`", indicating that a process or user is currently accessing files or directories within the `/mnt/sys` mount point, preventing it from being unmounted, please do:

  ```shell
  ## Use the `fuser` command to list the PIDs of processes accessing `/mnt/sys`:
  fuser -m /mnt/sys
  ## Alternatively, use `lsof` for a more detailed list of open files:
  lsof /mnt/sys
  ## Perform a "lazy" unmount (if necessary): 
  ## For situations where immediate unmounting is not critical and processes cannot be easily terminated
  sudo umount -l /mnt/sys
  ```



## Solution 2 - I do have a Live USB - Better Method

To create/reinstall the GRUB bootloader on Ubuntu using the distribution's ISO, you need to boot from a live USB, access a terminal, mount your installed system's partitions, `chroot` into it, and then run the GRUB installation commands. 

#### Prerequisites

- An Ubuntu installation ISO file, which you can download from the official Ubuntu downloads page.
- A USB flash drive (at least 4GB) and a tool like Rufus or the built-in Ubuntu Startup Disk Creator to make it bootable.
- Access to your computer's BIOS/UEFI settings to boot from the USB drive. 



#### 1- Boot from the Live USB 

1. Insert the bootable Ubuntu USB into your computer.

2. Restart your computer and enter your BIOS or UEFI firmware settings (usually by pressing a key like `F2`, `F10`, `F12`, or `Del` during startup).

3. Change the boot order to prioritize the USB drive.

4. Save the changes and exit the BIOS/UEFI.

5. When the Ubuntu boot menu appears, select **"Try Ubuntu"** to enter the live session. 

#### 2- Identify Your Partitions

Once in the live session, open a terminal (press `Ctrl + Alt + T` or search for "**Terminal**" in the applications menu). 

1. List your disk partitions to identify your main Ubuntu partition and your EFI System Partition (ESP) using the following command:

   ```
   sudo fdisk -l
   ```
   
2. Look for your main Linux partition (usually a large partition with the "Linux" label or type) and note its name (e.g., `/dev/sda2`). Look for your EFI partition (a small partition, often labeled "EFI" or "vfat", typically around 100-500MB) and note its name (e.g., `/dev/sda1`). 

#### 3- Mount the Partitions 

Mount your main Ubuntu partition to the `/mnt` directory. Replace `/dev/sda2` with your actual Linux partition name: 

```
sudo mount /dev/sda2 /mnt
```

If you have a separate `/boot` or `/boot/efi` partition, mount them as well: 

```
# If you have a separate /boot partition
sudo mount /dev/sda3 /mnt/boot

# Mount the EFI partition (replace /dev/sda1 with your EFI partition name)
sudo mount /dev/sda1 /mnt/boot/efi
```

Bind mount necessary system directories: 

```shell
for i in /sys /proc /run /dev; do sudo mount --rbind "$i" "/mnt$i"; done
```

#### 4- Update the `/etc/fstab` file

Note down the UUIDs of /dev/sda1 and /dev/sda2:

```shell
blkid /dev/sda
## UUID of /dev/sda1: 285B-2D13
## UUID of /dev/sda2: bf044976-c0c6-4fcf-822f-085c98009ead

sudo nano /mnt/sdb2/etc/fstab
```

Modify the content of `/etc/fstab` to be:

```shell
# <File system> <mount point>   <type>  <options>     <dump> <pass>
# / was on /dev/sda2 during curtin installation
/dev/disk/by-uuid/bf044976-c0c6-4fcf-822f-085c98009ead / ext4 defaults 0 1
# /boot/efi was on /dev/sda1 during curtin installation
/dev/disk/by-uuid/285B-2D13 /boot/efi vfat defaults 0 1
/swap.img	    none	swap	sw	0	0
```

Save it.

#### 5- Chroot into Your Installation 

Change the root directory to your installed Ubuntu system: 

```
sudo chroot /mnt
```

#### 6- Reinstall and Update GRUB 

Now you are working within your actual installed system's environment.

**a) For UEFI systems: Reinstall the GRUB EFI packages**

```
apt-get install --reinstall grub-efi-amd64 shim-signed
```

**b) Install GRUB to your main disk**. Replace `/dev/sda` with your actual *disk* name (not the partition number, e.g., use `/dev/sda`, not `/dev/sda2`):

```
grub-install /dev/sda
```

(On UEFI systems, this command installs the GRUB files to the EFI partition and sets it as the default bootloader entry).

**c) Update the GRUB configuration** to detect any other operating systems (like Windows):

```
update-grub
```

#### 7- Exit and Reboot 

If no errors occurred, exit the `chroot` environment and unmount the partitions: 

```
exit
sudo umount -R /mnt
sudo umount -R -l /mnt    ## If the system complains: /mnt is busy
```



#### Alternative Method for the steps 1-7 above

For a simpler, often quicker solution, you can use the **Boot-Repair** utility from the live session, which can automate this process. You can install it in the live terminal with the following commands: 

```
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt update
sudo apt install boot-repair
boot-repair
```

Then, simply select "**Recommended Repair**" and follow the prompts. 

Finally, reboot your system. Make sure to remove the live USB drive and change the boot order back to your main hard drive in the BIOS/UEFI settings if necessary. The GRUB menu should now appear, allowing you to boot into Ubuntu normally. 



## Post-migration

After these steps, `/dev/sdb` will have a GRUB bootloader installed, configured to boot the system from `/dev/sdb`. 

Now, it's a good time to remove other disks and boot up with the new SATA disk with GPT/UEFI system.

You may need to adjust your BIOS/UEFI settings to boot from `/dev/sdb` if you intend to use it as the primary boot disk.

Once booted, the new SATA disk becomes `/dev/sda` since no other disks is connected.



#### Update UEFI Boot Entries (if necessary)

Use `efibootmgr` to create or modify UEFI boot entries to point to the GRUB bootloader on `/dev/sda`.

```
sudo efibootmgr -c -d /dev/sda -p 1 -L "My New Ubuntu OS" -l "\EFI\Ubuntu\shimx64.efi" 
```

(Adjust `-p 1` for the correct partition number, `-L` for a descriptive label, and `-l` for the correct bootloader path.)



## The End
