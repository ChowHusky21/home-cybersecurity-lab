# Troubleshooting Log

Real debugging work encountered while building and maintaining this lab.

## Containers Not Surviving a Restart
After adding WebGoat as the fourth target, `docker ps` showed only 2 of 4
containers running (`webgoat` and `ssh-target`) — DVWA and Juice Shop had
silently exited at some point. The root cause: none of the original
`docker run` commands for those two containers included a restart policy,
so a target VM / Docker daemon restart didn't bring them back.

- `docker start dvwa juice-shop` appeared to work for `juice-shop`, but
  DVWA's restart attempt silently failed again — most likely stale
  MySQL lock/PID files inside the bundled LAMP-stack image, left over from
  an earlier unclean stop, causing `dvwa-docker`'s internal startup script
  to fail on relaunch.
- A separate fresh `docker run` (without `--name`) for DVWA then
  succeeded, but created an auto-named orphan container
  (`priceless_shtern`) instead of fixing the original.

**Resolution:** removed the broken original `dvwa` container, stopped and
removed `priceless_shtern`, then created a clean new `dvwa` container from
scratch with `--restart unless-stopped` baked in from creation. Also
cleaned up three other orphaned/anonymous exited `juice-shop` containers
(`stupefied_almeida`, `beautiful_jepsen`, `jovial_tesla`) left over from
earlier restart attempts — the actual running `juice-shop` container was
never broken, it just had dead siblings accumulating alongside it.

Applied `docker update --restart unless-stopped` retroactively to all four
target containers afterward, to prevent this recurring after any future
reboot.

## WebGoat's Persistent "(unhealthy)" Status
`docker ps` shows WebGoat as persistently `(unhealthy)`. Diagnosed via:

```
docker inspect --format '{{json .State.Health}}' webgoat
```

The image's built-in health check shells out to `curl`, but `curl` isn't
actually installed in the container's filesystem
(`/bin/sh: 1: curl: not found`). This is a packaging oversight in the
image itself — its health check and its installed tools drifted out of
sync — not a real problem: WebGoat's login page was independently
confirmed working in Firefox regardless.

Since nothing else in this lab depends on Docker's health-check status,
this was left as a cosmetic non-issue. An optional silence-it fix, if
desired: `docker update --no-healthcheck webgoat`.

## Other Setup Issues (resolved earlier, summarized)
A number of other issues were worked through during initial VM creation
and are noted here for completeness, though their detailed command-by-command
fixes live only in earlier Drive document revisions:

- Kali's graphical installer showed a black screen — worked around by
  adding a serial console device.
- Kali's `eth0` was missing its NetworkManager profile after install —
  fixed manually via `nmcli`.
- Cross-VM connectivity broke due to a stale network attachment — fixed
  with a full VM restart.
- The target VM's boot took longer than expected due to `netplan` waiting
  on an interface that has no link by design — fixed by setting
  `optional: true` for that interface in the netplan config.
- Docker's official apt repository was abandoned after hitting a genuine
  upstream GPG key mismatch; Ubuntu's native `docker.io` package was used
  instead.

## Still Undecided
- Whether to revisit MicroK8s later with a bumped RAM allocation.
- Whether to add VS Code to the Kali VM via its apt repo (ARM64 snap
  support was uncertain at setup time, so this was deferred).
