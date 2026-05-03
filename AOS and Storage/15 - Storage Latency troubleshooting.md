# [AOS / Storage] — Storage Latency Troubleshooting

## 1. Short definition

**Storage latency troubleshooting** is the process of identifying why a VM, application, or cluster is experiencing slow storage I/O response times, then isolating whether the bottleneck is in the **guest OS, hypervisor, virtual disk, AOS storage path, CVM, physical disks, network replication path, workload pattern, or cluster health**.

In Nutanix terms, you are usually trying to answer:

> “Is this latency caused by the application workload, the VM, the hypervisor, the AOS distributed storage layer, the CVM, disk tiering, replication, cluster imbalance, or an infrastructure fault?”

AOS uses a distributed storage architecture with **Controller VMs — CVMs — on each node**, and Nutanix describes this model as a real-time, fully distributed data path designed for performance, resilience, and low latency under failure conditions. ([Nutanix][1])

---

## 2. Clear explanation

In a traditional enterprise storage model, you often troubleshoot latency by looking at a centralized SAN array, LUNs, controllers, fabric, paths, and host multipathing. In Nutanix, the model is different because storage is **software-defined and distributed across the cluster**.

At a high level:

1. A VM issues read/write I/O.
2. The hypervisor sends that I/O to the local Nutanix storage path.
3. The local **CVM** handles the storage request.
4. The **Stargate** process inside the CVM manages I/O.
5. AOS decides where the data should be read from or written to.
6. Data may involve local disks, remote replicas, OpLog, Extent Store, cache, SSD, HDD, or NVMe depending on the configuration and workload pattern.

Nutanix explains that AOS dynamically places writes based on real-time disk performance, usage, and health. The storage layer includes **OpLog** and **Extent Store**: OpLog is used to quickly persist and replicate bursty random writes, while Extent Store is the bulk persistent storage layer that receives sequential writes or data drained from OpLog. ([Nutanix][1])

For troubleshooting, the key is to avoid jumping directly to “storage is slow.” You need to segment the problem:

| Layer          | What you validate                                                 |
| -------------- | ----------------------------------------------------------------- |
| Application    | Query latency, batch jobs, application errors, change in workload |
| Guest OS       | Disk queue, filesystem, drivers, CPU steal/wait, OS logs          |
| VM / vDisk     | IOPS, throughput, latency, outstanding I/O, block size            |
| Hypervisor     | Host CPU, memory pressure, AHV/ESXi path, scheduler issues        |
| CVM            | CVM CPU, memory, service health, Stargate behavior                |
| AOS storage    | OpLog, Extent Store, read source, write destination, metadata     |
| Physical disks | SSD/HDD/NVMe health, saturation, errors, tiering                  |
| Network        | Replication path, east-west latency, packet loss, congestion      |
| Cluster        | Resiliency, rebalancing, rebuilds, upgrades, alerts               |

A strong interview answer should show that you understand **latency is a symptom**, not a root cause.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, this topic matters because storage latency cases are usually:

* **High-impact**: databases, ERP systems, VDI, Kubernetes platforms, and business-critical workloads are sensitive to storage delay.
* **Escalation-prone**: customers often perceive any application slowness as “Nutanix storage is slow.”
* **Cross-functional**: resolution may involve Support, SRE, Engineering, Account teams, Professional Services, hypervisor specialists, networking teams, and the customer’s application owners.
* **Data-driven**: the manager must push the team toward evidence: metrics, timelines, baselines, workload changes, alerts, and reproduction steps.
* **Communication-heavy**: customers need clear updates, not only technical investigation.

Nutanix positions AOS around dynamic distributed storage, automated data placement, and consistent low-latency performance during failure conditions. That means storage latency escalations directly challenge one of the core value propositions of the platform. ([Nutanix][1])

Your role is not to be the deepest Stargate engineer in the room. Your role is to make sure the team:

1. Defines the symptom precisely.
2. Separates perceived slowness from measured latency.
3. Identifies the affected blast radius.
4. Correlates latency with cluster events.
5. Engages the correct technical owners.
6. Communicates risk, mitigation, and next steps to the customer.

---

## 4. Key concepts

### A. Frontend vs backend latency

**Frontend latency** is what the VM or guest OS experiences.
**Backend latency** is what the storage system experiences internally while serving the I/O.

