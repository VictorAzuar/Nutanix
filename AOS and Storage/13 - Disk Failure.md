# [AOS / Storage] — Disk Failure

## 1. Short definition

A **disk failure** in Nutanix AOS is the loss, degradation, or predicted failure of a physical storage device inside a Nutanix node. In an AOS cluster, disks are part of a distributed storage fabric, so a single disk failure should normally **not cause immediate data loss** because data is replicated across the cluster. Nutanix documentation states that when a data drive fails, the cluster detects the hardware alert and starts reducing the impact of a second failure by rebuilding replicas through internal services such as **Curator** and **Stargate**. ([Nutanix Portal][1])

For an interview, the key message is:

> “A disk failure in Nutanix is not treated like a traditional single-server disk problem. It is a distributed storage resiliency event. The priority is to confirm cluster health, resiliency status, customer impact, rebuild progress, capacity headroom, and safe replacement workflow.”

---

## 2. Clear explanation

In a traditional architecture, storage may depend on RAID groups, SAN controllers, LUNs, and dedicated storage arrays. In Nutanix, storage is software-defined and distributed across nodes. AOS Storage is designed to replace traditional SAN/NAS patterns with an automated storage fabric that scales across on-premises and cloud environments. ([Nutanix][2])

Each Nutanix node contributes compute, memory, and storage. A **Controller VM**, or **CVM**, runs on each node and participates in the distributed storage system. The hypervisor sees storage in a relatively simple way, but under the hood, AOS distributes data, metadata, replicas, and I/O services across the cluster.

When a disk fails, the important sequence is usually:

1. **Detection**
   Prism / AOS receives a hardware alert that a disk is failed, missing, degraded, or predicted to fail.

2. **Impact assessment**
   Support checks whether the cluster is still resilient, whether any VM or application is impacted, and whether the system is under performance pressure.

3. **Replica rebuild / re-protection**
   Nutanix attempts to rebuild missing replicas elsewhere in the cluster, assuming enough healthy resources and capacity exist. Nutanix documentation describes Curator instructing Stargate to create another replica of guest VM data previously stored on the failed drive. ([Nutanix Portal][1])

4. **Replacement planning**
   The failed disk is identified by node, block, slot, serial number, disk type, and failure state. Replacement is coordinated with the customer, field services, hardware partner, or internal process.

5. **Post-replacement validation**
   After the disk is replaced, Support confirms that the disk is discovered, partitioned/added if required, alerts are cleared, NCC checks are clean, and resiliency is restored.

The interview-relevant point is not that you personally know every low-level command. The important part is that you can lead the incident with the right mental model:

> “I need to know whether this is a contained hardware event, a degraded resiliency event, or a customer-impacting storage incident.”

---

## 3. Why it matters for Nutanix / Worldwide Support

Disk failure matters heavily for a **Manager, Worldwide Support** role because it sits at the intersection of:

* enterprise customer trust,
* data protection,
* support SLAs,
* hardware logistics,
* escalation management,
* SRE-style risk reduction,
* and cross-functional collaboration.

A disk failure may sound routine, but for an enterprise customer it can trigger serious concerns: “Are my VMs safe?”, “Are we at risk of data loss?”, “Can we continue production?”, “Do we need emergency maintenance?”, “Why did performance degrade?”, “When will resiliency be restored?”

Nutanix documentation explicitly frames hardware failures as expected parts of the datacenter lifecycle, and says the architecture is designed to tolerate failures depending on cluster fault tolerance and container replication factor. ([Nutanix Portal][3])

For a support manager, your job is to ensure the team can:

* distinguish **noise** from **risk**,
* communicate clearly with customers,
* avoid unsafe replacement actions,
* coordinate parts and field engineering,
* escalate quickly when resiliency is compromised,
* maintain SLA discipline,
* and protect customer confidence during uncertainty.

A strong interview answer should show that you understand both the technical and operational dimensions:

> “For Nutanix Support, disk failure is a common but high-trust event. The technology is designed to tolerate it, but the support motion must still be disciplined: verify resiliency, assess customer impact, manage replacement safely, monitor rebuild, and communicate risk in business language.”

---

## 4. Key concepts

