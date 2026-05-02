# AHV / Virtualization — VMware ESXi

## 1. Short definition

**VMware ESXi** is a bare-metal hypervisor: it runs directly on physical server hardware and provides the virtualization layer that allows multiple virtual machines to share CPU, memory, storage, and networking resources. In VMware vSphere, **ESXi provides the virtualization capabilities**, while **vCenter Server centrally manages multiple ESXi hosts**. ([Broadcom TechDocs][1])

For Nutanix interviews, you should understand ESXi from two angles:

1. **As a customer environment Nutanix must support**, because many enterprise Nutanix customers still run VMware vSphere on Nutanix HCI.
2. **As a comparison point to AHV**, Nutanix’s native hypervisor, which is integrated into Nutanix Cloud Infrastructure and managed through Prism. ([Nutanix][2])

---

## 2. Clear explanation

A traditional virtualized data center has three main layers:

1. **Physical hardware**: servers, CPUs, memory, NICs, disks.
2. **Hypervisor layer**: ESXi, AHV, Hyper-V, KVM, etc.
3. **Virtual machines**: guest OSs and applications.

In VMware environments, **ESXi** runs directly on each physical host. It abstracts the hardware and exposes virtual CPU, virtual memory, virtual disks, and virtual NICs to VMs. Multiple ESXi hosts are usually grouped into a **vSphere cluster**, managed by **vCenter Server**.

In a Nutanix environment running VMware vSphere, the architecture is different from a classic SAN-based VMware environment. Nutanix uses a **Controller VM — CVM — on every node**. The CVMs form a distributed, shared-nothing storage layer. Nutanix describes this as a highly resilient converged compute and storage solution for virtual environments such as VMware vSphere. ([Nutanix][3])

The important distinction:

| Layer                      | VMware on traditional infrastructure | VMware on Nutanix                                            |
| -------------------------- | ------------------------------------ | ------------------------------------------------------------ |
| Hypervisor                 | ESXi                                 | ESXi                                                         |
| Management                 | vCenter                              | vCenter + Prism                                              |
| Storage                    | SAN / NAS / vSAN / external arrays   | Nutanix AOS / Distributed Storage Fabric                     |
| Storage presented to ESXi  | VMFS, NFS, vSAN, etc.                | Nutanix storage container usually presented as NFS datastore |
| Nutanix-specific component | Not applicable                       | CVM on every node                                            |

Nutanix documentation states that **AOS storage presents usable storage to vSphere as an NFS datastore**, and that the storage container’s replication factor determines usable capacity and failure tolerance: RF2 tolerates one component failure, RF3 tolerates two. ([Nutanix][4])

So, in interview language:

> “ESXi is the VMware hypervisor layer. On Nutanix, ESXi still manages VM execution, CPU scheduling, memory, virtual networking, and VM lifecycle through vCenter, but storage is provided by Nutanix AOS through the CVM layer. That means escalations often require correlating vSphere symptoms with Nutanix storage, CVM health, networking, and cluster state.”

That is exactly the level expected from a **technical escalation manager**: not necessarily writing low-level ESXi kernel commands from memory, but understanding where the responsibility boundaries are and how to drive a structured escalation.

---

## 3. Why it matters for Nutanix / Worldwide Support

This topic matters because Nutanix support sits at the intersection of **hypervisor, storage, networking, hardware, and customer operations**.

A Worldwide Support Manager must be able to:

1. **Understand customer language**
   Enterprise customers may say: “vCenter shows datastore latency,” “ESXi host disconnected,” “VMs are stunned,” “DRS is not evacuating,” “HA failed to restart VMs,” or “NFS datastore is inaccessible.”

2. **Distinguish Nutanix vs VMware ownership**
   Not every issue seen in vCenter is caused by Nutanix. The root cause may be ESXi configuration, vCenter, licensing, network MTU, DNS/NTP, host drivers, firmware, VM snapshots, third-party backup, or a Nutanix CVM/AOS/storage issue.

3. **Coordinate cross-functional escalation**
   A manager needs to coordinate Nutanix Support, SREs, Engineering, VMware/Broadcom support, account teams, TAMs, and the customer’s infrastructure team.

4. **Protect SLAs and customer trust**
   A production VMware-on-Nutanix issue can impact thousands of VMs. Support leadership must manage severity, communications, mitigation, and root cause clarity.

