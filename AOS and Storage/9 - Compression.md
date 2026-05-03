# [AOS / Storage] — Compression

## 1. Short definition

**Compression in Nutanix AOS** is a storage-efficiency feature that reduces the physical capacity consumed by VM data by storing data in compressed form. In Nutanix, compression is part of the broader **AOS Storage data efficiency** stack, together with thin provisioning, clones, deduplication, and erasure coding. Nutanix supports both **inline compression** and **post-process compression**. Official Nutanix documentation states that inline compression compresses large or sequential I/O as it is written to the **Extent Store**, while post-process compression first writes data uncompressed and later compresses it cluster-wide using the **Curator** framework. ([Nutanix Portal][1])

---

## 2. Clear explanation

In a Nutanix cluster, storage is distributed across nodes and managed by AOS. Each node runs a **Controller VM**, or **CVM**, which handles storage I/O for the VMs on that host. Nutanix aggregates local disks across nodes into a distributed storage pool. AOS is responsible for preserving data availability and integrity across disk, node, hypervisor, and software failures. ([Nutanix Portal][2])

Compression sits in that storage layer as a **capacity optimization mechanism**.

At a practical level:

* A VM writes data.
* AOS receives the write through the local CVM.
* Depending on the I/O pattern, the data may go through the **OpLog** or directly to the **Extent Store**.
* Compression may happen immediately, or later in the background.
* Reads of compressed data require decompression before the data is returned to the VM.

Nutanix describes two main compression modes:

### Inline compression

Inline compression happens **during the write path**. Nutanix documentation says inline compression compresses sequential data streams or large I/O sizes — more than 64 KB — as they are written to the capacity store, also called the **Extent Store**. ([Nutanix Portal][1])

For interview purposes, the key phrase is:

> Inline compression reduces capacity usage as data is ingested, but AOS applies it selectively to avoid hurting latency-sensitive random I/O.

Do not explain it as “everything is compressed synchronously all the time.” A better answer is: Nutanix is designed to apply compression intelligently depending on data type and I/O pattern.

### Post-process compression

Post-process compression writes data first in an uncompressed state, then compresses it later as a background operation. Nutanix documentation states that post-process compression uses the **Curator** framework to compress data cluster-wide. ([Nutanix Portal][1])

This matters operationally because capacity savings may not appear immediately after data is written. In an escalation, a customer may say:

> “We enabled compression, but our capacity usage did not drop immediately.”

A good response is:

> “That can be expected depending on whether the data is being compressed inline or post-process. We need to check the container configuration, the workload pattern, Curator activity, and whether enough time has passed for background compression to complete.”

### Compression is not isolated

Compression interacts with:

* **Deduplication**
* **Erasure Coding**
* **Replication Factor**
* **Snapshots**
* **Clones**
* **Curator scans**
* **Workload compressibility**
* **CPU overhead**
* **Read/write latency**
* **Cluster capacity pressure**

Nutanix positions compression as one component of a broader data-efficiency model. AOS Storage includes thin provisioning, intelligent cloning, compression, deduplication, and erasure coding, all designed to optimize capacity while scaling with the cluster. ([Nutanix Portal][3])

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, compression matters because it sits at the intersection of **customer expectations, storage capacity, performance, and escalation management**.

Customers often enable compression expecting immediate, risk-free capacity savings. In reality, support needs to manage trade-offs:

* Does the workload actually compress well?
* Is the issue about capacity, performance, or both?
* Is the customer comparing logical used capacity with physical used capacity?
* Are savings delayed because compression is post-process?
* Is Curator healthy?
* Is there capacity pressure?
* Is there a licensing, version, configuration, or best-practice constraint?
* Is the customer mixing compression with dedupe or erasure coding?
* Is there a performance impact during heavy background activity?

For the manager role, you do not need to debug every low-level Stargate or Curator metric yourself, but you must be able to **lead the escalation intelligently**.

You should sound like someone who can coordinate SREs, support engineers, account teams, and customers:

> “Compression is a capacity-efficiency feature, but in support we need to validate the workload profile, the configured mode, the observed savings, and any performance side effects. I would separate the escalation into capacity expectation, configuration validation, health of the background jobs, and workload impact.”

