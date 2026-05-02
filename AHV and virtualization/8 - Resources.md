# AHV / Virtualization — CPU / memory / disk / NIC

## 1. Short definition

**AHV — Acropolis Hypervisor — is Nutanix’s native virtualization layer for running virtual machines on Nutanix HCI.** In practical terms, AHV provides the VM runtime — vCPU, memory, virtual disks, virtual NICs, live migration, HA, virtual networking, and integration with Prism — while AOS/CVM provide the distributed storage and control-plane services underneath.

For interviews, think of this topic as:

> “Can I understand and troubleshoot a VM as a set of virtualized resources — CPU, memory, disk, and network — running on a distributed Nutanix HCI platform?”

Nutanix describes AHV as the default hypervisor option for Nutanix HCI, integrating virtualization, networking, infrastructure, and operations management through Prism. ([Nutanix][1])

---

## 2. Clear explanation

A VM on AHV is not just “a server.” It is a combination of four main resource planes:

1. **CPU**
2. **Memory**
3. **Disk / storage I/O**
4. **NIC / network I/O**

When a customer reports that “the application is slow,” “the VM is unreachable,” or “the database froze,” the support team must isolate which resource plane is failing or saturated.

### CPU

In AHV, a VM receives one or more **vCPUs**. These are scheduled onto physical CPU cores on the AHV hosts. The key interview idea is not only “how many vCPUs does the VM have,” but whether the VM is **right-sized**.

A common enterprise issue is **over-allocation**. Giving a VM too many vCPUs can actually make performance worse because the scheduler needs to place more virtual CPU work on physical CPU resources. Nutanix best-practice documentation recommends using only as many vCPUs as the VM actually requires, and generally increasing vCPUs rather than cores per vCPU when scaling CPU resources. ([Nutanix][2])

Important AHV-specific point: CPU and memory can be hot-added to AHV VMs, but with limitations. Nutanix documentation says you can increase memory and number of CPUs while the VM is powered on, but you cannot decrease CPU or memory while powered on, and you cannot change cores per socket while powered on. ([Nutanix][3])

### Memory

Memory is allocated to the VM from the host’s physical RAM. In Nutanix, memory planning matters not only for guest VMs but also for the **Controller VM — CVM**. The CVM is central to Nutanix storage and cluster services, so starving the host or CVM is dangerous.

For AHV, memory sizing also interacts with **hugepages**, **NUMA**, VM placement, HA reservations, and workload performance. Nutanix documentation notes that AHV uses hugepages to provide better performance, and some workloads benefit from hugepages at the VM level. ([Nutanix][4])

For a support manager, the practical point is:

> Memory issues are not only “is RAM full?” They include VM sizing, host pressure, ballooning/swapping symptoms, CVM health, NUMA alignment, HA reservations, and whether recent changes caused contention.

### Disk / storage

In Nutanix, VM disks are **vDisks** backed by the Nutanix distributed storage layer. This is where AHV differs from a classic three-tier architecture.

In a traditional VMware + SAN model, the hypervisor consumes external shared storage. In Nutanix HCI, storage is distributed across nodes and served through the Nutanix architecture. AHV VM disks are associated with Nutanix storage containers, and Nutanix documentation notes that AHV vDisks can be migrated between containers, with some operational restrictions during migration. ([Nutanix][5])

The support-relevant point:

> Disk performance problems may involve the VM guest OS, the virtual disk, the storage container, the CVM, the local node, replication, data resiliency, snapshots, migration, or backend storage pressure.

For Windows VMs, **VirtIO drivers** are especially important. Nutanix documentation notes that VirtIO drivers are delivered separately from AOS releases and should be kept current through NGT or the latest VirtIO package from the Nutanix Support portal. ([Nutanix][6])

### NIC / network

A VM NIC is a **virtual network adapter** connected to an AHV virtual network. That virtual network maps to VLANs and bridges on the host.

Nutanix documentation states that AHV supports VLANs for CVMs, AHV hosts, and guest VMs; guest VM networks can be created and managed in Prism, aCLI, or REST; and each AHV virtual network maps to a single VLAN and bridge. ([Nutanix][7])

Networking issues are usually escalation-heavy because they can involve many domains:

* Guest OS configuration
* VM NIC state
* VLAN assignment
* IP addressing
* AHV virtual switch / bridge
* Physical NICs
* Bonds / uplinks
* Top-of-rack switch config
* Firewall / security rules
* DNS / routing
* External customer network