5. **Explain AHV positioning without sounding anti-VMware**
   Nutanix AHV is Nutanix’s native hypervisor, integrated into NCI and managed through Prism. Nutanix positions AHV as a modern virtualization platform for data center, edge, and public cloud workloads, with enterprise features such as HA, live migration, DR, and Prism-based management. ([Nutanix][2])
   But many customers still run VMware. A good support manager must be credible in both worlds.

---

## 4. Key concepts

### ESXi

ESXi is the bare-metal hypervisor installed on each physical host. It runs VMs and provides virtualized access to CPU, memory, storage, and networking.

Interview phrasing:

> “ESXi is the execution layer. It is where VMs actually run. vCenter is the management plane, but the workload impact usually happens at the ESXi host, datastore, network, or VM layer.”

### vCenter Server

vCenter centrally manages ESXi hosts, clusters, VMs, permissions, templates, DRS, HA, vMotion, alarms, and tasks. Nutanix documentation also states that vCenter enables centralized management of multiple ESXi hosts. ([Nutanix][5])

Support relevance:

> “If vCenter is down, VMs may continue running on ESXi, but centralized management, migrations, monitoring, and some operational workflows are affected.”

### vSphere cluster

A vSphere cluster is a group of ESXi hosts managed together, usually with HA and DRS enabled.

Common support concerns:

* Host disconnected from vCenter.
* HA admission control blocking operations.
* DRS not migrating VMs.
* Maintenance mode failing.
* Resource imbalance.
* Inconsistent host versions or drivers.

### vMotion

vMotion migrates a running VM from one ESXi host to another with minimal or no visible downtime.

Support relevance:

* Used before maintenance.
* Used during host evacuation.
* Can fail due to CPU compatibility, networking, licensing, datastore access, or resource constraints.

### DRS — Distributed Resource Scheduler

DRS balances VMs across ESXi hosts using vMotion according to resource demand, rules, and policies. Broadcom documentation describes DRS as responsible for moving VMs across ESXi hosts using vMotion and policy rules. ([Broadcom TechDocs][6])

Support relevance:

* A customer may report “cluster imbalance.”
* DRS may not move VMs due to anti-affinity rules, reservations, maintenance mode, HA admission control, or resource constraints.

### HA — High Availability

vSphere HA restarts VMs on surviving hosts after a host failure. It is not the same as zero-downtime availability. It is restart-based protection.

Support relevance:

* If a host fails, HA should restart VMs elsewhere if resources and policies allow it.
* HA may fail due to insufficient capacity, datastore heartbeat issues, isolation response configuration, or host/network issues.

Broadcom documentation notes that vSphere Availability covers business continuity solutions such as vSphere HA and Fault Tolerance. ([Broadcom TechDocs][7])

### Datastore

A datastore is logical storage used by ESXi to store VM files: VMX configuration files, VMDKs, logs, snapshots, templates, and ISOs.

On Nutanix with VMware vSphere, Nutanix storage containers are commonly presented as NFS datastores. Nutanix documentation states that NFS is the recommended datastore type in vSphere because of performance and scalability advantages over iSCSI. ([Nutanix][8])

### Nutanix CVM

The **Controller VM** is a Nutanix VM running on every node. It handles storage services and participates in the distributed storage fabric.

Nutanix states that the CVM plays a pivotal role in data management, handling user I/O, data placement, metadata, data integrity, availability, garbage collection, compression, deduplication, erasure coding, snapshots, replication, and performance optimization. ([Nutanix][9])

Interview phrasing:

> “In VMware-on-Nutanix, the CVM is critical. If ESXi is healthy but the local CVM or cluster storage services are degraded, VMs may experience datastore latency, I/O errors, or availability symptoms.”

### Nutanix Distributed Storage Fabric / AOS Storage

AOS aggregates local disks across nodes into a distributed storage layer. In VMware environments, it presents storage to ESXi as datastores. Nutanix describes all CVMs working together to aggregate storage resources into a single global pool consumed by guest VMs, with storage services designed to preserve data and system integrity during node, disk, application, or hypervisor software failures. ([Nutanix][10])

### AHV

AHV is Nutanix’s native hypervisor. It is integrated into Nutanix Cloud Infrastructure and managed through Prism. Nutanix states that AHV is not a separate standalone product to install or license, and that it is managed through Prism. ([Nutanix][2])

Interview positioning:

> “I would not position AHV simply as ‘ESXi replacement.’ I would position it as Nutanix’s integrated virtualization layer, reducing operational fragmentation by aligning compute, storage, networking, upgrades, and support under the Nutanix platform.”

