# Operations on Grub

by Marcuz-Apl



## Configure the Grub timeout

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



