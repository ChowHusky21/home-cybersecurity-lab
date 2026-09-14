# Architecture

## Host Environment
- **Machine:** MacBook, Apple M2 (Apple Silicon / ARM64)
- **Hypervisor:** UTM, using "Virtualize" (not "Emulate") for ARM64 guests.
  "Use Apple Virtualization" is left unchecked for both VMs — both run on
  the standard QEMU backend.
- VirtualBox was ruled out first, since it has no ARM64 support on Apple
  Silicon.

## Networks
- **Shared Network (NAT):** `192.168.64.0/24` — outbound internet access
  only, used for tool/package updates.
- **Host-Only network** ("Default (private)"): `192.168.128.0/24` — the
  isolated segment shared between the attacker and target VMs, with no
  path to the internet or the real home network.

## Virtual Machines

### Attacker VM — Kali Linux
- Kali Linux 2026.2, QEMU (Virtualize), 4096 MB RAM, 4 CPU cores, UEFI,
  64 GB disk, read-only shared directory.
- Dual-homed:
  - `eth0` — NAT — `192.168.64.6/24` (internet access confirmed)
  - `eth1` — Host-Only — `192.168.128.8/24` (cross-VM connectivity confirmed)
- A serial console device was added as a workaround for a graphical-install
  black-screen issue during setup.
- Guest tools: `spice-vdagent` and `qemu-guest-agent` installed directly.

### Target VM — Ubuntu Server
- Ubuntu Server 26.04.1 LTS, QEMU (Virtualize), 2048 MB RAM, 2 CPU cores,
  UEFI, 64 GB disk, read-only shared directory.
- Single interface, Host-Only **only** by design:
  - `enp0s1` — Host-Only — `192.168.128.6/24`
  - No internet access — this is intentional isolation, not an oversight.
- Guest tools: `qemu-guest-agent` installed via a temporary-NAT-then-remove
  pattern (see below); `spice-vdagent` was skipped since the VM is headless.

## IP Addressing Summary
| VM | Interface | Network | Address |
|---|---|---|---|
| Attacker (Kali) | eth0 | NAT | 192.168.64.6/24 |
| Attacker (Kali) | eth1 | Host-Only | 192.168.128.8/24 |
| Target (Ubuntu Server) | enp0s1 | Host-Only | 192.168.128.6/24 |

## Isolation Pattern: Temporary NAT
The target VM has no permanent internet access. Whenever an offline
package or container image needs to be pulled onto it, a temporary NAT
interface is added, the package/image is fetched, and the NAT interface
is removed again — returning the VM to Host-Only-only before any testing
resumes.

## Verification Checklist (all confirmed complete)
- [x] Kali `eth0` (NAT): has an IP and internet access
- [x] Kali `eth1` (Host-Only): has an IP
- [x] Target `enp0s1` (Host-Only): has an IP
- [x] Cross-VM ping succeeds in both directions
- [x] Target has **no** internet access (`ping 8.8.8.8` is unreachable)

This last check is the isolation boundary this whole lab depends on — it's
verified before any testing against the target begins.
