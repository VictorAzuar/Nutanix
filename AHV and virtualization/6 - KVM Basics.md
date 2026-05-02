# AHV / Virtualization — KVM Basics

## 1. Short definition

**KVM — Kernel-based Virtual Machine — is the Linux kernel virtualization technology that allows a physical server to run multiple virtual machines.** In Nutanix AHV, KVM is one of the core underlying technologies, together with QEMU and Nutanix’s own AOS/AHV management and storage integration layers. Nutanix’s AHV architecture is based on **Linux, QEMU, and KVM**, and guest VMs run using hardware-assisted virtualization. 

For interview purposes, explain it like this:

> “KVM is the Linux-based virtualization engine. AHV uses KVM and QEMU as foundational components, but Nutanix adds enterprise management, storage integration, high availability, lifecycle operations, and Prism-based administration on top. As a Support Manager, I don’t need to debug kernel code, but I need to understand how VM execution, host resources, storage I/O, networking, and live migration interact during incidents.”

---

## 2. Clear explanation

At a basic level, virtualization separates **physical resources** from **logical workloads**.

A physical Nutanix node has:

* CPU
* Memory
* Physical NICs
* Local disks / SSDs / NVMe
* A hypervisor layer
* Controller VM, or **CVM**
* User VMs

In AHV, the hypervisor runs the customer’s virtual machines, while the Nutanix **Controller VM** provides the distributed storage and data services layer. Nutanix documentation describes AHV deployments where the **CVM runs as a VM**, and the physical storage controller is presented to the CVM using **PCI passthrough**, allowing the CVM to access storage hardware directly and bypass the hypervisor for storage device access. 

### KVM’s role

KVM itself is a Linux kernel module. It turns Linux into a hypervisor by allowing virtual machines to use CPU virtualization extensions such as Intel VT-x or AMD-V.

In the AHV/KVM stack, three components are especially important:

| Component              | Practical meaning                                                                                                                                         |
| ---------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **KVM kernel module**  | Provides the kernel-level virtualization capability.                                                                                                      |
| **QEMU / qemu-kvm**    | Emulates or virtualizes VM hardware and runs VM processes in user space.                                                                                  |
| **libvirt / libvirtd** | Management API/daemon used to control KVM/QEMU domains. Nutanix documentation states that communication between AOS and KVM/QEMU occurs through libvirt.  |

### How to explain the stack verbally

A good interview explanation:

> “KVM provides the low-level virtualization capability in the Linux kernel. QEMU provides the VM hardware model and runs the VM process. libvirt provides the management interface used to create, start, stop, migrate, and configure VMs. AHV uses these components but wraps them inside a Nutanix-managed enterprise virtualization platform, integrated with Prism, AOS, distributed storage, networking, HA, and lifecycle management.”

### Important distinction: KVM vs AHV

Do **not** say “AHV is just KVM.”

A better version:

> “AHV is not just vanilla KVM. KVM is the virtualization foundation, but AHV is the Nutanix enterprise hypervisor platform. The value comes from the integration with AOS, Prism, DSF, VM HA, live migration, networking, automation, and supportability.”

Nutanix positions AHV as an enterprise-grade hypervisor integrated into **Nutanix Cloud Infrastructure**, managed through **Prism**, and not licensed as a separate standalone hypervisor product. ([Nutanix][1])

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, this topic matters because many critical escalations will appear as “application down” or “VM slow,” but the actual problem may sit somewhere in the virtualization layer.

You need enough fluency to coordinate the right technical investigation across:

* AHV / hypervisor
* CVM / AOS
* Storage I/O
* Networking
* Guest OS
* Prism management plane
* Customer application team
* Third-party vendors, especially VMware, Microsoft, backup vendors, security tools, and networking vendors

Nutanix markets AHV as a modern virtualization platform for VMs across datacenter, edge, and public-cloud-oriented environments. It emphasizes enterprise capabilities such as HA, live migration, disaster recovery, performance integration with the Nutanix Distributed Storage Fabric, and Prism-based unified management. ([Nutanix][1])

For you, the managerial value is not to behave like the deepest AHV engineer. It is to:

* Understand the escalation domain.
* Ask the right triage questions.
* Detect whether the issue is compute, memory, storage, network, guest OS, or management plane.
* Manage severity, customer communication, and internal coordination.
* Avoid misrouting escalations.
* Translate complex technical findings into customer-impact language.
* Know when to involve senior SREs, engineering, account teams, or third-party vendors.