### 4.1 Data drive vs boot / metadata / performance device

Not every disk has the same role. Nutanix documentation for drive failures explains that drives can store persistent data, storage metadata, oplog, and Controller VM boot files. It also notes that cold-tier data is stored on HDDs, while SSDs can hold metadata, oplog, hot-tier data, and CVM boot files depending on platform design. ([Nutanix Portal][4])

For interviews, do not oversimplify by saying “a disk is just a disk.” A failed HDD used for cold data has a different risk/performance profile from a failed SSD involved in metadata, oplog, or boot.

Practical framing:

* **HDD failure**: often capacity/cold-tier related; common because HDDs have moving parts.
* **SSD failure**: potentially more performance-sensitive, especially if used for hot data, metadata, or oplog.
* **Boot device failure**: may affect CVM or host boot behavior depending on platform.
* **Metadata / performance device issue**: usually higher severity than a simple capacity-tier disk failure.

---

### 4.2 Replication Factor: RF2 and RF3

Nutanix uses replication to maintain data availability.

* **RF2** generally means two copies of data.
* **RF3** generally means three copies of data.

Nutanix documentation states that default clusters use redundancy factor 2, which can tolerate a single node or drive failure, while redundancy factor 3 can allow tolerance of two node or drive failures in different blocks when requirements are met. ([Nutanix Portal][5])

Interview-safe explanation:

> “With RF2, one failure is expected to be tolerated. But during rebuild, the system may be in a reduced-resiliency state. A second failure before re-protection completes can increase risk. With RF3, the system has stronger tolerance, but it requires more resources and must be correctly configured.”

---

### 4.3 Fault tolerance and failure domains

A failure domain is the level at which the platform can tolerate failures: disk, node, block, rack, etc.

Nutanix documentation describes resiliency levels based on replication factor, disk count, node count, block count, rack count, and whether erasure coding is enabled. ([Nutanix Portal][6])

For a support manager, this matters because the question is not just:

> “Did one disk fail?”

The better question is:

> “Given the current RF, node count, block/rack layout, capacity, and other alerts, what is the actual resiliency exposure?”

---

### 4.4 Stargate and Curator

At a simplified level:

* **Stargate** handles the distributed I/O path.
* **Curator** handles background tasks such as scanning, balancing, and data movement/rebuild activities.

Official Nutanix documentation describes Curator instructing Stargate to create another replica of guest VM data stored on the failed drive after a disk failure. ([Nutanix Portal][1])

Interview phrasing:

> “I would not try to manually micromanage the data rebuild. I would verify that the Nutanix services responsible for re-protection are healthy, that rebuild is progressing, and that there is enough capacity and performance headroom.”

---

### 4.5 Prism alerts and hardware view

Prism is the primary operational interface for many support workflows. Nutanix documentation says the Hardware Overview view displays hardware-specific performance and usage statistics plus recent hardware alerts and events, and that the information is dynamically updated. ([Nutanix Portal][7])

For disk failure:

* use Prism to identify the affected node/block/slot,
* confirm the alert,
* check cluster health,
* inspect hardware diagram/table,
* check events,
* and validate post-replacement status.

---

### 4.6 NCC health checks

**NCC**, Nutanix Cluster Check, is a health validation tool. Current Prism documentation says NCC checks can be run from the Prism Element Health dashboard or from the CVM command line. ([Nutanix Portal][8])

In a support interview, NCC is important because it signals disciplined troubleshooting:

> “I would use Prism and NCC to validate the current health state, not rely only on one alert or customer observation.”

---

### 4.7 Rebuild, re-protection, and capacity headroom

After a disk fails, AOS must restore the expected number of replicas. This requires:

* healthy disks,
* healthy nodes,
* enough free capacity,
* stable CVMs,
* functioning storage services,
* and acceptable cluster performance.

A key escalation risk is **insufficient capacity**. If the cluster is already close to full, re-protection may be slow, blocked, or risky.

A strong support-manager statement:

> “One of my first checks would be whether the cluster has enough free capacity and whether the rebuild is progressing. A disk failure in a healthy, well-sized cluster is usually routine; a disk failure in a capacity-constrained or already-degraded cluster is a much more serious escalation.”

