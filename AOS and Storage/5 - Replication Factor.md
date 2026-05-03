# [AOS / Storage] Replication Factor

## 1. Short definition

**Replication Factor (RF)** in Nutanix AOS defines how many copies of data are maintained across the cluster for availability and fault tolerance.

In practical terms:

* **RF2** = two copies of data. The cluster can normally tolerate **one node or disk failure**.
* **RF3** = three copies of data. The cluster can tolerate **two node or drive failures**, assuming the failures are in appropriate independent failure domains and the cluster meets the requirements.
* RF is configured at the **storage container** level, not per individual VM in the usual operational sense. Nutanix documentation states that data RF is configured through Prism at the container level. 

Nutanix describes RF as a core mechanism for data protection and availability because the system can continue serving data from another full copy without recomputing data from parity during normal failure handling. ([portal.nutanix.com][1])

---

## 2. Clear explanation

Nutanix AOS is a distributed storage platform. Instead of relying on a traditional external SAN or RAID controller, each Nutanix node contributes local storage, and AOS presents that distributed storage as a resilient storage layer to the hypervisor and VMs.

When a VM writes data, Nutanix does not simply store that data on the local disk of the node. The write is handled by the local **Controller VM**, or **CVM**, and then replicated synchronously to another CVM, or to two other CVMs depending on the configured RF. Nutanix Bible describes that after data is written to the local **OpLog**, it is synchronously replicated to one or two other CVMs’ OpLogs before the write is acknowledged as successful. 

That sentence is very important for interviews.

It means:

1. The write is not considered safe until the required number of replicas exists.
2. RF is part of the write path.
3. Availability has a capacity and latency/network cost.
4. Replication is distributed across the cluster, not pinned to a static “mirror node.”
5. If the cluster loses too much hardware, or lacks enough capacity to rebuild, the customer may enter a degraded or unavailable state.

### RF2

With **RF2**, AOS maintains two copies of data. This is the standard/default posture for most environments. Nutanix’s Prism documentation says Nutanix clusters have RF2 by default, meaning they can tolerate a single node or drive failure. ([portal.nutanix.com][2])

A simple explanation:

> “RF2 means Nutanix keeps two independent copies of data across the cluster. If one disk or node fails, the system should continue serving I/O from the surviving copy while AOS rebuilds protection elsewhere, assuming the cluster has enough healthy resources and capacity.”

### RF3

With **RF3**, AOS keeps three copies of data. Nutanix positions RF3 for environments with higher protection requirements, but it requires more nodes and more storage capacity. Prism 7.0 documentation states that configuring RF3 requires a minimum of five nodes, and Nutanix recommends RF3 for sites with high protection requirements. ([portal.nutanix.com][3])

A simple explanation:

> “RF3 increases fault tolerance by maintaining three copies of data, allowing the platform to survive two simultaneous failures under the right failure-domain conditions. The tradeoff is capacity: three copies consume significantly more storage than RF2.”

### Important distinction: RF vs redundancy factor

You may see both **Replication Factor** and **Redundancy Factor** in Nutanix material.

For interview purposes:

* **Replication Factor** usually refers to the number of data copies in a storage container.
* **Redundancy Factor** is often used at the cluster/platform resiliency level, especially in Prism documentation around RF2/RF3 behavior.
* In casual technical conversation, people often use RF for both, but in a support interview you should be precise.

The safe way to say it:

> “At the storage-container level, RF defines how many data copies AOS maintains. At the cluster resiliency level, Nutanix also discusses redundancy factor, which maps to the cluster’s ability to tolerate node, disk, block, or rack failures depending on configuration and failure domains.”

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, you do not need to sound like the person who will personally debug every Stargate or Curator issue. You need to sound like the leader who understands why RF affects **availability, escalation severity, recovery strategy, customer communication, and risk management**.

Replication Factor matters because it directly impacts:

### Availability

If a customer loses a node or disk, RF determines whether the data remains available and whether the environment is operating in a degraded but protected state or a degraded and risky state.

### Incident severity

