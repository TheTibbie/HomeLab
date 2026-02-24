# Operations and Backups

## Backups

Both Pi-hole containers are included in the weekly Proxmox → PBS backup job. Retention, garbage collection, and integrity verification are managed centrally on PBS rather than at the job level.

See [PBS weekly backups](../../02-proxmox/pbs-weekly-backups/) for full configuration details.

---

## Operational checks

Routine checks are lightweight given the simplicity of the service:

- Confirm both web UIs are reachable
- Confirm both DNS endpoints respond to queries
- Spot-check blocking behavior periodically

---

## Recovery posture

If one instance is unavailable, clients may continue resolving via the second DNS entry depending on client behavior — see [DNS design and redundancy approach](02-dns-and-redundancy.md) for detail on how fallback works in practice. For container corruption or misconfiguration, the preferred recovery path is a PBS restore rather than manual remediation.
