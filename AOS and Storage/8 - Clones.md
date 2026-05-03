# [AOS / Storage] — Clones

## 1. Short definition

In Nutanix AOS, a **clone** is a space-efficient copy of a VM, vDisk, or volume that is typically created from snapshot technology rather than by fully copying all data blocks immediately.

For interview purposes: **a clone lets Nutanix create fast, storage-efficient copies of workloads by sharing unchanged data blocks and writing only new changes separately.**

Nutanix describes intelligent cloning as based on snapshots using a **redirect-on-write** mechanism. VMs store data as **vDisk files**, and a cloned VM uses shared base data plus new write metadata instead of duplicating the whole disk upfront. ([Nutanix Portal][1])

---

## 2. Clear explanation

A traditional full copy duplicates every block of data. If you clone a 1 TB VM, the platform may need to read and write close to 1 TB of data before the copy is complete.

A Nutanix clone behaves differently.

At the AOS storage layer, the VM disk is represented as one or more **vDisks**. These vDisks are divided into logical chunks and mapped to physical storage across the cluster. When a clone is created, Nutanix does not immediately duplicate every block. Instead, it creates a new logical disk structure that references the same underlying unchanged data as the source.

The important idea is:

> The source and the clone can initially share the same base data, but once either side writes new data, AOS redirects that write to new storage blocks and updates metadata accordingly.

This is why clones are usually very fast to create and space efficient at the beginning.

Nutanix documentation states that when a VM is cloned, the system marks the base vDisk read-only and creates another vDisk for reads and writes. ([Nutanix Portal][1]) Nutanix also documents that snapshots allocate writes on a new block rather than writing them to a change log, which helps avoid extra lookup overhead on writes. ([Nutanix Portal][2])

In Prism Central, cloning is exposed operationally as a VM action. A cloned VM inherits most of the source VM configuration, except the name. ([Nutanix Portal][3])

So, at support-manager level, you do **not** need to explain every internal metadata structure in detail. You do need to explain:

* clones are fast because they are metadata-efficient;
* clones are space-efficient because unchanged data is shared;
* writes after cloning consume additional capacity;
* clone chains and snapshots can affect troubleshooting, capacity, and performance conversations;
* cloning is operationally tied to VM lifecycle, templates, test/dev, backup/restore, DR, Kubernetes PVC workflows, and enterprise automation.

---

## 3. Why it matters for Nutanix / Worldwide Support

Clones matter because they sit at the intersection of **storage efficiency, VM operations, customer expectations, capacity planning, and incident escalation**.

For a Worldwide Support Manager at Nutanix, the topic is relevant because customers may escalate issues such as:

* “My VM clone is taking too long.”
* “The clone was created, but the VM does not boot.”
* “The cloned VM has duplicate hostname, IP, SID, or application identity.”
* “Capacity suddenly increased after cloning many VMs.”
* “Performance degraded after a large number of clones or snapshots.”
* “A clone from a snapshot is inconsistent.”
* “A Kubernetes PVC restored from snapshot is not behaving as expected.”
* “A VDI, test/dev, or lab environment created hundreds of clones and now the cluster is under pressure.”

This is not just a storage feature. It becomes a support-management problem because it can involve:

* AOS storage behavior;
* AHV or VMware hypervisor behavior;
* Prism workflows;
* guest OS customization;
* networking identity conflicts;
* application consistency;
* backup/DR expectations;
* capacity and performance KPIs;
* escalation between Support, SRE, Engineering, and Customer Success.

Your positioning should be:

> “I do not need to be the engineer writing the AOS metadata path, but I need to understand clone behavior deeply enough to triage impact, ask the right questions, coordinate storage/hypervisor/application teams, and communicate clearly with enterprise customers.”

---

## 4. Key concepts

### Clone vs full copy

A **full copy** duplicates all data. A **clone** initially references existing data and stores only changes separately. This makes clones faster and more space efficient, especially for test/dev, VDI, templates, and rapid provisioning.

