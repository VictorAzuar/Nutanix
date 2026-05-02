# AHV / Virtualization — VM snapshots

## 1. Short definition

A **VM snapshot** is a point-in-time capture of a virtual machine’s disk state, and sometimes selected VM metadata, used to restore, clone, replicate, or recover a VM after a change, failure, corruption, or operational mistake. In Nutanix AHV, snapshots are managed from Prism Element / Prism Central workflows and are part of the broader AOS data protection model. Nutanix documentation distinguishes simple AHV VM snapshots from **Recovery Points** created through Prism Central protection policies. ([Nutanix][1])

For interviews, the key phrase is:

> “A snapshot is not a backup by itself. It is a short-term point-in-time recovery mechanism that must be governed by retention, capacity, consistency, and recovery objectives.”

---

## 2. Clear explanation

In classic virtualization, a VM snapshot preserves the VM at a given point in time. After the snapshot is taken, new writes are tracked separately so the platform can reconstruct either the current VM state or the previous snapshot state.

In Nutanix, the snapshot mechanism is tightly integrated with **AOS Distributed Storage Fabric / DSF**, rather than being only a hypervisor-side file operation. Nutanix describes its snapshot architecture as **VM-centric**, granular, and based on a **redirect-on-write** model. That matters because it is designed to avoid some of the heavy penalties traditionally associated with copy-on-write snapshot chains. ([Nutanix][2])

A simplified flow:

1. A snapshot is taken for a VM or protected entity.
2. The original point-in-time blocks are preserved.
3. New writes go elsewhere, while metadata tracks which block version belongs to which point in time.
4. The snapshot can later be used to restore, clone, replicate, or create a recovery point.

However, snapshots still have operational consequences. More snapshots mean more changed data and more metadata to track, which can affect performance and capacity if not governed properly. Nutanix performance guidance explicitly notes that more snapshots and more changed data can impose a substantial performance penalty. ([Nutanix][3])

A critical distinction for AHV:

* **AHV VM snapshots taken directly from the VM menu are not application-consistent by default.**
* **Application-consistent snapshots** are available through data protection / recovery workflows using supported guest mechanisms, such as VSS for Windows when properly configured. Nutanix documentation notes that AHV VM snapshots from the VM menu are not application-consistent, while application-consistent snapshots are available through Protection Domain snapshots and Recovery Points in Prism Central. ([Nutanix][4])

That distinction is extremely interview-relevant.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, VM snapshots matter because they sit at the intersection of virtualization, storage, backup, DR, customer risk, performance, and escalation management.

You do not need to sound like the deepest storage engineer in the room. You do need to show that you understand the operational risk:

* Snapshots are often created before upgrades, patching, migrations, risky application changes, or maintenance windows.
* Customers may assume snapshots are backups, which can create dangerous recovery expectations.
* Snapshot sprawl can consume storage, increase metadata overhead, and affect VM or cluster performance.
* Snapshot consistency determines whether the recovered application is merely crash-consistent or actually application-consistent.
* Snapshot failures can block backup jobs, replication, DR compliance, or change windows.
* Snapshot-related incidents often involve multiple teams: support, SRE, storage, hypervisor, backup vendor, application owner, and customer leadership.

For Nutanix, snapshots are also part of the value proposition of the platform: Nutanix AOS provides integrated storage-level snapshot and data protection capabilities, including local and replicated protection models. Nutanix data protection documentation describes snapshots as point-in-time states of entities such as VMs and Volume Groups, used for restoration and replication. ([Nutanix][5])

As a support manager, your value is not only knowing the command. Your value is knowing how to control the incident:

> “What is the recovery objective? What changed? Is the snapshot crash-consistent or application-consistent? Is the customer trying to restore, clone, replicate, or delete? What is the capacity and performance impact? Which team owns the next action?”

---

## 4. Key concepts

### Snapshot vs backup

A snapshot is usually local, fast, and point-in-time. A backup is normally a more durable recovery copy, often stored independently, retained longer, and governed by backup policy.

