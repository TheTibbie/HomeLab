# Configuration Backups

## Summary

This section documents the configuration-backup project completed to make the homelab easier to rebuild after a major failure.

These are not the same as the normal Proxmox guest backups stored on PBS.

PBS protects VM and LXC workloads. The configuration-backup project protects the settings, exports, scripts, and recovery information needed to rebuild the infrastructure itself.

No backup archives or secret material are stored in this repository.

---

## Backup Goal

The project focused on answering a simple recovery question:

```text
If the lab hardware failed, could the important infrastructure be rebuilt without relying on memory?
```

Configuration backups were created for the major active infrastructure components and copied off-host.

---

## Current Coverage

The documented configuration backup set includes:

- Proxmox host configuration from all four nodes
- Sensitive Proxmox cluster configuration
- Proxmox Backup Server host and application configuration
- OPNsense configuration export
- Omada Controller configuration export
- Both Pi-hole configurations
- Observability configuration and selected persistent state
- Uptime Kuma configuration and state
- Home Assistant native backup
- Dashy configuration
- Reverse proxy / Caddy configuration
- Supporting custom scripts and systemd units where required for recovery

Some application-specific configuration is intentionally not documented in this repository.

---

## Off-Host Storage

The backup set is stored in private OneDrive storage under a dedicated HomeLab backup structure.

Backups were grouped by system rather than placing everything into one large archive. This makes individual service recovery easier and reduces the chance that one damaged archive affects the entire configuration backup set.

---

## Security

High-value secret material is encrypted before leaving the host.

Examples include private keys, identity state, credential-bearing Proxmox/PBS configuration, and monitoring secrets.

Passwords and encryption passphrases are stored separately in the password manager and are not stored beside the backup files.

---

## Pages

- [Backup coverage](01-backup-coverage.md)
- [Security and validation](02-security-and-validation.md)

---

## Current Status

The initial full configuration-backup project is complete.

The next requirement is operational rather than architectural: repeat the exports after meaningful infrastructure changes and periodically validate that the recovery material is still usable.