This distinction is critical. A VM may report high latency because of guest OS queues, CPU contention, application behavior, or hypervisor scheduling, even if the backend storage path is healthy.

In Nutanix troubleshooting, detailed pages such as **2009/vdisk_stats** can show vDisk-level latency histograms, I/O sizes, randomness, working set information, read source, and write destination. Nutanix Bible notes that the page includes frontend read/write latency histograms and other vDisk-level details useful for storage performance analysis. ([NutanixBible.com][2])

### B. Read latency vs write latency

Do not treat all latency as the same.

**Read latency** may be related to:

* Whether reads come from cache, SSD, HDD, or remote locations.
* Working set size.
* Data locality.
* Tiering behavior.
* Cold reads after migration or failover.
* Disk saturation.
* Snapshot/clone-related behavior.

**Write latency** may be related to:

* OpLog pressure.
* Replication factor.
* Network path between nodes.
* CVM load.
* SSD/NVMe performance.
* Sustained write workload.
* AOS background activity.
* Disk or node health.

Nutanix documentation states that random I/O can be written to OpLog, while sequential I/O may bypass OpLog and go directly to Extent Store. The 2009/vdisk_stats page can expose read source and write destination, which is useful when isolating whether latency is tied to HDD reads, OpLog writes, or Extent Store behavior. ([NutanixBible.com][2])

### C. OpLog

**OpLog** is a persistent write buffer/journal-like component used for bursty random writes. It is not just a volatile cache. Nutanix describes OpLog as persistent and replicated, designed to quickly persist and replicate bursty random writes. ([Nutanix][1])

Interview phrase:

> “For write latency, I would want to understand whether the workload is bursty random, sustained random, or sequential, because that affects whether the I/O is handled through OpLog or goes directly to Extent Store.”

### D. Extent Store

**Extent Store** is the bulk persistent storage layer in AOS. It can span storage tiers and receives either sequential/sustained writes directly or data drained from OpLog. ([Nutanix][1])

If reads are coming from slower tiers, such as HDD in hybrid clusters, read latency may increase. Nutanix Bible specifically highlights checking the read source when troubleshooting high read latency, noting that high read latency can be caused by reads served from HDD / Estore HDD. ([NutanixBible.com][2])

### E. Stargate

**Stargate** is the AOS service responsible for handling storage I/O. For interview purposes, you do not need to explain internal algorithms in depth, but you should know that Stargate is central to the Nutanix I/O path.

Practical phrasing:

> “If the case points to AOS storage latency, I would expect the team to review Stargate-related metrics, vDisk stats, latency histograms, read source, write destination, outstanding I/O, and any service health issues.”

### F. CVM

The **Controller VM** runs Nutanix distributed services on each node. AOS services are distributed across CVMs to avoid a single point of failure, and any node can assume leadership of services when required. ([Nutanix][1])

CVM health is critical. If a CVM is CPU-constrained, memory-constrained, unhealthy, or affected by service issues, storage performance may degrade.

### G. Replication Factor and resiliency

Nutanix writes data to multiple places depending on the replication factor. Nutanix describes AOS writing data to two or three places depending on RF, with one copy placed on the node where the application is running and additional copies placed according to the availability domain. ([Nutanix][3])

For write latency, replication matters because writes often need remote persistence before acknowledgment. Therefore, inter-node network issues can look like storage latency.

### H. Outstanding I/O and queue depth

High latency can be the result of high queueing, not necessarily a failing disk. Key metrics include:

* Average latency.
* Average operation size.
* Average outstanding I/O.
* IOPS.
* Throughput.
* Read/write ratio.
* Random/sequential profile.

Nutanix Bible recommends reviewing **average latency, average operation size, and average outstanding I/O** when investigating performance issues. ([NutanixBible.com][2])

### I. Noisy neighbor

A different VM may consume excessive IOPS or throughput and affect other workloads. Nutanix Prism Central supports **Storage QoS**, where administrators can set throttle limits on a VM to prevent noisy VMs from over-utilizing storage resources. ([Nutanix Portal][4])

For support leadership, this matters because the fix may be operational rather than code-level: isolate the workload, throttle it, migrate it, resize it, or schedule batch jobs differently.

### J. Cluster events

Storage latency often correlates with events such as:

* Node failure.
* Disk failure.
* Rebuild.
* Rebalancing.
* Upgrade.
* Snapshot activity.
* Backup job.
* Replication job.
* VM migration.
* Capacity pressure.
* Hardware alerts.
* Network instability.

A good escalation manager always asks: **“What changed?”**

---

## 5. How it appears in a real escalation

### Scenario

A customer opens a Severity 1 escalation:

> “Our Oracle database on Nutanix is slow. Storage latency jumped from 3 ms to 40 ms. The business is impacted. We need Nutanix to fix the cluster immediately.”

### What may be happening

Possible causes:

* A backup job started and changed the I/O profile.
* A batch process increased random writes.
* Reads are being served from HDD instead of SSD/cache.
* A CVM is overloaded.
* A disk is degraded or failing.
* A node is rebuilding after a hardware event.
* Network replication between CVMs is experiencing packet loss.
* A noisy neighbor VM is consuming excessive storage resources.
* The VM has high outstanding I/O because the application is driving more requests than the stack can serve.
* The issue is actually inside the guest OS, database, filesystem, or hypervisor scheduler.

### How you should manage it

As a support manager, your job is to drive structured triage:

1. **Confirm impact**

   * Which application?
   * How many VMs?
   * Business impact?
   * Since when?
   * Is it ongoing or intermittent?

2. **Define the latency**

   * Read or write?
   * Average or peak?
   * Frontend or backend?
   * Which vDisks?
   * Which time window?

3. **Check cluster health**

   * Any alerts?
   * Any failed disks or nodes?
   * Any CVM service issues?
   * Any recent upgrades or maintenance?

4. **Correlate workload**

   * Did IOPS increase?
   * Did block size change?
   * Did read/write ratio change?
   * Any backup, scan, migration, replication, or batch job?

5. **Check blast radius**

   * One VM?
   * One host?
   * One storage container?
   * One cluster?
   * Multiple clusters?

6. **Engage specialists**

   * SRE / senior support engineer for deep AOS metrics.
   * Hypervisor specialist if AHV/ESXi path is suspected.
   * Network team if replication latency is suspected.
   * Customer DBA/application owner if workload changed.
   * Engineering if there is a suspected product defect.

7. **Communicate clearly**

   * Current hypothesis.
   * Evidence collected.
   * Immediate mitigations.
   * Next data needed.
   * Ownership.
   * ETA for next update.

Strong manager-level phrase:

> “I would avoid declaring a root cause too early. I would first separate frontend VM latency from backend AOS latency, correlate the spike with cluster events and workload changes, and then assign clear workstreams: storage path, CVM health, physical media, network replication, hypervisor, and application behavior.”

---

## 6. Triage questions I should ask

### Impact and scope

1. When did the latency start?
2. Is it continuous, intermittent, or periodic?
3. Which applications are impacted?
4. Which VMs and vDisks are affected?
5. Is the issue isolated to one node, one cluster, one workload, or all workloads?
6. What is the business impact?
7. Is there a workaround currently in place?

### Latency characterization

8. Is the latency read, write, or both?
9. Are we looking at average latency, percentile latency, or maximum spikes?
10. What was the baseline before the incident?
11. What is the current latency?
12. Is the latency observed in Prism, guest OS, application logs, database metrics, or monitoring tools?
13. Are IOPS and throughput higher than usual?
14. What is the block size?
15. Is the workload random or sequential?

### Nutanix / AOS checks

16. Are there active Prism alerts?
17. Are all CVMs healthy?
18. Are Stargate services healthy?
19. Is there any disk, node, or NIC degradation?
20. Is the cluster rebuilding, rebalancing, upgrading, or under maintenance?
21. Is capacity close to threshold?
22. Are reads coming from SSD/cache or HDD?
23. Are writes going to OpLog or Extent Store?
24. Is there high outstanding I/O?

### Environmental checks

25. Any recent application release?
26. Any backup, antivirus, indexing, scan, or replication job?
27. Any VM migration?
28. Any hypervisor change?
29. Any network change?
30. Any firmware or driver update?
31. Any new workload onboarded?

### Customer-management questions

32. What is the customer’s desired outcome: root cause, immediate mitigation, capacity plan, or executive update?
33. Who is the technical decision-maker?
34. Who owns the application?
35. What is the communication cadence expected during the escalation?
36. Is this tied to an SLA or contractual obligation?

