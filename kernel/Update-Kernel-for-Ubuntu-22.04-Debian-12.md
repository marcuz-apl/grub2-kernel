# Update Kernel of Ubuntu 22.04 & Debian 12



## Part 1 - Intro

I've installed Ubuntu 22.04.5 on a old and hardcore workstation: Dell Precision 7820 Tower with very fancy parameters:

- CPU x 2: Intel Xeon 4116 Silver with 24 cores each, and

- GPU x 2: Nvidia Quadro P5000 graphics card with 16GB VRAM each, and

- RAM: 256 GB (64GB x 4), and

- HDD x 4: Toshiba SATA HDD with capacity of 1.8TB each.

The Ubuntu was installed on the 4th HDD, `/dev/sdd` in the naming system of Linux and the **default kernel** is 6.8.0-40,  and upgraded automatically to **6.8.0-87** once installation finished. The **big issue is it can not boot up** with default `xorg video` driver, but going to a black screen. 

Then I installed **Debian 12.9** which runs smoothly with kernel **6.1.0-29**.

It turns out the kernel 6.8.0-40 / 87 of Ubuntu 22.04 has a compatibility issue with the hardware I mentioned above. Then thinking about downgrade to an older kernel to give a try. The two options are:

- the GA **kernel 5.15.0.161** of Ubuntu 22.04.1, and 
- the default **kernel 6.1.0-29** of Debian 12.9.



## Part 2 - Kernel Versions of Ubuntu 22.04

| Ubuntu release | Kernel version | Remarks                             |
| -------------- | -------------- | ----------------------------------- |
| 22.04          | 5.15           | (GA) linux-image-5.15.0-161-generic |
| 22.04.2        | 5.19           | (HWE) linux-image-5.19.0-41-generic |
| 22.04.3        | 6.2            | (HWE) linux-image-6.2.0-26-generic  |
| 22.04.4        | 6.5            | (HWE) linux-image-6.5.0-15-generic  |
| 22.04.5        | 6.8            | (HWE) linux-image-6.8.0-87-generic  |
| 22.04.x        | latest         | (HWE) linux-generic-hwe-22.04       |



## Part 3 - Downgrade Kernel to GA 5.15.0-161 using Boot-Repair tool

Boot with a Live DVD or ISO file of Ubuntu 22.04.5.

```shell
## Install the app
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt update
sudo apt install boot-repair

## mount the kernel files
sudo mount /dev/sdd2 /mnt
sudo mount /dev/sdd1 /mnt/boot/efi

## Remove the current kernel: 6.8.0-40/87 with grub config
sudo rm /mnt/boot/vmlinuz*
sudo rm /mnt/boot/initrd*
sudo rm /mnt/boot/Config*
sudo rm /mnt/boot/System.map*
sudo rm -rf /mnt/boot/efi/EFI/ubuntu
## As such there are 3 memtest86+ files and 2 folders (efi, grub) left.

## Then run the app
boot-repair
## Click "Advanced options" to specify details of the perging and installing.
```

It shall grab the best GA kernel (5.15.0-161) for this old workstation.

Then umount the partitions and reboot the system.

```shell
sudo umount /mnt/boot/efi
sudo umount /mnt

reboot
```

Once it boots up, run command to confirm the kernel version:

```shell
uname -srvm
## Linux 5.15.0-161-generic #171-Ubuntu SMP Sat Oct 11 08:17:01 UTC 2025 x86_64

uname -r
## 5.15.0-161-generic
```

Obviously this GA kernel `5.15.0-161-generic` is in good standing.



## Part 4 - Install Ubuntu 22.04.3 and Remove the updated kernel

After a few hiccups with Ubuntu 22.04.5, I turned to a lower version of Ubuntu 22.04.3, which started with an **initial kernel (GA): 6.2.0-26-generic**, without any issue. But when Installing the Ubuntu release, the kernel actually got **updated to 6.8.0-87-generic** at first place.

Then What we need to do is remove the "too-new" Kernel 6.8.0-87-generic.