---

## 5. How it appears in a real escalation

### Escalation example 1 — “All VMs on one ESXi host are slow”

Customer report:

> “Several production VMs are experiencing high latency. vCenter shows datastore latency on one host.”

Possible domains:

* ESXi host CPU/memory contention.
* VM snapshot chain or backup stun.
* Network issue between ESXi and CVM.
* Local CVM degraded or services down.
* Disk or node issue in Nutanix cluster.
* NFS datastore issue.
* Third-party backup or antivirus.
* Firmware/driver mismatch.
* Noisy neighbor workload.

Manager-level response:

> “I would first confirm blast radius: one VM, one host, one datastore, one cluster, or one site. Then I would split the investigation into compute, storage, and network. For VMware-on-Nutanix, I would have the team check vCenter alarms, ESXi host health, VM resource contention, datastore latency, CVM health, AOS cluster status, recent changes, and whether backups or snapshots correlate with the incident window.”

### Escalation example 2 — “ESXi host disconnected from vCenter”

Possible causes:

* Management network failure.
* DNS/NTP issue.
* vCenter service issue.
* ESXi management agents issue.
* Host overloaded.
* Certificate/authentication issue.
* Host isolation.
* Physical NIC/switch problem.

Support risk:

VMs may still be running, but operations are impaired. The customer may panic because vCenter visibility is lost.

Good manager communication:

> “A disconnected host in vCenter does not automatically mean the VMs are down. First we validate customer impact, VM reachability, and host management connectivity. Then we check whether the problem is vCenter-side, ESXi management-plane-side, or network-side.”

### Escalation example 3 — “Host cannot enter maintenance mode”

Possible causes:

* DRS cannot evacuate VMs.
* HA admission control reserves capacity.
* Affinity/anti-affinity rules.
* VM has local devices attached.
* Insufficient resources on remaining hosts.
* VM migration network issue.
* Datastore visibility issue.

Broadcom documentation notes that in clusters using DRS and HA with admission control enabled, VMs may not be evacuated from hosts entering maintenance mode because resources are reserved for failover; manual vMotion may be required. ([Broadcom TechDocs][11])

Support manager angle:

> “I would avoid treating this as a generic ‘maintenance mode failed’ issue. I would ask the team to identify which VM blocked evacuation and why: resource reservation, affinity rule, vMotion network, datastore access, HA policy, or a locked VM operation.”

### Escalation example 4 — “VMware customer considering AHV migration”

Customer concern:

> “With VMware licensing changes, we are evaluating AHV but are worried about operational risk.”

Support leadership response:

> “The support role is to reduce migration risk, not to oversell. I would ask about workload criticality, OS mix, backup dependencies, network design, operational tooling, change windows, rollback requirements, and skills readiness. Nutanix Move can help migrate VMs from sources including VMware ESXi to AHV, but planning, validation, and stakeholder communication remain critical.” ([Nutanix][2])

---

## 6. Triage questions I should ask

### Impact and severity

1. Which applications or business services are impacted?
2. How many VMs are affected?
3. Is the issue affecting one VM, one ESXi host, one datastore, one cluster, or multiple sites?
4. Is this performance degradation, partial outage, or full outage?
5. When did it start, and what changed before it started?

### VMware layer

6. Are there vCenter alarms?
7. Are any ESXi hosts disconnected, not responding, or in maintenance mode?
8. Are HA, DRS, or vMotion errors present?
9. Are VMs showing high CPU ready, memory ballooning, swapping, or disk latency?
10. Are there active snapshots, backups, or stuck tasks?

### Nutanix layer

11. Are all CVMs up?
12. Is the Nutanix cluster healthy in Prism?
13. Are there AOS, disk, node, or storage container alerts?
14. Is the affected datastore an NFS datastore backed by Nutanix storage?
15. Is latency local to one host/CVM or cluster-wide?

### Network layer

16. Are management, vMotion, storage, and VM networks healthy?
17. Any recent switch, VLAN, MTU, LACP, DNS, or routing changes?
18. Is packet loss or high latency visible between ESXi hosts, CVMs, and vCenter?
19. Are physical NICs up and balanced?
20. Any top-of-rack switch alerts?

### Change and dependency management