---

## 7. Likely interview questions

1. How would you troubleshoot a customer reporting high storage latency on Nutanix?
2. How do you distinguish application slowness from storage latency?
3. What Nutanix components are involved in the storage I/O path?
4. What is the role of the CVM?
5. What is Stargate?
6. What is OpLog used for?
7. What is Extent Store?
8. How would you investigate high write latency?
9. How would you investigate high read latency?
10. What metrics would you ask your engineers to check?
11. How would you manage a Sev1 storage latency escalation?
12. How would you communicate with an angry enterprise customer?
13. How do you prevent premature root-cause conclusions?
14. What could cause storage latency during a cluster rebuild?
15. How do you handle a case where the customer insists “Nutanix is slow” but the evidence is incomplete?
16. How would you coordinate Support, SRE, Engineering, and Account teams?
17. How do you use KPIs like MTTR and SLA in this kind of escalation?
18. What would you escalate to Engineering?
19. How would you run a post-incident review?
20. How would your previous cloud/SaaS experience help you in this scenario?

---

## 8. Model answers in English

### Q1. How would you troubleshoot high storage latency on Nutanix?

**Model answer:**

> I would start by defining the symptom precisely. I would ask whether the latency is read, write, or both; whether it is frontend latency seen by the VM or backend latency inside the storage layer; which VMs and vDisks are affected; and when the issue started.
>
> Then I would check the scope: single VM, single host, single cluster, or multiple workloads. After that, I would correlate the latency spike with cluster events such as disk failures, node issues, rebuilds, upgrades, backups, replication jobs, or workload changes.
>
> From a Nutanix perspective, I would expect the engineers to review Prism metrics, CVM health, Stargate-related metrics, vDisk statistics, latency histograms, read source, write destination, outstanding I/O, IOPS, throughput, and block size. I would also make sure we do not ignore the guest OS, hypervisor, application, and network replication path.
>
> As a manager, my role would be to keep the investigation structured, avoid premature conclusions, assign clear workstreams, maintain customer communication, and drive either mitigation or root cause depending on the severity.

### Q2. What is the difference between read latency and write latency in Nutanix?

**Model answer:**

> Read latency and write latency can have very different causes. For read latency, I would look at where the reads are being served from: cache, SSD, HDD, or remote data. If reads are coming from a slower tier, or if the working set is larger than expected, latency may increase.
>
> For write latency, I would look at the write path, including whether the workload is random, sequential, bursty, or sustained. Nutanix uses OpLog for bursty random writes and Extent Store for bulk persistent storage, so the workload pattern matters. I would also consider replication factor, inter-node network health, CVM load, and disk health.
>
> In an escalation, I would not simply say “storage is slow.” I would separate read and write behavior and validate each part of the path with metrics.

### Q3. What is OpLog?

**Model answer:**

> OpLog is a persistent write component in the Nutanix AOS storage architecture. It is used to quickly persist and replicate bursty random writes. It should not be described as a simple volatile cache, because writes in OpLog are persistent and replicated.
>
> From a troubleshooting perspective, OpLog is relevant when investigating write latency, especially for bursty random workloads. I would want to know whether the workload is overwhelming the write path, whether there is pressure on the devices used for OpLog, and whether network replication or CVM health is contributing to latency.

### Q4. What is Extent Store?

**Model answer:**

> Extent Store is the bulk persistent storage layer in AOS. It receives sequential or sustained writes directly, and it also receives data drained from OpLog. It can span storage tiers depending on the node configuration.
>
> For troubleshooting, Extent Store matters because reads or writes involving slower tiers, especially in hybrid environments, may affect latency. If read latency is high, I would ask where the reads are being served from. If they are coming from HDD rather than a faster tier, that may explain part of the behavior.

### Q5. What is Stargate?

**Model answer:**

> Stargate is the AOS service responsible for handling storage I/O. It is central to the Nutanix data path. In a storage latency escalation, engineers would typically look at Stargate-related metrics, vDisk stats, latency histograms, read source, write destination, and outstanding I/O.
>
> As a manager, I do not need to replace the deep SRE, but I need to know enough to ask the right questions and make sure the investigation is following the correct technical path.

