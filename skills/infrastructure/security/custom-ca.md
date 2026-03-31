# Custom Certificate Authority

For private infrastructure, create your own CA and build iPXE to trust it instead of the default iPXE root CA.

## Generate a Root CA

```bash
# Generate private key
openssl genrsa -out ca.key 4096

# Generate self-signed root certificate (10 years)
openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 \
  -out ca.crt -subj "/CN=My iPXE Root CA"
```

## Issue a Server Certificate

```bash
# Generate server key
openssl genrsa -out server.key 2048

# Generate CSR
openssl req -new -key server.key -out server.csr \
  -subj "/CN=boot.example.com"

# Sign with your CA
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out server.crt -days 365 -sha256
```

Install `server.crt` and `server.key` on your HTTPS boot server.

## Build iPXE with Custom Trust

```bash
cd ipxe/src
make bin/undionly.kpxe TRUST=../certs/ca.crt
```

The `TRUST=` parameter replaces the default iPXE root CA. iPXE will only trust certificates signed by your CA.

### Trust Multiple CAs

```bash
make bin/undionly.kpxe TRUST=../certs/ca1.crt,../certs/ca2.crt
```

## Disable Cross-Certificate Downloads

When using a custom CA, you likely don't need cross-certificates:

```
#!ipxe
set crosscert ""
chain https://boot.example.com/boot.ipxe
```

## UEFI Platform Certificates

On UEFI platforms, iPXE can use certificates provided by the firmware. This is configured automatically — no build options needed.

## Cross-Sign a Public CA

If you need to trust both your private CA and public CAs (e.g., for downloading from public mirrors):

```bash
# Cross-sign the public CA's root with your private CA
openssl x509 -req -in public-ca-root.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out public-ca-cross.crt -days 365 -sha256
```

Serve `public-ca-cross.crt` from your cross-certificate server.
