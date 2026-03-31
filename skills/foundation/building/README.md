# Build iPXE from Source

## Prerequisites

Install build dependencies:

**Debian/Ubuntu:**
```bash
sudo apt-get install -y git gcc binutils make perl liblzma-dev mtools mkisofs
```

**Fedora/RHEL:**
```bash
sudo dnf install -y git gcc binutils make perl xz-devel mtools genisoimage
```

**macOS (cross-compilation only):**
```bash
brew install x86_64-elf-gcc mtools xorriso
```

Optional: `syslinux` (for ISO creation), `mkisofs`/`genisoimage`/`xorrisofs` (for CD images).

## Clone and Build

```bash
git clone https://github.com/ipxe/ipxe.git
cd ipxe/src
make
```

This builds the default set of binaries in `bin/`.

## Common Build Targets

Build specific targets with `make bin/<target>`:

| Target | Use Case |
|---|---|
| `make bin/ipxe.iso` | Bootable CD-ROM image |
| `make bin/ipxe.usb` | Bootable USB stick image (write with `dd`) |
| `make bin/undionly.kpxe` | BIOS chainloading via TFTP |
| `make bin/ipxe.efi` | UEFI boot binary |
| `make bin/ipxe.lkrn` | Linux kernel format (for GRUB/bootloader) |
| `make bin/808610d3.rom` | NIC-specific ROM (vendor ID 8086, device ID 10d3) |
| `make bin/808610d3.mrom` | NIC-specific ROM for cards needing >64KB |

### UEFI Targets

UEFI builds use different platform directories:

```bash
# 64-bit UEFI
make bin-x86_64-efi/ipxe.efi
make bin-x86_64-efi/snponly.efi

# 32-bit UEFI
make bin-i386-efi/ipxe.efi

# ARM64 UEFI (cross-compile)
make CROSS=aarch64-linux-gnu- bin-arm64-efi/ipxe.efi
```

### Driver Selection

- **`ipxe`** — includes almost all PCI NIC drivers (large binary)
- **`undionly`** — uses the existing BIOS UNDI driver (small, for chainloading)
- **`snponly`** / **`snp`** — uses UEFI SNP driver
- **`808610d3`** — specific PCI vendor/device ID (Intel I217-LM in this example)

## Embedded Scripts

Build with a custom startup script:

```bash
make bin/undionly.kpxe EMBED=my-boot-script.ipxe
```

The script runs automatically when iPXE starts. See `skills/infrastructure/scripting` for script syntax.

## Custom Build Configuration

Override build-time options by creating local header files:

```bash
# Enable HTTPS and additional commands
cat > config/local/general.h << 'EOF'
#define DOWNLOAD_PROTO_HTTPS
#define PING_CMD
#define NTP_CMD
#define CONSOLE_CMD
#define NSLOOKUP_CMD
#define VLAN_CMD
#define REBOOT_CMD
#define POWEROFF_CMD
EOF

make bin/undionly.kpxe
```

See `skills/foundation/configuration` for all build options.

## Debug Builds

Enable debug output for specific modules:

```bash
make bin/undionly.kpxe DEBUG=dhcp,tcp,http
```

Multiple modules can be comma-separated. Debug output goes to the active console.

## Write USB Image

```bash
make bin/ipxe.usb
sudo dd if=bin/ipxe.usb of=/dev/sdX bs=4M status=progress
sync
```

> **Warning:** Double-check the target device (`/dev/sdX`). `dd` will overwrite the entire device without confirmation.

## Write CD-ROM Image

```bash
make bin/ipxe.iso
# Burn to disc or mount in a VM
```

## Build Output

All built binaries appear under `bin/` (BIOS), `bin-x86_64-efi/` (64-bit UEFI), `bin-i386-efi/` (32-bit UEFI), or `bin-arm64-efi/` (ARM64). The build system derives the architecture from the platform directory name.
