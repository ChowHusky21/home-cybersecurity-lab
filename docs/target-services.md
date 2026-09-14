# Target Services

All four services run as Docker containers on the target Ubuntu Server VM.
Docker (`docker.io` + `docker-compose-v2`, installed via `apt`, version
29.1.3-0ubuntu4.1) was chosen over snap Docker and over MicroK8s (which
would have needed more RAM than was available). All four containers were
given `--restart unless-stopped` so they survive a VM/daemon reboot (see
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

## 3. SSH Weak-Credential Target
- Image: `devdotnetorg/openssh-server:ubuntu`
- Port: `2222`, `USER_PASSWORD=letmein123`
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

## Current State
All four containers run simultaneously on the target VM (`juice-shop` on
3000, `dvwa` on 8080, `ssh-target` on 2222, `webgoat` on 8081/9090), with
comfortable RAM headroom (~630 Mi available) at that load. All four have
`--restart unless-stopped` applied.

## Logging
No extra logging tooling was needed:
- SSH attempts: `docker logs -f ssh-target` (sshd runs inside the
  container, so its log lives there rather than the host's `auth.log`)
- Container activity generally: `docker logs -f <container>`
- Web app requests: handled internally by Juice Shop / DVWA / WebGoat
