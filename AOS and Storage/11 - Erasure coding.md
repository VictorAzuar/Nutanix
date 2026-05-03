# [AOS / Storage] — Erasure Coding

## 1. Short definition

**Erasure coding** in Nutanix AOS is a storage-efficiency and data-protection mechanism that reduces capacity overhead by replacing extra replicated copies of cold data with **parity information**, while still allowing data to be rebuilt after a disk, node, or fault-domain failure. Nutanix documentation describes it as increasing usable cluster capacity by using parity rather than simple replication, with savings that are additional to compression and deduplication. ([Nutanix Portal][1])

In Nutanix, you will often see it referred to as **EC** or **EC-X**.

---

## 2. Clear explanation

In a normal replicated storage model, data protection is achieved by keeping multiple copies of the same data.

For example:

* With **RF2**, Nutanix keeps two copies of data.
* With **RF3**, Nutanix keeps three copies of data.
* This is simple and fast, but it consumes more raw capacity.

Erasure coding works differently. Instead of keeping a full second or third copy forever, AOS can take eligible data blocks, group them into a **stripe**, calculate **parity**, and then remove some replicated copies. If a component fails, the system can reconstruct the missing data from the remaining data blocks and parity.

A simple mental model:

```text
Replication:
Data A + Copy A = protected, but expensive in capacity.

Erasure Coding:
Data A + Data B + Data C + Data D + Parity = protected, more capacity-efficient.
```

In Nutanix, standard erasure coding is typically applied to **write-cold data**, meaning data that has not changed for a defined period. Official Nutanix guidance says the default write-cold window is **seven days**, and that savings may be reduced for workloads with many overwrites outside the erasure-coding window. ([Nutanix Portal][2])

The important support-management point is this:

**Erasure coding is not primarily a performance feature. It is a capacity-efficiency feature with resiliency implications.**

It helps customers get more usable storage from the same cluster, but it can introduce additional rebuild and read overhead during failure or recovery scenarios. Nutanix notes that read performance can be affected during rebuild, depending on strip size and read load. ([Nutanix Portal][2])

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, erasure coding matters because it sits at the intersection of:

* Customer capacity pressure.
* Storage resiliency.
* Performance expectations.
* Failure-domain design.
* Enterprise escalation handling.
* Misconfiguration or misunderstanding of Nutanix storage efficiency.

A customer may open a high-severity case saying:

> “We enabled erasure coding, but we are not seeing the expected savings.”

Or:

> “After a disk/node failure, latency increased and VMs are slower.”

Or:

> “We are running out of capacity even though EC is enabled.”

As a support manager, you do not need to personally debug every low-level extent-group detail, but you must understand enough to:

* Ask the right triage questions.
* Avoid misleading the customer.
* Understand what the SRE/storage engineer is investigating.
* Explain trade-offs clearly to enterprise stakeholders.
* Escalate correctly to engineering when behavior looks abnormal.
* Keep the discussion grounded in business impact: risk, performance, capacity, SLA, recovery timeline.

The key leadership angle:

**Erasure coding is a customer-expectation management topic.**
Customers may think “EC enabled” means “instant savings with no trade-offs.” In reality, savings depend on workload profile, coldness of data, Curator scans, cluster size, strip size, fault-domain awareness, and current health.

---

## 4. Key concepts

### Replication Factor: RF2 and RF3

**Replication Factor** defines how many copies of data Nutanix maintains.

* **RF2** protects against one failure scenario.
* **RF3** provides higher resiliency but consumes more capacity.
* Erasure coding works with the underlying protection model but reduces the capacity overhead of replicated data once data becomes eligible.

Nutanix states that erasure coding is supported with a minimum of **4 nodes for RF2** and **6 nodes for RF3**. ([Nutanix Portal][1])

### Parity

Parity is calculated information that allows the system to reconstruct missing data if part of the stripe is lost.

In an interview, avoid going too deep into mathematics. A good explanation is:

> “Parity is additional information generated from a set of data blocks. If one block is lost, the system can use the remaining data blocks plus parity to rebuild the missing block.”

### Stripe / Strip

Nutanix documentation describes EC strips using a data/parity model, where **N** represents information/data egroups and **K** represents parity blocks. ([Nutanix Portal][3])

A practical interview explanation:

> “A strip is the group of data and parity units used by erasure coding. Larger strips can improve space efficiency but may increase rebuild cost and affect reads during failure scenarios.”

