# [AOS / Storage] — Node Failure

## 1. Short definition

A **node failure** in Nutanix means that one physical server in the cluster, or a critical component of that server such as the **hypervisor**, **CVM**, disks, or network path, becomes unavailable. In Nutanix HCI, a node is not just compute: it contributes **CPU, memory, storage capacity, storage performance, and AOS services** to the distributed cluster.

The key interview message:

> In Nutanix AOS, node failure is handled through distributed storage, replication factor, checksums, self-healing, HA restart mechanisms, and background re-protection. The support challenge is not only restoring the failed node, but protecting customer availability, understanding degraded resiliency, managing risk during rebuild, and communicating clearly during the escalation.

Nutanix documentation describes a node as composed of a **physical host and a Controller VM**, and states that either component can fail while the cluster uses built-in resiliency mechanisms to maintain operations. ([Nutanix Portal][1])

---

## 2. Clear explanation

In a traditional architecture, if a storage array fails, many hosts may lose access to shared storage. Nutanix is different because it uses a **distributed storage fabric**. Each node contributes local disks, but AOS presents storage to workloads as a unified storage layer across the cluster.

When a node fails, several things may happen at the same time:

1. **Compute impact**
   VMs running on that node may stop or become unavailable. If HA is enabled and enough resources exist, the hypervisor can restart those VMs on healthy nodes.

2. **Storage impact**
   Data that had one copy on the failed node must still be available from other replicas. AOS uses **replication factor** and **checksums** to maintain availability and integrity. Nutanix documentation states that after a node or disk failure, data is re-replicated across healthy nodes to restore the configured replication factor; this process is called **re-protection**. ([Nutanix Portal][2])

3. **Resiliency state changes**
   The cluster may move from a healthy state to a degraded state. For example, an RF2 cluster can normally tolerate one copy being unavailable, but while degraded it may have less tolerance for an additional failure.

4. **Background recovery starts**
   Services such as **Curator** identify under-replicated or inconsistent data and coordinate repair/rebalancing. Nutanix documentation describes Curator as a background process responsible for file-system operations including tiering, rebalancing, and fixing redundancy issues. ([Nutanix Portal][3])

5. **Operational risk increases**
   Even if customers see no immediate outage, the environment may be at higher risk until re-protection completes and cluster health returns to green.

A good verbal explanation for interview purposes:

> “In Nutanix, a node failure affects both compute and storage because the node contributes resources to the distributed platform. AOS keeps data available through replication and checksums. If a node disappears, workloads may be restarted elsewhere by HA, while AOS serves data from surviving replicas and begins re-protection in the background. From a support management perspective, the priority is to confirm customer impact, check data resiliency, avoid unsafe operations, coordinate hardware or platform recovery, and communicate the degraded-risk window clearly.”

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Worldwide Support Manager**, node failure matters because it is a classic enterprise escalation scenario: high customer anxiety, possible production impact, hardware/software/network ambiguity, and cross-functional coordination.

You are not expected to behave like a Senior SRE IC writing low-level AOS internals from memory. You are expected to demonstrate that you can:

* Understand the technical risk.
* Ask the right triage questions.
* Protect customer data and availability.
* Coordinate Support, SRE, Engineering, hardware logistics, and Customer Success.
* Communicate status, next actions, risk, and ETA without overpromising.
* Drive the case toward restoration and post-incident learning.

Nutanix clusters are designed around fault tolerance domains such as node, block, and rack; Nutanix documentation defines fault domains as entities that can fail without impacting cluster operations, depending on configuration. ([Nutanix Portal][4])

The managerial angle is this:

> The customer does not care whether the root cause is AHV, ESXi, a CVM, a disk, a top-of-rack switch, a power event, or a firmware issue. They care whether production is safe, whether data is protected, whether performance is degraded, and whether Nutanix Support is in control.

---

## 4. Key concepts

### Node

A **node** is a physical server participating in a Nutanix cluster. It contributes compute and storage resources. In Nutanix terminology, a node includes the physical host and the **Controller VM**. ([Nutanix Portal][1])

