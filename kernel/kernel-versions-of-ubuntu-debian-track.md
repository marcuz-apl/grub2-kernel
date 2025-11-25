# Ubuntu 22+ / Debian 12+ Kernel Versions

by macuz-apl | 18 November 2025



## Kernel Versions of Ubuntu 22.04

| Ubuntu release | Kernel version | Kernal name                    |
| -------------- | -------------- | ------------------------------ |
| 22.04          | 5.15 (GA)      | linux-image-5.15.0-161-generic |
| 22.04.2        | 5.19 (HWE)     | linux-image-5.19.0-41-generic  |
| 22.04.3        | 6.2 (HWE)      | linux-image-6.2.0-26-generic   |
| 22.04.4        | 6.5 (HWE)      | linux-image-6.5.0-15-generic   |
| 22.04.5        | 6.8 (HWE)      | linux-image-6.8.0-40-generic   |
| 22.04.x        | latest         | linux-generic-hwe-22.04        |



## Kernel Versions of Ubuntu 24.04

| Ubuntu release | Kernel version       | Kernel name                   |
| -------------- | -------------------- | ----------------------------- |
| 24.04          | 6.8 (GA)             | linux-image-6.8.0-87-generic  |
| 24.04.2        | 6.8 (GA), 6.11 (HWE) | linux-image-6.11.0-40-generic |
| 24.04.3        | 6.8 (GA), 6.14 (HWE) | linux-image-6.14.0-29-generic |

** GA = General Availability

** HWE = Hardware Enablement



## Kernel Versions of Debian 12/13

| Debian release | Kernel version | Kernel name                  |
| -------------- | -------------- | ---------------------------- |
| 12.1           | 6.1 (GA)       | linux-image-6.1.0-10-generic |
| 12.9           | 6.1            | linux-image-6.1.0-29-generic |
| 13.2           | 6.12           | linux-image-6.12.57-deb13    |



## Switch kernels of Ubuntu 24.04

How to switch kernels, take 24.04.2 as example:

```shell
## from GA to HWE: install the HWE kernel by using:
sudo apt install linux-generic-hwe-24.04
## from HWE to GA: install the stable GS kernel by:
sudo apt install linux-generic
```



## Update/Reinstall a specific kernel

In some cases, the kernel may be broken, then reinstall or update it.

```shell
## The must-have are `image` and `headers` while `modules` and `modules-extra` may be needed
sudo apt install --reinstall linux-{headers,image,modules,modules-extra}-6.14.0-29-generic
## May engage the tools package
sudo apt install --reinstall linux-tools-6.14.0-29-generic
## update the `initrd`
sudo update-initramfs -u -k 6.14.0-29-generic
## then reboot
reboot
```

