# AHV / Virtualization — Hyper-V overview

## 1. Short definition

**Hyper-V** is Microsoft’s **type-1 enterprise hypervisor** for running Windows and Linux virtual machines on Windows Server, Windows client editions, and Azure Local. It provides VM isolation, virtual networking, storage integration, live migration, clustering, and disaster recovery features. Microsoft describes Hyper-V as a hardware virtualization platform that runs directly on compute hardware and supports large-scale VM operations. ([Microsoft Learn][1])

For your Nutanix interviews, Hyper-V matters mainly as a **comparison point and migration source**: many enterprise customers may be moving from Hyper-V, VMware ESXi, or public cloud into **Nutanix AHV**, Nutanix’s native hypervisor integrated with Nutanix Cloud Infrastructure and Prism. Nutanix states that AHV is built directly into NCI, managed through Prism, and integrated with storage, networking, and management services. ([Nutanix][2])

---

## 2. Clear explanation

Hyper-V is Microsoft’s virtualization layer. It allows one physical host to run multiple isolated virtual machines. Each VM receives virtualized resources such as vCPU, memory, virtual disks, and virtual NICs.

From a support perspective, Hyper-V should not be treated as just “Microsoft’s version of VMware.” It is part of a broader Microsoft ecosystem:

* **Windows Server** provides the host operating system.
* **Hyper-V** provides compute virtualization.
* **Failover Clustering** provides VM high availability.
* **Cluster Shared Volumes**, SAN, SMB, iSCSI, Fibre Channel, or Storage Spaces Direct can provide shared or distributed storage.
* **Hyper-V Virtual Switch** provides virtual networking.
* **System Center Virtual Machine Manager**, **Windows Admin Center**, and **PowerShell** are common management layers.
* **Hyper-V Replica** and **Azure Site Recovery** may be used for disaster recovery.

In contrast, **Nutanix AHV** is designed as part of a hyperconverged platform. AHV is integrated with **AOS**, **Distributed Storage Fabric**, **Prism**, cluster operations, VM management, networking, HA, live migration, and lifecycle workflows. Nutanix documentation describes AHV as the native Nutanix hypervisor that integrates virtualization, networking, infrastructure, and operations management through Nutanix Prism. ([Nutanix][3])

The practical distinction:

**Hyper-V mindset:**
“I have Windows hosts, clustering, storage, virtual switching, and Microsoft management tools. I need to understand which layer is failing.”

**Nutanix AHV mindset:**
“I have an HCI cluster where compute, storage, virtualization, and management are tightly integrated. I need to understand the VM, host, CVM, AOS, AHV, storage path, networking, and Prism view together.”

For your role, you do not need to become a deep Hyper-V engineer. You need to understand it well enough to:

* speak credibly with customers migrating from Hyper-V to AHV;
* understand VM, storage, networking, and HA concepts;
* compare Hyper-V and AHV without oversimplifying;
* guide escalations toward the right technical domain;
* ask good triage questions when the customer says, “This worked on Hyper-V but fails on AHV.”

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Worldwide Support Manager at Nutanix**, Hyper-V matters because customers often come from heterogeneous enterprise environments. They may use Nutanix with AHV, VMware ESXi, Microsoft Hyper-V, Kubernetes, public cloud, or mixed infrastructures.

You need to understand Hyper-V to manage escalations involving:

* **Migration to AHV** from Hyper-V.
* VM boot or driver issues after migration.
* Differences in virtual disk controllers, guest tools, and drivers.
* Networking differences between Hyper-V virtual switches and AHV networks.
* HA and live migration expectations.
* Performance comparisons before and after migration.
* Customer expectations shaped by Microsoft tooling.
* Third-party backup, monitoring, antivirus, and security integrations.

Nutanix specifically mentions **Nutanix Move** as a tool for migrating VMs from VMware ESXi, Microsoft Hyper-V, and public clouds to AHV. ([Nutanix][2]) That means Hyper-V knowledge is directly relevant to customer onboarding, modernization projects, and escalations during migration waves.

The support manager angle is important: you are not expected to debug every kernel-level issue personally, but you are expected to understand the failure domain, coordinate the right experts, protect customer confidence, and prevent misclassification of incidents.

A strong interview positioning would be:

> “I do not position myself as a Senior SRE individual contributor for Hyper-V internals. My value is that I understand the virtualization stack well enough to lead escalations, ask the right triage questions, separate compute, storage, networking, guest OS, and platform issues, and coordinate the right technical owners while keeping the customer aligned.”

---

## 4. Key concepts

### 4.1 Type-1 hypervisor

Hyper-V is a **type-1 hypervisor**, meaning the virtualization layer runs directly on hardware rather than as a normal application inside a host OS. Microsoft describes Hyper-V as running directly on computing hardware and providing near-native performance and isolation. ([Microsoft Learn][1])

Interview-safe explanation:

> “Hyper-V is a type-1 Microsoft hypervisor used to run isolated virtual machines at enterprise scale. It abstracts compute, memory, networking, and storage resources so multiple workloads can run on the same physical infrastructure.”

### 4.2 Parent partition and child partitions

Hyper-V uses a parent partition, usually the Windows Server management partition, and child partitions, which are the guest VMs. The parent partition manages device access, VM lifecycle, and communication with the hypervisor.

You do not need to go too deep here, but know the concept because it helps explain why host OS health, drivers, patching, and management services matter.

### 4.3 Virtual machines

A Hyper-V VM consists of:

* vCPUs;
* assigned or dynamic memory;
* virtual disks, often **VHD** or **VHDX**;
* virtual NICs;
* firmware type, usually **Generation 1** or **Generation 2**;
* checkpoints;
* integration services;
* guest OS drivers.

In a Nutanix migration context, VM compatibility matters. A VM that boots correctly on Hyper-V may require changes when moved to AHV, such as disk controller adaptation, boot mode validation, guest driver installation, network configuration, or cleanup of old integration components.

### 4.4 Generation 1 vs Generation 2 VMs

Hyper-V has **Generation 1** and **Generation 2** VMs.

* **Generation 1** uses legacy BIOS-style boot.
* **Generation 2** uses UEFI-based boot and supports more modern security features.

This may matter in migrations because boot mode, disk layout, drivers, and OS support can affect whether a migrated VM starts correctly.

### 4.5 Hyper-V Virtual Switch

The **Hyper-V Virtual Switch** is a software-based Layer 2 Ethernet switch used to connect VMs to external networks, internal networks, or private VM-only networks. Microsoft states that it connects VMs to networks outside the Hyper-V host and provides policy enforcement for security, isolation, and service levels. ([Microsoft Learn][4])

Key things to understand:

* external virtual switch connects VMs to the physical network;
* internal virtual switch connects VMs and host only;
* private virtual switch connects VMs to each other only;
* VLANs are commonly used;
* security policies, port ACLs, DHCP guard, ARP/ND spoofing protection, bandwidth controls, and monitoring can exist at the virtual switch layer. ([Microsoft Learn][4])

Nutanix AHV has its own networking model, including virtual networks, host networking, bridges, bonds, VLANs, and integration with Prism and Flow/Networking features depending on deployment.

### 4.6 Storage

Hyper-V supports multiple storage architectures:

* local storage;
* SAN;
* iSCSI;
* Fibre Channel;
* SMB;
* Cluster Shared Volumes;
* Storage Spaces Direct;
* VHD/VHDX virtual disks.

Microsoft documentation highlights Hyper-V’s flexibility across local disks, SAN connectivity, SMB/NFS-style file-based scenarios, and hyperconverged storage options such as Storage Spaces Direct. ([Microsoft Learn][1])

Nutanix AHV is different because AHV relies on the Nutanix **Distributed Storage Fabric**. Nutanix documentation states that AHV uses DSF for storage provisioning, snapshots, clones, and data protection, and that VM disks are presented as raw SCSI block devices through an optimized I/O path. ([Nutanix][3])

This is one of the most important Nutanix interview points: in AHV, virtualization and distributed storage are integrated parts of the HCI platform.

### 4.7 High availability

Hyper-V typically uses **Windows Server Failover Clustering** for high availability. If a host fails, clustered VMs can restart on another host, assuming storage and cluster health allow it. Microsoft describes Failover Clustering with Cluster Shared Volumes as enabling automatic failover of VMs between cluster nodes. ([Microsoft Learn][1])

AHV also includes VM high availability and scheduling. Nutanix documentation states that AHV includes high availability and dynamic scheduling capabilities. ([Nutanix][3])