1- **Check the kernel version you are currently using** with the following command. Ensure the output is **not** `6.8.0-87-generic` or any version you intend to remove. If you are using this kernel, you  must reboot your system and select a different, older, but working  kernel from the GRUB menu's "__Advanced options for Ubuntu__" submenu before proceeding

```shell
## Ensure the output is not 
uname -srv
```

The output belikes:

```text
Linux 6.2.0-26-generic #26~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Thu Jul 13 16:27:29 UTC 2
```

2- **List the installed kernel packages** to confirm their exact names:

```shell
dpkg --list | grep 6.8.0-87-generic
```

The output belike:

```text
linux-headers-6.8.0-87-generic             6.8.0-87.88~22.04.1
linux-image-6.8.0-87-generic               6.8.0-87.88~22.04.1
linux-modules-6.8.0-87-generic             6.8.0-87.88~22.04.1
linux-modules-extra-6.8.0-87-generic       6.8.0-87.88~22.04.1
linux-tools-6.8.0-87-generic               6.8.0-87.88~22.04.1
```

3- **Remove the kernel image and headers** using the `apt purge` command, which removes the packages and their config files:

```shell
sudo apt purge linux-{headers,image,modules,modules-extra,tools}-6.8.0-87-generic
ls -la /boot
```

The process will remove the 5 packages and update the `/boot` folder, and update the grub menu.



## Part 5 - More Experiments on Kernels for Ubuntu 22.04

An official pool of kernel can be found by:

```shell
apt-cache showpkg linux-image
```

Out of many, the potential candidates of kernel are:

```text
## Out of many, the potential candicates are:
   ## Generic
linux-image-5.15.0-161-generic
linux-image-5.19.0-50-generic (issue: NMD + NSE)
linux-image-6.2.0-39-generic  (issue: NMD + NSE)
linux-image-6.5.0-45-generic  (issue: NMD + NSE)
linux-image-6.8.0-87-generic  (big issue: BO)
   ## Nvidia
linux-image-5.15.0-1091-nvidia
linux-image-5.19.0-1014-nvidia (issue: NMD + NSE)
linux-image-6.2.0-1015-nvidia  (issue: NMD + NSE)
linux-image-6.5.0-1024-nvidia  (issue: NMD + NSE)
linux-image-6.8.0-1043-nvidia  (big issue: BO)
```

The experiments are performed from lower version to higher versions. And during the tests, there are 3 main types of issues:

- Issue #1: No multi-screen display ("**NMD**"), this can be fixed by installing NVIDIA driver.
- Issue #2: Not supporting exFAT FS ("**NSE**"), installing `exfatprogs, exfat-fuse` didn't fix the issue. So reformat the HDD.
- Issue #3: Black-Out ("**BO**"), this is disaster since I cannot do anything but reboot or shutdown.

__1. kernel 6.2.0-39-generic__

So We give a try of `linux-image-6.2.0-39-generic`:

```shell
sudo apt install linux-image-6.2.0-39-generic linux-headers-6.2.0-39-generic -y
## then reboot
reboot
```

It turns out this kernel `6.2.0-39-generic` has some issues: (1) NMD; (2) NSE.

Then remove this kernel out:

```shell
dpkg --list | grep 6.2.0-39-generic
sudo apt purge linux-{headers,image,modules}-6.2.0-39-generic
sudo apt autoremove
ls -la /boot
```

__2. kernel 6.5.0-45-generic__

So We give a try of `linux-image-6.5.0-45-generic`:

```shell
sudo apt install linux-image-6.5.0-45-generic linux-headers-6.5.0-45-generic -y
## then reboot
reboot
```

It turns out this kernel `6.5.0-45-generic` has some issues: (1) NMD; (2) NSE.

Then remove this kernel out:

```shell
dpkg --list | grep 6.5.0-45-generic
sudo apt purge linux-{headers,image,modules}-6.5.0-45-generic
sudo apt autoremove
ls -la /boot
```

__3. kernel 5.19.0-50-generic__

So We switched to the lower version and give a try of `linux-image-5.19.0-50-generic`:

```shell
sudo apt install linux-image-5.19.0-50-generic linux-headers-5.19.0-50-generic -y
## then reboot
reboot
```

It turns out this kernel `65.19.0-50-generic` has some issues: (1) NMD; (2) NSE.

