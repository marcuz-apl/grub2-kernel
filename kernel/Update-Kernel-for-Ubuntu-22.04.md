# Update Kernel of Ubuntu 22.04



## Intro

I've installed Ubuntu 22.04 on a old and hardcore workstation: Dell Precision 7820 Tower with very fancy parameters:

- CPU x 2: Intel Xeon 4116 Silver with 24 cores each, and

- GPU x 2: Nvidia Quadro P5000 graphics card with 16GB VRAM each, and

- RAM: 256 GB (64GB x 4), and

- HDD x 4: Toshiba SATA HDD with capacity of 1.8TB each.

The Ubuntu was installed on the 4th HDD, `/dev/sdd` in the naming system of Linux and the **default kernel** is 6.8.0-40,  and upgraded automatically to **6.8.0-87** once installation finished. The **big issue is it can not boot up** with default `xorg video` driver, but going to a black screen. 

Then I installed **Debian 12.9** which runs smoothly with kernel **6.1.0-29**.

It turns out the kernel 6.8.0-40 / 87 of Ubuntu 22.04 has a compatibility issue with the hardware I mentioned above. Then thinking about downgrade to an older kernel to give a try. The two options are:

- the GA **kernel 5.15.0.161** of Ubuntu 22.04.1, and 
- the default **kernel 6.1.0-29** of Debian 12.9.



## Kernel Versioning of Ubuntu 22.04

| Ubuntu release | Kernel version | Remarks                             |
| -------------- | -------------- | ----------------------------------- |
| 22.04          | 5.15           | (GA) linux-image-5.15.0-161-generic |
| 22.04.2        | 5.19           | (HWE) linux-image-5.19.0-generic    |
| 22.04.4        | 6.5            | (HWE) linux-image-6.5.0-generic     |
| 22.04.5        | 6.8            | (HWE) linux-image-6.8.0-40-generic  |
| 22.04.x        | latest         | (HWE) linux-generic-hwe-22.04       |



## Downgrade Kernel to (GA) 5.15.0-161 using Boot-Repair tool

Boot with a Live DVD or ISO file of Ubuntu 22.04.

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



## Update kernel 5.15.0-161 to 6.1.0-158 for Ubuntu 22.04

An official pool of kernel can be found by:

```shell
apt-cache showpkg linux-image
```

Out of many, the potential candidates of kernel are:

```text
## Out of many, the potential candicates are:
   ## Generic
linux-image-5.19.0-50-generic
linux-image-6.2.0-39-generic (tested issues)
linux-image-6.5.0-45-generic (tested issues)
   ## Nvidia
linux-image-5.15.0-1091-nvidia
linux-image-5.19.0-1014-nvidia
linux-image-6.2.0-1015-nvidia
linux-image-6.5.0-1024-nvidia (tested issues)
linux-image-6.8.0-1043-nvidia
```

__1. kernel 6.5.0-45-generic__

So We give a try of `linux-image-6.5.0-45-generic`:

```shell
sudo apt install linux-image-6.5.0-45-generic linux-headers-6.5.0-45-generic
## then reboot
reboot
```

It turns out this kernel `6.5.0-45-generic` has some issues: (1) no multi-screen; (2) not supporting Removable HDD in `exfat` filesystem, etc.

Then reboot and select "Kernel 5.15.0-161-generic" and then remove this kernel out:

```shell
sudo apt remove linux-image-6.5.0-45-generic linux-headers-6.5.0-45-generic
## It should update the grub
```

__2. kenel 6.5.0-1024-nvidia__

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

Uninstall the NVIDIA driver:

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

__3. kernel 6.2.0-39-generic__

Also tried `6.2.0-39-generic`, which is similar as `6.5.0-1024-nvidia`, but having no issue when installing the NVIDIA proprietary driver.

__4. kernel 6.5.10-generic__ from deb.sipwise.com

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

__5. kernel 6.1.158-1-generic__ from debian.stanford.edu and ftp.debian.org

```shell
## Downloads
wget https://debian.stanford.edu/debian/pool/main/l/linux/linux-headers-6.1.0-41-amd64_6.1.158-1_amd64.deb
wget https://ftp.debian.org/debian/pool/main/l/linux-signed-amd64/linux-image-6.1.0-41-amd64_6.1.158-1_amd64.deb
## Installs

```



## Update Debian 12.9 Kernel from 6.1.0-41 to LTS 6.1.148-1

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



## Update Kernel 6.1.0-41 to 6.5.10-1 for Debian 12.9

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



## Stable and Signed editions of kernel

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