### CVM — Controller VM

The **Controller VM** runs Nutanix services on each node and is central to local storage I/O. If a CVM fails, I/O may temporarily be served through another CVM, but there can be a short disruption or hang while paths are restored. Nutanix documentation states that AOS uses replication factor and checksums for data redundancy and may trigger re-protection when a CVM goes down. ([Nutanix Portal][2])

### AOS distributed storage

AOS Storage is Nutanix’s scale-out storage layer. The Nutanix Bible describes AOS storage as appearing to the hypervisor like centralized storage, while I/O is handled locally when possible for performance. ([NutanixBible.com][5])

### Replication Factor — RF2 / RF3

Replication Factor defines how many copies of data are maintained.

* **RF2**: two copies of data.
* **RF3**: three copies of data.

In RF2, the cluster is generally designed to tolerate one copy failure. In RF3, it can tolerate more failure scenarios, depending on cluster size, placement, and fault domain configuration.

Critical interview nuance:

> RF does not mean “any failure is harmless.” It means the cluster has a defined failure tolerance under supported conditions. During re-protection or when multiple failures occur, risk increases.

### Re-protection

**Re-protection** is the process of restoring the desired replication level after data becomes under-replicated because of a disk, CVM, or node failure. Nutanix documentation explicitly describes re-replication after node or disk failure as re-protection. ([Nutanix Portal][2])

### Data Resiliency Status

This is a key operational indicator. Before planned node shutdown, Nutanix documentation advises checking that the cluster can tolerate a single-node failure and verifying that **Data Resiliency Status** is green/OK. ([Nutanix Portal][6])

In an interview, mention this as one of your first checks.

### Stargate

**Stargate** is the data I/O service in the Nutanix CVM. In practical terms, it handles storage I/O paths between workloads, hypervisors, and the distributed storage fabric. Nutanix component documentation describes Stargate as a service running on every node to serve localized I/O. ([Nutanix Community][7])

### Curator

**Curator** is the background service responsible for scanning metadata, identifying cleanup/optimization/repair tasks, and coordinating rebalancing or redundancy repair. Nutanix documentation describes Curator as scanning metadata, identifying cleanup and optimization tasks, and using MapReduce-style distributed processing. ([Nutanix Portal][8])

### Cassandra / Medusa / Zookeeper / Zeus

These are internal distributed-system services:

* **Cassandra**: distributed metadata store.
* **Medusa**: interface to Cassandra.
* **Zookeeper**: cluster configuration manager.
* **Zeus**: interface to Zookeeper.

The Nutanix Bible describes Cassandra as the distributed metadata store and Zookeeper as storing cluster configuration, with leadership election behavior. ([NutanixBible.com][9])

For your role, you do not need to debug these like an AOS engineer, but you should recognize the names and know that they are part of cluster control-plane health.

### Block and rack awareness

A Nutanix **block** can contain multiple nodes sharing physical components such as power supplies, backplane, fans, and front panel components. Nutanix documentation says block fault tolerance places redundant data and metadata copies on nodes in different blocks. ([Nutanix Portal][10])

This matters because not all “node failures” are isolated. A power/backplane issue can affect multiple nodes in the same block.

### Maintenance mode

For planned work, a node should be placed into maintenance mode so workloads can be evacuated safely. Nutanix AHV documentation notes that some VMs, such as those with GPU, CPU passthrough, PCI passthrough, or host affinity policies, may not migrate automatically and require special handling. ([Nutanix Portal][11])

---

## 5. How it appears in a real escalation

### Scenario

A customer opens a Sev1/P1 case:

> “One Nutanix node went down in our production cluster. Several VMs restarted, performance is degraded, Prism shows resiliency warnings, and the customer is worried about data loss.”

### What may be happening technically

Possible causes include:

* Physical server failure.
* Power supply or block-level hardware issue.
* CVM down or unresponsive.
* Hypervisor host failure.
* Network isolation between CVMs.
* Disk or controller failure causing node instability.
* Firmware/driver issue.
* Recent upgrade or maintenance error.
* Resource exhaustion.
* Multiple failures causing reduced fault tolerance.

