
# System Backup and Restore Instructions

## Mount the external USB drive
1. Connect the USB drive to the USB controller.
    - Shut down the system. 
    - In theory, you can probably just hot plug it without shutdown.
    - Reboot the system.
2. Look at `lsblk` to find the name of the new drive device. 
    - Something like `usb1` or `sdb1` (typically `sda1` is the internal boot drive).
3. The drive should already have a file system on it, check.
    - Run `sudo fdisk -l /dev/usb` (or whatever the block device is in `lsblk`).

## Prepare for backup
4. Make a mount point if needed.
   - Look for something like `/mnt/usb`.
   - If it doesn’t exist, create one: `sudo mkdir /mnt/usb`.
5. Mount the identified usb device on the mount point.
   - Run `sudo mount /dev/usb0 /mnt/usb`. This assumes the device is `/dev/usb0`, use the one identified earlier.
6. Check that the usb device appears in the file system.
   - Run `df`. You should see a new entry mounted on `/mnt/usb`.

## Backup process
7. Look at files on the USB drive, delete files, copy files to USB drive as needed.
8. Make a tarball backup of the entire system onto the USB drive (check drive space first).
    ```bash
    cd /
    ```
    ```bash
     tar cpzf /mnt/usb/helium-backup-20230803.tgz --exclude=/boot --exclude=/dev --exclude=/proc --exclude=/run --exclude=/var/cache/apt --exclude=/mnt
    ```
    - This will take a while. You can add the `v` option to `tar` for verbose output.

## Cleanup
9. Unmount the USB drive.
    - Run `umount /dev/usb`.
10. Check that the usb is unmounted.
    - Run `df`. You should see no entry for `/mnt/usb`.
11. Disconnect USB drive.
    - Shut down system.
    - In theory, you can probably just hot unplug it.
    - Reboot system.