### Clone vs snapshot

A **snapshot** is a point-in-time recovery object. A **clone** is usually a new usable object created from an existing VM, vDisk, image, or snapshot.

In simple interview language:

> “A snapshot preserves a point in time. A clone turns a source or snapshot into a usable copy.”

Nutanix snapshots use redirect-on-write behavior, and Nutanix clones are built on that snapshot-based mechanism. ([Nutanix Portal][4])

### Redirect-on-write

In redirect-on-write, existing blocks are not overwritten in place. New writes are directed to new blocks, and metadata is updated to point to the right version of the data.

This is important because it helps Nutanix create efficient snapshots and clones without immediately copying the full disk.

### vDisk

A **vDisk** is the AOS-level virtual disk object backing VM disk files. Nutanix documentation says vDisks at the Nutanix layer back the files AOS presents to VMs, and each vDisk is owned by a Controller VM. ([Nutanix Portal][4])

### CVM

The **Controller VM** is the Nutanix software component running on each node that provides storage services. In an escalation involving clones, the CVM layer may be relevant because AOS storage metadata, IO path, and vDisk ownership are handled through the distributed storage architecture.

### Metadata

Clones rely heavily on metadata. The platform needs to know which blocks belong to the source, which are shared, and which have diverged after writes.

A support manager does not need to debug metadata internals directly, but should know that clone performance and consistency issues may involve metadata operations, not only raw disk throughput.

### Space efficiency

Clones are space efficient at creation time, but they are not “free forever.” As cloned VMs write new data, additional blocks are consumed.

A common customer misunderstanding is:

> “I cloned a VM and expected no capacity impact.”

Correct support response:

> “The initial clone is space efficient, but post-clone writes consume additional capacity. We need to check the number of clones, write rate, snapshot retention, deduplication/compression behavior, and current cluster capacity headroom.”

### Application consistency

A storage-level clone may be crash-consistent, but that does not automatically mean the application inside the VM is cleanly quiesced.

For databases, clustered applications, domain controllers, or transactional systems, you need to consider guest tools, application quiescing, pre/post scripts, backup integration, and vendor-specific recovery practices.

### Guest customization

A cloned VM may inherit OS-level identity from the source. That can cause duplicate hostnames, duplicate IP addresses, duplicate machine IDs, duplicate Windows SIDs, duplicate application node IDs, or licensing problems.

This is especially important in enterprise support because the storage clone may be technically successful while the application outcome is still wrong.

### Prism

**Prism Central** and **Prism Element** expose clone operations operationally. Prism Central documentation states that a cloned VM inherits most source VM configuration except the name. ([Nutanix Portal][3])

That sentence is interview-relevant: it implies that post-clone customization matters.

### Kubernetes / CSI volume clones and snapshots

In cloud-native environments, clones may appear through persistent volume operations. Nutanix CSI documentation states that its CSI driver supports taking volume snapshots and restoring volumes, using Nutanix Volumes snapshot capability. ([Nutanix Portal][5])

For your profile, this is useful because you can connect Nutanix cloning concepts with Kubernetes PVC restore, test environments, and stateful workloads.

---

## 5. How it appears in a real escalation

### Scenario 1 — Clone created successfully, but application fails

A customer clones a production VM to create a test environment. Prism shows the clone was successful. The VM powers on, but the application fails because it has the same hostname, IP address, cluster identity, or database node ID as production.

The support risk is that the customer says:

> “Nutanix clone broke my application.”

A strong support manager response separates infrastructure success from guest/application readiness:

> “First, we validate whether the clone operation completed successfully at the AOS and hypervisor layer. Then we check whether the cloned guest was properly customized before joining the network or starting the application. Many clone incidents are not storage corruption; they are identity, network, or application-consistency issues.”

### Scenario 2 — Capacity grows after mass cloning

A customer creates hundreds of clones for VDI or QA. Initially everything looks fine. Later, capacity usage grows quickly.

Possible cause:

* many clones start writing unique data;
* snapshots are retained too long;
* deduplication/compression expectations were misunderstood;
* the cluster is near capacity thresholds;
* workloads are write-heavy.

Support angle:

> “The clone is efficient at creation time, but write divergence consumes new physical capacity. We need to review clone count, write rate, snapshot retention, storage efficiency savings, and cluster capacity runway.”

### Scenario 3 — Performance complaint after clone storm

A customer automates many clone operations in parallel. Suddenly they report high latency or VM provisioning delays.

Relevant triage:

* How many clones?
* Created from same source?
* Same container/storage policy?
* Same time window?
* Any concurrent snapshots, backups, replications, or upgrades?
* Cluster latency before and after?
* CVM CPU/memory pressure?
* Metadata service health?
* Hot source image or template?

Support-manager framing:

> “I would treat this as a capacity/performance and concurrency incident, not just a clone failure. I would stabilize the customer first, reduce concurrency if needed, check cluster health, and involve storage/SRE specialists if metadata or CVM resource pressure appears.”

### Scenario 4 — Clone from snapshot has inconsistent data

A customer clones from a snapshot and finds database corruption or missing transactions.

Important distinction:

* The Nutanix snapshot may be crash-consistent.
* The application may require application-consistent snapshotting.
* The customer may need guest quiescing, database backup mode, VSS, or application-aware tooling.

Support-manager answer:

> “I would avoid immediately calling this data corruption. I would verify the snapshot type, whether the guest/application was quiesced, the timing of the snapshot, and the application recovery logs.”

### Scenario 5 — Kubernetes PVC restored from snapshot does not behave

A team restores a stateful Kubernetes workload using Nutanix CSI snapshots. The PVC is restored, but the pod fails.

Potential causes:

* wrong StorageClass;
* access mode mismatch;
* filesystem issue;
* application-level inconsistency;
* pod security or mount permissions;
* CSI driver/version compatibility;
* volume attachment problem.

Connection to your cloud/SaaS background:

> “This looks similar to restoring a cloud disk snapshot in AWS, Azure, or GCP: the storage object may restore correctly, but the workload still depends on orchestration, identity, configuration, and application recovery.”

---

## 6. Triage questions I should ask

### Customer impact

1. What is the business impact: production outage, degraded performance, failed deployment, or test/dev issue?
2. How many VMs, volumes, or PVCs are affected?
3. Is this blocking a critical customer service or only an internal provisioning workflow?
4. When did the issue start, and what changed immediately before it?

### Clone scope

5. What exactly was cloned: VM, vDisk, volume, snapshot, template, image, or PVC?
6. Was the clone created from a powered-on VM, powered-off VM, template, image, or snapshot?
7. Was the source VM application-consistent or only crash-consistent?
8. Was guest customization applied after cloning?

### Platform context

9. Which hypervisor is involved: AHV, ESXi, or another supported environment?
10. Was the operation done through Prism Central, Prism Element, API, CLI, automation, or a third-party tool?
11. What AOS, Prism, AHV, CSI, or hypervisor versions are involved?
12. Is the issue reproducible with one clone, or only at scale?

### Storage and performance

13. What is the current cluster capacity utilization?
14. Are there active snapshots, replications, backups, or protection-domain operations?
15. Did latency, IOPS, throughput, or CVM resource usage change during the clone operation?
16. Are many clones writing heavily after creation?

### Guest/application layer

17. Does the cloned VM boot?
18. Are there duplicate IPs, hostnames, SIDs, machine IDs, certificates, or application IDs?
19. Are application logs showing storage errors, identity conflicts, licensing problems, or database recovery issues?
20. Was the cloned system connected to the same network as production before customization?

### Escalation management