### What the customer may observe

* VMs restarted on other hosts.
* Some VMs temporarily unavailable.
* Increased latency.
* Prism alerts.
* Data resiliency status degraded.
* Hardware alerts.
* CVM or host unreachable.
* HA failover events.
* Support bundle requested.
* Concern about whether another node can fail safely.

### Your escalation-management priorities

1. **Confirm impact**

   * Which applications are affected?
   * Are workloads down, degraded, or recovered?
   * Is there data unavailability or only reduced resiliency?

2. **Stabilize**

   * Avoid unnecessary reboots.
   * Avoid shutting down another node.
   * Confirm no ongoing planned maintenance.
   * Verify cluster health and resiliency.

3. **Assess risk**

   * RF2 or RF3?
   * Current Data Resiliency Status?
   * Any additional disk, node, or network failures?
   * Is re-protection running?
   * Is capacity sufficient for re-protection?

4. **Coordinate**

   * Support engineer / SRE.
   * Hardware replacement team.
   * Engineering if suspected product defect.
   * Customer success/account team for executive communication.
   * Customer’s VMware/network/storage/application teams.

5. **Communicate**

   * Current state.
   * Customer impact.
   * Risk window.
   * Next technical action.
   * What not to do.
   * ETA ranges only when defensible.

Example customer-facing language:

> “At this stage, the cluster appears to be operating in a degraded resiliency state rather than a confirmed data-loss state. Our immediate priority is to validate data resiliency, confirm whether re-protection is progressing, and prevent any additional risky operations while we isolate whether the failure is hardware, CVM, hypervisor, or network related.”

---

## 6. Triage questions I should ask

### Customer impact

1. Which business services are affected?
2. Are VMs down, restarted, slow, or only showing alerts?
3. Is this production, DR, test, or mixed workload?
4. Is there a contractual SLA or executive visibility?
5. When did the issue start, and what changed before it?

### Cluster context

6. How many nodes are in the cluster?
7. Is the cluster RF2 or RF3?
8. What does **Data Resiliency Status** show?
9. Is the failed component a full node, host, CVM, disk, NIC, or network path?
10. Are there other active alerts?

### Workload / HA

11. Were affected VMs restarted by HA?
12. Are any VMs pinned by affinity rules?
13. Are there passthrough devices, GPUs, or non-migratable workloads?
14. Is there enough compute capacity on remaining nodes?

### Storage health

15. Is re-protection running?
16. Are there under-replicated extents?
17. Is cluster capacity sufficient?
18. Are there disk failures in addition to the node failure?
19. Are there latency spikes on read/write I/O?

### Network / platform

20. Are CVMs mutually reachable?
21. Any top-of-rack switch issue?
22. Any VLAN, MTU, LACP, bond, or NIC failure?
23. Any recent AOS, AHV, ESXi, firmware, or driver update?
24. Was there planned maintenance?

### Escalation control

25. What actions has the customer already taken?
26. Has anyone rebooted hosts, CVMs, or switches?
27. Is there an active maintenance window?
28. Do we need a live bridge?
29. Who is the customer decision-maker?
30. What is the next safest action?

---

## 7. Likely interview questions

### Manager / leadership interview

1. “How would you manage a Sev1 escalation where a customer has a node failure?”
2. “How do you balance technical investigation with customer communication?”
3. “How would you prevent an engineer from taking risky actions during a degraded state?”
4. “How do you define success in this type of escalation?”
5. “How would you communicate uncertainty to an enterprise customer?”
6. “How would you handle a customer demanding an RCA before service is restored?”
7. “How do you coordinate Support, SRE, Engineering, and Account teams?”
8. “What KPIs would you track after repeated node-failure escalations?”
9. “How do you coach engineers after a high-pressure incident?”
10. “What would you include in the post-incident review?”

### Technical / SRE interview

