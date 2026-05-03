# 14 - Node Failure

## 1. Purpose of this chapter
This chapter explains node failure as a combined compute, storage, network, and resiliency event. Support managers need this because node incidents often create high customer anxiety even when the cluster continues serving some workloads.

## 2. Core definition
A **node failure** is the loss or severe degradation of a Nutanix physical server or a critical node component such as the hypervisor, CVM, disks, or network path. In HCI, a node contributes compute, memory, storage, services, and failure-domain capacity.

## 3. Main explanation
When a node fails, the impact can appear across several layers at once. VMs running on that node may stop and require HA restart. Data replicas stored on that node must be served from surviving copies. AOS may begin re-protection to restore redundancy. Cluster capacity and performance may drop because one node’s resources are no longer available.

A node failure is therefore not just “one server down.” It changes workload placement, available compute, available storage, failure tolerance, rebuild pressure, and customer risk. The support manager must coordinate platform recovery while keeping customer communication precise: what is down, what is degraded, what remains protected, and what should wait until the cluster returns to a healthy baseline.

## 4. Key concepts
- **Compute impact**: VMs on the node may need HA restart or manual recovery.
- **Storage impact**: replicas on the node are unavailable until node recovery or re-protection.
- **CVM impact**: the node’s local storage controller and services are affected.
- **Failure domain**: node-level loss affects protection calculations.
- **HA**: hypervisor-level workload restart capability.
- **Re-protection**: AOS restoring data redundancy elsewhere.

## 5. Support relevance
Node failures often become executive escalations because they combine outage symptoms, degraded resiliency, hardware replacement, cluster-risk questions, and possible multi-vendor ownership. A manager must prevent unsafe parallel actions and maintain a disciplined incident bridge.

## 6. Escalation scenario
A four-node RF2 cluster loses one node during production hours. HA restarts some VMs on remaining nodes, but capacity and CPU headroom are now tight. Re-protection begins and application teams report intermittent latency.

The support response should confirm VM recovery, cluster protection state, resource headroom, rebuild progress, remaining failure tolerance, and whether non-critical workloads should be paused. The customer needs a clear explanation that service may be restored before the platform is fully protected or fully performant.

## 7. Triage questions
1. Did the entire node fail or only the hypervisor, CVM, disk, NIC, or power path?
2. Which VMs were running on the failed node and did HA restart them?
3. Is the cluster RF2 or RF3 and what failure tolerance remains?
4. Is re-protection active and progressing?
5. Are remaining nodes overloaded on CPU, memory, storage, or network?
6. Is the failed node visible in Prism or hardware-management tooling?
7. Were there recent upgrades, firmware changes, network changes, or power events?
8. Are there additional hardware alerts on surviving nodes?
9. What customer operations must be paused until resiliency is restored?

## 8. What to avoid
| Weak statement | Better alternative |
|---|---|
| “Only one node failed, so the cluster is fine.” | “A node failure may preserve service but reduces resources and protection until recovery completes.” |
| “HA solved the incident.” | “HA may restart VMs, but storage resiliency and performance still need validation.” |
| “Add a node and performance will recover.” | “Validate whether the bottleneck is compute, storage, network, rebuild, or workload placement.” |
| “Keep doing maintenance on other nodes.” | “Pause additional disruptive work until the cluster returns to a safe baseline.” |

## 9. Minimum to memorize
A node is both compute and storage in Nutanix. Node failure can affect VM availability, resource headroom, data resiliency, rebuild activity, and customer risk. Confirm HA, CVM/storage state, RF, capacity, and remaining failure tolerance before declaring recovery.

## 10. Glossary: keywords and acronyms
| Keyword / Acronym | Meaning |
|---|---|
| CVM | Controller VM running Nutanix services on a node. |
| Failure domain | Infrastructure unit whose loss must be tolerated by design. |
| HA | High Availability; workload restart capability after host failure. |
| Hypervisor | Software layer running VMs, such as AHV or ESXi. |
| Node | Physical Nutanix server contributing compute, storage, and services. |
| Re-protection | Restoring required data redundancy after failure. |
| RF2 | Two-copy data protection model. |
| RF3 | Three-copy data protection model. |
| Resource headroom | Remaining CPU, memory, storage, and network capacity. |
| Workload placement | Distribution of VMs across available hosts. |

## 11. External references (with all available links found on source file)
- Nutanix Portal documentation: https://portal.nutanix.com/page/documents
- Nutanix Bible: https://www.nutanixbible.com/
