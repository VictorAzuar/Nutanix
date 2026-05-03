# AOS / Storage — Snapshots

## 1. Short definition

A **snapshot** in Nutanix AOS is a **point-in-time copy/reference of a VM, vDisk, Volume Group, file, or protected entity** that can be used for restore, clone creation, replication, and disaster recovery workflows.

In practical enterprise-support language: a snapshot lets you say, “This workload looked like this at 10:00. If something breaks at 10:30, we may be able to roll back or recover from the 10:00 state.”

Nutanix documentation describes snapshots as point-in-time states of protected entities such as VMs and Volume Groups, used for restoration and replication. ([Nutanix Portal][1])

---

## 2. Clear explanation

In Nutanix, snapshots are part of the **AOS storage and data protection architecture**. AOS provides the distributed storage layer, commonly referred to as **DSF — Distributed Storage Fabric**, which aggregates local disks across the cluster and presents resilient, software-defined storage to the hypervisor. ([Nutanix][2])

A snapshot does **not** usually mean “copy the whole VM immediately.” That would be slow and expensive. Instead, Nutanix snapshots are designed to be efficient. Nutanix documentation describes AOS as using a **redirect-on-write** mechanism, which improves efficiency by avoiding a full data copy at snapshot creation time. ([Nutanix Portal][3])

Conceptually:

1. A VM or vDisk is running normally.
2. A snapshot is taken at a specific point in time.
3. After that point, new writes are redirected to new locations.
4. The snapshot keeps a reference to the previous data state.
5. The system can later restore, clone, replicate, or inspect that state.

This matters because snapshots are often used for:

* Backup workflows.
* Disaster recovery.
* Replication between clusters.
* Fast rollback before maintenance.
* Test/dev cloning.
* Recovery after accidental deletion, corruption, patch failure, ransomware, or application error.

However, a snapshot is **not the same as a backup**. A snapshot is usually stored within the same platform or protection workflow, while a backup should provide an independent recoverable copy according to the customer’s retention, compliance, and disaster recovery requirements. Nutanix’s own developer material explicitly makes this distinction: a snapshot can support backup workflows, but a snapshot itself is not a backup. ([nutanix.dev][4])

For interview purposes, you should be able to explain snapshots at three levels:

**Manager level:**
Snapshots are point-in-time recovery points used to reduce business risk and support fast restore/DR.

**Technical support level:**
Snapshots depend on AOS storage metadata, changed-block tracking/redirect-on-write behavior, protection domains or protection policies, schedules, retention, and replication.

**Escalation level:**
Snapshot issues can affect capacity, performance, restore success, replication lag, RPO/RTO, application consistency, and customer trust during an outage.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role at Nutanix, snapshots matter because they sit at the intersection of:

* **Storage**
* **Virtualization**
* **Disaster recovery**
* **Enterprise support**
* **Escalation management**
* **Customer data protection**
* **SLA/RPO/RTO expectations**
* **Incident communication**

In an enterprise escalation, the customer will not usually say, “Please explain redirect-on-write.” They will say:

> “Our production VM restore failed.”
> “Replication is delayed.”
> “The cluster is running out of capacity.”
> “Snapshots are consuming too much space.”
> “After restoring, the application is inconsistent.”
> “We need to prove our DR posture before an audit.”
> “Our backup vendor says Nutanix snapshots are the issue.”

As a support manager, your job is not to debug every metadata path personally. Your job is to:

* Understand the architecture well enough to ask the right questions.
* Separate customer-impacting risk from technical noise.
* Coordinate Support, SRE, Engineering, TAM, account teams, and possibly third-party vendors.
* Drive incident progress.
* Communicate clearly with enterprise customers.
* Know when the issue is operational, configuration-related, product-related, capacity-related, or a true defect.

Snapshots are also important because they are often used during **high-stress customer moments**: failed upgrades, ransomware recovery, database corruption, accidental deletion, disaster recovery tests, and major service-impacting incidents.

---

## 4. Key concepts

### Snapshot

A point-in-time representation of a workload, VM, vDisk, Volume Group, or file. It allows recovery or cloning from a previous state.