Nutanix recommends not changing strip size casually because larger strip sizes may increase space savings but also increase rebuild cost. ([Nutanix Portal][2])

### Write-cold data

Write-cold data is data that has not been modified recently. Standard EC-X is well suited for data that is not frequently overwritten.

Examples:

* Backup data.
* Archives.
* File shares with mostly static content.
* Logs after ingestion.
* Some object-storage or compliance workloads.

Poorer candidates:

* High-change databases.
* Heavy write workloads.
* Latency-sensitive transactional systems.
* Workloads with continuous overwrites.

### Curator

**Curator** is the Nutanix background process that performs cluster-wide maintenance and optimization tasks. For erasure coding, it is relevant because space savings may not appear immediately. Nutanix documentation says EC is asynchronous and that at least **two full Curator scans** are required to calculate data savings. ([Nutanix Portal][2])

Support-friendly phrasing:

> “If EC was recently enabled, I would not expect immediate savings. I would check whether the data has become write-cold and whether Curator has completed the required scans.”

### Inline erasure coding

Nutanix also supports **inline erasure coding** in some contexts. Inline EC creates EC strips without waiting for data to become write-cold. Nutanix documentation notes that inline EC is available from AOS 5.18 or later, requires EC to be enabled on the container first, and is generally handled through nCLI / support-guided configuration. ([Nutanix Portal][4])

For your interview level, you should know it exists, but avoid presenting yourself as a deep implementation owner unless asked.

Best wording:

> “Standard EC-X is typically post-process and write-cold oriented. Inline EC exists for specific use cases and versions, but I would treat that as something to validate against the customer’s AOS version, workload profile, and Nutanix support guidance.”

### Fault domains: node, block, rack

Erasure coding has to respect placement rules. Parity and data should be distributed so that a single failure domain does not take out too much of the stripe.

Official Nutanix guidance says that with erasure coding enabled, a minimum of **four blocks for RF2** or **six blocks for RF3** is required to maintain block awareness. ([Nutanix Portal][2])

Manager-level takeaway:

> “It is not enough to ask whether EC is enabled. We also need to understand whether the cluster has enough nodes, blocks, or racks to maintain the desired fault-domain awareness.”

---

## 5. How it appears in a real escalation

### Escalation scenario 1 — “We enabled EC but there are no savings”

Likely causes:

* EC was enabled recently.
* Data is still write-hot.
* Curator has not completed enough scans.
* Workload has frequent overwrites.
* Data reduction reporting has not caught up.
* Container settings are not what the customer expects.
* Cluster size or fault-domain constraints reduce effectiveness.

Support-manager response:

> “We should validate when EC was enabled, whether the target data is write-cold, whether Curator scans completed, and whether the workload profile is suitable. I would also set expectations that EC savings are asynchronous and workload-dependent, not immediate.”

### Escalation scenario 2 — “Latency increased after a disk/node failure”

Possible link to erasure coding:

* During failure or rebuild, EC-protected data may require reconstruction reads.
* Larger strips can increase rebuild cost.
* The cluster may already be under high read load.
* There may be concurrent Curator, ILM, rebuild, or capacity pressure.
* The issue may not be EC alone; it could involve storage latency, CVM load, disk health, network, or hypervisor contention.

Support-manager response:

> “I would treat this as a performance-under-degraded-state escalation. I would not assume EC is the only cause. I would check cluster health, rebuild activity, disk/node status, Stargate/CVM metrics, latency breakdown, current workload pressure, and whether the affected data is EC-protected.”

### Escalation scenario 3 — “Customer is near full capacity and wants to enable EC immediately”

Important caution:

* EC may help, but not instantly.
* Running close to full capacity is already risky.
* EC needs time and background processing.
* Write-hot data may not benefit.
* The customer may need immediate capacity relief through cleanup, expansion, migration, or workload reduction.

Support-manager response:

> “I would avoid promising immediate relief. I would explain that EC is a capacity-efficiency feature that may produce savings over time, especially on cold data. In parallel, I would drive immediate risk reduction: capacity cleanup, snapshot review, old VM/template removal, expansion planning, and business prioritization of critical workloads.”

### Escalation scenario 4 — “Customer enabled EC on the wrong workload”

Example: high-write database or latency-sensitive application.

Potential symptoms:

* Limited savings.
* Increased overhead during failure/rebuild.
* Customer dissatisfaction because expectations were wrong.

Support-manager response:

> “I would bring the conversation back to workload fit. EC is stronger for cold or less frequently modified data. For high-write or latency-sensitive workloads, we need to validate whether EC is appropriate and whether the container design matches the application profile.”

