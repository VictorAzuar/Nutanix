# AHV / Virtualization — Hypervisor Fundamentals

## 1. Short definition

A **hypervisor** is the software layer that allows multiple virtual machines to run on the same physical server by abstracting CPU, memory, storage, and networking resources.

**AHV — Acropolis Hypervisor —** is Nutanix’s native hypervisor, integrated with the Nutanix HCI platform, AOS, and Prism. Nutanix describes AHV as the default hypervisor option for Nutanix HCI, providing VM operations, live migration, high availability, virtual networking, and integrated management through Prism. ([Nutanix][1])

For your interview, the key message is:

> “AHV is not just a standalone hypervisor. In Nutanix, it is part of an integrated HCI operating model where virtualization, distributed storage, networking, resiliency, and management are tightly coupled.”

---

## 2. Clear explanation

Traditional virtualization separates the physical server from the virtual machines running on it. A physical host provides **CPU cores, RAM, physical NICs, and disks**. The hypervisor creates virtual equivalents: **vCPUs, vRAM, vNICs, and virtual disks**.

In a Nutanix environment, this is combined with **hyperconverged infrastructure**. Instead of having separate compute hosts connected to an external SAN, each Nutanix node contributes compute and storage to a distributed cluster. AHV runs the VMs, while **AOS** provides distributed storage and platform services.

The basic stack looks like this:

```text
Applications
   ↓
Guest OS inside VM
   ↓
Virtual hardware: vCPU, vRAM, vNIC, vDisk
   ↓
AHV hypervisor
   ↓
Nutanix AOS / Distributed Storage Fabric
   ↓
Physical Nutanix nodes: CPU, RAM, disks, NICs
```

A few important Nutanix-specific points:

AHV uses the **Nutanix Distributed Storage Fabric** to deliver storage services such as provisioning, snapshots, clones, and data protection to VMs. Nutanix documentation states that in AHV clusters, AOS presents disks to VMs as raw SCSI block devices and uses an iSCSI redirector on each AHV host to establish resilient storage paths across the cluster. ([Nutanix][1])

AHV also includes features that a virtualization administrator would expect: **VM lifecycle operations, live migration, high availability, virtual networking, and dynamic scheduling**. Nutanix positions AHV as easy to transition to from legacy virtualization platforms because the operational workflows are familiar. ([Nutanix][1])

The important distinction for you as a **support manager** is that you do not need to sound like a kernel engineer. You need to show that you understand where problems can occur:

```text
User / Application issue
Guest OS issue
VM resource issue
Hypervisor issue
Storage path issue
Network issue
Cluster health issue
Hardware issue
```

In enterprise support, the skill is not only knowing what a hypervisor is. The skill is knowing how to **scope the fault domain** quickly.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, AHV fundamentals matter because many critical escalations will involve ambiguity between:

* Application performance problem
* Guest OS problem
* VM configuration problem
* Hypervisor scheduling problem
* Storage latency problem
* Network packet loss or VLAN issue
* Cluster health or node issue
* Upgrade, compatibility, or firmware issue

Nutanix support teams need to help customers restore service quickly while coordinating across SREs, engineering, account teams, and sometimes third-party vendors such as VMware, Microsoft, backup vendors, network vendors, or cloud providers.

AHV matters specifically because it is Nutanix’s native virtualization layer. It is integrated with **Prism**, **AOS**, **LCM**, **NCC**, storage services, and cluster operations. The Acropolis Advanced Administration Guide also describes AOS as providing VM-centric storage, container-centric storage, Nutanix Volumes, Nutanix Files, storage optimization, data protection, DR, networking services, and Lifecycle Manager. ([Nutanix][2])

For the interview, your positioning should be:

> “I do not need to pretend I am a Senior SRE specializing in AHV internals. My value is that I can understand the architecture, ask the right triage questions, separate customer impact from technical noise, coordinate the right specialists, and drive the escalation toward restoration, root cause, and prevention.”

That is exactly the profile of a technical escalation manager.

---

## 4. Key concepts

### Hypervisor

The software layer that allows multiple VMs to run on one physical server.

In interviews, say:

> “The hypervisor abstracts physical compute, memory, network, and storage resources into virtual resources that can be assigned to VMs.”