21. Any recent ESXi patch, AOS upgrade, firmware update, LCM operation, or vCenter change?
22. Any recent backup, DR test, storage policy change, or VM migration?
23. Any third-party agents involved: backup, monitoring, antivirus, security scanning?
24. Was there a power, cooling, network, or hardware event?
25. Is there an open case with VMware/Broadcom or another vendor?

### Communication

26. Who is the customer incident commander?
27. What is the next executive update deadline?
28. What mitigation has already been attempted?
29. Is the customer asking for restoration, RCA, workaround, or migration advice?
30. What evidence do we need before making a change?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you manage a Sev1 escalation affecting VMware VMs on a Nutanix cluster?
2. How do you separate VMware, Nutanix, network, and customer-responsibility domains?
3. How would you communicate with an enterprise customer during a prolonged infrastructure incident?
4. How do you prevent escalation ping-pong between vendors?
5. How do you coach a support engineer who lacks VMware expertise?
6. How would you prioritize cases when multiple strategic customers are impacted?
7. What KPIs would you track for enterprise virtualization support?
8. How would you handle a customer demanding RCA before the technical investigation is complete?
9. How do you decide when to involve Engineering?
10. How would you explain AHV to a VMware-heavy customer without sounding biased?

### Technical / SRE interview

1. What is ESXi, and how is it different from vCenter?
2. What happens when an ESXi host fails in a vSphere HA cluster?
3. What is DRS, and why might it fail to migrate a VM?
4. What is vMotion, and what dependencies does it have?
5. How does storage work when VMware runs on Nutanix?
6. What is a Nutanix CVM, and why is it critical?
7. What would you check if vCenter reports high datastore latency?
8. How would you troubleshoot an ESXi host disconnected from vCenter?
9. What is the difference between VM performance issue and datastore performance issue?
10. What is the difference between HA restart and live migration?

### Advanced / panel interview

1. A customer reports intermittent VM stun during backup windows. How do you lead the escalation?
2. A host cannot enter maintenance mode before a planned upgrade. What do you check?
3. After an ESXi upgrade, several VMs lose network connectivity. How do you coordinate the investigation?
4. A customer says Nutanix storage is slow because vCenter shows NFS latency. How do you respond?
5. A VMware customer wants to migrate to AHV. What risks would you identify?
6. How would you structure a post-incident review for a cross-vendor virtualization outage?
7. What information should be included in the first 30 minutes of a Sev1 update?
8. How do you avoid making the wrong team chase the wrong layer?
9. What is your approach when logs are incomplete or the customer already rebooted systems?
10. How do you balance technical depth with executive communication?

---

## 8. Model answers in English

### Q1. What is ESXi?

**Model answer:**

> “ESXi is VMware’s bare-metal hypervisor. It runs directly on the physical server and provides the virtualization layer for virtual machines. ESXi handles VM execution, CPU and memory scheduling, virtual networking, and access to storage. In a vSphere environment, ESXi hosts are usually managed centrally by vCenter Server. For Nutanix customers running VMware, ESXi is still the hypervisor, but the storage layer is provided by Nutanix AOS through the CVM and distributed storage fabric.”

### Q2. What is the difference between ESXi and vCenter?

**Model answer:**

> “ESXi is the host-level hypervisor where VMs actually run. vCenter is the centralized management plane used to manage multiple ESXi hosts, clusters, permissions, alarms, templates, HA, DRS, and vMotion. If vCenter is unavailable, running VMs may continue operating on ESXi hosts, but centralized operations and visibility are degraded.”

### Q3. How does VMware run on Nutanix?

**Model answer:**

> “In VMware-on-Nutanix deployments, ESXi runs on the Nutanix nodes and vCenter manages the ESXi cluster. Nutanix AOS provides the distributed storage layer through Controller VMs running on every node. Storage is typically presented to ESXi as NFS datastores backed by Nutanix storage containers. So troubleshooting requires looking at vCenter and ESXi, but also Prism, CVM health, AOS cluster health, storage latency, and the network between these components.”

### Q4. A customer says: “vCenter shows datastore latency, Nutanix is slow.” How do you respond?

**Model answer:**

> “I would acknowledge the impact but avoid jumping to root cause. Datastore latency in vCenter is a symptom, not a complete diagnosis. I would first determine the blast radius: affected VM, host, datastore, cluster, or site. Then I would correlate vCenter metrics with Nutanix Prism alerts, CVM health, AOS cluster status, host resource contention, backup activity, snapshots, and network health. The goal is to quickly identify whether this is storage-layer, hypervisor-layer, VM-layer, or network-layer.”

