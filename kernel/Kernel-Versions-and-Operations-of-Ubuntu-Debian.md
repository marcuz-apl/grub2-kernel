# Kernel Versions and Operations of Ubuntu / Debian 

by macuz-apl | 18 November 2025



## Terminology

** GA = General Availability

** HWE = Hardware Enablement



## Kernel Versions of Ubuntu 22.04

| Ubuntu release | Kernel version        | Kernal name                    |
| -------------- | --------------------- | ------------------------------ |
| 22.04          | 5.15 (GA)             | linux-image-5.15.0-161-generic |
| 22.04.2        | 5.15 (GA), 5.19 (HWE) | linux-image-5.19.0-41-generic  |
| 22.04.3        | 5.15 (GA), 6.2 (HWE)  | linux-image-6.2.0-26-generic   |
| 22.04.4        | 5.15 (GA), 6.5 (HWE)  | linux-image-6.5.0-15-generic   |
| 22.04.5        | 5.15 (GA), 6.8 (HWE)  | linux-image-6.8.0-40-generic   |
| 22.04.x        | latest                | linux-generic-hwe-22.04        |



## Kernel Versions of Ubuntu 24.04

| Ubuntu release | Kernel version       | Kernel name                   |
| -------------- | -------------------- | ----------------------------- |
| 24.04          | 6.8 (GA)             | linux-image-6.8.0-87-generic  |
| 24.04.2        | 6.8 (GA), 6.11 (HWE) | linux-image-6.11.0-40-generic |
| 24.04.3        | 6.8 (GA), 6.14 (HWE) | linux-image-6.14.0-36-generic |



## Kernel Versions of Debian 12/13

| Debian release | Kernel version | Kernel name                |
| -------------- | -------------- | -------------------------- |
| 12.1           | 6.1 (GA)       | linux-image-6.1.0-10-amd64 |
| 12.9           | 6.1            | linux-image-6.1.0-29-amd64 |
| 13.2           | 6.12           | linux-image-6.12.57-deb13  |



## Switch kernels of GA and HWE of Ubuntu 24.04

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



## Stop Upgrading Kernel

For some reasons, we need to stay wherever it is. We can prevent Ubuntu 22.04 from upgrading the kernel by **pinning** kernel packages or by **disabling unattended upgrades**. Pinning is the most direct method; it involves creating a file that prevents any `linux-*` package from being installed or upgraded. Disabling unattended upgrades prevents the system from automatically installing new kernels, although manual `apt upgrade` commands could still install them. 

**Method 1: Pinning kernel packages**

This method creates a rule for `apt` to ignore all kernel packages. 

1. Open a terminal and run the following command to create a new preferences file:

   ```shell
   sudo nano /etc/apt/preferences.d/nolinuxupgrades
   ```

2. Paste the following lines into the file:

   ```
   Package: linux-*
   Pin: version *
   Pin-Priority: -1
   ```

   This configuration tells `apt` to never install or upgrade any package that starts with `linux-`, as indicated by the negative `Pin-Priority`.

    

**Method 2: Disabling unattended upgrades**

This method prevents automatic upgrades, but you will still be notified of available updates and can install them manually. 

1. **Disable the unattended-upgrades service:**

   ```
   sudo systemctl disable --now unattended-upgrades
   ```

   This stops the service and prevents it from starting on the next boot.

2. **Modify the automatic upgrades configuration:**

   - Edit the `20auto-upgrades` file:

     ```
     sudo nano /etc/apt/apt.conf.d/20auto-upgrades
     ```

   - Change the line `APT::Periodic::Unattended-Upgrade "1";` to `APT::Periodic::Unattended-Upgrade "0";`.

   - You may also want to set `APT::Periodic::Update-Package-Lists "0";` if you don't want the system to check for new packages at all, though this is not required to stop upgrades.

   - Save the file and exit the editor. 

**Important considerations**

- **Security:** Disabling automatic upgrades means you will no longer receive critical security patches automatically. You will need to perform manual updates to stay secure.
- **Kernel Modules:** If you have third-party kernel modules, like some Nvidia drivers, they may not be compatible with newer kernels. However, pinning the package is still the best way to prevent the kernel itself from being upgraded.



## Install new Kernel Manually

**1- Install an official Kernel**, that's really easy.

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

**2- Install an unofficial Kernel**, it's a little hassle, but manageable.

Basically we have to find and install the correct version of the very kernel, say `6.1.0-41-amd64`.

