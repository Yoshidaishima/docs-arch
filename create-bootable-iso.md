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
- download broadcom-wl-dkms (here) [https://archlinux.org/packages/extra/x86_64/broadcom-wl-dkms/]
- download dependencies (**Note** pacman uses non-privileged user for downloads :. dest dir needs to be accessable

```
mkdir /tmp/downloads
pacman -Sy --downloadonly --cachedir /tmp/downloads dkms
```

**Note**

linux headers ```pacman -Q | grep linux-headers``` and kernel version ```uname -r``` must match exactly


