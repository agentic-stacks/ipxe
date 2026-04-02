# iPXE Command Reference

## Boot Commands

| Command | Description | Example |
|---|---|---|
| `autoboot` | Boot from the default network interface | `autoboot` |
| `chain` | Download and execute an image | `chain http://server/boot.ipxe` |
| `boot` | Execute the selected image | `boot` |

## Network Interface Commands

| Command | Description | Example |
|---|---|---|
| `ifstat` | Display network interfaces | `ifstat` |
| `ifopen` | Open (enable) an interface | `ifopen net0` |
| `ifclose` | Close (disable) an interface | `ifclose net0` |
| `ifconf` / `dhcp` | Configure interface via DHCP | `dhcp net0` |
| `iflinkwait` | Wait for network link-up | `iflinkwait --timeout 5000 net0` |
| `route` | Display routing table | `route` |
| `nstat` | Display neighbour table | `nstat` |
| `ipstat` | Display IP statistics | `ipstat` |

## VLAN Commands

| Command | Description | Example |
|---|---|---|
| `vcreate` | Create a VLAN interface | `vcreate --tag 100 net0` |
| `vdestroy` | Destroy a VLAN interface | `vdestroy net0-100` |

> **Build requirement:** `VLAN_CMD`

## Image Management Commands

| Command | Description | Example |
|---|---|---|
| `imgstat` | Display loaded images | `imgstat` |
| `imgfetch` / `module` / `initrd` | Download an image | `imgfetch http://server/file` |
| `kernel` / `imgselect` / `imgload` | Download and select executable image | `kernel http://server/vmlinuz` |
| `imgfree` | Discard all loaded images | `imgfree` |
| `imgargs` | Set image command-line arguments | `imgargs vmlinuz ip=dhcp` |
| `imgtrust` | Enable image trust requirement | `imgtrust` |
| `imgverify` | Verify image signature | `imgverify vmlinuz http://server/vmlinuz.sig` |
| `imgdecrypt` | Decrypt an encrypted image | `imgdecrypt file key` |
| `imgextract` | Extract compressed/archive image | `imgextract file` |
| `shim` | Configure UEFI shim image | `shim image` |
| `fdt` | Configure Flattened Device Tree | `fdt http://server/board.dtb` |

## Checksum Commands

| Command | Description | Example |
|---|---|---|
| `md5sum` | Calculate MD5 hash | `md5sum file` |
| `sha1sum` | Calculate SHA-1 hash | `sha1sum file` |
| `sha256sum` | Calculate SHA-256 hash | `sha256sum file` |
| `sha512sum` | Calculate SHA-512 hash | `sha512sum file` |

## SAN Commands

| Command | Description | Example |
|---|---|---|
| `sanboot` | Boot from SAN device | `sanboot iscsi:server::::iqn` |
| `sanhook` | Attach SAN device (no boot) | `sanhook iscsi:server::::iqn` |
| `sanunhook` | Detach SAN device | `sanunhook iscsi:server::::iqn` |
| `fcstat` | Display Fibre Channel ports | `fcstat` |
| `fcels` | Issue FC ELS request | `fcels net0` |

## Configuration Commands

| Command | Description | Example |
|---|---|---|
| `config` | Start interactive config tool | `config net0` |
| `show` | Display a setting value | `show net0/ip` |
| `set` | Set a configuration value | `set net0/ip 192.168.1.100` |
| `clear` | Delete a setting | `clear net0/ip` |
| `read` | Prompt user for a setting | `read net0/ip` |
| `inc` | Increment a numeric setting | `inc attempts` |
| `login` | Prompt for username/password | `login` |

## Flow Control Commands

| Command | Description | Example |
|---|---|---|
| `isset` | Test if a setting exists | `isset ${filename} \|\| echo not set` |
| `iseq` | Test string equality | `iseq ${platform} efi && echo UEFI` |
| `goto` | Jump to a script label | `goto retry` |
| `exit` | Exit current shell or script | `exit 0` |

## Menu and Form Commands

| Command | Description | Example |
|---|---|---|
| `menu` | Create a menu | `menu Boot Options` |
| `item` | Add menu/form item | `item linux Boot Linux` |
| `choose` | Select from menu | `choose --default linux --timeout 10000 target` |
| `form` | Create a form | `form Network Config` |
| `present` | Display a form | `present` |

## Certificate Commands

| Command | Description | Example |
|---|---|---|
| `certstat` | Display loaded certificates | `certstat` |
| `certstore` | Store a certificate | `certstore http://server/cert.der` |
| `certfree` | Discard certificates | `certfree` |

> **Build requirement:** `CERT_CMD`

## Console Commands

| Command | Description | Example |
|---|---|---|
| `console` | Configure console | `console --picture http://server/bg.png` |
| `colour` | Define a color | `colour --basic 0 --rgb 0x000000` |
| `cpair` | Define a color pair | `cpair --foreground 7 --background 0` |

> **Build requirement:** `CONSOLE_CMD`

## Request Parameter Commands

| Command | Description | Example |
|---|---|---|
| `params` | Create parameter list | `params` |
| `param` | Add a parameter | `param mac ${net0/mac}` |

> **Build requirement:** `PARAM_CMD`

Parameters are sent as POST form data with `chain`:
```
params
param mac ${net0/mac}
param asset ${asset}
chain http://server/boot.php##params
```

## Utility Commands

| Command | Description | Example |
|---|---|---|
| `echo` | Print text | `echo Hello world` |
| `prompt` | Wait for keypress | `prompt --timeout 5000 Press a key...` |
| `shell` | Open interactive shell | `shell` |
| `help` | List available commands | `help` |
| `sleep` | Delay (seconds) | `sleep 5` |
| `reboot` | Reboot the machine | `reboot` |
| `poweroff` | Power off the machine | `poweroff` |
| `cpuid` | Check CPU feature | `cpuid --ext 29 && echo 64-bit` |
| `sync` | Wait for background ops | `sync` |
| `nslookup` | Resolve hostname | `nslookup boot.example.com` |
| `ping` | Check connectivity | `ping 192.168.1.1` |
| `ntp` | Sync time via NTP | `ntp pool.ntp.org` |
| `pciscan` | Scan PCI devices | `pciscan` |
| `usbscan` | Scan USB devices | `usbscan` |
| `time` | Measure command duration | `time dhcp` |
| `lotest` | Loopback test | `lotest net0 net1` |
| `gdbstub` | Start GDB remote debug | `gdbstub serial` |
| `profstat` | Display profiling stats | `profstat` |