---

### 4.8 Safe replacement workflow

Nutanix documentation warns that if a drive has not completely failed, you must wait for data migration to complete before physically removing the disk; otherwise, data may not be correctly replicated to the replacement drive. ([Nutanix Portal][9])

This is a very interview-relevant point. It shows operational maturity.

Do not say:

> “Just pull the failed disk and replace it.”

Say:

> “I would verify the exact failed component, confirm migration/rebuild state, follow the official replacement workflow, and avoid physical removal until it is safe.”

---

## 5. How it appears in a real escalation

### Scenario A — Routine single disk failure, no customer impact

A customer opens a case because Prism shows a disk failed on one node. VMs are running normally. No latency spike. Cluster is RF2 and otherwise healthy.

Your support-manager approach:

1. Acknowledge the alert and customer concern.
2. Confirm cluster health and resiliency status.
3. Identify the disk: cluster, node, block, slot, serial, disk type.
4. Check whether rebuild/re-protection has started.
5. Confirm whether there are other hardware/storage alerts.
6. Coordinate replacement through the correct hardware process.
7. Keep the customer informed until the replacement and validation are complete.

Customer-facing language:

> “At this stage, this appears to be a contained disk failure. Nutanix is designed to tolerate a single disk failure when the cluster is healthy and correctly protected. We are validating resiliency, rebuild progress, and replacement requirements. We will keep monitoring until the cluster returns to a fully protected state.”

---

### Scenario B — Disk failure plus degraded resiliency

A disk fails, and Prism reports reduced resiliency or additional warnings. Customer is worried about production risk.

Manager approach:

1. Increase severity if resiliency exposure is real.
2. Ask whether there is application impact.
3. Validate RF, cluster capacity, node health, and other active failures.
4. Engage senior SRE / storage specialist.
5. Freeze non-essential risky changes.
6. Communicate risk clearly.
7. Drive replacement and re-protection as priority.

Customer-facing language:

> “The failed disk itself is understood, but the current priority is resiliency restoration. We are treating this as elevated risk because the cluster may have reduced tolerance to an additional failure until re-protection completes.”

---

### Scenario C — Disk failure plus performance degradation

Customer reports high latency after a disk alert. VMs are slow. Business impact exists.

Likely causes to investigate:

* rebuild traffic causing additional I/O pressure,
* hot data affected by SSD/performance tier failure,
* insufficient capacity,
* existing storage latency,
* CVM or Stargate health issue,
* network congestion affecting remote reads/writes,
* multiple simultaneous hardware issues.

Manager approach:

1. Treat as customer-impacting incident.
2. Separate **availability risk** from **performance impact**.
3. Assign technical owner and customer comms owner.
4. Check storage latency, IOPS, rebuild activity, CVM health, node health.
5. Provide regular updates with facts, not speculation.
6. Escalate to engineering if internal services behave unexpectedly.

---

### Scenario D — Customer wants to remove the disk immediately

This is a classic support leadership moment. The right answer is not just technical; it is risk control.

Response:

> “Before removing the disk, I would confirm whether the drive has fully failed or whether data migration is still required. If the drive is not fully failed, removing it prematurely can increase risk. I would follow the official Nutanix replacement process and make sure the customer understands why waiting for the safe state matters.”

This aligns with Nutanix replacement documentation warning against physical removal before migration completes when the drive has not completely failed. ([Nutanix Portal][9])

---

## 6. Triage questions I should ask

### Customer impact

1. Are any production VMs or applications down?
2. Is the customer seeing latency, packet loss, timeouts, or application errors?
3. When did the alert appear?
4. Was there any recent maintenance, upgrade, expansion, or power event?
5. Is the issue isolated to one workload, one node, one cluster, or all services?

### Cluster health

6. What is the current Prism health status?
7. Are there any other active alerts besides the disk failure?
8. Is the cluster RF2 or RF3?
9. Is resiliency degraded?
10. Is rebuild/re-protection progressing?

### Hardware details

11. Which node/block/slot is affected?
12. Is the failed device HDD, SSD, NVMe, boot device, metadata device, or capacity-tier disk?
13. Is the disk fully failed, missing, predicted-to-fail, or throwing recoverable errors?
14. Is the serial number confirmed?
15. Is this a Nutanix appliance, OEM appliance, or partner hardware model?