---

### Type 1 vs Type 2 hypervisor

A **Type 1 hypervisor** runs directly on the physical hardware. Examples include AHV, VMware ESXi, Microsoft Hyper-V, and KVM-based platforms.

A **Type 2 hypervisor** runs on top of a general-purpose operating system, such as VMware Workstation, VirtualBox, or Parallels.

For Nutanix interviews, focus on **Type 1 hypervisors**.

---

### AHV

**AHV** is Nutanix’s native hypervisor. It provides VM operations, live migration, HA, virtual networking, and integration with Prism. Nutanix describes AHV as the default option for Nutanix HCI and part of an integrated virtualization, networking, infrastructure, and operations management model. ([Nutanix][1])

Interview phrasing:

> “AHV is the virtualization layer of the Nutanix platform. It runs the VMs, while AOS provides the distributed storage and platform services underneath.”

---

### AOS

**AOS — Acropolis Operating System —** is the distributed software layer that provides storage and platform services across Nutanix nodes.

For your level, remember:

> “AHV runs the VM. AOS provides the distributed storage and cluster services behind it.”

---

### Prism Element and Prism Central

**Prism** is the Nutanix management interface.

* **Prism Element** manages an individual cluster.
* **Prism Central** provides centralized management across clusters.

For support, Prism is where customers and engineers check alerts, VM status, cluster health, performance, networking, and lifecycle operations.

---

### CVM — Controller VM

Each Nutanix node runs a **Controller VM**. The CVM is central to Nutanix architecture because it participates in the distributed storage and cluster services.

For interviews:

> “The CVM is not a customer workload VM. It is part of the Nutanix control and data services architecture.”

---

### VM lifecycle operations

Common VM operations include:

* Create VM
* Power on/off
* Reboot
* Clone
* Snapshot
* Migrate
* Resize CPU/RAM
* Attach or detach disks
* Attach or change networks
* Delete VM

These are standard virtualization workflows and are part of the operational surface support teams must understand.

---

### vCPU and CPU scheduling

A VM receives one or more **vCPUs**. The hypervisor schedules those vCPUs onto physical CPU cores.

Support relevance:

* Too many vCPUs can cause scheduling contention.
* CPU ready-like symptoms may appear as application slowness.
* A VM with high CPU utilization may be constrained by actual demand or by poor sizing.
* Noisy-neighbor workloads can affect performance if resource contention exists.

You do not need to go too deep into scheduler internals, but you should understand the operational symptoms.

---

### vRAM and memory overcommit

A VM receives allocated memory. Hypervisors may allow some degree of memory overcommit depending on platform and configuration.

Support relevance:

* Memory pressure can cause guest OS swapping.
* Swapping usually causes severe performance degradation.
* Performance issues may be misdiagnosed as storage problems when the real issue is memory pressure inside the guest.

---

### vDisk

A **vDisk** is a virtual disk presented to a VM. In AHV, Nutanix documentation states that AOS passes disks to VMs as raw SCSI block devices and uses an optimized I/O path through Nutanix storage services. ([Nutanix][1])

Support relevance:

* Disk latency may come from guest OS, virtual disk configuration, storage path, cluster health, physical disk issues, or workload behavior.
* You need to distinguish “the VM is slow” from “the storage layer is degraded.”

---

### vNIC

A **vNIC** is a virtual network interface assigned to a VM.

Support relevance:

* Wrong VLAN
* Missing IP configuration
* MTU mismatch
* Packet loss
* DNS issue
* Firewall/security policy
* Physical uplink issue
* Virtual switch/network configuration issue

Network problems often appear as application outages.

---

### Live migration

Live migration moves a running VM from one host to another with minimal disruption.

Why it matters:

* Maintenance
* Load balancing
* Host evacuation
* Hardware replacement
* Cluster optimization

Nutanix documentation lists live migration as one of the familiar AHV operational workflows. ([Nutanix][1])

---

### High Availability

**HA** allows workloads to restart or recover when a host fails.

For support interviews:

> “HA is about maintaining service availability when infrastructure components fail. In a real escalation, I would verify whether the workload restarted, whether capacity was available, whether the failure domain behaved as expected, and what residual customer impact remains.”