### Q5. What is the role of the Nutanix CVM?

**Model answer:**

> “The Controller VM is central to Nutanix architecture. It runs on each node and participates in the distributed storage fabric. It handles storage I/O, metadata, data placement, data protection, and performance optimization. In VMware-on-Nutanix, the ESXi host runs the VMs, but the Nutanix CVM provides the storage services behind the datastore. That is why CVM health is one of the first things I would validate in a VMware-on-Nutanix storage or performance escalation.”

### Q6. What happens when an ESXi host fails?

**Model answer:**

> “If vSphere HA is configured correctly and sufficient resources are available, VMs from the failed host are restarted on surviving hosts. This is not the same as live migration. It is a restart-based recovery mechanism. In a Nutanix environment, I would also validate that the Nutanix cluster remains healthy, that data remains available through the distributed storage fabric, and that there are no related CVM, disk, or network issues.”

### Q7. Why might vMotion fail?

**Model answer:**

> “vMotion can fail for several reasons: incompatible CPU features, EVC configuration, insufficient target resources, network connectivity issues, vMotion VMkernel misconfiguration, datastore visibility problems, active snapshots or locks, VM device attachments, or policy constraints. As a support manager, I would ask the engineer to identify the exact failed VM, the source and destination host, the vCenter task error, and whether the issue is systemic or isolated.”

### Q8. How do you manage a Sev1 escalation in this area?

**Model answer:**

> “I would structure the escalation around impact, ownership, mitigation, and communication. First, confirm the customer impact and blast radius. Second, establish a technical bridge with clear roles: VMware layer, Nutanix layer, network layer, application validation, and customer communications. Third, drive immediate mitigation while preserving evidence. Fourth, provide regular updates with facts, next actions, owners, and ETA for the next checkpoint. After restoration, I would drive RCA, corrective actions, and knowledge capture.”

### Q9. How would you explain AHV to a VMware customer?

**Model answer:**

> “I would explain AHV as Nutanix’s integrated enterprise hypervisor, not simply as a cheaper ESXi clone. AHV is built into Nutanix Cloud Infrastructure, managed through Prism, and integrated with Nutanix storage, networking, data protection, and lifecycle operations. For customers, the value is operational simplicity, integrated support, and reduced dependency on separate virtualization licensing and tooling. But migration must be assessed carefully based on workload compatibility, operational tooling, backup dependencies, and risk tolerance.”

### Q10. Are you a VMware expert?

**Model answer:**

> “I would not position myself as a senior VMware architect. My strength is leading enterprise support and operations across complex systems. I understand the VMware concepts required to manage escalations: ESXi, vCenter, HA, DRS, vMotion, datastores, networking, and performance triage. My role is to ask the right diagnostic questions, coordinate the right specialists, communicate clearly with the customer, and ensure the case progresses toward mitigation and RCA.”

This answer is important because it positions you correctly: **technical escalation manager**, not **Senior SRE individual contributor**.

---

## 9. Connection with my experience

Your Harmonic experience maps strongly to this topic.

| Your experience         | VMware/Nutanix interview translation                                                                                                      |
| ----------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| 24/7 enterprise support | You understand severity handling, shift coverage, and production pressure.                                                                |
| Incident management     | You can lead Sev1/Sev2 bridges for infrastructure outages.                                                                                |
| SLA / MTTR              | You understand restoration priority and measurable support outcomes.                                                                      |
| KPIs                    | You can track escalation aging, reopen rate, time to mitigation, time to RCA, backlog health.                                             |
| Cloud operations        | You understand distributed systems, capacity, monitoring, availability, and change risk.                                                  |
| Grafana / Kibana        | You can correlate symptoms across telemetry sources.                                                                                      |
| Jira / Confluence       | You can manage case workflow, postmortems, runbooks, and knowledge base quality.                                                          |
| Salesforce              | You understand enterprise support case lifecycle and customer communication.                                                              |
| AWS / Azure / GCP       | You can compare cloud abstractions with virtualized infrastructure: compute, storage, networking, availability zones, maintenance events. |
| Kubernetes              | You understand scheduling, node health, workload placement, and control-plane vs data-plane thinking.                                     |

The key bridge:

> “My background is not just ticket management. I have managed production operations where service health depends on compute, storage, network, monitoring, incident process, and customer communication. VMware-on-Nutanix is another complex distributed platform where the same operational discipline applies.”

That is a strong positioning statement.

