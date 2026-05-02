# AHV / Virtualization — Guest Tools

## 1. Short definition

**Nutanix Guest Tools (NGT)** is a software package installed inside a guest VM running on Nutanix AHV. It enables better integration between the Nutanix platform and the operating system inside the VM, especially for advanced VM management operations such as **graceful shutdown/restart, file-level restore, VSS-based snapshot coordination, and VM mobility workflows**. Nutanix describes NGT as software installed in Windows or Linux guest VMs to enable advanced VM management features. ([Nutanix][1])

For an interview, the simple version is:

> Guest Tools are the bridge between the hypervisor/platform and the guest operating system. They allow Nutanix to perform guest-aware operations safely, instead of treating the VM as a black box.

---

## 2. Clear explanation

In virtualization, the hypervisor can control the **VM container**: power state, virtual disks, virtual NICs, snapshots, placement, migration, and resource allocation. But the hypervisor does not automatically understand what is happening **inside** the guest OS: running applications, file systems, pending I/O, OS services, open transactions, or application state.

That is where **Guest Tools** come in.

On AHV, **Nutanix Guest Tools** are installed inside the VM so that Nutanix can coordinate certain operations with the guest operating system. Typical use cases include:

* Graceful VM shutdown and restart.
* Guest-aware snapshot workflows.
* Self-Service Restore / file-level restore.
* VM mobility and migration-related operations.
* Installation or upgrade of Nutanix VirtIO drivers in some Windows scenarios.

Nutanix documentation states that NGT enables **Self-Service Restore**, **Nutanix VSS**, and **Nutanix VM Mobility** features from Prism Central. ([Nutanix][2])

A key distinction:

**VirtIO drivers** are not the same thing as NGT.

* **VirtIO drivers** provide paravirtualized device drivers for AHV, especially storage and network devices.
* **NGT** provides guest integration and management features.

For Windows VMs on AHV, Nutanix says Windows requires the correct VirtIO network and disk drivers during VM creation, while NGT is optional unless you need the NGT-specific features. ([Nutanix][3])

So in an interview, avoid saying “NGT is required for every VM to run.” More accurate:

> For basic VM operation, especially on Windows, the critical component is the correct VirtIO driver set. NGT is optional from a pure boot/runtime perspective, but important for advanced operational features such as graceful power operations, guest-aware recovery, and self-service restore.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Worldwide Support Manager**, Guest Tools matter because they sit at the intersection of:

* Virtualization.
* Guest operating systems.
* Data protection.
* Backup/recovery.
* Customer operations.
* Escalation management.
* Change management.
* Supportability.

In enterprise support, incidents often happen not because the platform is completely down, but because an expected management operation does not work:

* “We cannot gracefully shut down VMs during maintenance.”
* “Self-service file restore does not work.”
* “Snapshots are crash-consistent instead of application-aware.”
* “A Windows VM has poor network or disk performance after migration.”
* “VM mobility from ESXi to AHV failed.”
* “The customer says Nutanix broke the application after a snapshot.”

As a support manager, you do not need to debug every driver or service personally, but you do need to understand:

1. **Where the responsibility boundary is**: Nutanix platform, AHV, NGT, VirtIO, guest OS, application, backup product, customer configuration.
2. **What evidence to collect**: VM OS, NGT status, driver version, AHV/AOS version, Prism alerts, task history, logs, backup/snapshot workflow, recent changes.
3. **How to manage the escalation**: customer impact, workaround, ownership, communication, engineering handoff, risk of data loss, SLA priority.
4. **How to prevent recurrence**: standardize NGT deployment, compatibility checks, upgrade planning, validation after VM migrations.

A strong manager-level answer should sound like this:

> I would treat NGT as an operational dependency for guest-aware lifecycle and recovery features. In an escalation, I would first separate whether the VM runtime itself is affected, whether the issue is with guest integration, or whether the customer expects application-consistent behavior that requires additional configuration. Then I would align Support, SRE, Engineering, and the customer around impact, evidence, workaround, and next steps.

