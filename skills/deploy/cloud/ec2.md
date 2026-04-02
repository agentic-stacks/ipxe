# iPXE on AWS EC2

## Available AMIs

iPXE publishes public AMIs from account `833372943033` across all AWS regions. Both `arm64` and `x86_64` architectures are available.

## Launch an Instance

The boot script is passed as EC2 user-data. The embedded iPXE firmware automatically runs DHCP and then fetches `http://169.254.169.254/latest/user-data` as an iPXE script.

### Example: Boot Tiny Core Linux

Create the boot script `boot.ipxe`:
```
#!ipxe
set base http://tinycorelinux.net/12.x/x86/release/distribution_files/
kernel ${base}/vmlinuz64 initrd=rootfs.gz initrd=modules64.gz
initrd ${base}/rootfs.gz
initrd ${base}/modules64.gz
boot
```

Launch the instance:
```bash
aws ec2 run-instances \
  --image-id ami-XXXXXXXXXXXXXXXXX \
  --instance-type t3.micro \
  --user-data file://boot.ipxe \
  --key-name my-keypair
```

Find the current AMI ID:
```bash
aws ec2 describe-images --owners 833372943033 \
  --filters "Name=name,Values=ipxe*" \
  --query 'Images | sort_by(@, &CreationDate) | [-1].ImageId' \
  --output text
```

### Example: iSCSI Boot

Boot a diskless instance from an iSCSI target in the same VPC:

```
#!ipxe
dhcp
sanboot iscsi:192.168.1.50::::iqn.2024-01.com.example:target0
```

## Supported Boot Methods

- HTTP kernel + initrd download
- iSCSI SAN boot (target must be in the same VPC)
- IPv4 and IPv6 networking

## Troubleshooting

View boot console output:
```bash
aws ec2 get-console-output --instance-id i-0123456789abcdef0 --output text
```

> **Note:** EC2 console output may be delayed by several minutes.
