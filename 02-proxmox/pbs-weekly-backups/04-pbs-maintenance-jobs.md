# PBS Maintenance Jobs (Prune, GC, and Verify)

## Purpose

Manage backup lifecycle and storage integrity centrally on PBS for datastore `t7-backups`.

These jobs run automatically after the weekly backup window closes. They keep retention bounded and provide ongoing confidence that retained backups are intact and restorable.

---

## Maintenance Schedule

| Job | Schedule | Configuration |
|---|---|---|
| Prune | Sundays at `02:00` | `keep-last=2` |
| Garbage Collection | Sundays at `03:00` | Scheduled after prune |
| Verify | Sundays at `04:00` | Re-verify after 30 days |

---

## Why This Sequence Matters

The three jobs run in deliberate order.

Prune runs first to mark old snapshots for removal.

Garbage collection follows to reclaim the physical space freed by pruning. In a deduplicated datastore, space is not released until garbage collection runs, so the order matters.

Verify runs last against what remains. This provides a weekly confirmation that the retained backups are intact and restorable.

Running verify after prune and garbage collection means it checks the backups that actually matter, not snapshots that are about to be removed.

---

## Post-Cutover Note

After the OPNsense VLAN cutover, PBS was migrated onto the Management / Servers VLAN and validated from the Proxmox nodes.

The maintenance model did not change. PBS still owns retention, pruning, garbage collection, and verification after the network migration.

---

## Operational Standard

PBS maintenance jobs should remain enabled and reviewed periodically.

Routine checks should confirm:

- Prune jobs are scheduled
- Garbage collection jobs are scheduled
- Verify jobs are scheduled
- The expected datastore is targeted
- Backup and restore validation continues as part of normal operations
