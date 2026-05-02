# Compute/storage convergence

## 1. Short definition

**Compute/storage convergence** is the architectural principle behind **hyperconverged infrastructure (HCI)**: instead of having separate compute servers, storage arrays, and storage networks managed as independent silos, each node contributes both **CPU/RAM** and **local disks/SSDs** to a distributed platform. Software then aggregates those resources into a shared, resilient infrastructure layer.

In Nutanix terms, this is central to **Nutanix Cloud Infrastructure (NCI)**: **AOS** provides the distributed storage layer, **AHV or another supported hypervisor** runs the workloads, and **Prism** provides management and operational visibility. Nutanix describes HCI as combining servers and storage into a distributed infrastructure platform, replacing legacy architectures made of separate servers, storage networks, and storage arrays. ([Nutanix][1])

---

## 2. Clear explanation

In a traditional enterprise architecture, a VM usually runs on a compute host, but its data lives on an external SAN/NAS storage array. The path looks roughly like this:

**VM → Hypervisor → Host HBA/NIC → Storage network → Storage controller → Disk**

That architecture can work very well, but it creates operational silos: server team, storage team, virtualization team, network team. During incidents, troubleshooting often becomes a blame loop: “Is it VMware?”, “Is it storage latency?”, “Is it the fabric?”, “Is it the array?”, “Is it the host?”

In a converged HCI architecture, the physical server node contains both compute and storage. Nutanix then uses software to make the local disks across nodes behave like a resilient distributed storage system. Nutanix documentation describes its architecture as running a **Controller VM**, or **CVM**, on every node; those CVMs form a highly distributed shared-nothing infrastructure. ([Nutanix Portal][2])

A simplified Nutanix flow looks like this:

**VM → Hypervisor → Local CVM → Nutanix distributed storage fabric → Local/remote replicas across nodes**

The important interview point is this:

> In Nutanix HCI, storage is not “an external box.” Storage services are distributed across the same cluster that runs the workloads.

Each node contributes:

| Resource         | Role in the cluster                                                                                                                           |
| ---------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| CPU / RAM        | Runs guest VMs, AHV/ESXi, and the CVM                                                                                                         |
| SSD / HDD / NVMe | Provides local capacity and performance                                                                                                       |
| CVM              | Handles data services, I/O, metadata, placement, snapshots, replication, compression, deduplication, erasure coding, and resiliency functions |
| Network          | Carries VM traffic, management traffic, replication, and storage-related cluster communication                                                |

Nutanix states that the **CVM plays a pivotal role in data management**, handling user I/O, data placement, metadata management, data integrity, availability, garbage collection, compression, deduplication, erasure coding, snapshots, and replication. ([Nutanix Portal][3])

The key mental model:

**Compute/storage convergence means every node is both a workload host and part of the storage system.**

That has major operational implications. If a node is unhealthy, you are not only thinking about VM capacity; you are also thinking about storage resiliency, replica placement, data locality, rebuild activity, and performance impact.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role at Nutanix, this topic matters because it is the foundation of almost every serious escalation.

You do not need to sound like a kernel-level storage engineer, but you must be able to reason clearly about the relationship between:

* Workload performance.
* Storage latency.
* Node health.
* CVM health.
* Cluster resiliency.
* Capacity headroom.
* Network quality.
* Hypervisor behavior.
* Customer business impact.

Nutanix sells operational simplicity, but support escalations often happen when that simplicity is under pressure: degraded node, failed disk, saturated storage, high latency, failed upgrades, replication issues, or customer confusion after moving from a traditional SAN architecture to HCI.

Nutanix’s public positioning emphasizes that HCI brings compute, storage, networking, and virtualization into one efficient platform, simplifying operations and reducing legacy infrastructure silos. ([Nutanix][4]) As a support manager, your job is to make sure that promise survives real-world incidents.

For the interview, you should position yourself like this:

