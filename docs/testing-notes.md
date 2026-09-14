# Testing Notes

## SSH Brute-Force (Hydra)
The SSH weak-credential target (`devdotnetorg/openssh-server:ubuntu`,
port 2222, see [target-services.md](target-services.md)) was verified
end-to-end with a brute-force credential attack from the Kali attacker VM:

- Target: `ssh://192.168.128.6:2222`
- Tool: Hydra
- A quick test wordlist successfully found the login `root` / `letmein123`.
- A full `rockyou.txt` wordlist run was also confirmed working, with an
  estimated ~39.5 hour completion time at full size. This run was stopped
  early as a proof of concept rather than run to completion, and is
  resumable via `hydra -R`.
- Note: Kali ships `rockyou.txt` compressed by default, so it had to be
  decompressed first with `sudo gunzip /usr/share/wordlists/rockyou.txt`.

## Target Selection: Why Not Metasploitable3
Metasploitable3-Ubuntu was considered and ruled out as the SSH/network
target: it's an x86-only VM image with no confirmed ARM64 Docker port,
which doesn't run natively on Apple Silicon. `devdotnetorg/openssh-server`
was chosen instead as a genuinely ARM64-native substitute that fills the
same "weak-credential network service" role, and was confirmed crackable
via Hydra as described above.

## A Deliberate Exclusion: vsftpd 2.3.4
Early scoping considered an FTP-based vulnerable target, but building one
around the real vsftpd 2.3.4 backdoor was explicitly declined — that
backdoor is genuine historical malware, not vulnerable-by-design training
software, and doesn't belong in a self-hosted lab like this one.

## Scope Note
All testing described in this repository was performed exclusively
against the intentionally-vulnerable, self-hosted, fully isolated target
VM documented in [architecture.md](architecture.md) — never against any
system outside this lab. See [SECURITY.md](../SECURITY.md) for the full
policy.