A good interview answer:

> “I treat snapshots as operational recovery points, not as a full backup strategy. They are useful before change windows or for short-term rollback, but they need retention, monitoring, and integration with backup or DR policies.”

### Crash-consistent vs application-consistent

A **crash-consistent snapshot** captures the VM disk state as if the machine suddenly lost power. It may be enough for stateless or resilient applications, but databases or transactional systems may need log replay or integrity checks.

An **application-consistent snapshot** coordinates with the guest OS or application so in-flight writes are flushed and the application is in a cleaner recovery state. Nutanix documentation refers to crash-consistent and application-consistent snapshot types in data protection contexts. ([Nutanix][5])

For Windows workloads, application consistency often involves **VSS**. Nutanix documentation references the Nutanix native in-guest **VmQuiesced Snapshot Service / VSS agent** for VMs that support VSS. ([Nutanix][4])

### AHV snapshot vs Prism Central Recovery Point

A simple AHV snapshot is typically a VM-level point-in-time snapshot managed in Prism Element / AHV workflows. A **Recovery Point** is the term used in Prism Central protection policy workflows. Nutanix documentation states that AOS generates recovery points when protecting a VM with a protection policy, and community guidance summarizes the terminology difference as “Snapshot” being a Prism Element / AHV term and “Recovery Point” being associated with Prism Central. ([Nutanix][1])

Interview wording:

> “I would clarify whether we are talking about an ad hoc AHV VM snapshot or a policy-driven Recovery Point, because the operational behavior, consistency, retention, and DR expectations may differ.”

### Redirect-on-write

Nutanix describes its snapshot functionality as using a **redirect-on-write** algorithm. Instead of copying old blocks before overwriting them, new writes are redirected and metadata preserves the previous point-in-time view. This supports efficient snapshots and clones, although snapshot accumulation still needs governance. ([Nutanix][2])

### Snapshot chain / sprawl

Snapshot sprawl happens when snapshots are created and not removed according to policy. Even efficient snapshot architectures can suffer when customers keep too many snapshots, keep them too long, or generate heavy write churn after snapshot creation. Nutanix community guidance recommends keeping snapshot trees short and snapshots fresh, and using proper long-term backup solutions for long-term retention. ([Nutanix Community][6])

### VM disks and Volume Groups

Nutanix Volume Groups are collections of vDisks that can be attached to VMs. They can be used to separate data disks from boot disks and support protection policies that target only the data vDisks. ([Nutanix][7])

This matters because a customer escalation may not involve only a basic VM disk. It may involve:

* boot disk snapshot
* application data disk
* Volume Group
* shared-disk workload
* replication policy
* backup vendor integration

### API and automation

Nutanix exposes modern v4 APIs through Prism Central, including namespaces for Virtual Machine Management, Data Protection, Volumes, Monitoring, and related platform operations. Nutanix also states that legacy APIs v0.8, v1, v2, and v3 are planned for deprecation starting with the AOS and PC upgrade release planned for Q4 CY2026, and recommends migration to v4 APIs and SDKs. ([Nutanix][8])

For a manager interview, you do not need to code the API live. But you should understand that enterprise customers may automate snapshot creation, reporting, cleanup, and compliance checks.

---

## 5. How it appears in a real escalation

### Scenario A — Snapshot sprawl causing capacity pressure

A customer reports that a Nutanix cluster is approaching critical storage utilization. Investigation shows many old VM snapshots created before previous change windows and never deleted.

Support manager response:

* Confirm business impact: production risk, capacity threshold, affected workloads.
* Identify snapshot age, owner, VM criticality, and retention reason.
* Separate safe cleanup candidates from snapshots tied to backup / DR workflows.
* Coordinate with customer application owners before deletion.
* Track capacity recovery and cluster health after cleanup.
* Drive prevention: retention policy, reporting, change-window cleanup checklist.

What you should say in an interview:

> “I would avoid telling the customer to delete everything immediately. I would first identify snapshot ownership, age, policy association, and recovery dependency. Then I would coordinate a controlled cleanup plan and ensure we prevent recurrence with retention and monitoring.”

