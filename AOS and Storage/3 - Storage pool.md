# AOS / Storage — Storage Pool

## 1. Short definition

A **Storage Pool** in Nutanix AOS is the aggregated pool of physical disks in a Nutanix cluster. It is the physical capacity layer from which **Storage Containers** are created, and those containers then hold VM virtual disks, volume groups, files, or other storage-consuming entities.

In simple interview language:

> “A Nutanix storage pool is the physical disk aggregation layer of the cluster. AOS groups SSDs and HDDs/NVMe devices across nodes into a distributed pool, and then logical containers are created on top of that pool to present storage to workloads.”

Nutanix documentation describes the storage pool as a “pool of physical disks,” and Prism documentation explains that Nutanix storage is organized hierarchically into storage pool, storage container, volume group, and virtual disk components. ([portal.nutanix.com][1])

---

## 2. Clear explanation

In a traditional architecture, you often have external shared storage: SAN, NAS, storage controllers, RAID groups, LUNs, aggregates, volumes, and datastores. Nutanix changes the model. Each node contributes local disks, and AOS aggregates those disks into a distributed storage layer.

At a high level:

```text
Physical disks across nodes
        ↓
Storage Pool
        ↓
Storage Containers
        ↓
vDisks / VMs / Volume Groups / Files
        ↓
Applications
```

The **Storage Pool** is not normally the object an application team interacts with. It is more of an infrastructure-level resource. The operationally visible objects are usually **Storage Containers**, datastores, volume groups, VMs, snapshots, and capacity metrics.

A **Storage Container** is a logical subset of the Storage Pool. Nutanix documentation states that containers are created within a storage pool to hold vDisks used by VMs, and also notes that, unless there is a specific reason to have multiple containers, Nutanix recommends a single storage pool with a single storage container. ([portal.nutanix.com][2])

This distinction matters:

| Layer             | What it means                                | Interview framing                                            |
| ----------------- | -------------------------------------------- | ------------------------------------------------------------ |
| Storage Pool      | Physical disk aggregation across the cluster | “Where the cluster-wide physical capacity lives.”            |
| Storage Container | Logical storage boundary inside the pool     | “Where policies and workload placement are usually managed.” |
| vDisk             | VM disk object                               | “The actual storage consumed by a VM.”                       |
| Volume Group      | Block storage object, often iSCSI use cases  | “Used when workloads need direct block devices.”             |

AOS also provides distributed storage behavior. Nutanix describes AOS storage as giving VMs low-latency access by serving storage resources locally where possible, and contrasts that with traditional external storage controllers across the network. ([portal.nutanix.com][3])

So, verbally:

> “The storage pool is the physical capacity foundation. AOS aggregates disks across nodes, then exposes logical containers on top. Workloads don’t usually consume the pool directly; they consume containers, vDisks, volume groups, or file/object services built on top of the distributed storage fabric.”

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, Storage Pool knowledge matters because many serious enterprise escalations are actually storage-capacity, resiliency, or data-placement issues.

You do not need to be the deepest storage engineer in the room, but you must be able to:

1. Understand the difference between **logical capacity** and **physical capacity**.
2. Ask whether the issue is at the **pool**, **container**, **disk**, **node**, or **workload** level.
3. Recognize whether the customer is at risk of **resiliency degradation**, **capacity exhaustion**, or **performance degradation**.
4. Drive the escalation with the right urgency and stakeholders.
5. Communicate clearly to enterprise customers without oversimplifying the risk.

Nutanix’s storage model is central to the platform. Prism documentation explicitly frames storage pool, storage container, volume group, and virtual disk as the hierarchy used to organize capacity and performance. ([portal.nutanix.com][4])

For a support manager, this topic matters because a storage pool problem can become:

* A **P1 outage** if workloads cannot write.
* A **data resiliency risk** if the cluster cannot rebuild after a disk or node failure.
* A **performance incident** if storage imbalance, disk pressure, or rebuild activity increases latency.
* A **customer-confidence issue** if the customer believes “Nutanix storage is full” but cannot understand why usable capacity differs from raw capacity.

A strong manager answer would be:

> “I would treat storage pool issues as a combination of capacity, resiliency, and business-risk management. My role is not only to interpret the metric but to coordinate SREs, support engineers, account teams, and the customer around impact, safe remediation, and communication.”

---

## 4. Key concepts

### 4.1 Storage Pool vs Storage Container