### Q6. How would you manage an angry customer during a Sev1 latency case?

**Model answer:**

> I would first acknowledge the impact and establish a clear communication cadence. Then I would separate the technical bridge into structured workstreams: customer application and guest OS, hypervisor, Nutanix storage path, CVM health, physical media, network, and recent changes.
>
> I would make sure the team communicates facts, not guesses. For example: what we know, what we do not know yet, what data we are collecting, what mitigation options exist, and when the next update will be delivered.
>
> I would also ensure that we are tracking SLA, business impact, escalation ownership, and executive visibility if needed. After service is stabilized, I would drive a post-incident review focused on root cause, prevention, monitoring gaps, and customer trust recovery.

### Q7. How do you avoid blaming the wrong layer?

**Model answer:**

> I would insist on evidence-based isolation. Application slowness can come from database locks, CPU contention, memory pressure, guest OS queues, network latency, hypervisor scheduling, or storage latency. So I would compare metrics across layers and align them on the same timeline.
>
> If the application reports slow transactions, I would check whether guest disk latency increased at the same time. If guest latency increased, I would compare it with hypervisor and Nutanix backend metrics. If backend latency is normal but frontend latency is high, the issue may be outside AOS. If backend latency is also high, then we go deeper into AOS, CVM, disk, replication, and cluster events.

### Q8. What would you escalate to Engineering?

**Model answer:**

> I would escalate to Engineering when the support team has evidence suggesting a product defect, unexpected AOS behavior, service instability, reproducible performance degradation, or a known issue not covered by standard remediation. Before escalating, I would expect a clear problem statement, timeline, cluster version, workload profile, affected entities, logs, metrics, recent changes, and business impact.
>
> I would avoid vague escalations like “customer has latency.” Instead, I would package the case as: specific workload, specific vDisks, read or write latency, exact time window, observed metrics, steps already taken, hypotheses ruled out, and requested Engineering action.

---

## 9. Connection with your experience

Your background maps well to this topic because storage latency troubleshooting is not just about knowing every internal AOS detail. It is about **incident command, structured triage, and enterprise communication**.

You can connect your Harmonic experience this way:

| Your experience              | How to connect it to Nutanix storage latency                                                                                                                   |
| ---------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 24/7 enterprise support      | “I am used to managing high-pressure incidents with business impact and strict communication cadences.”                                                        |
| Incident management          | “I would structure the bridge, define workstreams, control escalation flow, and avoid random troubleshooting.”                                                 |
| SLA / MTTR                   | “I would track time-to-mitigation separately from time-to-root-cause.”                                                                                         |
| Cloud operations             | “The distributed nature of AOS is familiar to me because cloud platforms also require correlation across compute, storage, network, and control-plane layers.” |
| Monitoring: Grafana / Kibana | “I am comfortable correlating metrics, logs, time windows, and baselines.”                                                                                     |
| Jira / Confluence            | “I would document the timeline, actions, hypotheses, decisions, and post-incident review.”                                                                     |
| Salesforce                   | “I understand customer case management, severity, ownership, and executive communication.”                                                                     |
| Kubernetes / Linux           | “I can reason about workload behavior, guest OS metrics, I/O wait, containerized applications, and noisy workloads.”                                           |
| AWS / Azure / GCP            | “I understand how distributed systems fail and why latency must be isolated by layer rather than assumed.”                                                     |

A strong positioning statement:

> “My value is that I can manage the escalation technically without pretending to be the deepest individual contributor. I know how to drive the investigation, ask the right architectural questions, challenge weak hypotheses, protect the customer relationship, and bring Support, SRE, Engineering, and the customer into a coordinated resolution path.”

---

## 10. Minimum I need to memorize

Memorize these points:

1. **AOS is distributed storage**, not a traditional SAN.
2. Each node runs a **CVM** that hosts Nutanix services.
3. **Stargate handles storage I/O**.
4. **OpLog handles bursty random writes** and is persistent/replicated.
5. **Extent Store is bulk persistent storage**.
6. Separate **frontend latency** from **backend latency**.
7. Separate **read latency** from **write latency**.
8. For read latency, check **read source**: cache, SSD, HDD, remote.
9. For write latency, check **workload pattern, OpLog, replication, CVM load, and network**.
10. Always check **cluster health, alerts, disk/node failures, rebuilds, upgrades, and recent changes**.
11. Key metrics: **average latency, outstanding I/O, IOPS, throughput, block size, read/write ratio, randomness**.
12. In escalation, your job is to drive **scope, evidence, workstreams, mitigation, communication, and root cause**.

