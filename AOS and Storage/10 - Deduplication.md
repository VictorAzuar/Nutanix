# [AOS / Storage] — Deduplication

## 1. Short definition

**Deduplication** is a storage efficiency technique that reduces physical capacity usage by detecting duplicate data blocks and storing only one copy, while maintaining logical references for the VMs or workloads that need that data.

In Nutanix AOS, deduplication is part of the broader **AOS Storage data efficiency** stack, alongside thin provisioning, intelligent cloning, compression, and erasure coding. Nutanix documentation describes AOS Storage as using these techniques to improve capacity efficiency while scaling with the cluster architecture. ([Nutanix Portal][1])

---

## 2. Clear explanation

In a virtualized or HCI environment, many workloads contain repeated data: operating system files, application binaries, golden images, templates, cloned VMs, log patterns, and repeated blocks across similar workloads. Deduplication tries to identify those repeated blocks and avoid storing them multiple times.

In Nutanix, the important thing is not just “dedupe saves space.” The interview-relevant explanation is:

> Deduplication is a capacity optimization mechanism applied at the storage layer, typically configured at the storage container level, that can reduce physical storage consumption for workloads with duplicate block patterns. It must be evaluated together with workload type, performance impact, cluster sizing, memory/metadata requirements, and operational risk.

Nutanix AOS uses a distributed storage architecture. Local disks from each node are aggregated into a **storage pool**, and **storage containers** are created inside that pool. Different policies such as compression, deduplication, and replication factor can be applied at the container level. ([Nutanix Portal][2])

A practical mental model:

1. A VM writes data.
2. AOS processes the write through the distributed storage stack.
3. If deduplication is enabled and applicable, AOS fingerprints data blocks.
4. If an identical block already exists, the system can reference the existing block instead of writing another full physical copy.
5. The VM still sees its full logical disk. The saving happens underneath, in the storage layer.

Nutanix also documents a **Capacity Optimization Engine**, which increases effective storage capacity through techniques such as compression, deduplication, and erasure coding, using inline or post-process mechanisms depending on the feature and workload behavior. ([Nutanix Portal][3])

The key support nuance: **dedupe is not free**. It can save capacity, but it introduces metadata, fingerprinting, CPU/memory overhead, and operational considerations. For that reason, it should not be treated as a universal “enable everywhere” feature.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, deduplication matters because it sits exactly at the intersection of:

* customer cost pressure,
* capacity planning,
* storage performance,
* production risk,
* escalation handling,
* expectation management,
* and cross-functional troubleshooting.

A customer may open an escalation saying:

> “We enabled deduplication and expected 40% savings, but we only see 5%.”

Or:

> “After enabling dedupe, replication became slower.”

Or:

> “The cluster is running out of capacity even though Prism shows data reduction.”

As a support manager, you are not expected to personally debug every low-level AOS internal, but you must be able to guide the escalation correctly:

* confirm whether dedupe is supported for the cluster and version,
* check where it is enabled,
* understand the workload profile,
* avoid overpromising savings,
* separate capacity questions from performance questions,
* involve SREs / engineering when metadata, Curator, Stargate, or upgrade-related symptoms appear,
* communicate clearly to the customer.

Nutanix documentation specifically notes that capacity deduplication is enabled at the **storage container** level and also includes operational recommendations, such as CVM memory and metadata disk sizing requirements for capacity deduplication in certain versions. ([Nutanix Portal][4])

A strong manager-level answer should sound like this:

> “I would not position deduplication as a generic fix for capacity pressure. I would first validate workload suitability, cluster health, AOS version, container settings, data reduction metrics, replication impact, and whether the observed savings match the data pattern. Then I would coordinate support, SRE, and possibly engineering around evidence rather than assumptions.”

---

## 4. Key concepts

### 4.1 Logical capacity vs physical capacity

The VM sees **logical capacity**: for example, a 1 TB virtual disk.

The cluster consumes **physical capacity**: the real amount stored after replication, compression, deduplication, snapshots, metadata, and erasure coding.

Deduplication helps reduce physical consumption, but the saving is workload-dependent.

Example:

* 100 Windows VMs from the same golden image: high dedupe potential.
* encrypted database files: low dedupe potential.
* compressed video files: low dedupe potential.
* random data: almost no dedupe potential.

This is very relevant to your Harmonic background: media/video workloads are often already compressed, so the dedupe ratio may be poor. That is a good interview point.

---

### 4.2 Deduplication vs compression

These are related but different:

| Feature        | What it does                              | Best suited for                             |
| -------------- | ----------------------------------------- | ------------------------------------------- |
| Deduplication  | Removes duplicate blocks                  | Repeated data across VMs, clones, templates |
| Compression    | Reduces size of individual data blocks    | Compressible data patterns                  |
| Erasure coding | Reduces replication overhead using parity | Cold / capacity-focused data                |

Nutanix explicitly treats compression, deduplication, and erasure coding as separate data reduction techniques within AOS Storage. ([Nutanix Portal][3])

A good interview distinction:

> “Compression reduces the size of data; deduplication reduces repeated copies of the same data.”

---

### 4.3 Container-level policy

In Nutanix AOS, storage optimization features are commonly configured at the **storage container** level. A container is a logical storage construct presented to the hypervisor and used to host VMs. Nutanix documentation states that compression, deduplication, and replication factor policies can be applied per storage container. ([Nutanix Portal][2])

This matters operationally because different workloads may need different policies.

Example:

* VDI / cloned desktops: dedupe may make sense.
* production database: evaluate carefully.
* backup/archive container: maybe compression or erasure coding may be more relevant.
* latency-sensitive workload: avoid enabling features blindly.

---

### 4.4 Workload suitability

Deduplication works best when data has a lot of repetition.

Good candidates:

* VDI,
* full-clone persistent desktops,
* VM templates,
* OS images,
* application farms with similar binaries,
* P2V migrations with repeated operating system blocks.

Poor candidates:

* encrypted data,
* compressed media,
* compressed backups,
* random data,
* high-change databases,
* already deduplicated backup repositories,
* write-heavy workloads with little block commonality.

A support manager should not say “dedupe improves performance.” It may improve effective capacity and sometimes read-cache efficiency depending on configuration, but capacity dedupe is mainly a storage efficiency feature.

---

### 4.5 Cluster requirements and operational constraints

Nutanix documentation for Prism 6.10 states that deduplication reduces space usage by consolidating duplicate data blocks when capacity deduplication is enabled on a storage container. It also notes that deduplication is supported only on clusters with a minimum of three nodes, and that enabling dedupe on storage containers with protected VMs can lower replication speed. ([Nutanix Portal][5])

That last sentence is highly interview-relevant. It shows maturity: dedupe may interact with **data protection** and **replication** workflows.

A manager-level interpretation:

> “Before enabling dedupe in production, I would check not only capacity savings but also the impact on replication, backup windows, disaster recovery RPO/RTO, and customer SLAs.”

---

### 4.6 Metadata and CVM sizing

Deduplication requires metadata and fingerprinting. That means CVM resources matter.

Nutanix’s Prism 7.3 documentation states that Nutanix recommends Controller VMs with at least **32 GiB RAM** and **300 GiB SSDs for the metadata disk** to enable capacity deduplication in the documented scenario. It also notes that on-disk deduplication may be auto-disabled on some fresh AOS 7.3 deployments when dense-node capacity exceeds 92 TB per node. ([Nutanix Portal][4])

Do not memorize this as a universal rule for every version forever. Memorize it as an example of the support principle:

> “Dedupe depends on version-specific supportability, CVM resources, metadata capacity, and platform limits. I would always validate the current Nutanix documentation for the customer’s exact AOS version and hardware profile.”

---

### 4.7 Deduplication and performance

Deduplication can introduce overhead because the platform needs to calculate fingerprints, maintain metadata, and manage references. The practical support question is:

> Is the customer’s problem capacity efficiency, or is it performance degradation after enabling a feature?

For a performance escalation, you would check:

* when dedupe was enabled,
* whether latency changed before/after,
* affected workloads,
* CVM CPU and memory,
* storage latency,
* I/O pattern,
* Curator activity,
* Stargate / storage service health,
* cluster capacity headroom,
* ongoing replication or snapshot activity,
* AOS version and known issues.