The **Storage Pool** is the physical capacity aggregation.

The **Storage Container** is a logical segmentation inside the pool. Nutanix documentation describes a storage container as a subset of available storage within a storage pool, used to hold VM vDisks. ([portal.nutanix.com][2])

A useful analogy:

```text
Storage Pool = the whole warehouse
Storage Container = a managed section of the warehouse
vDisk = an item stored inside that section
VM = the application consuming the item
```

For interviews, avoid saying that the Storage Pool itself is the datastore. More precise:

> “The storage pool is the physical aggregation layer. Containers are the logical layer that can be presented to the hypervisor or consumed by workloads.”

---

### 4.2 Distributed Storage Fabric

Nutanix AOS does not depend on a single external storage controller. Each node contributes compute and storage resources. The Controller VMs, or CVMs, cooperate to provide distributed storage services.

Nutanix documentation says all CVMs work together to aggregate storage resources into a single global pool that guest VMs can consume. It also states that AOS manages storage resources to preserve data and system integrity during node, disk, application, or hypervisor failures. ([portal.nutanix.com][5])

For interview purposes:

> “The pool is cluster-wide, but the data path is distributed. That is why troubleshooting needs to consider node health, CVM health, disk health, network health, and workload behavior.”

---

### 4.3 Capacity: raw, usable, logical, and resilient

One of the biggest escalation traps is capacity confusion.

Customers may say:

> “We bought 100 TB. Why can we only use much less?”

You need to explain that usable capacity accounts for:

* Metadata overhead.
* Resiliency overhead.
* Replication factor.
* Reserved rebuild capacity.
* Compression/deduplication/erasure coding effects.
* Snapshots/clones.
* Oplog and operational overhead.

Nutanix documentation defines usable storage container capacity as the actual amount available after accounting for overheads such as system metadata, resiliency copies, and spare capacity for rebuilding during failures or maintenance. ([portal.nutanix.com][6])

A support-manager answer:

> “I would first align everyone on which capacity metric we are discussing: raw, physical used, logical used, container used, resilient capacity, or rebuild-reserved capacity. Many escalations are prolonged because stakeholders are looking at different capacity numbers.”

---

### 4.4 Replication Factor: RF2 and RF3

Replication Factor defines how many copies of data are stored for resiliency.

* **RF2** generally means two copies.
* **RF3** generally means three copies and higher failure tolerance.
* RF3 requires sufficient cluster size and failure-domain layout.

Nutanix documentation states that RF3 requires at least five nodes, blocks, or racks, and that guest VMs need data stored on RF3 containers to tolerate simultaneous failure of two nodes or drives in different blocks. ([portal.nutanix.com][7])

Interview framing:

> “RF is not just a capacity setting. It defines failure tolerance and has direct implications for usable capacity, rebuild behavior, and customer risk during hardware failures.”

---

### 4.5 Rebuild capacity reservation

When a disk or node fails, the cluster may need free capacity to rebuild data and restore resiliency. Nutanix has a **Rebuild Capacity Reservation** feature to reserve storage for self-healing. Nutanix documentation says rebuild capacity is calculated based on fault tolerance, failure domain, and total cluster capacity. ([portal.nutanix.com][8])

This is very relevant to escalation management.

A cluster can appear “not completely full,” but still be close to or beyond safe resilient capacity. The support manager must know that the operational question is not only:

> “Is there free space?”

but also:

> “Is there enough free space to survive and recover from a failure?”

Nutanix documentation also warns that enabling rebuild capacity reservation when current usage is close to the resilient capacity threshold may result in VM outage, and recommends ensuring usage is not more than 90% of the calculated resilient capacity threshold before enabling reservation. ([portal.nutanix.com][8])

---

### 4.6 Data efficiency

Storage pool capacity is affected by data efficiency features such as:

* Thin provisioning.
* Compression.
* Deduplication.
* Erasure Coding, often referred to as EC-X.
* Intelligent cloning.

Nutanix documentation states that AOS Storage provides data avoidance and efficiency through thin provisioning, intelligent cloning, compression, deduplication, and erasure coding. ([portal.nutanix.com][9])

For interviews, the key is not to claim that data efficiency is always positive with no trade-off. Say:

> “Data efficiency can improve usable capacity, but during escalations I would verify whether the feature is enabled, appropriate for the workload, and relevant to the symptom. I would not assume compression or deduplication will solve an urgent capacity issue without validating workload behavior and Nutanix guidance.”