For a manager, the key skill is knowing how to drive a structured isolation path instead of letting teams blame each other.

---

## 3. Why it matters for Nutanix / Worldwide Support

This topic matters because **most Nutanix support escalations eventually reduce to resource isolation**:

* Is it a VM issue?
* Is it a host issue?
* Is it a cluster issue?
* Is it storage?
* Is it network?
* Is it the guest OS?
* Is it a third-party integration?
* Is it customer configuration?
* Is it a product defect?

As a **Manager, Worldwide Support**, you are not expected to debug every kernel-level issue personally. But you are expected to understand enough to:

1. Ask the right triage questions.
2. Recognize weak troubleshooting.
3. Challenge incomplete hypotheses.
4. Coordinate SREs, support engineers, product, networking, storage, and customer teams.
5. Communicate clearly during escalations.
6. Protect SLA, MTTR, customer confidence, and escalation quality.

For Nutanix specifically, AHV is strategic because it is Nutanix’s native virtualization platform for enterprise workloads across datacenter, edge, and cloud environments. ([Nutanix][8])

A good support manager can say:

> “I do not need to be the deepest AHV engineer in the room, but I need to understand how VM CPU, memory, disk, and NIC behavior map to AHV, AOS, Prism, CVM, and the customer’s enterprise stack.”

That is exactly the positioning you want: **technical escalation manager, not Senior SRE IC**.

---

## 4. Key concepts

### 4.1 AHV

AHV is Nutanix’s native hypervisor. It provides enterprise virtualization capabilities such as VM operations, live migration, VM high availability, and virtual network management through Prism workflows. ([Nutanix][1])

Interview framing:

> AHV is the compute virtualization layer in the Nutanix stack, tightly integrated with AOS, Prism, storage, networking, and operational management.

### 4.2 AOS

**AOS — Acropolis Operating System —** is the Nutanix software layer that powers the distributed storage and platform services. In escalations, AHV and AOS are often discussed together because VM performance depends on both virtualization and the distributed storage fabric.

### 4.3 CVM

The **Controller VM** is a special Nutanix VM running on each node. It provides storage and cluster services. For AHV escalations, CVM health is critical because storage I/O and cluster operations depend on it.

A mistake to avoid in interviews:

> Do not treat CVM like a normal workload VM. It is infrastructure-critical.

### 4.4 Prism Element and Prism Central

**Prism Element** manages a single cluster.
**Prism Central** provides centralized management across clusters.

For support interviews, mention that Prism is usually the first operational interface for checking VM state, alerts, performance charts, tasks, events, and cluster health.

### 4.5 vCPU

A **vCPU** is the virtual CPU presented to the VM. More vCPUs are not always better. Oversizing can increase scheduling overhead and waste capacity. Nutanix guidance recommends using only as many vCPUs as the VM needs. ([Nutanix][2])

### 4.6 Cores per socket

AHV allows configuration of vCPU topology. For interview purposes, remember:

* vCPU count affects scheduling and guest capacity.
* Cores-per-socket may matter for OS/application licensing or NUMA behavior.
* Nutanix guidance favors increasing vCPUs rather than cores per vCPU when adding CPU capacity. ([Nutanix][2])
* Cores per socket cannot be changed while the VM is powered on. ([Nutanix][3])

### 4.7 Memory hot-add

AHV supports increasing memory while the VM is powered on, but not decreasing it while powered on. ([Nutanix][3])

Support framing:

> Hot-add is useful for mitigation during an incident, but permanent right-sizing should still happen through controlled change management.

### 4.8 Hugepages

Hugepages are larger memory pages that can improve performance for some workloads. Nutanix documentation states that AHV uses hugepages to provide best performance, and certain applications may benefit from VM-level hugepages. ([Nutanix][4])

### 4.9 NUMA

**NUMA — Non-Uniform Memory Access —** matters when large VMs span CPU and memory boundaries across sockets. For advanced performance issues, wrong sizing or placement can hurt latency-sensitive workloads.

Minimum interview point:

> For normal cases, I would not overfocus on NUMA initially. For large database, SAP, VDI, or latency-sensitive workloads, I would involve deeper performance analysis.

### 4.10 vDisk

A **vDisk** is a virtual disk attached to a VM. On Nutanix, it is backed by distributed storage, usually through a storage container.

Support framing:

> A disk issue can appear as guest OS latency, application timeout, high IOPS, snapshot impact, storage container pressure, CVM issue, or cluster resiliency problem.