---

## 6. Triage questions I should ask

### Business-impact questions

1. What is the customer impact: capacity risk, performance degradation, failed VM operations, or data availability?
2. Is production affected?
3. Which applications or VMs are impacted?
4. Is this a Sev1/Sev2 due to outage, degraded performance, or risk of outage?
5. What SLA or business deadline is at risk?

### Cluster and configuration questions

1. What AOS version is the cluster running?
2. Is the affected storage container configured with RF2 or RF3?
3. How many nodes, blocks, and racks are in the cluster?
4. Is block or rack awareness required?
5. When was erasure coding enabled?
6. Is this standard post-process EC or inline EC?
7. Was the EC configuration recently changed?
8. Was strip size changed from default?

### Workload questions

1. What type of workload is on the container?
2. Is the data write-hot or write-cold?
3. Are there frequent overwrites?
4. Is the workload read-heavy, write-heavy, or mixed?
5. Is it latency-sensitive?
6. Are backups, archives, file shares, databases, or VDI involved?

### Symptom questions

1. Are expected savings visible in Prism?
2. Did savings appear and then disappear?
3. Is latency high only during rebuild or always?
4. Are there current disk, node, CVM, or network alerts?
5. Is Curator running?
6. Are rebuild, rebalancing, ILM, or garbage collection tasks active?
7. Is the cluster close to full capacity?
8. Are there snapshots, clones, or old backups consuming capacity?

### Escalation-control questions

1. Has Nutanix support collected logs / NCC health checks?
2. Is engineering already engaged?
3. Has the customer made recent changes?
4. What has already been communicated to the customer?
5. What is the next update commitment?
6. Who owns customer communication: TAM, Support, SRE, Account team, or Engineering?

---

## 7. Likely interview questions

### Conceptual

1. What is erasure coding?
2. How is erasure coding different from replication?
3. Why would Nutanix use erasure coding?
4. What is EC-X?
5. What is parity?
6. What type of workloads benefit from erasure coding?
7. What workloads are poor candidates for erasure coding?

### Nutanix-specific

8. How does erasure coding relate to RF2 and RF3?
9. What is the minimum cluster size for EC with RF2 and RF3?
10. Why might EC savings not appear immediately?
11. What role does Curator play?
12. What is write-cold data?
13. What is inline erasure coding?
14. What does strip size affect?
15. How can EC affect performance during failures?

### Support / escalation

16. A customer enabled EC but sees no savings. How do you handle it?
17. A customer sees high latency during a rebuild. What do you check?
18. A customer near full capacity wants EC enabled as an emergency fix. What do you say?
19. How would you explain EC to a non-technical executive?
20. How would you coordinate a Sev2 escalation involving EC, capacity pressure, and performance degradation?

### Managerial

21. How do you balance technical accuracy with customer reassurance?
22. How would you coach a support engineer handling this case?
23. When would you escalate to engineering?
24. How would you manage communication cadence?
25. How would you prevent recurrence after the incident?

---

## 8. Model answers in English

### Q1. What is erasure coding?

**Model answer:**

> Erasure coding is a storage protection and efficiency technique that uses parity instead of keeping full additional replicas of all data. In Nutanix AOS, it increases usable capacity by transforming eligible data into erasure-coded stripes. If part of the data is lost because of a disk or node failure, the missing data can be rebuilt from the remaining data and parity. I would describe it as a capacity-efficiency mechanism with resiliency support, not as a pure performance optimization.

---

### Q2. How is erasure coding different from replication?

**Model answer:**

> Replication protects data by keeping full copies. RF2 keeps two copies, RF3 keeps three copies. That is simple and fast, but it consumes more raw capacity. Erasure coding protects data by keeping data fragments plus parity, which reduces storage overhead. The trade-off is that reconstruction during failure can be more computationally and I/O intensive than simply reading another replica.

---

### Q3. Why does erasure coding matter in Nutanix?

**Model answer:**

> It matters because Nutanix is a distributed storage platform where customers care about usable capacity, resiliency, and operational simplicity. Erasure coding allows customers to improve capacity efficiency while maintaining protection. From a support perspective, it is important because customer expectations must be managed carefully: savings are workload-dependent, usually asynchronous, and can be affected by write patterns, Curator scans, cluster size, and failure-domain configuration.

---

### Q4. A customer enabled EC yesterday but sees no savings. What do you do?

**Model answer:**

