# 6 - Metadata Services

## 1. Purpose of this chapter
This chapter explains why metadata services matter in AOS. For a support manager, the goal is not to memorize internal schemas, but to understand that distributed storage depends on accurate state, placement, coordination, and background repair information.

## 2. Core definition
**Metadata services** are the AOS components that track where data lives, how it is protected, what cluster state exists, and what background work is required to keep storage consistent, balanced, and recoverable.

## 3. Main explanation
In distributed storage, data is split and spread across nodes and disks. AOS must know which vDisk maps to which extents, where replicas are stored, which data is under-protected, which services are healthy, and which repair or cleanup tasks are pending.

Commonly referenced Nutanix components include Cassandra for distributed metadata, Medusa as an access layer, Zookeeper/Zeus for configuration and coordination, Curator for background scanning and repair, and Stargate for I/O decisions that depend on this state.

Support managers should recognize metadata alerts as potentially high risk until impact is understood. Some metadata issues may be management-plane noise; others can affect data path, repair progress, upgrades, resiliency, or customer confidence.

## 4. Key concepts
- **Cassandra**: distributed metadata store.
- **Medusa**: metadata access layer commonly associated with Cassandra.
- **Zookeeper / Zeus**: configuration and coordination services.
- **Curator**: background scanning, cleanup, balancing, and repair work.
- **Stargate**: uses metadata context to serve storage I/O.
- **Quorum / coordination**: distributed systems need consistent agreement on state.
- **Under-replication**: data has fewer replicas than expected.

## 5. Support relevance
Metadata service issues can appear during upgrades, node failures, disk failures, rebuild delays, inconsistent health alerts, cluster-service instability, or failed management operations. The manager should avoid drowning the customer in component names. Communicate impact, risk, mitigation, and next action.

## 6. Escalation scenario
Prism reports Cassandra or metadata-service errors after a node outage. VMs appear online, but re-protection progress is unclear and the customer is concerned about data integrity.

The manager should establish whether the issue affects foreground I/O, management visibility, or background repair. Technical owners should validate CVM health, metadata service health, Stargate health, cluster resiliency, NCC results, and recent failure timeline. Customer communication should focus on whether data remains available, whether protection is degraded, and what is being done to restore a healthy baseline.

## 7. Triage questions
1. Is customer I/O affected, or is the alert management-plane only?
2. Which metadata or coordination service is reporting unhealthy?
3. Are all CVMs reachable and participating normally?
4. Is Stargate healthy on all nodes?
5. Is any data under-replicated or rebuild work stalled?
6. Did the issue follow a node outage, disk failure, upgrade, or network event?
7. Are NCC or Prism health checks showing broader cluster risk?
8. Are logs and diagnostics sufficient for SRE or engineering escalation?
9. What precise customer-facing risk statement is accurate now?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “It is only metadata, so it is not serious.” | “Metadata health can affect placement, repair, resiliency, and operations; validate impact first.” |
| “Cassandra is down, so data is lost.” | “Do not infer data loss from a component alert; verify I/O, redundancy, and cluster state.” |
| “The customer does not need details.” | “The customer needs clear impact, risk, next steps, and update cadence without unnecessary internal noise.” |
| “Engineering owns this now.” | “Engineering may own deep diagnosis, but support still owns impact management and customer communication.” |

## 9. Minimum to memorize
Metadata services maintain the map and state that distributed storage needs. Cassandra, Medusa, Zookeeper/Zeus, Curator, and Stargate are names to recognize. In escalations, determine whether metadata symptoms affect I/O, resiliency, repair, upgrades, or only visibility.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| Cassandra | Distributed metadata store used by Nutanix AOS. |
| Curator | Background framework for scans, cleanup, balancing, and repair. |
| Data path | Foreground route used to serve workload I/O. |
| Medusa | Metadata access layer associated with Cassandra. |
| Metadata | Information describing data placement, protection, and state. |
| NCC | Nutanix Cluster Check; health validation tool. |
| Quorum | Minimum agreement required among distributed components. |
| Stargate | AOS service handling storage I/O. |
| Under-replicated | Data has fewer copies than required by policy. |
| Zeus | Access library/interface associated with Zookeeper configuration. |
| Zookeeper | Coordination/configuration service in distributed systems. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