### 4.11 Storage container

A Nutanix **storage container** is a logical storage construct used by VMs and vDisks. Nutanix documentation notes that vDisks can be migrated between containers, but some operations are restricted while migration is in progress. ([Nutanix][5])

### 4.12 VirtIO

**VirtIO** provides paravirtualized device drivers, especially relevant for Windows VMs on AHV. Without proper VirtIO drivers, Windows may not recognize AHV virtual disks or may perform poorly. Nutanix warns that installing older NGT can replace newer VirtIO drivers, so driver currency matters. ([Nutanix][6])

### 4.13 vNIC

A **vNIC** is the VM’s virtual network adapter. It connects the guest OS to AHV virtual networking.

### 4.14 VLAN

AHV virtual networks map to VLANs. Nutanix documentation states that each AHV virtual network maps to a single VLAN and bridge. ([Nutanix][7])

### 4.15 OVS / bridge

AHV uses virtual switching/bridging concepts to connect VMs, hosts, CVMs, and physical uplinks. For a manager interview, you do not need to memorize every command, but you should understand that VM traffic flows from guest OS → vNIC → AHV virtual network/bridge → host networking → physical NIC/bond → switch.

### 4.16 Host, cluster, and HA

AHV runs on hosts that form a Nutanix cluster. VM availability depends on host health, cluster capacity, HA configuration, and storage resiliency. Nutanix documentation describes AHV VM high availability and resource reservation for host failure tolerance. ([Nutanix][9])

---

## 5. How it appears in a real escalation

### Scenario A — “Critical database VM is slow after migration to AHV”

Customer impact:

* Business-critical database latency.
* Application users report timeouts.
* Customer says performance was better on VMware.

Likely investigation path:

1. Confirm business impact, timeline, and change history.
2. Check VM CPU ready/saturation equivalents, vCPU sizing, and host contention.
3. Check memory pressure inside the guest and on the AHV host.
4. Check disk latency, IOPS, throughput, queue depth, snapshots, storage container, CVM health.
5. Check VirtIO driver version if Windows.
6. Check network latency if the DB depends on remote app tiers.
7. Compare before/after metrics.
8. Engage performance/SRE specialists if the issue spans AHV, AOS, and guest OS.

Manager value:

> Keep the bridge structured. Prevent random troubleshooting. Force hypotheses, evidence, owners, and next update time.

---

### Scenario B — “VM is unreachable after network change”

Customer impact:

* Production VM unavailable.
* Application team blames Nutanix.
* Network team says nothing changed.

Likely investigation path:

1. Is the VM powered on?
2. Is the guest OS network stack up?
3. Does the VM have the correct vNIC connected?
4. Is the VM attached to the correct AHV virtual network?
5. Is the VLAN correct?
6. Does the host uplink/bond carry that VLAN?
7. Are other VMs on the same network affected?
8. Is it isolated to one host, one VLAN, one cluster, or one VM?
9. Was there a recent switch, firewall, routing, DNS, or security policy change?

Manager value:

> Drive isolation: one VM vs many VMs, one host vs all hosts, one VLAN vs all VLANs, Nutanix vs customer network.

---

### Scenario C — “Windows VM cannot see the disk during installation”

Likely cause:

* Missing VirtIO storage driver.

Support explanation:

> On AHV, Windows may need VirtIO drivers to recognize paravirtualized storage devices. The remediation is to attach/load the appropriate VirtIO driver package or use an image/template where drivers are already included. Nutanix documentation emphasizes keeping VirtIO drivers current and notes they are not necessarily aligned with AOS releases. ([Nutanix][6])

Manager value:

> This is not a severity-one platform outage. It is a known provisioning/configuration path. The support response should be fast, documented, and educational.

---

### Scenario D — “Cluster has capacity, but VM HA cannot be guaranteed”

Possible causes:

* Not enough reserved memory for host failure.
* Oversized VMs.
* Host imbalance.
* HA policy/resource reservation issue.
* Mixed host sizing.
* Maintenance mode constraints.

Nutanix documentation notes that enabling HA Guarantee mode reports memory reserved and how many AHV host failures the system can tolerate. ([Nutanix][9])

Manager value:

> Explain the trade-off between maximizing utilization and reserving failover capacity. This is a customer expectation and risk-management conversation, not just a technical setting.

---

### Scenario E — “Application latency after CPU hot-add”

Possible cause:

* CPU hot-add helped capacity but did not fully optimize VM topology or storage queues.
* Nutanix documentation notes that the multiqueue virtio-scsi controller uses one request queue per vCPU, and adding vCPU to a running VM does not create new queues, which may limit performance. ([Nutanix][2])

Manager value:

> Hot-add can be a tactical mitigation. The permanent fix may require a controlled VM power cycle or resizing window.

---

## 6. Triage questions I should ask

### Business / incident context

1. What is the customer-visible impact?
2. Which application or service is affected?
3. Is this production, pre-production, or test?
4. When did the issue start?
5. Was there a recent change — migration, upgrade, patch, network change, storage policy change, VM resize, backup job, snapshot, or failover?
6. Is there an SLA breach risk?
7. Is the issue ongoing, intermittent, or resolved but requiring RCA?

### Scope

1. One VM or multiple VMs?
2. One host or multiple hosts?
3. One cluster or multiple clusters?
4. One VLAN/network or all networks?
5. One storage container or all storage?
6. One OS type — Windows/Linux — or mixed?
7. One application tier — DB/app/web — or full service?

### CPU

1. Is the VM CPU saturated inside the guest?
2. How many vCPUs are assigned?
3. Was the VM recently resized?
4. Is the VM oversized compared with actual usage?
5. Is host CPU contention visible?
6. Is the workload latency-sensitive?
7. Is NUMA relevant because the VM is large?

### Memory

1. Is the guest OS under memory pressure?
2. Is the AHV host under memory pressure?
3. Was memory hot-added?
4. Is the VM swapping?
5. Are there large VMs competing on the same host?
6. Is HA reserving enough memory?
7. Are CVM memory requirements respected?

### Disk / storage

1. What is the observed symptom — latency, IOPS cap, throughput, errors, filesystem freeze?
2. Is latency seen inside the guest, in Prism, or both?
3. Are snapshots, backups, clones, or migrations running?
4. Is the vDisk on the expected storage container?
5. Is the problem read-heavy or write-heavy?
6. Are other VMs on the same host/container affected?
7. Is the CVM healthy?
8. Are VirtIO drivers current for Windows workloads?

### NIC / network

1. Is the vNIC connected?
2. Is the VM on the correct AHV network/VLAN?
3. Does the guest OS have the right IP, gateway, DNS, and routes?
4. Is only inbound, outbound, or both directions affected?
5. Are other VMs on the same VLAN affected?
6. Is the issue isolated to one AHV host?
7. Were there physical switch, firewall, routing, or VLAN trunk changes?
8. Are there packet drops, MTU mismatches, or bond/uplink issues?

### Escalation management

1. Who owns each layer: Nutanix, customer network, OS team, application team, storage/SRE?
2. What evidence supports each hypothesis?
3. What is the next reversible mitigation?
4. What data is needed before escalation to engineering?
5. What is the customer communication cadence?
6. What is the rollback plan if a change is proposed?

---

## 7. Likely interview questions

### Basic / manager-level

1. What is AHV?
2. How would you explain AHV to a customer coming from VMware?
3. What are the main VM resources you would check during a performance issue?
4. Why can overprovisioning vCPUs hurt performance?
5. What is the difference between vCPU and physical CPU?
6. What is the role of the CVM?
7. What is Prism used for?
8. How would you troubleshoot a VM that is slow?
9. How would you troubleshoot a VM that lost network connectivity?
10. What is VirtIO and why does it matter?

### Enterprise support / escalation

11. A customer says “Nutanix is slow.” How do you structure the escalation?
12. How do you avoid finger-pointing between network, storage, OS, and Nutanix teams?
13. How would you communicate during a P1 incident where the root cause is unknown?
14. How do you decide when to engage engineering?
15. How do you balance mitigation and root cause analysis?
16. What metrics would you track for support performance?
17. How do SLA, MTTR, and customer sentiment influence escalation handling?
18. How do you coach an engineer who jumps to conclusions without evidence?

### More technical / SRE-style

19. What could cause high disk latency on an AHV VM?
20. What could cause a VM to be unreachable after a VLAN change?
21. What are the risks of CPU/memory hot-add?
22. Why do Windows VMs need VirtIO drivers on AHV?
23. What is the difference between a VM disk issue and a backend storage issue?
24. What is NUMA and when would you care?
25. How does HA capacity planning affect VM placement and failover?
26. What information would you collect before escalating to engineering?
27. What would you check if only VMs on one AHV host have network problems?
28. What would you check if only one VM has poor storage performance?

