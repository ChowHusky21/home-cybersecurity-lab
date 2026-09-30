# Target Services

Four containerized services run on the target Ubuntu Server VM, plus a
separate Windows 11 target VM for OS-level practice. Docker (`docker.io` +
`docker-compose-v2`, installed via `apt`, version 29.1.3-0ubuntu4.1) was
chosen over snap Docker and over MicroK8s (which would have needed more
RAM than was available). All four containers were given
`--restart unless-stopped` so they survive a VM/daemon reboot (see
[troubleshooting.md](troubleshooting.md) for why that was needed).

## 1. OWASP Juice Shop
- Image: `bkimminich/juice-shop`
- Port: `3000`
- Confirmed reachable from Kali at `http://192.168.128.6:3000`
- Modern, intentionally-vulnerable web application used for general
web-app vulnerability training.

## 2. DVWA (Damn Vulnerable Web Application)
- Image: `kaakaww/dvwa-docker:latest`
- Port: `8080`
- Confirmed reachable from Kali at `http://192.168.128.6:8080/login.php`
- Login: `admin` / `password`
- Chosen over `vulnerables/web-dvwa` for confirmed ARM64 support.
- The image ships with a packaging bug that crash-loops the container on
every start (a file it expects is missing from the image); a durable
entrypoint-wrapper fix is applied and verified — see
[troubleshooting.md](troubleshooting.md) for the root cause and fix.

## 3. SSH Weak-Credential Target
- Image: `devdotnetorg/openssh-server:ubuntu`
- Port: `2222`
- Chosen as an ARM64-native substitute for Metasploitable3-Ubuntu, which
was ruled out (x86-only VM image, no confirmed ARM64 Docker port) — see
the decisions log in [testing-notes.md](testing-notes.md) for the full
reasoning, including why the real vsftpd 2.3.4 backdoor was explicitly
not used.
- Verified end-to-end with Hydra — see [testing-notes.md](testing-notes.md).

## 4. OWASP WebGoat
- Image: `webgoat/webgoat`
- Ports: `8081` → container's `8080` (WebGoat itself), `9090` (WebWolf,
its companion app)
- `TZ=America/Los_Angeles` set explicitly, since some WebGoat lessons issue
JWTs whose validity depends on the container's clock matching a real
timezone.
- Confirmed reachable from Kali at `http://192.168.128.6:8081/WebGoat/login`
- OWASP's own official training app; confirmed ARM64 support directly from
its GitHub releases.
- Shows a persistent `(unhealthy)` status in `docker ps` — diagnosed as a
cosmetic packaging bug (its healthcheck calls a binary that isn't
installed in the image), not a real problem. See
[troubleshooting.md](troubleshooting.md).

## 5. Windows 11 Target VM
- Not a container — a separate UTM guest VM, ARM64, running natively
(not emulated).
- Host-only network only, isolated the same way as the Ubuntu target: no
internet access once built and patched.
- Confirmed reachable to and from the Kali attacker VM.
- Currently a plain, updated Windows 11 install used for OS-level
enumeration and network-service practice; not yet deliberately
configured as vulnerable-by-design.

## Service Enumeration
`nmap -sV -O` against the Ubuntu target confirmed the expected exposed
services and OS fingerprint: OpenSSH (both the host's own SSH and the
SSH target container's separate OpenSSH instance on 2222), OWASP Juice
Shop on 3000, and Apache Tomcat serving WebGoat/WebWolf on 8081/9090 —
consistent with what each container's own documentation above states,
confirmed independently from the attacker VM's point of view rather than
just from the target's `docker ps` output.

## Current State
All four Ubuntu-target containers run simultaneously (`juice-shop` on
3000, `dvwa` on 8080, `ssh-target` on 2222, `webgoat` on 8081/9090), with
comfortable RAM headroom at that load, all with `--restart unless-stopped`
applied. The Windows 11 target VM runs independently alongside them on
the same isolated network.

## Logging
No extra logging tooling was needed:
- SSH attempts: `docker logs -f ssh-target` (sshd runs inside the
container, so its log lives there rather than the host's `auth.log`)
- Container activity generally: `docker logs -f <container>`
- Web app requests: handled internally by Juice Shop / DVWA / WebGoat
