# Client Certificates

iPXE supports mutual TLS — the server verifies the client's identity via a client certificate embedded in the firmware.

## Generate a Client Certificate

```bash
# Generate client key
openssl genrsa -out client.key 2048

# Generate CSR
openssl req -new -key client.key -out client.csr \
  -subj "/CN=ipxe-client-01"

# Sign with your CA
openssl x509 -req -in client.csr -CA ca.crt -CAkey ca.key \
  -CAcreateserial -out client.crt -days 365 -sha256
```

## Build iPXE with Client Certificate

```bash
cd ipxe/src
make bin/undionly.kpxe \
  TRUST=../certs/ca.crt \
  CERT=../certs/client.crt \
  KEY=../certs/client.key
```

The client certificate and private key are embedded in the firmware. iPXE presents them during TLS handshake when the server requests client authentication.

## Configure the Web Server

Example with nginx:
```nginx
server {
    listen 443 ssl;
    server_name boot.example.com;

    ssl_certificate /etc/ssl/server.crt;
    ssl_certificate_key /etc/ssl/server.key;
    ssl_client_certificate /etc/ssl/ca.crt;
    ssl_verify_client on;

    location /boot/ {
        root /var/www/html;
    }
}
```

## Per-Machine Certificates

For fleet deployments, build unique firmware per machine with distinct client certificates. This allows the boot server to identify each machine and serve machine-specific boot configurations.

```bash
# Build for machine-01
make bin/undionly.kpxe CERT=certs/machine-01.crt KEY=certs/machine-01.key TRUST=certs/ca.crt

# Build for machine-02
make bin/undionly.kpxe CERT=certs/machine-02.crt KEY=certs/machine-02.key TRUST=certs/ca.crt
```

The server can read the client certificate's CN from the TLS handshake and return a machine-specific boot script.