---

## 8. Model answers in English

### Q1. What is AHV?

**Model answer:**

> AHV, or Acropolis Hypervisor, is Nutanix’s native hypervisor for running virtual machines on Nutanix HCI. It provides the compute virtualization layer — VM lifecycle, CPU, memory, virtual disks, virtual NICs, live migration, HA, and virtual networking — and it is managed through Prism. What makes it important in Nutanix is that it is tightly integrated with AOS and the distributed storage fabric, so troubleshooting AHV workloads requires looking at both the VM layer and the underlying cluster health.

---

### Q2. How would you troubleshoot a slow VM on AHV?

**Model answer:**

> I would start by defining impact and scope: one VM or multiple VMs, one host or multiple hosts, one application or a wider cluster issue. Then I would isolate the resource plane: CPU, memory, disk, or network. For CPU, I would check guest utilization, vCPU sizing, and host contention. For memory, I would check guest pressure, host pressure, and whether the VM was recently resized. For disk, I would look at latency, IOPS, snapshots, backups, storage container, CVM health, and VirtIO drivers for Windows. For network, I would check vNIC state, VLAN, routing, DNS, firewall, and whether other VMs on the same network are affected. As a manager, I would make sure the bridge has clear hypotheses, owners, evidence, and customer updates, rather than letting the escalation become random troubleshooting.

---

### Q3. Why can assigning too many vCPUs be a problem?

**Model answer:**

> More vCPUs are not always better. If a VM has more vCPUs than it needs, it can waste cluster capacity and potentially increase scheduling complexity. In a virtualized environment, vCPUs need to be scheduled onto physical CPU resources, so right-sizing is important. For a support escalation, I would not immediately assume that adding CPU fixes performance. I would compare actual utilization, workload behavior, host contention, and recent changes before recommending resizing.

---

### Q4. What is the role of the CVM?

**Model answer:**

> The Controller VM is a core Nutanix infrastructure component that runs on each node and provides cluster and storage services. It should not be treated like a normal workload VM. In performance or storage-related escalations, CVM health is a key part of the investigation because VM I/O depends on the Nutanix storage architecture. From a support management perspective, checking CVM health early helps avoid misdiagnosing storage symptoms as only guest OS or application issues.

---

### Q5. How would you handle a customer escalation where the VM is unreachable?

**Model answer:**

> I would first separate availability from networking. Is the VM powered on? Is the guest OS running? Is the vNIC connected? Then I would validate the network path: correct AHV virtual network, correct VLAN, guest IP configuration, gateway, DNS, routing, firewall, and physical switch path. I would also check scope: whether other VMs on the same VLAN, host, or cluster are affected. If only one VM is affected, I would focus on guest OS, vNIC, or VM configuration. If multiple VMs on one host are affected, I would look at host networking, uplinks, or switch connectivity. During the escalation, I would keep customer communication structured around impact, findings, next actions, and ETA for the next update.

---

### Q6. What is VirtIO and why does it matter on AHV?

**Model answer:**

> VirtIO provides paravirtualized drivers used by guest operating systems, especially Windows, to interact efficiently with AHV virtual storage and network devices. Without the correct VirtIO drivers, a Windows VM may not detect its disk during installation or may have suboptimal performance. In support, VirtIO driver version and installation status are basic checks for Windows VM provisioning and performance issues.

---

### Q7. How would you explain disk latency in a Nutanix AHV environment?

**Model answer:**

> I would explain that disk latency can come from multiple layers: the application, guest OS, virtual disk, AHV host, CVM, storage container, data services, snapshots, backups, migrations, or physical hardware. In Nutanix, storage is distributed, so the investigation needs to correlate guest-visible latency with Prism metrics, cluster health, CVM health, and recent operational tasks. The goal is to determine whether the issue is isolated to a VM, a host, a storage container, or the cluster.

---

### Q8. How would you manage an escalation where the technical root cause is unclear?

**Model answer:**

> I would create structure quickly. First, confirm business impact and severity. Second, define scope and timeline. Third, establish workstreams — compute, storage, network, guest OS, application, and recent changes. Fourth, assign owners and require evidence for each hypothesis. Fifth, communicate regularly with the customer using clear language: what we know, what we do not know yet, what we are testing, and what the next update will contain. I would push for mitigation in parallel with root cause analysis, especially in a P1 situation.

---