1. “What happens in Nutanix when a node fails?”
2. “What is the role of the CVM during node failure?”
3. “What is re-protection?”
4. “What is the difference between RF2 and RF3?”
5. “What is Data Resiliency Status?”
6. “How does HA relate to AOS storage resiliency?”
7. “What Nutanix services would you associate with storage I/O and metadata?”
8. “What are Stargate and Curator?”
9. “What risks exist during re-protection?”
10. “How would you distinguish between host failure, CVM failure, and network isolation?”

### Advanced / panel interview

1. “A customer lost one node in an RF2 cluster and wants to power off another node for maintenance. What do you say?”
2. “A node failed and performance degraded, but no VMs are down. How do you manage the escalation?”
3. “The customer has VMware on Nutanix. What teams or layers do you involve?”
4. “The cluster is showing degraded resiliency after hardware replacement. What next?”
5. “How do you communicate risk when the system is still serving I/O?”
6. “How would you build a runbook for node-failure triage?”
7. “How do you ensure global consistency across support teams handling these cases?”

---

## 8. Model answers in English

### Q1. What happens when a Nutanix node fails?

**Model answer:**

> “In Nutanix, a node contributes both compute and storage resources. If a node fails, the immediate compute impact is that VMs running on that host may need to be restarted on other hosts by HA, assuming HA is configured and sufficient resources are available. On the storage side, AOS continues serving data from surviving replicas based on the configured replication factor. The cluster then starts re-protection, which means rebuilding or re-replicating data across healthy nodes to restore the desired resiliency level. As a support manager, I would focus on customer impact, data resiliency status, whether re-protection is progressing, and preventing any additional risky actions while the cluster is degraded.”

### Q2. What is re-protection?

**Model answer:**

> “Re-protection is the background process by which AOS restores the configured replication factor after a failure. For example, if a node or disk fails and some data becomes under-replicated, AOS creates new copies on healthy nodes so the cluster returns to its expected fault tolerance. Operationally, re-protection is important because the customer may be online, but the cluster is still in a higher-risk state until re-protection completes.”

### Q3. What is the difference between HA and storage resiliency?

**Model answer:**

> “HA and storage resiliency solve different parts of the failure. HA is about restarting or recovering VM compute on another host after the original host fails. Storage resiliency is about ensuring the VM data remains available and protected through replication across the Nutanix cluster. In an escalation, I would check both: are workloads running, and is the storage layer still resilient?”

### Q4. What would you do first in a node-failure escalation?

**Model answer:**

> “First, I would establish impact and safety. I would confirm whether production workloads are down or degraded, whether HA restarted the VMs, and whether the customer is still serving I/O. In parallel, I would ask the technical team to check Prism alerts, Data Resiliency Status, the failed component, RF level, active hardware alerts, and whether there are additional failures. I would also make sure the customer avoids rebooting or shutting down additional nodes until we confirm the cluster can tolerate it.”

### Q5. A customer in RF2 lost one node and wants to shut down another node. What do you say?

**Model answer:**

> “I would advise them not to proceed until we validate cluster resiliency. In RF2, losing one node may already place the cluster in a degraded state, so shutting down another node could create a much higher risk of data unavailability. I would ask the engineer to verify Data Resiliency Status, active alerts, re-protection progress, and Nutanix supportability guidance before approving any further maintenance.”

### Q6. How would you communicate with an executive customer during the escalation?

**Model answer:**

> “I would separate confirmed facts from ongoing investigation. For example: ‘We have confirmed one node is unavailable. Workloads are currently running with degraded resiliency. We are validating whether re-protection is progressing and whether there are any secondary failures. The immediate instruction is not to perform additional node maintenance or reboots until we confirm the cluster is safe.’ That keeps the customer informed without speculating.”

### Q7. What are Stargate and Curator?

**Model answer:**

> “At a high level, Stargate is associated with storage I/O handling in the Nutanix CVM, while Curator is a background service responsible for scanning metadata and coordinating cleanup, rebalancing, and redundancy repair tasks. I would not position myself as an AOS internals engineer, but I know these services are important when investigating storage health, I/O behavior, and re-protection.”

### Q8. How does this connect to your current support-management experience?

**Model answer:**