### Scenario B — Application restore fails after snapshot rollback

A customer restores a database VM from an AHV snapshot and the OS boots, but the application is inconsistent.

Likely cause:

* Snapshot was crash-consistent, not application-consistent.
* Application writes were in flight.
* Customer assumed snapshot was equivalent to an app-aware backup.

Your escalation posture:

* Clarify snapshot type and creation method.
* Involve application/database owners.
* Validate logs and application recovery procedures.
* Check whether VSS / guest quiescing / protection policy was configured.
* Set expectations: platform snapshot may restore disk state, but application recovery may require database-level procedures.

### Scenario C — Backup job fails because snapshot cannot be created or removed

A backup tool uses Nutanix snapshots as part of its workflow. Backups start failing because snapshots are stuck, API calls fail, or cleanup does not complete.

Triage:

* Which backup product?
* Is the failure on create, quiesce, replicate, or delete?
* Is the VM powered on?
* Are guest tools / VSS healthy?
* Is Prism Central / Prism Element reachable?
* Are there API authentication or version compatibility issues?
* Is there an existing snapshot task stuck?
* Are cluster services healthy?

Escalation management angle:

> “I would define the failing operation precisely. ‘Backup failed’ is too broad. I want to know whether the failure is snapshot creation, guest quiescing, data transfer, catalog update, or snapshot cleanup.”

### Scenario D — Performance degradation after snapshots

A high-write VM starts showing latency after multiple snapshots or after a long-lived snapshot during a maintenance window.

Triage:

* Is latency read, write, or both?
* When were snapshots created?
* How many snapshots exist?
* What changed after the snapshot?
* Is the VM high-churn, such as database, logging, VDI, Kafka, Elasticsearch?
* Is the cluster under capacity, CPU, memory, network, or storage pressure?
* Are backup or replication jobs running?

Support-manager framing:

> “I would correlate performance metrics with snapshot lifecycle events and workload write rate. I would not assume snapshots are the only cause, but they are a key variable when latency begins after a maintenance, backup, or DR event.”

### Scenario E — Customer asks for a guaranteed rollback before upgrade

Before a major application or OS patch, the customer wants to take a snapshot and asks whether rollback is guaranteed.

Good answer:

> “A snapshot is a useful rollback control, but I would validate whether the application requires application consistency, whether external dependencies are involved, and whether the snapshot is part of an approved backup/DR plan. For critical production upgrades, I would not rely only on an ad hoc VM snapshot.”

---

## 6. Triage questions I should ask

### Business impact

1. Which application or service is affected?
2. Is this production, pre-production, or test?
3. What is the current customer impact?
4. Is there an active outage, degraded performance, failed backup, failed restore, or blocked change?
5. What are the RTO and RPO expectations?

### Snapshot context

6. Was the snapshot created manually, by Prism Central policy, by a protection domain, or by a backup tool?
7. Is it an AHV VM snapshot or a Prism Central Recovery Point?
8. When was it created?
9. How many snapshots or recovery points exist for the VM?
10. Is this snapshot local only, replicated, or part of a DR workflow?

### Consistency

11. Was the snapshot crash-consistent or application-consistent?
12. Was guest quiescing enabled?
13. For Windows, was VSS involved and healthy?
14. For databases, were application-level backup or freeze/thaw procedures used?
15. Was the VM under heavy write load during snapshot creation?

### Platform health

16. Are Prism Element and Prism Central healthy?
17. Are cluster services healthy?
18. Is storage capacity within safe thresholds?
19. Is there abnormal latency, packet loss, CVM resource pressure, or node degradation?
20. Are there failed or stuck tasks?

### Restore / rollback

21. Is the customer restoring the original VM or cloning to a new VM?
22. Do they need file-level restore, full VM restore, or application-level recovery?
23. Are external dependencies involved, such as databases, identity services, DNS, load balancers, or external storage?
24. Has the restore been tested before?
25. What is the rollback decision deadline?