A strong support manager should be able to say:

> “We need to separate VM symptoms from hypervisor symptoms. If one VM is affected, I want to inspect guest OS, VM configuration, vCPU/memory pressure, and virtual NIC/disk behavior. If multiple VMs on the same host are affected, I want host-level metrics. If VMs across the cluster are affected, I want to look at storage fabric, network, cluster services, and recent changes.”

---

## 4. Key concepts

### 4.1 Type-1 hypervisor

AHV is a bare-metal enterprise hypervisor: it runs directly on the physical node and hosts VMs. Nutanix describes AHV as a virtualization platform for enterprise VMs and as part of the Nutanix Cloud Infrastructure platform. ([Nutanix][1])

Interview wording:

> “AHV is the Nutanix native hypervisor. It provides the virtualization layer for VMs and is managed through Prism, instead of requiring a separate hypervisor management stack.”

---

### 4.2 KVM

KVM provides the Linux kernel virtualization function.

You should know:

* It relies on CPU virtualization extensions.
* It allows Linux to host VMs.
* It is not the same thing as a full management product.
* In AHV, KVM is integrated into Nutanix’s operational model.

Model phrase:

> “KVM is the engine; AHV is the enterprise platform built around it.”

---

### 4.3 QEMU

QEMU provides the VM hardware emulation/virtualization layer. In Nutanix AHV architecture, **qemu-kvm** runs in user space for every VM/domain and is used for hardware-assisted virtualization. 

Practical meaning:

* Each VM has an associated QEMU process.
* QEMU represents virtual hardware to the guest OS.
* Problems may involve virtual disk, virtual NIC, CPU model, drivers, or guest compatibility.

---

### 4.4 libvirt

libvirt is the API/daemon used to manage KVM/QEMU. Nutanix documentation states that communication between AOS and KVM/QEMU occurs through **libvirtd**. 

Practical meaning:

* VM lifecycle operations eventually interact with libvirt.
* Start/stop/migrate/configuration operations depend on this control plane.
* If VM operations fail, the issue may involve management services, libvirt, host state, resource availability, or cluster services.

---

### 4.5 CVM — Controller VM

The **Controller VM** is one of the most important Nutanix-specific concepts.

In AHV deployments, the CVM runs as a VM, but it is not just another customer workload. It provides Nutanix storage and data services. The Nutanix Bible AHV architecture document describes disks being presented to the CVM through PCI passthrough so that the storage controller and attached devices are passed directly to the CVM, bypassing the hypervisor for that hardware path. 

Interview-level explanation:

> “The CVM is central to Nutanix architecture. AHV runs user VMs, but the CVM provides the distributed storage services. That means in escalations I need to distinguish between a VM-level issue, a hypervisor/host issue, and a CVM/storage-fabric issue.”

---

### 4.6 HVM — Hardware Virtual Machine

Nutanix documentation states that full hardware virtualization is used for guest VMs in AHV. 

Practical meaning:

* The guest OS believes it is running on real hardware.
* Virtualization is assisted by CPU hardware extensions.
* Performance depends on correct virtual CPU, memory, storage, network, and driver configuration.

---

### 4.7 Prism

**Prism** is the Nutanix management interface. Nutanix describes AHV as being managed through Nutanix Prism, the same web console used to manage the cluster’s compute, storage, and networking. ([Nutanix][1])

For support:

* Prism is usually the first operational view.
* It provides alerts, VM status, host status, events, and capacity/performance indicators.
* Prism Central and Prism Element may both matter depending on scope.

---

### 4.8 VM HA

Nutanix describes AHV as supporting **High Availability**, where VMs are restarted on healthy hosts after a physical host failure. ([Nutanix][1])

Support angle:

* HA is not “zero downtime.”
* HA usually means automated restart after failure.
* Live migration is different from HA.

Good interview distinction:

> “Live migration is planned or controlled movement of a running VM without application disruption. HA is recovery after host failure, where VMs are restarted elsewhere. The customer impact and expected RTO are different.”

---

### 4.9 Live migration

Nutanix describes AHV live migration as the ability to move VMs between hosts in a cluster without disruption to the application or users, especially for maintenance scenarios. ([Nutanix][1])

Support relevance:

* Used during maintenance.
* Depends on host compatibility, network health, storage availability, and resource capacity.
* Can fail because of CPU compatibility, insufficient resources, VM configuration, network problems, or active host issues.

---

### 4.10 CPU generation compatibility

The Nutanix Bible explains that AHV determines the lowest processor generation in a cluster and constrains QEMU domains to that level, similarly to VMware EVC, so VMs can live migrate across hosts with different processor generations. 

This is a strong interview point because it shows you understand enterprise operational constraints:

> “In mixed CPU clusters, live migration requires CPU feature compatibility. AHV abstracts or constrains exposed CPU capabilities so VMs can move safely between hosts.”

---

### 4.11 Memory and CPU overcommit

Virtualization allows more vCPUs to be assigned than physically available because workloads rarely use all allocated CPU constantly. The Nutanix Bible also describes memory overcommit as a way to increase VM density, noting that AHV memory overcommit is optional, disabled by default, and configurable per VM in the referenced AOS context. 

Support manager angle:

* Overcommit can improve utilization.
* Overcommit can also create contention.
* Contention appears as latency, CPU ready/wait symptoms, swapping, slow applications, or noisy-neighbor behavior.

---

### 4.12 Affinity and anti-affinity

Nutanix AHV supports VM-host affinity and anti-affinity controls. VM-host affinity can tie a VM to a host or host group, while anti-affinity keeps selected VMs from running on the same host for availability reasons. 

Support angle:

* Affinity can cause capacity problems if too restrictive.
* Anti-affinity protects clustered applications from single-host failure.
* Licensing, appliances, databases, and HA pairs commonly use these policies.

---

## 5. How it appears in a real escalation

### Scenario 1 — “Several VMs are slow after maintenance”

Possible causes:

* Host resource contention after VM evacuation/rebalancing
* Failed or incomplete live migration
* VM placement imbalance
* Storage latency
* Network path issue
* CVM service issue
* Guest OS resource exhaustion
* Recently changed VM configuration

Manager-level escalation approach:

> “I would first establish blast radius: one VM, one host, one cluster, or one application tier. Then I would correlate the issue with maintenance timing, VM migrations, host alerts, Prism events, resource utilization, and storage/network metrics. I would make sure we communicate customer impact clearly while the SREs isolate whether this is AHV, storage fabric, network, or guest/application-level.”

---

### Scenario 2 — “VM failed to restart after host failure”

Relevant concepts:

* AHV HA
* Host failure detection
* Available cluster capacity
* VM configuration
* Affinity rules
* Storage availability
* Management/control plane state

Questions to drive the bridge:

* Did the host actually fail, or did it become isolated?
* Were HA settings enabled and correctly configured?
* Was there enough capacity on remaining hosts?
* Were affinity rules blocking placement?
* Were affected VMs using special devices, passthrough, pinned resources, or unusual configuration?
* Did Prism log VM restart attempts or placement errors?

---

### Scenario 3 — “Live migration failed”

Likely areas:

* CPU compatibility
* Destination host capacity
* Network connectivity between hosts
* VM configuration
* Attached devices
* Host/CVM health
* Prism/libvirt operation failure

Good manager statement:

> “I would treat live migration failure as both a technical issue and an operational-risk issue. If maintenance depends on migration, I would ask whether we have a safe rollback plan, whether the workload can tolerate a controlled reboot, and whether other VMs are affected.”

---

### Scenario 4 — “Application owner says Nutanix caused the outage”

Your job is to avoid both defensiveness and premature blame.

A good response:

> “We need to validate the claim using evidence. I would ask for application timestamps, affected VM names, user impact, error messages, and recent changes. Then I would correlate those with Prism events, VM metrics, host health, network/storage indicators, and any migration or HA activity. The goal is not to prove Nutanix is innocent; the goal is to establish the failure domain quickly and communicate facts.”

---

### Scenario 5 — “Customer migrating from VMware to AHV”

Nutanix states that AHV supports migration from multiple sources, including enterprise hypervisors and public cloud providers, using Nutanix Move. ([Nutanix][1])

Escalation risks:

* Guest drivers
* IP/MAC/network mapping
* VMware Tools vs Nutanix Guest Tools
* Backup compatibility
* Application licensing
* Downtime window
* Performance baseline differences
* Operational team unfamiliarity with AHV

Support manager positioning:

> “For VMware-to-AHV migrations, I would focus on risk management: pre-checks, compatibility, rollback, communication plan, migration waves, post-migration validation, and clear ownership between Nutanix, the customer, and any third-party vendors.”

---

## 6. Triage questions I should ask

### Impact and scope

1. How many VMs are affected?
2. Are affected VMs on the same AHV host?
3. Are they on the same cluster?
4. Are they using the same subnet, VLAN, storage container, or application tier?
5. Is the issue performance degradation, unavailability, failed operation, or data access?

### Timeline

6. When did the issue start?
7. Was there maintenance, upgrade, migration, failover, or configuration change?
8. Did the issue begin after a host reboot, CVM restart, network change, or storage event?
9. Is the issue ongoing or intermittent?

### VM-level

10. Which VMs are affected?
11. Are VM CPU, memory, disk, or NIC metrics abnormal?
12. Was the VM recently resized?
13. Does the guest OS show CPU, memory, disk, or network saturation?
14. Are Nutanix Guest Tools or appropriate drivers installed where relevant?

### Host-level

15. Are all AHV hosts healthy?
16. Is one host showing higher CPU, memory, network, or I/O load?
17. Are there alerts in Prism?
18. Did any host enter maintenance mode?
19. Did any host recently fail, reboot, or become unreachable?

### Storage/CVM

20. Are CVMs healthy?
21. Is there storage latency?
22. Is the Nutanix cluster healthy?
23. Are there disk, metadata, or replication alerts?
24. Is the issue affecting reads, writes, or both?

### Network

25. Are affected VMs on the same VLAN/subnet?
26. Are there packet drops, MTU mismatches, firewall changes, or routing issues?
27. Did physical networking change recently?
28. Are virtual NICs connected and mapped correctly?

### HA / migration

29. Did HA trigger?
30. Did live migration occur?
31. Did any migration fail?
32. Are there affinity or anti-affinity rules?
33. Is there enough cluster capacity to restart or migrate VMs?

### Customer communication

34. What is the business impact?
35. What is the SLA or severity?
36. Who owns the application?
37. Who can validate recovery?
38. What is the next customer-facing update deadline?

---

## 7. Likely interview questions

1. What is KVM, and how does it relate to Nutanix AHV?
2. Is AHV just KVM?
3. What is the role of QEMU in virtualization?
4. What is libvirt used for?
5. How would you explain AHV to a VMware customer?
6. What is the difference between HA and live migration?
7. How would you triage a VM performance issue on AHV?
8. How would you handle an escalation where multiple VMs are slow?
9. What is the CVM, and why is it important?
10. What would you check if live migration fails?
11. What could prevent a VM from restarting after host failure?
12. How do affinity and anti-affinity rules affect availability?
13. What is CPU compatibility, and why does it matter for migration?
14. How would you communicate with a customer during a Sev1 virtualization incident?
15. When would you involve engineering?
16. How would you distinguish a Nutanix platform issue from a guest OS or application issue?
17. What risks exist when migrating from VMware to AHV?
18. What metrics would you monitor for AHV VM performance?
19. How would you manage a disagreement between the customer, Nutanix SREs, and a third-party vendor?
20. As a support manager, how technical do you need to be?

---

## 8. Model answers in English

### Q1. What is KVM, and how does it relate to AHV?

> “KVM stands for Kernel-based Virtual Machine. It is the Linux kernel virtualization technology that allows a physical host to run virtual machines using hardware-assisted virtualization. In Nutanix AHV, KVM is part of the foundation, together with QEMU and libvirt. However, AHV is more than vanilla KVM. Nutanix adds the enterprise management layer, Prism integration, AOS integration, distributed storage, HA, live migration, networking, lifecycle management, and supportability.”

---

### Q2. Is AHV just KVM?

> “No. KVM is the underlying virtualization technology, but AHV is the Nutanix enterprise hypervisor platform. Saying AHV is just KVM would miss the operational value Nutanix provides: Prism management, integration with the Nutanix Distributed Storage Fabric, high availability, live migration, disaster recovery, automation, and enterprise support workflows.”

---

### Q3. What is QEMU?

> “QEMU is the component that provides the virtual hardware model for the VM. In a KVM-based architecture, KVM provides kernel-level acceleration, while QEMU runs the VM process in user space and presents virtualized devices such as disks, NICs, and controllers to the guest OS. In AHV, QEMU/KVM is managed through Nutanix’s AHV and AOS layers rather than manually by the customer.”

