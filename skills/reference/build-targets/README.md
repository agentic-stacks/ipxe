# iPXE Build Targets Reference

Build syntax: `make [platform/]<driver>.<extension>`

## Platforms

| Platform | Architecture | Firmware | Example |
|---|---|---|---|
| `bin/` (default) | x86 | BIOS (PCBIOS) | `make bin/undionly.kpxe` |
| `bin-i386-efi/` | x86 32-bit | UEFI | `make bin-i386-efi/ipxe.efi` |
| `bin-x86_64-efi/` | x86 64-bit | UEFI | `make bin-x86_64-efi/ipxe.efi` |
| `bin-x86_64-pcbios/` | x86 64-bit | BIOS | `make bin-x86_64-pcbios/ipxe.usb` |
| `bin-i386-linux/` | x86 | Linux userspace | `make bin-i386-linux/ipxe.linux` |
| `bin-x86_64-linux/` | x86 64-bit | Linux userspace | `make bin-x86_64-linux/ipxe.linux` |
| `bin-arm64-efi/` | ARM64 | UEFI | `make bin-arm64-efi/ipxe.efi` |
| `bin-arm32-efi/` | ARM32 | UEFI (limited) | `make bin-arm32-efi/ipxe.efi` |
| `bin-arm64-linux/` | ARM64 | Linux userspace | `make bin-arm64-linux/ipxe.linux` |

EFI platforms with `-sb` suffix are for Secure Boot signing submissions.

## Cross-Compilation

```bash
# ARM64
make CROSS=aarch64-linux-gnu- bin-arm64-efi/ipxe.efi

# ARM32
make CROSS=arm-linux-gnueabi- bin-arm32-efi/ipxe.efi
```

## Drivers

| Driver | Description |
|---|---|
| `ipxe` | All PCI NIC drivers bundled (large) |
| `undionly` | BIOS UNDI driver only (small, for chainloading) |
| `snponly` | UEFI SNP driver, first NIC only |
| `snp` | UEFI SNP driver, all NICs |
| `VVVVDDDD` | Specific PCI vendor/device ID (e.g., `808610d3`) |
| `ecm--ncm` | Multiple drivers combined with `--` separator |

## Extensions (Output Formats)

| Extension | Purpose | Platform |
|---|---|---|
| `.kpxe` | PXE chainloading (preserves UNDI stack) | BIOS |
| `.pxe` | PXE NBP (headerless) | BIOS |
| `.rom` | NIC expansion ROM (≤64KB) | BIOS |
| `.mrom` | NIC expansion ROM (>64KB) | BIOS |
| `.iso` | Bootable CD-ROM (ISOLINUX) | BIOS |
| `.usb` | Bootable USB stick | BIOS |
| `.lkrn` | Linux kernel format (for GRUB) | BIOS |
| `.efi` | EFI executable | UEFI |
| `.efirom` | EFI NIC expansion ROM | UEFI |
| `.linux` | Linux ELF executable (testing) | Linux |

## Common Build Commands

```bash
# Chainloading (most common)
make bin/undionly.kpxe

# UEFI chainloading
make bin-x86_64-efi/snponly.efi

# Bootable ISO
make bin/ipxe.iso

# Bootable USB
make bin/ipxe.usb

# NIC ROM (Intel I217-LM)
make bin/808610d3.rom

# NIC ROM >64KB
make bin/808610d3.mrom

# All VMware ROMs
make bin/8086100f.mrom bin/808610d3.mrom bin/10222000.rom bin/15ad07b0.rom

# With embedded script
make bin/undionly.kpxe EMBED=boot.ipxe

# With custom trust
make bin/undionly.kpxe TRUST=ca.crt

# With client cert
make bin/undionly.kpxe CERT=client.crt KEY=client.key

# Debug build
make bin/undionly.kpxe DEBUG=dhcp,tcp,http

# Named configuration
make bin/undionly.kpxe CONFIG=production
```
