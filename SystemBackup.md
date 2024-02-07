
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
6. Make a mount point if needed.
   - Look for something like `/mnt/usb`.
   - If it doesn’t exist, create one: `sudo mkdir /mnt/usb`.
7. Mount the identified usb device on the mount point.
   - Run `sudo mount /dev/usb0 /mnt/usb`. This assumes the device is `/dev/usb0`, use the one identified earlier.
8. Check that the usb device appears in the file system.
   - Run `df`. You should see a new entry mounted on `/mnt/usb`.

## Backup process
9. Look at files on the USB drive, delete files, copy files to USB drive as needed.
10. Make a tarball backup of the entire system onto the USB drive (check drive space first).
    ```bash
    cd / tar cpzf /mnt/usb/helium-backup-20230803.tgz --exclude=/boot --exclude=/dev --exclude=/proc --exclude=/run --exclude=/var/cache/apt --exclude=/mnt
    ```
    This will take a while. You can add the `v` option to `tar` for verbose output.

## Cleanup
11. Unmount the USB drive.
    - Run `umount /dev/usb`.
12. Check that the usb is unmounted.
    - Run `df`. You should see no entry for `/mnt/usb`.
13. Disconnect USB drive.
14. Shut down system. In theory, you can probably just hot unplug it.
15. Reboot system.