### VM-centric snapshot

Nutanix emphasizes VM-centric protection rather than only traditional LUN-centric storage protection. This is important because enterprise customers care about restoring workloads, not just abstract storage objects. Nutanix material describes snapshots as granular and VM-centric, at the vDisk level rather than only at a larger LUN/container level. ([nutanix.dev][4])

### Redirect-on-write

A storage mechanism where new writes after a snapshot are redirected elsewhere, while the snapshot preserves references to the previous state. This helps make snapshots faster and more efficient than full-copy approaches. Nutanix describes AOS snapshots as using redirect-on-write for efficiency. ([Nutanix Portal][3])

### Crash-consistent snapshot

A snapshot equivalent to the state of a system after a sudden power loss. The filesystem and application may need recovery on boot. It is usually easier to create than an application-consistent snapshot.

### Application-consistent snapshot

A snapshot where the application is quiesced or coordinated so that in-flight writes are flushed and the application can recover cleanly. For Windows workloads, Nutanix documentation states that application-consistent snapshots can invoke **VSS — Volume Shadow Copy Service** under specific conditions. ([Nutanix Portal][5])

### Protection Domain

In Prism Element data protection workflows, a **Protection Domain** groups protected entities such as VMs and applies schedules, retention, and replication policies. Nutanix training material describes a Protection Domain as being made up of VMs and a policy including snapshots, replication locations, and schedules. ([nutanix-technologybootcamp.readthedocs.io][6])

### Protection Policy / Recovery Point

In newer Prism Central-oriented workflows, snapshots may be referred to as **Recovery Points**. Nutanix documentation notes that snapshots are called Recovery Points in Prism Central when using protection policies. ([Nutanix Portal][7])

### Retention

How many snapshots are kept and for how long. Poor retention configuration can cause capacity pressure, replication backlog, or customer surprise during restore.

### RPO

**Recovery Point Objective**: how much data loss is acceptable. If snapshots happen every hour, the theoretical RPO may be around one hour, assuming replication and recovery are working correctly.

### RTO

**Recovery Time Objective**: how long it takes to recover service. Snapshots help reduce RTO, but restore speed depends on workload size, cluster health, network, replication status, and operational execution.

### Replication

Snapshots can be replicated to another cluster for disaster recovery. This introduces dependencies: network bandwidth, schedule, remote site capacity, protection policy health, and replication lag.

### Snapshot chain / changed data

Multiple snapshots mean the platform must track more changed data over time. Even efficient snapshots are not free. Nutanix performance guidance warns that more snapshots mean more changed data and can impose performance penalties. ([Nutanix Portal][3])

---

## 5. How it appears in a real escalation

### Escalation scenario 1 — Customer cannot restore a critical VM

A customer reports:

> “We deleted a production VM by mistake. We need to restore from last night’s snapshot, but the restore is failing.”

You would lead the escalation by checking:

* Is there a valid snapshot/recovery point?
* Is it local or remote?
* Is the VM protected by a Protection Domain or Protection Policy?
* Was the snapshot completed successfully?
* Is there enough cluster capacity to restore?
* Is the issue restore workflow, storage metadata, Prism UI, API, or hypervisor integration?
* What is the business impact?
* What is the customer’s RTO?
* Is there a workaround: clone from snapshot, restore to alternate VM, restore on DR site?

Your communication should avoid overpromising. You would say something like:

> “We have confirmed that a recovery point exists from the requested timestamp. The next priority is to validate whether the failure is related to restore workflow, capacity constraints, or metadata access. We are parallelizing investigation with storage and platform specialists while preserving the recovery point.”

---

### Escalation scenario 2 — Snapshot retention causing capacity pressure

A customer reports:

> “The cluster is running out of space and we suspect snapshots are consuming too much capacity.”

Triage direction:

* Check snapshot count and retention policy.
* Identify large/high-change-rate VMs.
* Check whether replication is stuck, causing retained snapshots to accumulate.
* Check if backup integrations are leaving snapshots behind.
* Confirm whether deletion is safe or whether retention is required for compliance.
* Avoid asking the customer to delete snapshots blindly.