### Ownership and communication

26. Who owns the application?
27. Who owns the backup policy?
28. Who can approve snapshot deletion?
29. Who is the customer incident commander?
30. What communication cadence is expected?

---

## 7. Likely interview questions

1. What is a VM snapshot?
2. How is a snapshot different from a backup?
3. How do snapshots work in Nutanix AHV?
4. What is the difference between crash-consistent and application-consistent snapshots?
5. Are AHV VM snapshots application-consistent by default?
6. What is a Recovery Point in Prism Central?
7. How would you troubleshoot a failed snapshot?
8. How would you handle snapshot sprawl?
9. Can snapshots affect performance?
10. What would you ask a customer before deleting old snapshots?
11. How would you manage an escalation where a customer cannot restore from a snapshot?
12. How do snapshots relate to DR and replication?
13. What metrics would you review in a snapshot-related performance case?
14. How would you explain snapshots to a non-technical customer executive?
15. How would you lead a bridge call involving Nutanix SREs, backup vendor, and customer application owners?
16. What is redirect-on-write?
17. What are the risks of using snapshots before a major change?
18. How would you prevent snapshot incidents from recurring?
19. What is the role of automation in snapshot lifecycle management?
20. How would your SaaS/cloud support experience transfer to snapshot escalations?

---

## 8. Model answers in English

### Q1. What is a VM snapshot?

> A VM snapshot is a point-in-time capture of a virtual machine’s disk state, and sometimes associated VM metadata, that allows an administrator to restore, clone, or recover a VM to a previous state. In an enterprise support context, I treat snapshots as short-term operational recovery points, not as a replacement for backups. The important questions are always: what consistency level was used, how long the snapshot will be retained, what capacity impact it has, and whether it is part of a broader backup or DR policy.

### Q2. How is a snapshot different from a backup?

> A snapshot is usually a fast, local point-in-time mechanism, often stored on the same platform and intended for short-term rollback or recovery. A backup is normally an independent, governed copy with retention, cataloging, off-platform storage, and restore validation. In customer escalations, I would be careful with expectations: a snapshot can help recover quickly, but for compliance, ransomware resilience, or long-term recovery, the customer needs a proper backup and DR strategy.

### Q3. Are AHV VM snapshots application-consistent by default?

> My understanding is that ad hoc AHV VM snapshots taken directly from the VM workflow are not application-consistent by default. For application consistency, especially with Windows workloads, you need supported guest quiescing mechanisms such as VSS and the appropriate Nutanix data protection or recovery workflow. In an escalation, I would clarify how the snapshot was created before making any statement about recoverability.

### Q4. What is the difference between crash-consistent and application-consistent?

> A crash-consistent snapshot captures the disk as if the VM had suddenly lost power. The OS or application may recover, but transactional applications can require log replay or repair. An application-consistent snapshot coordinates with the guest OS or application so pending writes are flushed and the application is in a cleaner recovery state. For databases or business-critical applications, this distinction is essential.

### Q5. How would you troubleshoot a failed snapshot?

> I would first scope the failure: is it failing to create, quiesce, replicate, restore, or delete the snapshot? Then I would check whether the snapshot was manual, policy-driven, or triggered by a backup tool. I would review Prism tasks, VM state, guest tools or VSS health, cluster health, capacity, and any API or authentication errors. From a management perspective, I would also define customer impact, assign owners, set a communication cadence, and separate workaround, recovery, and root-cause tracks.

### Q6. Can snapshots affect performance?

> Yes. Even efficient snapshot implementations can have impact if snapshots are excessive, long-lived, or attached to high-churn workloads. Nutanix uses storage-aware redirect-on-write snapshots, which is designed for efficiency, but snapshot sprawl and heavy changed data still require governance. In a performance escalation, I would correlate latency with snapshot creation time, backup windows, replication activity, write rate, and cluster resource metrics.

### Q7. How would you handle snapshot sprawl?

