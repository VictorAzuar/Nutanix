# AOS / Storage — Distributed Storage Fabric

## 1. Short definition

**Distributed Storage Fabric (DSF)** is the core Nutanix AOS storage layer. It aggregates local disks from all nodes in a Nutanix cluster and presents them to the hypervisor as a single resilient, high-performance datastore, while keeping I/O distributed and local whenever possible. Nutanix describes DSF as the “heart” of AOS and the software-defined storage technology that pools drives from every node into a unified storage pool. ([Nutanix][1])

In interview language:

> DSF is Nutanix’s distributed storage engine. It replaces the traditional SAN/NAS model by pooling local node storage across the cluster, protecting data through replication and self-healing, and serving VM I/O locally through the Controller VM.

---

## 2. Clear explanation

In a traditional virtualization environment, you often have:

* Compute hosts running VMs.
* External SAN/NAS storage.
* A storage network between them.
* Separate storage controllers, LUNs, RAID groups, multipathing, zoning, and array-level troubleshooting.

Nutanix changes this model. Each node contributes compute and local storage. A **Controller VM**, or **CVM**, runs on each node and provides the Nutanix storage intelligence. AOS runs as the operating system of the CVM and provides data protection, space efficiency, scalability, automated tiering, and other data services. ([Nutanix][2])

DSF then pools SSD, NVMe, and HDD resources across the cluster and presents storage to the hypervisor as if it were centralized storage. Nutanix documentation explains that DSF exposes storage through protocols such as SMB, NFS, and SCSI, without a single point of failure; the hypervisor sees something similar to centralized storage, but I/O is managed locally for performance. ([Nutanix][2])

The key idea is:

> The hypervisor sees shared storage.
> The physical data path is distributed.
> The system tries to serve reads locally.
> Writes are protected across nodes.
> Failures trigger automatic healing.

The main logical layers are:

| Layer                              | Practical meaning                                                                                                                                     |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Physical disks**                 | SSD, NVMe, HDD devices inside Nutanix nodes.                                                                                                          |
| **Storage Pool**                   | Logical pool of physical storage devices across the cluster.                                                                                          |
| **Container**                      | Logical segmentation of the storage pool where policies such as replication factor, compression, deduplication, and erasure coding can be configured. |
| **vDisk**                          | A VM disk or file stored on DSF.                                                                                                                      |
| **vBlock / Extent / Extent Group** | Internal data units used by DSF to map, distribute, replicate, optimize, and move data.                                                               |

Nutanix’s AOS guide describes a storage pool as a logical pool of physical SSD/HDD devices that can span multiple nodes, and a container as the logical segmentation where features such as redundancy factor are configured and applied to VMs or files. ([Nutanix][2])

The important operational concepts are:

**Data locality**
DSF tries to keep VM data on the same node where the VM is running. This reduces read latency and network traffic. If a VM moves because of vMotion, live migration, or HA, the data can be moved afterward to restore locality. Nutanix explicitly describes data locality as a mechanism to reduce network reads and optimize performance. ([Nutanix][2])

**Replication factor**
Instead of traditional RAID as the primary protection mechanism, Nutanix stores multiple copies of data across different nodes. Nutanix’s public AOS FAQ says RF2 keeps at least two copies of data on different nodes, so if a node or disk fails, data remains available from a replica. ([Nutanix][1])

**Self-healing**
When a disk or node fails, AOS automatically starts re-replicating data to restore the desired protection level. This is crucial in enterprise support because the manager needs to understand whether the platform is degraded, healing, or at risk. Nutanix describes this self-healing process as automatic background re-replication after failures. ([Nutanix][1])

**Tiering and balancing**
DSF can move hot data to faster media and cold data to lower tiers. Nutanix documentation describes Intelligent Tiering as the automatic movement of frequently accessed data to SSD and less frequently accessed data to HDD, and Automatic Disk Balancing as a mechanism that distributes data uniformly across the cluster. ([Nutanix][2])

**Optimization services**
Compression, deduplication, and erasure coding improve storage efficiency. Nutanix’s AOS material describes DSF as supporting deduplication, compression, and erasure coding for capacity optimization. ([Nutanix][2])

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, DSF matters because many critical customer escalations ultimately touch storage behavior, even when the first symptom appears elsewhere.

A customer may report:

* “VMs are slow.”
* “Latency increased after a node failure.”
* “We lost a disk and now the cluster is degraded.”
* “Prism shows storage alerts.”
* “Capacity is almost full.”
* “Backups or snapshots are consuming too much space.”
* “After migration, performance is worse.”
* “The customer is blaming Nutanix, VMware, the network, or the application team.”

As a support manager, you do not need to debug every extent map personally, but you must understand the DSF mental model well enough to:

* Ask the right triage questions.
* Separate performance, capacity, availability, and configuration issues.
* Understand the risk level during a degraded state.
* Communicate clearly with customer executives.
* Coordinate SREs, engineering, account teams, and customer infrastructure teams.
* Avoid misleading promises during re-replication, rebuild, or capacity pressure.
* Explain why an issue may be storage-related, hypervisor-related, network-related, or application-related.

Strong interview positioning:

> For this role, I do not need to present myself as the deepest storage engineer in the room. I need to show that I understand the Nutanix storage architecture well enough to lead an escalation, ask precise questions, protect customer trust, and coordinate the right technical resources.

---

## 4. Key concepts

### AOS

**AOS** is the core Nutanix software layer that runs inside the Controller VM and provides the storage, data services, management, and platform functionality. Nutanix describes AOS as the base operating system and data plane that encapsulates storage, compute, security, and network runtime, running with AHV or other hypervisors such as VMware ESXi and Microsoft Hyper-V. ([Nutanix][2])

Interview phrase:

> AOS is the Nutanix software layer that provides the distributed storage and data services underneath the workloads.

---

### CVM

The **Controller VM** is the virtual machine running Nutanix services on each node. It is central to the data path. If the CVM has issues, storage I/O and cluster services may be affected.

Interview phrase:

> The CVM is the local storage controller for each node, but collectively the CVMs form a distributed storage system.

---

### DSF

**Distributed Storage Fabric** is the distributed storage layer of AOS. It pools physical drives across the cluster and exposes datastore-like storage to the hypervisor.

Nutanix’s AOS guide states that DSF pools flash and HDD storage across the cluster and presents it as a datastore to the hypervisor with no single point of failure. ([Nutanix][2])

Interview phrase:

> DSF gives the hypervisor the abstraction of shared storage while internally distributing, replicating, balancing, and optimizing data across nodes.

---

### Storage Pool

A **Storage Pool** is the logical aggregation of physical disks across nodes. In most configurations, one storage pool is enough. Nutanix documentation says storage pools can span multiple Nutanix nodes and expand with the cluster. ([Nutanix][2])

Support relevance:

* Disk failures.
* Capacity imbalance.
* Node expansion.
* Cluster scaling.
* Storage pool health alerts.

---

### Container

A **Container** is a logical segment of the storage pool. It contains VMs or vDisks and is where many storage policies are configured. Nutanix documentation describes containers as logical segmentations where features like redundancy factor are configured at container level and applied at VM or file level. ([Nutanix][2])

Support relevance:

* RF2/RF3 configuration.
* Compression.
* Deduplication.
* Erasure coding.
* Capacity and performance policy.

---

### vDisk

A **vDisk** is a virtual disk stored on DSF, such as a VM disk. Nutanix documentation describes a vDisk as a file larger than 512 KB on DSF, including VMDK and VM hard disks. ([Nutanix][2])

Support relevance:

* VM-level performance.
* Disk latency.
* Snapshot impact.
* Corruption suspicion.
* Backup/restore cases.

---

### Extents and extent groups

DSF breaks data into smaller logical and physical units such as vBlocks, extents, and extent groups. These are used internally for distribution, replication, compression, deduplication, striping, and movement.

You do **not** need to memorize every low-level structure, but you should know that Nutanix does not treat storage as one monolithic disk. It breaks VM data into chunks and distributes them intelligently.

Interview phrase:

> At a high level, DSF breaks VM disks into smaller units, maps them through metadata, distributes them across nodes and disks, and uses that structure for replication, locality, optimization, and healing.

---

### Stargate

**Stargate** is a key Nutanix service responsible for data I/O. The Nutanix Bible describes Stargate as the Data I/O manager and the main interface from the hypervisor via NFS, iSCSI, or SMB, running on every node to serve localized I/O. 

Support relevance:

* Storage latency.
* I/O path issues.
* Local vs remote reads.
* CVM resource pressure.
* Node-level storage service health.

Interview phrase:

> If Prism shows storage latency or if VMs experience I/O delays, Stargate health and the local CVM data path become important areas for the SRE team to inspect.

---

### Cassandra / Medusa

**Cassandra** stores distributed metadata. The Nutanix Bible says Cassandra stores and manages cluster metadata in a distributed ring-like manner and is accessed through Medusa. 

Support relevance:

* Metadata health.
* Cluster consistency.
* Service instability.
* More advanced escalations.

Interview phrase:

> I would not pretend to manually troubleshoot Cassandra internals, but I understand that metadata availability and consistency are critical for locating and serving data correctly in a distributed storage system.

---

### Zookeeper / Zeus

**Zookeeper** manages cluster configuration and leadership/quorum-related functions. The Nutanix Bible describes it as storing cluster configuration such as hosts, IPs, and state, with leader election behavior. 

Support relevance:

* Cluster services not starting.
* Quorum issues.
* Leadership/election issues.
* CVM or node outages.

---

### Curator

**Curator** handles distributed background tasks such as disk balancing, proactive scrubbing, cleanup, and other cluster maintenance tasks. Nutanix Bible describes Curator as responsible for managing and distributing tasks across the cluster, including disk balancing and proactive scrubbing. 

Support relevance:

* Rebalancing after node addition.
* Cleanup after snapshot deletion.
* Capacity reclamation.
* Background job impact.
* Post-failure healing behavior.

---

### Replication Factor: RF2 / RF3

**RF2** means two copies.
**RF3** means three copies.

RF protects data across failures. Nutanix’s public FAQ describes RF2 as keeping every piece of data on at least two different nodes. ([Nutanix][1])

Support relevance:

* Node failure.
* Disk failure.
* Degraded state.
* Rebuild time.
* Risk communication.

Interview phrase:

> RF is fundamental for explaining customer risk. A cluster may remain online after a failure, but the question is whether it is fully protected, degraded, or exposed to additional failure risk while healing.

---

### Data locality

Data locality means storing VM data close to the compute running that VM. Nutanix documentation says data locality ensures read I/O does not have to go through the network, optimizing performance and reducing network congestion. ([Nutanix][2])

Support relevance:

* VM migration.
* HA events.
* Performance after vMotion.
* Network reads.
* Hot data movement.

---

### Intelligent tiering

DSF moves hot data to faster storage and cold data to slower/lower-cost tiers. Nutanix describes DSF as automatically monitoring access patterns and moving data between SSD and HDD tiers. ([Nutanix][2])

Support relevance:

* Performance variability.
* Mixed workloads.
* Hybrid SSD/HDD clusters.
* Capacity/performance trade-offs.

---

### Compression, deduplication, erasure coding

These are storage efficiency mechanisms.

* **Compression** reduces data size.
* **Deduplication** removes duplicate data.
* **Erasure Coding** uses parity-like mechanisms to reduce capacity overhead compared with full replicas.

Nutanix’s AOS documentation lists deduplication, compression, and erasure coding as DSF capacity optimization features. ([Nutanix][2])

Support relevance:

* Capacity savings.
* Performance overhead.
* Workload suitability.
* Misconfigured policies.
* Unexpected capacity growth.

---

## 5. How it appears in a real escalation

### Scenario 1 — Disk failure with customer anxiety

**Customer symptom**

> “Prism shows a disk failure. Are we at risk? Are VMs safe? Why is performance degraded?”

**What is happening**

A disk failed. DSF should continue serving data from replicas. The cluster may enter a degraded state and start re-replicating data to restore the configured replication factor.

**Manager-level response**

You need to establish:

* Current cluster health.
* Which node/disk failed.
* RF level.
* Whether re-replication is active.
* Whether any VMs are impacted.
* Whether there are additional hardware alerts.
* Capacity headroom for healing.
* Whether replacement hardware is needed.
* Whether the customer is inside SLA or business-critical window.

Good escalation language:

> The platform is designed to tolerate disk and node failures through distributed replication. Our immediate priority is to confirm the current protection state, verify that re-replication is progressing, check for any secondary risks, and provide the customer with a clear recovery timeline and impact assessment.

---

### Scenario 2 — VM latency after node failure

