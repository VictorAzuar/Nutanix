# 15 - Storage Latency Troubleshooting

## 1. Purpose of this chapter
This chapter explains how to reason about storage latency without jumping to root cause. Support managers need this because latency escalations are high-pressure, cross-layer, and often politically difficult: application, hypervisor, storage, network, and vendor teams may all blame each other.

## 2. Core definition
**Storage latency troubleshooting** is the structured process of isolating why I/O response time increased and determining whether the cause is in the guest OS, workload, hypervisor, CVM, AOS storage path, disks, network, background tasks, capacity, or cluster health.

## 3. Main explanation
Latency is a symptom, not a root cause. A VM can show slow application response for many reasons: database locks, guest disk queues, CPU pressure, remote I/O, CVM overload, active rebuild, saturated disks, network packet loss, backup jobs, snapshot activity, or application change.

A manager does not need to interpret every low-level counter, but must keep the investigation structured. Start with impact and scope, then segment the path:

```text
Application -> Guest OS -> VM/vDisk -> Hypervisor -> CVM/Stargate -> AOS storage -> Disk/tier -> Network -> Cluster state
```

Good escalation leadership prevents two common failures: treating every slow application as a storage defect, and treating every Prism latency metric as automatically customer-impacting.

## 4. Key concepts
- **Read latency vs write latency**: different causes and evidence.
- **IOPS vs throughput**: many small operations differ from large sequential transfers.
- **Outstanding I/O**: queued or in-flight operations.
- **Local vs remote reads**: data locality affects network dependency.
- **CVM/Stargate health**: key storage-path validation point.
- **Background activity**: rebuild, rebalancing, snapshots, backups, and replication can compete for resources.
- **Noisy neighbor**: one workload affects others through shared resources.

## 5. Support relevance
Storage latency cases are often P1/P2 because customers experience business degradation before root cause is known. The manager must establish severity, objective evidence, affected scope, recent changes, safe mitigations, and a communication cadence. Strong leadership means assigning hypotheses to owners and avoiding vendor-blame loops.

## 6. Escalation scenario
A customer says “Nutanix storage is slow” because a SQL Server workload changed from 5 ms to 80 ms write latency. Prism shows elevated write latency on one node during the backup window, and a rebuild is active after a disk failure.

The manager should separate the evidence: application symptom, VM/vDisk metrics, node concentration, backup timing, rebuild activity, CVM health, disk tier, and network counters. Immediate mitigation may involve pausing non-critical backup activity, validating rebuild progress, and protecting production workload priority while the technical team confirms whether latency is caused by rebuild contention, workload change, or another fault.

## 7. Triage questions
1. Is latency read, write, mixed, throughput, or application response time?
2. Which workloads are affected and which similar workloads are healthy?
3. Is the issue visible in application metrics, guest OS, hypervisor, and Prism?
4. Did it start after a change: backup, patch, migration, failover, node issue, or network change?
5. Are affected VMs concentrated on one host, node, container, disk tier, or network path?
6. Are all CVMs and Stargate services healthy?
7. Is rebuild, rebalancing, snapshot deletion, replication, or backup activity running?
8. Is capacity pressure or degraded resiliency present?
9. What safe mitigation can reduce impact while root cause analysis continues?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “Prism shows latency, so storage is the root cause.” | “Prism latency is evidence; correlate it with VM, guest, hypervisor, workload, and timeline data.” |
| “The application is slow, so Nutanix is slow.” | “Application latency can come from many layers; isolate the I/O path before assigning cause.” |
| “Latency is normal during rebuild.” | “Some impact may occur during rebuild, but we must measure severity and protect critical workloads.” |
| “Network is not relevant to storage.” | “In distributed storage, east-west network health can directly affect remote I/O and replication.” |

## 9. Minimum to memorize
Latency is a symptom. Segment the path from application to guest OS, VM, hypervisor, CVM, storage services, disks, network, and cluster state. Correlate metrics with timeline and scope before declaring root cause.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| CVM | Controller VM involved in Nutanix storage I/O. |
| Data locality | Serving data near the VM to reduce remote reads. |
| Disk queue | Pending I/O waiting to be served. |
| East-west traffic | Traffic between nodes inside the cluster. |
| I/O | Input/output operation such as read or write. |
| IOPS | Input/output operations per second. |
| Latency | Time required to complete an operation. |
| Noisy neighbor | Workload consuming shared resources and affecting others. |
| Outstanding I/O | Operations in flight or queued. |
| Rebuild | Recovery work that may consume storage resources. |
| Stargate | AOS data-path service involved in storage I/O. |
| Throughput | Amount of data transferred per second. |
| Write latency | Time required to complete write operations. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