> “I understand that in HCI, compute and storage cannot be escalated as isolated domains. A performance case may look like a VM issue, but the root cause could involve CVM resource pressure, storage latency, replication traffic, network drops, disk rebuilds, noisy workloads, or capacity imbalance. As a support manager, I would drive structured triage across those layers while keeping customer communication clear and business-impact focused.”

That is exactly the positioning of a **technical escalation manager**, not a Senior SRE IC.

---

## 4. Key concepts

### 4.1 HCI versus traditional three-tier infrastructure

Traditional three-tier infrastructure separates:

* Compute hosts.
* Storage arrays.
* Storage network.
* Virtualization management.

HCI collapses much of this into a software-defined cluster. Nutanix defines HCI as a distributed infrastructure platform that combines servers and storage and replaces separate servers, storage networks, and storage arrays. ([Nutanix][1])

In interview language:

> “The operational shift is from managing infrastructure by hardware silos to managing it as a distributed software platform.”

### 4.2 Nutanix AOS

**AOS** is the core Nutanix software layer that enables distributed storage and data services. Nutanix describes AOS as the scale-out storage technology that makes HCI possible. ([Nutanix][5])

For interview purposes, remember:

* AOS abstracts local disks into a distributed storage fabric.
* AOS provides resilience and data services.
* AOS is central to Nutanix’s ability to scale out by adding nodes.
* AOS is where many support escalations become technical: latency, rebuilds, metadata, capacity, snapshots, replication, dedupe, compression, erasure coding.

### 4.3 Controller VM, or CVM

The **CVM** is one of the most important Nutanix concepts for this topic.

Each Nutanix node runs a CVM. The CVM handles storage I/O and distributed storage services. Nutanix states that the CVM handles user I/O, data placement, metadata management, data integrity, availability, garbage collection, compression, deduplication, erasure coding, snapshots, and replication. ([Nutanix Portal][3])

Interview-safe explanation:

> “The CVM is the local storage controller running as a VM on each Nutanix node. Instead of relying on an external storage controller, Nutanix distributes storage control across CVMs in the cluster.”

### 4.4 Distributed Storage Fabric

The Nutanix **Distributed Storage Fabric**, often abbreviated as **DSF**, aggregates storage resources across nodes. Nutanix documentation states that CVMs work together to aggregate storage into a global pool that guest VMs can consume, and that the distributed storage fabric preserves data and system integrity during node, disk, application, or hypervisor failures. ([Nutanix Portal][6])

Practical meaning:

* VM storage is distributed.
* Data has replicas or protection mechanisms.
* The system is designed to survive component failures.
* Storage operations are software-coordinated across the cluster.

### 4.5 Data locality

A major HCI performance concept is **data locality**: serving VM I/O from storage resources close to the VM where possible.

Nutanix documentation says AOS storage drives performance and low latency by providing storage resources to VMs locally on the same host, allowing the local storage controller to handle I/O for VMs running on that physical node. ([Nutanix Portal][7])

Interview-safe explanation:

> “Data locality reduces unnecessary network hops and helps keep latency predictable, but the cluster must still maintain replicas and resilience across nodes.”

### 4.6 Shared-nothing architecture

Nutanix describes its architecture as a **shared-nothing infrastructure**, where each node contributes resources and storage services are distributed rather than centralized in a single controller pair. ([Nutanix Portal][2])

Practical meaning:

* No single external storage array is required for the core HCI storage path.
* Resilience comes from distributed software and replicas.
* Scale-out is achieved by adding nodes.
* Failure domains matter: disk, node, block, rack, network.

### 4.7 Scale-out

A core HCI advantage is that capacity and performance can scale by adding nodes. Nutanix NCI includes AOS as the scale-out storage technology that enables HCI. ([Nutanix][5])

Important nuance for interviews:

HCI scaling is powerful, but not magically infinite. Real systems still require planning around:

* CPU/RAM utilization.
* Storage capacity.
* IOPS.
* Latency.
* Network bandwidth.
* Failure tolerance.
* Licensing.
* Workload profile.
* Rebuild windows.
* Upgrade compatibility.

### 4.8 Compute-only nodes

Advanced but useful nuance: Nutanix supports scenarios where an existing HCI cluster can be expanded with **AHV compute-only nodes**, subject to version and network requirements. Nutanix documentation states that an existing HCI cluster must have at least three HCI nodes, compute and HCI nodes must be on the same network/subnet, and AOS/AHV version requirements apply. ([Nutanix Portal][8])

Why this matters:

It shows that convergence is the default HCI model, but enterprise architectures may still need flexible scaling patterns. You do not need to master this deeply now, but it is useful to know that not every node necessarily has identical storage contribution in every deployment.

---

## 5. How it appears in a real escalation

### Scenario: Customer reports severe VM latency after a node/disk event

A large enterprise customer opens a P1 escalation:

> “Our business-critical application is slow. The database VMs on Nutanix are experiencing high latency. We migrated from a traditional SAN to Nutanix and now we think the cluster cannot handle production workloads.”

As a support manager, you should avoid jumping directly to “storage problem” or “customer workload problem.” In HCI, the correct escalation framing is multi-layered.

You would drive the team to establish:

1. **Business impact**

   * Which application is affected?
   * How many users?
   * Revenue/customer-facing impact?
   * Is there a workaround?
   * Is this a P1, P2, or performance degradation?

2. **Scope**

   * One VM, multiple VMs, one host, one cluster, one site?
   * Only write latency or also read latency?
   * AHV only, ESXi only, or mixed environment?
   * Specific time window?

3. **Recent changes**

   * Node failure?
   * Disk replacement?
   * Upgrade?
   * New workload?
   * Snapshot/backup job?
   * Replication job?
   * Network change?
   * Firmware or hypervisor change?

4. **Cluster health**

   * Any degraded node?
   * Any unhealthy CVM?
   * Any disk failures?
   * Any rebuild or rebalancing activity?
   * Any capacity pressure?
   * Any metadata or Stargate-related alerts?
     *(You do not need to overuse internal service names in a manager interview unless you are confident.)*

5. **Performance indicators**

   * VM latency.
   * Controller latency.
   * Disk latency.
   * CPU ready / CPU contention.
   * CVM CPU/RAM pressure.
   * Network drops/retransmits.
   * IOPS and throughput.
   * Read/write mix.
   * Hotspots.

6. **Customer communication**

   * Acknowledge impact.
   * Provide structured investigation path.
   * Avoid premature RCA.
   * Give regular updates.
   * Separate mitigation from root cause.
   * Align support, engineering, account team, and customer stakeholders.

The key message:

> In HCI, a storage escalation may involve compute, and a compute escalation may involve storage. The support manager must coordinate cross-domain triage without allowing the case to fragment.

---

## 6. Triage questions I should ask

### Business and severity

1. What is the customer-visible impact?
2. Is production down, degraded, or at risk?
3. Which business service or application is affected?
4. When did the issue start?
5. Is the issue constant, intermittent, or workload-dependent?
6. Are there contractual SLA implications?

### Scope

7. Is the issue affecting one VM, several VMs, one host, one cluster, or multiple sites?
8. Are all workloads affected or only a specific application tier?
9. Is the issue isolated to AHV, ESXi, or a specific hypervisor cluster?
10. Are both reads and writes affected?

### Recent changes

11. Was there a recent AOS, AHV, ESXi, firmware, or driver upgrade?
12. Was a node added, removed, rebooted, or placed in maintenance mode?
13. Were disks recently replaced or failed?
14. Were backup, snapshot, replication, or DR jobs running?
15. Did networking change recently: VLANs, MTU, switch firmware, LACP, routing, firewall?

### Cluster health