---

## 4. Key concepts

### NGT as guest integration

NGT allows AHV/Prism to interact more safely with the guest OS. Without Guest Tools, the platform can still perform many infrastructure-level actions, but some operations become less graceful or less guest-aware.

Example:

* Hard power-off: hypervisor-level action.
* Graceful shutdown: requires guest OS cooperation.

Nutanix documentation says NGT initiates and performs soft shutdown and restart operations within the VM, ensuring a safe and graceful shutdown/restart. ([Nutanix][4])

### Graceful power operations

A hypervisor can force a VM off, but that is similar to pulling the power cable from a physical server. A graceful shutdown lets the OS close services, flush buffers, and reduce the risk of corruption.

This matters during:

* Host maintenance.
* Cluster upgrades.
* Planned shutdowns.
* Customer-requested VM restarts.
* Incident recovery actions.

### Self-Service Restore / File-Level Restore

Self-Service Restore allows VM administrators to recover files from Nutanix snapshots/recovery points with less administrator intervention. Nutanix documentation says the Nutanix administrator should deploy NGT on the VM and enable the feature. ([Nutanix][5])

This is important because customers often expect “snapshot” to mean “I can recover my file.” The platform capability depends on configuration, guest tools, permissions, snapshot policy, and supported OS/application behavior.

### VSS / guest-aware snapshots

For Windows workloads, **VSS** stands for **Volume Shadow Copy Service**. It helps coordinate snapshots with the Windows OS and VSS-aware applications.

Nutanix documentation notes that after enabling Nutanix Guest Tools, the VSS snapshot feature is enabled by default for supported VMs, but it also clarifies that AHV VM snapshots are not automatically application-consistent; application-consistent snapshots are associated with Protection Domain based snapshots and Recovery Points in Prism Central. ([Nutanix][6])

This is an important interview nuance:

> Not every snapshot is application-consistent. Guest tools can help coordinate guest-aware operations, but the exact consistency level depends on the snapshot mechanism, workload, VSS support, and protection configuration.

### VirtIO drivers

VirtIO drivers are AHV paravirtualized drivers, especially important for Windows VMs. Nutanix describes VirtIO as a collection of drivers for paravirtual devices that enhance VM stability and performance on AHV. ([Nutanix][7])

The distinction matters in escalations:

* VM cannot see disk during Windows install → likely VirtIO driver issue.
* VM has poor network/disk performance → check VirtIO version/configuration.
* Graceful shutdown or SSR not working → check NGT status/services.
* File-level restore failing → check NGT + SSR configuration.
* Application consistency concern → check VSS, snapshot type, protection policy.

### Compatibility

Guest OS compatibility should be checked against Nutanix’s Compatibility Matrix, especially for supported OS versions, AHV versions, and guest features. Nutanix provides a compatibility matrix for guest operating systems and Nutanix platforms/hypervisors. ([Nutanix][8])

For a manager, this becomes a supportability question:

> Are we troubleshooting a supported configuration, an unsupported guest OS, an outdated driver/tool version, or a configuration drift issue?

---

## 5. How it appears in a real escalation

### Scenario 1 — Graceful shutdown fails during maintenance

**Customer impact:** The customer is performing planned maintenance. Several VMs do not shut down gracefully from Prism. The maintenance window is at risk.

**Likely areas:**

* NGT not installed.
* NGT installed but not running.
* Guest OS service issue.
* Communication issue between CVM/Prism and the guest agent.
* Unsupported guest OS.
* Permissions or configuration issue.
* Customer expectation mismatch: they expect guest-aware operation, but tools were never deployed.

**Manager-level handling:**

You would drive impact assessment first:

* How many VMs are affected?
* Are they production VMs?
* Is there a safe workaround?
* Can the customer perform OS-level shutdown from inside the VM?
* Is the maintenance window at risk?
* Do we need to pause the change?

