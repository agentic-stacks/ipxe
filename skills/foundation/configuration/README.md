# Build-Time Configuration

iPXE features are controlled by C header files in `ipxe/src/config/`. Override defaults by creating files in `config/local/` — never edit the originals (they are tracked by git).

## How to Override

Create a matching file in `config/local/`:

```bash
cd ipxe/src
mkdir -p config/local
cat > config/local/general.h << 'EOF'
/* Enable HTTPS */
#define DOWNLOAD_PROTO_HTTPS

/* Enable useful commands */
#define PING_CMD
#define NTP_CMD
#define NSLOOKUP_CMD
#define CONSOLE_CMD
#define REBOOT_CMD
#define POWEROFF_CMD
#define VLAN_CMD
EOF
```

Then rebuild: `make bin/undionly.kpxe`

Use `#define` to enable a feature and `#undef` to disable one.

## config/general.h — Protocols and Commands

### Download Protocols

| Option | Description | Default |
|---|---|---|
| `DOWNLOAD_PROTO_HTTPS` | HTTPS support | Disabled |
| `NET_PROTO_IPV6` | IPv6 protocol | Disabled |
| `NET_PROTO_FCOE` | Fibre Channel over Ethernet | Disabled |

### Commands

| Option | Description | Default |
|---|---|---|
| `PING_CMD` | `ping` command | Disabled |
| `NTP_CMD` | `ntp` command | Disabled |
| `NSLOOKUP_CMD` | `nslookup` command | Disabled |
| `CONSOLE_CMD` | `console` command | Disabled |
| `REBOOT_CMD` | `reboot` command | Disabled |
| `POWEROFF_CMD` | `poweroff` command | Disabled |
| `VLAN_CMD` | `vcreate`/`vdestroy` commands | Disabled |
| `TIME_CMD` | `time` command | Disabled |
| `LOTEST_CMD` | `lotest` loopback testing | Disabled |
| `PCI_CMD` | `pciscan` command | Disabled |
| `USB_CMD` | `usbscan` command | Disabled |
| `PARAM_CMD` | `params`/`param` commands | Disabled |
| `CERT_CMD` | `certstat`/`certstore`/`certfree` commands | Disabled |
| `IMAGE_TRUST_CMD` | `imgtrust`/`imgverify` commands | Disabled |
| `IMAGE_CRYPT_CMD` | `imgdecrypt` command | Disabled |
| `IMAGE_ARCHIVE_CMD` | `imgextract` command | Disabled |
| `IPSTAT_CMD` | `ipstat` command | Disabled |
| `NEIGHBOUR_CMD` | `nstat` command | Disabled |
| `FCMGMT_CMD` | `fcstat`/`fcels` commands | Disabled |
| `PROFSTAT_CMD` | `profstat` command | Disabled |

### Image Formats

| Option | Description | Default |
|---|---|---|
| `IMAGE_GZIP` | GZIP compressed images | Disabled |
| `IMAGE_ZLIB` | ZLIB compressed images | Disabled |
| `IMAGE_PNG` | PNG image support (for framebuffer) | Disabled |
| `IMAGE_PNM` | PNM image support | Disabled |
| `IMAGE_UCODE` | CPU microcode update images | Disabled |

## config/console.h — Console Settings

| Option | Description | Default |
|---|---|---|
| `CONSOLE_PCBIOS` | BIOS keyboard/monitor console | Enabled |
| `CONSOLE_SERIAL` | Serial port console | Disabled |
| `CONSOLE_SYSLOG` | Remote syslog console | Disabled |
| `CONSOLE_SYSLOGS` | Encrypted syslog (TLS) | Disabled |
| `CONSOLE_VMWARE` | VMware logfile console | Disabled |
| `CONSOLE_FRAMEBUFFER` | Graphical framebuffer | Disabled |
| `KEYBOARD_MAP` | Keyboard layout (default: `us`) | `us` |
| `LOG_LEVEL` | Log message verbosity | `LOG_NONE` |

## config/serial.h — Serial Port Settings

| Option | Description | Default |
|---|---|---|
| `COMCONSOLE` | I/O port address | `0x3f8` (COM1) |
| `COMSPEED` | Baud rate | `115200` |
| `COMDATA` | Data bits | `8` |
| `COMPARITY` | Parity | `0` (none) |
| `COMSTOP` | Stop bits | `1` |

## config/crypto.h — Cryptography Settings

| Option | Description | Default |
|---|---|---|
| `TIMESTAMP_ERROR_MARGIN` | Allowed clock skew for signed timestamps | Platform-dependent |

## config/settings.h — Settings Sources

| Option | Description | Default |
|---|---|---|
| `CPUID_SETTINGS` | Expose CPU ID as settings | Disabled |
| `MEMMAP_SETTINGS` | Expose memory map as settings | Disabled |
| `VMWARE_SETTINGS` | Expose VMware GuestInfo as settings | Disabled |
| `VRAM_SETTINGS` | Expose video RAM contents as settings | Disabled |

## Recommended Production Configuration

```c
/* config/local/general.h — production baseline */
#define DOWNLOAD_PROTO_HTTPS
#define IMAGE_TRUST_CMD
#define CERT_CMD
#define PING_CMD
#define NTP_CMD
#define NSLOOKUP_CMD
#define CONSOLE_CMD
#define REBOOT_CMD
#define POWEROFF_CMD
#define VLAN_CMD
#define PARAM_CMD
```

## Named Build Configurations

Maintain multiple configurations by creating named subdirectories:

```bash
mkdir -p config/local/production
mkdir -p config/local/debug

# Build with a named config
make bin/undionly.kpxe CONFIG=production
```