Support implication: when a customer reports a VM outage, you must understand whether the expectation was live migration, HA restart, application-level failover, disaster recovery, or backup restore. These are different things.

### 4.8 Live migration

Hyper-V supports live migration: moving a running VM between hosts with minimal or no service interruption. Microsoft also documents live migration, shared-nothing live migration, and storage migration as part of Hyper-V availability and mobility features. ([Microsoft Learn][1])

AHV also supports live migration. Nutanix documentation describes AHV live migration as moving a VM from one host to another without noticeable VM downtime, transferring VM memory while the VM continues running. ([Nutanix][3])

Escalation relevance:

* failed host maintenance;
* long migration time;
* VM stun or perceived application interruption;
* insufficient bandwidth;
* high memory dirtying rate;
* host resource pressure;
* compatibility or affinity restrictions.

### 4.9 Disaster recovery

Hyper-V environments may use:

* Hyper-V Replica;
* Azure Site Recovery;
* storage replication;
* backup platforms;
* application-level replication.

Microsoft describes Hyper-V Replica as asynchronous VM replication to secondary sites and Azure Site Recovery as an extension of Hyper-V disaster recovery capabilities. ([Microsoft Learn][1])

Nutanix environments may use Nutanix native replication, snapshots, protection domains, or third-party backup/DR tooling depending on architecture.

A support manager should distinguish:

* backup;
* snapshot;
* replication;
* HA;
* DR;
* failover;
* failback;
* RPO;
* RTO.

### 4.10 Management tools

Hyper-V environments may be managed with:

* Hyper-V Manager;
* Failover Cluster Manager;
* Windows Admin Center;
* System Center Virtual Machine Manager;
* PowerShell;
* Azure Arc / Azure Local tooling in some environments.

Microsoft lists Windows Admin Center, Hyper-V Manager, PowerShell, and System Center Virtual Machine Manager as management options. ([Microsoft Learn][1])

Nutanix AHV is managed primarily through:

* Prism Element;
* Prism Central;
* CLI tools;
* APIs;
* lifecycle management workflows;
* support bundles and logs.

Nutanix states AHV is managed through Prism, the same web console used to manage the Nutanix cluster. ([Nutanix][2])

---

## 5. How it appears in a real escalation

### Escalation scenario 1 — Migration from Hyper-V to AHV fails

A customer is migrating a Windows Server VM from Hyper-V to AHV using Nutanix Move. The VM migrates, but after cutover it does not boot.

Likely areas:

* boot mode mismatch: BIOS vs UEFI;
* missing VirtIO drivers;
* incompatible storage controller;
* Windows bootloader issue;
* old Hyper-V integration components;
* disk conversion issue;
* secure boot setting;
* generation mismatch;
* guest OS compatibility.

What you should do as manager:

* confirm migration method and source VM configuration;
* confirm whether Nutanix Move completed cleanly;
* identify whether this is a single VM or repeatable pattern;
* check business impact and rollback option;
* involve migration/AHV specialists if needed;
* keep customer communication structured: current status, suspected domain, next action, rollback path.

Model customer-facing framing:

> “We are treating this as a migration cutover issue, not yet as a platform-wide AHV failure. The immediate focus is to confirm boot mode, disk controller compatibility, guest driver state, and whether the VM has a clean rollback path on the source Hyper-V environment.”

### Escalation scenario 2 — Customer says AHV performance is worse than Hyper-V

A customer migrated a SQL Server VM from Hyper-V to AHV and reports higher latency.

Likely areas:

* guest drivers;
* VM sizing;
* CPU ready / host contention;
* storage latency;
* network throughput;
* SQL workload pattern;
* missing performance baseline;
* different storage architecture;
* backup or antivirus activity;
* misaligned expectations.

Triage approach:

* compare before/after metrics;
* collect VM-level, host-level, cluster-level, and application-level data;
* validate VirtIO drivers;
* check storage latency and IOPS;
* check CPU/memory pressure;
* verify network path;
* separate perceived slowness from measurable degradation.

Model manager answer:

> “I would avoid jumping to a hypervisor conclusion. I would first establish a baseline: what changed, what metric degraded, when it started, whether it affects one VM or many, and whether the bottleneck is guest OS, compute, storage, networking, or application-level. Then I would coordinate AHV, storage, and application specialists around the same timeline.”

### Escalation scenario 3 — Network connectivity after Hyper-V migration

