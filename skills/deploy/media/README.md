# Create Bootable ISO and USB Images

Build iPXE as bootable media for machines where chainloading or ROM flashing is not practical.

## Build ISO Image

```bash
cd ipxe/src
make bin/ipxe.iso
```

Burn to CD or mount in a VM's virtual CD drive.

### ISO with Embedded Script

```bash
make bin/ipxe.iso EMBED=boot-script.ipxe
```

The script runs automatically on boot. Without an embedded script, iPXE drops to an interactive shell.

## Build USB Image

```bash
cd ipxe/src
make bin/ipxe.usb
```

Write to a USB stick:

```bash
sudo dd if=bin/ipxe.usb of=/dev/sdX bs=4M status=progress
sync
```

> **Warning:** `dd` overwrites the entire device. Triple-check the device path (`/dev/sdX`). Use `lsblk` to identify the correct device.

### USB with Embedded Script

```bash
make bin/ipxe.usb EMBED=boot-script.ipxe
```

## UEFI Media

For UEFI systems, build the EFI binary and create an EFI-bootable image:

```bash
# 64-bit UEFI
make bin-x86_64-efi/ipxe.efi

# ARM64 UEFI (cross-compile)
make CROSS=aarch64-linux-gnu- bin-arm64-efi/ipxe.efi
```

The `.efi` binary can be placed on a FAT32-formatted USB stick in the EFI boot path:
```
/EFI/BOOT/BOOTX64.EFI    # for x86_64
/EFI/BOOT/BOOTAA64.EFI   # for ARM64
```

## Linux Kernel Format

For booting iPXE from an existing bootloader (GRUB, syslinux):

```bash
make bin/ipxe.lkrn
```

GRUB configuration:
```
menuentry "iPXE" {
    linux16 /boot/ipxe.lkrn
}
```

With an embedded script via initrd (no recompilation needed):
```
menuentry "iPXE" {
    linux16 /boot/ipxe.lkrn
    initrd16 /boot/boot-script.ipxe
}
```

## When to Use Each Format

| Format | Best For |
|---|---|
| `.iso` | VMs, one-off testing, machines with CD drives |
| `.usb` | Physical machines without PXE ROM or network |
| `.efi` | UEFI-only machines, EFI shell boot |
| `.lkrn` | Adding iPXE to an existing GRUB/syslinux menu |
