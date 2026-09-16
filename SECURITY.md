# Security Policy

## Scope
This is a **documentation and lab-configuration repository**, not
software with ongoing releases — there are no "supported versions" in
the usual sense. Everything here describes a personal, fully isolated,
offline virtual lab used for cybersecurity self-education.

## Supported Versions

This is a documentation repository, not versioned software — there are no supported/unsupported release lines. The current `main` branch reflects the lab's current, maintained state.

## Isolation Guarantee
The target VM in this lab has no internet access and is reachable only
from the attacker VM over a host-only virtual network. No testing
documented in this repository was performed against any system outside
this isolated environment.

## Responsible Use
The tools and techniques referenced here (Hydra, DVWA, WebGoat, OWASP
Juice Shop) are all publicly available, intentionally-vulnerable
training software. Nothing in this repo should be used against any
system without that system owner's explicit, written authorization.

## Reporting a Concern
If you believe any content in this repository is inaccurate, misleading,
or could be misread as guidance for unauthorized access, please contact
daniel.t.harris@gmail.com directly rather than opening a public issue.