Nutanix’s performance architecture emphasizes local storage resources and local read servicing to reduce latency and avoid traditional SAN/NAS controller bottlenecks. ([Nutanix Portal][6]) Deduplication troubleshooting should therefore be framed inside the broader distributed AOS storage architecture, not as a standalone storage-array feature.

---

## 5. How it appears in a real escalation

### Scenario 1 — “We enabled dedupe but savings are poor”

Customer says:

> “We enabled deduplication on the container, but Prism shows almost no reduction. Is Nutanix dedupe broken?”

Likely causes:

* workload has low duplicate block patterns,
* data is encrypted or compressed,
* dedupe has not completed post-process optimization yet,
* wrong container,
* insufficient data age / Curator cycle,
* metrics misunderstood,
* snapshots or replicas consuming additional space,
* customer confusing logical and physical usage.

Manager-level response:

> “I would first validate whether the workload is a good dedupe candidate. Then I would check the container configuration, data reduction metrics, time since enablement, Curator activity, and whether other factors such as snapshots, replication, or reserved capacity are affecting the view. I would avoid calling it a product issue until we have evidence that the expected duplicate patterns actually exist.”

---

### Scenario 2 — “Replication slowed down after enabling dedupe”

Customer says:

> “After enabling dedupe, our DR replication speed decreased.”

This aligns with Nutanix documentation noting that if deduplication is enabled on storage containers with protected VMs, the system lowers replication speed. ([Nutanix Portal][5])

Support framing:

* confirm protected VMs are on dedupe-enabled container,
* check replication schedule and backlog,
* compare before/after replication throughput,
* validate RPO impact,
* review capacity pressure vs DR requirements,
* decide whether dedupe benefits justify replication impact.

Manager-level answer:

> “I would treat this as a trade-off between capacity efficiency and data protection performance. If the customer’s RPO is at risk, I would prioritize replication health and involve the right technical team to validate whether dedupe should remain enabled for that protected workload.”

---

### Scenario 3 — “Cluster is nearly full; customer wants to enable dedupe urgently”

Customer says:

> “We are at 88% capacity. Can we enable dedupe to avoid buying nodes?”

Risks:

* dedupe savings may not be immediate,
* workload may not dedupe well,
* capacity pressure itself can degrade cluster behavior,
* metadata overhead matters,
* enablement may increase background activity,
* emergency capacity issues need conservative handling.

Good support manager response:

> “I would not present dedupe as an emergency capacity rescue without validation. I would first assess growth rate, reclaimable space, snapshots, garbage collection, orphaned data, replication, overprovisioning, and whether adding capacity is required. Dedupe can be part of the plan, but it should not be the only mitigation.”

---

### Scenario 4 — “Performance degraded after storage efficiency changes”

Customer says:

> “Latency increased after enabling dedupe and compression.”

Triage direction:

* compare latency before/after,
* check if change correlates with Curator scans or optimization jobs,
* inspect CVM resource pressure,
* validate affected workloads,
* check container policy,
* confirm whether dedupe, compression, or another event is the actual cause.

Manager-level angle:

> “I would separate correlation from causation. I would ask the SREs to collect performance data, timeline the change, and compare workload behavior before and after. In parallel, I would keep the customer updated with evidence-based checkpoints.”

---

## 6. Triage questions I should ask

### Business / impact questions

1. What is the customer impact: capacity alert, VM latency, replication delay, or failed operation?
2. Which applications or tenants are affected?
3. Is there an SLA, RPO, RTO, or production deadline at risk?
4. Is this a new deployment, migration, upgrade, or recent configuration change?
5. Is the customer asking for immediate mitigation or root cause?

### Nutanix configuration questions

1. Which AOS version and Prism version are running?
2. On which storage container is deduplication enabled?
3. Is dedupe enabled for capacity, cache, or both, depending on version/options?
4. How many nodes are in the cluster?
5. What is the hardware profile: all-flash, hybrid, dense node, CVM memory, metadata disk size?
6. Are protected VMs or replication policies involved?
7. Are compression or erasure coding also enabled?

### Workload questions