### Capacity and risk

16. What is the current usable/free capacity?
17. Is the cluster close to capacity limits?
18. Are there snapshots, backups, clones, or workloads consuming unexpected capacity?
19. Is there enough space for re-protection?
20. Are there any additional failed disks, nodes, CVMs, NICs, or power components?

### Operational controls

21. Has a support bundle or NCC output been collected?
22. Has field replacement been initiated?
23. Is there a maintenance window requirement?
24. Are there customer change freezes or business-critical periods?
25. Who owns customer communication, technical investigation, and hardware logistics?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you manage a disk failure escalation for a strategic enterprise customer?
2. How do you communicate risk when the platform is designed to tolerate the failure?
3. How do you prioritize a disk failure case?
4. What KPIs would you monitor in support operations for hardware/storage incidents?
5. How would you coach a support engineer handling this case?
6. How do you avoid unnecessary escalation while still protecting customer trust?
7. What would you do if the customer is pushing to replace the disk immediately?
8. How do you handle a disk failure that becomes a performance incident?
9. How would you manage handoff between regions in a worldwide support model?
10. How do you distinguish customer-impacting urgency from technical severity?

### Technical / SRE interview

1. What happens in Nutanix when a disk fails?
2. What is RF2 vs RF3?
3. What are Stargate and Curator?
4. What is the difference between a disk failure and a node failure?
5. How do you validate cluster resiliency?
6. Why is capacity important during rebuild?
7. What would you check in Prism?
8. What is NCC used for?
9. Why can premature disk removal be dangerous?
10. How could a disk failure cause latency?

### Panel / escalation interview

1. A customer has one failed disk and one node showing intermittent issues. How do you proceed?
2. A customer says, “Nutanix promised no data loss. Why are you escalating this?” How do you answer?
3. The rebuild is not progressing. What do you do?
4. A field engineer replaced the wrong disk. How do you manage the situation?
5. The customer wants an RCA. What would you include?
6. How do you coordinate support, engineering, hardware logistics, and account teams?
7. How do you handle communication during a long rebuild?
8. How would you improve support process after repeated disk-related escalations?

---

## 8. Model answers in English

### Question 1: What happens when a disk fails in Nutanix?

**Model answer:**

> “In Nutanix, a disk failure is handled by the distributed storage layer rather than by a traditional external storage array. The cluster detects the failed or degraded drive, raises alerts in Prism, and starts restoring the expected data protection level by rebuilding replicas elsewhere in the cluster, assuming there is enough capacity and the rest of the cluster is healthy.
>
> From a support perspective, I would first check customer impact, cluster health, replication factor, resiliency status, rebuild progress, and any additional alerts. I would also identify the exact disk, node, slot, and serial number before coordinating replacement. The key is to treat it not only as a hardware replacement, but as a resiliency-management event.”

---

### Question 2: How would you manage a customer escalation for a failed disk?

**Model answer:**

> “I would split the response into three tracks: technical validation, customer communication, and replacement coordination.
>
> Technically, I would confirm whether the cluster is still resilient, whether workloads are impacted, whether rebuild is progressing, and whether there are other alerts such as capacity pressure, CVM issues, or additional hardware failures.
>
> For communication, I would reassure the customer without overpromising. I would explain that Nutanix is designed to tolerate disk failures, but that we are validating the actual risk based on the cluster state.
>
> Operationally, I would make sure the replacement is coordinated safely, with the correct disk identified and with post-replacement validation through Prism and health checks.”

---

### Question 3: What is the risk of a second failure?

**Model answer:**

> “The risk depends on the replication factor, current resiliency state, and failure domain. With RF2, the system is generally designed to tolerate one disk or node failure, but during the rebuild window the cluster may have reduced tolerance. If a second failure occurs before re-protection completes, the risk can increase significantly.
>
> That is why I would monitor rebuild progress, capacity headroom, and other hardware alerts closely. I would also communicate clearly to the customer that the first failure may be tolerated, but restoring full resiliency is time-sensitive.”

---

### Question 4: Why is capacity important during disk failure recovery?