> “The technology is different, but the incident pattern is familiar: production impact, degraded redundancy, customer anxiety, cross-team troubleshooting, and the need for disciplined communication. In my current role managing 24/7 enterprise support, I already work with incident severity, SLA, MTTR, monitoring, escalation paths, and post-incident reviews. For Nutanix, I would apply the same operational discipline while building deeper expertise in AOS, Prism, AHV, and distributed storage.”

---

## 9. Connection with my experience

Your strongest positioning is:

> “I am not interviewing as the deepest AOS storage engineer in the room. I am interviewing as a technical escalation manager who can understand the architecture, ask the right questions, coordinate experts, protect the customer, and drive resolution.”

Connect it to your Harmonic experience like this:

### Incident management

You already understand:

* Severity classification.
* War rooms / bridges.
* Ownership.
* Update cadence.
* Escalation paths.
* SLA and MTTR pressure.
* Customer-facing communication.

Translate that into Nutanix:

> “A node failure is not just a technical event; it is an availability-risk event. My job is to keep the technical investigation structured while ensuring the customer understands impact, risk, and next actions.”

### Cloud / SaaS operations

Your SaaS/cloud background maps well to:

* Distributed systems.
* Redundancy.
* Failover.
* Monitoring.
* Capacity risk.
* Degraded states.
* Post-incident learning.

Useful phrasing:

> “In cloud operations, I am used to distinguishing between service down, service degraded, and reduced redundancy. That distinction is very relevant to Nutanix node failures: the customer may still be online, but the platform may be operating with reduced fault tolerance.”

### KPIs

You can bring mature support-management language:

* Time to acknowledge.
* Time to technical owner.
* Time to mitigation.
* Time to restore resiliency.
* MTTR.
* Reopen rate.
* Escalation aging.
* Customer update SLA.
* RCA quality.
* Repeat incident rate.

### Coaching

You can say:

> “After an incident, I would review whether the team asked the right safety questions early enough: RF level, resiliency status, concurrent failures, capacity, recent changes, and whether the customer had been instructed not to perform risky operations.”

---

## 10. Minimum I need to memorize

Memorize these points cold:

1. A Nutanix node contributes both **compute and storage**.
2. A node includes a **physical host and a CVM**.
3. AOS uses **replication factor** and **checksums** to protect data.
4. After node or disk failure, AOS performs **re-protection**.
5. **HA** handles VM restart; **AOS resiliency** handles data availability.
6. **RF2** usually means two copies; losing one node creates a degraded-risk state.
7. **Data Resiliency Status** is a key check before planned node shutdown or risky action.
8. Do not casually reboot or shut down additional nodes during degraded resiliency.
9. **Stargate** relates to storage I/O.
10. **Curator** relates to background repair, rebalancing, and metadata scanning.
11. Node failure can be hardware, CVM, hypervisor, storage, or network related.
12. Your role is to stabilize, coordinate, communicate, and drive safe resolution.

One sentence to memorize:

> “In a Nutanix node failure, I separate compute recovery from storage resiliency: HA may restart VMs, while AOS serves data from replicas and re-protects the cluster in the background; my job is to confirm impact, validate resiliency, prevent unsafe actions, and coordinate a controlled recovery.”

---

## 11. Advanced / optional level

You can leave these as advanced unless the SRE interview goes deep:

* Detailed Cassandra ring behavior.
* Medusa internals.
* Zookeeper quorum specifics.
* Exact Stargate log interpretation.
* Curator job internals.
* Extent group placement algorithms.
* Erasure coding behavior during failure.
* Detailed AHV CLI syntax.
* VMware HA admission-control tuning.
* NCC check internals.
* Low-level support bundle analysis.
* Disk balancing algorithms.
* AOS version-specific implementation differences.
* Block/rack awareness design tradeoffs.
* Network packet-level debugging between CVMs.

However, you should recognize the terms and be able to say:

> “I understand the component’s role at a troubleshooting level, and I would rely on logs, NCC checks, Prism health, and senior SRE guidance for deeper service-level diagnosis.”

---

## 12. Final checklist

Before the interview, make sure you can answer these verbally:

* [ ] What is a Nutanix node?
* [ ] What is a CVM?
* [ ] What happens to VMs when a host fails?
* [ ] What happens to data when a node fails?
* [ ] What is replication factor?
* [ ] What is re-protection?
* [ ] What is Data Resiliency Status?
* [ ] Why is degraded resiliency dangerous even if workloads are still running?
* [ ] What would you tell a customer not to do?
* [ ] How would you manage communication during the escalation?
* [ ] What are Stargate and Curator at a high level?
* [ ] How would you connect this to your current 24/7 enterprise support experience?

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword   | Meaning                                                                                                                               |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| AOS                        | Acropolis Operating System; Nutanix core software layer for storage, virtualization services, management, resiliency, and operations. |
| AHV                        | Acropolis Hypervisor; Nutanix native hypervisor.                                                                                      |
| Alert                      | Prism or system notification indicating health, capacity, hardware, service, or resiliency issue.                                     |
| Availability Domain        | Failure boundary such as node, block, or rack.                                                                                        |
| Block                      | Physical Nutanix chassis/enclosure that may contain multiple nodes and shared hardware components.                                    |
| Block Awareness            | Placement strategy to keep redundant copies across different blocks when configured/supported.                                        |
| Cassandra                  | Distributed metadata store used by Nutanix services.                                                                                  |
| Checksum                   | Data-integrity mechanism used to detect corruption.                                                                                   |
| Cluster                    | Group of Nutanix nodes operating as one distributed platform.                                                                         |
| Controller VM              | VM running Nutanix services on each node; commonly called CVM.                                                                        |
| Curator                    | Nutanix background service for scanning metadata, cleanup, optimization, rebalancing, and redundancy repair.                          |
| CVM                        | Controller VM; Nutanix service VM running on each node.                                                                               |
| Data Resiliency Status     | Prism/cluster health indicator showing whether the cluster can tolerate failures safely.                                              |
| Degraded State             | Condition where the cluster is still operating but with reduced redundancy or increased risk.                                         |
| Disk Failure               | Loss or degradation of a physical drive in a node.                                                                                    |
| Distributed Storage Fabric | Nutanix storage architecture that pools local disks across nodes into a unified storage layer.                                        |
| ESXi                       | VMware hypervisor often used in enterprise Nutanix environments.                                                                      |
| Extent                     | Logical unit/chunk of data in distributed storage systems.                                                                            |
| Extent Group               | Grouping of extents used by Nutanix storage placement and replication.                                                                |
| Fault Domain               | Component or boundary that can fail, such as node, block, or rack.                                                                    |
| Firmware                   | Low-level software controlling hardware components such as disks, NICs, BIOS, or controllers.                                         |
| GCP                        | Google Cloud Platform.                                                                                                                |
| HA                         | High Availability; mechanism to restart workloads on healthy hosts after failure.                                                     |
| HCI                        | Hyperconverged Infrastructure; architecture combining compute, storage, and virtualization in one platform.                           |
| Host                       | Physical hypervisor server in the Nutanix cluster.                                                                                    |
| I/O                        | Input/output operations, usually storage reads and writes.                                                                            |
| KPI                        | Key Performance Indicator; metric used to measure support or operational performance.                                                 |
| LACP                       | Link Aggregation Control Protocol; network bonding protocol sometimes relevant in node/network issues.                                |
| Latency                    | Delay in completing an operation, often storage or network I/O.                                                                       |
| Maintenance Mode           | Controlled state used to evacuate or prepare a host/node for planned work.                                                            |
| Medusa                     | Nutanix interface to Cassandra metadata services.                                                                                     |
| Metadata                   | Data describing where and how workload data is stored.                                                                                |
| MTTR                       | Mean Time To Resolution or Recovery; key incident/support metric.                                                                     |
| MTU                        | Maximum Transmission Unit; network packet-size setting relevant to CVM/network troubleshooting.                                       |
| NCC                        | Nutanix Cluster Check; health-check framework used to validate cluster state.                                                         |
| Node                       | Physical Nutanix server contributing compute, storage, and services to the cluster.                                                   |
| Node Failure               | Loss or unavailability of a Nutanix node, host, CVM, or critical node component.                                                      |
| Oplog                      | Nutanix write-buffer/log area used in storage operations.                                                                             |
| P1 / Sev1                  | Highest-priority incident, usually major production impact.                                                                           |
| Prism                      | Nutanix management interface for monitoring, operations, alerts, and administration.                                                  |
| Prism Element              | Cluster-level Nutanix management interface.                                                                                           |
| Prism Central              | Centralized Nutanix management platform for multiple clusters and broader operations.                                                 |
| RCA                        | Root Cause Analysis; post-incident explanation of what happened, why, and how to prevent recurrence.                                  |
| Re-protection              | Process of restoring required data replicas after failure.                                                                            |
| Rebalancing                | Redistributing data or workload across healthy resources.                                                                             |
| RF                         | Replication Factor; number of data copies maintained for resiliency.                                                                  |
| RF2                        | Replication Factor 2; two copies of data.                                                                                             |
| RF3                        | Replication Factor 3; three copies of data.                                                                                           |
| Risk Window                | Period where the cluster is online but exposed to higher risk due to degraded resiliency.                                             |
| SLA                        | Service Level Agreement; contractual or operational target for response/resolution/availability.                                      |
| SRE                        | Site Reliability Engineer; role focused on reliability, automation, incident response, and production operations.                     |
| Stargate                   | Nutanix service responsible for storage I/O handling.                                                                                 |
| Storage Pool               | Group of physical storage devices across the cluster.                                                                                 |
| Support Bundle             | Diagnostic package collected for Nutanix Support analysis.                                                                            |
| ToR Switch                 | Top-of-rack switch; network device connecting nodes in a rack.                                                                        |
| Under-replicated Data      | Data that currently has fewer copies than required by the configured replication factor.                                              |
| VM                         | Virtual Machine.                                                                                                                      |
| VMware HA                  | VMware mechanism for restarting VMs after host failure.                                                                               |
| Workload                   | Application, VM, database, or service running on the infrastructure.                                                                  |
| Zookeeper                  | Cluster configuration and coordination service.                                                                                       |
| Zeus                       | Interface to Zookeeper cluster configuration services.                                                                                |