---

### 4.7 Prism and operational visibility

In real support, most customers and support teams will start with **Prism Element** or **Prism Central**. Prism documentation says the Storage Details page displays resiliency status and physical storage information for individual nodes. ([portal.nutanix.com][10])

The support manager should expect the team to check:

* Storage dashboard.
* Cluster capacity.
* Container capacity.
* Node-level capacity.
* Disk-level alerts.
* Rebuild status.
* Resiliency status.
* NCC health checks.
* Recent changes: node expansion, disk replacement, workload growth, snapshots, failed disks.

---

## 5. How it appears in a real escalation

### Scenario A — Customer says: “The Nutanix cluster is running out of storage”

Possible reality:

* Storage Pool is close to capacity.
* One or more containers are consuming unexpectedly high space.
* Snapshots or clones are growing.
* Data reduction expectations were wrong.
* A backup job or migration created a sudden write spike.
* Rebuild capacity is insufficient.
* One node or disk group is more heavily used than others.
* A failed disk/node triggered rebuild activity and increased usage.

Support-manager response:

> “I would immediately separate impact from diagnosis. First, I would confirm whether VMs are affected, whether writes are failing, whether latency has increased, and whether resiliency is degraded. Then I would have the technical team validate storage pool usage, container usage, snapshot growth, rebuild status, and recent workload changes. In parallel, I would set customer communication cadence and define containment options, such as deleting safe snapshots, expanding capacity, pausing non-critical workloads, or engaging engineering if the behavior looks abnormal.”

---

### Scenario B — Disk failure followed by resiliency warning

Possible reality:

* A disk failed.
* AOS is rebuilding data.
* The Storage Pool needs enough free capacity to restore redundancy.
* Performance may temporarily degrade during rebuild.
* If capacity is too high, the cluster may not be able to regain resiliency safely.

Escalation risk:

* Customer may ask: “Are we at risk of data loss?”
* Support must avoid panic but communicate risk accurately.

Model response:

> “The key is to confirm whether the cluster remains within its configured failure tolerance. I would ask the SREs to check resiliency status, rebuild progress, failed disk details, available capacity, and whether there are additional hardware alerts. Customer communication should explain whether the system is protected, degraded, or exposed to additional failure risk.”

---

### Scenario C — VM latency after cluster expansion or heavy workload

Possible reality:

* Workload is write-intensive.
* Rebuild or rebalancing is running.
* Hot data locality may be affected.
* Storage pool capacity is fine, but performance is constrained by disk, CVM, network, or workload pattern.
* The issue is not “space” but I/O path, latency, or resource contention.

Model response:

> “I would avoid reducing the issue to storage capacity only. I would look at latency, IOPS, throughput, CVM health, disk health, network, hypervisor metrics, and workload changes. In Nutanix, storage is distributed, so performance troubleshooting requires correlation across compute, storage, and network layers.”

---

### Scenario D — Customer created multiple storage containers and sees unexpected behavior

Possible reality:

* Customer misunderstood pool vs container.
* Different containers have different policies.
* Capacity reservation or RF settings differ.
* Workloads are distributed across containers inconsistently.
* Container-level settings influence workload behavior.

Nutanix documentation notes that Nutanix recommends a single storage pool with a single storage container unless there is a specific reason to have multiple storage containers. ([portal.nutanix.com][2])

Manager framing:

> “I would ask why multiple containers were created: isolation, policy separation, migration, hypervisor requirements, or historical reasons. Then I would validate whether the current design is intentional or adding operational complexity.”

---

## 6. Triage questions I should ask

### Impact and urgency

1. Are any VMs down, paused, read-only, or unable to write?
2. Is this a production cluster?
3. Is the customer experiencing application outage, degraded performance, or only an alert?
4. Is the issue affecting one VM, one container, one node, or the whole cluster?
5. What is the business impact and SLA exposure?

### Capacity and storage pool

6. What is the current Storage Pool usage?
7. What is the usable capacity versus raw capacity?
8. Is the cluster close to resilient capacity threshold?
9. Is rebuild capacity reservation enabled?
10. Is capacity consumption increasing gradually or suddenly?

### Containers and workloads

11. Which storage containers are consuming the most capacity?
12. Are there recent VM deployments, migrations, backups, restores, or snapshots?
13. Are there large snapshots, clones, or orphaned objects?
14. Are workloads thin-provisioned or thick-provisioned?
15. Are there high-churn workloads such as databases, logging, analytics, or backup repositories?