Manager-level risk:

* Deleting snapshots may reduce recovery options.
* Not deleting may risk cluster capacity exhaustion.
* The decision may need customer approval, backup team involvement, and possibly Engineering review.

---

### Escalation scenario 3 — Application inconsistency after restore

A customer reports:

> “The VM restored, but SQL Server / Exchange / application X is inconsistent.”

Triage direction:

* Was the snapshot crash-consistent or application-consistent?
* Was VSS enabled and successful for Windows?
* Was the application supported for app-consistent snapshot?
* Are Nutanix Guest Tools involved?
* Are there application logs showing quiescing failure?
* Was the database using external disks, Volume Groups, or multiple VMs?
* Was consistency required across several VMs?

This is where you demonstrate maturity: the restore operation may have worked technically, but the customer outcome failed because the application consistency model was misunderstood.

---

### Escalation scenario 4 — Replication lag / missed RPO

A customer reports:

> “Our DR snapshots are not arriving at the remote site. We are missing our RPO.”

Triage direction:

* Is local snapshot creation succeeding?
* Is remote replication configured correctly?
* Is there network latency, packet loss, or bandwidth saturation?
* Is the remote cluster healthy?
* Is there enough capacity at the target?
* Did a large change rate overwhelm the schedule?
* Was there a recent upgrade, topology change, firewall change, or certificate issue?
* What RPO is contractually/business critical?

As a support manager, your language should be risk-based:

> “The immediate customer risk is RPO exposure. We need to determine whether the issue is snapshot generation, replication transport, remote ingestion, or target capacity. Until replication catches up, the customer’s recoverable point may be older than expected.”

---

## 6. Triage questions I should ask

Use these in interviews and real escalations.

### Business impact

1. What workload is affected?
2. Is this production, pre-production, or test/dev?
3. What is the customer’s required RPO and RTO?
4. Is there active data loss, service degradation, or only a failed protection job?
5. Is this related to an ongoing outage, DR test, migration, upgrade, or audit?

### Snapshot scope

6. Is the snapshot for a VM, vDisk, Volume Group, file, container, or Protection Domain?
7. Is it managed through Prism Element, Prism Central, a Protection Domain, a Protection Policy, or a third-party backup tool?
8. Is the expected snapshot visible in Prism/API/CLI?
9. Is the snapshot local or replicated?

### Consistency

10. Is the snapshot crash-consistent or application-consistent?
11. For Windows, was VSS used and did it complete successfully?
12. Does the application require multi-VM consistency?
13. Are there databases, clustered applications, or external volumes involved?

### Capacity and performance

14. How many snapshots exist?
15. What is the retention policy?
16. Are high-change-rate workloads protected?
17. Is the cluster under capacity pressure?
18. Are there performance symptoms during snapshot creation, deletion, replication, or restore?

### Replication / DR

19. Is replication enabled?
20. Is the remote site reachable and healthy?
21. Is replication lag increasing?
22. Has the network, firewall, DNS, certificate, or routing changed recently?
23. Is the target cluster short on capacity?

### Recent change

24. Was there a recent AOS, AHV, ESXi, Prism, backup agent, or application upgrade?
25. Was a new backup product introduced?
26. Were protection policies changed?
27. Did someone manually delete snapshots or modify retention?

### Escalation management

28. What is the customer’s desired recovery outcome?
29. What actions could destroy or reduce recovery options?
30. Who must approve destructive actions?
31. Do we need Engineering, SRE, backup vendor, VMware team, network team, or database team involved?
32. What is the next customer-facing update time?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you manage an escalation where a customer cannot restore from a Nutanix snapshot?
2. How do you balance technical investigation with customer communication during a DR incident?
3. How would you handle a customer who says Nutanix snapshots caused a production outage?
4. How do you decide when to involve Engineering?
5. How would you manage a situation where Support, SRE, and a third-party backup vendor disagree?
6. What KPIs would you track for snapshot-related support issues?
7. How do snapshots relate to RPO and RTO?
8. What would you communicate to an executive customer during a failed recovery event?