16. Are all CVMs healthy and reachable?
17. Are there disk, node, or cluster alerts in Prism?
18. Is the cluster rebuilding or rebalancing data?
19. Is capacity close to warning or critical thresholds?
20. Are there latency alerts or storage controller alerts?

### Performance

21. What are the VM latency, IOPS, and throughput numbers?
22. Is the bottleneck CPU, memory, storage, or network?
23. Is there a noisy-neighbor workload?
24. Is CVM CPU or memory under pressure?
25. Is the workload write-heavy, read-heavy, or latency-sensitive?

### Escalation management

26. Who owns the customer communication?
27. Has engineering been engaged?
28. Do we need a mitigation before full RCA?
29. Is there a rollback or workload migration option?
30. What is the next update commitment to the customer?

---

## 7. Likely interview questions

### Manager / leadership interview

1. Explain compute/storage convergence in simple terms.
2. Why does HCI change the way support teams troubleshoot incidents?
3. How would you manage a P1 escalation involving high VM latency on a Nutanix cluster?
4. How do you avoid siloed troubleshooting between compute, storage, virtualization, and networking teams?
5. What KPIs would you use to measure support quality in HCI escalations?
6. How would you communicate with an angry enterprise customer during a suspected storage performance issue?
7. How do you balance mitigation and root cause analysis?
8. How would you coach a support engineer who jumps to conclusions too early?
9. How would you prioritize cases when multiple enterprise customers report performance degradation?
10. How would you work with SREs, engineering, TAMs, and account teams during a critical escalation?

### Technical / SRE interview

1. What is HCI?
2. How is Nutanix different from a traditional SAN-based architecture?
3. What is the role of the CVM?
4. What is AOS?
5. What is AHV?
6. What is Prism?
7. What is data locality?
8. What happens when a node or disk fails in an HCI cluster?
9. Why can network issues appear as storage latency in HCI?
10. What metrics would you check in a VM latency case?
11. How would you distinguish between VM-level, hypervisor-level, storage-level, and network-level problems?
12. What are the risks of running a cluster near capacity?
13. What kind of issues can occur during rebuild or rebalancing?
14. What is the operational impact of convergence on upgrades and maintenance?
15. How would you explain HCI to a customer coming from VMware plus external SAN?

### Advanced / panel interview

1. A customer says, “Nutanix storage is slow.” How do you lead the escalation?
2. A node failed and now the cluster is degraded. How do you communicate risk?
3. A customer migrated from SAN to Nutanix and reports worse performance. What do you investigate?
4. A backup job causes latency every night. What is your escalation approach?
5. Engineering says the platform is healthy, but the customer still sees application impact. What do you do?
6. How would you define a good RCA for a distributed infrastructure incident?
7. How would you improve global support readiness for HCI performance escalations?
8. How would you train L1/L2 teams to identify HCI-specific symptoms earlier?

---

## 8. Model answers in English

### Q1. What is compute/storage convergence?

**Model answer:**

“Compute/storage convergence is the idea of combining compute and storage resources into the same distributed infrastructure platform. In a traditional architecture, compute hosts consume storage from an external SAN or NAS. In an HCI architecture like Nutanix, each node contributes CPU, memory, and local storage, and software aggregates those local disks into a resilient distributed storage fabric.

For Nutanix, this is fundamental. AOS provides the distributed storage services, the CVM on each node handles storage I/O and data services, and Prism gives operational visibility. The practical impact is that troubleshooting cannot be done in silos. A VM latency issue may involve the workload, hypervisor, CVM, disk subsystem, network, replication, capacity, or rebuild activity.”

### Q2. Why is HCI relevant for enterprise support?

**Model answer:**

“HCI changes enterprise support because the infrastructure layers are more integrated. That simplifies operations for customers, but it also means support teams need to triage incidents across compute, storage, networking, virtualization, and platform health together.

