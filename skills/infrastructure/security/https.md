# HTTPS Configuration

## Enable HTTPS

Add to `config/local/general.h`:
```c
#define DOWNLOAD_PROTO_HTTPS
```

Rebuild iPXE. All `https://` URLs now work.

## Trust Chain Architecture

iPXE ships with one trusted root certificate: the **iPXE root CA**. To validate certificates from public CAs (Let's Encrypt, DigiCert, etc.), iPXE downloads cross-signed certificates that bridge the public CA to the iPXE root CA.

```
Public CA (e.g., Let's Encrypt)
    │
    └── Cross-signed by iPXE root CA
            │
            └── iPXE root CA (built into firmware)
```

## Cross-Certificate Source

By default, iPXE fetches cross-signed certificates from `http://ca.ipxe.org/auto`. This source covers nearly all CAs trusted by Firefox, with 90-day validity periods.

### Override the Source

In an iPXE script or at the command line:
```
set crosscert http://ca.ipxe.org/auto
```

### Private Networks

For networks without internet access, mirror the cross-certificates locally and redirect:
```
set crosscert http://192.168.1.20/ipxe-certs/auto
```

> **Note:** HTTPS is not needed for downloading cross-certificates — they are validated against the iPXE root CA, not the download channel.

## Supported TLS Versions

iPXE supports TLS 1.0, 1.1, and 1.2 with these algorithms:

- **Public-key:** RSA, ECDSA
- **Key exchange:** RSA, DHE, ECDHE
- **Ciphers:** AES-128-GCM, AES-256-GCM, AES-128-CBC, AES-256-CBC
- **Hashes:** SHA-256, SHA-384, SHA-512 (and legacy MD5, SHA-1)
- **Curves:** X25519, P-256, P-384
