# Network Boot Linux Distributions

## Basic HTTP Boot

The simplest approach: download kernel and initrd via HTTP, then boot.

```
#!ipxe
dhcp
kernel http://192.168.1.20/linux/vmlinuz initrd=initrd.img
initrd http://192.168.1.20/linux/initrd.img
boot
```

### Set Up the HTTP Server

1. Copy the kernel (`vmlinuz`) and initrd (`initrd.img`) from your distribution's install media to a web server directory
2. Ensure the web server is accessible from the boot network

Example with nginx:
```bash
sudo mkdir -p /var/www/html/linux
sudo cp /mnt/iso/images/pxeboot/vmlinuz /var/www/html/linux/
sudo cp /mnt/iso/images/pxeboot/initrd.img /var/www/html/linux/
```

## Fedora / RHEL / CentOS

These distributions provide `vmlinuz` and `initrd.img` in the `images/pxeboot/` directory of the install media.

```
#!ipxe
dhcp
set mirror http://dl.fedoraproject.org/pub/fedora/linux/releases/41/Server/x86_64/os
kernel ${mirror}/images/pxeboot/vmlinuz initrd=initrd.img inst.repo=${mirror}
initrd ${mirror}/images/pxeboot/initrd.img
boot
```

The `inst.repo` kernel argument tells Anaconda where to find the installation packages.

## Ubuntu / Debian

Ubuntu provides a `netboot` tarball with kernel and initrd:

```
#!ipxe
dhcp
set mirror http://archive.ubuntu.com/ubuntu/dists/noble/main/installer-amd64/current/legacy-images/netboot/ubuntu-installer/amd64
kernel ${mirror}/linux initrd=initrd.gz
initrd ${mirror}/initrd.gz
boot
```

For automated installs, pass a preseed or cloud-init URL as a kernel argument:
```
kernel ${mirror}/linux initrd=initrd.gz auto url=http://192.168.1.20/preseed.cfg
```

## Kernel Arguments

Pass kernel command-line arguments on the `kernel` line:

```
kernel http://server/vmlinuz initrd=initrd.img ip=dhcp console=ttyS0,115200n8 quiet
```

Common arguments:
| Argument | Purpose |
|---|---|
| `initrd=<name>` | Name of initrd (must match `initrd` command) |
| `ip=dhcp` | Use DHCP for kernel networking |
| `inst.repo=<url>` | Installation source (Fedora/RHEL) |
| `auto url=<url>` | Preseed file (Debian/Ubuntu) |
| `console=ttyS0,115200n8` | Serial console output |
| `quiet` | Suppress verbose boot messages |

## Multiple Initrds

iPXE supports loading multiple initrd images:

```
#!ipxe
kernel http://server/vmlinuz initrd=initrd.img initrd=extra.img
initrd http://server/initrd.img
initrd http://server/extra.img
boot
```

The kernel receives them concatenated. Useful for adding drivers or microcode to the initial ramdisk.

## Menu-Driven OS Selection

See `skills/infrastructure/scripting` for building interactive boot menus that let operators choose between multiple Linux distributions or installation profiles.