---

### Dynamic scheduling

Nutanix AHV includes **Acropolis Dynamic Scheduling**, which monitors cluster resources and helps with VM placement and resource contention avoidance. Nutanix documentation describes dynamic scheduling as part of AHV’s resiliency and operational feature set. ([Nutanix][1])

Support relevance:

* VM placement
* Hotspot avoidance
* Resource contention
* Maintenance operations
* Balancing workloads across hosts

---

### Storage path

In AHV, storage is not simply “local disk.” It is delivered through the Nutanix distributed storage architecture.

Nutanix documentation states that each AHV host runs an iSCSI redirector, which establishes a resilient path from each VM to storage across the cluster, preferably redirecting to a healthy local Stargate service. ([Nutanix][1])

Support relevance:

* Storage latency can be caused by host issues, CVM issues, disk issues, replication, rebuild activity, network issues, or workload spikes.
* You need to coordinate storage, virtualization, and possibly networking specialists.

---

### VirtIO

**VirtIO** provides paravirtualized drivers commonly used for better VM I/O performance.

Nutanix documentation notes that for maximum AHV Turbo VM performance, Windows VMs should have the latest Nutanix VirtIO package installed. ([Nutanix][1])

Support relevance:

* Missing or outdated drivers can affect disk or network performance.
* Driver compatibility can become important after migrations or upgrades.

---

### AHV Turbo

AHV Turbo improves storage I/O performance by bypassing QEMU for the I/O path and using a multi-queue approach. Nutanix documentation states that this can lower CPU usage and increase storage I/O available to VMs, and that it is transparent to VMs and enabled by default on AHV clusters. ([Nutanix][1])

You probably do not need to memorize implementation details, but you should know that AHV has optimized data path capabilities.

---

## 5. How it appears in a real escalation

### Scenario 1 — Critical VM performance degradation

A customer reports:

> “Our ERP database VM running on Nutanix AHV is extremely slow. Users are blocked globally.”

Possible technical domains:

* Guest OS CPU saturation
* Memory pressure or swapping
* Storage latency
* vDisk queue depth
* Noisy neighbor
* CVM issue
* Host hardware issue
* Network issue between application and database
* Recent change: upgrade, migration, backup, snapshot, antivirus, batch job

Your role:

1. Confirm business impact.
2. Establish timeline.
3. Identify affected workloads.
4. Check whether the issue is isolated or cluster-wide.
5. Engage virtualization/storage/network SREs.
6. Drive parallel investigation.
7. Keep customer communication structured.
8. Push toward mitigation before perfect RCA.

Strong manager-level response:

> “I would first separate impact management from root-cause analysis. While the SREs check Prism alerts, host health, VM metrics, storage latency, and recent changes, I would ensure the customer gets a clear communication cadence and that we evaluate immediate mitigation options such as moving the workload, reducing contention, reverting recent changes, or isolating the affected component.”

---

### Scenario 2 — VM unavailable after host failure

Customer reports:

> “A host failed, and some VMs did not come back as expected.”

Possible areas:

* HA configuration
* Cluster capacity
* Host failure detection
* VM restart priority
* Storage availability
* Network reachability after restart
* Application dependency order
* Customer expectation mismatch

Your response:

> “I would verify whether the cluster had enough failover capacity, whether HA behaved as designed, which VMs were affected, whether they restarted on other nodes, and whether the remaining issue is VM power state, OS boot, application dependency, storage, or network access.”

---

### Scenario 3 — Post-upgrade instability

Customer reports:

> “After an AOS/AHV upgrade, some VMs have intermittent network or disk performance issues.”

Possible areas:

* Compatibility matrix
* Firmware/driver versions
* VirtIO drivers
* LCM status
* NCC checks
* Known issues
* Third-party backup/security agents
* Network driver or physical NIC firmware

Nutanix documentation highlights Lifecycle Manager as the framework that tracks software and firmware versions and supports updates for platforms using Nutanix software. ([Nutanix][2])

Your manager-level contribution:

> “I would make sure we correlate the start of symptoms with the change window, validate compatibility and firmware/driver levels, review health checks, and quickly determine whether we need rollback, workaround, vendor escalation, or engineering engagement.”

---