A failed disk in an RF2 cluster may be a controlled hardware replacement case. A second failure before rebuild completion may become a high-severity incident because the remaining replica set may be at risk.

### Customer communication

Customers will ask:

* “Are my VMs safe?”
* “Can I continue production workloads?”
* “Are we exposed to data loss?”
* “Why is the cluster showing under-replicated data?”
* “Can we postpone the maintenance?”
* “Can we add capacity instead of shutting workloads down?”

A support manager must translate technical risk into clear executive language.

### Capacity and resiliency

RF consumes capacity. RF2 roughly means full duplicate copies; RF3 adds another full copy. Nutanix documentation explicitly notes that RF provides high availability but at the cost of storage resources because full copies are required. ([portal.nutanix.com][1])

### Escalation judgment

You need to know when the case is not “just a disk failure” anymore. For example:

* Under-replicated data exists.
* The cluster is above resilient capacity.
* Another failure would cause unavailability.
* Rebuild is slow because of capacity, performance, or hardware bottlenecks.
* The customer wants to perform maintenance while the cluster is degraded.

Nutanix Bible states that when cluster usage exceeds resilient capacity, the cluster might not be able to tolerate and recover from failures anymore. It also says it is highly recommended not to exceed resilient capacity. 

That is exactly the type of risk a Worldwide Support Manager must understand and communicate.

---

## 4. Key concepts

### Replication Factor

The number of copies of data maintained for availability. Nutanix Bible defines RF as the number of copies of data maintained on the cluster for data availability. 

### Fault Tolerance

The number of failures the system can tolerate at the configured failure domain while maintaining data availability. Nutanix Bible distinguishes RF from **Fault Tolerance**, defining FT as the number of failures at the configured failure domain that can be handled while data is rebuilt completely. 

### Failure domain

The independent unit across which replicas should be distributed. It can be node-level, block-level, rack-level, depending on cluster design.

Simple interview explanation:

> “RF says how many copies exist. Failure domain says where those copies are placed so that one physical failure does not remove all copies.”

### CVM

The **Controller VM** is the Nutanix virtual appliance running on each node that provides storage services. The local CVM handles I/O and coordinates replication to other CVMs.

### OpLog

The **OpLog** is a persistent write buffer. It absorbs random writes, coalesces them, and replicates them before acknowledgement. Nutanix Bible describes OpLog as a staging area for writes, with synchronous replication to other CVMs before the write is acknowledged. 

### Stargate

A Nutanix service responsible for handling data I/O. You do not need to deeply debug Stargate as a support manager, but you should recognize it as a storage I/O service.

### Curator

A Nutanix background service/framework that performs distributed storage maintenance tasks, including scans, rebuilds, balancing, and post-process storage optimization. Nutanix Bible references Curator as the framework that automates rebuild of EC strips and performs post-process tasks. 

### Resilient capacity

The amount of usable capacity while preserving the ability to rebuild after a failure. Nutanix Bible defines resilient capacity as total cluster capacity minus the capacity needed to rebuild from configured fault-tolerance failures. 

### Under-replicated data

Data that currently has fewer replicas than the configured RF. This is a critical troubleshooting keyword. It usually means the cluster is not fully protected.

### RF2 vs RF3

RF2 is the common/default model. RF3 is used where higher fault tolerance is needed, but requires more nodes and storage. Prism 7.0 documentation says RF3 requires at least five nodes and is recommended for high-protection sites. ([portal.nutanix.com][3])

### RF vs Erasure Coding

RF stores full copies. **Erasure Coding**, or **EC-X**, uses parity to reduce storage overhead for suitable data. Nutanix Bible explains that RF provides high availability without requiring data recomputation on failure, but uses full copies; EC balances availability with reduced storage by encoding data with parity. 

Interview-safe wording:

> “RF is simpler and faster for failure reads because another full copy exists. Erasure Coding improves storage efficiency but introduces parity concepts and may have different behavior during rebuild or failure.”

---

## 5. How it appears in a real escalation

### Scenario 1: Disk failure in an RF2 cluster

