# 12 - Rebuild and Resync

## 1. Purpose of this chapter
This chapter explains rebuild and resync as the operational recovery process after failure, maintenance, or topology change. Support managers need this because the customer may be online while the platform is still restoring protection and consuming cluster resources.

## 2. Core definition
**Rebuild / resync** is the AOS process of restoring expected data placement and resiliency by recreating missing or stale data copies across healthy disks, nodes, or failure domains.

## 3. Main explanation
When a disk, node, or other failure domain becomes unavailable, some data may lose one of its expected copies. AOS identifies the affected data and rebuilds missing redundancy elsewhere in the cluster. Resync can also occur after maintenance, temporary outage, migration, or topology changes.

This work is distributed, but it is not cost-free. It can consume disk I/O, CPU, CVM resources, network bandwidth, and capacity. A well-sized cluster may absorb this with little visible impact. A busy or nearly full cluster may show latency, slow progress, or increased customer risk.

## 4. Key concepts
- **Re-protection**: restoring required replica count.
- **Under-replicated data**: data that currently lacks the expected redundancy.
- **Rebuild capacity**: free capacity needed to place replacement copies.
- **Rebalancing**: redistributing data/load after changes.
- **Planned vs unplanned event**: maintenance should be distinguished from failure.
- **Progress monitoring**: Prism/NCC/logs help track repair state.

## 5. Support relevance
Rebuild cases require careful risk communication. The cluster may be serving workloads, but until re-protection completes, it may have reduced failure tolerance. A manager should control unsafe actions, communicate the degraded-risk window, and ensure technical teams monitor progress and blockers.

## 6. Escalation scenario
A disk fails during a high-write backup window. Rebuild starts, but progress is slow and database latency increases.

The manager should coordinate two workstreams: recovery progress and workload impact. Validate remaining hardware health, RF state, capacity headroom, CVM/network load, active backups, and whether throttling or pausing non-critical jobs is safe. The customer update should explain that rebuild is restoring protection and may compete with production I/O depending on load.

## 7. Triage questions
1. What triggered rebuild or resync?
2. Which failure domain is affected: disk, node, block, rack, or site?
3. Is data under-replicated or fully protected?
4. Is rebuild progressing, stalled, or repeatedly restarting?
5. Is there enough capacity for re-protection?
6. Are backups, snapshots, replication, or batch jobs competing for I/O?
7. Are customers seeing latency, failed writes, or only alerts?
8. Are there additional disk, CVM, node, or network errors?
9. What operations must be paused until the cluster returns to green?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “Rebuild is automatic, so we can ignore it.” | “Rebuild is automatic, but progress, capacity, and workload impact must be monitored.” |
| “The cluster is online, so the incident is over.” | “Service may be online while resiliency is still degraded.” |
| “Replace more hardware immediately.” | “Follow the validated sequence; additional disruption during rebuild can increase risk.” |
| “Slow rebuild means product defect.” | “Slow rebuild may reflect capacity, workload, hardware, network, or throttling context.” |

## 9. Minimum to memorize
Rebuild restores protection after a failure. It consumes resources and depends on capacity, cluster health, and workload load. Support must communicate the difference between available service and restored resiliency.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| Curator | Background framework involved in scans, repair, and balancing. |
| Degraded state | Running with reduced redundancy or increased risk. |
| Rebalance | Redistributing data or load across the cluster. |
| Rebuild | Recreating missing redundancy after failure. |
| Rebuild capacity | Free capacity needed to restore protection. |
| Re-protection | Restoring the expected replica count. |
| Resync | Bringing stale or interrupted data back to expected state. |
| RF | Replication Factor; configured number of data copies. |
| Under-replicated | Data currently has fewer copies than required. |
| Workload contention | Production I/O competing with background repair work. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