### Scenario 4 — Customer asks whether AHV replaces VMware

This is likely in a business or architecture discussion.

A balanced answer:

> “AHV can replace VMware for many enterprise workloads, but the decision depends on operational maturity, ecosystem dependencies, backup tooling, automation, network design, application certification, and migration risk. From a support perspective, I would focus on workload readiness, supportability, compatibility, and operational runbooks rather than presenting it as a simple hypervisor swap.”

---

## 6. Triage questions I should ask

### Business impact

1. What service or application is affected?
2. How many users, sites, or customers are impacted?
3. Is this production, pre-production, or DR?
4. Is the impact outage, degradation, intermittent errors, or risk only?
5. Is there a regulatory, financial, or executive escalation attached?

### Scope

6. Is one VM affected, multiple VMs, one host, one cluster, or multiple clusters?
7. Are only AHV workloads affected, or also external systems?
8. Is the issue isolated to compute, storage, network, or management plane?
9. Are Prism alerts showing cluster-wide health problems?
10. Are other workloads on the same host showing similar symptoms?

### Timeline

11. When did the issue start?
12. Was there a recent change?
13. Any AOS/AHV/firmware upgrade?
14. Any VM migration?
15. Any backup, snapshot, restore, antivirus scan, batch job, or replication event?

### VM-level checks

16. What are CPU, memory, disk, and network metrics for the VM?
17. Is the guest OS reporting CPU saturation, swapping, disk queueing, or network errors?
18. Are VirtIO drivers installed and current where relevant?
19. Did the VM recently change size or configuration?
20. Is the issue inside the guest OS or visible from the hypervisor layer?

### Host / cluster checks

21. Which host is the VM running on?
22. Are there host alerts?
23. Is there resource contention?
24. Is dynamic scheduling moving workloads?
25. Is HA configured and healthy?
26. Is there enough cluster capacity?
27. Are any nodes down, degraded, or under maintenance?

### Storage checks

28. Is there elevated storage latency?
29. Is the issue read, write, or both?
30. Are there disk, CVM, Stargate, replication, or rebuild alerts?
31. Are snapshots, clones, backups, or replication tasks running?
32. Is the storage issue isolated to one VM or cluster-wide?

### Network checks

33. Can the VM reach its gateway?
34. Is the correct VLAN assigned?
35. Any MTU, DNS, firewall, routing, or security policy change?
36. Are other VMs on the same network affected?
37. Are physical uplinks or ToR switches healthy?

### Escalation management

38. Who is the customer technical owner?
39. Who is the business decision-maker?
40. What communication cadence is expected?
41. What is the next mitigation decision point?
42. Which team owns the next action: support, SRE, engineering, customer, or third-party vendor?

---

## 7. Likely interview questions

1. What is a hypervisor?
2. What is the difference between AHV, AOS, and Prism?
3. How would you explain AHV to a customer coming from VMware?
4. What are the key components involved when a VM runs on Nutanix?
5. What could cause a VM performance issue on AHV?
6. How would you triage a critical VM outage?
7. How would you distinguish between a guest OS issue and a hypervisor issue?
8. What is live migration, and why does it matter?
9. What is high availability in a virtualization platform?
10. What would you check if a VM did not restart after a host failure?
11. How do storage and virtualization interact in Nutanix HCI?
12. What is the role of the CVM?
13. What would you check after an AHV or AOS upgrade if customers report degraded performance?
14. How would you manage communication during a P1 escalation?
15. How technical should a support manager be in this type of environment?
16. What is your gap with Nutanix technology, and how are you closing it?
17. How would you coordinate between SREs, engineering, account teams, and the customer?
18. How would you handle a customer blaming Nutanix when the issue may be in VMware, networking, backup, or the guest OS?
19. What metrics would you use to manage enterprise support quality?
20. How do you prevent repeated escalations after the immediate incident is resolved?

---

## 8. Model answers in English

### Q1. What is a hypervisor?

**Model answer:**

> “A hypervisor is the software layer that abstracts physical server resources — CPU, memory, storage, and networking — and presents them as virtual resources to virtual machines. In an enterprise environment, the hypervisor is critical because it allows workload consolidation, mobility, high availability, and operational flexibility. From a support perspective, it is also an important fault domain: when a VM is slow or unavailable, we need to determine whether the issue is inside the guest OS, in the VM configuration, in the hypervisor, in storage, in networking, or in the underlying hardware.”

