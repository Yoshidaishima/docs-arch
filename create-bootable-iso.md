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