After migration from Hyper-V to AHV, the VM boots but cannot communicate with the application network.

Likely areas:

* VLAN mismatch;
* static IP conflict;
* Windows NIC re-enumeration;
* old hidden NIC retaining IP config;
* AHV network not mapped correctly;
* firewall rules;
* MAC address change;
* port group / virtual switch assumptions from Hyper-V;
* upstream switch trunking.

Support manager focus:

* source and target network mapping;
* whether issue is one VM, VLAN, subnet, or cluster-wide;
* packet path from guest to AHV network to physical switch;
* recent changes;
* customer rollback or workaround.

### Escalation scenario 4 — Host maintenance / live migration issue

Customer tries to place a host in maintenance mode, but VM migration is slow or fails.

Possible causes:

* insufficient target host capacity;
* high memory dirty rate;
* network bandwidth limitations;
* affinity policies;
* VM configuration restrictions;
* ongoing backup/snapshot activity;
* host or cluster health issue.

Nutanix documentation notes that live migration can be affected by dirty memory rate and limited network bandwidth, and AHV includes adaptive CPU management to improve migration success and reduce migration time. ([Nutanix][3])

Manager-level framing:

> “I would treat this as an operational risk during planned maintenance. First I would stop the change window if customer impact is likely, then validate cluster health, available capacity, migration errors, and whether specific VMs are preventing host evacuation.”

---

## 6. Triage questions I should ask

### Customer impact

1. What is the business impact?
2. Is production affected?
3. Is this a complete outage, degradation, or migration blocker?
4. How many VMs, hosts, clusters, or sites are affected?
5. Is there an active change window or cutover window?
6. What is the required RTO or SLA?

### Scope

7. Is the issue affecting one VM, multiple VMs, one host, one cluster, or all migrated workloads?
8. Did the issue start after migration, upgrade, host maintenance, network change, storage change, or backup activity?
9. Was the VM previously running on Hyper-V?
10. Is the source VM still available for rollback?

### VM and guest OS

11. What guest OS is running?
12. Was it a Hyper-V Generation 1 or Generation 2 VM?
13. Is the VM BIOS or UEFI?
14. Are the required guest drivers installed?
15. Does the VM boot?
16. Does the OS see disks and NICs correctly?
17. Are there hidden or stale NICs inside Windows?
18. Are there application logs showing errors?

### Storage

19. Is the problem boot, disk visibility, latency, IOPS, or corruption?
20. What was the source disk format?
21. What storage path existed on Hyper-V?
22. Is the issue visible at guest, AHV, AOS, or cluster level?
23. Are snapshots, backups, or replication tasks running?

### Network

24. Which VLAN/subnet should the VM use?
25. How was the Hyper-V virtual switch mapped to AHV networking?
26. Is the correct AHV network assigned?
27. Is the upstream switch trunk allowing the required VLAN?
28. Can the VM reach gateway, DNS, domain controllers, application peers?
29. Did the MAC address or IP change?

### Performance

30. What baseline exists from Hyper-V?
31. Which metric is degraded: CPU, memory, storage latency, network throughput, application response time?
32. Is the issue constant or time-based?
33. Is it linked to backup, antivirus, batch jobs, or peak traffic?
34. Are other VMs on the same host affected?

### Escalation management

35. Who owns the application?
36. Who owns network, storage, Windows, Nutanix, and migration tooling?
37. Is there a rollback decision point?
38. What data do we need before escalating to engineering?
39. What is the next customer update time?
40. What is the agreed workaround?

---

## 7. Likely interview questions

1. What is Hyper-V?
2. How would you explain the difference between Hyper-V and Nutanix AHV?
3. Why does Hyper-V knowledge matter for a Nutanix support manager?
4. What are common issues when migrating from Hyper-V to AHV?
5. A VM migrated from Hyper-V to AHV does not boot. How do you triage?
6. A customer says performance was better on Hyper-V. How do you handle that?
7. How does virtual networking work in Hyper-V?
8. What would you check if a migrated VM has no network connectivity?
9. What is live migration?
10. What is the difference between HA and live migration?
11. How would you manage a critical escalation involving a failed migration cutover?
12. What data would you ask the SRE team to collect?
13. How would you communicate with a frustrated enterprise customer?
14. How do you separate platform issue, guest OS issue, and application issue?
15. How would you position your technical depth if you are not a Senior SRE IC?
16. What do you know about Nutanix AHV?
17. What is Prism?
18. What is Nutanix Move?
19. What is the role of guest drivers in virtualization?
20. What would you memorize before a Nutanix interview?

