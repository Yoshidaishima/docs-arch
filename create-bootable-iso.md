# Create custom bootable arch iso
> Use case: Installing arch on wifi only systems with proprietary drivers

```
sudo pacman -S archiso
cp -r /usr/share/archiso/configs/releng  ~/custom-archiso

# Support for broadcom proprietary drivers
# echo "broadcom-wl-dkms" >> ~/custom-archiso/packages.x86_64

# Conflicting packages
sed -i 's/broadcom-wl/broadcom-wl-dkms/' ~/custom-archiso/packages.x86_64

# Build iso
sudo mkarchiso -v ~/custom-archiso

# Flash to usb
sudo dd bs=4M if=$ARCH_ISO_PATH of=$USB_DEV_PATH status=progress oflag=sync
```

> If flashing fails in vm, transfer to host and use dd ( some hypervisors cant flash to usb reliably )

### Issues
DKMS is not supported in the arch install medium
- broadcom-wl-dkms sets up dmks hooks
- dkms requires
  - fully installed kernel ( not complete in live usb )
  - matching linux headers ( linux headers are not yet installed )
  - persistent fs ( fs not persistent )

# Simpler options
## Full arch on usb
- boot from arch live usb
- install arch to another usb
  - this will be persistent, can install drivers
## Copy over the packages and install from tar
### Packages
- download broadcom-wl-dkms (here) [https://archlinux.org/packages/extra/x86_64/broadcom-wl-dkms/]
- download dependencies (**Note** pacman uses non-privileged user for downloads :. dest dir needs to be accessable

```
mkdir /tmp/downloads
pacman -Sy --downloadonly --cachedir /tmp/downloads dkms
```

**Note**

linux headers ```pacman -Q | grep linux-headers``` and kernel version ```uname -r``` must match exactly

# Better option
- use regular archiso install medium
- create ext4 usb with pacman packages
- use pacstrap and pacman to install from local repo

### Install what you need in arch vm
At least the following

```
pacman -S base linux linux-firmware
```

Copy entire pacman cache build repo and transfer to usb

```
cp -r /var/cache/pacman/pkg ./pkg
cd pkg && repo-add ./custom.db.tar.zst *.pkg.tar.zst && cd -
scp ./pkg $USB
```

On target machine copy files to cache

```
cp -r $MOUNTED_DRIVE /mnt/var/cache/pacman/pkg
```

### Install Paragon (Mac)
- format usb to Ext4
- transfer files
**Note** could also partition a single usb drive to include the packages and the iso

### Boot & install

- boot the target machine with both the archiso usb and the packages usb inserted
- follow regular archiso install steps
- mount repo to /mnt/repo/pkg
- local install :

```
pacstrap -U /mnt /mnt/repo/pkg/*-pkg.tar.zst
```
