# ISC dhcpd Configuration

## Define iPXE Option Space

Add to `/etc/dhcpd.conf`:

```
# iPXE options
option space ipxe;
option ipxe-encap-opts code 175 = encapsulate ipxe;
option ipxe.priority code 1 = signed integer 8;
option ipxe.keep-san code 8 = unsigned integer 8;
option ipxe.skip-san-boot code 9 = unsigned integer 8;
option ipxe.no-pxedhcp code 176 = unsigned integer 8;

# iPXE feature indicators
option ipxe.http code 19 = unsigned integer 8;
option ipxe.https code 20 = unsigned integer 8;
option ipxe.iscsi code 17 = unsigned integer 8;
option ipxe.aoe code 18 = unsigned integer 8;
option ipxe.ftp code 21 = unsigned integer 8;
option ipxe.tftp code 22 = unsigned integer 8;
option ipxe.dns code 23 = unsigned integer 8;
option ipxe.efi code 36 = unsigned integer 8;
option ipxe.fcoe code 37 = unsigned integer 8;
```

## Chainloading with Architecture Detection

```
option client-architecture code 93 = unsigned integer 16;

if exists user-class and option user-class = "iPXE" {
    # iPXE client — serve the real boot script via HTTP
    filename "http://192.168.1.20/boot.ipxe";
} elsif option client-architecture = 00:00 {
    # BIOS client — chainload iPXE
    filename "undionly.kpxe";
    next-server 192.168.1.10;
} elsif option client-architecture = 00:07 {
    # UEFI x86-64 client
    filename "ipxe.efi";
    next-server 192.168.1.10;
} elsif option client-architecture = 00:09 {
    # UEFI x86-64 (alternate architecture ID)
    filename "ipxe.efi";
    next-server 192.168.1.10;
} else {
    filename "ipxe.efi";
    next-server 192.168.1.10;
}
```

## Eliminate ProxyDHCP Delay

If you are not running a ProxyDHCP server, eliminate the wait:

```
option ipxe.no-pxedhcp 1;
```

> **Warning:** Only add this if you are not running a ProxyDHCP server. If you are, iPXE needs the ProxyDHCP response.

## Feature-Based Routing

Serve different boot configurations based on iPXE capabilities:

```
if exists ipxe.https {
    filename "https://192.168.1.20/boot.ipxe";
} elsif exists ipxe.http {
    filename "http://192.168.1.20/boot.ipxe";
} else {
    filename "boot.ipxe";
    next-server 192.168.1.10;
}
```

## iSCSI Boot via DHCP

```
filename "";
option root-path "iscsi:192.168.1.50::::iqn.2024-01.com.example:target0";
```

## FCoE Boot via DHCP

```
if exists ipxe.fcoe {
    option root-path "fcp:20:00:52:54:00:aa:b7:01:0";
}
```

## Complete Example

```
# /etc/dhcpd.conf

option space ipxe;
option ipxe-encap-opts code 175 = encapsulate ipxe;
option ipxe.priority code 1 = signed integer 8;
option ipxe.keep-san code 8 = unsigned integer 8;
option ipxe.skip-san-boot code 9 = unsigned integer 8;
option ipxe.no-pxedhcp code 176 = unsigned integer 8;

option client-architecture code 93 = unsigned integer 16;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option domain-name-servers 192.168.1.1;

    if exists user-class and option user-class = "iPXE" {
        filename "http://192.168.1.20/boot.ipxe";
        option ipxe.no-pxedhcp 1;
    } elsif option client-architecture = 00:00 {
        filename "undionly.kpxe";
        next-server 192.168.1.10;
    } else {
        filename "ipxe.efi";
        next-server 192.168.1.10;
    }
}
```