**Model answer:**

> “Capacity matters because the cluster needs enough healthy storage to recreate replicas and restore the desired protection level. A disk failure in a healthy cluster with enough free capacity is usually manageable. But if the cluster is close to full, rebuild may be delayed or blocked, and the customer may remain in a degraded state longer.
>
> In an escalation, I would check usable capacity, storage pool health, active snapshots or clones, and whether any workloads are consuming abnormal storage. Capacity is not just a planning metric; during failure recovery, it becomes a resiliency metric.”

---

### Question 5: What would you check in Prism?

**Model answer:**

> “I would check the hardware view to identify the failed disk, the health dashboard for active alerts, the cluster resiliency status, recent events, storage capacity, performance metrics, and whether rebuild or re-protection activity is ongoing.
>
> I would also use NCC where appropriate to validate the broader cluster health. Prism gives the operational view, but I would not stop at the visible disk alert. I would look for correlated issues that could change the severity of the case.”

---

### Question 6: How would you explain this to a non-technical customer executive?

**Model answer:**

> “I would say: one storage device has failed, but the platform is designed to keep data available by maintaining redundant copies across the cluster. Our current focus is to verify that protection is intact or being restored, confirm that applications are not impacted, replace the failed component safely, and validate that the system returns to a fully healthy state.
>
> I would avoid unnecessary technical detail unless requested, but I would be transparent about risk if resiliency is degraded.”

---

### Question 7: How do you position yourself as a manager rather than a Senior SRE?

**Model answer:**

> “I would not try to be the deepest storage engineer in the room. My value is that I understand the architecture well enough to ask the right questions, assess risk, prioritize correctly, and coordinate the right people.
>
> In a disk failure escalation, I would expect my SREs or senior support engineers to validate low-level commands and logs, while I ensure clear ownership, SLA discipline, customer communication, safe execution, and escalation to engineering or hardware teams when needed.”

---

## 9. Connection with my experience

Your background maps very well to this topic.

At Harmonic, you already manage:

* 24/7 support operations,
* customer escalations,
* incident management,
* SLA and MTTR,
* monitoring,
* cloud operations,
* team coordination,
* and post-incident follow-up.

The Nutanix-specific gap is not the management pattern. The gap is the platform vocabulary and storage architecture.

You can translate your experience like this:

| Your current experience        | Nutanix disk failure equivalent                                     |
| ------------------------------ | ------------------------------------------------------------------- |
| Incident management            | Failed disk with degraded resiliency or customer impact             |
| SLA / MTTR tracking            | Time to acknowledge, diagnose, replace, and restore resiliency      |
| Monitoring with Grafana/Kibana | Prism alerts, events, NCC, hardware health                          |
| Cloud operations               | Distributed infrastructure health and redundancy                    |
| Escalation management          | Engaging SRE, engineering, field services, TAM/account teams        |
| Customer communication         | Explaining risk, impact, next steps, and ETA without overpromising  |
| Team coaching                  | Ensuring engineers validate impact, not just close hardware tickets |

A strong personal positioning statement:

> “My experience is in running enterprise support operations where infrastructure incidents must be managed with discipline: impact assessment, SLA ownership, technical escalation, clear communication, and prevention of recurrence. For Nutanix, I would apply the same operating model, while building deeper fluency in AOS storage, Prism, RF, resiliency, and hardware replacement workflows.”

---

## 10. Minimum I need to memorize

You should be able to explain these without notes:

1. **A disk failure in Nutanix is a distributed storage resiliency event, not just a hardware swap.**

2. **AOS replicates data across the cluster.**
   A single disk failure should normally not cause data loss if the cluster is healthy and correctly protected.

3. **RF2 vs RF3:**
   RF2 generally tolerates one failure; RF3 provides stronger tolerance but requires more resources and correct configuration.

4. **Curator and Stargate are involved in re-protection.**
   Curator coordinates background repair/rebuild work; Stargate participates in the data path and replica creation.

5. **Prism is the main operational UI.**
   Use it to check hardware alerts, affected disk, health, events, capacity, and performance.

6. **NCC validates cluster health.**
   It is useful before/after maintenance, during escalations, and when checking broader cluster health.