That is the level expected from a **technical escalation manager**, not a pure IC storage engineer.

---

## 4. Key concepts

### AOS Storage

**AOS**, formerly associated with Acropolis Operating System, is the Nutanix software layer that provides distributed storage, availability, data services, and cluster management.

For compression, the important point is that this is not a feature of a single array controller. It is part of a **distributed storage fabric**.

### CVM

The **Controller VM** runs on each Nutanix node and handles storage I/O. In troubleshooting, CVMs are central because storage services, metadata, data path processes, and cluster health depend on them.

### OpLog

The **OpLog** is a persistent write buffer used to absorb and coalesce random writes before data is drained to the Extent Store. Nutanix’s architecture material explains that the AOS storage layer is divided into regions such as OpLog and Extent Store, and that OpLog quickly persists and replicates bursty random writes. ([nutanix.dev][4])

For compression, the key point is:

* Random or small writes are handled carefully to protect performance.
* Large or sequential writes are better candidates for immediate compression.
* You should avoid claiming compression is always applied uniformly across all writes.

### Extent Store

The **Extent Store** is the persistent bulk storage area in AOS. Nutanix documentation and architecture material describe it as the destination for data drained from OpLog or written directly when the workload is sequential or sustained. ([nutanix.dev][4])

Compression is especially relevant here because Nutanix documentation says inline compression compresses data as it is written to the Extent Store. ([Nutanix Portal][1])

### Curator

**Curator** is Nutanix’s distributed background framework for cluster-wide storage tasks. In the context of compression, post-process compression relies on Curator to identify and compress data after it has already been written. ([Nutanix Portal][1])

In an escalation, Curator health and backlog can affect when expected space savings materialize.

### Compression delay

Compression delay refers to the time between writing data and compressing it when using post-process compression. Nutanix Prism documentation for compression states that post-process compression compresses data after it is written, and the delay is configurable; it also notes a recommended delay of 60 minutes in that Prism guide. ([Nutanix Portal][5])

For interviews, phrase it carefully:

> “In post-process compression, savings may be delayed because data is first written uncompressed and compressed later by background jobs.”

### Inline vs post-process

| Mode                     | Practical meaning                         | Interview framing                                                              |
| ------------------------ | ----------------------------------------- | ------------------------------------------------------------------------------ |
| Inline compression       | Compresses suitable data during ingestion | Faster capacity reduction, but must be designed to avoid harming write latency |
| Post-process compression | Compresses data later in the background   | Better for avoiding immediate write-path overhead, but savings are delayed     |

### Compression vs deduplication

Compression reduces the size of individual data blocks or extents.

Deduplication eliminates duplicate copies of identical data.

They are related but not the same. Nutanix documentation groups compression, deduplication, and erasure coding under data reduction / capacity optimization. ([Nutanix Portal][6])

A useful interview sentence:

> “Compression reduces the size of data; deduplication reduces repeated data; erasure coding reduces replication overhead for cold or suitable data.”

### Compression vs erasure coding

Compression reduces the data footprint by encoding data more compactly.

**Erasure coding** saves capacity by storing data with parity rather than full replicas. Nutanix Prism documentation says erasure coding capacity savings are in addition to deduplication and compression savings. ([Nutanix Portal][7])

In a customer escalation, never mix these up.

---

## 5. How it appears in a real escalation

### Scenario 1 — “We enabled compression but do not see savings”

A customer enables compression on a storage container and expects immediate capacity reduction.

Possible causes:

* Compression is post-process and Curator has not completed.
* Existing data may need background scans.
* The workload is not highly compressible.
* The customer is looking at the wrong metric.
* Snapshots or clones are retaining old data.
* Savings are masked by new writes.
* Deduplication or erasure coding expectations are being confused with compression.
* There is insufficient cluster health for background tasks to run efficiently.

Manager response:

> “I would first align on the customer’s expectation: what saving they expected, when they enabled compression, and which metric they are using. Then I would have the team validate the container policy, compression mode, data type, Curator status, cluster capacity trend, and whether snapshots or ongoing ingestion are masking the effect.”

### Scenario 2 — “Performance got worse after enabling compression”

Possible causes:

* The customer enabled efficiency features during peak workload.
* CPU pressure increased on CVMs.
* Curator or background tasks are competing with foreground I/O.
* The workload is latency-sensitive.
* The issue is coincidental and actually caused by another event: node issue, disk latency, network issue, noisy neighbor, hypervisor problem, or guest-level change.

Good escalation framing:

> “I would avoid assuming compression is the root cause. I would compare before-and-after latency, IOPS, throughput, CVM CPU, Stargate behavior, Curator activity, and cluster events. If needed, I would coordinate a temporary mitigation or policy adjustment while engineering or senior SREs validate the root cause.”

### Scenario 3 — “Capacity is critically low”

Compression may be considered as a mitigation, but it is not magic.

Important support logic:

* Compression may not deliver immediate relief.
* Post-process compression needs time and resources.
* If the cluster is already critically full, background efficiency may be constrained.
* You may need safer short-term actions: add capacity, delete unneeded snapshots, move workloads, reduce ingest rate, or engage Nutanix Support for emergency guidance.

Strong manager-level answer:

> “I would not present compression as an emergency fix without validation. In a capacity-critical state, the priority is protecting availability. Compression can help, but I would also look at snapshot cleanup, workload growth, container usage, reclaimable space, expansion options, and support-led recovery actions.”

### Scenario 4 — “Customer is comparing Nutanix to VMware vSAN or a traditional SAN”

You may be asked to explain how Nutanix compression differs.

Useful positioning:

> “In Nutanix, compression is part of the distributed AOS storage fabric. It is not just a controller-side feature. It interacts with the CVM data path, Extent Store, Curator, and other data efficiency services. That means support must understand both the feature and the distributed system behavior.”

---

## 6. Triage questions I should ask

### Customer impact

1. What is the actual business impact: capacity risk, performance degradation, failed writes, VM latency, or expectation mismatch?
2. Is the issue affecting one VM, one container, one cluster, or multiple clusters?
3. Is this production, DR, VDI, database, file services, or general virtualization?

### Timeline

4. When was compression enabled?
5. Was it enabled inline or post-process?
6. Did the problem start immediately after the change, or later?
7. Were there other changes: AOS upgrade, node expansion, workload migration, snapshot policy change, backup job, hypervisor change?

### Configuration

8. Which storage container or policy is affected?
9. Is deduplication also enabled?
10. Is erasure coding enabled?
11. What is the replication factor?
12. Is this AHV, ESXi, or another supported hypervisor scenario?

### Capacity

13. What are logical used, physical used, free capacity, and data reduction ratio?
14. Are snapshots consuming significant capacity?
15. Is there rapid data growth?
16. Is the data type likely compressible: logs, text, databases, media, encrypted data, already-compressed backups?

### Performance

17. What changed in latency, IOPS, throughput, and queue depth?
18. Is the workload random write-heavy, sequential, read-heavy, or mixed?
19. Are CVMs showing CPU, memory, disk, or network pressure?
20. Are there cluster alerts or unhealthy services?

### Background services

21. Is Curator running normally?
22. Is there a Curator backlog?
23. Are background tasks coinciding with peak production traffic?
24. Are there alerts related to storage services, metadata, disks, or nodes?

### Escalation management

25. What does the customer need: explanation, mitigation, RCA, workaround, or engineering escalation?
26. Is there an SLA breach or executive visibility?
27. Do we need a bridge call, TAM involvement, SRE escalation, or engineering review?

---

## 7. Likely interview questions

1. What is compression in Nutanix AOS?
2. What is the difference between inline and post-process compression?
3. Why might a customer not see immediate savings after enabling compression?
4. How would you troubleshoot a performance issue after compression was enabled?
5. How does compression differ from deduplication?
6. How does compression interact with erasure coding?
7. What role does Curator play in compression?
8. What questions would you ask during a compression-related escalation?
9. How would you explain compression to a non-technical customer executive?
10. How would you manage an escalation where the customer blames compression for latency?
11. What metrics would you want your SREs to check?
12. How would you decide whether to involve engineering?
13. How would you communicate risk if the cluster is nearly full?
14. What kind of workloads benefit most from compression?
15. What workloads may show limited compression savings?
16. How would your previous cloud/SaaS support experience help you manage this type of escalation?

---

## 8. Model answers in English