---

## 8. Model answers in English

### Question: What is Hyper-V?

**Model answer:**

> Hyper-V is Microsoft’s enterprise virtualization platform. It is a type-1 hypervisor built into Windows Server and Windows that allows organizations to run multiple isolated virtual machines on the same physical infrastructure. From a support perspective, I think of Hyper-V as a stack that includes the hypervisor, the Windows host, virtual networking, storage integration, clustering, management tools, and disaster recovery mechanisms. That matters because many incidents are not purely “hypervisor issues”; they can involve guest OS, storage, network, cluster state, drivers, or operational changes.

### Question: How would you compare Hyper-V and Nutanix AHV?

**Model answer:**

> Hyper-V is Microsoft’s hypervisor, tightly integrated with Windows Server, Failover Clustering, PowerShell, System Center, and Windows Admin Center. Nutanix AHV is Nutanix’s native hypervisor integrated into Nutanix Cloud Infrastructure, with management through Prism and deep integration with AOS, distributed storage, networking, lifecycle management, and cluster operations.
>
> In an escalation, I would avoid presenting this as simply Hyper-V versus AHV. I would focus on the operational model. Hyper-V environments often involve Microsoft clustering and separate storage or S2D design choices. AHV runs as part of an HCI platform where compute, storage, virtualization, and management are more integrated. That changes how we triage performance, HA, migration, networking, and lifecycle issues.

### Question: Why does Hyper-V matter for a Nutanix support manager?

**Model answer:**

> It matters because enterprise customers rarely operate in a single-platform world. A Nutanix customer may be migrating from Hyper-V, comparing AHV behavior to Hyper-V, or integrating Nutanix into a Microsoft-heavy environment. As a support manager, I need enough Hyper-V fluency to understand the customer’s baseline, ask the right triage questions, identify whether we are dealing with migration, guest OS, network, storage, or AHV behavior, and coordinate the right SMEs. My role is not to replace the SRE, but to lead the escalation effectively and protect customer trust.

### Question: A VM migrated from Hyper-V to AHV does not boot. How would you triage it?

**Model answer:**

> I would first establish scope and business impact: is this one VM, a batch of VMs, or a production cutover blocker? Then I would check the migration path, the original Hyper-V VM generation, BIOS or UEFI configuration, disk layout, guest OS version, driver state, and whether Nutanix Move completed without errors. I would also confirm whether there is a rollback path to the source Hyper-V VM.
>
> Technically, I would ask the team to check boot mode, storage controller compatibility, VirtIO drivers, Windows bootloader errors, and whether the VM can see its boot disk. Operationally, I would keep the customer updated with a clear timeline, separate investigation from recovery, and define a rollback decision point if the cutover window is at risk.

### Question: A customer says “AHV is slower than Hyper-V.” What do you do?

**Model answer:**

> I would not accept or reject that conclusion immediately. I would ask what metric is slower: application response time, CPU, memory, storage latency, IOPS, network throughput, or user experience. Then I would compare pre-migration and post-migration baselines, validate VM sizing, guest drivers, host resource contention, storage latency, network path, and application-level factors.
>
> From a management perspective, I would align the customer and technical teams around evidence. The goal is to isolate whether the bottleneck is in the guest OS, workload configuration, AHV host, AOS storage path, network, backup process, or application. That avoids emotional platform debates and drives the escalation toward measurable facts.

### Question: What is the difference between HA and live migration?

**Model answer:**

> Live migration is a planned mobility operation where a running VM is moved from one host to another with minimal or no perceived downtime. It is typically used for maintenance or load balancing. HA is a resilience mechanism for unplanned failures: if a host fails, the platform restarts affected VMs on healthy hosts, assuming capacity and cluster health allow it.
>
> In customer communication, that distinction is important because live migration protects planned operations, while HA reduces downtime after failure. Neither is the same as application-level clustering or disaster recovery.

### Question: What is the Hyper-V Virtual Switch?

**Model answer:**