As a support manager, I would focus on structured triage: define business impact, isolate the scope, check recent changes, validate cluster health, review performance metrics, and coordinate the right technical owners. I would also make sure customer communication is clear: what we know, what we do not know yet, what mitigation we are pursuing, and when the next update will be provided.”

### Q3. What is the role of the Nutanix CVM?

**Model answer:**

“The CVM, or Controller VM, is the local storage controller running on each Nutanix node. It is central to the Nutanix architecture because it handles storage I/O and participates in distributed storage services across the cluster.

In practical terms, the CVM helps manage data placement, metadata, integrity, availability, snapshots, replication, and storage efficiency features. In a support escalation, CVM health is critical. If a CVM is under resource pressure, unreachable, or unhealthy, it can affect storage behavior and VM performance.”

### Q4. How would you handle a P1 escalation where a customer reports high latency on Nutanix?

**Model answer:**

“I would first stabilize the escalation process, not jump immediately to root cause. I would confirm business impact, affected workloads, severity, timeline, and any recent changes. Then I would coordinate technical triage across VM metrics, hypervisor health, CVM health, storage latency, disk or node alerts, network errors, capacity, and background operations such as rebuilds, snapshots, backups, or replication.

In parallel, I would assign clear ownership: one person leading customer communication, one or more engineers driving technical investigation, and escalation to SRE or engineering if product-level analysis is needed. I would communicate in phases: current impact, immediate mitigation options, investigation path, next update time, and later a proper RCA. The key is to reduce customer anxiety while keeping the investigation disciplined.”

### Q5. How is Nutanix different from a traditional SAN architecture?

**Model answer:**

“In a traditional SAN architecture, storage is usually provided by a dedicated external array over a storage network. Compute hosts are separate consumers of that storage. In Nutanix HCI, the storage layer is distributed across the same nodes that run the workloads. Each node contributes local storage, and AOS aggregates those resources into a shared distributed platform.

Operationally, this reduces dependency on separate storage arrays and storage teams, but it also means support has to understand the interaction between VM placement, node health, CVM behavior, disk health, network traffic, and data resilience.”

### Q6. Why can a network issue look like a storage issue in HCI?

**Model answer:**

“In HCI, storage services are distributed across nodes, so the network is part of the storage path for replication, metadata communication, remote reads or writes, rebuilds, and cluster coordination. If there are packet drops, MTU mismatches, switch congestion, or bad links, the symptom may appear as storage latency or VM performance degradation.

That is why I would not treat ‘storage latency’ as purely a disk problem. I would also check network health, interface errors, recent switch changes, VLAN or MTU issues, and whether latency correlates with replication or rebuild traffic.”

### Q7. How would you explain convergence to an enterprise customer?

**Model answer:**

“I would explain it in operational terms: instead of having separate compute servers connected to a separate storage array, Nutanix uses a cluster of nodes where each node contributes compute and storage. The platform software pools those resources and provides resilience, performance, and management.

The benefit is simpler operations and scale-out growth. The support implication is that when we troubleshoot performance or availability, we look at the cluster as an integrated system rather than separate compute and storage silos.”

### Q8. What do you need to know as a manager rather than a Senior SRE?

**Model answer:**

“I do not need to replace the Senior SRE in deep packet analysis or internal storage service debugging. But I do need to understand the architecture well enough to ask the right questions, identify weak triage, challenge premature conclusions, communicate customer risk, and coordinate the correct experts.

For example, if a case involves VM latency, I should know that CVM health, cluster capacity, disk failures, rebuilds, snapshots, replication, network quality, and workload profile are all relevant. My value is in leading the escalation end-to-end while being technically credible.”

---

## 9. Connection with my experience

Your Harmonic experience maps well to this topic if you frame it correctly.

You already have experience in:

* 24/7 enterprise support.
* Incident management.
* SLA and MTTR ownership.
* Monitoring and alerting.
* Cloud operations.
* Escalation coordination.
* Customer communication.
* Jira/Confluence/Salesforce workflows.
* Grafana/Kibana-based observability.
* Cross-functional coordination.