Then coordinate technical triage:

* Confirm NGT status.
* Check VM OS and compatibility.
* Check recent upgrades or image changes.
* Compare affected and unaffected VMs.
* Review Prism tasks/alerts.
* Decide whether to involve AHV, Prism, or Engineering.

### Scenario 2 — File-level restore fails

**Customer impact:** A user deleted a critical file and expects recovery from snapshot.

**Likely areas:**

* SSR not enabled.
* NGT missing or unhealthy.
* Snapshot/recovery point not available.
* Guest OS unsupported.
* Permissions issue.
* Customer confusing VM-level snapshot with application-aware backup.

**Manager-level handling:**

You would clarify the recovery objective:

* What file?
* Which VM?
* What timestamp?
* Is there a valid recovery point?
* Is this urgent business-impacting data loss?
* Is there an alternate restore path from backup software?

Then align expectations:

> We need to verify whether the VM had NGT and SSR enabled before the recovery point was created. If not, file-level restore may not be available from that snapshot, and we may need to use an alternate recovery method.

### Scenario 3 — Windows VM performance issue after migration to AHV

**Customer impact:** After moving workloads to AHV, a Windows VM has slow disk or network performance.

**Likely areas:**

* VirtIO drivers missing or outdated.
* Incorrect VM configuration.
* Storage/network bottleneck.
* Guest OS issue.
* Application-level bottleneck.
* Post-migration cleanup incomplete.

**Manager-level handling:**

You would not jump to “Nutanix performance issue.” You would structure triage:

* Is the issue limited to migrated Windows VMs?
* Are VirtIO drivers installed and current?
* Is NGT installed only for advanced features, or are drivers the real issue?
* What are the observed metrics: latency, IOPS, throughput, CPU ready, packet loss?
* Is the bottleneck inside the guest, AHV, storage fabric, physical network, or application?

### Scenario 4 — Snapshot consistency dispute

**Customer impact:** Customer expected a snapshot to be application-consistent, but the restored application/database is not clean.

**Likely areas:**

* Crash-consistent vs application-consistent misunderstanding.
* VSS not working.
* NGT not enabled.
* Protection policy not configured correctly.
* Application not VSS-aware or writer failed.
* Third-party backup responsibility.

**Manager-level handling:**

This is a classic escalation where communication matters as much as technical triage.

You would say:

> First I would establish what type of snapshot was taken, whether the VM was Windows or Linux, whether VSS or application quiescing was configured, and whether the workload supports that consistency model. Then I would avoid assigning blame prematurely and focus the team on evidence: snapshot type, recovery point configuration, NGT/VSS status, application logs, and the customer’s documented recovery expectations.

---

## 6. Triage questions I should ask

### Impact and urgency

1. What is the customer impact: outage, degraded service, failed maintenance, failed recovery, or risk?
2. How many VMs are affected?
3. Are these production workloads?
4. Is there an SLA, RTO, or RPO at risk?
5. Is there an immediate workaround?

### Environment

6. Which AOS/AHV version is running?
7. Is the VM on AHV, ESXi, or migrated between hypervisors?
8. What guest OS and version is installed?
9. Is the OS supported according to Nutanix compatibility guidance?
10. Was there a recent upgrade, migration, template change, or image update?

### NGT status

11. Is Nutanix Guest Tools installed?
12. Is NGT enabled for the VM in Prism?
13. Are the NGT services running inside the guest OS?
14. Is the NGT version compatible with the cluster/AOS version?
15. Is the issue affecting all NGT-enabled VMs or only a subset?

### Feature-specific

16. Is the issue related to graceful shutdown, restart, SSR, VSS, mobility, or something else?
17. For SSR: was Self-Service Restore enabled before the restore was needed?
18. For snapshots: is the customer expecting crash consistency or application consistency?
19. For Windows: are VSS writers healthy?
20. For performance: are VirtIO drivers installed and current?