> First, I would validate the timeline and confirm when EC was enabled. Then I would check the workload profile: if the data is still being actively overwritten, it may not be eligible. I would also verify the write-cold window and whether the required Curator scans have completed. I would explain to the customer that EC savings are not immediate; they are asynchronous and depend on data coldness and background optimization. In parallel, I would check for capacity pressure and propose immediate risk-reduction actions if the cluster is close to full.

---

### Q5. A customer reports latency during a rebuild on an EC-enabled container. How do you approach it?

**Model answer:**

> I would treat it as a degraded-state performance escalation. Erasure coding can affect read performance during rebuild because data may need to be reconstructed from parity, but I would avoid assuming it is the only cause. I would check cluster health, disk and node status, rebuild progress, CVM and Stargate metrics, read/write latency, current workload load, network health, and any concurrent background jobs. As a manager, I would ensure the technical team isolates the bottleneck while I manage customer communication, impact assessment, and update cadence.

---

### Q6. Should all workloads use erasure coding?

**Model answer:**

> No. Erasure coding is best suited for data that becomes cold or is not frequently overwritten, such as backups, archives, file shares, logs after ingestion, or other capacity-oriented workloads. For high-write, latency-sensitive, or transactional workloads, the savings may be limited and the operational trade-offs may not be worth it. The correct decision depends on workload profile, cluster design, performance requirements, and resiliency objectives.

---

### Q7. How would you explain erasure coding to an executive customer?

**Model answer:**

> I would say: “Erasure coding is a way to store protected data more efficiently. Instead of keeping complete extra copies of every block forever, the system calculates parity that can be used to rebuild data if something fails. This improves usable capacity, but the benefit appears over time and depends on the type of data. During failure or rebuild situations, there can be additional overhead, so we need to validate whether it is appropriate for each workload.”

---

### Q8. What would you memorize about Nutanix EC for an interview?

**Model answer:**

> I would remember that Nutanix EC-X is a storage-efficiency feature based on parity, mainly useful for write-cold data. It works with RF2 or RF3, but has minimum cluster-size requirements: four nodes for RF2 and six nodes for RF3. Savings are asynchronous and depend on Curator scans and workload behavior. It is useful for capacity optimization, but during rebuild or degraded states it may have performance trade-offs, especially for read-heavy workloads.

---

### Q9. How would you handle a disagreement between the customer, support engineer, and engineering about whether EC caused the incident?

**Model answer:**

> I would separate hypothesis from evidence. EC could be a contributing factor, especially during rebuild, but we need data: timeline, configuration changes, workload behavior, latency metrics, rebuild status, health checks, and logs. I would ask the support engineer to build a clear technical timeline and ask engineering for a specific confirmation or exclusion of EC as a root or contributing cause. With the customer, I would communicate transparently: what we know, what we are validating, what actions are safe now, and when we will provide the next update.

---

## 9. Connection with my experience

Your Harmonic background maps well to this topic if you position yourself correctly.

You do **not** need to sound like a Nutanix storage kernel engineer. You need to sound like a support leader who understands enough storage architecture to drive escalations effectively.

Relevant experience to emphasize:

### Incident management

You already understand degraded-state behavior: systems often behave differently during failure, failover, rebuild, or recovery. That maps directly to EC scenarios where performance may be acceptable in normal conditions but degraded during rebuild.

Interview bridge:

> “In my current role, I often distinguish between steady-state performance and degraded-state performance. I would apply the same logic to EC-related escalations: understand what changed, what background recovery is running, what customer workload is affected, and what risk remains.”

### SLA / MTTR

EC escalations may not always be full outages, but they can become urgent due to capacity pressure or performance risk.

Interview bridge:

> “I would manage the case around customer impact and risk: is this a capacity-risk case, a performance degradation, or a data-availability concern? That determines severity, communication cadence, and escalation path.”

### Monitoring and observability

Your experience with Grafana, Kibana, cloud operations, and KPIs helps because Nutanix escalations require evidence: latency, throughput, disk health, node status, background tasks, and timelines.

Interview bridge:

> “I would want the team to correlate Prism metrics, NCC results, logs, and customer-side application symptoms. In my current environment, I use the same principle: do not debug from one metric; build a timeline across systems.”

### Cloud / SaaS operations

Cloud platforms also use capacity optimization, replication, object storage durability, and background repair concepts. You can connect EC to distributed-systems trade-offs.

Interview bridge:

> “The concept is similar to distributed cloud storage trade-offs: replication is simple and fast but capacity-expensive; parity-based schemes are more efficient but can introduce reconstruction overhead. The support challenge is to know when the trade-off is appropriate.”

### Team leadership

You can position yourself as the person who ensures technical accuracy, customer trust, and internal alignment.

Interview bridge:

> “My role would be to make sure the engineer is investigating the right technical path, the customer understands realistic expectations, and internal stakeholders have a clear timeline, owner, and next action.”

---

## 10. Minimum I need to memorize

Memorize these points:

1. **Erasure coding reduces capacity overhead using parity instead of full replicas.**
2. **In Nutanix it is commonly referred to as EC or EC-X.**
3. **It is a capacity-efficiency feature, not primarily a performance feature.**
4. **It is strongest for write-cold or low-change data.**
5. **Default write-cold window is commonly seven days.** ([Nutanix Portal][2])
6. **Savings are asynchronous and require Curator activity; at least two full Curator scans may be needed for savings calculation.** ([Nutanix Portal][2])
7. **Minimum supported cluster size: 4 nodes for RF2, 6 nodes for RF3.** ([Nutanix Portal][1])
8. **Read performance can be affected during rebuild.** ([Nutanix Portal][2])
9. **Larger strip sizes may improve savings but increase rebuild cost; do not casually change them.** ([Nutanix Portal][2])
10. **For support escalations, ask about workload type, AOS version, RF, cluster size, fault domains, when EC was enabled, Curator scans, current health, and business impact.**

A strong 30-second version:

> “Erasure coding in Nutanix AOS is a storage-efficiency mechanism that uses parity to reduce the capacity overhead of replicated data. It is best suited for write-cold data and is processed asynchronously, so savings are not immediate and depend on Curator scans and workload behavior. It supports resiliency, but during rebuild or failure scenarios it can introduce additional read or reconstruction overhead. As a support manager, I would focus on workload fit, cluster health, capacity risk, customer expectations, and escalation coordination.”

---

## 11. Advanced / optional level

You can leave these as advanced unless the interview becomes very technical:

* Exact EC strip layout by AOS version.
* Low-level extent group placement.
* Stargate internals.
* Zeus metadata parameters.
* Manual nCLI configuration details.
* Inline EC operational caveats.
* Detailed Curator job internals.
* Mathematical Reed-Solomon / XOR implementation.
* Exact parity placement across tiers.
* Advanced block/rack awareness behavior.
* Interactions with every Nutanix feature: Files, Objects, Volumes, Metro, snapshots, replication, DR.

However, you should be able to recognize these terms and say:

> “I understand the concept and support implications, but I would validate exact implementation details against the customer’s AOS version and current Nutanix documentation.”

That is a good manager-level answer.

---

## 12. Final checklist

Before an interview, make sure you can answer:

* [ ] What is erasure coding?
* [ ] How is it different from replication?
* [ ] What is RF2 / RF3?
* [ ] Why does EC improve usable capacity?
* [ ] Why are savings not immediate?
* [ ] What is write-cold data?
* [ ] What is Curator’s role?
* [ ] What workloads are good candidates?
* [ ] What workloads are poor candidates?
* [ ] What happens during rebuild?
* [ ] Why can performance degrade during failure?
* [ ] What cluster-size requirements should I remember?
* [ ] What triage questions would I ask in a customer escalation?
* [ ] How would I explain EC to a non-technical customer?
* [ ] How would I manage a customer who expects EC to fix capacity immediately?
* [ ] How would I coordinate Support, SRE, Engineering, TAM, and the customer?

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                                 |
| ------------------------ | ------------------------------------------------------------------------------------------------------- |
| AOS                      | Acropolis Operating System; Nutanix distributed storage and virtualization platform software layer.     |
| AHV                      | Acropolis Hypervisor; Nutanix native hypervisor.                                                        |
| Archive workload         | Workload with data that is usually written once and rarely modified.                                    |
| Block awareness          | Data-placement strategy that protects against failure of a physical block/chassis.                      |
| Capacity efficiency      | Ability to store more effective data using less raw storage.                                            |
| Capacity overhead        | Extra storage consumed for protection, copies, parity, metadata, or snapshots.                          |
| Cold data                | Data that has not been recently modified.                                                               |
| Compression              | Data-reduction method that stores data in a smaller encoded form.                                       |
| Container                | Nutanix storage container where VM disks or other data reside.                                          |
| Curator                  | Nutanix background process for maintenance, scans, optimization, and data reduction tasks.              |
| CVM                      | Controller VM; Nutanix VM running storage/controller services on each node.                             |
| Data locality            | Nutanix design principle where data is served close to the VM when possible.                            |
| Data reduction           | Technologies that reduce consumed storage, such as compression, deduplication, and erasure coding.      |
| Deduplication            | Data-reduction method that avoids storing duplicate data blocks.                                        |
| Degraded state           | Operating condition after a failure where redundancy or performance may be reduced.                     |
| Disk failure             | Loss or fault of a physical storage device.                                                             |
| EC                       | Erasure Coding; parity-based storage efficiency and protection method.                                  |
| EC-X                     | Nutanix term commonly used for erasure coding functionality.                                            |
| Egroup                   | Extent group; Nutanix storage unit used internally for data placement.                                  |
| Erasure coding           | Method that uses data fragments and parity to rebuild lost data while reducing capacity overhead.       |
| Failure domain           | Physical or logical boundary whose failure must be tolerated, such as disk, node, block, or rack.       |
| Hot data                 | Data that is actively being written or modified.                                                        |
| HCI                      | Hyperconverged Infrastructure; architecture combining compute, storage, and virtualization.             |
| ILM                      | Information Lifecycle Management; movement/management of data across tiers or states.                   |
| Inline EC                | Erasure coding performed inline without waiting for data to become write-cold.                          |
| KPI                      | Key Performance Indicator; metric used to track operational performance.                                |
| Latency                  | Time taken to complete an I/O operation or request.                                                     |
| MTTR                     | Mean Time To Repair/Recover; average time to restore service.                                           |
| nCLI                     | Nutanix command-line interface.                                                                         |
| NCC                      | Nutanix Cluster Check; health-check and diagnostic framework.                                           |
| Node                     | Physical Nutanix server participating in the cluster.                                                   |
| Parity                   | Calculated information used to reconstruct missing data.                                                |
| Prism                    | Nutanix management and monitoring interface.                                                            |
| Prism Central            | Centralized Nutanix management plane for multiple clusters and advanced services.                       |
| Prism Element            | Cluster-level Nutanix management interface.                                                             |
| Rack awareness           | Data-placement strategy protecting against rack-level failure.                                          |
| Rebuild                  | Process of reconstructing lost or degraded data after failure.                                          |
| Reconstruction overhead  | Extra CPU, network, and I/O needed to rebuild data from parity.                                         |
| RF                       | Replication Factor; number of copies maintained for data protection.                                    |
| RF2                      | Replication Factor 2; maintains two copies of data.                                                     |
| RF3                      | Replication Factor 3; maintains three copies of data.                                                   |
| SRE                      | Site Reliability Engineer; engineering role focused on reliability, automation, and production systems. |
| SLA                      | Service Level Agreement; contractual or operational service target.                                     |
| Stargate                 | Nutanix data I/O service running on CVMs.                                                               |
| Storage container        | Logical storage pool/policy boundary in Nutanix.                                                        |
| Strip / Stripe           | Group of data and parity units used by erasure coding.                                                  |
| Strip size               | Number of data/parity elements in an erasure-coded strip.                                               |
| Thin provisioning        | Allocating logical storage without consuming all physical capacity upfront.                             |
| Triage                   | Initial structured investigation to assess impact, scope, severity, and likely cause.                   |
| Usable capacity          | Storage capacity available for customer workloads after overhead.                                       |
| Write-cold window        | Time period after which unchanged data becomes eligible for erasure coding.                             |
| Write-heavy workload     | Workload with frequent writes or overwrites; often a weaker fit for EC.                                 |
| XOR                      | Logical operation often used conceptually to explain parity calculation.                                |

[1]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide%3Awc-erasure-coding-overview-wc-c.html&utm_source=chatgpt.com "Prism pc.7.5 - Erasure Coding - Nutanix"
[2]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism%3Awc-erasure-coding-best-practices-requirements-wc-c.html&utm_source=chatgpt.com "Prism 7.5 - Erasure Coding Best Practices and Requirements - Nutanix"
[3]: https://portal.nutanix.com/docs/Nutanix-Objects-v5_3%3Atop-wider-ec-strips-r.html?utm_source=chatgpt.com "Objects 5.3 - Larger Erasure Coding Strips - portal.nutanix.com"
[4]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-Prism-vpc_2022_4%3Awc-erasure-coding-overview-wc-c.html&utm_source=chatgpt.com "Prism pc.2022.4 - Erasure Coding (Prism Central) - Nutanix"