### Q9. What do you need to know as a manager versus what should be handled by Senior SREs?

**Model answer:**

> As a manager, I need to understand the architecture, symptoms, triage path, risk, customer impact, and escalation mechanics. I should be able to ask good questions about CPU, memory, disk, network, CVM health, Prism metrics, recent changes, and third-party dependencies. I do not need to personally perform every low-level command or debug every kernel-level issue. That deeper analysis belongs to Senior SREs or engineering. My responsibility is to ensure the right experts are engaged, the troubleshooting is evidence-based, and the customer receives clear communication.

---

### Q10. How does your previous SaaS/cloud support experience apply here?

**Model answer:**

> My background in 24/7 enterprise support, incident management, SLA, MTTR, monitoring, escalations, and cloud operations maps very well to this role. The technology stack is different, but the operational discipline is similar: define impact, isolate scope, correlate metrics, manage stakeholders, drive mitigation, and ensure a strong RCA. My gap is Nutanix-specific depth in AHV, AOS, Prism, and HCI, but I am actively closing that gap by studying the architecture and mapping it to support scenarios I already understand.

---

## 9. Connection with my experience

Your current experience is highly relevant. The strongest bridge is this:

| Your experience                  | Nutanix AHV equivalent                                      |
| -------------------------------- | ----------------------------------------------------------- |
| SaaS incident management         | Enterprise customer escalation management                   |
| Cloud operations                 | Hybrid cloud / HCI operations                               |
| Monitoring with Grafana/Kibana   | Prism metrics, alerts, events, NCC, logs                    |
| SLA / MTTR ownership             | Support case severity, response, restoration, RCA           |
| 24/7 support leadership          | Worldwide Support operating model                           |
| Escalation coordination          | Customer, SRE, engineering, account team, field team        |
| Kubernetes/cloud troubleshooting | Layered isolation: compute, storage, network, control plane |
| Jira/Confluence/Salesforce       | Case workflow, RCA documentation, knowledge management      |
| Coaching support engineers       | Improving triage quality and escalation discipline          |

The sentence to practice:

> “My strength is not that I already know every AHV command. My strength is that I know how to lead complex technical escalations, structure ambiguous incidents, protect customer trust, and coordinate experts across infrastructure, cloud, networking, and application layers. I am building Nutanix-specific depth on top of that operational foundation.”

That is a credible positioning for this role.

---

## 10. Minimum I need to memorize

Memorize these points cold.

### AHV basics

* AHV = Nutanix native hypervisor.
* It runs VMs on Nutanix HCI.
* Managed through Prism.
* Integrated with AOS and Nutanix distributed storage.
* Supports enterprise VM operations such as live migration, HA, virtual networking, and VM lifecycle management. ([Nutanix][1])

### CPU

* vCPU is virtual CPU assigned to a VM.
* More vCPU is not always better.
* Right-sizing matters.
* CPU hot-add is possible, but decreasing CPU while powered on is not. ([Nutanix][3])
* Nutanix recommends using only as many vCPUs as required. ([Nutanix][2])

### Memory

* Memory pressure can exist inside the guest, on the host, or at cluster/HA level.
* AHV uses hugepages for performance. ([Nutanix][4])
* Memory can be increased while powered on, but not decreased while powered on. ([Nutanix][3])
* CVM memory/health matters.

### Disk

* VM disks are vDisks.
* vDisks live on Nutanix storage containers.
* Disk latency can involve guest OS, vDisk, container, CVM, snapshots, backups, migration, or cluster health.
* Windows VMs need proper VirtIO drivers. ([Nutanix][6])

### NIC

* VM network adapter = vNIC.
* AHV virtual networks map to VLANs and bridges.
* Each AHV virtual network maps to a single VLAN and bridge. ([Nutanix][7])
* Network troubleshooting requires isolating VM, VLAN, host, physical switch, firewall, routing, and DNS.

### Manager-level escalation

* Always start with impact, scope, timeline, and recent changes.
* Separate symptoms from root cause.
* Assign owners per layer.
* Communicate what is known, unknown, next action, and next update.
* Drive mitigation and RCA in parallel.

---

## 11. Advanced / optional level

You do not need to master these deeply for a manager interview, but you should recognize them.

### Advanced CPU / memory

* NUMA topology
* vNUMA
* CPU pinning / affinity
* Large VM sizing
* Scheduler behavior
* CPU steal/ready-style concepts
* Hugepage tuning
* Hot-add side effects
* Licensing implications of socket/core topology

