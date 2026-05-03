# AOS / Storage — Metadata Services

## 1. Short definition

In Nutanix AOS storage, **metadata services** are the distributed control-plane components that track **where data is, how it is protected, how it maps to VMs/vDisks, and what background maintenance is required**.

For interview purposes, say it this way:

> Nutanix metadata services are the distributed services that allow AOS to behave like a single resilient storage system while data is actually spread across many nodes, disks, Controller VMs, and replicas.

The key services to understand are:

* **Cassandra** — distributed metadata store.
* **Medusa** — access layer in front of Cassandra.
* **Zookeeper / Zeus** — cluster configuration, membership, and coordination.
* **Curator** — background scanning, cleanup, repair, balancing, and optimization.
* **Stargate** — data I/O service that consumes metadata to read/write data correctly.

Nutanix documentation describes AOS storage as scale-out storage where I/O is handled locally through the Controller VM, while the distributed system abstracts that complexity from the hypervisor and workloads. 

---

## 2. Clear explanation

A Nutanix cluster is not a traditional storage array with a pair of controllers and a central metadata database. It is a **distributed storage fabric**. Each node contributes compute, storage devices, and a **Controller VM**, also called a **CVM**.

The hypervisor sees storage through standard protocols such as **iSCSI, NFS, or SMBv3**, depending on the hypervisor and use case. Behind that, the CVM and AOS handle the distributed storage logic. Nutanix’s AOS Storage documentation explains that communication between the hypervisor and Nutanix occurs through the local CVM interface, and that Stargate inside the CVM handles storage I/O requests and interaction with other CVMs and physical devices. 

The important part is this:

> Data is distributed, replicated, moved, cached, repaired, and optimized across the cluster. Metadata is what allows AOS to know what exists, where it lives, who owns it, whether it is healthy, and what needs to happen next.

A simplified flow:

1. A VM issues a read or write.
2. The hypervisor sends the I/O to the local CVM.
3. **Stargate** receives the I/O.
4. Stargate uses metadata to understand the vDisk, extents, replicas, and placement.
5. Data is written locally and/or replicated to other CVMs depending on the protection policy.
6. **Cassandra / Medusa** maintain distributed metadata.
7. **Curator** scans metadata and the extent store to identify cleanup, rebalancing, under-replication, corruption, or optimization tasks.
8. **Zookeeper / Zeus** maintain cluster configuration and service coordination.

A useful mental model:

| Layer            | Practical role                                               |
| ---------------- | ------------------------------------------------------------ |
| Stargate         | “Handles the I/O.”                                           |
| Cassandra        | “Stores distributed metadata.”                               |
| Medusa           | “Provides access to Cassandra metadata.”                     |
| Zookeeper / Zeus | “Keeps cluster configuration and coordination consistent.”   |
| Curator          | “Runs background housekeeping and repair based on metadata.” |
| Prism / NCC      | “Surfaces health, alerts, and checks to operators/support.”  |

Nutanix documentation describes Curator as a process running on each node, with a Curator master periodically scanning the metadata database and identifying cleanup and optimization tasks for Stargate or other components; analysis is distributed across Curator nodes using MapReduce. ([portal.nutanix.com][1])

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Worldwide Support Manager**, metadata services matter because many critical customer escalations are not just “VM is slow” or “disk failed.” They are really questions about:

* **Data availability** — Is the data protected?
* **Replica health** — Are objects under-replicated?
* **Cluster resiliency** — Can the cluster tolerate another failure?
* **Performance** — Are metadata or background tasks contributing to latency?
* **Upgrade safety** — Are core services healthy before an AOS/LCM operation?
* **Customer confidence** — Can support explain what is happening without creating panic?

You do **not** need to debug Cassandra internals like a senior storage engineer. But you do need to understand enough to manage escalations intelligently:

> Is this a data-path issue, a metadata/control-plane issue, a background maintenance issue, or an infrastructure issue affecting these services?

That distinction is managerial gold in an interview.

A strong manager-level answer would be:

> “I would not treat metadata service alerts as isolated service failures. I would look at customer impact, data protection state, cluster health, recent changes, disk/node events, Stargate health, Curator activity, and whether the issue is affecting I/O, resiliency, or only background optimization. Then I would coordinate SRE, engineering, and customer communication around risk and impact.”