21. Is there a workaround: create fewer clones, clone from a powered-off/golden image, restore from another snapshot, isolate network, or reduce concurrency?
22. Do we need SRE, storage engineering, hypervisor specialists, networking, or application owners?
23. What customer communication cadence is required?
24. What evidence do we need before escalating to Engineering?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you handle an escalation where a customer says Nutanix clones are consuming more storage than expected?
2. How would you communicate clone-related limitations to a non-technical customer executive?
3. How do you distinguish between a platform issue and a guest/application issue?
4. How would you coordinate Support, SRE, Engineering, and Account teams during a clone-related P1?
5. What KPIs would you monitor in a support team handling clone/snapshot incidents?
6. How would you coach an engineer who jumps too quickly to “it is not Nutanix”?
7. How would you manage customer expectations when the clone operation succeeded but the application failed?

### Technical / SRE interview

1. What is the difference between a clone and a snapshot?
2. Why are Nutanix clones space efficient?
3. What is redirect-on-write?
4. What risks exist when cloning a running VM?
5. What could cause capacity growth after creating many clones?
6. What would you check if clone operations are slow?
7. What would you check if a cloned VM does not boot?
8. How do clones relate to AOS vDisks?
9. What is the role of Prism in cloning?
10. How would cloning differ in AHV versus VMware operationally?

### Advanced panel / escalation interview

1. A customer created 500 clones from a golden image and now reports latency. How do you run the escalation?
2. A database clone is inconsistent. What do you ask first?
3. A customer claims Nutanix lost data after cloning from a snapshot. How do you respond?
4. A clone works on one cluster but fails on another. What variables do you compare?
5. A Kubernetes team restored a PVC using a Nutanix snapshot and the pod fails. How do you troubleshoot?
6. How do you decide when to escalate to Engineering?
7. How do you write the customer-facing update during a clone-related P1?

---

## 8. Model answers in English

### Question: What is a Nutanix clone?

**Model answer:**

“A Nutanix clone is a fast, space-efficient copy of a VM, vDisk, or volume that uses AOS snapshot-based storage mechanisms instead of immediately duplicating all blocks. At creation time, the clone can share unchanged base data with the source. As new writes occur, AOS writes changed data separately and updates metadata. From a support perspective, I would always distinguish between the storage clone operation and the guest or application readiness of the cloned workload.”

---

### Question: What is the difference between a snapshot and a clone?

**Model answer:**

“A snapshot is a point-in-time recovery object. A clone is a usable copy created from a source object or snapshot. In practical terms, a snapshot helps preserve or restore a state, while a clone is usually used to run a separate workload, create test/dev environments, provision VMs, or recover services. In support escalations, I would ask whether the issue is with snapshot creation, clone creation, VM boot, or application consistency, because those are different layers.”

---

### Question: Why are clones space efficient?

**Model answer:**

“They are space efficient because the platform does not need to copy every block immediately. The clone initially references existing unchanged data, and only new or changed writes consume additional physical capacity. That means clones are efficient at creation time, but they are not free indefinitely. In a production escalation, I would check write divergence, snapshot retention, deduplication/compression behavior, and overall capacity headroom.”

---

### Question: What is redirect-on-write?

**Model answer:**

“Redirect-on-write means that when data changes, the platform writes the new data to a new location rather than overwriting the existing block in place. Metadata is then updated to point to the correct version of the data. This is important for snapshots and clones because it enables efficient sharing of unchanged data while preserving point-in-time consistency.”

---

### Question: A customer says the clone succeeded but the application is broken. What do you do?

**Model answer:**

“I would separate the investigation into layers. First, I would validate the Nutanix operation: did the clone complete, does the VM exist, are the vDisks attached, does the VM power on, and are there platform alerts? Then I would check the guest and application layer: hostname, IP address, SID or machine ID, certificates, application cluster identity, database recovery logs, and whether guest customization was performed. I would communicate clearly that a successful infrastructure clone does not automatically guarantee application consistency or uniqueness.”

---

### Question: A customer created hundreds of clones and now capacity is growing. How would you explain it?

**Model answer:**

