# iPXE Scripting

iPXE scripts automate boot sequences. Any command available at the iPXE command line can be used in a script.

## Script Basics

Scripts are plain text files starting with `#!ipxe`:

```
#!ipxe
dhcp
chain http://192.168.1.20/boot.ipxe
```

No file extension is required, but `.ipxe` is conventional.

## Labels and Goto

Define labels with `:labelname` and jump with `goto`:

```
#!ipxe
:retry
dhcp || goto retry
chain http://192.168.1.20/boot.ipxe || goto retry
```

## Conditional Operators

| Operator | Meaning |
|---|---|
| `&&` | Run next command only if previous succeeded |
| `\|\|` | Run next command only if previous failed |
| `;` | Run next command regardless |

```
#!ipxe
dhcp && echo DHCP succeeded || echo DHCP failed
```

## Variables and Settings

Access any iPXE setting with `${setting}`:

```
#!ipxe
dhcp
echo IP address: ${net0/ip}
echo MAC address: ${net0/mac}
echo Platform: ${platform}
echo Build arch: ${buildarch}
```

### Variable Substitution in URLs

```
chain http://192.168.1.20/boot.php?mac=${net0/mac}&asset=${asset:uristring}
```

The `:uristring` suffix URL-encodes the value.

### Set Custom Variables

```
set server http://192.168.1.20
set mirror ${server}/linux
kernel ${mirror}/vmlinuz
```

## Error Handling

By default, iPXE terminates a script when any command fails. Use `||` to handle errors:

```
#!ipxe
dhcp || goto no_dhcp
chain http://server/boot.ipxe || goto boot_failed

:no_dhcp
echo DHCP failed, retrying in 5 seconds...
sleep 5
goto retry

:boot_failed
echo Boot failed. Dropping to shell.
shell
```

## Menus

Build interactive boot menus:

```
#!ipxe
menu Boot Menu
item linux    Boot Linux
item winpe    Boot Windows PE
item shell    iPXE Shell
choose --default linux --timeout 10000 target && goto ${target}

:linux
kernel http://192.168.1.20/linux/vmlinuz initrd=initrd.img
initrd http://192.168.1.20/linux/initrd.img
boot

:winpe
kernel http://192.168.1.20/winpe/wimboot
initrd http://192.168.1.20/winpe/BCD       BCD
initrd http://192.168.1.20/winpe/boot.sdi  boot.sdi
initrd http://192.168.1.20/winpe/boot.wim  boot.wim
boot

:shell
shell
```

The `--timeout 10000` auto-selects the default item after 10 seconds (in milliseconds).

## Forms

Collect structured input:

```
#!ipxe
form Configure Network
item ip        IP Address
item netmask   Subnet Mask
item gateway   Gateway
present
set net0/ip ${ip}
set net0/netmask ${netmask}
set net0/gateway ${gateway}
```

## Architecture Detection

Detect 32-bit vs 64-bit CPU:

```
#!ipxe
cpuid --ext 29 && set arch x86_64 || set arch i386
echo Architecture: ${arch}
```

## Embedded Scripts

Three methods to embed scripts into iPXE firmware:

### Method 1: Compile-Time (EMBED)

```bash
make bin/undionly.kpxe EMBED=boot.ipxe
```

Requires recompilation to change. Works with all output formats.

### Method 2: Bootloader Command Line

```
# GRUB
menuentry "iPXE" {
    linux16 /boot/ipxe.lkrn dhcp && chain http://server/boot.ipxe
}
```

Uses `.lkrn` format. May need character escaping. No recompilation for changes.

### Method 3: Initrd

```
# GRUB
menuentry "iPXE" {
    linux16 /boot/ipxe.lkrn
    initrd16 /boot/boot.ipxe
}
```

Uses a separate `.ipxe` file loaded as initrd. Easiest to update.

## Restoring Ctrl-B Shell Access

Embedded scripts disable the "Press Ctrl-B" prompt. Restore it:

```
#!ipxe
prompt --key 0x02 --timeout 2000 Press Ctrl-B for the iPXE command line... && shell ||
dhcp
chain http://192.168.1.20/boot.ipxe
```

## Common Patterns

### Retry with Backoff

```
#!ipxe
set attempts:int32 0
:retry
inc attempts
dhcp && goto booted || echo Attempt ${attempts} failed
sleep 5
goto retry

:booted
chain http://192.168.1.20/boot.ipxe || goto retry
```

### Per-MAC Boot Selection

```
#!ipxe
dhcp
chain http://192.168.1.20/boot/${net0/mac:hexhyp}.ipxe || chain http://192.168.1.20/boot/default.ipxe
```

This tries a MAC-specific script first, then falls back to default.

### VLAN Boot

```
#!ipxe
vcreate --tag 100 net0
dhcp net0-100
chain http://192.168.1.20/boot.ipxe
```

## Comments

Use `#` for comments:

```
#!ipxe
# Obtain an IP address
dhcp
# Boot from the server
chain http://192.168.1.20/boot.ipxe # inline comment
```

## Legacy Compatibility

iPXE recognizes `#!gpxe` headers for backward compatibility with gPXE scripts. gPXE cannot run iPXE scripts.

## User Input

### Prompt for Keypress

```
prompt --key 0x1b --timeout 5000 Press ESC for advanced options... && goto advanced || goto default
```

### Prompt for Text Input

```
read net0/ip
echo You entered: ${net0/ip}
```

### Login Prompt

```
login
echo Username: ${username}
echo Password: (hidden)
```
