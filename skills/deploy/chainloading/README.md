# Deploy iPXE via Chainloading

Chainloading is the most common way to deploy iPXE. The existing PXE ROM on each machine loads iPXE from a TFTP server, then iPXE takes over for the actual boot.

## Build the Chainload Binary

```bash
cd ipxe/src

# BIOS systems
make bin/undionly.kpxe

# UEFI systems
make bin-x86_64-efi/ipxe.efi
```

Copy the binary to your TFTP server (e.g., `/var/lib/tftpboot/`).

## Configure DHCP

Point DHCP clients to the iPXE binary:

```
next-server 192.168.1.10;    # Your TFTP server IP
filename "undionly.kpxe";     # For BIOS clients
```

See `skills/infrastructure/dhcp` for full ISC dhcpd and Microsoft DHCP configuration.

## Solve the Infinite Boot Loop

> **Critical:** Without loop prevention, iPXE will DHCP again, receive `undionly.kpxe` again, and reboot in an infinite loop.

Three methods to break the loop:

### Method 1: DHCP User-Class Detection (Recommended)

Configure DHCP to serve different files based on whether the client is iPXE or legacy PXE:

**ISC dhcpd:**
```
if exists user-class and option user-class = "iPXE" {
    filename "http://192.168.1.20/boot.ipxe";
} elsif option client-architecture = 00:00 {
    filename "undionly.kpxe";
} else {
    filename "ipxe.efi";
}
```

iPXE sends `user-class = "iPXE"` in its DHCP requests. Legacy PXE does not. This lets DHCP route iPXE clients to the real boot script via HTTP.

### Method 2: Embedded Script

Build iPXE with a startup script that bypasses the DHCP filename:

```bash
cat > chainload.ipxe << 'EOF'
#!ipxe
dhcp
chain http://192.168.1.20/boot.ipxe
EOF

make bin/undionly.kpxe EMBED=chainload.ipxe
```

The embedded script runs immediately and chains to the real boot script. No DHCP configuration changes needed.

### Method 3: autoexec.ipxe

Place a file named `autoexec.ipxe` on the TFTP server. iPXE automatically discovers and executes it after DHCP.

> **Note:** Not all platforms support `autoexec.ipxe`. Test before relying on this method.

## BIOS vs UEFI Chainloading

Configure DHCP to serve the correct binary based on client architecture:

```
option client-architecture code 93 = unsigned integer 16;

if exists user-class and option user-class = "iPXE" {
    filename "http://192.168.1.20/boot.ipxe";
} elsif option client-architecture = 00:00 {
    # BIOS
    filename "undionly.kpxe";
} elsif option client-architecture = 00:07 {
    # UEFI x86-64
    filename "ipxe.efi";
} elsif option client-architecture = 00:09 {
    # UEFI x86-64 (alternate)
    filename "ipxe.efi";
} else {
    filename "ipxe.efi";
}
```

## Verify Chainloading

1. Boot a test machine (VM recommended — see Critical Rule 6)
2. Watch for: PXE ROM → DHCP → TFTP download of `undionly.kpxe` → iPXE banner
3. iPXE should then DHCP again and receive the HTTP boot script URL
4. The boot script executes (no infinite loop)

## Performance

Eliminate ProxyDHCP delays if you are not running a ProxyDHCP server:

```
option ipxe.no-pxedhcp 1;
```

Add this to your DHCP configuration. See `skills/infrastructure/dhcp` for the full iPXE option space definition.