### Technical / SRE interview

9. What is a snapshot?
10. What is the difference between snapshot and backup?
11. What is the difference between crash-consistent and application-consistent snapshots?
12. How does redirect-on-write help snapshot efficiency?
13. Why can too many snapshots become a problem?
14. How can snapshots affect capacity?
15. How would you troubleshoot replication lag?
16. What is the role of Prism Element and Prism Central in data protection?
17. What is a Protection Domain?
18. What is a Recovery Point?
19. How would VSS relate to Nutanix snapshots?
20. How would you approach a restore failure?

### Panel / advanced escalation interview

21. A customer’s DR test failed because the restored app is inconsistent. How do you lead the incident?
22. A customer has thousands of snapshots and severe capacity pressure. What do you do?
23. A snapshot deletion operation appears stuck. How do you handle customer communication?
24. A backup vendor claims Nutanix APIs are causing snapshot sprawl. How do you investigate?
25. A customer wants a guarantee that snapshots equal backups. How do you respond?

---

## 8. Model answers in English

### Q1. What is a snapshot in Nutanix AOS?

**Model answer:**

> A snapshot in Nutanix AOS is a point-in-time representation of a protected entity, such as a VM, vDisk, Volume Group, or file. It allows the platform to restore, clone, or replicate that state later. From a support perspective, I would describe it as a recovery point, but I would also be careful to explain that a snapshot is not the same thing as an independent backup. It is part of the data protection workflow and must be managed with proper retention, replication, and recovery validation.

---

### Q2. What is the difference between a snapshot and a backup?

**Model answer:**

> A snapshot captures the state of a workload at a specific point in time, usually very efficiently and often within the same storage or virtualization platform. A backup is a broader data protection mechanism that should provide an independent recoverable copy, often with separate retention, compliance, and off-platform protection. In an escalation, I would avoid telling a customer that snapshots alone are their backup strategy unless their architecture explicitly supports that with replication, retention, and external protection requirements.

---

### Q3. How do snapshots relate to RPO and RTO?

**Model answer:**

> Snapshots directly influence RPO because the snapshot schedule determines how recent the available recovery point is. For example, if snapshots run every hour and complete successfully, the customer may have roughly a one-hour recovery point objective, depending on replication and policy health. Snapshots can also reduce RTO because restoring or cloning from a snapshot is usually faster than rebuilding from scratch. However, actual RTO depends on workload size, cluster health, target capacity, replication status, and the operational recovery process.

---

### Q4. What is the risk of having too many snapshots?

**Model answer:**

> Snapshots are efficient, but they are not free. Too many snapshots increase metadata and changed-data tracking, consume capacity, and may affect performance or operational tasks such as deletion, replication, and restore. In a support case, I would check retention policies, high-change-rate VMs, backup integrations, replication backlog, and whether snapshots are accumulating due to a failed protection workflow. I would also be careful not to recommend deletion without understanding the customer’s recovery and compliance requirements.

---

### Q5. What is crash-consistent versus application-consistent?

**Model answer:**

> A crash-consistent snapshot is similar to the state of a system after a power failure. The VM can often boot, but the application may need recovery. An application-consistent snapshot coordinates with the guest OS or application so that writes are flushed and the application is in a recoverable state. For Windows workloads, this can involve VSS. In a customer escalation, this distinction is critical because a restore may technically succeed while the application is still inconsistent if the wrong consistency level was used.

---

### Q6. How would you handle an escalation where restore from snapshot fails?

**Model answer:**

> I would first clarify the business impact: production or non-production, the required RTO, and whether there is an active outage. Then I would validate that the expected recovery point exists, determine whether it is local or replicated, and check whether the failure is related to capacity, metadata, Prism workflow, hypervisor integration, or application consistency. I would run parallel tracks: one team validating the recovery path, another checking cluster health and logs, and another preparing customer communication. I would also protect the recovery point and avoid destructive actions until we understand the risk.

---

### Q7. How would you explain Nutanix snapshots to an enterprise customer during an incident?