---

## 10. Minimum I need to memorize

You should be able to say these confidently:

1. **ESXi is the VMware bare-metal hypervisor.**
2. **vCenter is the centralized management plane for ESXi hosts and clusters.**
3. **vSphere = the broader VMware virtualization platform, including ESXi and vCenter.**
4. **HA restarts VMs after host failure; vMotion moves running VMs between hosts.**
5. **DRS balances workloads using vMotion and policies.**
6. **A datastore stores VM files such as VMDKs, VMX files, logs, templates, and snapshots.**
7. **On Nutanix with VMware, storage is provided by AOS/CVMs, commonly presented as NFS datastores.**
8. **The Nutanix CVM is critical because it handles distributed storage services.**
9. **Prism manages Nutanix infrastructure; vCenter manages VMware hosts and VMs.**
10. **High datastore latency in vCenter is a symptom, not necessarily root cause.**
11. **A support manager must determine blast radius, coordinate owners, protect evidence, communicate clearly, and drive mitigation.**
12. **AHV is Nutanix’s native integrated hypervisor, managed through Prism.**

Memorize this one-liner:

> “In VMware-on-Nutanix, ESXi runs the VMs, vCenter manages the VMware cluster, and Nutanix AOS/CVMs provide the distributed storage layer. Effective troubleshooting means correlating symptoms across all three.”

---

## 11. Advanced / optional level

You do **not** need to master these deeply before the interview, but you should recognize them:

1. **VMkernel networking**

   * Management VMkernel.
   * vMotion VMkernel.
   * Storage/NFS VMkernel.
   * VLANs, MTU, NIC teaming.

2. **CPU Ready**

   * VMware metric indicating VMs waiting for CPU scheduling.
   * Important for performance troubleshooting.

3. **Memory ballooning / swapping**

   * Indicates memory pressure.
   * Can cause severe VM performance degradation.

4. **APD / PDL**

   * All Paths Down / Permanent Device Loss.
   * Storage connectivity conditions in VMware.

5. **EVC — Enhanced vMotion Compatibility**

   * Masks CPU features to allow vMotion across different CPU generations.

6. **VM snapshots**

   * Useful for short-term protection, but long snapshot chains can cause performance and consolidation issues.

7. **VM stun**

   * Temporary VM pause, often noticed during snapshot, backup, consolidation, or storage events.

8. **LCM / upgrade sequencing**

   * Nutanix Lifecycle Manager and upgrade workflows.
   * Nutanix documentation notes that when upgrading hypervisor hosts, customers must review VMware or Microsoft component compatibility and upgrade order, such as vCenter planning. ([Nutanix][12])

9. **Nutanix storage best practices**

   * Nutanix recommends NFS for vSphere datastores in many contexts and recommends not using Nutanix storage containers as generic-purpose NFS/SMB file shares; Nutanix Files should be used for file services. ([Nutanix][13])

10. **PSOD — Purple Screen of Death**

* ESXi kernel crash.
* Nutanix documentation notes ESXi uses `/scratch` for logs during PSOD/kernel dumps and warns that moving scratch from SATA DOM/M.2 to NFS or another datastore is not supported. ([Nutanix][14])

For your role, advanced knowledge is useful mainly to ask better questions and avoid being fooled by superficial symptoms.

---

## 12. Final checklist

Before an interview, you should be able to answer:

* [ ] What is ESXi?
* [ ] What is vCenter?
* [ ] What is vSphere?
* [ ] What is AHV?
* [ ] How does VMware run on Nutanix?
* [ ] What is the Nutanix CVM?
* [ ] What does Prism manage vs what vCenter manages?
* [ ] What is a datastore?
* [ ] Why does Nutanix commonly present storage to ESXi as NFS?
* [ ] What is HA?
* [ ] What is DRS?
* [ ] What is vMotion?
* [ ] What would you check during datastore latency?
* [ ] What would you check during ESXi host disconnection?
* [ ] What would you check if host maintenance mode fails?
* [ ] How would you manage a Sev1 involving VMware-on-Nutanix?
* [ ] How would you explain AHV to a VMware customer?
* [ ] How would you avoid cross-vendor blame?
* [ ] How would you communicate uncertainty to an enterprise customer?
* [ ] How would you position yourself as a technical escalation manager?

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword   | Meaning                                                                                  |
| -------------------------- | ---------------------------------------------------------------------------------------- |
| AHV                        | Acropolis Hypervisor; Nutanix native hypervisor.                                         |
| AOS                        | Acropolis Operating System; Nutanix distributed infrastructure software.                 |
| APD                        | All Paths Down; VMware condition where storage paths are unavailable.                    |
| Ballooning                 | VMware memory reclamation technique; can indicate memory pressure.                       |
| Bare-metal hypervisor      | Hypervisor installed directly on physical hardware.                                      |
| Blast radius               | Scope of impact: VM, host, cluster, site, or service.                                    |
| Broadcom                   | Current owner/vendor for VMware product documentation and support.                       |
| Cluster                    | Group of hosts managed together for availability and resource pooling.                   |
| Confluence                 | Documentation platform used for runbooks, notes, and knowledge base.                     |
| Controller VM              | Nutanix VM that provides storage and data services on each node.                         |
| CPU Ready                  | VMware metric showing time a VM waits for CPU scheduling.                                |
| CVM                        | Controller VM; Nutanix storage/data-services VM on every node.                           |
| Datastore                  | Logical storage location for VM files in VMware.                                         |
| Deduplication              | Storage optimization that removes duplicate data blocks.                                 |
| Distributed Storage Fabric | Nutanix distributed storage layer across cluster nodes.                                  |
| DRS                        | Distributed Resource Scheduler; balances VMs across ESXi hosts.                          |
| DSF                        | Distributed Storage Fabric; Nutanix distributed storage architecture.                    |
| EVC                        | Enhanced vMotion Compatibility; CPU compatibility mode for vMotion.                      |
| ESXi                       | VMware bare-metal hypervisor.                                                            |
| Fault Tolerance            | VMware feature for higher VM availability using a secondary VM.                          |
| Firmware                   | Low-level software on hardware devices such as NICs or disks.                            |
| Grafana                    | Monitoring and visualization tool.                                                       |
| HA                         | High Availability; restarts VMs after host failure.                                      |
| HCI                        | Hyperconverged Infrastructure; combines compute, storage, and virtualization.            |
| Hypervisor                 | Software layer that runs and isolates virtual machines.                                  |
| I/O                        | Input/output operations, usually storage or network traffic.                             |
| iSCSI                      | Block storage protocol over IP networks.                                                 |
| Jira                       | Issue and workflow tracking system.                                                      |
| Kibana                     | Log search and visualization tool.                                                       |
| LACP                       | Link Aggregation Control Protocol; bundles network links.                                |
| LCM                        | Lifecycle Manager; Nutanix tool for coordinated infrastructure updates.                  |
| Live Migration             | Moving a running VM between hosts without planned downtime.                              |
| Management plane           | Control and administration layer, such as vCenter or Prism.                              |
| Maintenance mode           | Host state used for planned work after evacuating workloads.                             |
| MTTR                       | Mean Time To Resolve/Recover; support recovery metric.                                   |
| MTU                        | Maximum Transmission Unit; maximum packet size on a network.                             |
| NCI                        | Nutanix Cloud Infrastructure.                                                            |
| NFS                        | Network File System; file protocol commonly used for VMware datastores on Nutanix.       |
| NIC                        | Network Interface Card.                                                                  |
| Noisy neighbor             | Workload consuming excessive shared resources.                                           |
| NTP                        | Network Time Protocol; time synchronization service.                                     |
| PDL                        | Permanent Device Loss; VMware condition where storage device is permanently unavailable. |
| Prism                      | Nutanix management interface for clusters and infrastructure.                            |
| Prism Central              | Centralized Nutanix management plane for multiple clusters.                              |
| Prism Element              | Nutanix management interface for a single cluster.                                       |
| PSOD                       | Purple Screen of Death; ESXi kernel crash screen.                                        |
| RCA                        | Root Cause Analysis.                                                                     |
| RF                         | Replication Factor; Nutanix data redundancy setting.                                     |
| RF2                        | Replication Factor 2; tolerates one component failure.                                   |
| RF3                        | Replication Factor 3; tolerates two component failures.                                  |
| SDDC                       | Software-Defined Data Center.                                                            |
| Sev1                       | Highest-severity incident, usually major production impact.                              |
| SLA                        | Service Level Agreement.                                                                 |
| Snapshot                   | Point-in-time VM or storage state; useful but risky if long-lived.                       |
| Storage container          | Nutanix logical storage construct presented to hypervisors.                              |
| Storage latency            | Delay in completing storage I/O operations.                                              |
| Support escalation         | Process of raising case urgency or involving higher expertise.                           |
| TAM                        | Technical Account Manager.                                                               |
| Thin provisioning          | Allocating storage on demand instead of reserving full capacity upfront.                 |
| VM                         | Virtual Machine.                                                                         |
| VMDK                       | VMware virtual disk file.                                                                |
| VMkernel                   | ESXi kernel networking/system layer used for management, vMotion, storage.               |
| VMX                        | VMware VM configuration file.                                                            |
| vMotion                    | VMware live migration of a running VM between hosts.                                     |
| vCenter Server             | Centralized VMware management server for ESXi environments.                              |
| vCPU                       | Virtual CPU assigned to a VM.                                                            |
| vDisk                      | Virtual disk used by a VM.                                                               |
| VLAN                       | Virtual LAN; network segmentation mechanism.                                             |
| vNIC                       | Virtual network interface card.                                                          |
| vSphere                    | VMware virtualization platform including ESXi and vCenter.                               |
| vSphere HA                 | VMware High Availability feature for VM restart after host failure.                      |
| vSphere Client             | Web UI used to manage vCenter and ESXi.                                                  |
| vSwitch                    | Virtual switch inside ESXi for VM and VMkernel networking.                               |
| Workload portability       | Ability to move workloads across platforms or environments.                              |