“I would explain that clones are initially space efficient because unchanged data is shared, but each clone consumes additional capacity as it writes unique data. If hundreds of clones are active and write-heavy, physical capacity can grow quickly. I would review the number of clones, write rate, snapshot retention, storage efficiency metrics, and cluster capacity thresholds. Operationally, I would also discuss provisioning controls, alerting, and lifecycle cleanup.”

---

### Question: A clone operation is slow. What would you check?

**Model answer:**

“I would first define what ‘slow’ means: clone creation time, VM registration, power-on time, guest boot, or application start. Then I would check cluster health, capacity utilization, latency, CVM resource usage, concurrent clone operations, snapshot or replication activity, and whether automation is creating many clones in parallel. I would also compare whether the issue happens from a specific source VM, template, storage container, cluster, or hypervisor.”

---

### Question: How would you handle a P1 escalation involving clones?

**Model answer:**

“I would run it like a structured incident. First, confirm business impact and stabilize the customer: stop risky automation, avoid powering on conflicting clones, and preserve logs. Second, split the investigation by layer: Prism workflow, AOS storage, hypervisor, guest OS, networking, and application. Third, assign clear owners: support engineer for case coordination, SRE/storage specialist for platform health, application owner for guest consistency, and account team for executive communication. Finally, maintain a predictable communication cadence and document facts, actions, next steps, and decision points.”

---

### Question: How technical do you need to be as a support manager?

**Model answer:**

“I need enough technical depth to ask the right questions, challenge assumptions, understand severity, and communicate credibly with both engineers and customers. I do not need to replace a Senior SRE in debugging internal metadata structures, but I do need to understand AOS clone behavior, capacity implications, consistency risks, and escalation paths. My value is combining technical fluency with incident leadership.”

---

## 9. Connection with my experience

Your current experience maps strongly to this topic.

At Harmonic, you already manage:

* 24/7 support operations;
* incident management;
* escalations;
* SLA and MTTR;
* monitoring;
* cloud operations;
* customer communication;
* cross-functional troubleshooting.

The bridge to Nutanix clones is:

| Your experience     | Nutanix clone relevance                                                                            |
| ------------------- | -------------------------------------------------------------------------------------------------- |
| Cloud operations    | Similar to managing EBS/Azure Disk/GCP persistent disk snapshots and restored volumes              |
| Incident management | Clone issues often become capacity, performance, or application recovery incidents                 |
| SLA / MTTR          | You need to reduce time to isolate whether the issue is storage, hypervisor, guest, or application |
| Monitoring          | Capacity, latency, IOPS, error rates, and job failures are key signals                             |
| Escalations         | Clone issues may require Support, SRE, Engineering, Account teams, and customer admins             |
| Kubernetes          | PVC snapshot/restore workflows map well to Nutanix CSI volume snapshot concepts                    |
| Grafana / Kibana    | Useful for correlating clone events with latency, capacity, workload, and error trends             |
| Jira / Confluence   | Strong for case documentation, RCA, runbooks, and support playbooks                                |
| Salesforce          | Relevant for case severity, customer communication history, and escalation governance              |

A strong way to position yourself:

> “My gap is Nutanix-specific implementation detail, but the operational pattern is familiar: fast provisioning, snapshot-based recovery, capacity side effects, consistency risks, and customer-facing incident management. I know how to run the escalation while quickly bringing in the right storage or SRE depth.”

---

## 10. Minimum I need to memorize

You should be able to say these points confidently:

1. **A clone is a usable copy; a snapshot is a point-in-time state.**
2. **Nutanix clones are space efficient because they share unchanged data at creation time.**
3. **AOS uses redirect-on-write snapshot technology for efficient snapshots and clones.**
4. **Post-clone writes consume additional physical capacity.**
5. **A successful clone does not guarantee guest or application consistency.**
6. **Cloned VMs may need customization: hostname, IP, SID, machine ID, certificates, app identity.**
7. **Clone issues must be triaged by layer: Prism, AOS, hypervisor, guest OS, network, application.**
8. **Capacity and performance incidents can appear after clone storms or write-heavy clones.**
9. **For databases and transactional systems, application consistency matters.**
10. **As a manager, your role is to coordinate, prioritize, communicate, and escalate with technical credibility.**

