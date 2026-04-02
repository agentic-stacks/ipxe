# iPXE on Google Cloud

## Available Images

iPXE publishes images in the `ipxe-images` project:

| Image Family | Architecture | Firmware |
|---|---|---|
| `ipxe` | x86_64 | BIOS |
| `ipxe-uefi-x86-64` | x86_64 | UEFI |
| `ipxe-uefi-arm64` | ARM64 | UEFI |

## Launch an Instance

The boot script is passed as instance metadata under the `ipxeboot` key. The embedded firmware runs DHCP, prints diagnostics, then downloads and executes the metadata script.

### Example: Boot from HTTP

Create the boot script `boot.ipxe`:
```
#!ipxe
kernel http://192.168.1.20/vmlinuz initrd=initrd.img
initrd http://192.168.1.20/initrd.img
boot
```

Launch the instance:
```bash
gcloud compute instances create my-ipxe-instance \
  --image-project ipxe-images \
  --image-family ipxe \
  --machine-type e2-micro \
  --metadata-from-file=ipxeboot=boot.ipxe
```

### Google Virtual NIC (gVNIC)

iPXE images support gVNIC-enabled instance types. No special configuration needed.

## Troubleshooting

View serial port output:
```bash
gcloud compute instances get-serial-port-output my-ipxe-instance
```

## Build Custom Images

Compile iPXE with custom configuration and import using the `gce-import` tool from the iPXE source repository's `contrib/` directory:

```bash
cd ipxe/src
make bin-x86_64-efi/ipxe.efi
# Use contrib/gce-import to create a GCE image
```