[1]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_3%3Aarc-node-failure-c.html "https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_3%3Aarc-node-failure-c.html"
[2]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_3%3Aarc-controller-vm-failure-c.html "https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_3%3Aarc-controller-vm-failure-c.html"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2096-Nutanix-Core-Performance%3Adata-tiering.html "https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2096-Nutanix-Core-Performance%3Adata-tiering.html"
[4]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v7_3%3Awc-cluster-fault-domains-c.html "https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v7_3%3Awc-cluster-fault-domains-c.html"
[5]: https://www.nutanixbible.com/pdf/4c-book-of-aos-storage.pdf "https://www.nutanixbible.com/pdf/4c-book-of-aos-storage.pdf"
[6]: https://portal.nutanix.com/page/documents/details?targetId=Network-Replacement-Cisco-HCI%3Aahv-node-shutdown-ahv-t.html "https://portal.nutanix.com/page/documents/details?targetId=Network-Replacement-Cisco-HCI%3Aahv-node-shutdown-ahv-t.html"
[7]: https://next.nutanix.com/how-it-works-22/controller-vm-components-38055 "https://next.nutanix.com/how-it-works-22/controller-vm-components-38055"
[8]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v5_20%3Aarc-cluster-components-c.html "https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v5_20%3Aarc-cluster-components-c.html"
[9]: https://www.nutanixbible.com/pdf/2f-book-of-basics-cluster-components.pdf "https://www.nutanixbible.com/pdf/2f-book-of-basics-cluster-components.pdf"
[10]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_3%3Aarc-block-awareness-c.html "https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_3%3Aarc-block-awareness-c.html"
[11]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_8%3Aahv-node-maintenance-mode-put-ahv-t.html "https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_8%3Aahv-node-maintenance-mode-put-ahv-t.html"