Verbal summary to memorize:

> “In Nutanix, clones are fast and space-efficient copies built on AOS snapshot technology. They initially share unchanged data with the source, and new writes consume additional capacity through redirect-on-write behavior. In support, the key is to separate clone creation from guest customization and application consistency. Many escalations are not simply ‘clone failed’; they are capacity, performance, identity, or application-recovery issues.”

---

## 11. Advanced / optional level

You can leave these as advanced unless the SRE interview goes deep:

* Internal AOS metadata structures.
* Detailed vDisk block map mechanics.
* Extents and extent groups.
* Deep CVM ownership and metadata-service behavior.
* Performance implications of deep snapshot/clone trees.
* Garbage collection and cleanup internals.
* Specific CLI commands for low-level snapshot/vDisk inspection.
* Engineering-level debugging of Stargate, Curator, Cassandra, or other internal services.
* Detailed differences between AHV, ESXi, and Hyper-V clone semantics.
* Advanced Nutanix Files, Objects, or Volumes clone behavior.
* API-level automation details unless asked.

However, it is worth knowing the vocabulary because senior SREs may mention it.

A good answer if pushed:

> “I understand the conceptual model: vDisks, metadata, shared base data, redirect-on-write, and post-clone divergence. I would not pretend to debug internal metadata structures without the right tools and SRE support, but I know which symptoms would indicate that we need deeper AOS investigation.”

---

## 12. Final checklist

Before an interview, check that you can answer these out loud:

* Can I explain clone vs snapshot in less than 30 seconds?
* Can I explain why clones are fast?
* Can I explain why clones still consume capacity over time?
* Can I name three risks after cloning a VM?
* Can I explain why a cloned database may be inconsistent?
* Can I triage “clone succeeded but VM does not work”?
* Can I triage “many clones caused capacity growth”?
* Can I triage “clone storm caused performance degradation”?
* Can I connect clones to AHV, Prism, AOS, vDisks, and CVMs?
* Can I explain this as a support manager, not only as an engineer?
* Can I provide a calm customer-facing explanation?
* Can I identify when to involve SRE or Engineering?

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                        |
| ------------------------ | ---------------------------------------------------------------------------------------------- |
| AHV                      | Acropolis Hypervisor; Nutanix native hypervisor                                                |
| AOS                      | Acropolis Operating System; Nutanix distributed storage and platform software                  |
| API                      | Application Programming Interface; used for automation and integration                         |
| Application consistency  | State where the application is safely quiesced or recoverable after clone/snapshot             |
| AWS                      | Amazon Web Services; public cloud platform                                                     |
| Azure                    | Microsoft public cloud platform                                                                |
| Backup                   | Copy or recovery mechanism used to restore data after failure                                  |
| Capacity headroom        | Free usable capacity available before risk thresholds are reached                              |
| Clone                    | Usable copy of a VM, disk, volume, or snapshot-derived object                                  |
| Clone storm              | Large number of clone operations executed in a short period                                    |
| Compression              | Storage efficiency technique that reduces physical data size                                   |
| Confluence               | Documentation and knowledge-base platform                                                      |
| Crash consistency        | Data state equivalent to sudden power loss; may require application recovery                   |
| CSI                      | Container Storage Interface; standard for Kubernetes storage plugins                           |
| CVM                      | Controller VM; Nutanix VM providing storage services on each node                              |
| Deduplication            | Storage efficiency technique that removes duplicate data blocks                                |
| DR                       | Disaster Recovery; recovery strategy after major failure                                       |
| EBS                      | Elastic Block Store; AWS block storage service                                                 |
| Engineering escalation   | Escalation to product engineering for suspected defect or deep product issue                   |
| Enterprise support       | Support function for business-critical customer environments                                   |
| ESXi                     | VMware hypervisor                                                                              |
| Extent                   | Logical or physical chunk of storage data used internally by distributed storage systems       |
| Full copy                | Complete duplication of all data blocks                                                        |
| GCP                      | Google Cloud Platform                                                                          |
| Golden image             | Standard base VM image used to create clones or templates                                      |
| Guest customization      | Post-clone changes to OS identity, hostname, network, or application settings                  |
| Guest OS                 | Operating system running inside a VM                                                           |
| HCI                      | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform |
| Hypervisor               | Software layer that runs virtual machines                                                      |
| IOPS                     | Input/Output Operations Per Second; storage performance metric                                 |
| Jira                     | Issue and workflow tracking platform                                                           |
| Kubernetes               | Container orchestration platform                                                               |
| Latency                  | Time taken for an operation, often storage read/write response time                            |
| Machine ID               | Unique OS or application identity that may need regeneration after cloning                     |
| Metadata                 | Data describing where blocks are stored and how objects reference them                         |
| MTTR                     | Mean Time To Restore/Resolve; incident recovery metric                                         |
| Nutanix Volumes          | Nutanix block storage service                                                                  |
| P1                       | Priority 1 incident; usually critical business impact                                          |
| PVC                      | Persistent Volume Claim; Kubernetes request for persistent storage                             |
| Prism Central            | Centralized Nutanix management plane                                                           |
| Prism Element            | Cluster-level Nutanix management interface                                                     |
| Quiescing                | Pausing or flushing application/OS activity to create a consistent state                       |
| RCA                      | Root Cause Analysis                                                                            |
| Redirect-on-write        | Technique where new writes go to new blocks instead of overwriting existing ones               |
| Replication              | Copying data to another location for resilience or DR                                          |
| Restore                  | Recovery of data or workload from backup, snapshot, or clone                                   |
| SID                      | Security Identifier; Windows identity value that may duplicate after cloning                   |
| SLA                      | Service Level Agreement                                                                        |
| Snapshot                 | Point-in-time state of data used for recovery or cloning                                       |
| Snapshot retention       | Policy defining how long snapshots are kept                                                    |
| SRE                      | Site Reliability Engineering                                                                   |
| Storage container        | Nutanix logical storage grouping for VM disks and policies                                     |
| Storage efficiency       | Reduction of physical capacity usage through clone sharing, compression, or deduplication      |
| Template                 | Reusable VM base configuration for deployment                                                  |
| Test/dev                 | Non-production environment used for testing and development                                    |
| Throughput               | Amount of data transferred per unit of time                                                    |
| Triage                   | Initial structured investigation to classify and prioritize an issue                           |
| vDisk                    | Nutanix virtual disk object backing VM disk data                                               |
| VDI                      | Virtual Desktop Infrastructure                                                                 |
| VM                       | Virtual Machine                                                                                |
| VSS                      | Volume Shadow Copy Service; Microsoft framework for consistent snapshots                       |
| Write divergence         | Growth of unique data after clones start writing changes                                       |

[1]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2032-Data-Efficiency%3Aintelligent-cloning.html&utm_source=chatgpt.com "Intelligent Cloning - portal.nutanix.com"
[2]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2096-Nutanix-Core-Performance%3Ahigh-performance-snapshots-and-clones.html&utm_source=chatgpt.com "High-Performance Snapshots and Clones - Nutanix"
[3]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_7_3%3Amul-vm-clone-acropolis-pc-t.html&utm_source=chatgpt.com "Prism pc.7.3 - Cloning a VM through Prism Central (AHV)"
[4]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2005-Data-Protection%3Anutanix-snapshots.html&utm_source=chatgpt.com "Nutanix Snapshots"
[5]: https://portal.nutanix.com/docs/CSI-Volume-Driver-v3_3%3Acsi-csi-driver-create-snapshot-c.html?utm_source=chatgpt.com "Third-Party Integrations 3.3 - Volume Snapshots (Nutanix Volumes)"