> The Hyper-V Virtual Switch is a software-based Layer 2 Ethernet switch that connects VMs to external, internal, or private networks. It supports VLANs, isolation, traffic policies, monitoring, and security features. In a migration to AHV, virtual networking is one of the key areas to validate because a VM may move successfully but lose connectivity if VLANs, virtual network mappings, IP settings, or upstream switch configuration are not aligned.

### Question: How do you position yourself if you are not a Senior SRE individual contributor?

**Model answer:**

> I position myself as a technical escalation manager. I am technical enough to understand the architecture, ask precise triage questions, challenge assumptions, and coordinate across SRE, engineering, support, customer success, and the customer’s own infrastructure teams. My strength is leading high-pressure incidents: structuring the investigation, protecting SLA and communication quality, driving ownership, and ensuring that the right technical experts are focused on the right failure domain.

### Question: What do you know about AHV?

**Model answer:**

> AHV is Nutanix’s native enterprise hypervisor, integrated into Nutanix Cloud Infrastructure and managed through Prism. It provides core virtualization capabilities such as VM lifecycle management, high availability, live migration, and virtual networking, while integrating with Nutanix AOS and the distributed storage layer. From a support perspective, what makes AHV important is that it is not an isolated hypervisor product; it is part of the Nutanix HCI operating model, so troubleshooting often requires looking across VM, host, CVM, storage, network, Prism, and cluster health together.

---

## 9. Connection with my experience

Your background maps strongly to this topic.

At Harmonic, you already operate in the world that Nutanix support cares about:

* 24/7 enterprise support;
* incident management;
* customer escalations;
* SLA and MTTR ownership;
* cloud operations;
* monitoring;
* cross-functional coordination;
* Jira / Confluence workflows;
* customer-facing communication;
* operational discipline.

The bridge you should make in interviews:

> “My current experience is not limited to ticket handling. I manage production-impacting incidents where the challenge is to isolate the failure domain, coordinate technical teams, manage customer communication, protect SLA, and drive resolution. Virtualization escalations follow the same operating pattern: define scope, establish impact, gather evidence, isolate compute/storage/network/guest/application layers, communicate clearly, and prevent recurrence.”

Specific connection points:

| Your current experience        | Nutanix / Hyper-V / AHV relevance                                    |
| ------------------------------ | -------------------------------------------------------------------- |
| 24/7 support leadership        | Worldwide Support requires global incident discipline                |
| SLA / MTTR                     | Critical for enterprise escalations and customer trust               |
| Cloud operations               | Helps understand hybrid cloud, VM operations, automation, monitoring |
| Monitoring with Grafana/Kibana | Useful for evidence-based troubleshooting                            |
| Jira/Confluence                | Strong for escalation tracking, RCA, knowledge base, handoffs        |
| Incident bridges               | Directly relevant to Sev1/Sev2 customer escalations                  |
| Coaching engineers             | Relevant to managing SRE/support teams                               |
| Customer communication         | Essential for high-pressure enterprise cases                         |
| AWS/Azure/GCP knowledge        | Helps understand hybrid and migration scenarios                      |
| Kubernetes/Linux               | Useful adjacent knowledge for modern workloads on Nutanix            |

Your gap is not leadership; your gap is product-specific fluency. That is fixable.

---

## 10. Minimum I need to memorize

Memorize these points cold:

1. **Hyper-V is Microsoft’s type-1 hypervisor** for running Windows/Linux VMs.
2. Hyper-V is commonly integrated with **Windows Server, Failover Clustering, Hyper-V Virtual Switch, VHD/VHDX storage, PowerShell, Windows Admin Center, and SCVMM**.
3. **AHV is Nutanix’s native hypervisor**, integrated with **NCI, AOS, Prism, DSF, networking, HA, and lifecycle operations**.
4. Hyper-V and AHV are both enterprise hypervisors, but AHV is part of an **HCI platform**, not just a standalone hypervisor.
5. Nutanix AHV is managed via **Prism**.
6. Nutanix Move can support migration from **Hyper-V to AHV**.
7. Common migration issues include **boot mode, drivers, disk controllers, guest tools, network mapping, VLANs, IP conflicts, and performance baselines**.
8. **HA is not live migration**.
9. **Live migration is planned mobility**; **HA is recovery after host failure**.
10. In escalations, never jump directly to “AHV issue”; isolate **guest OS, VM config, host, storage, network, cluster, application, and recent changes**.
11. Your role is **technical escalation manager**, not deep IC debugger.
12. Strong support leadership means **impact, scope, timeline, ownership, communication, mitigation, RCA, and prevention**.