---

### Q4. What is libvirt?

> “libvirt is a management API and daemon used to manage KVM/QEMU virtual machines. It provides a control interface for operations such as creating, starting, stopping, configuring, and migrating VMs. In AHV, Nutanix uses this layer internally as part of how AOS communicates with KVM/QEMU.”

---

### Q5. What is the difference between HA and live migration?

> “Live migration is a planned or controlled movement of a running VM from one host to another, typically without application disruption. HA is a recovery mechanism after a host failure; the VM is restarted on another healthy host. Live migration is about continuity during planned operations. HA is about recovery after failure.”

---

### Q6. How would you triage a VM performance issue on AHV?

> “I would start by defining scope and impact: one VM, several VMs on one host, or multiple VMs across the cluster. Then I would correlate the timeline with recent changes, migrations, alerts, maintenance, or upgrades. Technically, I would look at VM CPU, memory, disk latency, network throughput, and guest OS metrics. If several VMs on the same host are affected, I would look at host-level resource contention. If the issue is cluster-wide, I would involve storage fabric, CVM health, network, and Prism events. As a manager, I would also establish severity, ownership, customer update cadence, and escalation path.”

---

### Q7. What would you check if live migration fails?

> “I would check destination host capacity, host health, network connectivity, VM configuration, CPU compatibility, affinity rules, attached devices, and any Prism or AHV events explaining the failure. I would also assess operational risk: if maintenance depends on migration, we need a safe rollback or alternative plan before increasing customer impact.”

---

### Q8. What is the CVM, and why does it matter?

> “The Controller VM is central to Nutanix architecture. It provides the Nutanix storage and data services on each node. In AHV, customer VMs run on the hypervisor, while the CVM provides the distributed storage services. During escalations, I need to distinguish between a guest VM issue, a hypervisor issue, and a CVM or storage-fabric issue because the symptoms may look similar to the customer.”

---

### Q9. How would you explain AHV to a VMware customer?

> “I would explain AHV as Nutanix’s native enterprise hypervisor, integrated into the Nutanix Cloud Infrastructure platform. The customer gets VM lifecycle management, HA, live migration, storage integration, DR capabilities, and Prism-based operations without a separate hypervisor management stack. I would avoid positioning it only as a cost replacement for VMware; the stronger message is operational simplicity, integration, and supportability.”

---

### Q10. How do you handle a Sev1 escalation involving AHV?

> “I would first stabilize the incident process: confirm business impact, affected workloads, severity, bridge ownership, customer update cadence, and internal escalation path. In parallel, I would drive technical triage by separating symptoms into VM, host, cluster, storage, network, and guest OS domains. I would ensure logs, timelines, and recent changes are captured early. My role would be to keep the investigation structured, remove blockers for the SREs, communicate clearly with the customer, and escalate to engineering when evidence suggests a product defect or unknown failure mode.”

---

### Q11. What could prevent a VM from restarting after host failure?

> “Potential causes include insufficient capacity on remaining hosts, affinity rules, storage or CVM health issues, VM configuration constraints, management-plane problems, or a broader cluster issue. I would check whether HA was enabled and triggered, whether Prism recorded restart or placement errors, whether other VMs restarted successfully, and whether the failure is isolated or systemic.”

---

### Q12. What is your technical depth as a support manager?

> “I am not positioning myself as a Senior SRE individual contributor. My value is technical escalation leadership. I need enough technical fluency to understand the architecture, ask precise triage questions, challenge assumptions, route issues correctly, and communicate with both engineers and customers. For deeper AHV internals, I would rely on senior SREs and engineering, but I would still own the escalation process, customer confidence, and operational outcome.”

---

## 9. Connection with my experience

Your current experience maps very well to this topic if you position it correctly.

### Your SaaS/cloud/operations experience translates into AHV support like this:

| Your experience                    | AHV / Nutanix equivalent                                                           |
| ---------------------------------- | ---------------------------------------------------------------------------------- |
| 24/7 enterprise support leadership | Global support operations for mission-critical infrastructure                      |
| Incident management                | Sev1/Sev2 customer escalation ownership                                            |
| SLA / MTTR                         | Availability, recovery, and customer-impact management                             |
| Monitoring with Grafana/Kibana     | Prism alerts, VM metrics, host metrics, logs, event correlation                    |
| Cloud operations                   | Hybrid cloud / infrastructure operations mindset                                   |
| Kubernetes / Linux                 | Comfort with distributed systems, nodes, workloads, resource contention            |
| Jira / Confluence                  | Case tracking, RCA documentation, knowledge management                             |
| Salesforce                         | Customer case lifecycle and escalation visibility                                  |
| Coaching support teams             | Developing SRE/support engineers and improving triage quality                      |
| Escalation management              | Cross-functional coordination with engineering, account teams, TAMs, and customers |

The strongest positioning:

> “My background is in leading enterprise support operations for complex, always-on platforms. AHV is a different technology domain, but the operational patterns are familiar: resource contention, incident severity, customer communication, escalation hygiene, monitoring, change correlation, SLA pressure, and cross-functional resolution. I am building Nutanix-specific depth around AHV, AOS, Prism, HCI, storage, and networking so I can lead escalations credibly without pretending to be the deepest individual contributor in the room.”

---

## 10. Minimum I need to memorize

Memorize these points cold:

1. **KVM = Linux kernel virtualization technology.**
2. **AHV uses Linux, QEMU, and KVM as foundational components.**
3. **AHV is not just KVM; it is Nutanix’s enterprise hypervisor platform.**
4. **QEMU runs the VM process and provides virtual hardware.**
5. **libvirt is the management/API layer used to control KVM/QEMU domains.**
6. **CVM is central to Nutanix because it provides storage/data services.**
7. **Prism is the management interface for AHV and Nutanix clusters.**
8. **HA restarts VMs after host failure.**
9. **Live migration moves running VMs between hosts with minimal/no disruption.**
10. **Affinity controls placement; anti-affinity improves availability.**
11. **CPU compatibility matters for live migration across different host generations.**
12. **Troubleshooting starts with scope: one VM, one host, one cluster, or external dependency.**
13. **As a manager, your role is technical escalation leadership, not kernel-level debugging.**

A concise answer you can use repeatedly:

> “For AHV/KVM basics, I focus on the operational model: KVM provides the Linux virtualization engine, QEMU provides the VM hardware layer, libvirt provides management control, and Nutanix AHV integrates those into an enterprise HCI platform managed by Prism and backed by AOS distributed storage. In support escalations, the key is to isolate whether the issue is VM-level, host-level, cluster-level, storage, network, or guest/application-related.”

---

## 11. Advanced / optional level

You do **not** need to master these deeply for a manager interview, but knowing they exist helps you sound credible.

### Advanced areas

* Detailed QEMU process troubleshooting
* libvirt XML/domain internals
* AHV CLI commands such as `acli`
* Host-level log analysis
* CVM service-level troubleshooting
* Open vSwitch internals
* VirtIO driver behavior
* vNUMA tuning
* CPU pinning
* Huge pages
* SR-IOV / PCI passthrough
* Advanced VM migration failure modes
* AHV networking flows
* Disaster recovery replication internals
* Nutanix Move migration internals
* Performance tuning for databases, VDI, or latency-sensitive workloads

How to position advanced gaps:

> “I am still building hands-on depth in AHV internals such as advanced networking, storage-path analysis, and low-level hypervisor logs. However, I understand the architecture and the operational failure domains well enough to lead escalations, ask the right questions, and coordinate senior technical resources effectively.”

---

## 12. Final checklist

Before an interview, you should be able to explain:

* [ ] What KVM is.
* [ ] What QEMU does.
* [ ] What libvirt does.
* [ ] Why AHV is more than vanilla KVM.
* [ ] What the CVM does.
* [ ] Why AHV is important inside Nutanix HCI.
* [ ] Difference between AHV, AOS, Prism, and CVM.
* [ ] Difference between HA and live migration.
* [ ] How to triage VM performance issues.
* [ ] How to manage a live migration failure.
* [ ] How to handle a host failure escalation.
* [ ] How to communicate with a customer during a virtualization incident.
* [ ] How to connect your Harmonic experience to Nutanix support leadership.
* [ ] How to admit technical learning areas without weakening your manager positioning.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------------------- |
| `acli`                   | Nutanix command-line tool used for AHV/VM operations.                                           |
| AHV                      | Acropolis Hypervisor; Nutanix native enterprise hypervisor.                                     |
| AMD-V                    | AMD CPU virtualization extension used by hypervisors.                                           |
| Anti-affinity            | Policy to keep selected VMs away from the same host.                                            |
| AOS                      | Acropolis Operating System; Nutanix core software platform.                                     |
| API                      | Application Programming Interface; programmatic control interface.                              |
| Bare-metal hypervisor    | Hypervisor running directly on physical hardware.                                               |
| Blast radius             | Scope of impact across VMs, hosts, clusters, or services.                                       |
| Cluster                  | Group of Nutanix nodes operating as one system.                                                 |
| Controller VM            | See CVM.                                                                                        |
| CPU compatibility        | Ability for VMs to run or migrate across hosts with different CPUs.                             |
| CPU overcommit           | Assigning more virtual CPU capacity than physical CPU capacity.                                 |
| CVM                      | Controller VM; Nutanix VM providing storage/data services on a node.                            |
| Datastore                | Generic virtualization term for storage used by VMs.                                            |
| DR                       | Disaster Recovery; recovery capability after site or major failure.                             |
| DSF                      | Distributed Storage Fabric; Nutanix distributed storage layer.                                  |
| EVC                      | Enhanced vMotion Compatibility; VMware CPU compatibility feature.                               |
| Guest OS                 | Operating system running inside a virtual machine.                                              |
| HA                       | High Availability; automatic recovery/restart after failure.                                    |
| HCI                      | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform. |
| Host                     | Physical server running the hypervisor and VMs.                                                 |
| HVM                      | Hardware Virtual Machine; VM using hardware-assisted virtualization.                            |
| Intel VT-x               | Intel CPU virtualization extension used by hypervisors.                                         |
| Kernel                   | Core part of an operating system.                                                               |
| KVM                      | Kernel-based Virtual Machine; Linux kernel virtualization technology.                           |
| libvirt                  | API/daemon used to manage KVM/QEMU virtual machines.                                            |
| Live migration           | Moving a running VM between hosts with minimal/no disruption.                                   |
| Memory overcommit        | Allocating more VM memory than physically available, based on expected usage.                   |
| MTTR                     | Mean Time To Restore/Resolve; incident recovery metric.                                         |
| Nutanix Move             | Nutanix tool for migrating workloads to AHV.                                                    |
| Nutanix Prism            | Nutanix web management interface for clusters and VMs.                                          |
| PCI passthrough          | Directly assigning physical PCI hardware to a VM.                                               |
| Prism Central            | Centralized Nutanix management plane for multiple clusters.                                     |
| Prism Element            | Cluster-level Nutanix management interface.                                                     |
| QEMU                     | Emulator/virtualizer that provides virtual hardware and runs VM processes.                      |
| qemu-kvm                 | QEMU using KVM acceleration for virtual machines.                                               |
| RCA                      | Root Cause Analysis; post-incident investigation output.                                        |
| RTO                      | Recovery Time Objective; target time to restore service.                                        |
| Sev1                     | Highest-severity incident, usually critical business impact.                                    |
| SLA                      | Service Level Agreement; contracted support/service target.                                     |
| SRE                      | Site Reliability Engineer; role focused on reliability and operations.                          |
| Storage latency          | Delay in storage read/write operations.                                                         |
| Type-1 hypervisor        | Hypervisor running directly on server hardware.                                                 |
| vCPU                     | Virtual CPU assigned to a VM.                                                                   |
| vDisk                    | Virtual disk attached to a VM.                                                                  |
| VirtIO                   | Paravirtualized driver framework commonly used with KVM/QEMU.                                   |
| Virtual machine          | Software-defined computer running on a hypervisor.                                              |
| Virtual NIC              | Virtual network interface assigned to a VM.                                                     |
| VM                       | Virtual Machine.                                                                                |
| VM affinity              | Policy controlling which host or hosts a VM can run on.                                         |
| VM HA                    | VM High Availability; restart of VMs after host failure.                                        |
| vMotion                  | VMware term for live migration.                                                                 |
| vNIC                     | Virtual Network Interface Card.                                                                 |
| vNUMA                    | Virtual NUMA topology exposed to a VM for performance optimization.                             |
| Workload                 | Application, service, or VM running on infrastructure.                                          |

[1]: https://www.nutanix.com/products/ahv "AHV: Virtualization Solution for Enterprise | Nutanix"