7. **Capacity matters during rebuild.**
   Without enough free capacity, re-protection can be delayed or impaired.

8. **Do not remove a disk blindly.**
   Confirm whether migration/rebuild is complete and follow the official replacement workflow.

9. **Customer communication must balance reassurance and risk transparency.**

10. **As a manager, your role is coordination, prioritization, communication, and escalation control — not pretending to be the deepest storage IC.**

---

## 11. Advanced / optional level

These are useful but not mandatory for a manager interview unless the SRE panel goes deeper.

### Advanced topics to understand later

* detailed AOS data path,
* extent groups and metadata,
* Cassandra / Medusa / Zookeeper roles,
* oplog behavior,
* SSD tiering and hot/cold data placement,
* erasure coding impact during failures,
* block awareness and rack awareness,
* rebuild throttling,
* degraded node vs failed disk scenarios,
* AHV vs ESXi operational differences,
* hardware platform differences,
* disk firmware and LCM,
* SED / encryption implications,
* detailed CLI troubleshooting.

### Useful advanced statement

> “At my level, I would not claim to know every internal AOS command, but I understand that disk failure handling depends on the distributed storage services, replica placement, cluster capacity, and failure domain. I would rely on Prism, NCC, logs, and senior SRE validation to make safe operational decisions.”

---

## 12. Final checklist

Before an interview, you should be able to answer “yes” to each item:

* [ ] I can explain what happens when a Nutanix disk fails.
* [ ] I can explain why a single disk failure does not normally equal data loss.
* [ ] I can explain RF2 vs RF3 in simple terms.
* [ ] I can describe why rebuild/re-protection matters.
* [ ] I can explain why capacity headroom matters.
* [ ] I can describe the role of Prism in disk failure triage.
* [ ] I can explain what NCC is used for.
* [ ] I can explain why premature disk removal can be risky.
* [ ] I can ask good triage questions during an escalation.
* [ ] I can communicate the situation to a technical engineer and a customer executive.
* [ ] I can position myself as a technical escalation manager, not a storage-only IC.
* [ ] I can connect this topic to my SaaS/cloud incident-management experience.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------------------- |
| AHV                      | Acropolis Hypervisor; Nutanix native hypervisor.                                                |
| AOS                      | Acropolis Operating System; Nutanix distributed infrastructure software layer.                  |
| AOS Storage              | Nutanix software-defined distributed storage fabric.                                            |
| Availability Domain      | Failure boundary such as disk, node, block, or rack.                                            |
| Block                    | Nutanix chassis containing one or more nodes.                                                   |
| Block Awareness          | Placement strategy to tolerate block-level failures.                                            |
| Boot Drive               | Disk/device used for boot-related components, depending on platform.                            |
| Capacity Headroom        | Free usable capacity needed for growth, rebuild, and resiliency.                                |
| Cold Tier                | Lower-performance storage tier, often HDD-based in hybrid systems.                              |
| Controller VM            | VM running Nutanix storage/control services on each node.                                       |
| Curator                  | AOS service involved in background scans, balancing, and re-protection workflows.               |
| CVM                      | Controller VM.                                                                                  |
| Data Drive               | Disk used to store user/workload data in the cluster.                                           |
| Degraded Resiliency      | State where expected protection level is reduced.                                               |
| Disk Failure             | Physical or logical failure/degradation of a storage device.                                    |
| Disk Slot                | Physical location of a disk in a node/chassis.                                                  |
| Erasure Coding           | Capacity efficiency technique that stores data with parity-like protection.                     |
| ESXi                     | VMware hypervisor supported in some Nutanix environments.                                       |
| Field Engineer           | Engineer responsible for on-site hardware replacement or repair.                                |
| Failure Domain           | Infrastructure scope affected by a failure: disk, node, block, rack, etc.                       |
| Firmware                 | Low-level device software; disk firmware can affect stability and compatibility.                |
| Grafana                  | Monitoring/visualization platform; relevant analogy from your background.                       |
| Hardware Alert           | Prism or system alert indicating physical component issue.                                      |
| HCI                      | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform. |
| HDD                      | Hard Disk Drive; spinning disk, often used for capacity/cold tier.                              |
| Hot Tier                 | High-performance storage tier, usually SSD/NVMe-based.                                          |
| I/O                      | Input/output operations between workloads and storage.                                          |
| IOPS                     | Input/output operations per second; storage performance metric.                                 |
| Jira                     | Ticket/workflow system; analogous to support case tracking.                                     |
| Kibana                   | Log analysis/visualization tool; relevant analogy from your background.                         |
| LCM                      | Life Cycle Manager; Nutanix lifecycle/firmware/software update tool.                            |
| Metadata                 | Data describing where and how workload data is stored.                                          |
| MTTR                     | Mean Time To Repair/Restore; key support operations metric.                                     |
| NCC                      | Nutanix Cluster Check; health-check tool for Nutanix environments.                              |
| Node                     | Physical server in a Nutanix cluster.                                                           |
| NVMe                     | High-performance SSD interface/protocol.                                                        |
| Oplog                    | Performance-sensitive write path component in Nutanix storage architecture.                     |
| Prism                    | Nutanix management interface for cluster operations and monitoring.                             |
| Prism Central            | Centralized Nutanix management plane for multiple clusters.                                     |
| Prism Element            | Cluster-level Nutanix management interface.                                                     |
| Re-protection            | Restoring the expected number of data replicas after a failure.                                 |
| Rebuild                  | Process of recreating data replicas on healthy storage.                                         |
| Redundancy Factor        | Number of data copies maintained for protection.                                                |
| RF2                      | Redundancy Factor 2; generally two copies, tolerating one failure.                              |
| RF3                      | Redundancy Factor 3; generally three copies, stronger failure tolerance.                        |
| RCA                      | Root Cause Analysis.                                                                            |
| SED                      | Self-Encrypting Drive.                                                                          |
| SLA                      | Service Level Agreement.                                                                        |
| SSD                      | Solid State Drive; flash-based storage device.                                                  |
| Stargate                 | AOS service involved in the distributed storage data path.                                      |
| Storage Pool             | Logical grouping of physical storage resources.                                                 |
| Support Bundle           | Collected logs/diagnostics for troubleshooting.                                                 |
| Triage                   | Initial structured assessment of impact, severity, and next actions.                            |
| VM                       | Virtual Machine.                                                                                |
| VMware                   | Virtualization vendor; ESXi/vSphere may run on Nutanix.                                         |
| vSphere                  | VMware virtualization management/platform stack.                                                |
| Worldwide Support        | Global support operating model across regions/time zones.                                       |