A customer opens a P1/P2 because Prism reports a failed disk and under-replicated data.

As support manager, you should think:

* Is the cluster still serving production workloads?
* Is data under-replicated?
* Is rebuild running?
* Is there enough resilient capacity?
* Are there additional hardware alerts?
* Is the customer planning maintenance or another risky operation?
* Is this a single failure or part of a larger pattern?

Customer-facing explanation:

> “The cluster is currently degraded because one disk has failed. With RF2, the system is designed to tolerate a single disk or node failure, but until rebuild completes, the risk posture is elevated. Our priority is to confirm cluster health, verify whether any data is under-replicated, check resilient capacity, and restore full protection before any non-essential maintenance.”

### Scenario 2: Node failure during rebuild

A node fails while the cluster was already rebuilding from a disk failure.

This is a more serious escalation.

Why?

Because RF2 gives one additional copy. If another failure occurs before protection is restored, some data may lose redundancy or become unavailable depending on placement and failure domain.

Support-manager framing:

> “This is no longer a routine hardware replacement. We need immediate technical triage, escalation to senior SRE/storage specialists, customer risk communication, and a controlled recovery plan. We should avoid unnecessary changes until the replica state and cluster health are understood.”

### Scenario 3: Customer asks whether they should enable RF3

A financial services customer asks whether RF3 is necessary for their critical production cluster.

Good answer:

> “RF3 can be appropriate for high-protection environments because it can tolerate more simultaneous failures, but it requires more nodes and consumes more storage. I would not treat it as a default recommendation without reviewing workload criticality, cluster size, capacity headroom, failure domains, RTO/RPO expectations, and operational model. For many workloads RF2 is sufficient; for critical sites where the business impact of a second failure is unacceptable, RF3 may be justified.”

### Scenario 4: Capacity close to resilient capacity

The customer says: “We still have free space, why are you telling us we are at risk?”

This is a classic support conversation.

Answer:

> “Free capacity and resilient capacity are not the same. Free capacity tells us how much space remains. Resilient capacity tells us whether the cluster can still rebuild after a failure while maintaining the configured protection policy. A cluster can have free space but still be too close to the threshold needed for safe rebuild.”

### Scenario 5: Performance degradation during rebuild

After a node replacement or disk replacement, the customer sees latency increase.

Possible explanation:

> “During rebuild, the cluster is doing background work to restore the desired replication factor. That can increase I/O, network, and disk activity. The support focus is to verify whether rebuild is progressing normally, whether customer workload latency is within acceptable bounds, whether throttling or scheduling is needed, and whether there are additional bottlenecks.”

---

## 6. Triage questions I should ask

Use these in interviews and real escalations.

### Customer impact

1. Which applications or VMs are affected?
2. Is there data unavailability, VM unresponsiveness, or only an alert?
3. What is the business impact?
4. Is this production, DR, VDI, database, or test/dev?
5. Is the customer currently in a change window or maintenance activity?

### Cluster state

6. What is the configured RF: RF2 or RF3?
7. Which storage containers are affected?
8. Is there under-replicated data?
9. Is rebuild in progress?
10. Is rebuild progressing, stalled, or repeatedly failing?

### Failure details

11. Was the failure a disk, node, block, rack, CVM, network, or hypervisor issue?
12. Are there multiple simultaneous failures?
13. Did a second failure occur before rebuild completed?
14. Is the failure domain node-level, block-level, or rack-level?

### Capacity and resiliency

15. Is the cluster below or above resilient capacity?
16. Is there enough free capacity to rebuild?
17. Is storage skewed across nodes?
18. Are any nodes significantly more utilized than others?

### Performance and network

19. Is latency elevated at the VM, hypervisor, CVM, or disk layer?
20. Are there network errors between CVMs?
21. Is the replication network healthy?
22. Are we seeing packet loss, congestion, or MTU issues?

### Operational control

23. Has the customer made recent changes?
24. Are they planning additional maintenance?
25. Should we pause non-essential operations until protection is restored?
26. Do we need senior SRE, engineering, hardware logistics, TAM, or account team involvement?

