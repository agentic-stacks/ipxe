# Console Configuration

iPXE supports multiple console types for input/output. Consoles are enabled at build time and configured at runtime.

## Console Types

| Console | Build Option | Runtime Config | Use Case |
|---|---|---|---|
| BIOS (default) | `CONSOLE_PCBIOS` | — | Local keyboard/monitor |
| Serial | `CONSOLE_SERIAL` | `config/serial.h` | Headless servers, data centers |
| Syslog | `CONSOLE_SYSLOG` | `set syslog <ip>` | Remote log collection |
| Encrypted syslog | `CONSOLE_SYSLOGS` | `set syslogs <ip>` | Secure remote logging |
| Framebuffer | `CONSOLE_FRAMEBUFFER` | `console` command | High-res graphics, backgrounds |
| VMware | `CONSOLE_VMWARE` | — | VMware `vmware.log` output |

## Serial Console

### Enable at Build Time

```c
/* config/local/console.h */
#define CONSOLE_SERIAL
```

### Configure Serial Port

Default: COM1, 115200 baud, 8N1. Override in `config/local/serial.h`:

```c
/* config/local/serial.h */
#define COMCONSOLE 0x3f8    /* COM1=0x3f8, COM2=0x2f8 */
#define COMSPEED   115200   /* Baud rate */
#define COMDATA    8        /* Data bits */
#define COMPARITY  0        /* 0=none, 1=odd, 2=even */
#define COMSTOP    1        /* Stop bits */
```

### Connect

```bash
# Linux
screen /dev/ttyUSB0 115200

# or
minicom -D /dev/ttyUSB0 -b 115200
```

## Syslog Console

Send iPXE output to a remote syslog server.

### Enable at Build Time

```c
/* config/local/console.h */
#define CONSOLE_SYSLOG
```

### Configure at Runtime

```
#!ipxe
set syslog 192.168.1.30
dhcp
chain http://192.168.1.20/boot.ipxe
```

### Receive Syslog

On the syslog server:
```bash
# Listen with netcat (quick test)
nc -u -l 514

# Or configure rsyslog/syslog-ng to receive UDP 514
```

## Encrypted Syslog

```c
/* config/local/console.h */
#define CONSOLE_SYSLOGS
```

```
#!ipxe
set syslogs 192.168.1.30
```

Requires a TLS-capable syslog server.

## Graphical Framebuffer

```c
/* config/local/console.h */
#define CONSOLE_FRAMEBUFFER

/* config/local/general.h */
#define IMAGE_PNG
```

Activate in a script:
```
#!ipxe
console --picture http://192.168.1.20/background.png
```

Supports higher resolutions and arbitrary colors compared to the BIOS console.

## VMware Console

```c
/* config/local/console.h */
#define CONSOLE_VMWARE
```

All iPXE output appears in the VM's `vmware.log` file. Only functions inside VMware VMs.

## Keyboard Layout

Change the keyboard layout:

```c
/* config/local/console.h */
#define KEYBOARD_MAP us
```

Available maps are in the iPXE source under `hci/keymap/`.

## Log Level

Control which messages appear:

```c
/* config/local/console.h */
#define LOG_LEVEL LOG_ALL
```

Log levels: `LOG_NONE`, `LOG_ALL`. Enable `LOG_ALL` for maximum diagnostic output.

## Console Usages

Each console handles specific output types:

| Usage | Description |
|---|---|
| STDOUT | Normal command output |
| DEBUG | Debug messages (requires debug build) |
| TUI | Text user interfaces (menus, forms, config tool) |
| LOG | Log messages (requires `LOG_LEVEL LOG_ALL`) |

Multiple consoles can be active simultaneously. For example, BIOS console for local interaction + syslog for remote logging.
