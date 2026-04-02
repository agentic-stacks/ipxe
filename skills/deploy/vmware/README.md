# Deploy iPXE in VMware

Replace VMware's default PXE ROM with iPXE for HTTP boot, iSCSI, scripting, and other iPXE capabilities.

## Supported Virtual Network Adapters

| VMware Adapter | iPXE Driver | ROM File |
|---|---|---|
| e1000 | intel | `bin/8086100f.mrom` |
| e1000e | intel | `bin/808610d3.mrom` |
| vlance | pcnet32 | `bin/10222000.rom` |
| vmxnet3 | vmxnet3 | `bin/15ad07b0.rom` |

## Build ROM Images

```bash
cd ipxe/src
make bin/8086100f.mrom bin/808610d3.mrom bin/10222000.rom bin/15ad07b0.rom
```

Copy the ROM files to a shared location:
```bash
sudo mkdir -p /usr/lib/vmware/resources
sudo cp bin/8086100f.mrom bin/808610d3.mrom bin/10222000.rom bin/15ad07b0.rom /usr/lib/vmware/resources/
```

## Configure the VM

Edit the `.vmx` file to use iPXE ROMs:

```
ethernet0.virtualDev = "e1000"
ethernet0.opromsize = 262144
e1000bios.filename = "/usr/lib/vmware/resources/8086100f.mrom"
e1000ebios.filename = "/usr/lib/vmware/resources/808610d3.mrom"
nbios.filename = "/usr/lib/vmware/resources/10222000.rom"
nx3bios.filename = "/usr/lib/vmware/resources/15ad07b0.rom"
```

The `opromsize = 262144` (256KB) ensures enough space for the iPXE ROM.

## VMware-Specific Build Options

### VMware Console Logging

Enable iPXE output in `vmware.log`:

```c
/* config/local/console.h */
#define CONSOLE_VMWARE
```

```bash
make bin/8086100f.mrom
```

iPXE messages appear in the VM's `vmware.log` file — useful for debugging boot scripts.

### GuestInfo Settings

Configure iPXE network settings directly via the `.vmx` file:

```c
/* config/local/settings.h */
#define VMWARE_SETTINGS
```

Then in `.vmx`:
```
guestinfo.ipxe.net0.ip = "192.168.1.100"
guestinfo.ipxe.net0.netmask = "255.255.255.0"
guestinfo.ipxe.net0.gateway = "192.168.1.1"
guestinfo.ipxe.dns = "192.168.1.1"
guestinfo.ipxe.filename = "http://192.168.1.20/boot.ipxe"
```

## Embedded Scripts for VMware

Build with an embedded script to avoid DHCP configuration:

```bash
cat > vmware-boot.ipxe << 'EOF'
#!ipxe
dhcp
chain http://192.168.1.20/boot.ipxe
EOF

make bin/8086100f.mrom EMBED=vmware-boot.ipxe
```

## Verify

1. Power on the VM
2. Watch for the iPXE banner in the VM console
3. If using `CONSOLE_VMWARE`, check `vmware.log` for detailed output
