# DHCP Configuration for iPXE

DHCP is the control plane for iPXE boot. The DHCP server tells each client what to boot and from where.

## Choose Your DHCP Server

| Server | Platform | Config File | Guide |
|---|---|---|---|
| ISC dhcpd | Linux | `/etc/dhcpd.conf` | `isc-dhcpd.md` |
| Microsoft DHCP | Windows Server | MMC snap-in | `ms-dhcp.md` |

## Common Patterns

### Boot Filename Options

| Protocol | DHCP `filename` Example |
|---|---|
| TFTP | `pxelinux.0` or `undionly.kpxe` |
| HTTP | `http://192.168.1.20/boot.ipxe` |
| iSCSI | `""` (use `root-path` instead) |

### iPXE-Specific DHCP Options

Both DHCP servers can send iPXE-specific options. Define the iPXE option space:

| Option | Code | Type | Purpose |
|---|---|---|---|
| `ipxe.priority` | 1 | signed int8 | Settings priority |
| `ipxe.keep-san` | 8 | unsigned int8 | Preserve SAN connection |
| `ipxe.skip-san-boot` | 9 | unsigned int8 | Attach SAN without boot |
| `ipxe.no-pxedhcp` | 176 | unsigned int8 | Skip ProxyDHCP wait |

### Feature Detection

iPXE advertises its capabilities via DHCP options. Test for features before assigning boot paths:

| Feature | Option Code | Meaning |
|---|---|---|
| `ipxe.http` | 19 | HTTP download support |
| `ipxe.https` | 20 | HTTPS download support |
| `ipxe.iscsi` | 17 | iSCSI support |
| `ipxe.aoe` | 18 | AoE support |
| `ipxe.ftp` | 21 | FTP support |
| `ipxe.tftp` | 22 | TFTP support |
| `ipxe.dns` | 23 | DNS support |
| `ipxe.efi` | 36 | EFI firmware |
| `ipxe.fcoe` | 37 | FCoE support |

### Chainloading Loop Prevention

> **Critical:** iPXE re-requesting its own chainload binary creates an infinite boot loop. Always implement one of these methods.

See `skills/deploy/chainloading` for the three methods: DHCP user-class detection, embedded scripts, or autoexec.ipxe.