**Customer symptom**

> “After a node went down, several VMs are slower. The application team says storage latency increased.”

**Possible DSF angle**

* Data may be served remotely.
* The cluster may be rebuilding or rebalancing.
* CVM resources may be under pressure.
* Network may be contributing to remote read/write latency.
* Hot data may need to re-localize.
* Background healing may compete with production I/O.

Good escalation language:

> I would separate availability from performance. The VMs may remain available because DSF can serve replicas, but performance can still be affected during failure handling, remote access, or healing. I would ask the SREs to validate Stargate health, CVM resource usage, cluster I/O latency, re-replication progress, and network health between nodes.

---

### Scenario 3 — Capacity nearly full

**Customer symptom**

> “The cluster is at 90% capacity. Can we keep running? Why is usable capacity so low?”

**Possible DSF angle**

* Replication factor overhead.
* Snapshots.
* Garbage collection pending.
* Compression/dedup not effective for workload.
* Erasure coding not enabled or not suitable.
* Imbalance across nodes.
* Rebuild cannot complete due to insufficient free space.

Good escalation language:

> Capacity pressure in a distributed storage system is not just a storage consumption issue; it is a resilience issue. If the cluster lacks headroom, healing, rebalancing, and snapshot operations may be constrained. I would prioritize reducing risk, identifying major consumers, validating snapshot and protection domain usage, and planning expansion or cleanup.

---

### Scenario 4 — After VM migration, performance drops

**Customer symptom**

> “We moved VMs and now they are slower.”

**Possible DSF angle**

* Data locality not yet restored.
* Reads going remote.
* Network path issue.
* Hot data movement still in progress.
* Hypervisor scheduling issue.
* Not necessarily a Nutanix storage defect.

Good escalation language:

> After VM movement, I would check whether the performance change correlates with data locality, network reads, CVM load, or hypervisor-level contention. I would avoid jumping directly to a storage defect until we correlate VM movement, latency metrics, and cluster activity.

---

### Scenario 5 — Multi-vendor blame case: Nutanix vs VMware vs network

**Customer symptom**

> “VMware says storage is slow. Network says everything is fine. Application says Nutanix is the bottleneck.”

**Manager-level value**

This is where a support manager matters. You need to organize a structured bridge:

* Define symptoms and timeline.
* Get objective metrics from Prism, hypervisor, guest OS, and application.
* Separate read/write latency.
* Identify affected vs unaffected VMs.
* Compare nodes, containers, and workloads.
* Check recent changes.
* Avoid vendor-blame language.
* Assign owners for each hypothesis.

Good escalation language:

> I would drive the escalation through evidence, not assumptions. We need a shared timeline, affected workload list, latency metrics from Prism and the hypervisor, recent changes, and clear ownership for each layer: guest OS, hypervisor, CVM, network, and application.

---

## 6. Triage questions I should ask

### Business impact

1. Which applications or services are affected?
2. Is this production, pre-production, or DR?
3. Is the impact outage, degradation, intermittent latency, or alert-only?
4. How many VMs, users, sites, or customers are affected?
5. Is there a critical business window, audit, batch job, backup window, or migration in progress?

### Timeline and change correlation

6. When did the issue start?
7. What changed recently: upgrades, node additions, disk replacement, VM migrations, backups, snapshots, firmware, network changes, hypervisor changes?
8. Was there a power, network, or hardware event?
9. Did the issue start suddenly or gradually?

### Cluster health

10. What does Prism show for cluster health?
11. Are there disk, node, CVM, storage container, or capacity alerts?
12. Is the cluster degraded or fully protected?
13. Is re-replication, healing, balancing, or cleanup running?
14. Are all CVMs up and healthy?

### Performance

15. Is the issue read latency, write latency, throughput, IOPS, or application response time?
16. Is latency visible in Prism, the hypervisor, guest OS, or only the application?
17. Are all VMs affected or only specific VMs?
18. Are affected VMs concentrated on one node, one container, one host, or one workload type?
19. Is there a backup, snapshot, replication, antivirus, or batch job running?

### Capacity

20. What is current used, free, and reserved capacity?
21. Is the cluster close to a high-utilization threshold?
22. Are snapshots or protection domains consuming unexpected space?
23. Are compression, deduplication, or erasure coding enabled?
24. Is there enough free capacity for rebuild/healing?