A concise interview phrase to memorize:

> “My approach is to separate the virtualization stack into failure domains: guest OS, VM configuration, hypervisor, host resources, storage path, networking, management plane, and application behavior. That lets the team avoid assumptions and drive the escalation with evidence.”

---

## 11. Advanced / optional level

You can leave these as advanced unless the interview becomes deeply technical:

* Hyper-V parent/child partition internals.
* VMBus architecture.
* Hyper-V enlightenments.
* Synthetic vs emulated devices.
* Detailed VHDX internals.
* Cluster Shared Volumes deep troubleshooting.
* Storage Spaces Direct internals.
* Hyper-V Replica configuration.
* SCVMM fabric design.
* Shielded VM / Host Guardian Service internals.
* SR-IOV, RDMA, VMQ, SET deep networking.
* NUMA topology and CPU scheduler internals.
* AHV Turbo internal data path.
* AHV CLI deep commands.
* Open vSwitch internals in AHV.
* Stargate / iSCSI redirector deep troubleshooting.
* Advanced AOS and CVM service internals.

However, you should be able to recognize these terms. If an SRE mentions them, you should not look lost.

A good response when you reach your depth boundary:

> “I understand the concept and support impact, but I would bring in the specialist for low-level implementation details. My responsibility would be to make sure we have the right logs, timeline, reproduction pattern, customer impact, and escalation ownership.”

---

## 12. Final checklist

Before the interview, you should be able to explain:

* [ ] What Hyper-V is.
* [ ] What AHV is.
* [ ] Why AHV is central to Nutanix HCI.
* [ ] How Hyper-V and AHV differ operationally.
* [ ] What Nutanix Prism does.
* [ ] What Nutanix Move is used for.
* [ ] What can go wrong during Hyper-V-to-AHV migration.
* [ ] How to triage a VM that does not boot after migration.
* [ ] How to triage a VM with no network after migration.
* [ ] How to respond to “performance was better before.”
* [ ] Difference between HA, live migration, backup, snapshot, replication, and DR.
* [ ] How to manage a Sev1 migration escalation.
* [ ] How to communicate without overpromising.
* [ ] How to position yourself as a technical escalation manager.
* [ ] Which parts you know deeply and which parts you would route to SRE/engineering.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword         | Meaning                                                                                    |
| -------------------------------- | ------------------------------------------------------------------------------------------ |
| AHV                              | Acropolis Hypervisor; Nutanix native enterprise hypervisor.                                |
| AOS                              | Acropolis Operating System; Nutanix software layer providing storage and cluster services. |
| ARP                              | Address Resolution Protocol; maps IP addresses to MAC addresses.                           |
| Azure Local                      | Microsoft hybrid infrastructure platform using Azure-connected on-premises resources.      |
| Azure Site Recovery              | Microsoft DR service for replication, failover, and failback.                              |
| BIOS                             | Legacy firmware boot mode used by older VM types.                                          |
| Checkpoint                       | Hyper-V point-in-time VM state, similar conceptually to a snapshot.                        |
| Cluster Shared Volumes / CSV     | Microsoft clustering feature allowing nodes to access shared storage volumes.              |
| Confluence                       | Documentation and knowledge management tool.                                               |
| CVM                              | Controller VM; Nutanix VM that runs core storage and cluster services on each node.        |
| DHCP Guard                       | Hyper-V switch feature preventing rogue DHCP server behavior.                              |
| Disaster Recovery / DR           | Recovery strategy for site, platform, or major infrastructure failure.                     |
| Distributed Storage Fabric / DSF | Nutanix distributed storage layer used by AHV and AOS.                                     |
| DNS                              | Domain Name System; resolves names to IP addresses.                                        |
| Failback                         | Returning workloads to the original site after DR failover.                                |
| Failover                         | Moving or restarting workloads on another host or site after failure.                      |
| Failover Clustering              | Microsoft clustering technology used for HA in Hyper-V environments.                       |
| Generation 1 VM                  | Hyper-V VM using legacy BIOS-style architecture.                                           |
| Generation 2 VM                  | Hyper-V VM using UEFI-based architecture.                                                  |
| Grafana                          | Monitoring and visualization platform.                                                     |
| Guest OS                         | Operating system running inside a VM.                                                      |
| HA                               | High Availability; keeps workloads running or restarts them after failures.                |
| HCI                              | Hyperconverged Infrastructure; combines compute, storage, virtualization, and management.  |
| Host                             | Physical server running the hypervisor.                                                    |
| Hyper-V                          | Microsoft enterprise hypervisor for running virtual machines.                              |
| Hyper-V Manager                  | Microsoft GUI tool for managing Hyper-V hosts and VMs.                                     |
| Hyper-V Replica                  | Microsoft asynchronous VM replication feature for DR.                                      |
| Hyper-V Virtual Switch           | Software Layer 2 switch connecting Hyper-V VMs to networks.                                |
| iSCSI                            | IP-based storage protocol used to access block storage over networks.                      |
| Jira                             | Ticketing and workflow management tool.                                                    |
| Kibana                           | Log search and visualization tool.                                                         |
| Kubernetes                       | Container orchestration platform.                                                          |
| Live Migration                   | Moving a running VM between hosts with minimal disruption.                                 |
| MAC Address                      | Layer 2 network hardware address.                                                          |
| MTTR                             | Mean Time To Resolution/Recovery; key operational support metric.                          |
| NCI                              | Nutanix Cloud Infrastructure; Nutanix HCI platform.                                        |
| ND                               | Neighbor Discovery; IPv6 mechanism similar to ARP.                                         |
| NIC                              | Network Interface Card; physical or virtual network adapter.                               |
| Nutanix Move                     | Nutanix migration tool for moving VMs from platforms such as Hyper-V, ESXi, and cloud.     |
| Prism                            | Nutanix management interface for clusters, VMs, storage, networking, and operations.       |
| Prism Central                    | Centralized Nutanix management plane across clusters.                                      |
| Prism Element                    | Nutanix management interface for a single cluster.                                         |
| RCA                              | Root Cause Analysis; post-incident analysis identifying cause and prevention.              |
| RDMA                             | Remote Direct Memory Access; low-latency network technology.                               |
| RPO                              | Recovery Point Objective; acceptable data loss window.                                     |
| RTO                              | Recovery Time Objective; target time to restore service.                                   |
| SAN                              | Storage Area Network; dedicated storage network/infrastructure.                            |
| SCVMM                            | System Center Virtual Machine Manager; Microsoft enterprise virtualization manager.        |
| Sev1 / Sev2                      | High-severity incident classifications.                                                    |
| SLA                              | Service Level Agreement; contractual or operational service target.                        |
| SMB                              | Server Message Block; Microsoft file-sharing protocol used in some storage designs.        |
| Snapshot                         | Point-in-time copy/state used for recovery or protection workflows.                        |
| SR-IOV                           | Single Root I/O Virtualization; allows VMs near-direct access to hardware resources.       |
| Storage Spaces Direct / S2D      | Microsoft software-defined storage for Windows Server HCI.                                 |
| Storage Migration                | Moving VM storage without necessarily moving compute execution.                            |
| UEFI                             | Modern firmware boot mode replacing legacy BIOS.                                           |
| vCPU                             | Virtual CPU assigned to a VM.                                                              |
| VHD                              | Older Microsoft virtual hard disk format.                                                  |
| VHDX                             | Modern Microsoft virtual hard disk format.                                                 |
| VLAN                             | Virtual LAN; Layer 2 network segmentation method.                                          |
| VM                               | Virtual Machine; software-defined computer running on a hypervisor.                        |
| VMQ                              | Virtual Machine Queue; Microsoft network performance feature.                              |
| VMBus                            | Hyper-V communication channel between host and guest components.                           |
| Windows Admin Center             | Microsoft web-based management tool for Windows Server and Hyper-V.                        |
| Windows Server Failover Cluster  | Cluster used to provide HA for Windows workloads and Hyper-V VMs.                          |
| Workload                         | Application or service running inside a VM or platform.                                    |

[1]: https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/overview "Hyper-V virtualization in Windows Server and Windows | Microsoft Learn"
[2]: https://www.nutanix.com/products/ahv "AHV: Virtualization Solution for Enterprise | Nutanix"
[3]: https://www.nutanix.com/content/dam/nutanix/documents/certifications/ahv-admin.pdf "AHV Administration Guide"
[4]: https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/virtual-switch "Hyper-V Virtual Switch | Microsoft Learn"