1. What workload type is stored in the container?
2. Is the data encrypted?
3. Is the data already compressed?
4. Is it VDI, cloned VM, database, backup, file server, media, logs, or Kubernetes persistent volumes?
5. Is the workload read-heavy, write-heavy, random, or sequential?
6. How much duplicate data should we realistically expect?

### Observability / troubleshooting questions

1. What are the current logical usage, physical usage, and data reduction ratio?
2. Did savings appear after a Curator scan or optimization cycle?
3. Are there capacity alerts?
4. Are there CVM CPU, memory, or disk bottlenecks?
5. Are there increased storage latencies?
6. Are replication jobs delayed?
7. Are snapshots or protection domains contributing to capacity usage?
8. Are NCC checks clean?
9. Are there known issues for this exact AOS version?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you explain deduplication to an enterprise customer?
2. A customer enabled dedupe and did not get the expected savings. How would you handle the escalation?
3. How would you balance customer pressure for capacity savings with performance or replication risk?
4. How would you coordinate between support engineers, SREs, product engineering, and account teams?
5. How would you communicate uncertainty to a frustrated customer?
6. What KPIs would you track during a dedupe-related escalation?
7. How would you avoid overpromising data reduction benefits?

### Technical / SRE interview

1. What is the difference between deduplication and compression?
2. Where is deduplication configured in Nutanix?
3. What workloads are good candidates for dedupe?
4. What workloads are poor candidates?
5. What are the possible performance impacts of dedupe?
6. How can dedupe affect replication or protected VMs?
7. What would you check if dedupe savings are lower than expected?
8. How does dedupe fit into AOS distributed storage?
9. What role do CVM resources and metadata play?
10. How would you triage high latency after enabling dedupe?

### Panel / escalation interview

1. A Fortune 500 customer is close to full capacity and asks to enable dedupe immediately. What do you do?
2. A customer claims Nutanix dedupe is broken. How do you prove or disprove that?
3. Dedupe saves capacity but replication RPO is now breached. What is your decision framework?
4. Support says it is workload behavior; the customer says it is a Nutanix defect. How do you manage the conversation?
5. Engineering needs logs but the customer wants an ETA. How do you communicate?

---

## 8. Model answers in English

### Q1. What is deduplication in Nutanix AOS?

**Model answer:**

> Deduplication in Nutanix AOS is a storage efficiency capability that reduces physical capacity consumption by identifying duplicate data blocks and storing only one physical copy while maintaining logical references for the workloads. In Nutanix, it should be understood as part of the AOS data efficiency stack, together with compression, thin provisioning, cloning, and erasure coding. From a support perspective, the key point is that dedupe effectiveness depends heavily on workload characteristics, container configuration, cluster resources, and the customer’s operational requirements.

---

### Q2. How is deduplication different from compression?

**Model answer:**

> Compression reduces the size of individual data blocks by encoding them more efficiently. Deduplication removes duplicate copies of identical blocks. So compression is about making data smaller, while dedupe is about avoiding storing the same data multiple times. In an enterprise support case, I would evaluate both separately because they have different benefits, overheads, and workload suitability.

---

### Q3. Where is deduplication configured in Nutanix?

**Model answer:**

> Deduplication is typically configured at the storage container level. AOS aggregates local disks into a storage pool, and storage containers are created inside that pool. Policies such as compression, deduplication, and replication factor can be applied to containers. That matters because not all workloads should necessarily share the same storage efficiency policy.

---

### Q4. What workloads benefit most from deduplication?

**Model answer:**

> Workloads with repeated blocks benefit most: VDI, VM templates, cloned VMs, operating system images, application farms, or migration scenarios where many systems have similar binaries. Workloads such as encrypted data, compressed media, random data, or already deduplicated backup repositories usually have much lower dedupe potential.

---

### Q5. A customer says dedupe savings are much lower than expected. What would you do?

**Model answer:**

> I would first validate the expectation. Dedupe is workload-dependent, so low savings do not automatically mean a product issue. I would check the workload type, whether the data is encrypted or compressed, the storage container settings, AOS version, time since dedupe was enabled, data reduction metrics, Curator activity, snapshots, and replication. I would also compare logical versus physical usage and explain to the customer what the platform is actually reporting. If the data pattern should produce savings but does not, I would escalate with evidence to the appropriate SRE or engineering team.