**Model answer:**

> I would keep the explanation outcome-focused. I would say that snapshots provide point-in-time recovery points that can be used to restore or clone workloads. Then I would explain where we are in the process: whether the recovery point exists, whether it is local or remote, whether the restore workflow is succeeding, and what risks remain. I would avoid deep internals unless the customer asks, but I would be transparent about RPO, RTO, capacity, and application consistency.

---

### Q8. How would you manage a disagreement between Nutanix Support and a backup vendor?

**Model answer:**

> I would separate facts from assumptions. First, I would define the failing operation: snapshot creation, retention cleanup, API call, replication, restore, or application consistency. Then I would collect timestamps, job IDs, Prism events, backup job logs, API responses, and cluster health data. I would assign owners for each investigation stream and keep the customer focused on impact and next actions. The goal is not to prove blame first; it is to restore service or reduce risk, then complete root-cause analysis.

---

### Q9. What would you monitor operationally around snapshots?

**Model answer:**

> I would monitor snapshot job success rate, failed protection jobs, replication lag, snapshot age, retention compliance, cluster capacity, high-change-rate workloads, restore test success, and the number of stale or orphaned snapshots. From a management perspective, I would also track case volume, escalation rate, time to mitigation, time to restore, and recurring root causes such as misconfiguration, insufficient capacity, unsupported app-consistency expectations, or third-party integration issues.

---

### Q10. Why are snapshots important in HCI?

**Model answer:**

> In HCI, compute, storage, and virtualization are tightly integrated. Snapshots are therefore not just a storage feature; they are part of the workload lifecycle, DR strategy, replication model, and operational recovery process. In Nutanix, this integration allows VM-centric data protection and efficient recovery workflows, but it also means support teams must understand the interaction between AOS storage, Prism, hypervisor behavior, guest OS consistency, and customer recovery expectations.

---

## 9. Connection with your experience

Your Harmonic background maps well to this topic.

You already have strong experience in:

* 24/7 support operations.
* Incident management.
* SLA and MTTR.
* Monitoring and alerting.
* Escalation coordination.
* Customer communication.
* Cloud operations.
* Cross-functional troubleshooting.
* Jira / Confluence / Salesforce workflows.

You should position snapshots as another **enterprise reliability mechanism**, similar to how you would think about:

* Backup/restore validation in SaaS.
* Rollback after failed deployment.
* Incident mitigation.
* Data recovery after corruption.
* RPO/RTO management.
* Customer-facing communication during service risk.
* Post-incident RCA.

A good positioning statement for you:

> “My strength is not only knowing what a snapshot is technically, but understanding how it fits into enterprise recovery, customer communication, SLA risk, and escalation leadership. In my current role, I manage 24/7 operational incidents where the priority is to restore service, protect customer data, coordinate technical teams, and communicate clearly. I would apply the same discipline to Nutanix snapshot and DR escalations.”

This frames you as a **technical escalation manager**, not as someone pretending to be a senior AOS storage engineer.

---

## 10. Minimum I need to memorize

Memorize these points cold.

1. A snapshot is a **point-in-time recovery point**.
2. A snapshot is **not the same as a backup**.
3. Nutanix snapshots support **restore, clone, replication, and DR**.
4. AOS uses efficient snapshot mechanisms such as **redirect-on-write**.
5. Too many snapshots can affect **capacity, performance, replication, and operations**.
6. Snapshots may be **crash-consistent** or **application-consistent**.
7. Windows application-consistent snapshots may involve **VSS**.
8. **Protection Domains** and **Protection Policies** define what is protected, when, and how long.
9. In Prism Central, snapshots may appear as **Recovery Points**.
10. Snapshot schedule affects **RPO**.
11. Restore execution affects **RTO**.
12. Escalations often involve **restore failure, replication lag, capacity pressure, app inconsistency, or third-party backup tools**.
13. Never recommend deleting snapshots without checking **retention, compliance, and recovery risk**.
14. As a manager, your role is to coordinate, prioritize, communicate, and escalate correctly.

---