### Network and hypervisor

25. Are there network errors, packet drops, MTU issues, or switch changes?
26. Are CVM and hypervisor management/storage networks healthy?
27. Has vMotion/live migration occurred recently?
28. Are hosts overloaded on CPU or memory?
29. Are there hypervisor datastore latency alarms?

### Customer management

30. Who is the technical owner on the customer side?
31. Who is the executive stakeholder?
32. What communication cadence does the customer expect?
33. What is the next decision point: restore service, reduce risk, replace hardware, tune performance, or explain RCA?

---

## 7. Likely interview questions

1. What is the Nutanix Distributed Storage Fabric?
2. How is Nutanix storage different from traditional SAN/NAS?
3. What is the role of the CVM in Nutanix storage?
4. What is data locality and why does it matter?
5. What happens when a disk fails in a Nutanix cluster?
6. What happens when a node fails?
7. What is replication factor?
8. What is the difference between RF2 and RF3?
9. What is a storage pool?
10. What is a storage container?
11. Where would compression, deduplication, and erasure coding be configured conceptually?
12. What is Stargate?
13. What is Curator?
14. What is Cassandra used for in Nutanix?
15. How would you handle a customer escalation reporting storage latency?
16. How would you manage a Sev1 where the customer blames Nutanix storage?
17. How would you communicate risk during re-replication?
18. How do you distinguish storage latency from application latency?
19. How would you lead a multi-vendor escalation involving Nutanix, VMware, and network teams?
20. As a manager, how deep do you need to go technically?

---

## 8. Model answers in English

### Q1. What is the Distributed Storage Fabric?

**Model answer**

> Distributed Storage Fabric, or DSF, is the core storage layer of Nutanix AOS. It pools the local disks from all nodes in the cluster and presents them to the hypervisor as shared storage. Internally, however, the data path is distributed: each node runs a Controller VM, data is replicated across nodes, reads are served locally whenever possible, and the platform can self-heal after disk or node failures.
>
> From a support perspective, DSF is critical because many customer escalations around latency, capacity, data protection, and availability are ultimately related to how data is distributed, replicated, rebuilt, or accessed across the cluster.

---

### Q2. How is Nutanix storage different from traditional SAN storage?

**Model answer**

> In a traditional architecture, compute hosts depend on an external storage array over a storage network. That introduces separate storage controllers, LUNs, zoning, multipathing, and a different operational model.
>
> Nutanix uses a hyperconverged model. Storage is local to each node, but AOS and DSF aggregate it into a distributed storage fabric. The hypervisor still sees shared storage, but the storage intelligence is software-defined and distributed across the CVMs.
>
> The operational benefit is simpler scaling and built-in resilience. The support challenge is that troubleshooting requires looking across compute, CVM, storage services, network, and workload behavior, not only at a storage array.

---

### Q3. What is data locality?

**Model answer**

> Data locality means Nutanix tries to keep a VM’s data on the same node where the VM is running. This is important because local reads avoid unnecessary network hops, reduce latency, and reduce storage network congestion.
>
> In an escalation, data locality may become relevant after VM migrations, HA events, or workload movement. If a VM moves to another node, the system may initially serve some data remotely and then re-localize hot data over time. As a support manager, I would not overstate this as the only cause of latency, but I would make sure the SRE team checks locality, remote reads, CVM load, and network health.

---

### Q4. What happens when a disk fails?

**Model answer**

> With replication factor enabled, Nutanix stores multiple copies of data across different nodes. If a disk fails, the system should continue serving data from the remaining replicas. Then AOS starts a self-healing process to restore the required replication factor by re-replicating data to healthy disks or nodes.
>
> In a customer escalation, I would focus on four things: whether data remains available, whether the cluster is degraded, whether healing is progressing, and whether there is enough capacity and hardware health to return to a fully protected state. I would also make sure the customer understands the difference between “service is online” and “cluster is fully protected.”

---

### Q5. What is the role of the CVM?

**Model answer**

> The Controller VM is the Nutanix software controller running on each node. It hosts the AOS services that participate in the distributed storage and management plane. Instead of having a physical external storage controller, Nutanix distributes that role across CVMs.
>
> For support, CVM health is essential. If a CVM is overloaded, unavailable, or has service issues, it can affect storage I/O, cluster health, or management operations.