```shell
## linux-image (pick up 6.1.0-41-amd64)
wget https://ftp.debian.org/debian/pool/main/l/linux-signed-amd64/linux-image-6.1.0-41-amd64_6.1.158-1_amd64.deb

## linux-headers
wget https://ftp.debian.org/debian/pool/main/l/linux-signed-amd64/linux-headers-amd64_6.1.158-1_amd64.deb

## usb-storage-modules
wget https://ftp.debian.org/debian/pool/main/l/linux-signed-amd64/usb-storage-modules-6.1.0-41-amd64-di_6.1.158-1_amd64.udeb
```

Then Install the linux-headers and linux-image:

```shell
sudo dpkg -i linux-headers-amd64_6.1.158-1_amd64.deb
sudo dpkg -i linux-image-6.1.0-41-amd64_6.1.158-1_amd64.deb
```

**3- Install the specific module**, which introduces more steps as below:

**`.udeb` (micro-deb) packages are strictly intended for use within the Debian Installer environment** and are not meant to be installed on a normal, running Debian system. Installing a `.udeb` on a standard system can cause issues because they are stripped-down, lack proper dependency management for a full system, and are not designed to be uninstalled or upgraded. 

If you need a specific kernel module on your installed system, the standard approaches are:

- **Use the regular `.deb` package version** of the module, which you can install using standard package managers like `apt` or `apt-get`.
- **Compile the module yourself** against your current kernel headers. 

#### Standard Installation of a Kernel Module 

To install a module on a running system, first check if a standard Debian package for it exists in the repositories. 

1. **Search for the module package:**

   ```
   apt search <module-name>
   ```

2. **Install the package using `apt`:**

   ```
   sudo apt install <package-name> 
   ```

#### Manual Method (if no standard package exists)

If no standard package is available, you will need to manually integrate the module file (`.ko` file) into your kernel's module directory. 

1. **Extract the `.udeb` file**: Since a `.udeb` is an `ar` archive, you can extract its contents using standard archive tools.

   ```
   ar x <module-name>.udeb
   tar -xvf data.tar.xz # or data.tar.gz, depending on compression
   ```

2. **Locate the kernel module file**: The extracted files will contain the `.ko` (kernel object) file(s). They are usually found within a directory structure mimicking the `/lib/modules/` path.

3. **Copy the module to the correct directory**: Move the `.ko` file to the appropriate location under `/lib/modules/$(uname -r)/kernel/drivers/` for your current kernel release.

   ```
   sudo cp /path/to/your/module.ko /lib/modules/$(uname -r)/kernel/drivers/<SUBSYSTEM>/
   ```

   *Replace `<SUBSYSTEM>` with the relevant subsystem directory (e.g., `net`, `pci`, `usb`, etc.)*.

4. **Update the module dependencies**: Run `depmod` to register the new module and its dependencies in the system's module database.

   ```
   sudo depmod -a
   ```

5. **Load the module**: Use `modprobe` to load the module into the running kernel.

   ```
   sudo modprobe <module_name> # without the .ko extension
   ```

6. **Verify the module is loaded**:

   ```
   lsmod | grep <module_name>
   ```

7. **Ensure the module loads at boot**: Create a configuration file in `/etc/modules-load.d/` to have the module load automatically on future reboots.

   ```
   echo <module_name> | sudo tee /etc/modules-load.d/<module_name>.conf
   ```



## Remove a Specific Kernel Manually



**1- Check the kernel version you are currently using** with the following command. Ensure the output is **not** `6.8.0-87-generic` or any version you intend to remove. If you are using this kernel, you  must reboot your system and select a different, older, but working  kernel from the GRUB menu's "__Advanced options for Ubuntu__" submenu before proceeding. I chose `kernel 6.2.0-26-generic`.

```shell
## Ensure the output is not 6.8.0-87-generic
uname -srv
```

The output be likes:

```text
Linux 6.2.0-26-generic #26~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Thu Jul 13 16:27:29 UTC 2
```

The output indicates a different kernel version (`6.2.0-26`) against what we are going to remove (`6.8.0-87`), then it's safe to proceed.

> [!TIP]
>
> For some cases, you may forget to switch to another kernel and try to remove the current kernel which is in use. At that case, you will be presented with a warning message advising a no-go. 
>
> You could preceed to remove the current kernel if you have a multiple kernel installed (The system will pick up the second kernel for you). Otherwise, please finish the switch-over.



**2- List the installed kernel packages** to confirm their exact names:

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

**3- Remove the kernel image and headers** using the `apt purge` command, which removes the packages and their config files:

```shell
sudo apt purge linux-{headers,image,modules,modules-extra,tools}-6.8.0-87-generic
sudo apt autoremove
ls -la /boot
```

The process will remove the 5 packages, clean up the `/boot` folder, and update the grub menu.