That positions you correctly as a **technical escalation manager**, not as an individual contributor pretending to be a Cassandra specialist.

---

## 4. Key concepts

### 4.1 Metadata vs data

**Data** is the actual VM content: guest OS blocks, application data, files, database pages, and so on.

**Metadata** is the map and state information needed to manage that data:

* Which vDisk exists.
* Which extents belong to that vDisk.
* Which extent groups contain the data.
* Where replicas are located.
* Which node/disk owns or serves a piece of data.
* Whether objects are healthy, stale, corrupt, or under-replicated.
* What background work is required.

AOS storage uses structures such as **storage pools**, **containers**, **vDisks**, **vBlocks**, **extents**, and **extent groups**. Nutanix documentation describes a vDisk as being logically composed of vBlocks, which map to extents stored as extent groups. 

### 4.2 Cassandra — distributed metadata store

In Nutanix architecture, **Cassandra** is the distributed database used to store cluster metadata. Nutanix documentation describes Cassandra as a distributed database that stores metadata about guest VM data stored in a Nutanix datastore. ([portal.nutanix.com][2])

Practical meaning:

* It is not customer application data.
* It is not “just logs.”
* It is critical for AOS to know where VM data lives.
* It runs distributed across the cluster.
* If metadata health is degraded, support must take it seriously.

Interview phrase:

> “Cassandra is part of the storage control plane. If it is unhealthy, the concern is not only performance; it may affect the platform’s ability to track and manage data placement, protection, and cluster state.”

### 4.3 Medusa — metadata access layer

**Medusa** is commonly described as the access interface or abstraction layer in front of Cassandra.

Practical meaning:

* Other AOS services should not need to talk directly to Cassandra internals.
* Medusa provides the service layer for metadata access.
* If Medusa/Cassandra has issues, Stargate and other services may be impacted indirectly.

Interview phrase:

> “I think of Medusa as the service interface to the distributed metadata store. For support triage, I care about whether metadata access is healthy and whether dependent services like Stargate are affected.”

### 4.4 Zookeeper / Zeus — cluster configuration and coordination

**Zookeeper** manages cluster configuration, membership, and coordination. Nutanix documentation states that Zeus is the library other components use to access cluster configuration, implemented using Apache Zookeeper; Zookeeper runs on three or five nodes depending on redundancy factor, using an odd number to avoid split-brain-style tie problems. ([portal.nutanix.com][2])

Practical meaning:

* Who is in the cluster?
* Which services are active?
* Which nodes are members?
* Which component is leader/master?
* What is the current cluster configuration?

Interview phrase:

> “If Cassandra is the metadata database, Zookeeper/Zeus is closer to the cluster coordination and configuration layer.”

### 4.5 Curator — background metadata scanning and repair

**Curator** is the background maintenance engine.

It scans metadata, detects cleanup or optimization needs, and helps maintain storage health. Nutanix documentation says Curator responsibilities include ensuring file system metadata consistency and scanning the extent store for corrupt or under-replicated data. ([portal.nutanix.com][3])

Curator is relevant during:

* Disk failures.
* Node failures.
* Re-replication.
* Data balancing.
* Garbage collection.
* Metadata cleanup.
* Post-upgrade health.
* Capacity and tiering maintenance.

Nutanix AOS storage documentation explains that, after a node failure, a Curator scan finds data previously hosted on the failed node and its replicas; then all nodes participate in reprotection. 

### 4.6 Stargate — data path service that relies on metadata

**Stargate** handles storage I/O. It is not normally described as a “metadata service,” but it depends on metadata to serve data correctly.

Nutanix documentation states that Stargate handles storage I/O requests from user VMs and persistence, including replication factor behavior. 

Practical escalation point:

* If Stargate is unhealthy, customer I/O may be affected.
* If Cassandra/Medusa metadata access is unhealthy, Stargate behavior may be indirectly affected.
* If Curator is busy, customers may see background load.
* If a CVM is unavailable, I/O may be redirected to other CVMs, potentially increasing latency. Nutanix documentation describes CVM failure handling where I/O is redirected to other CVMs, with potential latency increase because I/O goes over the network. 

