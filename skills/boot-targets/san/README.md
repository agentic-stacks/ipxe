# Boot from SAN (iSCSI, FCoE, AoE)

iPXE can boot operating systems directly from Storage Area Network targets without any local disk.

## iSCSI Boot

### Basic iSCSI Boot

```
#!ipxe
dhcp
sanboot iscsi:192.168.1.50::::iqn.2024-01.com.example:target0
```

### iSCSI with Authentication (CHAP)

```
#!ipxe
dhcp
set username my-initiator-user
set password my-initiator-secret
set initiator-iqn iqn.2024-01.com.example:initiator0
sanboot iscsi:192.168.1.50::::iqn.2024-01.com.example:target0
```

For mutual CHAP (bidirectional authentication):
```
set reverse-username my-target-user
set reverse-password my-target-secret
```

### iSCSI via DHCP Root Path

Configure the iSCSI target in DHCP instead of the boot script:

**ISC dhcpd:**
```
filename "";
option root-path "iscsi:192.168.1.50::::iqn.2024-01.com.example:target0";
```

iPXE reads `root-path` and boots from the specified iSCSI target.

### iSCSI Root Path Syntax

```
iscsi:<server>:<protocol>:<port>:<LUN>:<target-iqn>
```

| Field | Description | Example |
|---|---|---|
| `server` | iSCSI target IP or hostname | `192.168.1.50` |
| `protocol` | Protocol number (usually omit) | `` |
| `port` | TCP port (default 3260) | `3260` |
| `LUN` | Logical Unit Number | `0` |
| `target-iqn` | Target IQN | `iqn.2024-01.com.example:target0` |

### Keep SAN Connection

When booting an OS that takes over network stack (Windows), preserve the SAN connection:

```
set keep-san 1
sanboot iscsi:192.168.1.50::::iqn.2024-01.com.example:target0
```

### Skip SAN Boot

Connect to SAN for data access without booting from it:

```
set skip-san-boot 1
sanhook iscsi:192.168.1.50::::iqn.2024-01.com.example:data-target
```

## Fibre Channel over Ethernet (FCoE)

> **Build requirement:** Enable `NET_PROTO_FCOE` in `config/local/general.h`.

```
#!ipxe
sanboot fcp:<wwn>:<lun>
```

Manage FC ports with:
```
fcstat    # Display Fibre Channel ports
fcels     # Issue Fibre Channel ELS request
```

> **Build requirement:** Enable `FCMGMT_CMD` for `fcstat` and `fcels`.

## ATA over Ethernet (AoE)

```
#!ipxe
sanboot aoe:e0.0
```

AoE target format: `e<shelf>.<slot>`

## SAN Commands Reference

| Command | Purpose |
|---|---|
| `sanboot <uri>` | Boot from SAN device |
| `sanhook <uri>` | Attach SAN device (no boot) |
| `sanunhook <uri>` | Detach SAN device |

## Relevant Settings

| Setting | Purpose |
|---|---|
| `initiator-iqn` | iSCSI initiator name |
| `username` / `password` | CHAP credentials |
| `reverse-username` / `reverse-password` | Mutual CHAP credentials |
| `keep-san` | Preserve SAN connection on OS handoff |
| `skip-san-boot` | Attach without booting |
| `root-path` | SAN target (from DHCP) |
| `san-filename` | SAN boot filename |
