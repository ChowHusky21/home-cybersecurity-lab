# Home Cybersecurity Lab

A self-hosted, fully isolated penetration-testing lab built and run on a
single Mac (Apple Silicon) using UTM/QEMU virtualization — for personal,
hands-on cybersecurity skill-building.

## Status Badges

_N/A — this repository documents a lab environment, not a software release; there is no CI/CD build, code-quality, or vulnerability-scan pipeline to badge._

## ⚠️ Scope & Disclaimer
This lab, and every testing tool or technique documented here, was used
**exclusively against intentionally-vulnerable, self-hosted, fully
isolated virtual machines** that I built and own, for personal
education. Nothing here was run against, or is intended for use against,
any system I do not own or have explicit authorization to test. See
[SECURITY.md](SECURITY.md) for the full policy. **Do not use these
techniques against any system without explicit written authorization.**
I am not responsible for any misuse of the information in this repo.

## Architecture
- **Host:** MacBook, Apple M2 (ARM64), hypervisor: UTM (QEMU backend, not
  Apple Virtualization) — chosen because VirtualBox has no ARM64 support
  on Apple Silicon.
- **Attacker VM:** Kali Linux 2026.2 — dual-homed: one NIC on a NAT
  network (internet access, for tool updates only) and one NIC on a
  host-only network shared with the target.
- **Target VM:** Ubuntu Server 26.04.1 LTS — host-only network **only**,
  by design: no internet access, fully isolated from the host's real
  network.
- Full architecture diagram and IP layout: [docs/architecture.md](docs/architecture.md)

## Architecture & Data Handling
All processing described in this repository happens **locally**, inside isolated virtual machines on the author's own hardware. No third-party service receives data from this lab. Required privileges are limited to standard VM/hypervisor access (UTM) and Docker on the host.

## Target Services
Four containerized, intentionally-vulnerable applications run on the target VM via Docker, each publicly known training software (not
custom-built vulnerable code):
- **OWASP Juice Shop** — modern web-app vulnerability training
- **DVWA** (Damn Vulnerable Web Application) — classic web vuln training
- **A weak-credential SSH service** — brute-force practice target
- **OWASP WebGoat** — guided web-app security lessons

Full configuration details: [docs/target-services.md](docs/target-services.md)

## Testing Performed
Brute-force credential testing against the SSH target using Hydra,
including a full rockyou.txt wordlist run. Full methodology and results:
[docs/testing-notes.md](docs/testing-notes.md)

## Setup / Verification
The lab includes a documented verification checklist confirming: both
VMs have correct network connectivity, cross-VM connectivity works in
both directions, and the target VM has **no internet access** (verified
by a failed ping to 8.8.8.8) — confirming the isolation boundary holds
before any testing begins.

## Verification & Secure Installation
_N/A — no releases._ This repo is documentation, not a software release — there are no binaries or scripts to verify via SHA-256/GPG. To reproduce the environment:
```bash
git clone https://github.com/ChowHusky21/home-cybersecurity-lab.git
```

**Never pipe a remote script into a shell** — this repo does not ask you to, and any fork that does should not be trusted.

## Configuration & Usage

No credentials are stored in this repository. If you reproduce this lab, keep any VM or service credentials in a local `.env` file (see `.env.example`) and never commit them. Run all services with the least privilege needed — the intentionally-vulnerable containers should stay on the isolated host-only network documented in the README, never bridged to a live network.

## Troubleshooting Log
Real debugging work performed while building this lab (container
restart-policy issues, orphaned container cleanup, a false-positive
health check) is documented in [docs/troubleshooting.md](docs/troubleshooting.md).

## Security
See [SECURITY.md](SECURITY.md) for the lab's scope, isolation
guarantees, and how to report a concern.

## License
[MIT](LICENSE) — documentation and configuration notes only; no exploit
code is included or distributed.