[1]: https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/7-0/vsphere-virtual-machine-administration-guide-7-0/introduction-to-vmware-vsphere-virtual-machinesvm-admin/virtual-machines-and-the-virtual-infrastructurevm-admin.html?utm_source=chatgpt.com "Virtual Machines and the Virtual Infrastructure"
[2]: https://www.nutanix.com/products/ahv "AHV: Virtualization Solution for Enterprise | Nutanix"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2092-vSphere-Storage%3ABP-2092-vSphere-Storage&utm_source=chatgpt.com "VMware vSphere Best Practices - Nutanix"
[4]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2092-vSphere-Storage%3Acontainers-for-vmware-vsphere.html&utm_source=chatgpt.com "Containers for VMware vSphere - portal.nutanix.com"
[5]: https://portal.nutanix.com/docs/vSphere-Admin6-AOS-v7_3%3Avsp-cluster-configuration-vcenter-vsphere-c.html?utm_source=chatgpt.com "AOS 7.3 - vCenter Configuration - portal.nutanix.com"
[6]: https://techdocs.broadcom.com/us/en/vmware-tanzu/data-solutions/tanzu-greenplum/7/greenplum-database/vsphere-vsphere-drs-ha-setup.html?utm_source=chatgpt.com "vSphere DRS and High Availability - VMware"
[7]: https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/7-0/vsphere-availability.html?utm_source=chatgpt.com "vSphere Availability - VMware"
[8]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_5%3Awc-storage-management-wc-c.html&utm_source=chatgpt.com "Prism 6.5 - Storage Management - Nutanix"
[9]: https://portal.nutanix.com/page/documents/details?targetId=Advanced-Admin-AOS%3Aapp-nutanix-cloud-infra-cvm-field-specifications-c.html&utm_source=chatgpt.com "AOS 7.5 - Controller VM (CVM) Specifications - Nutanix"
[10]: https://portal.nutanix.com/page/documents/details?targetId=vSphere-Admin6-AOS-v6_0%3Avsp-cluster-introduction-vsphere-c.html&utm_source=chatgpt.com "AOS 6.0 - Overview - Nutanix"
[11]: https://techdocs.broadcom.com/us/en/vmware-cis/vsphere/vsphere/8-0/vsphere-availability/creating-and-using-vsphere-ha-clusters/vsphere-ha-interoperability/using-vmware-ha-and-drs-together.html?utm_source=chatgpt.com "Using vSphere HA and DRS Together - techdocs.broadcom.com"
[12]: https://portal.nutanix.com/page/documents/details?targetId=vSphere-Admin6-AOS-v6_10%3Avsp-node-upgrade-vsphere-c.html&utm_source=chatgpt.com "AOS 6.10 - ESXi Host Upgrade - portal.nutanix.com"
[13]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2022_6%3Awc-storage-components-wc-c.html&utm_source=chatgpt.com "Prism pc.2022.6 - Storage Components - portal.nutanix.com"
[14]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2092-vSphere-Storage%3Abp-vsphere-storage-settings.html&utm_source=chatgpt.com "vSphere Storage Settings - portal.nutanix.com"