### Resiliency

16. What RF is configured: RF2 or RF3?
17. Is resiliency currently OK, degraded, or rebuilding?
18. Are there failed disks, failed nodes, or CVM alerts?
19. Is the failure domain node, block, or rack?
20. Is the cluster able to tolerate another failure?

### Performance

21. Is the symptom capacity, latency, IOPS, throughput, or availability?
22. Are there CVM CPU/memory/network issues?
23. Is there rebuild, rebalancing, or curator activity?
24. Are hot spots visible at disk, node, or VM level?
25. Did the problem start after a change?

### Process and communication

26. Who is the customer technical owner?
27. Is there an open P1/P2 escalation?
28. Has Nutanix engineering been engaged?
29. What workaround is acceptable to the customer?
30. What communication cadence has been agreed?

---

## 7. Likely interview questions

### Conceptual questions

1. What is a Nutanix Storage Pool?
2. What is the difference between a Storage Pool and a Storage Container?
3. How does Nutanix storage differ from traditional SAN/NAS?
4. What does AOS do in the storage layer?
5. How does replication factor affect usable capacity?
6. Why is raw capacity different from usable capacity?
7. What is rebuild capacity reservation?
8. What happens when a disk or node fails?
9. Why might Nutanix recommend a single storage pool and single storage container?
10. How would you explain Storage Pool to a non-storage executive?

### Troubleshooting questions

11. A customer says their cluster is full. What do you check first?
12. A customer has a disk failure and a resiliency alert. How do you manage the escalation?
13. A customer sees high latency after a storage alert. What questions do you ask?
14. How would you distinguish a capacity issue from a performance issue?
15. How would you coordinate support, SRE, engineering, and account teams during a storage escalation?
16. What customer-facing message would you give if resiliency is degraded?
17. How do you prevent a storage incident from becoming a recurring escalation?
18. What KPIs would you track for storage-related support cases?

### Managerial questions

19. How technical should a support manager be in a storage escalation?
20. How would you coach a support engineer handling a high-pressure storage incident?
21. How would you communicate uncertainty to an enterprise customer?
22. How would you decide when to escalate to engineering?
23. How would you run a post-incident review for a Storage Pool capacity incident?

---

## 8. Model answers in English

### Q1. What is a Storage Pool in Nutanix?

> “A Storage Pool is the physical storage aggregation layer in a Nutanix cluster. AOS groups the local disks from the cluster nodes into a distributed pool, and then storage containers are created on top of that pool. The workloads usually consume storage through containers, vDisks, volume groups, or file services, not directly from the pool. For support purposes, I would look at the storage pool when investigating cluster-wide capacity, resiliency, or disk-related issues.”

---

### Q2. What is the difference between a Storage Pool and a Storage Container?

> “The Storage Pool is the underlying physical capacity layer. The Storage Container is a logical segmentation of that pool. Containers are where VM disks are stored and where policies such as replication factor or data efficiency may be relevant. In an escalation, I would check both: the pool tells me about physical capacity and resiliency, while the container helps identify which workloads or policies are driving consumption.”

---

### Q3. Why is usable capacity lower than raw capacity?

> “Usable capacity is lower because the platform needs space for resiliency, metadata, rebuild operations, and sometimes multiple data copies depending on the replication factor. Additional factors such as snapshots, clones, compression, deduplication, and erasure coding can also affect the reported numbers. In a customer conversation, I would avoid discussing only raw TB and instead align everyone on physical used, logical used, usable capacity, and resilient capacity.”

---

### Q4. A customer says the cluster is almost full. What do you do?

> “First I would assess impact: are VMs affected, are writes failing, is latency increasing, and is this a production outage or a warning? Then I would ask the technical team to check storage pool usage, container usage, snapshot growth, recent workload changes, rebuild status, and resiliency state. As a manager, I would establish customer communication cadence, assign technical ownership, identify safe containment actions, and escalate to engineering if the behavior is abnormal or risk is increasing.”

---

### Q5. What is rebuild capacity reservation?

> “Rebuild capacity reservation is reserved space that allows the cluster to self-heal after failures such as disk, node, block, or rack failure, depending on the configured failure domain and fault tolerance. It matters because a cluster can have some free space but still not have enough safe capacity to rebuild after a failure. In an escalation, I would ask whether rebuild reservation is enabled, whether usage is close to resilient capacity, and whether the cluster can tolerate another failure.”

