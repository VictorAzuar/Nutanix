# 3 - Storage Pool

## 1. Purpose of this chapter
This chapter explains the storage pool as the physical capacity foundation of AOS. Support managers need this distinction because capacity escalations often become confused when customers mix raw capacity, usable capacity, container capacity, resilient capacity, and snapshot consumption.

## 2. Core definition
A **Storage Pool** is the Nutanix AOS aggregation of physical storage devices across cluster nodes. Logical storage objects such as containers, vDisks, and volume groups are built on top of this pool.

## 3. Main explanation
The storage pool is where the cluster-wide physical storage foundation lives. It is not usually what application teams consume directly. Workloads normally consume storage through storage containers, datastores, vDisks, volume groups, files, or higher-level services.

The practical hierarchy is:

```text
Physical disks across nodes -> Storage Pool -> Storage Containers -> vDisks / Volume Groups / VMs
```

In support, storage pool analysis is used to understand physical capacity, disk contribution, resiliency, rebuild ability, node expansion, and cluster-wide storage pressure. A customer may focus on “we bought X TB,” but the support conversation must align on raw, usable, logical, physical used, reserved, and resilient capacity.

## 4. Key concepts
- **Raw capacity**: total physical disk capacity before platform overhead.
- **Usable capacity**: practical capacity after metadata, resiliency, and operational overhead.
- **Logical usage**: what workloads think they have consumed.
- **Physical usage**: actual consumed storage after efficiency features.
- **Resilient capacity**: capacity state that preserves recovery ability.
- **Rebuild reserve**: capacity needed to restore protection after failure.
- **Storage container**: logical layer where workloads and policies are usually managed.

## 5. Support relevance
Storage pool cases are rarely just “space” cases. They can involve failed disks, rebuild limitations, snapshot growth, backup bursts, poor cleanup, workload expansion, capacity-efficiency assumptions, and customer risk communication. The manager should force a clean distinction between capacity warning, production impact, degraded protection, and imminent outage.

## 6. Escalation scenario
A customer reports the cluster is at 92% used and asks whether production can continue safely. At the same time, a disk has failed and re-protection is slow.

The support manager should treat this as a resiliency risk, not only a capacity complaint. Confirm whether writes are failing, whether VMs are affected, whether re-protection can complete, what capacity is consumed by snapshots or clones, whether cleanup is safe, and whether expansion or workload reduction is required.

## 7. Triage questions
1. What are raw, usable, physical used, logical used, and free capacity?
2. Is the cluster close to a resilient capacity threshold?
3. Is rebuild capacity reservation enabled or relevant?
4. Which containers or workloads are consuming the most space?
5. Are snapshots, clones, backups, restores, or migrations causing growth?
6. Is re-protection, rebalancing, or cleanup active?
7. Are there failed disks, degraded nodes, or CVM alerts?
8. Is capacity pressure affecting latency, write availability, or only alerts?
9. What immediate actions are safe versus risky?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “There is free space, so we are safe.” | “We need to validate free capacity, resilient capacity, and rebuild ability.” |
| “The storage pool is the datastore.” | “The pool is the physical aggregation layer; containers or datastores are logical presentation layers.” |
| “Deduplication will fix the full cluster.” | “Data efficiency may help only if supported, suitable, and timely for the workload.” |
| “Delete snapshots until space is free.” | “Identify snapshot ownership, retention, consistency requirements, and safe deletion order first.” |

## 9. Minimum to memorize
Storage Pool means distributed physical capacity. Containers sit on top of it. Capacity escalations must distinguish raw, usable, logical, physical, reserved, and resilient capacity. A full pool can become a data-protection and availability risk, not just an accounting issue.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| Capacity headroom | Free operational room for growth, rebuilds, snapshots, and recovery. |
| Container | Logical storage object created on the storage pool. |
| Logical usage | Capacity presented or consumed logically by workloads. |
| Physical usage | Actual backend storage consumed after overhead and efficiency. |
| Rebuild reserve | Capacity set aside or required for recovery after failures. |
| Resilient capacity | Capacity state that preserves configured failure tolerance. |
| RF | Replication Factor; number of copies maintained. |
| Snapshot | Point-in-time reference that can consume space as data changes. |
| Storage Pool | Aggregated physical storage across Nutanix nodes. |
| Usable capacity | Capacity available after system and resiliency overhead. |
| vDisk | Virtual disk object stored on AOS. |
| Volume Group | Nutanix block storage object, often used for direct block access. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