[1]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_8%3Aarc-data-drive-failure-c.html&utm_source=chatgpt.com "Prism 6.8 - Data Drive Failure - Nutanix"
[2]: https://www.nutanix.com/products/nutanix-cloud-infrastructure/distributed-storage?utm_source=chatgpt.com "A Modern Storage Fabric - Nutanix"
[3]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v7_0%3Awc-cluster-failure-handling-c.html&utm_source=chatgpt.com "Prism 7.0 - Failure Handling in a Nutanix Cluster"
[4]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v7_3%3Aarc-disk-failures-c.html&utm_source=chatgpt.com "Prism 7.3 - Drive Failures - portal.nutanix.com"
[5]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v6_10%3Aarc-redundancy-factor3-c.html?utm_source=chatgpt.com "Prism 6.10 - Redundancy Factor 3 - Nutanix"
[6]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_8%3Aarc-data-resiliency-levels-r.html&utm_source=chatgpt.com "Prism 6.8 - Data Resiliency Levels for Rack Fault Tolerance - Nutanix"
[7]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_7%3Awc-hardware-management-wc-c.html&utm_source=chatgpt.com "Prism 6.7 - Hardware Management - Nutanix"
[8]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_5%3Ancc-ncc-checks-run-c.html?utm_source=chatgpt.com "Prism 7.5 - Nutanix Cluster Check (NCC)"
[9]: https://portal.nutanix.com/page/documents/details?targetId=Hardware_Replacement-Acr_v4_6%3Abre_hdd_replace_prepare_wc_t.html&utm_source=chatgpt.com "AOS 4.6 - Preparing to Replace a Data Drive - portal.nutanix.com"
