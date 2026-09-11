# Backup Security and Validation

## Goal

Keep configuration backups recoverable without placing high-value secret material into GitHub or leaving sensitive archives unprotected in off-host storage.

---

## Off-Host Copy

Configuration backups are copied to private OneDrive storage under a dedicated HomeLab backup structure.

The off-host copy is important because a configuration backup stored only on the system it protects does not help if that system or its local storage is lost.

---

## Encryption Approach

Not every backup is handled the same way.

Native application encryption is used where available. Especially sensitive configuration that does not already have suitable protection is encrypted before leaving the host.

Sensitive examples include:

- Proxmox and PBS credential-bearing configuration
- Private key material
- Identity state
- Monitoring credentials and secrets

Encryption passwords are kept in the password manager rather than beside the backup files.

---

## Integrity Validation

SHA-256 hashes were used throughout the project to confirm that files copied off-host matched the source backup.

For encrypted sensitive archives, encryption was tested before the plaintext staging copy was removed.

The general process was:

```text
Create backup
    ↓
Calculate hash
    ↓
Encrypt if required
    ↓
Copy off-host
    ↓
Verify destination
    ↓
Remove temporary plaintext/staging data
```

---

## Application Consistency

Services using SQLite or other stateful local data were handled carefully so a copied database was not captured in the middle of a write.

Where required, the application or container was briefly stopped while the consistent state was copied, then started again and validated.

Native application export functions were preferred where they provided a cleaner recovery path.

---

## Recovery Documentation

Backup directories include recovery notes and checksum information where appropriate.

The goal is not only to possess the files. The backup should still be understandable if recovery is being performed months later on replacement hardware.

---

## Ongoing Standard

The first full configuration-backup pass is complete.

Going forward, backups should be refreshed after meaningful changes such as:

- Network redesigns
- New infrastructure services
- Major Proxmox or PBS configuration changes
- DNS or reverse-proxy changes
- Monitoring architecture changes
- Changes to custom recovery-critical scripts or service units

Periodic restore testing remains more valuable than assuming a successful copy automatically means a usable backup.
