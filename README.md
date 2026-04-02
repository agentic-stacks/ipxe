# iPXE — Agentic Stack

An [agentic stack](https://github.com/agentic-stacks/agentic-stacks) that teaches AI agents to build, deploy, and operate [iPXE](https://ipxe.org) network boot infrastructure.

## What This Stack Covers

- **Foundation** — iPXE architecture, building from source, build-time configuration
- **Deployment** — Chainloading, ROM burning, bootable media, VMware, AWS EC2, Google Cloud
- **Boot Targets** — Linux distributions, Windows PE/WDS/SCCM, SAN (iSCSI/FCoE/AoE)
- **Infrastructure** — DHCP server configuration, iPXE scripting, HTTPS and certificate security
- **Operations** — Troubleshooting decision trees, console configuration
- **Reference** — Command reference, settings reference, build targets, known issues

## Usage

```bash
agentic-stacks init agentic-stacks/ipxe my-netboot-project
cd my-netboot-project
```

The agent reads `CLAUDE.md` and becomes an iPXE expert operator.

## Composability

This stack pairs well with:
- Hardware provisioning stacks (Dell, HP, Supermicro) for bare-metal fleet management
- Kubernetes/OS stacks for automated cluster bootstrapping
- Configuration management stacks (Ansible, Salt) for post-boot provisioning

## Authoring

This stack follows the [authoring guide](https://github.com/agentic-stacks/agentic-stacks/blob/main/docs/guides/authoring-a-stack.md).