### Advanced storage

* Stargate/CVM I/O path
* AHV Turbo
* Storage replication factor
* Data locality
* Erasure coding
* Compression/deduplication
* Snapshot chain behavior
* Storage container policies
* vDisk migration internals
* Queue depth tuning
* Backend disk/hardware failure behavior

### Advanced network

* Open vSwitch / OVS internals
* Bond modes
* LACP
* MTU / jumbo frames
* VLAN trunking
* Flow Network Security
* Microsegmentation
* Overlay networking
* Packet capture on AHV hosts
* Host/CVM/guest traffic separation

### Advanced operations

* NCC health checks
* Log bundle collection
* aCLI / nCLI / host commands
* Prism Central automation
* REST APIs
* LCM upgrade workflows
* Engineering escalation evidence package

For your role, the target is not to become the person who runs every command. The target is to understand what those areas mean, when they matter, and which expert to involve.

---

## 12. Final checklist

Before an interview, you should be able to explain:

* [ ] What AHV is.
* [ ] How AHV fits with AOS, Prism, CVM, and HCI.
* [ ] What vCPU means.
* [ ] Why too many vCPUs can be bad.
* [ ] What memory pressure means at guest, host, and cluster level.
* [ ] Why CVM health matters.
* [ ] What a vDisk is.
* [ ] Why VirtIO matters for Windows VMs.
* [ ] What a vNIC is.
* [ ] How AHV virtual networks relate to VLANs.
* [ ] How to triage a slow VM.
* [ ] How to triage an unreachable VM.
* [ ] How to manage a P1 escalation with unclear root cause.
* [ ] How to communicate with enterprise customers during uncertainty.
* [ ] How to position yourself as a technical escalation manager, not a Senior SRE IC.

Strong final interview sentence:

> “For AHV VM issues, I would structure troubleshooting around CPU, memory, disk, and network, while always correlating guest-level symptoms with Prism, host, CVM, and cluster health. My role as a support manager would be to keep the escalation evidence-based, customer-focused, and moving toward mitigation and RCA.”

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                         |
| ------------------------ | ------------------------------------------------------------------------------- |
| aCLI                     | Acropolis CLI; command-line tool for AHV and VM operations.                     |
| AHV                      | Acropolis Hypervisor; Nutanix native hypervisor.                                |
| AOS                      | Acropolis Operating System; Nutanix platform/storage software layer.            |
| API                      | Application Programming Interface; programmable access to platform functions.   |
| AWS                      | Amazon Web Services; public cloud platform.                                     |
| Azure                    | Microsoft public cloud platform.                                                |
| Bond                     | Logical grouping of physical NICs for redundancy or throughput.                 |
| Bridge                   | Virtual network bridge connecting VM traffic to host networking.                |
| Capacity planning        | Ensuring enough compute, memory, storage, and HA headroom.                      |
| Cluster                  | Group of Nutanix nodes operating as one platform.                               |
| Confluence               | Documentation and knowledge-management platform.                                |
| Core per socket          | VM CPU topology setting affecting CPU presentation to guest OS.                 |
| CPU                      | Central Processing Unit; compute resource for workloads.                        |
| CPU contention           | Multiple workloads competing for limited physical CPU.                          |
| CPU hot-add              | Adding CPU to a running VM without powering it off.                             |
| CVM                      | Controller VM; Nutanix infrastructure VM providing storage/cluster services.    |
| Data resiliency          | Ability to tolerate failures without data loss or service impact.               |
| DNS                      | Domain Name System; resolves names to IP addresses.                             |
| Enterprise support       | Support model for business-critical customer environments.                      |
| Escalation               | Raising priority or involving higher expertise for an issue.                    |
| Firewall                 | Security device or rule set controlling network traffic.                        |
| GCP                      | Google Cloud Platform; public cloud platform.                                   |
| Guest OS                 | Operating system running inside a VM.                                           |
| HA                       | High Availability; ability to keep/restart workloads after failures.            |
| HCI                      | Hyperconverged Infrastructure; compute, storage, and virtualization integrated. |
| Host                     | Physical server running AHV and Nutanix services.                               |
| Hot-add                  | Adding resources to a powered-on VM.                                            |
| Hugepages                | Larger memory pages used to improve performance for some workloads.             |
| I/O                      | Input/Output; read/write or network operations.                                 |
| IOPS                     | Input/Output Operations Per Second; storage performance metric.                 |
| IP                       | Internet Protocol address; network identifier for a system.                     |
| Jira                     | Issue and workflow tracking platform.                                           |
| KPI                      | Key Performance Indicator; operational performance metric.                      |
| Kubernetes               | Container orchestration platform.                                               |
| LACP                     | Link Aggregation Control Protocol; dynamic network link aggregation.            |
| Latency                  | Time taken for an operation, often storage or network response time.            |
| Linux                    | Common enterprise operating system used in infrastructure workloads.            |
| Live migration           | Moving a running VM between hosts with minimal disruption.                      |
| MTTR                     | Mean Time To Restore/Resolve; average time to recover service.                  |
| MTU                      | Maximum Transmission Unit; largest packet size on a network path.               |
| NCC                      | Nutanix Cluster Check; health-check utility for Nutanix clusters.               |
| NGT                      | Nutanix Guest Tools; guest tools package for Nutanix VMs.                       |
| NIC                      | Network Interface Card; physical or virtual network adapter.                    |
| NUMA                     | Non-Uniform Memory Access; CPU/memory locality architecture.                    |
| OVS                      | Open vSwitch; virtual switching component used in AHV networking.               |
| Prism Central            | Centralized Nutanix management plane for multiple clusters.                     |
| Prism Element            | Nutanix management interface for a single cluster.                              |
| P1                       | Priority 1 incident; usually critical business impact.                          |
| RCA                      | Root Cause Analysis; explanation of why an incident occurred.                   |
| REST                     | API architectural style commonly used for automation.                           |
| Right-sizing             | Matching VM resources to real workload needs.                                   |
| Routing                  | Network path selection between subnets or networks.                             |
| Salesforce               | CRM/case-management platform often used for support workflows.                  |
| SLA                      | Service Level Agreement; contractual support or availability target.            |
| Snapshot                 | Point-in-time VM or disk state used for recovery/protection.                    |
| SRE                      | Site Reliability Engineer; role focused on reliability and operations.          |
| Storage container        | Nutanix logical storage construct used by VM disks.                             |
| Throughput               | Amount of data transferred per unit of time.                                    |
| Triage                   | Structured initial diagnosis and prioritization.                                |
| Uplink                   | Physical network path from host to external network.                            |
| vCPU                     | Virtual CPU assigned to a VM.                                                   |
| vDisk                    | Virtual disk attached to a VM.                                                  |
| VLAN                     | Virtual Local Area Network; logical network segmentation.                       |
| VM                       | Virtual Machine; software-defined server running on a hypervisor.               |
| vNIC                     | Virtual Network Interface Card assigned to a VM.                                |
| vNUMA                    | Virtual NUMA topology presented to a VM.                                        |
| VMware                   | Enterprise virtualization platform commonly compared with AHV.                  |
| VirtIO                   | Paravirtualized drivers for efficient VM disk/network access.                   |
| Windows VM               | Virtual machine running Microsoft Windows.                                      |

[1]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v10_0%3Aapp-ahv-access-initial-config-t.html&utm_source=chatgpt.com "AHV 10.0 - AHV Administration Guide - portal.nutanix.com"
[2]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anutanix-ahv-cpu-configuration.html&utm_source=chatgpt.com "Nutanix AHV CPU Configuration"
[3]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v10_3%3Aahv-vm-memory-and-cpu-configuration-c.html?utm_source=chatgpt.com "Virtual Machine Memory and CPU Hot-Plug Configurations"
[4]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anutanix-ahv-memory-configuration.html&utm_source=chatgpt.com "Nutanix AHV Memory Configuration"
[5]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v10_3%3Aahv-vdisk-migrate-t.html?utm_source=chatgpt.com "AHV 10.3 - Migrating a vDisk to Another Container - portal.nutanix.com"
[6]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_10%3Avm-vm-virtio-ahv-c.html&utm_source=chatgpt.com "AHV 6.10 - Nutanix VirtIO for Windows"
[7]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anutanix-ahv-virtual-local-area-networks.html&utm_source=chatgpt.com "Nutanix AHV Virtual Local Area Networks"
[8]: https://www.nutanix.com/products/ahv?utm_source=chatgpt.com "AHV: Virtualization Solution for Enterprise | Nutanix"
[9]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2156-NC2-on-Azure%3Atn-nutanix-ahv-virtual-machine-high-availability.html&utm_source=chatgpt.com "Nutanix AHV Virtual Machine High Availability"
