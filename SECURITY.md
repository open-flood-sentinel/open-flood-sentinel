# Security policy

In a warning system, security is part of safety. An attacker who can **fake** alarms could cause panic or wear down trust. An attacker who can **suppress** alarms could cost lives.

## Threats we design against
- Spoofed or replayed heartbeats that make destroyed nodes look alive and so **hide a flood** (mitigated by per-node message authentication codes and frame counters, see [PROTOCOL.md](docs/PROTOCOL.md))
- Radio jamming, which causes mass silence (detected by stations measuring the noise floor)
- Compromised station or base credentials
- Unauthorised alert triggering through the dashboard or API

## Reporting a vulnerability

**Do not open a public issue.** Use GitHub's **private vulnerability reporting** (Security tab → "Report a vulnerability"). We aim to acknowledge reports within 7 days.