---

### Q2. How would you explain AHV?

**Model answer:**

> “AHV is Nutanix’s native hypervisor. It provides the virtualization layer for running VMs on Nutanix infrastructure. What makes it different from looking at a standalone hypervisor is that AHV is integrated with Nutanix AOS, distributed storage, networking, high availability, dynamic scheduling, and Prism management. So when I think about AHV support, I do not only think about VM power operations. I think about the full HCI stack: VM, host, CVM, storage path, network, cluster health, lifecycle, and customer impact.”

---

### Q3. What is the difference between AHV, AOS, and Prism?

**Model answer:**

> “AHV is the hypervisor that runs the virtual machines. AOS is the distributed software layer that provides storage and platform services across the Nutanix cluster. Prism is the management and operations interface used to monitor, configure, and operate the environment. In simple terms: AHV runs the workloads, AOS provides the distributed infrastructure services, and Prism gives administrators the operational control plane.”

---

### Q4. How would you triage a VM performance issue on AHV?

**Model answer:**

> “I would start by defining the impact and scope: which application, how many users, one VM or many VMs, one host or the full cluster. Then I would check the timeline and recent changes: upgrades, migrations, snapshots, backups, network changes, or workload spikes. Technically, I would split the investigation into guest OS metrics, VM resource allocation, host contention, storage latency, network health, and cluster alerts in Prism. As a manager, I would ensure parallel workstreams: one focused on mitigation, one on technical diagnosis, and one on customer communication. The goal is to reduce MTTR first, then complete RCA and prevention.”

---

### Q5. How would you distinguish a guest OS issue from a hypervisor issue?

**Model answer:**

> “I would compare what the guest OS sees with what the platform sees. If the guest reports high CPU, memory swapping, disk queueing, or application errors, but the hypervisor and cluster are healthy, the issue may be inside the VM or application stack. If multiple VMs on the same host or cluster show similar symptoms, or Prism shows host, storage, or network alerts, then the issue may be at the infrastructure layer. I would avoid jumping to conclusions and would correlate metrics across guest, VM, host, storage, and network layers.”

---

### Q6. What would you check if a customer says ‘AHV is slow’?

**Model answer:**

> “I would first make the statement measurable. ‘Slow’ could mean high application latency, long login times, poor disk performance, packet loss, CPU saturation, or delayed VM operations. I would ask for affected workloads, timing, user impact, recent changes, and whether the issue is isolated or widespread. Then I would check VM CPU, memory, disk latency, network metrics, host utilization, cluster health, storage alerts, and ongoing background operations such as backups, snapshots, replication, or rebuilds. I would also make sure the communication with the customer is structured, because vague performance escalations can become emotional unless we turn them into evidence-based workstreams.”

---

### Q7. What is live migration and why does it matter?

**Model answer:**

> “Live migration is the ability to move a running VM from one physical host to another with minimal disruption. It matters because it supports maintenance, load balancing, hardware replacement, and operational resiliency. In a support scenario, live migration can be a mitigation tool if a VM is affected by host-level contention or maintenance risk. However, I would also verify that the root cause is understood, because moving the VM may reduce impact without solving the underlying issue.”

---

### Q8. What is HA in a virtualization platform?

**Model answer:**

> “High availability is the platform capability to recover workloads when infrastructure components fail, typically by restarting VMs on healthy hosts if a host becomes unavailable. In an enterprise escalation, I would look at whether HA was configured correctly, whether the cluster had sufficient capacity, which VMs were affected, whether they restarted as expected, and whether the remaining issue is infrastructure, guest OS, application dependency, or customer expectation.”

---

### Q9. How does Nutanix HCI change the way you troubleshoot virtualization?

**Model answer:**

> “In a traditional architecture, compute, storage, and networking are often managed as separate layers. In Nutanix HCI, those layers are more integrated. That simplifies operations in many ways, but in support it also means we need to understand the interactions between VM, AHV host, CVM, distributed storage, network, and Prism. For troubleshooting, I would avoid looking at the hypervisor in isolation. I would look at the full service path from application to guest OS, VM, hypervisor, storage fabric, physical network, and cluster health.”