### 4.7 Metadata drive / disk health

Older Nutanix documentation notes that Cassandra uses metadata disks for fast metadata read/write responses and that these uses are designed for failure through replication to other metadata disks in the cluster. ([portal.nutanix.com][4])

For interview purposes, avoid overclaiming specific hardware details unless you know the platform generation. But the principle is useful:

> Metadata availability depends on distributed redundancy, but disk/node failures can still trigger service alerts, repair activity, and customer concern.

---

## 5. How it appears in a real escalation

### Scenario 1 — Customer reports VM latency after disk failure

Customer says:

> “After a disk failed, several business-critical VMs became slower. Prism shows warnings. Is our data safe?”

What may be happening:

* A disk failed.
* AOS detected under-replicated data.
* Curator is scanning metadata and the extent store.
* The cluster is re-replicating data.
* Background repair competes for resources.
* Stargate continues serving I/O, but latency may increase depending on cluster load and remaining capacity.

Support-manager response:

> “First, I would separate data availability from performance impact. I would check whether the cluster is still within fault tolerance, whether any data is under-replicated, whether Curator/reprotection tasks are active, and whether Stargate or CVM services are healthy. Then I would communicate the current risk, expected repair behavior, and any actions the customer should avoid, such as additional maintenance or disruptive operations until resiliency is restored.”

### Scenario 2 — Metadata service alert after node/CVM instability

Customer says:

> “We see Cassandra/Medusa/Zookeeper alerts after a CVM reboot. Are we at risk?”

Possible cause:

* CVM went down or restarted.
* Metadata services restarted or temporarily lost quorum/peer communication.
* Cluster may be recovering.
* I/O may have been redirected to remote CVMs.
* Need to distinguish transient restart from persistent metadata degradation.

Support-manager response:

> “I would check whether this is transient after a CVM event or persistent across multiple nodes. I would validate cluster health, service status, quorum/coordination health, Cassandra/Medusa state, Stargate status, and whether any customer I/O impact is ongoing. Communication should be calm but precise: what happened, current customer impact, current data protection state, and next technical actions.”

### Scenario 3 — Upgrade blocked by metadata health checks

Customer says:

> “LCM/AOS upgrade cannot proceed because cluster health checks fail.”

Possible cause:

* Core services unhealthy.
* Curator tasks pending.
* Cassandra/metadata service warnings.
* Under-replicated data.
* NCC check failure.
* Capacity/resiliency issue.

Support-manager response:

> “I would not force an upgrade over metadata or storage-health warnings. I would treat the pre-check failure as a risk control. The priority is to restore cluster health, resolve under-replication or metadata-service issues, rerun NCC/LCM checks, and only then proceed.”

### Scenario 4 — Customer asks if Nutanix has a single metadata bottleneck

Good answer:

> “No, the architecture is distributed. Metadata is stored and managed across the cluster using distributed services, while I/O is served through CVMs and Stargate. That said, distributed does not mean magically immune: quorum, service health, network connectivity, disk health, and capacity still matter.”

---

## 6. Triage questions I should ask

Use these in interviews and real escalation simulations.

### Customer impact

1. Which workloads are affected?
2. Is the issue read latency, write latency, failed I/O, VM stun, or management-plane alert only?
3. Is the impact cluster-wide, node-specific, VM-specific, datastore/container-specific, or time-bound?
4. When did it start?
5. Was there a recent change: upgrade, LCM, firmware, network change, node reboot, disk replacement, migration, backup job?

### Cluster health

6. Are all CVMs up?
7. Are Stargate services healthy?
8. Are Cassandra/Medusa services healthy?
9. Is Zookeeper/Zeus healthy?
10. Is Curator running scans or repair tasks?
11. Are there NCC failures?
12. Is Prism showing critical alerts?

### Data protection

13. Is the cluster under-replicated?
14. Is RF2/RF3 protection satisfied?
15. Has fault tolerance been reduced?
16. Is re-protection in progress?
17. Are there disk, node, or block/rack failures?

### Performance and capacity

