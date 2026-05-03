# 2 - Controller VM (CVM)

## 1. Purpose of this chapter
This chapter explains the CVM as the operational center of Nutanix storage. For support leadership, CVM fluency is essential because many severe cases involving VM latency, failed upgrades, degraded resiliency, and storage path changes require fast recognition of CVM impact and risk.

## 2. Core definition
A **Controller VM (CVM)** is the Nutanix infrastructure VM running on each node that hosts core AOS services and participates in the distributed storage fabric. It acts as the local storage controller while cooperating with other CVMs across the cluster.

## 3. Main explanation
Nutanix replaces the external storage-controller model with software controllers distributed across nodes. Each host runs a hypervisor and a CVM. The hypervisor sends storage I/O through the local CVM, and CVMs coordinate data placement, replication, metadata, snapshots, efficiency services, and recovery behavior.

A CVM is not a normal workload VM. It is a privileged infrastructure component. When it is healthy, the local path can serve I/O efficiently. If a CVM or important service such as Stargate is unavailable, I/O may be redirected through another CVM. That can preserve availability, but it changes the data path and may increase latency or network dependency.

## 4. Key concepts
- **One CVM per node**: storage control is distributed, not centralized.
- **Local I/O path**: normal path where a VM uses the CVM on the same host.
- **Remote I/O path**: fallback path through another CVM during local CVM/service issues.
- **CVM networking**: critical for storage, metadata, replication, and cluster communication.
- **CVM resources**: CPU and memory should not be changed casually.
- **Stargate**: key service associated with storage I/O.
- **Prism visibility**: CVM issues often appear as node, service, storage, or performance alerts.

## 5. Support relevance
A CVM problem can look like a storage problem, a network problem, a host problem, or a VM problem. A support manager should ensure the team validates impact, affected node(s), service state, I/O path, network health, recent changes, and whether further customer actions could increase risk.

## 6. Escalation scenario
After planned maintenance, one CVM does not return to a healthy state. VMs remain online, but the host is using a remote storage path and customers report higher latency.

The right response is not panic and not casual restarts. Confirm whether workloads are affected, whether resiliency is degraded, whether other CVMs are healthy, whether the storage network can carry the redirected traffic, and whether maintenance must be paused. Communicate that availability may be preserved, but the platform is not in its preferred operating state until the local CVM path is restored.

## 7. Triage questions
1. Are all CVMs powered on, reachable, and reporting healthy services?
2. Is the issue isolated to one host or spread across the cluster?
3. Is Stargate healthy on the affected CVM?
4. Are VMs on one host showing higher latency than others?
5. Is I/O being served locally or remotely?
6. Did a restart, upgrade, IP change, network segmentation change, or host maintenance occur?
7. Are there packet loss, MTU, VLAN, switch, or NIC errors between CVMs?
8. Has anyone changed CVM CPU, memory, disk, or networking settings?
9. Is the customer asking for immediate mitigation, RCA, or both?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “The CVM is just another VM.” | “The CVM is an infrastructure VM in the storage and control path.” |
| “Remote I/O means everything is fine.” | “Remote I/O can preserve service, but it is a degraded path that may affect latency and risk.” |
| “Restart the CVMs and see what happens.” | “Validate cluster state and follow a controlled recovery path before disruptive actions.” |
| “Increase CVM memory to fix latency.” | “CVM resource changes must follow Nutanix guidance or Support direction.” |

## 9. Minimum to memorize
A CVM runs on every Nutanix node and provides local storage services while participating in the distributed fabric. If a CVM or Stargate fails, the cluster may redirect I/O through another CVM; that protects availability but may degrade performance and increases the need for careful escalation control.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| AOS | Nutanix software running core platform and storage services. |
| Backplane IP | Internal CVM communication address used for cluster traffic in some designs. |
| CVM | Controller VM providing storage and cluster services on each node. |
| Data path | Route used by workload I/O from VM to storage. |
| DSF | Distributed Storage Fabric; distributed Nutanix storage layer. |
| Local I/O | I/O handled by the CVM on the same node as the VM. |
| PCI passthrough | Hardware assignment method used to give the CVM direct access to storage devices. |
| Prism | Nutanix UI/API for operational visibility. |
| Remote I/O | I/O served by another CVM over the network. |
| Stargate | AOS data-path service involved in storage I/O. |
| VMDirectPath | VMware mechanism for passing hardware devices directly to a VM. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