> I would not start by blindly deleting snapshots. I would inventory snapshots by VM, age, owner, creation source, policy association, and business criticality. Then I would identify safe cleanup candidates, get customer approval where needed, and monitor capacity and performance during cleanup. After stabilization, I would drive prevention: retention policies, reporting, change-management cleanup tasks, and backup policy review.

### Q8. How would you explain snapshot risk to an executive customer?

> I would say: “Snapshots give us fast rollback options, but they are not a complete backup strategy. The risk is that old or unmanaged snapshots can consume capacity, affect performance, or create false confidence about recoverability. For this incident, we are validating which snapshots are safe to use or remove, and we will align the cleanup with your recovery objectives and application owners.”

### Q9. What would you do if a customer restored a VM but the application is corrupted?

> I would first avoid blaming the customer or the platform. I would confirm the snapshot type, whether it was crash-consistent or application-consistent, whether guest quiescing was enabled, and whether the application requires its own recovery procedure. I would involve the application or database owner, review logs, and determine whether another recovery point, backup, or application-level restore is available. I would manage the escalation with clear workstreams: platform validation, application recovery, and customer communication.

### Q10. How does this connect to SRE and support leadership?

> Snapshot issues are classic SRE/support problems because they combine technical depth with operational discipline. The manager must ensure correct triage, clear ownership, customer communication, risk management, and prevention. My role would be to make sure the team does not only fix the immediate snapshot problem but also improves monitoring, runbooks, retention, automation, and customer guidance to reduce recurrence.

---

## 9. Connection with my experience

Your background maps well to this topic if you present it correctly.

You already understand:

* **24/7 operations** → snapshot incidents often happen during backup windows, maintenance windows, or emergency rollbacks.
* **Incident management** → snapshot restore failures can become Sev1 or Sev2 events.
* **SLA / MTTR** → fast recovery depends on knowing whether a snapshot is usable, consistent, and recent.
* **KPIs** → relevant indicators include backup success rate, restore success rate, snapshot age, snapshot count, capacity consumed, failed tasks, MTTR, and recurrence rate.
* **Monitoring** → Grafana/Kibana experience transfers well to correlating snapshot events with latency, capacity, and application errors.
* **Cloud operations** → AWS/Azure/GCP also have snapshots, images, disks, consistency concerns, retention policies, and lifecycle automation.
* **Jira / Confluence** → useful for escalation tracking, postmortems, runbooks, known-error articles, and change-window checklists.
* **Salesforce / customer management** → useful for case ownership, escalation notes, customer updates, and executive communication.

How to position yourself:

> “I am not positioning myself as the deepest AHV storage engineer. I am positioning myself as a technical escalation manager who can understand the architecture, ask the right diagnostic questions, coordinate SREs and vendors, communicate clearly with enterprise customers, and drive operational prevention.”

That is the right positioning for this role.

---

## 10. Minimum I need to memorize

Memorize these points cold:

1. A snapshot is a **point-in-time recovery mechanism**, not a full backup.
2. AHV ad hoc VM snapshots are **not application-consistent by default**. Application consistency requires supported data protection / guest quiescing workflows. ([Nutanix][4])
3. Nutanix uses **AOS / DSF storage-integrated snapshots**, described as VM-centric and redirect-on-write. ([Nutanix][2])
4. Snapshot sprawl can create **capacity, metadata, operational, and performance risk**.
5. Always ask: **manual snapshot, protection policy, recovery point, or backup-tool snapshot?**
6. Always clarify **crash-consistent vs application-consistent**.
7. In escalations, define whether the customer needs **restore, clone, delete, replicate, or validate**.
8. Do not delete snapshots without checking **owner, age, policy, business dependency, and approval**.
9. Tie snapshot incidents to **RTO, RPO, SLA, MTTR, customer impact, and change management**.
10. As a manager, your role is **triage, coordination, communication, risk control, and prevention**.

---

## 11. Advanced / optional level

You can leave these as advanced unless the SRE panel goes deeper:

* Internal AOS metadata mechanisms behind snapshot tracking.
* Exact CLI/API commands for listing, creating, deleting, or restoring snapshots.
* Deep comparison of VMware snapshot chains vs Nutanix DSF snapshots.
* Backup vendor integrations and CBT-like behavior.
* Volume Group snapshot behavior for shared-disk or application-specific designs.
* NearSync / Async DR snapshot scheduling and replication internals.
* Cerebro, Stargate, Curator, and other internal AOS services involved in data protection.
* Detailed performance analysis of snapshot-heavy workloads.
* API migration specifics from Nutanix legacy APIs to v4 APIs.
* Cross-hypervisor DR using snapshots between ESXi and AHV environments.

Still, know enough vocabulary to not freeze if asked.

Good answer when you do not know the low-level detail:

> “I would need to validate the exact implementation detail with the relevant Nutanix documentation or an SRE, but operationally I would first determine the snapshot type, consistency model, policy source, capacity impact, and customer recovery objective.”

That is a strong manager-level answer.

---

## 12. Final checklist

Before your interview, you should be able to explain:

* [ ] What a VM snapshot is.
* [ ] Why a snapshot is not a backup.
* [ ] Crash-consistent vs application-consistent.
* [ ] Why AHV ad hoc VM snapshots are not application-consistent by default.
* [ ] What VSS / guest quiescing does.
* [ ] What Prism Element and Prism Central do in snapshot workflows.
* [ ] Snapshot vs Recovery Point.
* [ ] Snapshot sprawl risk.
* [ ] Capacity and performance impact.
* [ ] How snapshots relate to backup and DR.
* [ ] How to triage failed snapshot creation.
* [ ] How to triage failed restore.
* [ ] How to handle snapshot deletion safely.
* [ ] How to communicate snapshot risk to a customer executive.
* [ ] How to coordinate SRE, backup vendor, customer app owner, and support team.
* [ ] How your incident management background transfers directly.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword        | Meaning                                                                                                                         |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| ACLI                            | Acropolis CLI; Nutanix command-line interface for AHV / Acropolis operations.                                                   |
| AHV                             | Acropolis Hypervisor; Nutanix’s native virtualization hypervisor.                                                               |
| AOS                             | Acropolis Operating System; Nutanix core software platform providing storage, virtualization integration, and cluster services. |
| API                             | Application Programming Interface; used to automate platform operations such as VM or data protection workflows.                |
| Application-consistent snapshot | Snapshot coordinated with the guest OS or application so data is in a cleaner recoverable state.                                |
| Backup                          | Independent protected copy used for longer-term recovery, compliance, or disaster recovery.                                     |
| Backup window                   | Scheduled period when backup or snapshot jobs run.                                                                              |
| CBT                             | Changed Block Tracking; mechanism used by some platforms/tools to identify changed disk blocks for incremental backup.          |
| Change window                   | Approved time period for planned maintenance or production changes.                                                             |
| Clone                           | New VM or disk copy created from an existing VM, disk, snapshot, or recovery point.                                             |
| Consistency                     | Degree to which captured data can be safely recovered by the OS or application.                                                 |
| Crash-consistent snapshot       | Snapshot equivalent to the state after sudden power loss; may require application recovery.                                     |
| CVM                             | Controller VM; Nutanix VM running storage and cluster services on each node.                                                    |
| Data Protection                 | Nutanix features and policies for snapshots, replication, restore, and recovery.                                                |
| DR                              | Disaster Recovery; recovery strategy for major failures or site loss.                                                           |
| DSF                             | Distributed Storage Fabric; Nutanix software-defined storage layer across cluster nodes.                                        |
| Escalation                      | Formal process to involve higher support levels, engineering, SRE, vendors, or leadership.                                      |
| Guest quiescing                 | Pausing or flushing guest OS/application I/O before taking a snapshot.                                                          |
| HCI                             | Hyperconverged Infrastructure; compute, storage, virtualization, and networking integrated in one platform.                     |
| I/O                             | Input/Output; read and write operations to storage or network.                                                                  |
| KPI                             | Key Performance Indicator; metric used to measure operational performance.                                                      |
| Latency                         | Time taken to complete an operation, often critical in storage and application performance.                                     |
| MTTR                            | Mean Time To Restore/Resolve; average time to restore service or resolve an incident.                                           |
| Prism Central                   | Nutanix centralized management plane for multiple clusters and policy-driven operations.                                        |
| Prism Element                   | Nutanix cluster-level management interface.                                                                                     |
| Protection Domain               | Nutanix data protection construct used to group protected entities and manage snapshots/replication.                            |
| Protection Policy               | Policy-driven protection model, usually managed through Prism Central.                                                          |
| Recovery Point                  | Prism Central term for a protected point-in-time VM state created through protection workflows.                                 |
| Redirect-on-write               | Snapshot method where new writes are redirected while preserving the previous point-in-time data.                               |
| Replication                     | Copying protected data or snapshots to another cluster or site.                                                                 |
| Restore                         | Returning data, disk, VM, file, or application to a previous point in time.                                                     |
| Retention                       | Policy defining how long snapshots or backups are kept.                                                                         |
| RPO                             | Recovery Point Objective; maximum acceptable data loss measured in time.                                                        |
| RTO                             | Recovery Time Objective; maximum acceptable time to restore service.                                                            |
| Runbook                         | Documented operational procedure for incident handling or recurring tasks.                                                      |
| SLA                             | Service Level Agreement; committed service performance or response target.                                                      |
| Snapshot                        | Point-in-time capture of VM, disk, or protected entity state.                                                                   |
| Snapshot chain                  | Logical relationship between snapshots and changed data over time.                                                              |
| Snapshot cleanup                | Process of deleting or consolidating unneeded snapshots safely.                                                                 |
| Snapshot sprawl                 | Excessive, unmanaged, or long-lived snapshots creating risk.                                                                    |
| SRE                             | Site Reliability Engineering; discipline focused on reliability, automation, incident response, and operational excellence.     |
| Stargate                        | Nutanix service commonly associated with storage I/O handling in AOS architecture.                                              |
| Triage                          | Initial structured investigation to classify impact, scope, cause, and next action.                                             |
| VDisk                           | Virtual disk attached to a VM.                                                                                                  |
| VM                              | Virtual Machine; software-defined compute instance running an operating system and applications.                                |
| Volume Group                    | Nutanix group of vDisks attachable to VMs, often used for application data disks.                                               |
| VSS                             | Volume Shadow Copy Service; Microsoft Windows mechanism used for application-consistent snapshots.                              |