---

## 7. Likely interview questions

### Manager / leadership interview

1. Explain Replication Factor in Nutanix in simple terms.
2. Why does RF matter in enterprise support?
3. How would you communicate RF2 risk to a customer after a node failure?
4. How would you manage a P1 escalation involving under-replicated data?
5. How do you balance technical accuracy with executive-level communication?
6. When would you involve senior SREs or engineering?
7. How would you prevent a customer from making the situation worse during a degraded state?
8. How would you handle a customer demanding immediate root cause while recovery is still ongoing?
9. How would you prioritize cases when multiple customers are degraded?
10. What KPIs would you track around storage resiliency incidents?

### Technical / SRE interview

1. What is RF2?
2. What is RF3?
3. What is the difference between RF and erasure coding?
4. What happens during a write in an RF2 Nutanix cluster?
5. What is the role of the CVM in replication?
6. What is OpLog?
7. What is under-replicated data?
8. What is resilient capacity?
9. Why does RF3 require more nodes?
10. How can a cluster have free space but still be at resiliency risk?
11. How would you triage a failed node in an RF2 cluster?
12. What metrics would you check during rebuild?
13. What is the risk of performing maintenance while a cluster is degraded?
14. How does RF affect capacity planning?
15. How would you explain RF to a VMware customer used to SAN/RAID architecture?

### Panel / escalation case interview

1. A customer lost a node in an RF2 cluster and now wants to upgrade AOS. What do you do?
2. A second disk failed during rebuild. How do you lead the escalation?
3. A customer says Prism shows under-replicated data but all VMs are running. How do you explain the risk?
4. A database customer wants maximum availability but complains about storage overhead. How do you frame RF2, RF3, and EC?
5. The account team wants to reassure the customer, but the SRE says risk is high. How do you handle communication?

---

## 8. Model answers in English

### Q1. What is Replication Factor in Nutanix?

**Model answer:**

> “Replication Factor defines how many copies of data Nutanix AOS maintains across the cluster for availability. With RF2, the platform keeps two copies and can normally tolerate one node or disk failure. With RF3, it keeps three copies and can tolerate a higher level of failure, typically two failures under the right failure-domain conditions. The important point from a support perspective is that RF directly affects data availability, rebuild behavior, capacity consumption, and customer risk during incidents.”

### Q2. How does RF2 work during a write?

**Model answer:**

> “At a high level, when a VM writes data, the local Nutanix CVM handles the I/O and writes to the local OpLog. Before the write is acknowledged as successful, AOS synchronously replicates the data to another CVM’s OpLog for RF2, or to additional CVMs for higher RF. That means the write is acknowledged only after the required protection level exists. As a support manager, I do not need to debug every internal service, but I need to understand that replication is part of the write path and depends on healthy CVMs, storage, and network connectivity.”

### Q3. Why does RF matter in support escalations?

**Model answer:**

> “RF matters because it tells us how much failure the cluster can absorb before customer workloads are at risk. In an RF2 cluster, a single node or disk failure may be tolerated, but the cluster is degraded until protection is rebuilt. If another failure occurs before rebuild completes, the risk increases significantly. So in an escalation I would immediately focus on customer impact, under-replicated data, rebuild progress, resilient capacity, and whether we need to restrict customer changes until the cluster is healthy again.”

### Q4. How would you explain under-replicated data to a customer?

**Model answer:**

> “I would explain that under-replicated data means some data currently has fewer protected copies than the configured policy requires. It does not automatically mean data is lost, but it does mean the cluster’s risk posture is elevated. The priority is to restore the desired replication level by resolving the failed component, confirming sufficient capacity, and monitoring rebuild progress. I would avoid unnecessary maintenance or disruptive changes until protection is restored.”

### Q5. What is the difference between RF2 and RF3?

**Model answer:**

