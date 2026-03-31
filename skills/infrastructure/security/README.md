# iPXE Security

iPXE supports HTTPS, code signing, image encryption, and client certificates for secure network booting.

> **Critical Rule:** Always use HTTPS for boot scripts in production. HTTP boot scripts can be MITM'd to serve malicious kernels.

## Security Features

| Feature | Build Option | Commands | Guide |
|---|---|---|---|
| HTTPS downloads | `DOWNLOAD_PROTO_HTTPS` | — | `https.md` |
| Custom trust chain | `TRUST=` build parameter | `certstat`, `certstore`, `certfree` | `custom-ca.md` |
| Code signing | `IMAGE_TRUST_CMD` | `imgtrust`, `imgverify` | `code-signing.md` |
| Client certificates | `CERT=` / `KEY=` build parameters | — | `client-certs.md` |
| Image encryption | `IMAGE_CRYPT_CMD` | `imgdecrypt` | — |

## Quick Start: Enable HTTPS

```bash
cd ipxe/src
cat > config/local/general.h << 'EOF'
#define DOWNLOAD_PROTO_HTTPS
#define CERT_CMD
EOF
make bin/undionly.kpxe
```

iPXE now supports `https://` URLs. It trusts the iPXE root CA by default, which cross-signs most public CAs trusted by Firefox.
