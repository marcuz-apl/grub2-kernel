# Operations on Grub

by Marcuz-Apl



# Operations on Grub

## 1- Configure the Grub timeout

To change the GRUB menu timeout, open a terminal and edit the `/etc/default/grub` file using `sudoedit /etc/default/grub` or `sudo nano /etc/default/grub`. Find the `GRUB_TIMEOUT=` line and change the value to your desired number of seconds (e.g., `GRUB_TIMEOUT=10`), set it to `0` to boot immediately, or `-1` to wait indefinitely. Save the file and run `sudo update-grub` to apply the changes. 

Step 1: Open the GRUB configuration file 

```shell
sudo nano /etc/default/grub
## `sudoedit /etc/default/grub`
```

Step 2: Edit the `GRUB_TIMEOUT` value 

- Find the line of `GRUB_TIMEOUT=` and remove the `#` to enable it.
- Change the number to your desired timeout in seconds.
  - **To set a specific time:** Change it to a number, like `GRUB_TIMEOUT=15` for 15 seconds.
  - **To boot immediately:** Set it to `0`.
  - **To wait indefinitely:** Set it to `-1`.

Step 3: Save and exit the file 

Step 4: Update GRUB 

```shell
sudo update-grub
```



## 2- Set the default kernel in Grub

You can set the default GRUB2 kernel by using either the `grubby` command with the full path to the kernel, or by using `grub2-set-default` with an index number, and then regenerating the GRUB configuration file. For a persistent change after a kernel update, set the default by editing the `/etc/default/grub` file with the `GRUB_DEFAULT=saved` option, setting `GRUB_SAVEDEFAULT=true`, and running `grub2-mkconfig`. 

**Method 1A: Using `grubby` (RHEL/Oracle Linux)** 

1. List installed kernels to find the full path to the desired one.

   ```shell
   ls -la /boot
   ```

2. Set the default kernel using the full path to the `vmlinuz` file:

   ```
   sudo grubby --set-default /boot/vmlinuz-<kernel-version>
   ```

   For example:

   ```
   sudo grubby --set-default /boot/vmlinuz-5.15.0-208.159.3.2.el8uek.x86_64
   ```

   This command changes the default immediately and persists across reboots.

3. Alternatively, set the default by index:

   ```
   sudo grubby --set-default-index=<index-number>
   ```

   For example, to set the first kernel in the list as default:

   ```
   sudo grubby --set-default-index=0
   ```

   You can see the list of indexes by running `grubby --info`.

    

**Method 1B: Using `grub-customizer` (Debian/Ubuntu Linux)** 

1. List installed kernels to find the full path to the desired one.

   ```shell
   ls -la /boot
   ```

2. Add the ppa:

   ```
   sudo add-apt-repository ppa:danielrichter2007/grub-customizer
   ```

3. Update your package list and install the Grub Customizer:

   ```
   sudo apt update
   sudo apt install grub-customizer
   ```

4. Launch Grub Customizer to mess around:

   ```
   grub-customizer
   ```

   

**Method 2: Using `grub2-set-default` (RHEL/CentOS/Fedora and Debian)** 

1. List installed kernels to find the correct index number for the desired kernel. The first entry is usually index 0, the next is 1, and so on.

   ```shell
   sudp grep -P '^menuentry' /boot/grub/grub.cfg
   ```

   The presenting may not be very good, then refer to **Method 3** to take a look.

2. Set the default kernel using its index number:

   ```
   sudo grub-set-default <index-number>
   ```

   For example, to set the second kernel in the list as default:

   ```
   sudo grub-set-default 1
   ```

3. Regenerate the GRUB configuration file. The command depends on your system's firmware:

   - **BIOS:** `sudo grub2-mkconfig -o /boot/grub2/grub.cfg`
   - **UEFI:** `sudo grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg`

4. Reboot your system to apply the changes. 



**Method 3: Editing `/etc/default/grub` (Manual configuration)** 

1. Edit the `/etc/default/grub` file with a text editor:

   ```
   sudo nano /etc/default/grub
   ```
   
2. Set `GRUB_DEFAULT` to `saved` and add the `GRUB_SAVEDEFAULT=true` line. For example:

   ```
   GRUB_DEFAULT=saved
   GRUB_SAVEDEFAULT=true
   ```

3. Save and close the file.

4. Update the GRUB configuration file. The exact command may vary depending on your distribution (e.g., `sudo update-grub` for Debian/Ubuntu, or `sudo grub-mkconfig -o /boot/grub/grub.cfg` for others).

5. During the next boot, you can manually select a kernel from the menu. This selection will be saved as the default for all future boots. 