---

### Q6. Could deduplication affect performance?

**Model answer:**

> Yes, potentially. Deduplication requires fingerprinting and metadata management, so it can introduce overhead depending on the workload, cluster resources, and implementation path. I would not assume dedupe is the cause of a performance issue, but I would include it in the timeline. I would compare before-and-after metrics, check CVM CPU and memory, storage latency, I/O patterns, Curator activity, and whether other changes such as compression, snapshots, replication, or upgrades happened at the same time.

---

### Q7. What would you do if dedupe affects replication?

**Model answer:**

> I would treat it as a trade-off between capacity efficiency and data protection performance. Nutanix documentation notes that dedupe on containers with protected VMs can reduce replication speed, so I would validate whether protected VMs are involved, check replication backlog, RPO compliance, and customer business impact. If RPO is at risk, I would prioritize data protection and coordinate with SREs and the account team to decide whether dedupe should remain enabled for that workload.

---

### Q8. How would you explain dedupe to a non-technical executive customer?

**Model answer:**

> I would say: deduplication is like avoiding storing the same document many times. If many systems contain the same data, Nutanix can store one copy and reference it multiple times, reducing physical storage consumption. But the benefit depends on the type of data. If the data is already compressed, encrypted, or unique, there may be little to deduplicate. So we need to validate the workload before promising a specific saving.

---

### Q9. How would you lead this escalation as a manager rather than as an individual contributor?

**Model answer:**

> My role would be to structure the escalation, assign clear ownership, keep the customer informed, and make sure the technical investigation is evidence-based. I would ask the team to confirm configuration, workload suitability, cluster health, data reduction metrics, and timeline. I would also manage expectations with the customer: dedupe is not a guaranteed percentage saving, and any decision must consider capacity, performance, and replication impact. If needed, I would bring in SRE, product engineering, and the account team with a clear problem statement and current evidence.

---

### Q10. What should be avoided when discussing deduplication with customers?

**Model answer:**

> I would avoid promising a fixed reduction ratio, recommending dedupe blindly for all workloads, or treating it as an emergency substitute for capacity planning. I would also avoid ignoring side effects such as metadata overhead, performance impact, or replication behavior. The right approach is to validate workload suitability, cluster health, and business priorities before recommending a configuration change.

---

## 9. Connection with my experience

Your background maps very well to this topic if you position it correctly.

At Harmonic, you lead **24/7 enterprise support operations**, which means you already understand:

* incident ownership,
* SLA impact,
* escalation paths,
* MTTR,
* customer communication,
* monitoring,
* capacity pressure,
* cloud operations,
* post-incident review,
* and coordinating technical teams under pressure.

The Nutanix-specific gap is the storage architecture vocabulary: AOS, CVM, Prism, storage container, dedupe, compression, Curator, Stargate, replication factor, protected VMs, and distributed storage.

You can bridge the gap like this:

> “In my current role, I often deal with production incidents where customers experience performance degradation, capacity pressure, or SLA risk. The technology stack is different, but the escalation discipline is the same: define impact, establish timeline, collect metrics, validate recent changes, isolate variables, communicate clearly, and coordinate specialists. For Nutanix deduplication, I would apply that same operational model while relying on AOS-specific checks around container policies, workload suitability, CVM health, storage latency, and replication impact.”

Your media/SaaS background is also useful because video and media workloads often involve compressed data. That gives you a mature observation:

> “I know from media environments that not all data reduction techniques produce meaningful savings. For example, compressed media may not deduplicate or compress well, so I would avoid assuming generic storage efficiency ratios.”

That is exactly the kind of answer that makes you sound like a technical escalation manager, not someone memorizing Nutanix terms.

---

## 10. Minimum I need to memorize

Memorize these points:

1. **Deduplication removes duplicate blocks; compression reduces block size.**
2. **In Nutanix, dedupe is part of AOS Storage data efficiency.**
3. **It is commonly configured at the storage container level.**
4. **Best candidates:** VDI, clones, templates, similar OS/application images.
5. **Poor candidates:** encrypted, compressed, random, media, already deduped backup data.
6. **Savings are workload-dependent; never promise a fixed ratio.**
7. **Dedupe can introduce metadata and resource overhead.**
8. **Check CVM resources, AOS version, container config, workload type, and cluster health.**
9. **Protected VMs / replication can be affected; Nutanix documentation notes reduced replication speed when dedupe is enabled on containers with protected VMs.** ([Nutanix Portal][5])
10. **As a manager, your job is to coordinate evidence-based escalation, not personally debug every internal component.**

Verbal summary to practice:

> “Deduplication in Nutanix AOS is a storage efficiency feature that reduces physical capacity by avoiding duplicate block storage. It is useful for workloads with repeated data, such as VDI or cloned VMs, but it is not a universal fix for capacity issues. In support, I would validate workload suitability, container configuration, AOS version, cluster health, CVM resources, data reduction metrics, and any impact on replication or performance before making recommendations.”

---

## 11. Advanced / optional level

You do **not** need to master these deeply for a manager interview, but you should recognize the terms.

### Advanced areas to understand lightly

* **Fingerprinting**: how blocks are identified as duplicates.
* **Metadata overhead**: dedupe requires metadata structures to track references.
* **Curator**: Nutanix background service involved in scans, cleanup, and storage optimization workflows.
* **Stargate**: AOS data path service responsible for I/O handling.
* **Extent Store**: persistent storage layer where data is stored.
* **OpLog**: write buffer / staging area used in the AOS I/O path.
* **Unified Cache**: caching layer that can interact with read performance.
* **Post-process vs inline behavior**: whether optimization happens during write or later.
* **Data reduction reporting**: how Prism reports logical usage, physical usage, and savings.
* **Interaction with snapshots and replication**.
* **Version-specific limitations**: supportability may vary by AOS/Prism version and hardware profile.

Useful advanced phrase:

> “I would not troubleshoot dedupe in isolation. I would correlate Prism metrics, NCC health, CVM resource utilization, Curator activity, storage latency, replication state, and workload characteristics.”

---

## 12. Final checklist

Before an interview, make sure you can answer:

* Can I explain dedupe in one sentence?
* Can I distinguish dedupe from compression?
* Can I explain why dedupe is workload-dependent?
* Can I name good and bad dedupe candidates?
* Can I explain why enabling dedupe blindly is risky?
* Can I explain where dedupe is configured in Nutanix?
* Can I connect dedupe to storage containers and AOS?
* Can I describe a real escalation involving poor savings?
* Can I describe a real escalation involving replication impact?
* Can I explain how I would manage the customer conversation?
* Can I position myself as a technical escalation manager, not a low-level storage engineer?

Strong final interview line:

