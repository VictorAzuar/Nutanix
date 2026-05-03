# 15 - VM Snapshots

## 1. Purpose of this chapter

This chapter clarifies snapshots as operational recovery points, not a blanket backup strategy. It focuses on consistency, retention, performance, restore scope, and customer expectation management.

## 2. Core definition

A VM snapshot is a point-in-time capture of a VM’s disk state and sometimes metadata, used for rollback, cloning, replication, protection workflows, or short-term recovery.

## 3. Main explanation

Snapshots are valuable but often misunderstood. A snapshot can help recover from a bad change, accidental file deletion, patch issue, corruption event, or migration problem. But snapshot value depends on timing, retention, consistency level, application behavior, guest tools, and restore workflow.

Crash-consistent snapshots capture disk state as if power were lost. Application-consistent recovery usually needs guest coordination, such as VSS for Windows workloads and appropriate guest tools or backup integration. Long-lived or excessive snapshots can contribute to operational complexity and performance concerns depending on platform behavior and workload.

A support manager should always clarify what the customer needs to restore: full VM, vDisk, file, application object, or point in time. Do not let “we have snapshots” become a false sense of backup or DR readiness.

## 4. Key concepts

- Snapshots are point-in-time recovery mechanisms.
- Snapshot consistency matters: crash-consistent is not the same as application-consistent.
- Snapshots and backups serve different operational purposes.
- Restore scope must be explicit: VM, disk, file, or application.
- Retention and snapshot growth must be managed.

## 5. Support relevance

Snapshot escalations are emotionally charged because they often involve data loss, failed changes, or executive pressure. The support manager must slow the case down enough to avoid destructive recovery attempts while establishing exact restore objectives and available recovery points.

## 6. Escalation scenario

A customer deleted a critical folder and expects file-level restore from a VM snapshot. The escalation lead asks for the VM, file path, deletion timestamp, available snapshots, NGT/SSR status, backup software availability, and whether the application was quiesced. The team finds VM-level snapshots exist but file-level restore was not configured. Communication shifts to alternate restore paths and prevention for future incidents.

## 7. Triage questions

- What exactly needs to be restored: VM, disk, file, database, or application object?
- What timestamp is required and what recovery points exist?
- Was the snapshot crash-consistent or application-consistent?
- Were NGT, VSS, SSR, or backup agents configured before the snapshot?
- Is the customer asking for rollback, clone, mount, or file-level restore?
- Could the recovery action overwrite newer data?
- Are snapshots being used as a substitute for backup or DR?

## 8. What to avoid

- Weak: “We have a snapshot, so recovery is guaranteed.” Better: “Validate recovery point, consistency, restore scope, and tooling before promising recovery.”
- Weak: “Snapshots are backups.” Better: “Snapshots are useful recovery points, but backup/DR require separate design and retention.”
- Weak: “Rollback now.” Better: “Confirm business impact and data overwrite risk before rollback.”

## 9. Minimum to memorize

- Snapshots capture a point in time.
- Consistency and restore scope determine usefulness.
- Snapshots are not automatically backups.
- File-level restore may require guest tools or backup integration.
- Recovery actions can overwrite data and must be controlled.

## 10. Glossary: keywords and acronyms

| Keyword / Acronym | Meaning |
|---|---|
| Application-consistent | Snapshot coordinated with application state. |
| Crash-consistent | Disk state captured without application quiescing. |
| NGT | Nutanix Guest Tools. |
| Recovery point | Specific time available for restore. |
| Rollback | Returning a VM or disk to an earlier state. |
| Snapshot | Point-in-time capture. |
| SSR | Self-Service Restore. |
| VSS | Microsoft Volume Shadow Copy Service. |