The bridge to Nutanix is:

| Your experience                | Nutanix/HCI translation                                    |
| ------------------------------ | ---------------------------------------------------------- |
| SaaS/cloud incident management | Distributed infrastructure incident management             |
| SLA / MTTR                     | Enterprise support KPIs for critical infrastructure cases  |
| Monitoring with Grafana/Kibana | Prism, alerts, metrics, logs, cluster health               |
| Cloud operations               | Hybrid cloud / private cloud operational model             |
| Escalation coordination        | SRE, engineering, TAM, account team, customer bridge       |
| 24/7 support leadership        | Worldwide Support operating model                          |
| Customer-impact communication  | P1/P2 enterprise escalation communication                  |
| Root cause analysis            | RCA across compute, storage, network, hypervisor, workload |

A strong interview positioning:

> “My background is in leading enterprise support operations for complex production environments. The Nutanix-specific learning curve for me is AOS, AHV, Prism, CVM, and distributed storage internals. But the escalation patterns are familiar: define impact, isolate scope, correlate metrics, manage communication, drive mitigation, and ensure a high-quality RCA. I would bring strong operational leadership while ramping quickly on Nutanix-specific architecture.”

This is honest and credible.

---

## 10. Minimum I need to memorize

You should be able to explain these from memory:

1. **HCI definition**
   HCI combines compute, storage, virtualization, and management into a software-defined distributed platform.

2. **Traditional vs HCI architecture**
   Traditional: separate compute + SAN/NAS.
   HCI: each node contributes compute and storage to a cluster.

3. **Nutanix AOS**
   The core software layer that provides distributed storage and data services.

4. **Nutanix CVM**
   Controller VM running on each node; handles storage I/O and participates in distributed storage services.

5. **Prism**
   Nutanix management and monitoring interface.

6. **AHV**
   Nutanix’s built-in hypervisor.

7. **Data locality**
   Serving I/O close to where the VM runs to reduce latency and unnecessary network hops.

8. **Why HCI affects troubleshooting**
   Compute, storage, network, hypervisor, and workload behavior are interconnected.

9. **Common escalation symptoms**

   * VM latency.
   * High storage latency.
   * Node failure.
   * Disk failure.
   * CVM issue.
   * Capacity pressure.
   * Rebuild/rebalance.
   * Backup/snapshot/replication impact.
   * Network drops.

10. **Manager-level response**
    You coordinate impact assessment, technical triage, mitigation, escalation, customer communication, and RCA.

---

## 11. Advanced / optional level

You can leave these as advanced for now:

* Internal Nutanix service names beyond CVM.
* Deep Stargate/Curator/Cassandra/Zookeeper internals.
* Erasure coding implementation details.
* Advanced metadata behavior.
* AHV scheduler internals.
* Deep VMware-on-Nutanix best practices.
* Advanced storage performance tuning.
* Specific CLI command syntax.
* Packet-level analysis.
* Failure domain design across racks/blocks/sites.
* Nutanix Files, Objects, Flow, NC2, and DR internals.

However, you should be aware of these topics because Senior SREs may mention them. Your answer can be:

> “I understand the concept and the escalation relevance, but I would rely on the SRE or engineering specialist for deep internal service analysis while making sure the case remains structured and customer-focused.”

That is acceptable for a manager role.

---

## 12. Final checklist

Before an interview, you should be able to say yes to all of these:

* [ ] I can explain HCI in under 60 seconds.
* [ ] I can compare HCI with traditional SAN-based infrastructure.
* [ ] I can explain why Nutanix uses a CVM.
* [ ] I can explain what AOS does at a high level.
* [ ] I can explain why storage and compute troubleshooting are connected in HCI.
* [ ] I can describe a P1 latency escalation in a structured way.
* [ ] I can list the first triage questions I would ask.
* [ ] I can connect HCI incidents to SLA, MTTR, RCA, and customer communication.
* [ ] I can position myself as a technical escalation manager, not a Senior SRE.
* [ ] I can admit my Nutanix-specific ramp-up areas without sounding weak.
* [ ] I can explain how my cloud/SaaS/operations experience transfers to Nutanix support.
* [ ] I can speak about business impact and technical investigation in the same answer.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword   | Meaning                                                                                                         |
| -------------------------- | --------------------------------------------------------------------------------------------------------------- |
| AHV                        | Acropolis Hypervisor; Nutanix’s built-in virtualization hypervisor.                                             |
| AOS                        | Acropolis Operating System; Nutanix software layer providing distributed storage and data services.             |
| Availability               | Ability of the platform or service to remain usable during failures or maintenance.                             |
| Backup job                 | Scheduled data protection operation that can affect performance if resource-intensive.                          |
| Capacity headroom          | Free compute, memory, storage, or network capacity available before risk thresholds.                            |
| Cluster                    | Group of Nutanix nodes operating together as one infrastructure platform.                                       |
| Compute                    | CPU and memory resources used to run workloads.                                                                 |
| Compute-only node          | Node that contributes compute capacity without contributing HCI storage in certain supported designs.           |
| Convergence                | Combining infrastructure layers, especially compute and storage, into one platform.                             |
| Controller VM              | Nutanix VM running on each node that provides local storage controller functions.                               |
| CVM                        | Controller VM; key Nutanix component handling storage I/O and data services.                                    |
| Data locality              | Serving VM data from the same host or close to where the VM runs to reduce latency.                             |
| Data placement             | Process of deciding where data and replicas are stored across the cluster.                                      |
| Data protection            | Mechanisms such as snapshots, replication, and redundancy to protect workloads.                                 |
| Deduplication              | Storage efficiency technique that removes duplicate data blocks.                                                |
| Degraded cluster           | Cluster state where one or more components have failed or resilience is reduced.                                |
| Distributed Storage Fabric | Nutanix distributed storage layer aggregating local disks into a global pool.                                   |
| DR                         | Disaster Recovery; ability to recover workloads after a site or system failure.                                 |
| DSF                        | Distributed Storage Fabric; Nutanix distributed storage architecture.                                           |
| Enterprise support         | Support model for business-critical customers with strong SLA and escalation expectations.                      |
| Erasure coding             | Storage efficiency and resiliency technique using parity rather than full replicas.                             |
| Escalation                 | Process of raising a customer issue to higher support, SRE, engineering, or leadership attention.               |
| ESXi                       | VMware hypervisor commonly used in enterprise virtualization environments.                                      |
| Failure domain             | Boundary used to understand and limit the impact of component failures.                                         |
| Garbage collection         | Background process for reclaiming unused storage and improving efficiency.                                      |
| HCI                        | Hyperconverged Infrastructure; combines compute, storage, virtualization, and management in one platform.       |
| Hotspot                    | Resource concentration causing localized performance degradation.                                               |
| Hypervisor                 | Software layer that runs and manages virtual machines.                                                          |
| I/O                        | Input/output operations between workloads and storage.                                                          |
| IOPS                       | Input/output operations per second; common storage performance metric.                                          |
| Jira                       | Work tracking system often used for incidents, escalations, bugs, and tasks.                                    |
| Latency                    | Time taken for an operation to complete; critical in storage and application performance.                       |
| LCAP / LACP                | Link Aggregation Control Protocol; network link aggregation mechanism.                                          |
| MTTR                       | Mean Time To Resolution or Recovery; key support and operations metric.                                         |
| MTU                        | Maximum Transmission Unit; packet size setting that can affect network/storage behavior.                        |
| NAS                        | Network-Attached Storage; file-level storage system accessed over the network.                                  |
| NCI                        | Nutanix Cloud Infrastructure; Nutanix platform including AOS, AHV, DR, and related infrastructure capabilities. |
| Node                       | Physical server in a Nutanix cluster contributing compute and usually storage.                                  |
| Noisy neighbor             | Workload consuming excessive shared resources and affecting others.                                             |
| NVMe                       | High-performance storage interface commonly used for fast SSDs.                                                 |
| P1                         | Priority 1 incident; usually critical production impact.                                                        |
| Prism                      | Nutanix management and monitoring interface.                                                                    |
| RCA                        | Root Cause Analysis; formal explanation of what happened, why, impact, and prevention.                          |
| Rebalance                  | Process of redistributing data or load across the cluster.                                                      |
| Rebuild                    | Process of restoring data redundancy after disk or node failure.                                                |
| Replication                | Copying data to another node, cluster, or site for resilience or disaster recovery.                             |
| Resilience                 | Ability to tolerate failures while preserving service and data integrity.                                       |
| SAN                        | Storage Area Network; block storage architecture using dedicated storage infrastructure.                        |
| Scale-out                  | Growth model where capacity/performance increases by adding nodes.                                              |
| Shared-nothing             | Architecture where each node has its own resources and coordination is handled by distributed software.         |
| SLA                        | Service Level Agreement; contractual or operational service commitment.                                         |
| Snapshot                   | Point-in-time copy used for protection, recovery, or backup workflows.                                          |
| SRE                        | Site Reliability Engineer; role focused on reliability, automation, performance, and production operations.     |
| Storage array              | Dedicated external storage system used in traditional infrastructure.                                           |
| Storage controller         | Component that manages storage I/O and data services.                                                           |
| Storage fabric             | Distributed storage layer connecting and coordinating storage resources.                                        |
| Throughput                 | Amount of data transferred over time, usually measured in MB/s or GB/s.                                         |
| Triage                     | Structured process of assessing impact, scope, symptoms, and likely cause.                                      |
| VM                         | Virtual Machine; software-based server running on a hypervisor.                                                 |
| VMware                     | Enterprise virtualization platform commonly used with Nutanix in some environments.                             |
| Workload                   | Application, VM, service, or process running on the infrastructure.                                             |

