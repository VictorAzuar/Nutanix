# 7 - Snapshots

## 1. Purpose of this chapter
This chapter explains snapshots as operational recovery points, not as a magic substitute for backup. Support managers need this distinction because snapshot cases often occur during outages, failed restores, capacity pressure, replication lag, audit concerns, or ransomware recovery.

## 2. Core definition
A **snapshot** is a point-in-time representation of a VM, vDisk, volume group, or protected entity that can support restore, clone, replication, and disaster-recovery workflows.

## 3. Main explanation
A Nutanix snapshot usually does not copy every block immediately. It records a point in time and relies on storage metadata and changed-data behavior so that later writes can be tracked separately. This makes snapshots fast and space efficient at creation time, but not free forever.

Snapshots become larger as data changes after the snapshot. Long retention, high-change workloads, backup integrations, test/dev activity, and failed cleanup can turn snapshots into capacity risks. They can also create customer expectation issues: a snapshot can help recovery, but it is not the same as an independent backup copy with separate retention, compliance, and disaster-recovery guarantees.

## 4. Key concepts
- **Point-in-time state**: the workload state captured at a specific time.
- **Redirect-on-write / changed data**: new writes are tracked separately from the snapshot state.
- **Crash consistency vs application consistency**: a restored VM may boot but application state may still require validation.
- **Retention**: how long snapshots are kept.
- **Protection policy/domain**: snapshot and replication management construct.
- **RPO/RTO**: recovery-point and recovery-time expectations.

## 5. Support relevance
Snapshot incidents are high trust events. Customers ask whether data can be recovered, whether restores are consistent, whether capacity will run out, or why replication is lagging. The manager should clarify what recovery point exists, what it protects, what consistency level it provides, and what risk remains.

## 6. Escalation scenario
A customer needs to restore a production VM after a failed application patch. The VM has snapshots, but after restore the database service starts with consistency errors.

The right response is to separate platform restore success from application recovery success. Confirm snapshot time, application quiescing method, guest tools or agent involvement, database logs, restore target, rollback plan, and whether an application-aware backup is required. Communicate that the snapshot may provide a storage-level point in time, but the application still needs integrity validation.

## 7. Triage questions
1. What object was snapshotted: VM, vDisk, volume group, or protection entity?
2. What is the required restore point and business deadline?
3. Is the snapshot crash-consistent or application-consistent?
4. Are backups also available outside the cluster?
5. How much capacity do current snapshots consume?
6. Did the issue follow a patch, ransomware event, deletion, or failed migration?
7. Is replication involved, and what is the current RPO lag?
8. Is the restore being done in-place or to an isolated target?
9. Who validates the application after restore?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “A snapshot is a backup.” | “A snapshot can support recovery, but backup requires an independent recovery strategy.” |
| “The snapshot exists, so restore is guaranteed.” | “We must validate restore path, consistency, capacity, and application state.” |
| “Delete old snapshots to fix capacity.” | “Confirm ownership, retention, replication, compliance, and safe cleanup order first.” |
| “Snapshot restore fixes the incident.” | “Storage restore is one step; application validation and customer acceptance are still required.” |

## 9. Minimum to memorize
Snapshots are point-in-time recovery references. They are fast and efficient at creation, but consume capacity as data changes. They are not automatically independent backups and do not guarantee application consistency without the right workflow.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| Application consistency | Recovery state where the application can resume without logical corruption. |
| Backup | Independent recoverable copy governed by retention and recovery policy. |
| Crash consistency | State similar to a sudden power loss; OS may recover but application checks are needed. |
| Protection Domain | Nutanix construct for snapshot and replication workflows. |
| RPO | Recovery Point Objective; acceptable data-loss window. |
| RTO | Recovery Time Objective; acceptable restoration time. |
| Snapshot | Point-in-time representation of a protected object. |
| Snapshot retention | How long snapshot recovery points are kept. |
| Volume Group | Block storage object that may be protected or restored. |
| vDisk | Virtual disk associated with a VM. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Developer resources: https://www.nutanix.dev/
- Nutanix Bible: https://www.nutanixbible.com/
