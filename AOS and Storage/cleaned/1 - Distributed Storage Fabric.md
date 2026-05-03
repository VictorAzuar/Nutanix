# 1 - Distributed Storage Fabric

## 1. Purpose of this chapter
This chapter establishes the storage mental model for the whole module. A support leader must understand that Nutanix storage is not a traditional external array hidden behind hypervisor hosts. It is a distributed software-defined storage layer running across cluster nodes. This matters because most serious AOS cases combine symptoms from storage, compute, network, CVM health, capacity, and customer workload behavior.

## 2. Core definition
**Distributed Storage Fabric (DSF)** is the Nutanix AOS storage layer that pools local disks from cluster nodes and presents resilient shared storage to the hypervisor while distributing I/O, metadata, replication, and recovery work across the cluster.

## 3. Main explanation
In a conventional virtualization design, compute hosts depend on an external SAN or NAS. Nutanix changes the failure and troubleshooting model: each node contributes compute, local storage, and a Controller VM (CVM). The CVMs cooperate to form the storage fabric.

The hypervisor sees storage that behaves like shared storage. Internally, data is split, mapped, replicated, balanced, and served through a distributed data path. Reads should be local when possible; writes are protected according to the configured replication policy. When a disk or node fails, AOS serves data from surviving replicas and starts re-protection work.

For support management, DSF is the foundation for clear reasoning: latency is not automatically “the storage array”; capacity is not only raw terabytes; resiliency is not a binary promise; and a degraded cluster can be available while still carrying increased risk.

## 4. Key concepts
- **CVM**: the Nutanix storage/controller VM on each node.
- **Stargate**: the service commonly associated with storage I/O handling.
- **Metadata services**: maintain the map between logical data, replicas, placement, and cluster state.
- **Storage Pool**: distributed aggregation of physical disks.
- **Storage Container**: logical boundary where workloads and policies are typically organized.
- **Replication Factor**: the number of data copies maintained for resiliency.
- **Data locality**: serving VM reads from the local node when possible.
- **Re-protection**: restoring the expected redundancy after a failure.

## 5. Support relevance
DSF appears in cases involving VM latency, node failure, disk failure, capacity exhaustion, snapshot growth, rebalancing, rebuild activity, customer-facing performance complaints, and multi-vendor blame. A manager should lead with impact and evidence, not assumptions. The right posture is: define the symptom, isolate the layer, confirm cluster health, communicate the risk, and route deep component analysis to the correct specialist.

## 6. Escalation scenario
A customer reports that several database VMs became slow after a node failure. The cluster is online, but Prism shows resiliency warnings and active re-protection.

A strong support response separates **availability** from **performance**. Workloads may remain available because replicas are being served, while latency increases because data is remote, CVMs are under pressure, re-protection is active, or the storage network is carrying more traffic. The manager should confirm customer impact, affected VMs, RF level, failed component, re-protection progress, CVM health, capacity headroom, and next safe remediation step.

## 7. Triage questions
1. Which VMs, applications, and business processes are affected?
2. Is the symptom outage, latency, reduced throughput, alert-only, or failed writes?
3. Did the issue follow a disk failure, node failure, migration, upgrade, backup, or network change?
4. Is the cluster fully protected, degraded, or rebuilding?
5. Are all CVMs and Stargate services healthy?
6. Is I/O local or remote for the affected workloads?
7. Is capacity sufficient for rebuild and normal workload growth?
8. Are performance metrics consistent across Prism, hypervisor, guest OS, and application monitoring?
9. Are multiple vendors involved, and who owns each hypothesis?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “Nutanix eliminates storage complexity.” | “Nutanix changes the storage operating model; troubleshooting becomes distributed and cross-layer.” |
| “The cluster is resilient, so there is no risk.” | “The cluster may remain online while resiliency is degraded; we need to verify current protection and failure tolerance.” |
| “Storage is definitely the root cause.” | “Storage latency is a symptom; we need to correlate VM, hypervisor, CVM, disk, network, and workload evidence.” |
| “Adding nodes will fix performance.” | “Expansion helps only if the measured bottleneck is capacity, compute, or distributed storage resources that scale with nodes.” |

## 9. Minimum to memorize
DSF is Nutanix’s distributed storage layer. CVMs on each node cooperate to pool local disks, serve VM I/O, replicate data, preserve locality when possible, and self-heal after failures. In support escalations, always distinguish service availability, degraded resiliency, performance impact, and root cause.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| AOS | Nutanix software layer that provides distributed storage and platform services. |
| Container | Logical storage boundary built on top of the storage pool. |
| CVM | Controller VM; Nutanix infrastructure VM running storage and cluster services. |
| Data locality | Serving data near the VM’s current compute location to reduce latency. |
| DSF | Distributed Storage Fabric; Nutanix distributed storage architecture. |
| Extent | Internal logical unit used to organize data in AOS. |
| HCI | Hyperconverged Infrastructure; integrated compute, storage, virtualization, and management. |
| Prism | Nutanix management interface for health, alerts, capacity, and operations. |
| Re-protection | Background restoration of required redundancy after failure. |
| RF | Replication Factor; number of data copies maintained. |
| Stargate | AOS service associated with storage I/O handling. |
| Storage Pool | Aggregated physical storage across cluster nodes. |

## 11. External references (with all available links found on source file)
- Nutanix AOS / Distributed Storage Fabric reference: https://www.nutanix.com/products/aos
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