18. Is latency correlated with Curator, re-replication, snapshots, backups, or ILM?
19. Is SSD tier, HDD tier, or metadata disk capacity constrained?
20. Is the cluster close to capacity thresholds?
21. Are there hot nodes, hot disks, or skewed workloads?

### Network / distributed-system angle

22. Any packet loss or latency between CVMs?
23. Any ToR switch, VLAN, MTU, bond, NIC, or RDMA issue?
24. Are CVMs communicating normally?
25. Is the issue isolated to one rack/block/availability domain?

### Escalation management

26. What is the customer’s business impact?
27. What is the SLA/severity?
28. What has already been attempted?
29. Is engineering engagement required?
30. What is the next customer update time?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you manage a critical escalation involving Nutanix metadata service alerts?
2. How do you explain a complex distributed storage issue to an enterprise customer?
3. How do you balance technical investigation with customer communication?
4. What KPIs would you track in a support team handling storage/SRE escalations?
5. How would you decide when to escalate to engineering?
6. How do you prevent repeated escalations caused by the same underlying platform issue?
7. How would you coach engineers who are strong technically but weak in customer communication?
8. What is your approach when the customer asks, “Is my data safe?”

### Technical / SRE interview

9. What is the role of Cassandra in Nutanix?
10. What is Curator responsible for?
11. What is Stargate?
12. How does AOS continue serving I/O if a CVM is unavailable?
13. What is the difference between data path and metadata/control plane?
14. Why does metadata health matter in a distributed storage system?
15. What would you check after a disk failure?
16. What would you check before an AOS upgrade?
17. How would you troubleshoot latency after a node failure?
18. What role does Zookeeper play?

### Panel / escalation case

19. A customer has under-replicated data and high latency. How do you run the bridge?
20. A customer wants to continue an upgrade despite failed health checks. What do you do?
21. Engineering says the issue is fixed in a future release, but the customer needs mitigation now. How do you handle it?
22. A customer is angry because support keeps asking for logs. How do you restore confidence?

---

## 8. Model answers in English

### Q1. What are metadata services in Nutanix AOS?

**Model answer:**

> In Nutanix AOS, metadata services are the distributed services that track the relationship between VMs, vDisks, extents, replicas, placement, and cluster state. The actual data is distributed across nodes and disks, but AOS needs metadata to know where that data is, how it is protected, and how it should be served or repaired. At a high level, Cassandra stores distributed metadata, Medusa provides access to it, Zookeeper and Zeus manage cluster configuration and coordination, Curator performs background scans and maintenance, and Stargate uses that information to serve storage I/O.

### Q2. Why is Cassandra important in Nutanix?

**Model answer:**

> Cassandra is important because it is part of the distributed metadata layer. It stores metadata about the guest VM data managed by AOS. If Cassandra or metadata access is unhealthy, the concern is not simply a database process being down; it may affect how the cluster tracks data placement, protection, and storage state. As a support manager, I would treat Cassandra alerts as potentially high-risk until we validate customer impact, service health, replication state, and cluster resiliency.

### Q3. What is Curator?

**Model answer:**

> Curator is the AOS background maintenance service. It scans metadata and storage structures to identify work such as cleanup, optimization, balancing, consistency checks, and repair or re-protection after failures. In an escalation, Curator activity can be very relevant because after a disk or node failure it may drive the work needed to restore full resiliency, and that background activity can also correlate with temporary performance impact.

### Q4. What is the difference between Stargate and Cassandra?

**Model answer:**

> Stargate is primarily in the data path: it handles storage I/O from user VMs and manages persistence and replication behavior. Cassandra is part of the metadata layer: it stores distributed metadata that allows AOS to know where data is and how it is organized. In troubleshooting, I would ask whether the issue is affecting the data path directly, such as Stargate latency or failure, or whether it is a metadata/control-plane problem that may indirectly affect storage operations.

### Q5. How would you handle an escalation where Prism reports metadata service errors?

**Model answer:**

> I would first establish customer impact: whether VMs are experiencing failed I/O, latency, or whether the alert is currently management-plane only. Then I would check cluster health, CVM and Stargate status, Cassandra/Medusa health, Zookeeper/Zeus health, Curator activity, NCC results, recent changes, and any disk or node failures. From a management perspective, I would set a clear severity, assign technical owners, define the next customer update time, and communicate in terms of impact, risk, mitigation, and next steps rather than internal component noise.