Then remove this kernel out:

```shell
dpkg --list | grep 5.19.0-50-generic
sudo apt purge linux-{headers,image,modules}-5.19.0-50-generic
sudo apt autoremove
ls -la /boot
```



__4. kernel 6.5.0-1024-nvidia__, __kernel 6.2.0.1015-nvidia__, __Kernel 5.19.0-1014-nvidia__: same issue as above.



__5. GA kernel 6.1.0-29-amd64__ of Debian 12.9 (**Part 7**)

First Download the headers and image: 

basically there is no such exact version 6.1.0.29-amd64 anymore, but 6.1.0-41-amd64 on ftp.debian.org (signed) and debian.stanford.edu (unsigned). 

```shell
## linux-image (pick up 6.1.0-41-amd64)
wget https://ftp.debian.org/debian/pool/main/l/linux-signed-amd64/linux-image-6.1.0-41-amd64_6.1.158-1_amd64.deb
# wget https://debian.stanford.edu/debian/pool/main/l/linux/linux-image-6.1.0-41-amd64-unsigned_6.1.158-1_amd64.deb

## linux-headers
wget https://ftp.debian.org/debian/pool/main/l/linux-signed-amd64/linux-headers-amd64_6.1.158-1_amd64.deb
# wget https://debian.stanford.edu/debian/pool/main/l/linux/linux-headers-6.1.0-41-amd64_6.1.158-1_amd64.deb
```

Then install them:

```shell
sudo dpkg -i linux-{headers,image}-6.1.0-41-amd64_6.1.158-1_amd64.deb
```

(Optionally) run command:

```shell
## sudo update-grub
```

Then reboot the machine.

If uninstalling this kernel:

```shell
dpkg --list | grep 6.1.0-41-amd64
## linux-image-6.1.0-41-amd64
sudo apt purge linux-{headers,image}-6.1.0-41-amd64
sudo apt autoremove
```



## Part 6 - Install and Uninstall NVIDIA Driver with a Kernel

Try `linux-image-6.5.0-1024-nvidia`:

```shell
sudo apt install linux-image-6.5.0-1024-nvidia linux-headers-6.5.0-1024-nvidia
## then reboot
reboot
```

Still issue (no multi-screen, no supporting `exfat` filesystem), then install the NVIDIA proprietary driver:

```shell
## Download from NVIDIA Official website
wget https://us.download.nvidia.com/XFree86/Linux-x86_64/580.105.08/NVIDIA-Linux-x86_64-580.105.08.run
chmod +x ./NVIDIA-Linux-x86_64-580.105.08.run
## Install pre-requisites
sudo apt install gcc build-essential libglvnd-dev pkg-config
## Install the Driver
sudo ./NVIDIA-Linux-x86_64-580.105.08.run
## Reboot
reboot
```

Un-install the NVIDIA driver:

```shell
## remove the proprietary Nvidia driver
sudo dpkg -P $(dpkg -l | grep nvidia-driver | awk '{print $2}')
sudo apt autoremove
## Switch back to nouveau driver by downloading the latest version with apt
sudo apt update
sudo apt install xserver-xorg-video-nouveau
## Reboot
reboot
## Once rebooted, check the facts:
lsmod | grep nouveau
```



## Part 7 - Update Debian 12.9 Kernel from 6.1.0-29 to LTS 6.1.148-1

Debian 12.9 comes with Kernel 6.1.0-29-generic, working perfectly.

First Download the headers and image https://deb.sipwise.com/debian/pool/main/l/linux-signed-amd64 for the great collection of signed kernel pools:

```shell
## https://deb.sipwise.com/debian/pool/main/l/linux-signed-amd64/
## https://deb.sipwise.com/debian/pool/main/l/linux-signed-arm64/
## https://deb.sipwise.com/debian/pool/main/l/linux-signed-i386/
wget https://deb.sipwise.com/debian/pool/main/l/linux-signed-amd64/linux-headers-amd64_6.1.148-1_amd64.deb
wget https://deb.sipwise.com/debian/pool/main/l/linux-signed-amd64/linux-image-6.1.0-39-rt-amd64_6.1.148-1_amd64.deb
```

