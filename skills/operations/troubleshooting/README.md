# Troubleshoot iPXE Boot Failures

Symptom-based decision trees for diagnosing common iPXE problems.

## No DHCP Response

```
Machine boots iPXE but gets no IP address
│
├─ Is the network cable connected / link light on?
│  └─ No → Fix physical connection. Use `iflinkwait` to wait for link.
│
├─ Is the DHCP server running?
│  └─ No → Start DHCP service.
│
├─ Can other machines get DHCP on this network?
│  └─ No → DHCP server or network issue, not iPXE.
│
├─ Is iPXE using the right interface?
│  └─ Run `ifstat` to list interfaces. Try `dhcp net0` explicitly.
│
├─ Is there a VLAN between client and DHCP server?
│  └─ Yes → Create VLAN: `vcreate --tag <id> net0` then `dhcp net0-<id>`
│
└─ Enable debug: rebuild with `DEBUG=dhcp` and check output.
```

## Infinite Chainload Loop

```
Machine downloads undionly.kpxe, reboots, downloads again, repeats
│
├─ Is DHCP configured to detect iPXE clients?
│  └─ No → Add user-class detection. See skills/deploy/chainloading.
│
├─ Is the iPXE user-class option spelled correctly?
│  └─ Check: option user-class = "iPXE" (case-sensitive)
│
├─ Using embedded script instead of DHCP detection?
│  └─ Verify: make bin/undionly.kpxe EMBED=script.ipxe
│
└─ Using autoexec.ipxe?
   └─ Verify file is on TFTP server root. Not all platforms support this.
```

## Image Download Fails

```
iPXE reports "Could not fetch" or "Connection refused"
│
├─ Is the URL correct?
│  └─ Test from another machine: curl http://192.168.1.20/boot.ipxe
│
├─ Is the web server running?
│  └─ Check service status and port binding.
│
├─ DNS resolution failing?
│  └─ Use IP address instead. Or run `nslookup hostname` in iPXE.
│
├─ HTTPS certificate error?
│  └─ See "Certificate Errors" below.
│
├─ Firewall blocking the connection?
│  └─ Check firewall rules on boot server. iPXE needs TCP 80 (HTTP) or 443 (HTTPS).
│
└─ Enable debug: rebuild with `DEBUG=http,tcp,dns` and check output.
```

## Certificate Errors

```
iPXE reports "Permission denied" or "TLS handshake failed" on HTTPS URLs
│
├─ Is DOWNLOAD_PROTO_HTTPS enabled in the build?
│  └─ No → Add to config/local/general.h and rebuild.
│
├─ Is the server certificate issued by a trusted CA?
│  └─ Check: iPXE trusts its root CA by default. Public CAs work via crosscert.
│
├─ Can iPXE reach the crosscert server?
│  └─ Default is http://ca.ipxe.org/auto. Needs outbound HTTP.
│
├─ Is the system clock wildly wrong?
│  └─ Certificate validation uses timestamps. Sync with NTP: `ntp pool.ntp.org`
│
├─ Using a custom CA?
│  └─ Verify: built with TRUST=ca.crt pointing to the correct certificate.
│
└─ Self-signed certificate?
   └─ Build with TRUST= pointing to the self-signed cert.
```

## Script Syntax Errors

```
iPXE reports "Syntax error" or script doesn't execute
│
├─ Does the script start with #!ipxe?
│  └─ No → Add `#!ipxe` as the very first line. No spaces before it.
│
├─ Are label names valid?
│  └─ Labels must be alphanumeric: `:retry_boot` works, `:retry boot` does not.
│
├─ Are there Windows line endings (\r\n)?
│  └─ Convert to Unix: dos2unix script.ipxe
│
├─ Variable expansion working?
│  └─ Use ${variable} syntax. Check setting exists with `show variable`.
│
└─ Using reserved characters in URLs?
   └─ Use :uristring suffix: ${variable:uristring}
```

## Slow UEFI Boot

```
UEFI boot takes much longer than BIOS boot
│
└─ This is expected. UEFI network booting is substantially slower than BIOS.
   ├─ If performance matters, prefer BIOS boot mode when possible.
   └─ For UEFI-only hardware, this is a known limitation.
```

## Debugging Tools

| Tool | Purpose | How |
|---|---|---|
| Debug build | Verbose module output | `make bin/undionly.kpxe DEBUG=dhcp,tcp,http` |
| Serial console | Capture boot output | Enable `CONSOLE_SERIAL`, connect serial cable |
| Syslog | Remote log capture | `set syslog 192.168.1.30` in script |
| Packet capture | Network-level debugging | `tcpdump` on DHCP/boot server |
| VMware log | VM boot output | Enable `CONSOLE_VMWARE` build option |
| iPXE shell | Interactive debugging | Press Ctrl-B during boot (if not embedded) |

## Capture a Packet Trace

On the boot server:
```bash
sudo tcpdump -i eth0 -w ipxe-boot.pcap port 67 or port 68 or port 69 or port 80 or port 443
```

Analyze with Wireshark. Filter for DHCP, TFTP, and HTTP traffic.

## Git Bisect for Regressions

If a recent iPXE update broke something:

```bash
cd ipxe
git bisect start
git bisect bad HEAD
git bisect good <last-known-good-commit>
# Build and test at each step
cd src && make bin/undionly.kpxe && cd ..
# Mark result
git bisect good  # or: git bisect bad
```
