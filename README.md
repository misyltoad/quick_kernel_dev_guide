# Quick Kernel Development Setup Guide

## Step 1

### Make some rootfs to play with your kernel in:

```
virt-builder debian-12 \
        --arch x86_64 \
        --hostname kerneldev \
        --root-password password:kerneldev \
        --size 16G \
        --update \
        --format raw \
        --install bcachefs-tools \
        --selinux-relabel \
        -o ./rootfs_debian
```

This creates an up-to-date 16GB Debian rootfs called rootfs_debian with the root password `kerneldev` and the hostname `kerneldev`.

This also installs the package `bcachefs-tools` in the image. Neat!

## Step 2
### Build your kernel:

This guide assumes you know the basics of how to build a kernel in general.

The summary:
```
make defconfig
[ edit defconfig to include the features of what you are working on ]
make -j$(nproc)
```

*Note: For Fedora, and other images, you will need a kernel with CONFIG_BTRFS_FS=y which is NOT the default with defconfig.*

## Step 3
### Run your kernel

```
qemu-system-x86_64 \
    -drive file=rootfs_debian,format=raw,if=virtio \
    -enable-kvm \
    -append "root=/dev/vda1 console=ttyS0" \
    -kernel arch/x86/boot/bzImage \
    -cpu host \
    -nographic \
    -m 16G \
    -smp 8
```

This makes some VM with 16GB of RAM and 8 cores using our rootfs.

This uses your terminal for input and output because it specifies `-nographic`.

You can exit/quit the VM using `CTRL+A X`

`vda1` is just what happens to match the config for that Debian rootfs. If using another distro or version, you may need to change that number to eg. `vda2` or `vda3`. The quickest way to find it out is experimentation.

No need for initrd/ramdisk as all the modules are included in our `bzImage`.


### Want something more advanced like GPU passthru for developing GPU drivers?

Read this guide! It's great. You might have better luck with `virt-manager` than the commandline for launcing qemu.

https://wiki.archlinux.org/title/PCI_passthrough_via_OVMF