Then install them:

```shell
sudo dpkg -i linux-headers-6.1-amd64
sudo dpkg -i linux-image-6.1-amd64
## Or per downloads from http://deb.sipwise.com/
sudo dpkg -i linux-signed-amd64/linux-headers-amd64_6.1.148-1_amd64.deb
sudo dpkg -i inux-image-6.1.0-39-rt-amd64_6.1.148-1_amd64.deb
```



The Update the Grub:

```shell
update-grub
```

Finally reboot the machine:

```shell
reboot
```

Once it's up, check the kernel version:

```shell
uname -a
```



## Part 8 - Update Kernel 6.1.0-41 to 6.5.10-1 for Debian 12.9

First Download the headers and image:

```shell
wget https://deb.sipwise.com/debian/pool/main/l/linux-signed-amd64/linux-headers-amd64_6.5.10-1~bpo12+1_amd64.deb

wget https://deb.sipwise.com/debian/pool/main/l/linux-signed-amd64/linux-image-6.5.0-0.deb12.4-amd64_6.5.10-1~bpo12+1_amd64.deb
```

Then install them:

```shell
sudo dpkg -i linux-headers-amd64_6.5.10-1~bpo12+1_amd64.deb
sudo dpkg -i linux-image-6.5.0-0.deb12.4-amd64_6.5.10-1~bpo12+1_amd64.deb
```

(Optionally) run command:

```shell
update-grub
```

Then reboot the machine.

```shell
reboot
```

Once it's up, run the command to verify if the kernel has been updated.

```shell
uname -a
```



## Part 9 - Stable and Signed editions of kernel

The famous kernel maintenance tool - `maimline` can provide quite a few kernels, which are most likely unsigned editions. Here is the list of those signed editions from official websites and distributors:

Typical website for downloading kernels: 

https://pkgs.org/download/linux-image or 

https://ftp.debian.org/debian/pool/main/l/linux-signed-amd64 or

https://debian.stanford.edu/debian/pool/main/l/linux

Ubuntu 22.04 (Jammy) LTS

```shell
## Ubunru 22.04 LTS Main amd64
wget http://archive.ubuntu.com/ubuntu/pool/main/l/linux-signed/linux-image-5.15.0-25-generic_5.15.0-25.25_amd64.deb

## Ubuntu 22.04 LTS Proposed Main amd64 (6.1.158-1)
wget https://debian.stanford.edu/debian/pool/main/l/linux/linux-headers-6.1.0-41-amd64_6.1.158-1_amd64.deb
wget https://ftp.debian.org/debian/pool/main/l/linux-signed-amd64/linux-image-6.1.0-41-amd64_6.1.158-1_amd64.deb
# wget https://ftp.debian.org/debian/pool/main/l/linux-signed-amd64/linux-image-amd64_6.1.158-1_amd64.deb

## Ubuntu 22.04 LTS Proposed Main amd64 (Nvidia Oriented)
wget http://archive.ubuntu.com/ubuntu/pool/main/l/linux-signed-nvidia-6.2/linux-image-6.2.0-1006-nvidia_6.2.0-1006.6~22.04.2_amd64.deb
```

https://pkgs.org/download/linux-image for Ubuntu 24.04 (Noble) LTS

```shell
## Ubunru 24.04 LTS Main amd64
wget http://archive.ubuntu.com/ubuntu/pool/main/l/linux-signed/linux-image-6.8.0-31-generic_6.8.0-31.31_amd64.deb
## Ubuntu 24.04 LTS Proposed Main amd64 (generic - 6.14.0-36)
wget http://archive.ubuntu.com/ubuntu/pool/main/l/linux-signed-hwe-6.14/linux-image-6.14.0-36-generic_6.14.0-36.36~24.04.1_amd64.deb
## Ubuntu 24.04 LTS Proposed Main amd64 (Nvidia - 6.14.0-1014)
wget http://archive.ubuntu.com/ubuntu/pool/main/l/linux-signed-nvidia-6.14/linux-image-6.14.0-1014-nvidia_6.14.0-1014.14_amd64.deb
```



## The End