> “RF2 maintains two copies of data and is the default or common configuration for many workloads. RF3 maintains three copies and provides greater fault tolerance, but it consumes more capacity and requires a larger cluster. I would position RF3 for environments with high protection requirements where the business impact of multiple simultaneous failures is unacceptable. The decision should consider cluster size, capacity, failure domains, workload criticality, and business RTO/RPO expectations.”

### Q6. What is resilient capacity?

**Model answer:**

> “Resilient capacity is the capacity the cluster can use while still preserving enough space to rebuild after a failure. A cluster can have free capacity but still be at risk if it does not have enough capacity to restore the configured replication factor after a node or disk failure. In support, this is important because a customer may think ‘I still have free space,’ while the real question is whether the cluster can self-heal safely.”

### Q7. How would you lead a P1 escalation involving RF risk?

**Model answer:**

> “I would first establish customer impact and whether workloads are unavailable or degraded. Then I would make sure the technical team checks cluster health, failed components, under-replication, rebuild progress, capacity, and network health. In parallel, I would control communication: one clear owner, regular updates, explicit risk statements, and no unnecessary changes until the environment is stable. If the cluster is at risk of a second failure or rebuild is stalled, I would escalate to senior SREs or engineering and involve the account team for customer communication.”

### Q8. How would you connect this to your previous experience?

**Model answer:**

> “In my current support and operations role, I manage 24/7 enterprise incidents where availability, SLA, MTTR, monitoring, and customer communication are critical. Replication Factor is a Nutanix-specific storage concept, but the operational pattern is familiar: identify current redundancy state, quantify risk, stabilize the platform, prevent additional changes, communicate clearly, and drive recovery with measurable progress. My value is not only knowing the technical mechanism, but leading the escalation so the customer receives accurate guidance and the engineering team can focus on recovery.”

### Q9. What should a support manager avoid saying?

**Model answer:**

> “I would avoid saying ‘everything is safe’ just because VMs are still running. A degraded RF state may still be serving I/O, but the risk profile has changed. I would also avoid giving a root cause before recovery data is collected. The right message is: workloads may still be available, but protection is degraded, we are validating under-replication and rebuild progress, and we recommend avoiding non-essential changes until the cluster returns to a healthy state.”

---

## 9. Connection with my experience

Your current background maps very well to this topic.

You already understand the operational side:

* **Incident management** → RF incidents are classic high-severity infrastructure incidents.
* **SLA / MTTR** → Rebuild time, replacement time, and customer impact directly affect MTTR.
* **KPIs** → You can discuss time to detect, time to engage SRE, time to restore redundancy, time to close RCA.
* **Monitoring** → RF issues surface through alerts, capacity thresholds, degraded states, latency, and hardware events.
* **Escalations** → Under-replicated data requires careful escalation governance.
* **24/7 support** → Hardware failures and storage degradation often happen outside business hours.
* **Cloud operations** → Similar mindset to multi-AZ redundancy, replication, failover, and degraded-state operations.
* **Kubernetes / SaaS** → The conceptual parallel is replica count. In Kubernetes, one replica is not enough for HA; in Nutanix, one copy is not enough for storage resilience.
* **Grafana / Kibana** → You are used to correlating symptoms across layers. In Nutanix, you would correlate VM latency, CVM health, disk errors, network issues, and rebuild progress.
* **Jira / Confluence / Salesforce** → Useful for incident timeline, customer updates, case notes, postmortem, and action tracking.

A strong positioning sentence for you:

> “My advantage is that I already manage enterprise support under SLA pressure. I am building the Nutanix-specific vocabulary around AOS, RF, CVMs, Prism, and storage resiliency so that I can lead escalations with both technical credibility and operational discipline.”

---

## 10. Minimum I need to memorize

Memorize these points.