### Escalation management

21. What evidence has already been collected?
22. What has changed since the last known good state?
23. Is there a safe rollback?
24. Do we need Engineering, SRE, or a product specialist?
25. What is the next customer update time?

---

## 7. Likely interview questions

1. What are Nutanix Guest Tools?
2. Are Nutanix Guest Tools required for every AHV VM?
3. What is the difference between NGT and VirtIO drivers?
4. Why are Guest Tools important in enterprise support?
5. How would you troubleshoot a VM that does not shut down gracefully from Prism?
6. A customer cannot perform file-level restore. What would you check?
7. A Windows VM has poor performance after migration to AHV. Would you look at NGT?
8. What is the difference between crash-consistent and application-consistent snapshots?
9. How would you manage a customer escalation involving failed snapshot recovery?
10. How would you explain Guest Tools to a non-technical customer?
11. How would you prioritize an NGT-related issue across hundreds of production VMs?
12. What metrics or KPIs would you track for this type of support area?
13. How would you prevent recurring NGT-related incidents?
14. How would you coordinate between Support, SRE, Engineering, and the customer?
15. What would you do if the customer is running an unsupported guest OS?

---

## 8. Model answers in English

### Q1. What are Nutanix Guest Tools?

**Model answer:**

> Nutanix Guest Tools, or NGT, are software components installed inside a guest VM to improve integration between the Nutanix platform and the guest operating system. They enable guest-aware operations such as graceful shutdown and restart, self-service file restore, VSS-related snapshot coordination, and some VM mobility workflows. I think of them as the control bridge between Prism/AHV and what is happening inside the VM.

### Q2. Are Guest Tools mandatory for every VM?

**Model answer:**

> Not necessarily. A VM can run on AHV without NGT, assuming it has the required OS support and drivers. For Windows VMs, VirtIO drivers are critical for disk and network devices. NGT is more about advanced manageability and guest-aware operations. So in support, I would be careful to distinguish between a VM runtime issue, a driver issue, and an NGT feature issue.

### Q3. What is the difference between NGT and VirtIO?

**Model answer:**

> VirtIO is the paravirtualized driver layer used by the guest OS to efficiently access virtual devices such as disks and NICs on AHV. NGT is the guest tools package used for platform-to-guest integration, such as graceful power operations, self-service restore, VSS integration, and VM mobility support. In an escalation, this distinction matters because a Windows VM that cannot see a disk or has poor I/O performance may point to VirtIO, while a failed graceful shutdown or file-level restore may point to NGT.

### Q4. How would you troubleshoot graceful shutdown failure from Prism?

**Model answer:**

> I would first assess impact: how many VMs, production or non-production, and whether there is a maintenance window at risk. Technically, I would check whether NGT is installed and enabled on the affected VMs, whether the guest services are running, whether the guest OS is supported, and whether the same operation works from inside the OS. I would also compare affected and unaffected VMs to identify version, template, or configuration drift. As a manager, I would make sure we define a safe workaround, communicate risk clearly, and avoid forcing power operations unless the customer accepts the risk.

### Q5. A customer says file-level restore does not work. What do you check?

**Model answer:**

> I would verify whether Self-Service Restore was enabled for that VM, whether NGT was deployed and healthy, whether the relevant recovery point exists, and whether the guest OS and configuration are supported. I would also clarify the customer’s recovery objective: which file, what restore timestamp, and how urgent the business impact is. If SSR was not enabled before the snapshot or recovery point, I would manage expectations and look for an alternate restore path, such as the customer’s backup solution.

### Q6. How do Guest Tools relate to snapshots?

**Model answer:**

> Guest Tools can help with guest-aware snapshot workflows, particularly in Windows environments where VSS may be involved. However, I would be careful not to claim that every AHV snapshot is application-consistent just because NGT is installed. The consistency level depends on the snapshot mechanism, protection configuration, guest OS support, and application behavior. In a critical escalation, I would verify the exact type of snapshot or recovery point and whether VSS or application quiescing was successfully used.