> “For me, deduplication is not just a feature checkbox. It is an operational decision involving workload behavior, capacity economics, performance risk, replication impact, and customer expectations. As a support manager, I would make sure we validate those dimensions before recommending or changing anything in production.”

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword           | Meaning                                                                                                           |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| AOS                                | Acropolis Operating System; Nutanix’s core software platform for storage, virtualization, and cluster services    |
| AOS Storage                        | Nutanix distributed storage layer used by workloads running on the cluster                                        |
| AHV                                | Acropolis Hypervisor; Nutanix’s native hypervisor                                                                 |
| Capacity deduplication             | Deduplication aimed at reducing physical storage capacity usage                                                   |
| Capacity Optimization Engine / COE | AOS mechanism for improving storage efficiency using data reduction techniques                                    |
| Compression                        | Data reduction technique that reduces the size of data blocks                                                     |
| Container                          | Logical storage unit in Nutanix where policies such as dedupe, compression, and replication factor may be applied |
| Controller VM / CVM                | Nutanix virtual machine on each node that runs storage and cluster services                                       |
| Curator                            | Nutanix background service involved in scans, cleanup, and storage optimization                                   |
| Data efficiency                    | General category including thin provisioning, cloning, compression, deduplication, and erasure coding             |
| Data reduction                     | Techniques that reduce physical storage consumption                                                               |
| Deduplication                      | Storage efficiency technique that stores one copy of duplicate data blocks                                        |
| Dense node                         | Nutanix node with high storage capacity; relevant to some dedupe support considerations                           |
| DR                                 | Disaster Recovery; recovery strategy after major failure                                                          |
| EC-X                               | Nutanix erasure coding feature for storage capacity efficiency                                                    |
| Encryption                         | Process that transforms data; encrypted data usually deduplicates poorly                                          |
| Enterprise support                 | Support model for business-critical customer environments                                                         |
| Erasure coding                     | Capacity efficiency technique that uses parity instead of full duplicate replicas                                 |
| Escalation                         | Formal process for raising a customer issue to higher technical or management levels                              |
| Extent Store                       | Persistent storage layer in the Nutanix storage architecture                                                      |
| Fingerprinting                     | Process of creating identifiers for data blocks to detect duplicates                                              |
| Golden image                       | Standard VM image used to create many similar VMs                                                                 |
| HCI                                | Hyper-Converged Infrastructure; combines compute, storage, networking, and virtualization                         |
| I/O                                | Input/Output operations between workloads and storage                                                             |
| Inline                             | Processing performed during the write path                                                                        |
| KPI                                | Key Performance Indicator; metric used to track operational performance                                           |
| Logical capacity                   | Capacity seen by the VM or application before storage reduction                                                   |
| Metadata                           | Data used by the system to track blocks, references, and storage layout                                           |
| Metadata disk                      | Disk used by the CVM for metadata; relevant for dedupe supportability                                             |
| MTTR                               | Mean Time To Repair / Resolve; average time to restore service                                                    |
| NCC                                | Nutanix Cluster Check; health-check tool for Nutanix clusters                                                     |
| OpLog                              | Nutanix write buffer / staging component in the storage I/O path                                                  |
| Physical capacity                  | Actual storage consumed on the cluster after reduction, replicas, snapshots, and metadata                         |
| Prism                              | Nutanix management interface                                                                                      |
| Protected VM                       | VM covered by Nutanix data protection or replication configuration                                                |
| RPO                                | Recovery Point Objective; acceptable amount of data loss measured in time                                         |
| RTO                                | Recovery Time Objective; acceptable time to restore service                                                       |
| Replication                        | Copying data to another location or cluster for protection or disaster recovery                                   |
| Replication factor / RF            | Number of copies maintained for data availability                                                                 |
| SLA                                | Service Level Agreement; contractual or operational service commitment                                            |
| Snapshot                           | Point-in-time copy of VM or data state                                                                            |
| SRE                                | Site Reliability Engineer; role focused on reliability, automation, and production operations                     |
| Stargate                           | Nutanix service involved in the storage data path                                                                 |
| Storage container                  | Logical storage construct where VMs are placed and policies are applied                                           |
| Storage pool                       | Aggregation of physical storage resources across Nutanix nodes                                                    |
| Thin provisioning                  | Allocating logical capacity without consuming all physical capacity immediately                                   |
| Unified Cache                      | Nutanix caching layer used in the storage architecture                                                            |
| VDI                                | Virtual Desktop Infrastructure; often a good dedupe candidate                                                     |
| VM                                 | Virtual Machine                                                                                                   |
| Workload suitability               | Assessment of whether a workload is appropriate for a feature such as dedupe                                      |

[1]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2032-Data-Efficiency%3ATN-2032-Data-Efficiency&utm_source=chatgpt.com "Data Efficiency - Nutanix"
[2]: https://portal.nutanix.com/docs/vSphere-Admin6-AOS-v7_5%3Avsp-cluster-configuration-vsphere-r.html?utm_source=chatgpt.com "AOS 7.5 - Nutanix Software Configuration"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2032-Data-Efficiency%3Aaos-storage-data-reduction.html&utm_source=chatgpt.com "AOS Storage Data Reduction - portal.nutanix.com"
[4]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_3%3Asto-dedup-recommend-c.html?utm_source=chatgpt.com "Prism 7.3 - Deduplication - portal.nutanix.com"
[5]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_10%3Asto-dedup-recommend-c.html&utm_source=chatgpt.com "Prism 6.10 - Deduplication - portal.nutanix.com"
[6]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2096-Nutanix-Core-Performance%3Aaos-storage.html&utm_source=chatgpt.com "AOS Storage - Nutanix"
