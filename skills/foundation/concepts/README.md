# iPXE Concepts

## What Is iPXE

iPXE is the leading open-source network boot firmware. It replaces the PXE (Preboot eXecution Environment) ROM built into most network cards with a far more capable boot environment.

iPXE is free and open-source under the GNU GPL. It is the official replacement for gPXE (since 2010) and Etherboot. gPXE development has ceased (no commits since 2011) — all users should use iPXE.

## What iPXE Can Do That PXE Cannot

| Capability | Standard PXE | iPXE |
|---|---|---|
| Boot from HTTP server | No | Yes |
| Boot from iSCSI SAN | No | Yes |
| Boot from FCoE SAN | No | Yes |
| Boot from AoE SAN | No | Yes |
| Boot over wireless | No | Yes |
| Boot over wide-area network | No | Yes |
| Scripted boot logic | No | Yes |
| HTTPS with certificate verification | No | Yes |
| DNS resolution | No | Yes |
| Menu-driven boot selection | No | Yes |
| VLAN support | No | Yes |

## How Network Booting Works

```
Power On
  │
  ├─ NIC firmware runs (PXE or iPXE)
  │
  ├─ DHCP request → obtains IP address + boot filename
  │
  ├─ Downloads boot file (TFTP for PXE, HTTP/TFTP/iSCSI for iPXE)
  │
  └─ Executes boot file (kernel, script, or chain to next stage)
```

### The PXE Boot Flow

1. Machine powers on, NIC firmware initializes
2. NIC sends DHCP DISCOVER with PXE vendor extensions
3. DHCP server responds with IP address, `next-server` (TFTP IP), and `filename` (boot file)
4. NIC downloads the boot file via TFTP
5. NIC executes the downloaded file as a Network Bootstrap Program (NBP)

### The iPXE Boot Flow

iPXE extends this by supporting:
- HTTP/HTTPS URLs in the `filename` field (not just TFTP)
- Scripted boot logic — conditional branching, retry loops, menus
- SAN boot paths via `root-path` (iSCSI, FCoE, AoE)
- Multiple network interfaces and VLANs

## Deployment Models

### Chainloading (Most Common)

The existing PXE ROM loads iPXE from TFTP, then iPXE takes over and boots via HTTP/iSCSI/script. No hardware modification required. See `skills/deploy/chainloading`.

### ROM Replacement

iPXE is flashed directly onto the network card's expansion ROM. iPXE appears as a boot device in the BIOS. Persists when moving the card to another machine. See `skills/deploy/rom-burning`.

### Bootable Media

iPXE is burned to a CD-ROM (ISO) or USB stick. Useful for one-off boots or machines with no network card ROM. See `skills/deploy/media`.

### Virtual Machines

iPXE replaces the virtual NIC ROM in VMware, QEMU/KVM, or VirtualBox. See `skills/deploy/vmware`.

### Cloud Instances

iPXE AMIs/images boot EC2 or GCE instances with full iPXE scripting via user-data or instance metadata. See `skills/deploy/cloud`.

## Architecture: BIOS vs UEFI

iPXE supports both BIOS and UEFI firmware:

| Aspect | BIOS | UEFI |
|---|---|---|
| Binary format | `.kpxe`, `.pxe`, `.rom` | `.efi`, `.efirom` |
| Chainload file | `undionly.kpxe` | `ipxe.efi` |
| Boot speed | Fast | Substantially slower |
| Secure Boot | Not applicable | Supported (requires signing) |
| Driver model | UNDI (legacy NIC driver) | SNP (Simple Network Protocol) |

UEFI network booting is substantially slower than BIOS network booting. Prefer BIOS mode when possible for boot performance.

## Rolling Release Model

iPXE uses a rolling release — there are no versioned releases. Every commit is intended to be production-ready. Always use the latest code from the repository at https://github.com/ipxe/ipxe.

## Key Terminology

| Term | Meaning |
|---|---|
| **PXE** | Preboot eXecution Environment — Intel's network boot standard |
| **NBP** | Network Bootstrap Program — the file PXE downloads and executes |
| **UNDI** | Universal Network Device Interface — legacy NIC driver API used by BIOS |
| **SNP** | Simple Network Protocol — UEFI NIC driver API |
| **Chainloading** | Loading iPXE from an existing PXE ROM via TFTP |
| **SAN** | Storage Area Network — iSCSI, FCoE, AoE |
| **wimboot** | iPXE tool for booting Windows PE images via HTTP |
| **crosscert** | Cross-signed certificate for bridging public CAs to iPXE's root CA |