### Q7. How would you manage an escalation where the customer blames Nutanix for a failed recovery?

**Model answer:**

> I would first acknowledge the severity, especially if data recovery or business continuity is involved. Then I would structure the investigation around evidence: what recovery point was used, what consistency model was expected, whether NGT and VSS were enabled and healthy, whether the guest OS and application were supported, and what logs show from Prism, the guest OS, and the application. I would keep communication factual and avoid premature blame. My role would be to coordinate Support, SRE, Engineering, and the customer toward the fastest safe recovery path while preserving evidence for root cause analysis.

### Q8. What would you do if many VMs have outdated Guest Tools?

**Model answer:**

> I would treat it as a fleet hygiene and risk management problem. First, I would segment the VMs by criticality, OS, business owner, and feature dependency. Then I would work with the customer to define an upgrade plan, test on a small controlled group, confirm rollback options, and schedule production updates through change management. I would also track compliance over time, because outdated tools can become a recurring source of escalations during maintenance, recovery, or migration operations.

### Q9. How would you explain NGT to an executive customer?

**Model answer:**

> I would explain that Nutanix can manage the virtual machine from the infrastructure side, but some operations require cooperation from inside the operating system. Guest Tools provide that cooperation. They help the platform perform safer operations, such as graceful shutdowns and certain restore workflows. If they are missing or unhealthy, the VM may still run, but some advanced operational features may not behave as expected.

### Q10. How does this connect to support KPIs?

**Model answer:**

> NGT issues can affect MTTR, change success rate, recovery success rate, and escalation volume. For example, if graceful shutdown fails during maintenance, the change window expands. If file-level restore fails, recovery time increases. As a support manager, I would look not only at individual tickets but also at patterns: outdated tools, unsupported OS versions, repeated SSR failures, or migration-related driver problems. That allows the team to move from reactive support to preventive operational hygiene.

---

## 9. Connection with my experience

Your current background maps well to this topic because Guest Tools are not just a low-level technical component. They are part of operational reliability.

You can connect it to your experience like this:

### Incident management

You already manage incidents, MTTR, SLA, and escalations. NGT-related problems are often not “full platform down” incidents, but they can become high-severity when they block maintenance, shutdown, restore, or migration.

Good positioning:

> In my current role, I often separate infrastructure availability from operational functionality. A platform can be technically up, but if recovery, monitoring, or controlled maintenance fails, the customer still experiences operational risk. I would apply the same mindset to NGT-related escalations at Nutanix.

### Cloud operations

In SaaS/cloud operations, agents are common:

* Monitoring agents.
* Backup agents.
* Security agents.
* Log forwarders.
* Endpoint management agents.
* Kubernetes node agents.

NGT is conceptually similar: an in-guest component that enables platform-level orchestration and observability.

Good positioning:

> I am used to managing environments where control-plane operations depend on healthy agents inside workloads. The key is to verify agent health, version compatibility, permissions, network path, and the last successful operation before escalating deeper.

### Monitoring and KPIs

You can talk about proactive management:

* Percentage of VMs with NGT installed.
* Percentage of VMs with outdated tools.
* SSR-enabled critical workloads.
* Failed graceful shutdown attempts.
* Snapshot/restore success rate.
* Repeated incidents by OS/template/customer.
* Change failure rate during maintenance.

### Leadership

This topic lets you show you are not positioning yourself as a Senior SRE IC. You understand the technology enough to manage the escalation, ask the right questions, and drive the right people.

Strong framing:

> I would not try to replace the SRE or AHV specialist. My value would be to understand the technical dependencies well enough to structure the triage, remove ambiguity, manage customer communication, and make sure we converge quickly on either workaround, root cause, or engineering escalation.

---

## 10. Minimum I need to memorize

Memorize these points:

1. **NGT = Nutanix Guest Tools**, installed inside the guest VM.
2. NGT enables **advanced guest-aware VM management features**.
3. Key features: **graceful shutdown/restart, SSR/file-level restore, VSS support, VM mobility**.
4. NGT is different from **VirtIO drivers**.
5. **VirtIO** is critical for Windows VM disk/network performance on AHV.
6. NGT is not always mandatory for a VM to run.
7. If graceful shutdown fails, check **NGT installed/enabled/running**, guest OS support, and recent changes.
8. If file-level restore fails, check **SSR enabled, recovery point exists, NGT healthy, guest OS supported**.
9. Do not overpromise **application consistency**. Snapshot consistency depends on configuration and workload.
10. As a manager, focus on **impact, evidence, workaround, ownership, and communication cadence**.

A compact verbal version:

> NGT is the guest integration layer for AHV. It allows Prism and AHV to coordinate certain operations with the operating system inside the VM, such as graceful shutdown, file-level restore, VSS-related workflows, and VM mobility. It is different from VirtIO drivers, which are more about device performance and OS access to virtual disks and NICs. In support, I would check NGT health whenever a guest-aware operation fails, but I would check VirtIO when the issue is boot, disk, network, or performance-related.

---

## 11. Advanced / optional level

You do not need to master these deeply for a manager interview, but you should recognize them:

* Specific NGT service names inside Windows/Linux.
* Exact NGT installation and upgrade procedure.
* aCLI commands for NGT-driven power operations.
* Detailed VSS writer troubleshooting.
* Deep AHV host/CVM logs.
* Cross-hypervisor disaster recovery details.
* VM mobility internals.
* Specific compatibility edge cases by AOS/AHV/Prism version.
* Guest OS support matrix details.
* Interaction with third-party backup tools.
* Advanced Windows storage/network driver debugging.
* Linux guest quiescing details.
* Automation of NGT compliance across large VM fleets.

For the interview, phrase it like this:

> I would not claim to know every internal command yet, but I understand the operational model: NGT is required for certain guest-aware features, VirtIO is critical for device performance, and compatibility/version alignment is essential. In an escalation, I would drive structured triage and involve the right SRE or product specialist for deep command-level analysis.

---

## 12. Final checklist

Before the interview, you should be able to answer:

* Can I explain NGT in 30 seconds?
* Can I clearly distinguish NGT from VirtIO?
* Can I explain why Guest Tools matter for graceful shutdown?
* Can I explain why they matter for file-level restore?
* Can I avoid claiming that all snapshots are application-consistent?
* Can I describe a real escalation involving failed restore or shutdown?
* Can I ask good triage questions?
* Can I connect this to SLA, MTTR, change management, and customer communication?
* Can I position myself as a technical escalation manager rather than a Senior SRE IC?
* Can I say what I would check first without pretending to know every Nutanix command?

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword        | Meaning                                                                                         |
| ------------------------------- | ----------------------------------------------------------------------------------------------- |
| aCLI                            | Acropolis command-line interface used for AHV/VM operations.                                    |
| AHV                             | Acropolis Hypervisor; Nutanix’s native hypervisor.                                              |
| AOS                             | Acropolis Operating System; Nutanix software layer running the cluster.                         |
| Application-consistent snapshot | Snapshot coordinated with the application/OS to reduce recovery inconsistency.                  |
| Backup agent                    | Software inside a VM used by backup tools to coordinate backup/restore.                         |
| Change management               | Controlled process for planning, approving, and executing production changes.                   |
| CHDR                            | Cross-Hypervisor Disaster Recovery; recovery/mobility across hypervisors.                       |
| Crash-consistent snapshot       | Snapshot equivalent to sudden power loss; disk state captured without full app coordination.    |
| CVM                             | Controller VM; Nutanix VM running core storage/control services on each node.                   |
| Escalation                      | Process of raising a case to higher technical or management ownership.                          |
| File-Level Restore              | Recovery of individual files instead of restoring the full VM.                                  |
| Guest OS                        | Operating system running inside the virtual machine.                                            |
| Guest Tools                     | Software inside a VM that enables guest-aware platform operations.                              |
| HCI                             | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform. |
| Hypervisor                      | Software layer that runs and manages virtual machines.                                          |
| I/O                             | Input/output operations, usually disk or network activity.                                      |
| IOPS                            | Input/output operations per second; storage performance metric.                                 |
| KPI                             | Key Performance Indicator; operational measurement such as MTTR or SLA compliance.              |
| Maintenance window              | Approved period for planned infrastructure work.                                                |
| MTTR                            | Mean Time To Recovery/Resolution; average time to restore service or resolve incidents.         |
| NGT                             | Nutanix Guest Tools; guest integration software for Nutanix VMs.                                |
| Prism                           | Nutanix management interface.                                                                   |
| Prism Central                   | Centralized Nutanix management and operations platform.                                         |
| Prism Element                   | Cluster-level Nutanix management interface.                                                     |
| Protection Domain               | Nutanix data protection construct used for snapshot/replication policies.                       |
| Quiescing                       | Pausing or coordinating writes so snapshots are more consistent.                                |
| Recovery point                  | Snapshot or backup point used to restore data or VMs.                                           |
| RPO                             | Recovery Point Objective; acceptable amount of data loss measured in time.                      |
| RTO                             | Recovery Time Objective; target time to restore service.                                        |
| SLA                             | Service Level Agreement; contractual or operational service target.                             |
| Soft shutdown                   | Graceful OS-level shutdown initiated through platform integration.                              |
| SRE                             | Site Reliability Engineering; discipline focused on reliability, automation, and operations.    |
| SSR                             | Self-Service Restore; Nutanix feature for file-level recovery from snapshots/recovery points.   |
| Supportability                  | Whether a configuration is supported and diagnosable by the vendor.                             |
| VSS                             | Volume Shadow Copy Service; Windows framework for coordinating snapshots.                       |
| VSS writer                      | Windows component that coordinates application-specific VSS snapshots.                          |
| VM                              | Virtual Machine.                                                                                |
| VM Mobility                     | Nutanix feature area related to VM migration/conversion scenarios.                              |
| VirtIO                          | Paravirtualized driver framework used by AHV for efficient disk/network devices.                |
| Workaround                      | Safe temporary action to reduce impact while root cause is investigated.                        |

[1]: https://portal.nutanix.com/page/documents/details/?targetId=Web-Console-Guide-Prism-v5_16%3Aman-nutanix-guest-tool-c.html&utm_source=chatgpt.com "Prism 5.16 - Nutanix Guest Tools"
[2]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2023_4%3Amul-ngt-pc-introduction-c.html&utm_source=chatgpt.com "Prism pc.2023.4 - Nutanix Guest Tools"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anew-vm-with-new-os-installation.html&utm_source=chatgpt.com "New VM with New OS Installation - portal.nutanix.com"
[4]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_10%3Aahv-power-operations-vms-t.html&utm_source=chatgpt.com "Performing Power Operations on VMs by Using Nutanix Guest Tools (aCLI)"
[5]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v59%3Aman-file-level-restore-c.html&utm_source=chatgpt.com "Prism 5.9 - Self-Service Restore - Nutanix"
[6]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v10_0%3Awc-vm-manage-acropolis-wc-t.html?utm_source=chatgpt.com "AHV 10.0 - Managing a VM (AHV) - portal.nutanix.com"
[7]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_7%3Awc-windows-vm-provisioning-c.html&utm_source=chatgpt.com "AHV 6.7 - Windows VM Provisioning - Nutanix"
[8]: https://portal.nutanix.com/page/compatibility-interoperability-matrix/guestos/compatibility?utm_source=chatgpt.com "Nutanix Support & Insights"
