# Code Signing

Verify that downloaded boot images are authentic and unmodified before execution.

## Enable Code Signing

Add to `config/local/general.h`:
```c
#define IMAGE_TRUST_CMD
```

This enables the `imgtrust` and `imgverify` commands.

## Create a Code-Signing Certificate

```bash
# Generate signing key
openssl genrsa -out codesign.key 4096

# Generate CSR
openssl req -new -key codesign.key -out codesign.csr \
  -subj "/CN=iPXE Code Signing"

# Sign with your CA
openssl x509 -req -in codesign.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out codesign.crt -days 365 -sha256
```

## Sign a Boot Image

```bash
# Sign the boot script
openssl cms -sign -binary -noattr -in boot.ipxe \
  -signer codesign.crt -inkey codesign.key -certfile ca.crt \
  -outform DER -out boot.ipxe.sig
```

Place both `boot.ipxe` and `boot.ipxe.sig` on your web server.

## Verify in iPXE Script

```
#!ipxe
imgtrust

# Download and verify the boot script
imgfetch http://192.168.1.20/boot.ipxe
imgverify boot.ipxe http://192.168.1.20/boot.ipxe.sig
chain boot.ipxe
```

After `imgtrust`, iPXE refuses to execute any image that has not been verified with `imgverify`. This is enforced for the remainder of the session.

## Build with Trusted Signing Certificate

Embed the signing CA so iPXE can verify signatures:

```bash
make bin/undionly.kpxe TRUST=../certs/ca.crt EMBED=secure-boot.ipxe
```

## Workflow

1. Build iPXE with `IMAGE_TRUST_CMD` and `TRUST=ca.crt`
2. Embedded script calls `imgtrust` to enable verification
3. All subsequent image downloads require `.sig` files
4. iPXE verifies each image against the trusted CA before execution
5. If verification fails, iPXE refuses to execute the image
