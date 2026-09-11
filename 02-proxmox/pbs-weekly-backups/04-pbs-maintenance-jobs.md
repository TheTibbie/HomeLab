# PBS Maintenance Jobs (Prune, GC, and Verify)

## Purpose

Manage backup lifecycle and storage integrity centrally on PBS for datastore `t7-backups`.

The maintenance schedule was moved away from the weekend guest backup windows after the Proxmox backup jobs were staggered across Saturday and Sunday.

---

## Maintenance Schedule

| Job | Schedule | Configuration |
|---|---|---|
| Verify | Monday at `01:00` | `ignore-verified=1`, `outdated-after=30` |
| Prune | Wednesday at `02:00` | `keep-last=4`, `keep-monthly=3` |
| Garbage Collection | Wednesday at `03:00` | Scheduled after prune |

---

## Why the Schedule Changed

The original maintenance jobs ran directly after the Sunday backup window.

That worked when the cluster used a single weekly backup job, but the newer design spreads guest backups across Saturday and Sunday. Maintenance was moved to separate days so verification, pruning, and garbage collection do not compete with active guest backups.

The current order is deliberate:

1. Weekend guest backups complete.
2. Verify runs Monday against the retained backup data.
3. Prune runs Wednesday to apply retention.
4. Garbage collection follows prune and reclaims unused datastore space.

---

## Verify

The verification job runs Monday at `01:00`.

Current settings:

```text
ignore-verified=1
outdated-after=30
```

This avoids repeatedly re-verifying recently checked data while still forcing older backup data back through verification after 30 days.

---

## Prune and Garbage Collection

Prune runs Wednesday at `02:00` using:

```text
keep-last=4
keep-monthly=3
```

Garbage collection follows at `03:00`.

In a deduplicated datastore, prune determines which snapshots are no longer retained, while garbage collection performs the separate storage cleanup required to reclaim unused chunks.

---

## Validation

Historical prune, garbage collection, and verification jobs have completed successfully.

During the backup review, the datastore remained healthy and garbage collection had reclaimed unused storage as expected.

Routine checks should continue to confirm:

- Weekend backup jobs complete before maintenance begins
- Verify remains scheduled for Monday
- Prune and garbage collection remain scheduled for Wednesday
- The expected datastore is targeted
- Retention remains appropriate for available storage
- Periodic restore testing continues as part of normal operations