---

### Q6. How would you handle a storage latency escalation?

**Model answer**

> I would start by defining the impact and scope: which workloads are affected, when it started, whether it is read latency, write latency, throughput, or application response time, and whether the issue is visible in Prism, the hypervisor, the guest OS, or only the application.
>
> Then I would structure the investigation across layers: cluster health, CVM health, Stargate/storage service health, capacity, active background tasks such as healing or balancing, recent changes, hypervisor metrics, and network health.
>
> As a manager, my role is to keep the escalation disciplined: establish a timeline, assign hypotheses to owners, keep the customer updated, and ensure we distinguish evidence from assumptions. I would also escalate internally if we see signs of data unavailability, cluster degradation, or risk of additional failures.

---

### Q7. What is Stargate?

**Model answer**

> Stargate is one of the core Nutanix services responsible for data I/O. It runs on every node and handles storage operations between the hypervisor and the distributed storage fabric.
>
> In practical terms, if a customer reports VM storage latency, Stargate health and CVM resource usage are areas the technical team would inspect. As a manager, I do not need to debug Stargate internals myself, but I need to understand that it is central to the I/O path and therefore relevant in storage performance escalations.

---

### Q8. What is Curator?

**Model answer**

> Curator is a Nutanix background service responsible for distributed maintenance tasks such as disk balancing, proactive scrubbing, and cleanup.
>
> In support scenarios, Curator can be relevant after node additions, disk failures, snapshot deletion, or capacity cleanup. If a customer asks why capacity has not been reclaimed immediately or why balancing is still running, Curator-related background activity may be part of the explanation.

---

### Q9. What would you say to an angry customer during re-replication?

**Model answer**

> I would avoid generic reassurance and give a clear operational explanation. I would say: the cluster is designed to remain available during certain hardware failures by using distributed replicas. Right now, the system is restoring the desired protection level. Our priorities are to confirm workload availability, monitor the re-replication progress, check capacity headroom, watch for any secondary hardware risk, and provide regular updates until the cluster returns to a fully protected state.
>
> I would also be transparent that performance may be affected depending on workload, cluster load, and the amount of data being healed.

---

### Q10. How deep should a support manager go technically?

**Model answer**

> A support manager should be technically fluent enough to lead the escalation, ask precise questions, understand the risk, challenge weak hypotheses, and communicate credibly with both SREs and customers.
>
> I would not position myself as the engineer who manually analyzes every metadata structure. I would position myself as a technical escalation manager: someone who understands DSF architecture, knows which components matter in each scenario, and can coordinate engineering, SREs, field teams, and the customer toward resolution.

---

## 9. Connection with my experience

Your background maps well to this topic if you frame it correctly.

You already know:

* 24/7 enterprise support.
* Incident management.
* SLA and MTTR ownership.
* Escalation coordination.
* Cloud operations.
* Monitoring and observability.
* Customer-impact communication.
* Jira / Confluence / Salesforce workflows.
* Cross-functional coordination.

Translate that into Nutanix language:

| Your experience       | Nutanix DSF relevance                                                                                                 |
| --------------------- | --------------------------------------------------------------------------------------------------------------------- |
| Incident management   | Leading Sev1/Sev2 cases involving storage degradation, failed disks, failed nodes, or performance impact.             |
| SLA / MTTR            | Driving fast triage, ownership, escalation, and restoration while tracking customer-impact timelines.                 |
| Monitoring            | Using Prism, alerts, latency metrics, capacity metrics, health checks, and logs as evidence.                          |
| Cloud operations      | Understanding distributed systems, redundancy, self-healing, noisy neighbors, capacity planning, and failure domains. |
| Enterprise support    | Communicating technical risk to customers without creating panic or false certainty.                                  |
| Team leadership       | Coaching engineers to diagnose methodically instead of jumping to assumptions.                                        |
| Escalation management | Coordinating SRE, engineering, account teams, TAMs, hardware logistics, and customer infra teams.                     |

A strong positioning statement:

> My value is not that I have spent years as a Nutanix storage engineer. My value is that I understand distributed infrastructure operations, I know how to manage high-severity enterprise escalations, and I can quickly build fluency in Nutanix-specific architecture such as AOS and DSF. I can lead the process, communicate risk clearly, and ensure the right technical depth is brought in at the right time.