### Q1. What is compression in Nutanix AOS?

**Model answer:**

“Compression in Nutanix AOS is a storage efficiency capability that reduces the physical capacity consumed by VM data. It is part of the AOS data reduction stack, alongside deduplication, thin provisioning, cloning, and erasure coding. Nutanix supports both inline and post-process compression. From a support perspective, the important point is to understand the customer’s workload, the configured compression mode, the expected savings, and any impact on performance or background activity.”

---

### Q2. What is the difference between inline and post-process compression?

**Model answer:**

“Inline compression happens during the write path, mainly for suitable data patterns such as larger or sequential I/O. Post-process compression writes data first and compresses it later in the background, using Nutanix’s Curator framework. Inline compression can provide more immediate savings, while post-process compression may reduce write-path impact but introduces a delay before savings are visible. In an escalation, I would always confirm which mode is configured before setting expectations with the customer.”

---

### Q3. A customer enabled compression but does not see savings. What do you do?

**Model answer:**

“I would first clarify whether this is a true technical issue or an expectation issue. I would check when compression was enabled, whether it is inline or post-process, what data type is being written, and which metric the customer is using to measure savings. I would also ask the team to validate Curator activity, cluster health, snapshots, data growth rate, and whether the workload is compressible. If the data is encrypted, already compressed, or media-heavy, the savings may be limited. I would communicate that compression savings are workload-dependent and, for post-process compression, may take time to appear.”

---

### Q4. A customer reports latency after enabling compression. How would you manage the escalation?

**Model answer:**

“I would avoid assuming causality. I would establish a timeline, compare pre-change and post-change metrics, and separate foreground I/O latency from background activity. I would have the team review VM latency, IOPS, throughput, CVM CPU, storage service health, Curator tasks, disk latency, and cluster alerts. From a management perspective, I would keep the customer informed, define mitigation options, and involve senior SREs or engineering if we see evidence that the compression workflow or background services are contributing to the issue.”

---

### Q5. How is compression different from deduplication?

**Model answer:**

“Compression reduces the size of data by encoding it more efficiently. Deduplication removes duplicate copies of identical data. For example, compression helps when individual blocks are compressible, while deduplication helps when many VMs or files contain repeated data. In Nutanix, both are data efficiency features, but they solve different problems and have different workload suitability.”

---

### Q6. How would you explain this to a customer executive?

**Model answer:**

“I would say: compression is designed to help the platform use storage capacity more efficiently, but the actual savings depend on the type of data and the workload. Some savings may appear immediately, while others may happen after background optimization jobs complete. Our priority is to validate that the cluster is healthy, that the feature is configured correctly, and that capacity optimization is not affecting application performance.”

---

### Q7. What would you expect your SREs to check?

**Model answer:**

“I would expect the SREs to check the storage container or policy configuration, compression mode, data reduction statistics, Curator status, CVM resource usage, Stargate/storage service health, cluster alerts, capacity trends, VM latency, and workload profile. If performance is involved, I would also want correlation with recent changes, backups, snapshots, migrations, or upgrades.”

---

### Q8. When would you escalate to engineering?

**Model answer:**

“I would escalate to engineering when we have evidence of unexpected platform behavior, such as compression jobs failing, Curator not progressing despite healthy prerequisites, unexplained performance degradation correlated with compression, inconsistent reporting, or a suspected defect after an AOS upgrade. Before escalating, I would make sure support has collected logs, timelines, configuration details, metrics, and clear reproduction or impact data.”

---

### Q9. How does your previous experience apply?

**Model answer:**

“In my current role, I manage a 24/7 enterprise support team where we deal with incidents, SLAs, MTTR, monitoring, escalations, and customer communication. Compression-related issues are similar to cloud and SaaS operations problems: customers see symptoms such as capacity pressure or latency, but the root cause may involve background jobs, workload behavior, configuration, or expectation mismatch. My value is in structuring the escalation, keeping the customer informed, driving technical validation, and making sure the team separates symptoms from root cause.”

---

## 9. Connection with my experience

Your background maps well to this topic if you frame it correctly.

You should not try to position yourself as the deepest AOS storage engineer in the room. Instead, position yourself as a manager who can **lead storage-related escalations with enough technical fluency to challenge assumptions and coordinate experts**.