[1]: https://www.nutanix.com/hyperconverged-infrastructure?utm_source=chatgpt.com "What is Hyperconverged Infrastructure (HCI) - FAQs | Nutanix"
[2]: https://portal.nutanix.com/page/documents/details?targetId=vSphere-Admin6-AOS-v7_0%3AvSphere-Admin6-AOS-v7_0&utm_source=chatgpt.com "AOS 7.0 - vSphere Administration Guide for AOS - portal.nutanix.com"
[3]: https://portal.nutanix.com/page/documents/details?targetId=Advanced-Admin-AOS%3Aapp-nutanix-cloud-infra-cvm-field-specifications-c.html&utm_source=chatgpt.com "AOS 7.5 - Controller VM (CVM) Specifications - Nutanix"
[4]: https://www.nutanix.com/together/infrastructure?utm_source=chatgpt.com "Nutanix | Your infrastructure. All together now."
[5]: https://www.nutanix.com/library/datasheets/nci?utm_source=chatgpt.com "Nutanix Cloud Infrastructure (NCI)"
[6]: https://portal.nutanix.com/page/documents/details?targetId=vSphere-Admin6-AOS-v6_0%3Avsp-cluster-introduction-vsphere-c.html&utm_source=chatgpt.com "AOS 6.0 - Overview - Nutanix"
[7]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2096-Nutanix-Core-Performance%3Aaos-storage.html&utm_source=chatgpt.com "AOS Storage - Nutanix"
[8]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v11_0%3Aahv-create-cluster-ahv-compute-node-hci-node-t.html&utm_source=chatgpt.com "Expanding the Hyperconverged Infrastructure Cluster with AHV Compute Nodes"