---

### Q6. How would you handle a disk failure with resiliency degraded?

> “I would first determine whether the customer is experiencing impact or only a resiliency alert. Then I would confirm the failed component, current resiliency state, rebuild progress, available capacity, and whether there are additional hardware or CVM alerts. From a customer-management perspective, I would communicate the current protection level, the risk of additional failures, the remediation path, and the next update time. Internally, I would ensure the right storage/SRE expertise is engaged and that hardware replacement or engineering escalation is moving.”

---

### Q7. How does this relate to SRE thinking?

> “From an SRE perspective, Storage Pool issues are reliability issues. They affect error budgets, availability, MTTR, capacity planning, and failure recovery. I would look for leading indicators such as capacity growth rate, resiliency warnings, rebuild duration, recurring disk alerts, and high-latency patterns. The goal is not only to resolve the case but to reduce recurrence through better monitoring, documentation, automation, and customer education.”

---

### Q8. How would you explain this to an enterprise customer executive?

> “I would say: the cluster combines disks from multiple nodes into a distributed storage pool. On top of that, Nutanix creates logical storage areas for workloads. The alert we are investigating relates to how much safe capacity remains and whether the system has enough room to maintain or restore redundancy. Our immediate focus is to protect service availability, confirm data resiliency, and define the safest remediation path.”

---

### Q9. What would you avoid saying?

Avoid:

> “The Storage Pool is just a datastore.”

Better:

> “The Storage Pool is the physical distributed capacity layer. The datastore or storage container is the logical presentation layer.”

Avoid:

> “If there is free space, we are safe.”

Better:

> “We need to verify usable capacity, resilient capacity, failure tolerance, and rebuild capability.”

Avoid:

> “Compression or deduplication will fix the capacity problem.”

Better:

> “Data efficiency may help, but we need to validate workload suitability, current configuration, and operational risk before recommending changes.”

---

## 9. Connection with my experience

Your background maps well to this topic because you already understand the operational side:

| Your experience             | Nutanix Storage Pool equivalent                                           |
| --------------------------- | ------------------------------------------------------------------------- |
| 24/7 enterprise support     | Handling storage alerts, P1/P2 escalations, and customer communications   |
| Incident management         | Coordinating triage during capacity, resiliency, or performance incidents |
| SLA / MTTR ownership        | Restoring service and reducing time to containment                        |
| Monitoring: Grafana, Kibana | Reading trend data, usage growth, latency patterns, alert correlation     |
| Cloud operations            | Capacity planning, resilience, redundancy, failure domains                |
| Jira / Confluence           | Case tracking, escalation notes, postmortems, knowledge articles          |
| Salesforce                  | Customer case management and executive visibility                         |
| Coaching teams              | Helping engineers ask the right triage questions under pressure           |

Position yourself like this:

> “Although I am not positioning myself as a Senior SRE individual contributor, I understand the architecture well enough to drive the escalation. My value is connecting technical facts, customer impact, team coordination, and clear communication.”

That is exactly the posture for a **technical escalation manager**.

---

## 10. Minimum I need to memorize

Memorize these points:

1. **Storage Pool = physical disk aggregation layer.**
2. **Storage Container = logical segmentation inside the pool.**
3. **VMs consume vDisks stored in containers, not the pool directly.**
4. **AOS aggregates local storage across nodes into distributed storage.**
5. **Capacity must be understood as raw, usable, physical used, logical used, and resilient capacity.**
6. **RF2/RF3 affect resiliency and usable capacity.**
7. **Rebuild capacity matters after disk/node failures.**
8. **A storage alert can be a capacity issue, resiliency issue, performance issue, or customer-risk issue.**
9. **Prism is the usual operational starting point.**
10. **As a manager, your role is impact assessment, escalation ownership, customer communication, and coordination with technical experts.**

One concise verbal explanation:

> “In Nutanix AOS, the Storage Pool is the distributed physical capacity layer built from disks across the cluster. Storage Containers sit on top of that pool and hold VM disks or other storage objects. In support, Storage Pool issues matter because they affect capacity, resiliency, rebuild ability, and performance. During an escalation, I would validate impact, pool and container usage, resiliency state, failed components, recent changes, and safe remediation options while keeping the customer informed.”

---

## 11. Advanced / optional level