## 11. Advanced / optional level

You do **not** need to master these deeply before the interview, but you should recognize them.

* Detailed AOS metadata layout.
* Stargate/Cerebro internals.
* Snapshot metadata garbage collection.
* Exact CLI/API syntax for every snapshot operation.
* Deep VMware snapshot versus Nutanix snapshot interaction.
* Changed block tracking internals.
* Multi-site DR architecture.
* Metro Availability specifics.
* NearSync / synchronous / asynchronous replication details.
* Advanced Volume Group consistency.
* Database-specific quiescing behavior.
* Snapshot API integration with backup vendors.
* Performance impact modeling for high-change-rate workloads.
* Snapshot chain consolidation internals.
* Low-level CVM log analysis.

A safe interview phrase:

> “I would not claim to be the deepest AOS storage engineer in the room, but I understand the operational model: snapshots are point-in-time recovery objects tied to AOS storage, Prism protection workflows, retention, replication, consistency, and capacity. In an escalation, I know what questions to ask, what risks to manage, and when to bring in storage/SRE/engineering specialists.”

That is exactly the positioning you want.

---

## 12. Final checklist

Before the interview, make sure you can explain:

* [ ] What a snapshot is.
* [ ] Why snapshot ≠ backup.
* [ ] How snapshots support restore, clone, replication, and DR.
* [ ] What redirect-on-write means at a high level.
* [ ] Why snapshots consume capacity over time.
* [ ] Why high-change-rate VMs matter.
* [ ] What crash-consistent means.
* [ ] What application-consistent means.
* [ ] What VSS does for Windows workloads.
* [ ] What Protection Domains are.
* [ ] What Protection Policies / Recovery Points are.
* [ ] How snapshots affect RPO and RTO.
* [ ] How to triage failed restore.
* [ ] How to triage replication lag.
* [ ] How to triage snapshot-driven capacity pressure.
* [ ] How to communicate during a customer escalation.
* [ ] How to connect this to your incident management background.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword  | Meaning                                                                                                                  |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| AHV                       | Acropolis Hypervisor; Nutanix’s native hypervisor.                                                                       |
| AOS                       | Acropolis Operating System; Nutanix software layer providing storage, virtualization integration, and platform services. |
| API                       | Application Programming Interface; used by Prism, automation, and backup integrations.                                   |
| App-consistent snapshot   | Snapshot coordinated with the guest/application to improve recoverability.                                               |
| Backup                    | Independent recoverable data copy with retention and recovery objectives.                                                |
| Backup integration        | Third-party or native tool using Nutanix snapshots/APIs for protection workflows.                                        |
| CBT                       | Changed Block Tracking; tracking changed data for backup/replication efficiency.                                         |
| Clone                     | New object created from an existing snapshot or source object.                                                           |
| Consistency group         | Group of entities snapshotted together for consistent recovery.                                                          |
| Crash-consistent snapshot | Snapshot equivalent to abrupt power-loss state.                                                                          |
| CVM                       | Controller VM; Nutanix VM that runs core platform/storage services.                                                      |
| Data Protection           | Nutanix workflows for snapshots, replication, restore, and DR.                                                           |
| DR                        | Disaster Recovery; recovery of services after major failure.                                                             |
| DSF                       | Distributed Storage Fabric; Nutanix distributed storage layer.                                                           |
| ESXi                      | VMware hypervisor often used with Nutanix clusters.                                                                      |
| Garbage collection        | Cleanup of unused data/metadata after deletion or lifecycle operations.                                                  |
| Guest OS                  | Operating system running inside a VM.                                                                                    |
| HCI                       | Hyperconverged Infrastructure; integrated compute, storage, and virtualization.                                          |
| High-change-rate VM       | VM generating many writes, increasing snapshot/replication load.                                                         |
| Hypervisor                | Virtualization layer running VMs, such as AHV or ESXi.                                                                   |
| KPI                       | Key Performance Indicator; metric used to track operational performance.                                                 |
| LUN                       | Logical Unit Number; traditional block storage unit.                                                                     |
| Metadata                  | Data describing storage objects, references, snapshots, and layout.                                                      |
| Metro Availability        | Nutanix availability/DR feature for stretched or highly available environments.                                          |
| MTTR                      | Mean Time To Repair/Restore; average time to resolve or recover.                                                         |
| Orphaned snapshot         | Snapshot left behind unexpectedly, often by failed jobs or integrations.                                                 |
| PD                        | Protection Domain; Prism Element grouping of protected entities and schedules.                                           |
| Prism Central             | Centralized Nutanix management plane.                                                                                    |
| Prism Element             | Cluster-level Nutanix management interface.                                                                              |
| Protection Domain         | Group of VMs/entities with snapshot, retention, and replication policy.                                                  |
| Protection Policy         | Prism Central policy defining protection and recovery behavior.                                                          |
| Quiescing                 | Pausing/flushing application writes to prepare for consistent snapshot.                                                  |
| Recovery Point            | Prism Central term for snapshot-based recoverable point.                                                                 |
| Redirect-on-write         | Snapshot method where new writes go elsewhere while old state is preserved.                                              |
| Replication               | Copying snapshot/recovery data to another cluster or site.                                                               |
| Restore                   | Returning a workload or data object to a previous snapshot state.                                                        |
| Retention                 | Rules defining how long snapshots/recovery points are kept.                                                              |
| RPO                       | Recovery Point Objective; acceptable amount of data loss.                                                                |
| RTO                       | Recovery Time Objective; acceptable time to restore service.                                                             |
| SLA                       | Service Level Agreement; agreed service performance or response target.                                                  |
| Snapshot                  | Point-in-time representation of a VM, vDisk, Volume Group, or file.                                                      |
| Snapshot chain            | Sequence of snapshot references and changed data over time.                                                              |
| Snapshot schedule         | Frequency at which snapshots are created.                                                                                |
| Snapshot sprawl           | Excessive unmanaged snapshot growth.                                                                                     |
| SRE                       | Site Reliability Engineer; role focused on reliability, automation, and operations.                                      |
| Stargate                  | Nutanix service associated with storage I/O operations.                                                                  |
| Storage container         | Logical storage construct in Nutanix.                                                                                    |
| vDisk                     | Virtual disk attached to a VM.                                                                                           |
| VM                        | Virtual Machine.                                                                                                         |
| VM-centric snapshot       | Snapshot protection focused on VM-level recovery rather than only LUN-level storage.                                     |
| Volume Group              | Nutanix block storage construct that can be attached to VMs.                                                             |
| VSS                       | Volume Shadow Copy Service; Microsoft mechanism for Windows application-consistent snapshots.                            |

