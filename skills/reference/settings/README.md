# iPXE Settings Reference

Settings are accessed via `show`, `set`, `clear`, and `read` commands. Network device settings use the `net0/` prefix.

## Network Device Settings

| Setting | Type | Description | Example |
|---|---|---|---|
| `net0/mac` | hex | MAC address (read-only) | `show net0/mac` |
| `net0/bustype` | string | Bus type (read-only) | `show net0/bustype` |
| `net0/busloc` | uint32 | Bus location (read-only) | `show net0/busloc` |
| `net0/busid` | hex | Bus ID (read-only) | `show net0/busid` |
| `net0/chip` | string | Chip type (read-only) | `show net0/chip` |

## Wireless Settings

| Setting | Type | Description | Example |
|---|---|---|---|
| `ssid` | string | Wireless network name | `set ssid MyNetwork` |
| `active-scan` | int8 | Actively scan for networks | `set active-scan 1` |
| `key` | string | Wireless encryption key | `set key mypassword` |

## IPv4 Settings

| Setting | Type | Description | Example |
|---|---|---|---|
| `net0/ip` | ipv4 | IP address | `set net0/ip 192.168.1.100` |
| `net0/netmask` | ipv4 | Subnet mask | `set net0/netmask 255.255.255.0` |
| `net0/gateway` | ipv4 | Default gateway | `set net0/gateway 192.168.1.1` |
| `net0/dns` | ipv4 | DNS server | `set net0/dns 8.8.8.8` |
| `net0/domain` | string | DNS domain | `show net0/domain` |

## Boot Settings

| Setting | Type | Description | Example |
|---|---|---|---|
| `filename` | string | Boot filename (from DHCP) | `show filename` |
| `next-server` | ipv4 | TFTP server (from DHCP) | `show next-server` |
| `root-path` | string | SAN root path | `set root-path iscsi:...` |
| `san-filename` | string | SAN filename | `show san-filename` |
| `initiator-iqn` | string | iSCSI initiator name | `set initiator-iqn iqn.2024-01...` |
| `keep-san` | int8 | Preserve SAN on OS handoff | `set keep-san 1` |
| `skip-san-boot` | int8 | Attach SAN without booting | `set skip-san-boot 1` |
| `scriptlet` | string | Boot scriptlet | `show scriptlet` |
| `use-cached` | uint8 | Use cached DHCP settings | `set use-cached 1` |

## Host Identity Settings

| Setting | Type | Description | Example |
|---|---|---|---|
| `hostname` | string | Host name | `set hostname myhost` |
| `uuid` | uuid | System UUID (read-only) | `show uuid` |
| `user-class` | string | DHCP user class | `show user-class` |
| `manufacturer` | string | Manufacturer (read-only) | `show manufacturer` |
| `product` | string | Product name (read-only) | `show product` |
| `serial` | string | Serial number (read-only) | `show serial` |
| `asset` | string | Asset tag (read-only) | `show asset` |

## Authentication Settings

| Setting | Type | Description | Example |
|---|---|---|---|
| `username` | string | CHAP/HTTP username | `set username myuser` |
| `password` | string | CHAP/HTTP password | `set password mysecret` |
| `reverse-username` | string | Mutual CHAP username | `set reverse-username target` |
| `reverse-password` | string | Mutual CHAP password | `set reverse-password secret` |

## Cryptography Settings

| Setting | Type | Description | Example |
|---|---|---|---|
| `crosscert` | string | Cross-certificate source URI | `set crosscert http://ca.ipxe.org/auto` |
| `trust` | hex | Trusted root cert fingerprints | (set at build time) |
| `cert` | hex | Client certificate | (set at build time) |
| `privkey` | hex | Client private key | (set at build time) |

## System Information Settings (Read-Only)

| Setting | Type | Description | Example |
|---|---|---|---|
| `buildarch` | string | Build architecture | `show buildarch` |
| `cpumodel` | string | CPU model | `show cpumodel` |
| `cpuvendor` | string | CPU vendor | `show cpuvendor` |
| `platform` | string | Firmware platform (`pcbios` or `efi`) | `show platform` |
| `version` | string | iPXE version | `show version` |
| `memsize` | int32 | Memory size (KB) | `show memsize` |
| `unixtime` | uint32 | Seconds since epoch | `show unixtime` |

## Networking Settings

| Setting | Type | Description | Example |
|---|---|---|---|
| `dhcp-server` | ipv4 | DHCP server address (read-only) | `show dhcp-server` |
| `syslog` | ipv4 | Syslog server for logging | `set syslog 192.168.1.30` |
| `syslogs` | string | Encrypted syslog server | `set syslogs 192.168.1.30` |
| `sysmac` | hex | System MAC address | `show sysmac` |

## Miscellaneous Settings

| Setting | Type | Description | Example |
|---|---|---|---|
| `keymap` | string | Keyboard layout | `set keymap us` |
| `cwduri` | string | Current working directory URI | `show cwduri` |
| `priority` | int8 | Settings priority | `set priority 1` |
| `vram` | base64 | Video RAM contents | (used by framebuffer) |