One clean verbal summary:

> “For Nutanix storage latency, I would isolate the issue by layer: application, guest OS, hypervisor, VM/vDisk, CVM, AOS storage path, disks, network, and cluster events. Then I would distinguish read from write latency, frontend from backend latency, and workload-driven latency from platform-driven latency. As a support manager, I would keep the investigation evidence-based, customer-facing communication clear, and escalation ownership explicit.”

---

## 11. Advanced / optional level

You do **not** need to master these deeply before the manager interview, but you should recognize the terms:

* Stargate internal counters.
* 2009 advanced pages.
* vDisk latency histograms.
* Curator jobs.
* Cassandra-based metadata.
* Paxos.
* Extent groups.
* AES / RocksDB local metadata.
* Erasure coding behavior.
* ILM / tier migration.
* RF2 vs RF3 performance implications.
* Hybrid vs all-flash latency behavior.
* AHV vs ESXi storage path differences.
* CVM service restarts and impact.
* NCC health checks.
* Foundation / LCM / firmware interactions.
* SPDK / NVMe performance path.

For interviews, it is fine to say:

> “I would rely on the senior SREs for the deepest internal counters, but I know what evidence I expect them to collect: vDisk latency histograms, read source, write destination, outstanding I/O, CVM health, Stargate metrics, disk health, and correlation with cluster events.”

That is the right manager-level answer.

---

## 12. Final checklist

Before an interview, you should be able to explain:

* [ ] What storage latency means.
* [ ] Why “the app is slow” is not the same as “storage is slow.”
* [ ] The Nutanix I/O path at a high level.
* [ ] What CVM does.
* [ ] What Stargate does.
* [ ] What OpLog does.
* [ ] What Extent Store does.
* [ ] Difference between read and write latency.
* [ ] Difference between frontend and backend latency.
* [ ] Key metrics to check.
* [ ] How cluster events can affect latency.
* [ ] How to manage a Sev1 customer escalation.
* [ ] How to communicate uncertainty without sounding weak.
* [ ] How your cloud/SaaS incident background transfers to Nutanix.
* [ ] What to escalate to SRE vs Engineering.
* [ ] How to package a good escalation.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------------------- |
| AES                      | Autonomous Extent Store; local physical metadata store used by AOS.                             |
| AHV                      | Acropolis Hypervisor; Nutanix native hypervisor.                                                |
| AOS                      | Acropolis Operating System; Nutanix core distributed storage and platform software.             |
| Application latency      | Delay experienced by the business application or service.                                       |
| Average latency          | Mean I/O response time; more useful than isolated maximum spikes.                               |
| Backend latency          | Latency inside the storage/backend path.                                                        |
| Blast radius             | Scope of impact: one VM, host, cluster, application, or customer environment.                   |
| Block size               | Size of each I/O request, such as 4K, 8K, 64K, or 1MB.                                          |
| Cache                    | Fast data-serving layer used to reduce read latency.                                            |
| Cassandra                | Distributed database technology used in modified form for Nutanix metadata.                     |
| Cluster                  | Group of Nutanix nodes working as one distributed system.                                       |
| Cluster health           | Overall condition of services, nodes, disks, alerts, and resiliency.                            |
| Controller VM            | See CVM.                                                                                        |
| Curator                  | AOS service/framework involved in background jobs and data management.                          |
| CVM                      | Controller Virtual Machine; VM on each node running Nutanix services.                           |
| Data locality            | Serving data from the node where the VM runs, when possible.                                    |
| Disk queue               | Pending I/O requests waiting to be served.                                                      |
| East-west traffic        | Network traffic between nodes or services inside the cluster.                                   |
| Erasure Coding           | Space-efficient data protection method using parity.                                            |
| ESXi                     | VMware hypervisor supported in many Nutanix environments.                                       |
| Extent                   | Logical unit/chunk of stored data.                                                              |
| Extent group             | Fine-grained AOS data grouping used for distributed storage.                                    |
| Extent Store             | AOS bulk persistent storage layer.                                                              |
| Frontend latency         | Latency observed by the VM, OS, or application.                                                 |
| Guest OS                 | Operating system running inside the VM.                                                         |
| HCI                      | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform. |
| HDD                      | Hard Disk Drive; slower magnetic storage tier.                                                  |
| Hot data                 | Frequently accessed data.                                                                       |
| I/O                      | Input/output operation, usually read or write.                                                  |
| I/O wait                 | CPU wait time caused by pending disk I/O.                                                       |
| IOPS                     | Input/output operations per second.                                                             |
| ILM                      | Information Lifecycle Management; movement of data between tiers based on access patterns.      |
| Latency histogram        | Distribution view showing how many I/O operations fall into latency ranges.                     |
| LCM                      | Life Cycle Manager; Nutanix component for upgrades and firmware/software lifecycle.             |
| MTTR                     | Mean Time To Resolve or Recover; operational incident metric.                                   |
| NCC                      | Nutanix Cluster Check; health-check toolset.                                                    |
| NIC                      | Network Interface Card.                                                                         |
| Node                     | Physical Nutanix server in a cluster.                                                           |
| Noisy neighbor           | Workload consuming excessive shared resources and affecting others.                             |
| NVMe                     | High-performance flash storage interface.                                                       |
| OpLog                    | Persistent replicated write layer for bursty random writes.                                     |
| Outstanding I/O          | I/O requests currently in flight or waiting.                                                    |
| Paxos                    | Consensus algorithm used for strict consistency in distributed systems.                         |
| Prism                    | Nutanix management interface.                                                                   |
| Prism Central            | Centralized Nutanix management and operations interface.                                        |
| Prism Element            | Cluster-level Nutanix management interface.                                                     |
| QoS                      | Quality of Service; limits or controls resource consumption.                                    |
| Random I/O               | Non-sequential reads or writes; common in databases and VM workloads.                           |
| RDBMS                    | Relational Database Management System; example: Oracle, SQL Server, PostgreSQL.                 |
| Read latency             | Time required to complete read operations.                                                      |
| Read source              | Location/tier serving reads, such as cache, SSD, HDD, or remote.                                |
| Rebalancing              | Redistribution of data or load across the cluster.                                              |
| Rebuild                  | Restoration of resiliency after disk/node failure.                                              |
| Replication Factor / RF  | Number of data copies maintained for resiliency, usually RF2 or RF3.                            |
| RF2                      | Replication Factor 2; two copies of data.                                                       |
| RF3                      | Replication Factor 3; three copies of data.                                                     |
| Root cause               | Underlying reason for the incident.                                                             |
| SLA                      | Service Level Agreement.                                                                        |
| SLO                      | Service Level Objective.                                                                        |
| SSD                      | Solid State Drive; faster flash storage tier.                                                   |
| Stargate                 | AOS service responsible for handling storage I/O.                                               |
| Storage container        | Logical Nutanix storage construct used to present storage to VMs.                               |
| Storage latency          | Delay between issuing and completing storage I/O.                                               |
| Storage QoS              | Storage performance control, often used to limit noisy workloads.                               |
| Throughput               | Data volume transferred per second, usually MB/s or GB/s.                                       |
| Triage                   | Structured initial investigation and prioritization.                                            |
| vDisk                    | Virtual disk attached to a VM.                                                                  |
| VM                       | Virtual Machine.                                                                                |
| Write destination        | Location/path where new writes are being handled.                                               |
| Write latency            | Time required to complete write operations.                                                     |

[1]: https://www.nutanix.dev/2022/08/03/nutanix-benefit-1-dynamically-distributed-storage/ "https://www.nutanix.dev/2022/08/03/nutanix-benefit-1-dynamically-distributed-storage/"
[2]: https://www.nutanixbible.com/pdf/4f-book-of-aos-administration.pdf "https://www.nutanixbible.com/pdf/4f-book-of-aos-administration.pdf"
[3]: https://www.nutanix.dev/2022/08/10/nutanix-benefit-2-automated-app-aware-data-management/ "https://www.nutanix.dev/2022/08/10/nutanix-benefit-2-automated-app-aware-data-management/"
[4]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2024_2%3Amul-storage-qos-pc-c.html "https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2024_2%3Amul-storage-qos-pc-c.html"
