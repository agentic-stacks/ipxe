# Known Issues

iPXE uses a rolling-release model — there are no versioned releases. Track issues by git commit hash or date range.

## How to Use This File

Before upgrading iPXE or changing build options, check this file for relevant issues. When you encounter a new issue, document it following the template below.

## Issue Template

```
### [Short Description]

**Symptom:** What the operator sees
**Cause:** Why it happens
**Workaround:** Exact steps to fix or avoid
**Affected commits:** <hash>..<hash> or date range
**Status:** Open / Fixed in <commit-hash>
```

## General Known Issues

### UEFI Boot Performance

**Symptom:** UEFI network boot is substantially slower than BIOS boot.
**Cause:** UEFI SNP driver model adds overhead compared to BIOS UNDI.
**Workaround:** Use BIOS boot mode when possible. For UEFI-only hardware, this is a known limitation with no fix.
**Affected commits:** All
**Status:** Open (architectural limitation)

### Older BIOS Does Not Detect iPXE ROM

**Symptom:** After flashing iPXE to NIC ROM, it does not appear in BIOS boot menu.
**Cause:** Some older BIOSes require INT19 hooking for non-PnP ROMs.
**Workaround:** Enable `NONPNP_HOOK_INT19` build option:
```c
/* config/local/general.h */
#define NONPNP_HOOK_INT19
```
**Affected commits:** All
**Status:** Open (hardware limitation)

### autoexec.ipxe Not Supported on All Platforms

**Symptom:** `autoexec.ipxe` on TFTP server is not discovered or executed.
**Cause:** Not all iPXE platforms implement autoexec discovery.
**Workaround:** Use embedded scripts (EMBED=) or DHCP user-class detection instead.
**Affected commits:** All
**Status:** Open (platform-dependent feature)

## Reporting New Issues

When you encounter a new iPXE issue:

1. Identify the git commit: `cd ipxe && git log --oneline -1`
2. Check if the issue exists in the latest commit: `git pull && make bin/undionly.kpxe`
3. If the issue persists, use `git bisect` to find the introducing commit (see `skills/operations/troubleshooting`)
4. Document the issue using the template above
5. Report upstream at https://github.com/ipxe/ipxe/issues