Your existing experience connects directly:

| Your experience         | How to connect it to Nutanix compression                                                         |
| ----------------------- | ------------------------------------------------------------------------------------------------ |
| 24/7 enterprise support | Compression issues can become production-impacting capacity or latency escalations               |
| Incident management     | You know how to structure timeline, impact, mitigation, owner, and RCA                           |
| SLA / MTTR              | Compression-related delays or performance issues must be managed against customer urgency        |
| Monitoring              | You can correlate capacity, latency, throughput, CPU, and background activity                    |
| Cloud operations        | Similar to cloud storage optimization: capacity savings, background jobs, noisy neighbor effects |
| Grafana / Kibana        | Useful mindset for metric correlation and time-series analysis                                   |
| Jira / Confluence       | Useful for escalation tracking, action plans, and knowledge base documentation                   |
| Salesforce              | Relevant for customer case management and executive communication                                |
| Kubernetes / Linux      | Helps with distributed systems thinking, resource contention, logs, and service health           |

Strong positioning statement:

> “My value is not that I know every internal AOS command today. My value is that I understand how storage efficiency features affect enterprise operations, how to lead escalations under pressure, how to ask the right technical questions, and how to coordinate support, SRE, engineering, and customer stakeholders until the issue is resolved.”

---

## 10. Minimum I need to memorize

Memorize these points:

1. **Compression reduces physical storage consumption.**
2. Nutanix supports **inline** and **post-process** compression.
3. **Inline compression** happens during the write path for suitable data, especially larger or sequential writes.
4. **Post-process compression** happens later in the background using **Curator**.
5. Compression savings are **workload-dependent**.
6. Already-compressed, encrypted, or media-heavy data may not compress well.
7. Compression is different from **deduplication** and **erasure coding**.
8. Delayed savings do not automatically mean the feature is broken.
9. Performance complaints require correlation, not assumption.
10. In escalation, check: configuration, workload, capacity metrics, Curator, CVM health, cluster alerts, and timeline.
11. For a manager role, focus on **impact, triage, communication, escalation ownership, and risk management**.
12. Never promise compression as an emergency fix for a nearly full cluster without validation.

A concise interview version:

> “Nutanix AOS compression is a data efficiency feature that reduces physical storage usage. It can be inline, where suitable data is compressed during ingestion, or post-process, where data is compressed later by Curator. In support, the key is to validate the workload, mode, expected savings, cluster health, and performance impact. Compression is useful, but savings are workload-dependent and may not appear immediately.”

---

## 11. Advanced / optional level

You can leave these as advanced unless the interview becomes very technical:

1. Internal Stargate behavior.
2. Detailed OpLog compression internals.
3. Exact internal thresholds beyond the main “large/sequential I/O” concept.
4. Low-level Curator job scheduling.
5. Specific `ncli`, `acli`, or internal support commands.
6. Compression metadata layout.
7. AOS version-specific implementation differences.
8. Detailed interactions with BlockStore or SPDK.
9. Performance tuning at GFlag level.
10. Deep forensic analysis of data reduction reporting.

However, you should recognize the terms if they appear.

A safe response when pushed deeper:

> “I understand the architectural role of those components, but for exact internal counters or version-specific behavior I would rely on Nutanix documentation and senior SRE guidance. As a support manager, I would make sure the right diagnostics are collected and that the escalation is routed to the correct technical owner.”

That is a good answer for a manager role.

---

## 12. Final checklist

Before your interview, you should be able to explain:

* [ ] What AOS compression does.
* [ ] Difference between inline and post-process compression.
* [ ] Why savings may be delayed.
* [ ] Why some workloads compress poorly.
* [ ] How compression differs from deduplication.
* [ ] How compression differs from erasure coding.
* [ ] What Curator does in post-process compression.
* [ ] Why compression can appear in capacity escalations.
* [ ] Why compression can appear in performance escalations.
* [ ] What triage questions to ask.
* [ ] How to communicate compression behavior to a customer.
* [ ] How to involve SREs, engineering, TAM, or account teams.
* [ ] How to connect this topic to incident management, SLA, MTTR, and customer communication.
* [ ] How to avoid overclaiming technical depth beyond the manager role.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------------------- |
| AOS                      | Nutanix software layer providing distributed storage, data services, and cluster management.    |
| AHV                      | Acropolis Hypervisor; Nutanix’s native hypervisor.                                              |
| Background job           | Non-foreground task, such as post-process compression or data optimization.                     |
| Capacity optimization    | Reducing physical storage usage through efficiency techniques.                                  |
| Compression              | Reduces physical data size by storing data more compactly.                                      |
| Compression delay        | Time between data write and post-process compression.                                           |
| Container                | Logical storage unit where storage policies such as compression may apply.                      |
| Controller VM            | VM on each Nutanix node that provides storage services.                                         |
| Curator                  | Nutanix distributed background framework used for storage management tasks.                     |
| CVM                      | Controller Virtual Machine.                                                                     |
| Data reduction           | General category including compression, deduplication, and erasure coding.                      |
| Deduplication            | Removes duplicate copies of identical data.                                                     |
| EC-X                     | Nutanix erasure coding technology for capacity efficiency.                                      |
| Encrypted data           | Data transformed for security; often compresses poorly.                                         |
| Erasure coding           | Capacity-saving method using parity instead of full replicated copies.                          |
| ESXi                     | VMware hypervisor often used with Nutanix clusters.                                             |
| Extent Store             | Persistent bulk storage area in AOS.                                                            |
| Foreground I/O           | Active application read/write traffic.                                                          |
| HCI                      | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform. |
| ILM                      | Intelligent Lifecycle Management; moves data across tiers based on access patterns.             |
| Inline compression       | Compression performed during data ingestion/write path.                                         |
| IOPS                     | Input/Output Operations Per Second.                                                             |
| Latency                  | Time taken to complete an I/O operation.                                                        |
| Logical usage            | Capacity as seen before physical efficiency savings.                                            |
| MTTR                     | Mean Time To Resolution or Recovery.                                                            |
| OpLog                    | Persistent write buffer used for bursty random writes.                                          |
| Physical usage           | Actual storage consumed after efficiency techniques.                                            |
| Post-process compression | Compression performed after data is written, usually by background jobs.                        |
| Prism                    | Nutanix management interface for cluster and infrastructure operations.                         |
| RCA                      | Root Cause Analysis.                                                                            |
| RF                       | Replication Factor; number of copies maintained for availability.                               |
| SLA                      | Service Level Agreement.                                                                        |
| SRE                      | Site Reliability Engineer.                                                                      |
| Stargate                 | Nutanix storage data path service running in the CVM.                                           |
| Storage container        | Logical storage boundary where features such as compression may be configured.                  |
| Storage efficiency       | Techniques that reduce storage consumption or improve effective capacity.                       |
| Throughput               | Amount of data transferred per second.                                                          |
| VM                       | Virtual Machine.                                                                                |
| Workload profile         | Pattern of I/O behavior: random, sequential, read-heavy, write-heavy, mixed.                    |

[1]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2032-Data-Efficiency%3Anutanix-data-compression.html&utm_source=chatgpt.com "Nutanix Data Compression"
[2]: https://portal.nutanix.com/page/documents/details?targetId=vSphere-Admin6-AOS-v7_0%3AvSphere-Admin6-AOS-v7_0&utm_source=chatgpt.com "AOS 7.0 - vSphere Administration Guide for AOS - portal.nutanix.com"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2032-Data-Efficiency%3ATN-2032-Data-Efficiency&utm_source=chatgpt.com "Data Efficiency - Nutanix"
[4]: https://www.nutanix.dev/2022/08/03/nutanix-benefit-1-dynamically-distributed-storage/?utm_source=chatgpt.com "Nutanix Benefit 1: Dynamically Distributed Storage"
[5]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_7_3%3Asto-compression-c.html&utm_source=chatgpt.com "Prism pc.7.3 - Compression - portal.nutanix.com"
[6]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2032-Data-Efficiency%3Aaos-storage-data-reduction.html&utm_source=chatgpt.com "AOS Storage Data Reduction - portal.nutanix.com"
[7]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide%3Awc-erasure-coding-overview-wc-c.html&utm_source=chatgpt.com "Prism pc.7.5 - Erasure Coding - Nutanix"
