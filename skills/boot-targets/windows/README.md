# Network Boot Windows PE, WDS, and SCCM

iPXE boots Windows PE via HTTP using `wimboot`, which is significantly faster than TFTP-based methods like WDS. A typical 200MB Windows PE image downloads in less than two seconds on Gigabit Ethernet.

## wimboot

`wimboot` is an iPXE boot loader that extracts and boots Windows PE from a `.wim` image file. Download it from the iPXE releases or build from source.

## Boot Windows PE via HTTP

### Prepare the Boot Files

1. Install Windows ADK (Assessment and Deployment Toolkit) or Windows AIK (Automated Installation Kit)
2. Create a PE image using `copype`:
   ```cmd
   copype amd64 C:\WinPE_amd64
   copype x86 C:\WinPE_x86
   ```
3. Copy the following files to your web server:
   - `wimboot` (from iPXE)
   - `BCD` (from `WinPE_amd64/media/Boot/BCD`)
   - `boot.sdi` (from `WinPE_amd64/media/Boot/boot.sdi`)
   - `boot.wim` (from `WinPE_amd64/media/sources/boot.wim`)

### Create the Boot Script

Create `winpe.ipxe` on your web server:

```
#!ipxe
cpuid --ext 29 && set arch amd64 || set arch x86
kernel http://192.168.1.20/winpe/wimboot
initrd http://192.168.1.20/winpe/${arch}/BCD         BCD
initrd http://192.168.1.20/winpe/${arch}/boot.sdi    boot.sdi
initrd http://192.168.1.20/winpe/${arch}/boot.wim    boot.wim
boot
```

This script auto-detects 32-bit vs 64-bit CPU and loads the correct PE image.

### DHCP Configuration

Point iPXE clients to the boot script:
```
filename "http://192.168.1.20/winpe/winpe.ipxe";
```

## SCCM via HTTP

### Prepare SCCM Boot Files

1. Create `sccm.iso` from the Configuration Manager console
2. Extract `boot.wim` and modify it to include SCCM bootstrap files:
   - Mount `boot.wim` using ImageX (Windows) or `wimlib` (Linux)
   - Copy SCCM directories into the WinPE environment
   - Create `winpeshl.ini` to launch SCCM bootstrap
   - Unmount and save

3. Place on web server:
   - `wimboot`
   - `BCD`
   - `boot.sdi`
   - Modified `boot.wim`

### SCCM Boot Script

Create `sccm.ipxe`:
```
#!ipxe
kernel http://192.168.1.20/sccm/wimboot
initrd http://192.168.1.20/sccm/BCD         BCD
initrd http://192.168.1.20/sccm/boot.sdi    boot.sdi
initrd http://192.168.1.20/sccm/boot.wim    boot.wim
boot
```

## Adding Drivers to WinPE

If Windows PE does not detect the network adapter:

1. Mount `boot.wim`:
   ```bash
   wimlib-imagex mountrw boot.wim 1 /mnt/wim
   ```
2. Copy driver `.inf` and `.sys` files into the mounted image
3. Use `dism` (Windows) or `wimlib-imagex update` to inject drivers
4. Unmount and save:
   ```bash
   wimlib-imagex unmount --commit /mnt/wim
   ```

## iSCSI Target Installation

For diskless Windows deployments:

1. Boot into Windows PE via HTTP (as above)
2. From WinPE, connect to an iSCSI target and install Windows to it
3. Configure iPXE to `sanboot` the iSCSI target directly on subsequent boots:
   ```
   #!ipxe
   dhcp
   set keep-san 1
   sanboot iscsi:192.168.1.50::::iqn.2024-01.com.example:win-target
   ```

The `keep-san 1` setting preserves the SAN connection when Windows takes over.