---

## 10. Minimum I need to memorize

Memorize these points cold:

1. **AOS** is the Nutanix core software layer running in the CVM.
2. **DSF** is the distributed storage layer of AOS.
3. **CVM** runs on each node and participates in the storage/control plane.
4. DSF pools local disks across nodes and presents shared storage to the hypervisor.
5. The hypervisor may be **AHV**, **VMware ESXi**, or other supported hypervisors.
6. DSF uses **replication factor**, not traditional array RAID as the main mental model.
7. **RF2** means two copies; **RF3** means three copies.
8. After disk/node failure, DSF can continue serving data from replicas and self-heal.
9. **Data locality** means VM data is kept close to where the VM runs to reduce read latency.
10. **Stargate** is the main data I/O service.
11. **Curator** handles background tasks like balancing, scrubbing, and cleanup.
12. **Cassandra/Medusa** are related to distributed metadata.
13. **Zookeeper/Zeus** are related to cluster configuration and coordination.
14. **Storage Pool** aggregates physical disks.
15. **Container** is where storage policies are conceptually applied.
16. Compression, deduplication, and erasure coding are capacity optimization features.
17. In escalations, distinguish **availability**, **performance**, **capacity**, and **protection state**.
18. Your role is to lead the escalation, not pretend to be the deepest DSF engineer.

One sentence to memorize:

> DSF is the distributed storage layer of Nutanix AOS that pools local disks across nodes, presents shared storage to the hypervisor, keeps I/O local when possible, protects data through replication, and self-heals after failures.

---

## 11. Advanced / optional level

You can leave these as advanced unless the SRE interview goes deep:

* Exact vBlock, extent, and extent group internals.
* Cassandra ring mechanics.
* Paxos details.
* Detailed Stargate log interpretation.
* Manual Curator task analysis.
* Low-level metadata repair scenarios.
* Exact RF placement algorithms.
* Detailed EC-X layout.
* Advanced performance tuning.
* NCC command-level troubleshooting.
* AOS upgrade internals.
* Deep AHV vs ESXi data path differences.
* Specific CLI commands unless explicitly asked.

However, you should recognize these words and know where they fit:

* Stargate → I/O path.
* Curator → background maintenance.
* Cassandra / Medusa → metadata.
* Zookeeper / Zeus → cluster coordination/configuration.
* Cerebro → replication / DR.
* Prism → UI/API management plane.
* NCC → health checks.

---

## 12. Final checklist

Before an interview, you should be able to answer:

* Can I explain DSF in 60 seconds?
* Can I explain why Nutanix does not need a traditional SAN?
* Can I explain what the CVM does?
* Can I explain RF2 and RF3?
* Can I explain what happens after a disk failure?
* Can I explain data locality?
* Can I explain the difference between storage pool and container?
* Can I name Stargate, Curator, Cassandra, Zookeeper, and Prism at a high level?
* Can I lead a storage latency escalation without pretending to be the storage IC?
* Can I communicate customer risk during degraded protection?
* Can I connect DSF to my background in incident management, MTTR, monitoring, and enterprise support?

Best final interview framing:

> I see DSF as the foundation of many Nutanix support scenarios. For a Worldwide Support Manager, the critical skill is not only knowing the architecture, but knowing how that architecture behaves under failure, how to triage it under pressure, and how to communicate risk and progress to enterprise customers.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                      |
| ------------------------ | -------------------------------------------------------------------------------------------- |
| AHV                      | Acropolis Hypervisor; Nutanix’s native hypervisor.                                           |
| AOS                      | Acropolis Operating System; Nutanix core software layer providing storage and data services. |
| Automatic Disk Balancing | DSF mechanism that redistributes data to keep utilization balanced.                          |
| Availability             | Whether workloads remain accessible and running.                                             |
| Capacity Optimization    | Features that reduce storage usage, such as compression, deduplication, and erasure coding.  |
| Cassandra                | Distributed metadata store used by Nutanix cluster services.                                 |
| Cerebro                  | Nutanix service associated with replication and disaster recovery.                           |
| Cluster                  | Group of Nutanix nodes operating as one system.                                              |
| Container                | Logical segment of a storage pool where storage policies are applied.                        |
| Controller VM            | Virtual machine on each node running Nutanix AOS services.                                   |
| Curator                  | Nutanix service for background tasks such as balancing, scrubbing, and cleanup.              |
| CVM                      | Controller VM; local Nutanix software controller on each node.                               |
| Data Locality            | Keeping VM data on or near the node where the VM runs.                                       |
| Deduplication            | Removing duplicate data blocks to save capacity.                                             |
| Degraded State           | Condition where the cluster is running but not fully protected or healthy.                   |
| Disk Balancing           | Redistribution of data across disks or nodes.                                                |
| DSF                      | Distributed Storage Fabric; Nutanix distributed storage layer.                               |
| EC-X                     | Nutanix erasure coding mechanism for capacity-efficient protection.                          |
| Erasure Coding           | Parity-based capacity optimization method reducing replica overhead.                         |
| ESXi                     | VMware hypervisor commonly used with Nutanix.                                                |
| Extent                   | Internal DSF data unit used for mapping and distribution.                                    |
| Extent Group             | Physical storage unit containing extents on storage devices.                                 |
| Fault Domain             | Grouping used to reason about failure impact.                                                |
| Genesis                  | Nutanix service manager for starting and managing cluster components.                        |
| HA                       | High Availability; ability to keep services running during failures.                         |
| HCI                      | Hyperconverged Infrastructure; combines compute, storage, and virtualization.                |
| Hot Data                 | Frequently accessed data, typically placed on faster storage.                                |
| I/O                      | Input/Output; read and write operations.                                                     |
| IOPS                     | Input/output operations per second.                                                          |
| Intelligent Tiering      | Automatic movement of hot/cold data across storage tiers.                                    |
| Latency                  | Time taken to complete an I/O or application operation.                                      |
| Medusa                   | Interface used to access Cassandra metadata.                                                 |
| MTTR                     | Mean Time To Restore/Repair; key support metric.                                             |
| NCC                      | Nutanix Cluster Check; health-check tooling.                                                 |
| NFS                      | Network File System; storage protocol used in virtualization environments.                   |
| Node                     | Physical server participating in a Nutanix cluster.                                          |
| NVMe                     | High-performance flash storage interface/protocol.                                           |
| Prism                    | Nutanix management UI and API layer.                                                         |
| Protection State         | Whether data has the expected number of healthy replicas.                                    |
| Re-replication           | Rebuilding missing data copies after a failure.                                              |
| RF                       | Replication Factor; number of data copies maintained.                                        |
| RF2                      | Replication Factor 2; two copies of data.                                                    |
| RF3                      | Replication Factor 3; three copies of data.                                                  |
| RCA                      | Root Cause Analysis.                                                                         |
| Rebuild                  | Process of restoring data protection after failure.                                          |
| Remote Read              | Read served from another node instead of the local node.                                     |
| Resilience               | Ability to withstand failures while preserving service/data.                                 |
| SAN                      | Storage Area Network; traditional external block storage architecture.                       |
| SCSI                     | Storage command protocol used by disks and hypervisors.                                      |
| Self-Healing             | Automatic restoration of data protection after failures.                                     |
| SLA                      | Service Level Agreement.                                                                     |
| SMB                      | Server Message Block; file-sharing/storage protocol.                                         |
| Snapshot                 | Point-in-time data copy used for protection or recovery.                                     |
| SSD                      | Solid-State Drive.                                                                           |
| Stargate                 | Nutanix data I/O manager service.                                                            |
| Storage Pool             | Logical aggregation of physical storage devices across nodes.                                |
| Throughput               | Amount of data transferred per second.                                                       |
| vBlock                   | Logical chunk of virtual address space within a vDisk.                                       |
| vDisk                    | Virtual disk or VM disk stored on DSF.                                                       |
| VM                       | Virtual Machine.                                                                             |
| vMotion                  | VMware live migration of a VM between hosts.                                                 |
| Zookeeper                | Cluster configuration and coordination service.                                              |
| Zeus                     | Interface used to access Zookeeper.                                                          |

[1]: https://www.nutanix.com/products/nutanix-cloud-infrastructure/distributed-storage "A Modern Storage Fabric | Nutanix"
[2]: https://www.nutanix.com/content/dam/nutanix/documents/certifications/advanced-admin-aos.pdf "Acropolis Advanced Administration Guide"