---

### Q10. How do you compensate for not being a Nutanix deep specialist yet?

**Model answer:**

> “I would be transparent: my current strength is leading enterprise support operations, 24/7 incident management, escalation handling, SLA and MTTR ownership, team coaching, and cross-functional coordination. I am actively closing the Nutanix-specific gap by studying AHV, AOS, Prism, HCI, storage, networking, and escalation patterns. For this role, I do not position myself as a Senior SRE individual contributor. I position myself as a technical escalation manager who can understand the architecture, ask the right questions, coordinate specialists, communicate clearly with customers, and drive incidents to resolution and prevention.”

---

### Q11. How would you manage a P1 escalation involving AHV?

**Model answer:**

> “I would establish three parallel tracks. First, impact management: confirm affected services, users, severity, business risk, and required communication cadence. Second, technical mitigation: assign owners to check Prism alerts, VM metrics, host health, storage latency, network reachability, recent changes, and possible workarounds. Third, stakeholder management: keep the customer, account team, support leadership, and engineering aligned with facts, next actions, and decision points. My focus would be MTTR, clarity, ownership, and avoiding duplicated or contradictory troubleshooting.”

---

### Q12. How would you handle a customer blaming Nutanix when the issue may be third-party?

**Model answer:**

> “I would avoid a defensive posture. I would acknowledge the impact and keep Nutanix accountable for driving the investigation within our scope. At the same time, I would structure the evidence: what the Nutanix platform shows, what the guest OS shows, what the network path shows, and what third-party components are involved. If the evidence points to a backup agent, firewall, VMware component, OS driver, or application layer, I would coordinate with that vendor or customer team while keeping a single escalation rhythm. The customer should feel that we are leading the resolution, not just redirecting blame.”

---

## 9. Connection with my experience

Your Harmonic background maps well to this topic.

You already understand:

* 24/7 support operations
* Incident command
* Escalation management
* SLA and MTTR ownership
* Customer communication
* Cloud operations
* Monitoring and observability
* Jira / Confluence workflows
* Support KPIs
* Technical coordination across teams

You can translate your SaaS/cloud experience into Nutanix language like this:

| Your experience                     | Nutanix / AHV equivalent                                          |
| ----------------------------------- | ----------------------------------------------------------------- |
| Cloud service degradation           | VM / cluster / storage / network degradation                      |
| Incident bridge                     | P1 escalation bridge with support, SRE, engineering, account team |
| Monitoring in Grafana/Kibana        | Prism metrics, alerts, logs, NCC checks, platform health          |
| SLA / MTTR                          | Enterprise support response, restoration, escalation quality      |
| Kubernetes node/pod troubleshooting | VM/host/cluster troubleshooting                                   |
| Cloud capacity management           | Cluster capacity, host utilization, VM sizing                     |
| Change management                   | AOS/AHV/firmware/driver lifecycle operations                      |
| Customer-facing RCA                 | Technical RCA with corrective and preventive actions              |

A strong positioning line:

> “My previous environment was SaaS and cloud operations, but the operational discipline is highly transferable: define impact, isolate fault domains, coordinate specialists, communicate clearly, restore service, and prevent recurrence. What I am adding now is Nutanix-specific fluency around AHV, AOS, Prism, HCI storage, virtualization, and enterprise infrastructure escalation patterns.”

---

## 10. Minimum I need to memorize

Memorize these points:

1. **AHV is Nutanix’s native hypervisor.**
2. **AOS provides the distributed storage and platform services.**
3. **Prism is the management interface.**
4. **CVMs are core Nutanix service VMs, not customer workload VMs.**
5. **AHV supports VM operations, live migration, HA, virtual networking, and dynamic scheduling.**
6. **Nutanix HCI combines compute, storage, virtualization, and management in an integrated platform.**
7. **A VM performance issue can come from guest OS, VM sizing, host contention, storage latency, network, cluster health, hardware, or recent changes.**
8. **Support managers must drive impact reduction, communication, ownership, and escalation discipline.**
9. **Do not claim to be a deep AHV internals expert; position yourself as a technical escalation manager.**
10. **Always triage by impact, scope, timeline, recent changes, and fault domain.**