[1]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Element-Data-Protection-Guide-v5_15%3Awc-cluster-snapshots-wc-r.html&utm_source=chatgpt.com "PD-Based DR 5.15 - Snapshots - portal.nutanix.com"
[2]: https://www.nutanix.com/products/nutanix-cloud-infrastructure/distributed-storage?utm_source=chatgpt.com "A Modern Storage Fabric - Nutanix"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2096-Nutanix-Core-Performance%3Ahigh-performance-snapshots-and-clones.html&utm_source=chatgpt.com "High-Performance Snapshots and Clones - Nutanix"
[4]: https://www.nutanix.dev/2022/09/01/nutanix-benefit-4-granular-and-efficient-snapshots/?utm_source=chatgpt.com "Nutanix Benefit 4: Granular and Efficient Snapshots"
[5]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Element-Data-Protection-Guide-v6_8%3Awc-dr-application-consistent-snapshots-wc-r.html&utm_source=chatgpt.com "PD-Based DR 6.8 - Conditions for Application-consistent Snapshots"
[6]: https://nutanix-technologybootcamp.readthedocs.io/en/latest/lab_data_protection/lab_data_protection.html?utm_source=chatgpt.com "Lab - Data Protection - Read the Docs"
[7]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_7%3Awc-cluster-vm-snapshot-ahv-c.html&utm_source=chatgpt.com "Prism 6.7 - Virtual Machine Snapshots - Nutanix"
