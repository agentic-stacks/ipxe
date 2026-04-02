# Flash iPXE onto Network Card ROM

Burning iPXE into a network card's expansion ROM replaces the factory PXE firmware permanently.

## Benefits

- iPXE appears as a boot device in BIOS/UEFI boot menu
- Persists when moving the card to another machine
- No TFTP chainloading step — boots directly into iPXE
- No licensing fees (GPL — comply by publishing source modifications via git.ipxe.org)

## Identify Your Network Card

> **Critical Rule:** Always verify PCI vendor/device IDs before building. Wrong IDs produce non-functional firmware.

```bash
lspci -nn | grep -i ethernet
```

Example output:
```
02:00.0 Ethernet controller [0200]: Intel Corporation I350 Gigabit Network Connection [8086:1521] (rev 01)
```

The IDs are in the last brackets: `[8086:1521]` → vendor `8086`, device `1521`.

## Build the ROM Image

```bash
cd ipxe/src
make bin/80861521.rom
```

The filename format is `VVVVDDDD.rom` — vendor ID followed by device ID, no separator.

For ROMs larger than 64KB (required by some cards):
```bash
make bin/80861521.mrom
```

## Flash the ROM

> **Warning:** A bad flash can brick the network card. Ensure you have the correct PCI IDs and a known-good ROM image before proceeding. Consider testing in a VM first.

### Using flashrom (Generic)

```bash
sudo flashrom -p internal -w bin/80861521.rom
```

`flashrom` supports a wide variety of cards and chips. Check `flashrom --list-supported` for your hardware.

### Intel Adapters

Use Intel's `eeupdate` or `bootutil` tools:
```bash
# List adapters
bootutil -LIST

# Flash ROM to adapter N
bootutil -NIC=1 -UP=PXE -FILE=bin/80861521.rom
```

### Broadcom tg3 Adapters

Use Broadcom's `b57udiag` utility or `ethtool`:
```bash
ethtool -E eth0 magic 0x669955aa offset 0 < bin/VVVVDDDD.rom
```

### VMware Virtual Adapters

VMware uses ROM files configured in the `.vmx` file — no flashing needed. See `skills/deploy/vmware`.

## Verify the Flash

Reboot and check the BIOS boot menu. iPXE should appear as a boot option. During boot, you should see the iPXE banner:

```
iPXE -- Open Source Network Boot Firmware -- https://ipxe.org
```

## Restore Original ROM

Keep a backup of the original ROM before flashing:

```bash
# Save original (flashrom)
sudo flashrom -p internal -r original-rom-backup.bin

# Restore if needed
sudo flashrom -p internal -w original-rom-backup.bin
```

## Older BIOS Compatibility

Some older BIOSes do not detect iPXE as a boot device. Enable the `NONPNP_HOOK_INT19` build option:

```bash
echo '#define NONPNP_HOOK_INT19' > config/local/general.h
make bin/80861521.rom
```