Your 30-second version:

> “AHV is Nutanix’s native Type 1 hypervisor for running virtual machines on Nutanix HCI. It integrates with AOS, which provides distributed storage and platform services, and with Prism for management. From a support perspective, the important part is understanding the full path: VM, guest OS, hypervisor, host, CVM, storage fabric, network, and cluster health. In an escalation, I would focus on impact, scope, recent changes, metrics correlation, mitigation, customer communication, and proper engagement of SRE or engineering specialists.”

---

## 11. Advanced / optional level

You can leave these as advanced unless the SRE interview goes deeper:

* AHV internal architecture
* KVM/QEMU implementation details
* AHV Turbo internals
* iSCSI redirector deep dive
* Stargate service internals
* Advanced CPU scheduling behavior
* NUMA tuning
* Advanced processor compatibility
* Deep packet-level network troubleshooting
* Open vSwitch internals
* Microsegmentation with Flow
* Detailed storage replication internals
* Performance tuning for specific workloads such as Oracle, SQL Server, VDI, or SAP HANA
* Disaster recovery architecture
* Backup integration with third-party vendors
* API/automation with Prism Central

You should be able to recognize these terms and ask intelligent questions, but you do not need to present yourself as the person who personally debugs AHV internals.

Good phrase:

> “I understand the operational relevance of those areas, and I would involve the right SRE or engineering expert for deep analysis while keeping ownership of the escalation process, customer communication, and decision flow.”

---

## 12. Final checklist

Before the interview, you should be able to explain:

* What a hypervisor does.
* What AHV is.
* How AHV relates to AOS and Prism.
* What a CVM is.
* Why HCI changes the troubleshooting model.
* What happens when a VM is slow.
* What happens when a host fails.
* What live migration is.
* What HA is.
* What dynamic scheduling does at a high level.
* Why storage and networking are critical to virtualization escalations.
* How to structure a P1 escalation.
* How to communicate with enterprise customers under pressure.
* How your Harmonic experience maps to Nutanix support leadership.
* How to be transparent about your Nutanix-specific learning curve without sounding weak.

Final positioning statement:

> “My strength is not pretending to be the deepest AHV engineer in the room. My strength is being technical enough to understand the architecture, precise enough to ask the right diagnostic questions, experienced enough to manage high-pressure escalations, and disciplined enough to drive customer impact reduction, communication, RCA, and prevention.”

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword   | Meaning                                                                                               |
| -------------------------- | ----------------------------------------------------------------------------------------------------- |
| AHV                        | Acropolis Hypervisor; Nutanix’s native hypervisor.                                                    |
| AHV Turbo                  | AHV optimized I/O path designed to improve VM storage performance.                                    |
| AOS                        | Acropolis Operating System; Nutanix distributed software layer for storage and platform services.     |
| API                        | Application Programming Interface; used for automation and integration.                               |
| Backup                     | Process of copying data or VMs for recovery.                                                          |
| Cluster                    | Group of Nutanix nodes operating as one system.                                                       |
| Compute                    | CPU and memory resources used to run workloads.                                                       |
| Controller VM              | Nutanix service VM running on each node; also called CVM.                                             |
| CPU                        | Central Processing Unit; physical compute resource.                                                   |
| CPU contention             | Competition for physical CPU resources between workloads.                                             |
| CVM                        | Controller VM; Nutanix VM responsible for cluster and storage services.                               |
| Data path                  | Route taken by application or VM I/O through the infrastructure.                                      |
| Distributed Storage Fabric | Nutanix distributed storage architecture providing storage services to VMs.                           |
| DNS                        | Domain Name System; resolves names to IP addresses.                                                   |
| DR                         | Disaster Recovery; recovery capability after a major failure.                                         |
| Dynamic scheduling         | Automated workload placement and balancing based on resource conditions.                              |
| Enterprise support         | Support model for business-critical customer environments.                                            |
| Escalation                 | Process of raising an issue to higher technical or management attention.                              |
| Fault domain               | Area where a failure may originate, such as VM, host, storage, or network.                            |
| Firmware                   | Low-level software running on hardware components.                                                    |
| Flow                       | Nutanix networking/security capability, including microsegmentation features.                         |
| Guest OS                   | Operating system running inside a VM.                                                                 |
| HA                         | High Availability; capability to recover workloads after infrastructure failure.                      |
| HCI                        | Hyperconverged Infrastructure; integrated compute, storage, networking, and virtualization.           |
| Host                       | Physical server running the hypervisor and VMs.                                                       |
| Hypervisor                 | Software layer that runs and manages virtual machines.                                                |
| iSCSI                      | Internet Small Computer Systems Interface; protocol used to transport block storage over IP networks. |
| I/O                        | Input/Output; read and write operations to disk or network.                                           |
| IPMI                       | Intelligent Platform Management Interface; out-of-band server management interface.                   |
| Jira                       | Ticketing and workflow system often used for support and engineering tracking.                        |
| KMS                        | Key Management Server; manages encryption keys.                                                       |
| KVM                        | Kernel-based Virtual Machine; Linux virtualization technology.                                        |
| LCM                        | Lifecycle Manager; Nutanix framework for software and firmware lifecycle operations.                  |
| Live migration             | Moving a running VM between hosts with minimal disruption.                                            |
| MTTR                       | Mean Time To Repair/Restore; key incident management metric.                                          |
| MTU                        | Maximum Transmission Unit; largest packet size on a network path.                                     |
| NCC                        | Nutanix Cluster Check; health-check tool for Nutanix clusters.                                        |
| NIC                        | Network Interface Card; physical network adapter.                                                     |
| Node                       | Physical Nutanix server contributing resources to a cluster.                                          |
| Noisy neighbor             | Workload consuming excessive shared resources and affecting others.                                   |
| NUMA                       | Non-Uniform Memory Access; CPU/memory topology relevant to performance tuning.                        |
| OVS                        | Open vSwitch; virtual switching technology commonly associated with KVM-based platforms.              |
| P1                         | Priority 1 incident; usually critical production impact.                                              |
| Prism                      | Nutanix management interface.                                                                         |
| Prism Central              | Centralized Nutanix management across multiple clusters.                                              |
| Prism Element              | Nutanix management interface for an individual cluster.                                               |
| QEMU                       | Emulator/virtualization component used in KVM-based virtualization stacks.                            |
| RCA                        | Root Cause Analysis; explanation of what failed and how to prevent recurrence.                        |
| Replication                | Copying data to another location or system for availability or DR.                                    |
| SCSI                       | Small Computer System Interface; storage command protocol.                                            |
| SLA                        | Service Level Agreement; contractual or operational service commitment.                               |
| Snapshot                   | Point-in-time copy of VM or data state.                                                               |
| SRE                        | Site Reliability Engineer; role focused on reliability, automation, and operations.                   |
| Stargate                   | Nutanix service involved in storage I/O handling.                                                     |
| Storage latency            | Delay in reading from or writing to storage.                                                          |
| ToR switch                 | Top-of-Rack switch connecting servers in a rack.                                                      |
| Type 1 hypervisor          | Hypervisor running directly on physical hardware.                                                     |
| Type 2 hypervisor          | Hypervisor running on top of a general-purpose operating system.                                      |
| vCPU                       | Virtual CPU assigned to a VM.                                                                         |
| vDisk                      | Virtual disk assigned to a VM.                                                                        |
| VLAN                       | Virtual Local Area Network; logical network segmentation.                                             |
| VM                         | Virtual Machine; software-defined server running on a hypervisor.                                     |
| VM lifecycle               | Operations such as create, power on, migrate, clone, snapshot, resize, and delete.                    |
| vNIC                       | Virtual network interface assigned to a VM.                                                           |
| vRAM                       | Virtual memory assigned to a VM.                                                                      |
| VirtIO                     | Paravirtualized driver framework used to improve VM I/O performance.                                  |
| Workload                   | Application, service, or VM consuming infrastructure resources.                                       |

[1]: https://www.nutanix.com/content/dam/nutanix/documents/certifications/ahv-admin.pdf "AHV Administration Guide"
[2]: https://www.nutanix.com/content/dam/nutanix/documents/certifications/advanced-admin-aos.pdf "Acropolis Advanced Administration Guide"