You can leave these as advanced for now:

* Deep internal AOS metadata architecture.
* Curator internals.
* Cassandra/Ring internals.
* Extent store and extent group mechanics.
* Detailed Stargate data path.
* Advanced EC-X behavior.
* Specific CLI/API command syntax beyond basic awareness.
* Low-level NCC plugin interpretation.
* Advanced block/rack-aware failure-domain design.
* Performance tuning at disk queue, CVM, and vDisk level.

But you should recognize these terms if SREs mention them.

For example:

> “I’m familiar with the operational meaning of those components, but for low-level internals I would rely on the SRE or engineering specialist while ensuring the customer impact and escalation path are managed properly.”

That is a strong manager answer.

---

## 12. Final checklist

Before an interview, you should be able to explain:

* [ ] What a Storage Pool is.
* [ ] How it differs from a Storage Container.
* [ ] Why Nutanix storage is distributed.
* [ ] Why usable capacity differs from raw capacity.
* [ ] How RF2/RF3 influence resiliency and capacity.
* [ ] Why rebuild capacity matters.
* [ ] What to check when a customer says “storage is full.”
* [ ] What to check when a disk/node fails.
* [ ] How to communicate storage risk to a customer.
* [ ] How to position yourself as a technical escalation manager, not a Senior SRE IC.

A final polished interview answer:

> “For me, a Storage Pool is not just a storage object; it is an operational risk boundary. If the pool has capacity pressure, failed disks, insufficient rebuild capacity, or degraded resiliency, the impact can quickly move from a technical alert to a customer-facing incident. My role as a support manager would be to make sure the team validates the right metrics, understands the failure domain, identifies safe remediation, escalates appropriately, and communicates clearly with the customer.”

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword         | Meaning                                                                                                       |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| AOS                              | Acropolis Operating System; Nutanix software layer providing distributed storage and virtualization services. |
| AHV                              | Acropolis Hypervisor; Nutanix native hypervisor.                                                              |
| Alert                            | Prism or system warning indicating health, capacity, resiliency, or performance issue.                        |
| Block                            | Physical Nutanix chassis or failure-domain unit depending on platform/design.                                 |
| Capacity Reservation             | Space reserved for specific operational or resiliency purposes.                                               |
| Cluster                          | Group of Nutanix nodes operating as one distributed system.                                                   |
| Compression                      | Data efficiency technique that reduces stored data size.                                                      |
| Container                        | Logical storage segment inside a Storage Pool.                                                                |
| Controller VM / CVM              | Nutanix VM on each node that provides storage and cluster services.                                           |
| Curator                          | Nutanix background service associated with data management and maintenance tasks.                             |
| Data Efficiency                  | Techniques such as compression, deduplication, cloning, thin provisioning, and erasure coding.                |
| Data Locality                    | Nutanix concept where data is served locally when possible to reduce latency.                                 |
| Deduplication                    | Technique that avoids storing duplicate data blocks.                                                          |
| Disk Failure                     | Physical drive failure requiring rebuild or replacement.                                                      |
| Distributed Storage Fabric / DSF | Nutanix distributed storage architecture across nodes.                                                        |
| EC-X                             | Nutanix Erasure Coding technology for capacity efficiency.                                                    |
| Erasure Coding                   | Capacity-efficient resiliency method using parity rather than full extra copies.                              |
| Escalation                       | Formal process to involve higher-level support, SRE, engineering, or management.                              |
| Failure Domain                   | Scope of failure tolerance, such as disk, node, block, or rack.                                               |
| Grafana                          | Monitoring/visualization platform; relevant for metrics and trend analysis.                                   |
| HCI                              | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform.               |
| I/O                              | Input/Output; read and write operations.                                                                      |
| IOPS                             | Input/Output Operations Per Second; storage performance metric.                                               |
| Jira                             | Ticketing/workflow tool often used for incidents, bugs, and action tracking.                                  |
| Kibana                           | Log search and visualization tool.                                                                            |
| Latency                          | Time taken to complete an operation, often critical in storage performance.                                   |
| Logical Capacity                 | Capacity as seen by workloads before physical overhead or efficiency effects.                                 |
| MTTR                             | Mean Time To Repair/Recover; key incident-management metric.                                                  |
| NCC                              | Nutanix Cluster Check; health-check framework/tooling.                                                        |
| Node                             | Physical Nutanix server contributing compute and storage.                                                     |
| NVMe                             | High-performance storage device/interface type.                                                               |
| Oplog                            | Nutanix write-optimization area/cache used in the storage path.                                               |
| P1 / P2                          | Priority levels for severe support incidents.                                                                 |
| Physical Capacity                | Actual disk capacity consumed at the infrastructure layer.                                                    |
| Prism Central                    | Multi-cluster management and operations plane.                                                                |
| Prism Element                    | Cluster-level management interface.                                                                           |
| RAID                             | Traditional disk redundancy model; useful contrast with Nutanix distributed storage.                          |
| Raw Capacity                     | Total physical disk capacity before overheads.                                                                |
| Rebuild                          | Process of restoring data redundancy after disk/node failure.                                                 |
| Rebuild Capacity Reservation     | Reserved capacity for self-healing after failures.                                                            |
| Replication Factor / RF          | Number of data copies maintained for resiliency.                                                              |
| RF2                              | Replication Factor 2; typically two copies of data.                                                           |
| RF3                              | Replication Factor 3; typically three copies, higher resiliency, more capacity overhead.                      |
| Resiliency                       | Ability to tolerate failures while preserving data/service availability.                                      |
| Resilient Capacity               | Capacity level that accounts for ability to tolerate/recover from failures.                                   |
| SLA                              | Service Level Agreement; contractual or operational service target.                                           |
| SRE                              | Site Reliability Engineer/Engineering; reliability-focused engineering discipline.                            |
| SAN                              | Storage Area Network; traditional external block storage architecture.                                        |
| Snapshot                         | Point-in-time copy that can consume capacity as data changes.                                                 |
| SSD                              | Solid-State Drive; flash-based storage device.                                                                |
| Stargate                         | Nutanix service commonly associated with the data path.                                                       |
| Storage Container                | Logical storage object created within a Storage Pool.                                                         |
| Storage Pool                     | Aggregated physical disk pool across Nutanix cluster nodes.                                                   |
| Thin Provisioning                | Allocating logical capacity without consuming all physical capacity immediately.                              |
| Throughput                       | Amount of data transferred per unit of time.                                                                  |
| Usable Capacity                  | Capacity available after overheads such as metadata, resiliency, and rebuild reserve.                         |
| vDisk                            | Virtual disk object consumed by a VM.                                                                         |
| VM                               | Virtual Machine.                                                                                              |
| Volume Group                     | Nutanix block storage construct, often used for iSCSI/direct block use cases.                                 |
| Worldwide Support                | Global support organization handling enterprise customer issues across regions.                               |