1. **RF means number of data copies.**
2. **RF2 = two copies; usually tolerates one node or disk failure.**
3. **RF3 = three copies; higher protection, more capacity cost, minimum larger cluster requirements.**
4. **RF is configured at the storage container level.**
5. **Writes are synchronously replicated before acknowledgement.**
6. **CVMs handle storage I/O and replication.**
7. **OpLog is the write buffer involved in the write path.**
8. **Under-replicated data means risk is elevated.**
9. **Resilient capacity matters more than simple free capacity during failures.**
10. **Do not perform risky maintenance while degraded unless explicitly approved by senior technical owners.**
11. **RF is not the same as backup or disaster recovery.**
12. **RF protects against local hardware failure; it does not replace snapshots, backups, replication to another site, or DR strategy.**
13. **RF has a capacity cost: RF2 roughly requires duplicate data copies; RF3 requires an additional full copy.**
14. **As manager, your role is to lead triage, risk communication, escalation discipline, and customer confidence.**

A short verbal answer you can practice:

> “Replication Factor is the Nutanix AOS mechanism that defines how many copies of data are maintained across the cluster. RF2 keeps two copies and normally tolerates one disk or node failure; RF3 keeps three copies and provides higher resiliency at higher capacity cost. In support, RF matters because it tells us whether a cluster is protected, degraded, under-replicated, or at risk during hardware failures. As a manager, I would focus on impact, under-replication, rebuild progress, resilient capacity, and clear customer communication.”

---

## 11. Advanced / optional level

You do **not** need to master these deeply for a Manager interview, but you should recognize the terms.

### Advanced areas worth recognizing

* Stargate internals
* Cassandra metadata ring
* Paxos consistency
* Extents and extent groups
* Curator scans
* EC-X striping and parity
* Block awareness
* Rack awareness
* Storage tiering
* ILM
* Shadow clones
* Detailed NCC command output
* Advanced nCLI diagnostics
* AHV vs ESXi-specific storage behavior
* Rebuild throttling behavior
* Detailed CVM service logs

### How to speak about advanced details safely

Use this pattern:

> “I understand the concept at the operational level. For deep internals like Stargate logs, Curator behavior, or Cassandra metadata issues, I would involve the appropriate senior SRE or engineering specialist, while I manage impact, prioritization, and customer communication.”

This is exactly the right positioning for a **technical escalation manager**, not a Senior SRE IC.

---

## 12. Final checklist

Before your interview, be able to answer:

* Can I explain RF2 in less than 30 seconds?
* Can I explain RF3 and its tradeoff?
* Can I explain why RF is different from backup?
* Can I explain why under-replicated data is risky?
* Can I explain resilient capacity?
* Can I describe what happens during a failed disk escalation?
* Can I describe what happens if a second failure occurs during rebuild?
* Can I explain this to both an SRE and a CIO?
* Can I connect RF to my SaaS/cloud incident management experience?
* Can I avoid overclaiming deep Nutanix internals while still sounding technically credible?

Best final interview positioning:

> “I am not positioning myself as the deepest AOS storage engineer in the room. I am positioning myself as a support leader who understands the storage resiliency model well enough to lead escalations, ask the right questions, communicate risk accurately, and coordinate SRE, engineering, customer, and account teams during high-pressure incidents.”

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword   | Meaning                                                                                          |
| -------------------------- | ------------------------------------------------------------------------------------------------ |
| AOS                        | Acropolis Operating System; Nutanix distributed storage and cloud infrastructure software layer. |
| AHV                        | Acropolis Hypervisor; Nutanix’s native hypervisor.                                               |
| Availability               | Ability of the platform or workload to remain accessible during failures.                        |
| Availability Domain        | Logical failure boundary used to place replicas safely.                                          |
| Block Awareness            | Replica placement strategy that considers Nutanix hardware blocks as failure domains.            |
| Capacity Headroom          | Spare capacity available for growth, rebuilds, and safe operations.                              |
| Cassandra                  | Distributed metadata store technology used by Nutanix in modified form.                          |
| Cluster                    | Group of Nutanix nodes operating as one distributed system.                                      |
| Container                  | Nutanix storage container where storage policies such as RF can be configured.                   |
| Controller VM              | Nutanix virtual machine on each node that provides storage services.                             |
| Curator                    | Nutanix background framework for scans, rebuilds, balancing, and storage optimization.           |
| CVM                        | Controller VM; Nutanix VM responsible for storage I/O and services on each node.                 |
| Degraded State             | Condition where the cluster is operational but not fully protected or healthy.                   |
| Disk Failure               | Loss or malfunction of a physical storage device.                                                |
| Distributed Storage Fabric | Nutanix software-defined distributed storage layer.                                              |
| DSF                        | Distributed Storage Fabric; Nutanix’s distributed storage architecture.                          |
| EC-X                       | Erasure Coding in Nutanix; parity-based storage efficiency mechanism.                            |
| Erasure Coding             | Method that uses parity instead of full copies to reduce storage overhead.                       |
| Escalation                 | Process of raising case urgency or involving more senior technical/business stakeholders.        |
| Extent                     | Unit of stored data in Nutanix AOS.                                                              |
| Extent Group               | Grouping of extents used internally by AOS storage.                                              |
| Failure Domain             | Independent unit of failure, such as disk, node, block, or rack.                                 |
| Fault Tolerance            | Number of failures the system can survive at a configured failure domain.                        |
| FT                         | Fault Tolerance.                                                                                 |
| HCI                        | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform.  |
| ILM                        | Intelligent Lifecycle Management; Nutanix data movement based on access patterns and tiers.      |
| MTTR                       | Mean Time To Restore/Repair; time needed to restore service or protection.                       |
| NCC                        | Nutanix Cluster Check; diagnostic and health-check tool.                                         |
| Node                       | Physical Nutanix server participating in the cluster.                                            |
| Node Failure               | Loss or outage of a Nutanix node.                                                                |
| OpLog                      | Persistent write buffer used by AOS before data is drained to the extent store.                  |
| Paxos                      | Consensus algorithm used for consistency in distributed systems.                                 |
| Prism                      | Nutanix management and monitoring interface.                                                     |
| Prism Element              | Cluster-level Nutanix management interface.                                                      |
| Prism Central              | Centralized management interface for multiple Nutanix clusters.                                  |
| Rack Awareness             | Replica placement strategy that considers racks as failure domains.                              |
| RCA                        | Root Cause Analysis.                                                                             |
| Rebuild                    | Process of restoring the required number of data replicas after failure.                         |
| Redundancy Factor          | Cluster resiliency level describing tolerance to hardware failures.                              |
| Replica                    | A copy of data stored on another disk/node/CVM for protection.                                   |
| Replication Factor         | Number of data copies maintained for availability.                                               |
| Resilient Capacity         | Capacity usable while still preserving the ability to rebuild after failure.                     |
| RF                         | Replication Factor or Redundancy Factor, depending on context.                                   |
| RF2                        | Two data copies; common/default protection level.                                                |
| RF3                        | Three data copies; higher protection level with higher capacity cost.                            |
| RPO                        | Recovery Point Objective; maximum acceptable data loss window.                                   |
| RTO                        | Recovery Time Objective; target time to restore service.                                         |
| SLA                        | Service Level Agreement.                                                                         |
| SRE                        | Site Reliability Engineer / Engineering.                                                         |
| Stargate                   | Nutanix service responsible for handling storage I/O.                                            |
| Storage Container          | Logical storage construct where policies such as RF are applied.                                 |
| Storage Skew               | Uneven storage utilization across nodes.                                                         |
| Synchronous Replication    | Replication completed before acknowledging the write.                                            |
| Under-replicated Data      | Data with fewer copies than required by the configured RF.                                       |
| VM                         | Virtual Machine.                                                                                 |
| Write Acknowledgement      | Confirmation that a write is accepted after required protection steps complete.                  |

[1]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2056-MySQL-on-Nutanix%3Areplication-factor.html&utm_source=chatgpt.com "Replication Factor - Nutanix"
[2]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v6_10%3Aarc-redundancy-factor3-c.html?utm_source=chatgpt.com "Prism 6.10 - Redundancy Factor 3 - Nutanix"
[3]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v7_0%3Awc-replication-factor-3-c.html&utm_source=chatgpt.com "Prism 7.0 - Replication Factor 3 - portal.nutanix.com"