[1]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v10_3%3Awc-cluster-vm-snapshot-ahv-c.html?utm_source=chatgpt.com "AHV 10.3 - Virtual Machine Snapshots - portal.nutanix.com"
[2]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2005-Data-Protection%3Anutanix-snapshots.html&utm_source=chatgpt.com "Nutanix Snapshots"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2096-Nutanix-Core-Performance%3Ahigh-performance-snapshots-and-clones.html&utm_source=chatgpt.com "High-Performance Snapshots and Clones - Nutanix"
[4]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_8%3Awc-vm-manage-acropolis-wc-t.html&utm_source=chatgpt.com "AHV 6.8 - Managing a VM (AHV) - portal.nutanix.com"
[5]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Element-Data-Protection-Guide-v5_15%3Awc-cluster-snapshots-wc-r.html&utm_source=chatgpt.com "PD-Based DR 5.15 - Snapshots - portal.nutanix.com"
[6]: https://next.nutanix.com/how-it-works-22/snapshots-so-simple-and-yet-so-complicated-faq-38007?utm_source=chatgpt.com "Snapshots - so simple and yet so complicated - FAQ - Nutanix"
[7]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2105-Linux-on-AHV%3Anutanix-volume-groups.html&utm_source=chatgpt.com "Nutanix Volume Groups"
[8]: https://www.nutanix.dev/api-reference-v4/ "API Reference v4 Introduction – Nutanix.dev"