[1]: https://portal.nutanix.com/page/documents/details?targetId=Command-Ref-AOS-v7_0%3Aacl-ncli-storagepool-auto-r.html&utm_source=chatgpt.com "AOS 7.0 - storagepool: Storage Pool - portal.nutanix.com"
[2]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_8%3Awc-storage-components-wc-c.html&utm_source=chatgpt.com "Prism 6.8 - Storage Components - Nutanix"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2096-Nutanix-Core-Performance%3Aaos-storage.html&utm_source=chatgpt.com "AOS Storage - Nutanix"
[4]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_5%3Awc-storage-management-wc-c.html&utm_source=chatgpt.com "Prism 6.5 - Storage Management - Nutanix"
[5]: https://portal.nutanix.com/page/documents/details?targetId=vSphere-Admin6-AOS-v7_0%3AvSphere-Admin6-AOS-v7_0&utm_source=chatgpt.com "AOS 7.0 - vSphere Administration Guide for AOS - portal.nutanix.com"
[6]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v7_0%3Awc-usable-capacity-r.html&utm_source=chatgpt.com "Prism 7.0 - Storage Overheads and Usable Capacity"
[7]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_8%3Aarc-redundancy-factor3-c.html&utm_source=chatgpt.com "Prism 6.8 - Redundancy Factor 3 - portal.nutanix.com"
[8]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_7%3Awc-storage-rebuild-capacity-reserve-wc-c.html&utm_source=chatgpt.com "Prism 6.7 - Rebuild Capacity Reservation - Nutanix"
[9]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2032-Data-Efficiency%3ATN-2032-Data-Efficiency&utm_source=chatgpt.com "Data Efficiency - Nutanix"
[10]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v7_3%3Awc-storage-overview-view-wc-r.html&utm_source=chatgpt.com "Prism 7.3 - Storage Overview View - portal.nutanix.com"
