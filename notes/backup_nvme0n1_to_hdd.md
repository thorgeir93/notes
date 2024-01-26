# Background
My working laptob suddendly crashed after I did system update and reboot the PC.
I had USB stick in the laptop and meanwhile the reboot processing was going on I unplugged the USB stick, which wasn't mounted.

Now when I start the PC, I get one option, which is:
```
Reboot into firmware interface
```

# Backup

## Preparing - install live USB
Install USB with live OS. I used Arch.

## First attempt
Tried to use `dd` tool:

```
dd if=/dev/nvme0n1p3 conv=sync,noerror bs=64K | gzip -c  > /path/to/nvme0n1p3_backup.img.gz
```
the archived backup image was now stored in a external drive.
I opened the external drive in another PC and extracted the archive.
then I tried to mount the image:

```
sudo mount -o loop /mnt/backup/nvme0n1p3_backup.img /mnt/backup_mount
```

That did not work.
Then I start to read more.

## Second attempt
Try to do [File system cloning](https://wiki.archlinux.org/title/disk_cloning#File_system_cloning).

I needed to create a new partition on my Icybox HDD raid box.
That partition size needs to be equal or higer than the source hard drive.

Got bytes from the source drive:
```
lsblk --bytes
```
Converted that into MiB using ChatGPT.

Start create the partition:
```
sudo cfdisk /dev/sda
```

Set type as "linux file system".

I did not add any file system format, e.g. `ext4`.

Now we should have partition on my external drive.

Start the broken PC and boot up with the live USB.
Plug the external drive to broken PC.
Identify the hard drives:
```
lsblk -lp
```

then use `e2image` tool.

```
e2image -ra -p /dev/nvme0n1p3 /dev/sdb3
```

Test the newly updated part on the external drive.

# Fixing the PC
Run live OS USB on the broken PC.
Connect to the internet.

```
iwctl device list
iwctl station wlan0 scan
iwctl station wlan0 get-networks
iwctl station wlan0 connect <SSID>
```

Mount the hard drive (where the broken Arch is installed) on the PC and change to this system:
```
mkdir /mnt/os
mount /dev/nvmen0n1p2 /mnt/os
mount /dev/nvmen0n1p1 /mnt/os/boot
mount /dev/nvmen0n1p3 /mnt/os/home
arch-chroot /mnt/os
```

```
pacman -S dracut
pacman -S linux linux-api-headers linux-firmware linux-firmware-whence util-linux
```
See magic command in this [thread](https://forum.endeavouros.com/t/reboot-into-firmware-interface-problem/35427/6)

I was not able to find `kernel-install-for-dracut` module.

Poweroff, remove USB stick, power on, and the machine went up right.