### Q6. A disk failed and now Curator is active. What do you tell the customer?

**Model answer:**

> I would explain that AOS is designed to tolerate failures through distributed replication, but after a disk failure the cluster may need to scan metadata and re-protect affected data. Curator helps identify and coordinate that background work. The key points for the customer are whether data remains available, whether fault tolerance is temporarily reduced, whether re-protection is progressing, and whether performance impact is expected during the repair window. I would also advise avoiding additional disruptive maintenance until the cluster returns to a healthy protected state.

### Q7. How do you know whether it is a data-path issue or metadata issue?

**Model answer:**

> I would look at symptoms and scope. If VMs show high latency or failed I/O, Stargate, CVM health, network between CVMs, disks, and hypervisor paths become central. If the issue is about cluster state, placement, under-replication, service coordination, or Curator/NCC checks, the metadata and control-plane services become more relevant. In practice, these areas can overlap, so I would correlate Prism alerts, NCC output, service status, logs, recent events, and performance metrics.

### Q8. How would you position yourself as a manager rather than a Senior SRE?

**Model answer:**

> I would be technical enough to structure the investigation, ask the right questions, challenge assumptions, and communicate clearly with the customer, but I would not try to replace the senior SRE doing deep log analysis. My value is in coordinating the escalation: aligning support, SRE, engineering, and the customer; managing severity and communication; ensuring evidence is collected; avoiding unsafe actions; and driving the case to resolution and post-incident learning.

### Q9. What would you check before allowing an AOS upgrade?

**Model answer:**

> I would want the cluster to be healthy before the upgrade. That means no unresolved critical storage alerts, no ongoing under-replication, healthy CVMs and core services, successful NCC or LCM pre-checks, acceptable capacity headroom, and a clear rollback or mitigation plan. If metadata or storage-health checks fail, I would not recommend forcing the upgrade until the risk is understood and remediated.

### Q10. How would you explain metadata to a non-technical customer executive?

**Model answer:**

> I would compare metadata to the control map of the storage system. The customer data is the content, but metadata tells the platform where each piece is, how many protected copies exist, and what needs to be repaired or moved. In a distributed system like Nutanix, that map has to be consistent and resilient across the cluster. Our investigation is focused on confirming both service availability and the integrity of that control map.

---

## 9. Connection with my experience

Your Harmonic background maps well to this topic if you frame it correctly.

You already have experience with:

* 24/7 operations.
* Incident management.
* SLA and severity handling.
* MTTR reduction.
* Escalation bridges.
* Monitoring and alert correlation.
* Cloud/SaaS operations.
* Customer communication.
* Cross-functional coordination.
* Jira/Confluence/Salesforce-style operational workflows.

The bridge to Nutanix:

| Your experience                  | Nutanix equivalent positioning                              |
| -------------------------------- | ----------------------------------------------------------- |
| SaaS incident management         | Cluster/customer-impact triage                              |
| Monitoring with Grafana/Kibana   | Prism/NCC/log/metric correlation                            |
| SLA/MTTR ownership               | Severity management and restoration planning                |
| Cloud operations                 | Distributed infrastructure thinking                         |
| Kubernetes/Linux troubleshooting | Service health, node health, logs, resource pressure        |
| Escalation management            | Coordinating support, SRE, engineering, TAM/customer        |
| Customer communications          | Explaining data protection, risk, and next actions          |
| Team coaching                    | Developing engineers who can triage storage/platform issues |

A strong personal positioning statement:

> “My background is in managing enterprise operations where service health, customer impact, escalation discipline, and clear communication are critical. The Nutanix-specific layer I am building is the AOS/HCI vocabulary: CVMs, Stargate, Cassandra, Curator, Zookeeper, Prism, NCC, replication, and data locality. I do not need to be the deepest storage engineer in the room, but I need to understand the architecture well enough to lead the escalation, ask the right questions, protect the customer, and drive the right technical resources.”

---

## 10. Minimum I need to memorize

Memorize this core map:

| Component                          | Minimum meaning                                                      |
| ---------------------------------- | -------------------------------------------------------------------- |
| **AOS**                            | Nutanix distributed storage and platform software.                   |
| **CVM**                            | Controller VM running Nutanix storage/control services on each node. |
| **Stargate**                       | Handles storage I/O.                                                 |
| **Cassandra**                      | Distributed metadata store.                                          |
| **Medusa**                         | Access layer/interface to Cassandra metadata.                        |
| **Zookeeper**                      | Cluster coordination and configuration.                              |
| **Zeus**                           | Nutanix access library/interface for Zookeeper configuration.        |
| **Curator**                        | Background scan, cleanup, repair, balancing, optimization.           |
| **Prism**                          | Management UI/API for cluster visibility.                            |
| **NCC**                            | Nutanix Cluster Check, used for health validation.                   |
| **RF2/RF3**                        | Replication Factor: number of data copies/protection level.          |
| **Under-replicated**               | Data does not currently have the expected number of replicas.        |
| **Re-protection**                  | Background process to restore resiliency after a failure.            |
| **Data path**                      | Actual I/O serving path.                                             |
| **Control plane / metadata plane** | Services that manage state, placement, coordination, and metadata.   |

One verbal explanation to memorize:

> “In Nutanix AOS, VM data is distributed across the cluster. Metadata services maintain the map and state of that data. Cassandra stores distributed metadata, Medusa provides access to it, Zookeeper/Zeus handle cluster configuration and coordination, Curator scans and repairs/optimizes in the background, and Stargate uses that information to serve I/O. In support escalations, metadata health matters because it affects data protection, resiliency, repair operations, upgrade readiness, and sometimes performance.”

---

## 11. Advanced / optional level

You can leave these as advanced unless the interviewer goes deep:

* Cassandra ring internals.
* Paxos/consensus implementation details.
* Specific Cassandra table structures.
* Detailed Medusa API behavior.
* Low-level Curator MapReduce internals.
* Exact log file paths and commands.
* Specific Genesis/service-control commands.
* BlockStore/SPDK implementation details.
* RDMA data path internals.
* EC-X stripe/parity mechanics.
* Deep AHV iSCSI multipathing behavior.
* Forensic analysis of Stargate logs.

However, you should be comfortable saying:

> “I would rely on senior SREs for deep component-level log analysis, but I understand the function of each service, the risks, the triage flow, and the customer communication required.”

That is the right balance for the Manager role.

---

## 12. Final checklist

Before an interview, be able to explain:

* What metadata is.
* Why distributed storage needs metadata services.
* The difference between **Stargate**, **Cassandra**, **Curator**, and **Zookeeper**.
* How a disk/node/CVM failure can trigger metadata scans and re-protection.
* Why Curator activity can matter during performance escalations.
* Why metadata health matters before upgrades.
* How to communicate “data safe but resiliency degraded” versus “active customer impact.”
* How to lead an escalation without pretending to be the deepest SRE.
* How to connect your 24/7 support experience to distributed platform operations.
* How to ask triage questions around customer impact, cluster health, data protection, performance, and recent changes.

Your strongest interview framing:

> “For this role, I see metadata services as a critical part of the support manager’s mental model. They are not just internal components; they determine how confidently we can answer customer questions about availability, resiliency, performance, and risk during incidents.”

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword   | Meaning                                                                                  |
| -------------------------- | ---------------------------------------------------------------------------------------- |
| AHV                        | Acropolis Hypervisor; Nutanix native hypervisor.                                         |
| AOS                        | Acropolis Operating System; Nutanix distributed platform and storage software.           |
| AOS Storage                | Nutanix scale-out distributed storage layer.                                             |
| Availability               | Ability of workloads/data to remain accessible during failures.                          |
| BlockStore                 | Nutanix storage layer capability for efficient block/device handling.                    |
| Cassandra                  | Distributed metadata store used by Nutanix AOS.                                          |
| Cluster                    | Group of Nutanix nodes operating as one platform.                                        |
| Container                  | Logical storage segmentation in AOS, often mapped to a datastore.                        |
| Control plane              | Services managing configuration, state, metadata, and coordination.                      |
| Controller VM              | VM on each node running Nutanix storage and control services.                            |
| Curator                    | AOS background service for scans, cleanup, optimization, and repair.                     |
| CVM                        | Controller VM.                                                                           |
| Data locality              | Keeping or moving data close to the VM consuming it.                                     |
| Data path                  | Path used to serve actual read/write I/O.                                                |
| Datastore                  | Storage target presented to the hypervisor.                                              |
| Disk failure               | Physical or logical disk loss that may trigger re-protection.                            |
| Distributed Storage Fabric | Nutanix distributed storage architecture across nodes.                                   |
| EC-X                       | Erasure Coding feature for storage efficiency using parity.                              |
| Extent                     | Logical piece of data within AOS storage.                                                |
| Extent group               | Physical storage unit containing extents on disk.                                        |
| Fault tolerance            | Ability to survive disk/node/block/rack failures.                                        |
| Genesis                    | Nutanix service framework that starts and monitors services.                             |
| HCI                        | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in nodes. |
| I/O                        | Input/output; read and write operations.                                                 |
| ILM                        | Information Lifecycle Management; movement of data across tiers based on usage.          |
| iSCSI                      | Internet Small Computer Systems Interface; block storage protocol.                       |
| LCM                        | Life Cycle Manager; Nutanix lifecycle/upgrade management.                                |
| Medusa                     | Metadata access layer/interface in front of Cassandra.                                   |
| Metadata                   | Data about data: mappings, placement, ownership, replicas, and state.                    |
| Metadata service           | Service involved in storing, accessing, coordinating, or scanning metadata.              |
| MTTR                       | Mean Time To Repair/Restore; operational recovery metric.                                |
| NCC                        | Nutanix Cluster Check; health-check toolset.                                             |
| NFS                        | Network File System; file storage protocol.                                              |
| Node                       | Physical Nutanix server in a cluster.                                                    |
| OpLog                      | Write component used for fast random write handling.                                     |
| Prism                      | Nutanix management interface and API layer.                                              |
| Quorum                     | Minimum number of members required for safe distributed-system decisions.                |
| Re-protection              | Restoring full data protection after a failure or under-replication.                     |
| Replica                    | Protected copy of data stored on another disk/node.                                      |
| Replication Factor         | Number of data copies maintained for protection.                                         |
| RF2                        | Replication Factor 2; two copies of data.                                                |
| RF3                        | Replication Factor 3; three copies of data.                                              |
| SLA                        | Service Level Agreement.                                                                 |
| SMBv3                      | Server Message Block version 3; file storage protocol.                                   |
| SPDK                       | Storage Performance Development Kit; user-space storage framework.                       |
| Stargate                   | Nutanix service responsible for storage I/O handling.                                    |
| Storage pool               | Group of physical storage devices across Nutanix nodes.                                  |
| Under-replicated           | Data currently has fewer replicas than required.                                         |
| vBlock                     | Logical chunk of vDisk address space.                                                    |
| vDisk                      | Virtual disk/file managed by AOS storage.                                                |
| VM                         | Virtual Machine.                                                                         |
| Zookeeper                  | Distributed coordination/configuration service used in Nutanix architecture.             |
| Zeus                       | Nutanix library/interface for accessing Zookeeper-backed cluster configuration.          |

[1]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_5%3Aarc-cluster-components-c.html?utm_source=chatgpt.com "Prism 7.5 - Cluster Components - Nutanix"
[2]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_0%3Aapp-about-nutanix-complete-cluster-c.html&utm_source=chatgpt.com "Prism 6.0 - Nutanix Platform Overview"
[3]: https://portal.nutanix.com/page/documents/details?targetId=Nutanix-Clusters-AWS%3Aaws-clusters-aws-resolving-bad-disk-resources-c.html&utm_source=chatgpt.com "Cloud Clusters (NC2) Hosted - Resolving Bad Disk Resources - Nutanix"
[4]: https://portal.nutanix.com/page/documents/details?targetId=Web_Console_Guide-NOS_v4_1%3Aarc_metadata_drive_failure_c.html&utm_source=chatgpt.com "NOS 4.1 - Metadata Drive Failure - portal.nutanix.com"
