# AHV / Virtualization — AHV (Acropolis Hypervisor)

## 1. Short definition

**AHV, Acropolis Hypervisor, is Nutanix’s native virtualization layer for running and managing virtual machines on Nutanix HCI.** It is based on Linux KVM and is tightly integrated with **AOS**, **Prism**, Nutanix storage, networking, high availability, lifecycle management, and troubleshooting workflows. Nutanix positions AHV as the default hypervisor option for Nutanix HCI, with enterprise VM operations, live migration, VM high availability, virtual networking, and dynamic scheduling available through Prism. ([Nutanix][1])

For your interview positioning: **AHV is not “just another hypervisor.” It is part of the Nutanix operating model: compute, storage, virtualization, operations, and supportability managed as one platform.**

---

## 2. Clear explanation

Traditional virtualization usually separates several layers:

* **Hypervisor**: VMware ESXi, Microsoft Hyper-V, KVM, etc.
* **Storage**: SAN, NAS, LUNs, RAID groups, datastores.
* **Network**: physical switches, VLANs, virtual switches, distributed switches.
* **Management plane**: vCenter, System Center, scripts, monitoring tools.
* **Support model**: different vendors for compute, storage, hypervisor, backup, and network.

Nutanix changes this model by converging infrastructure into an **HCI cluster**, where each node contributes compute and storage. AOS pools local disks across the cluster through the **Distributed Storage Fabric**, presenting storage services to the hypervisor without relying on traditional SAN/NAS constructs. Nutanix documentation describes DSF as pooling flash and HDD storage across the cluster and presenting it as a datastore, while keeping I/O local and optimized where possible. ([Nutanix][2])

AHV is the native hypervisor that runs on top of this architecture. Nutanix documentation states that AOS can run on AHV or other hypervisors such as VMware ESXi and Microsoft Hyper-V, but AHV is the native Nutanix hypervisor and is based on Linux KVM. ([Nutanix][2])

A simplified mental model:

```text
VMs / Workloads
     |
AHV Hypervisor
     |
AOS services / Controller VM / Distributed Storage Fabric
     |
Nutanix HCI nodes: CPU, RAM, SSD/HDD/NVMe, networking
     |
Prism Element / Prism Central for operations and management
```

The important part for interviews is this: **AHV is deeply integrated into the Nutanix control plane and data plane.** You do not manage it like a standalone KVM host. You manage it as part of a Nutanix cluster through **Prism Element**, **Prism Central**, **AOS**, **LCM**, **NCC**, and support tooling.

AHV supports familiar enterprise virtualization operations:

* Creating and managing VMs.
* Allocating vCPU, memory, disks, and vNICs.
* Live migration.
* VM high availability.
* Host maintenance mode.
* Virtual network management.
* Snapshots, clones, and data protection.
* Dynamic scheduling / workload placement.
* Integration with Prism monitoring and alerts.

Nutanix documentation explicitly calls out AHV workflows such as VM operations, live migration, VM high availability, virtual network management, high availability, dynamic scheduling, and security features. ([Nutanix][1])

For storage I/O, AHV uses Nutanix’s distributed storage layer. The AHV documentation explains that AOS passes disks to VMs as raw SCSI block devices, each AHV host runs an iSCSI redirector, and QEMU is configured to use the redirector as the iSCSI target portal, redirecting I/O to a healthy Stargate service, preferably local. ([Nutanix][1])

In practical terms, when a VM has a performance problem on AHV, you do not only ask, “Is the hypervisor overloaded?” You ask:

* Is the VM CPU-ready or constrained?
* Is memory overcommitted or ballooning/swapping?
* Is storage latency local, remote, or cluster-wide?
* Is the CVM healthy?
* Is Stargate healthy?
* Is the node under maintenance or degraded?
* Is the VM running on the right host?
* Is there a network/VLAN/vSwitch issue?
* Did a recent LCM/AOS/AHV/firmware change happen?
* Are alerts visible in Prism or NCC?

That is the support mindset Nutanix will care about.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, AHV matters because it sits at the intersection of customer impact, escalation complexity, and Nutanix’s strategic value proposition.

You do not need to interview like a kernel engineer, but you do need to show that you understand the operational consequences of AHV in enterprise support.

### Why AHV is strategically important

AHV is Nutanix’s native alternative to external hypervisors such as VMware ESXi. It reduces dependency on a separate virtualization vendor and allows Nutanix to offer a more integrated platform experience. Nutanix documentation describes AHV as included in AOS, tightly integrated, and provided without additional licensing cost. ([Nutanix][2])

That has support implications:

* Customers may migrate from VMware to AHV.
* Customers may be new to AHV and need guidance.
* Escalations may involve misconceptions from VMware administrators.
* Support teams need to translate between VMware concepts and Nutanix/AHV concepts.
* Incidents can span virtualization, storage, networking, and management plane in one case.

### Why it matters for support leadership

As a support manager, you are expected to manage:

* Incident prioritization.
* Customer communication.
* Escalation paths.
* Cross-functional collaboration with SRE, engineering, product, account teams, and TAMs.
* SLA/MTTR expectations.
* High-severity customer situations.
* Team coaching and technical quality.

AHV incidents can become high-pressure quickly because virtualization is where customer workloads run. If a customer says, “Our production VMs are down after a host failure,” the support manager must understand enough to drive the escalation intelligently:

* Is this a VM HA issue?
* Is it a host failure?
* Is it a storage-path issue?
* Is it a networking issue?
* Is it an AOS/CVM issue?
* Is it a customer configuration issue?
* Is there data risk?
* Is a workaround possible?
* Does engineering need to be engaged?

Your value is not to personally debug every AHV command. Your value is to **lead the technical escalation with enough fluency to ask the right questions, challenge weak hypotheses, communicate accurately, and keep the team moving toward mitigation and RCA.**

---

## 4. Key concepts

### AHV

AHV is Nutanix’s native hypervisor. It is based on Linux KVM and provides enterprise VM management integrated with AOS and Prism. ([Nutanix][2])

Interview phrasing:

> AHV is Nutanix’s native KVM-based hypervisor, integrated with AOS and Prism, designed to run enterprise workloads on Nutanix HCI without requiring a separate virtualization management stack.

---

### AOS

**AOS, Acropolis Operating System**, is the core Nutanix software layer providing distributed storage, data services, security, networking integration, lifecycle management, and platform services. Nutanix documentation describes AOS as the base operating system/data plane packaging storage, compute, security, and network runtime. ([Nutanix][2])

Interview phrasing:

> I think of AOS as the distributed platform layer that makes the Nutanix cluster behave like a unified infrastructure system, while AHV provides the virtualization layer for VMs.

---

### Prism Element and Prism Central

**Prism Element** manages an individual Nutanix cluster. **Prism Central** provides centralized management across clusters and higher-level operations. In Nutanix documentation, Prism is described as the control plane for Nutanix infrastructure management, including simplified operations, troubleshooting, capacity insights, and upgrades. ([Nutanix][2])

Interview phrasing:

> For support, Prism is critical because it gives the operational view: alerts, performance, capacity, VM state, host state, and cluster health.

---

### Controller VM / CVM

A **Controller VM** runs on each Nutanix node and provides core AOS services. The CVM is central to Nutanix storage and cluster operations. For AHV troubleshooting, CVM health is critical because VM I/O depends on Nutanix storage services.

Interview phrasing:

> In a Nutanix environment, I would not troubleshoot a VM performance issue by looking only at the guest OS or hypervisor. I would also check CVM health, cluster services, storage latency, and Prism alerts.

---

### Distributed Storage Fabric / DSF

DSF pools storage from cluster nodes and provides distributed, resilient storage to VMs. Nutanix documentation describes DSF as pooling flash and HDD storage across the cluster, exposing storage systems with no single point of failure, and optimizing I/O locally. ([Nutanix][2])

Interview phrasing:

> DSF is one of the reasons Nutanix support cases are cross-domain: a VM issue can involve compute scheduling, virtual networking, and distributed storage behavior at the same time.

---

### VM High Availability

VM HA ensures that if a host fails, VMs can be restarted on another healthy host, assuming sufficient resources and proper configuration.

Interview phrasing:

> In an escalation, I would verify whether the issue is VM availability, host availability, cluster resource availability, or storage availability, because those are different failure domains.

---

### Live Migration

Live migration allows a running VM to move from one AHV host to another with minimal disruption, typically for maintenance or load balancing. Nutanix documentation lists live migration as one of AHV’s familiar workflows for virtualization teams. ([Nutanix][1])

Interview phrasing:

> Live migration is important operationally because it allows host maintenance and workload balancing without planned downtime, but during incidents I would verify whether migration is helping, failing, or masking an underlying resource issue.

---

### Acropolis Dynamic Scheduling / ADS

ADS is Nutanix’s workload placement and scheduling capability for AHV. Nutanix documentation describes ADS as making VM placement decisions using CPU, memory, and storage data points, continuously monitoring for anomalies and hotspots, and adjusting thresholds over time. 

Interview phrasing:

> ADS is similar in purpose to workload balancing: it helps place and migrate VMs based on resource contention and hotspots, but I would still validate whether the automation is behaving as expected during a customer escalation.

---

### AHV networking

AHV includes virtual networking, vNICs, VLANs, bridges/virtual switches, and integration with physical top-of-rack switching. Nutanix documentation describes AHV as including virtual network management and optional Flow Security and Networking for microsegmentation and advanced software-defined networking. ([Nutanix][1])

Interview phrasing:

> Many AHV escalations that look like VM outages are actually network-path issues: VLAN tagging, trunk configuration, IP conflicts, MTU mismatch, DNS, gateway, firewall, or upstream switch changes.

---

### VirtIO drivers

Windows VMs on AHV use Nutanix VirtIO drivers for optimized disk and network performance. The AHV guide notes that for maximum VM performance, the latest Nutanix VirtIO package should be installed for Windows VMs. ([Nutanix][1])

Interview phrasing:

> If a Windows VM has poor disk or network performance after migration to AHV, I would check VirtIO driver version and compatibility early in the triage.

---

### AHV Turbo

AHV Turbo improves the storage I/O path by bypassing QEMU for storage I/O and using a multi-queue approach, reducing CPU usage and increasing available storage I/O for VMs. Nutanix documentation states that AHV Turbo is transparent to VMs and enabled by default on VMs running in AHV clusters. ([Nutanix][1])

Interview phrasing:

> I would not lead with AHV Turbo in a manager interview, but I would recognize it as part of Nutanix’s optimized AHV I/O path and mention it if discussing performance escalations.

---

### NCC

**NCC, Nutanix Cluster Check**, is used to diagnose cluster health and identify recommended or qualified configurations. Nutanix documentation describes NCC as cluster-resident software that proactively runs hundreds of checks, raises alerts, and can automatically create support cases depending on the issue. ([Nutanix][2])

Interview phrasing:

> In support, NCC is important because it gives a standardized health baseline before deeper escalation. I would want the team to use it early, not after hours of speculation.

---

### LCM

**LCM, Lifecycle Manager**, tracks and updates software and firmware versions across Nutanix components. Nutanix documentation describes LCM as tracking software and firmware versions and allowing administrators to view inventory and update versions through Prism. ([Nutanix][2])

Interview phrasing:

> In escalations after upgrades or maintenance, I would immediately ask what changed: AOS, AHV, firmware, drivers, LCM activity, or third-party integrations.

---

## 5. How it appears in a real escalation

### Escalation scenario 1: VMs unavailable after host failure

Customer says:

> “One AHV host failed and several production VMs did not come back online. We need immediate help.”

Support manager view:

* Confirm business impact and severity.
* Identify affected VMs, cluster, host, and failure time.
* Check Prism alerts.
* Validate host state.
* Check whether VM HA was enabled/configured as expected.
* Check cluster resource availability.
* Confirm whether storage services are healthy.
* Determine whether VMs are powered off, stuck, inaccessible, or running but unreachable.
* Assign technical owner and customer communication owner.
* Establish update cadence.
* Escalate to SRE/engineering if VM HA behavior appears abnormal.

What you say in an interview:

> In this situation, I would first stabilize the incident: confirm scope, customer impact, affected workloads, and whether there is data risk. Then I would have the team validate host health, VM state, HA behavior, cluster capacity, CVM health, and Prism/NCC alerts. As a manager, my role is to keep the escalation structured, ensure the right specialists are engaged, maintain customer confidence with clear updates, and drive toward mitigation before deep RCA.

---

### Escalation scenario 2: Performance degradation after migration from VMware to AHV

Customer says:

> “After moving workloads from ESXi to AHV, some Windows VMs are slower.”

Technical areas:

* VirtIO drivers.
* VM sizing.
* CPU/memory contention.
* Storage latency.
* Network throughput.
* Guest OS configuration.
* Application baseline before/after migration.
* Backup/antivirus agents.
* Time synchronization.
* NUMA/vCPU sizing.
* Prism performance charts.

Manager view:

* Avoid blaming AHV or customer configuration prematurely.
* Ask for objective before/after metrics.
* Separate guest, hypervisor, storage, and network dimensions.
* Engage migration experts if needed.
* Create a clear action plan.

Model framing:

> I would push the team to establish a baseline: what changed, which VMs are affected, whether the degradation is CPU, memory, disk, or network, and whether it correlates with migration time. For Windows VMs on AHV, I would expect the team to check VirtIO drivers and compatibility. I would also ask for Prism performance data and NCC output to distinguish platform issues from guest or application issues.

---

### Escalation scenario 3: VM unreachable after network change

Customer says:

> “The VM is running in Prism, but the application is unreachable.”

Likely causes:

* Wrong VLAN assigned.
* Upstream switch trunk issue.
* Gateway/firewall rule changed.
* IP conflict.
* MTU mismatch.
* vNIC disconnected.
* Incorrect subnet.
* DNS issue.
* Security policy / microsegmentation issue.
* Guest firewall.

Support manager behavior:

* Keep investigation layered.
* Confirm VM power state does not equal service availability.
* Ask for recent network changes.
* Ensure network, virtualization, and customer application owners are aligned.

Model phrasing:

> I would treat this as a connectivity escalation, not simply a VM escalation. The first split is whether the VM is down, isolated, or the application is down. I would ask the team to verify vNIC state, VLAN, IP/gateway, guest firewall, upstream switch changes, and Prism network configuration. If Flow or microsegmentation is involved, I would include policy review in the triage.

---

### Escalation scenario 4: Customer wants to migrate from VMware to AHV

Customer concern:

> “Can AHV replace VMware for our production workloads?”

Support manager answer:

> AHV is Nutanix’s native hypervisor and supports enterprise virtualization operations such as VM management, live migration, high availability, dynamic scheduling, and Prism-based monitoring. I would not position migration as a blind lift-and-shift. I would recommend workload assessment, compatibility validation, migration planning, backup/rollback planning, operational training, and post-migration monitoring.

This is a good moment to show maturity: **do not oversell**. Enterprise customers respect controlled risk management.

---

## 6. Triage questions I should ask

### Impact and severity

1. Which workloads, VMs, applications, tenants, or business services are affected?
2. Is this production, pre-production, or development?
3. Is there revenue, customer-facing, or regulatory impact?
4. Is the issue availability, performance, data access, or manageability?
5. When did the issue start?
6. Is the issue ongoing, intermittent, or resolved but requiring RCA?

### Change correlation

7. Was there any recent AOS, AHV, LCM, firmware, network, storage, or Prism change?
8. Was there a migration from ESXi/Hyper-V to AHV?
9. Was a host placed into maintenance mode?
10. Were any VMs resized, cloned, restored, snapshotted, or migrated?
11. Were there network changes: VLAN, switch, firewall, routing, DNS, MTU?

### Cluster and host health

12. Are there active Prism alerts?
13. What does NCC report?
14. Are all CVMs up and healthy?
15. Are all AHV hosts reachable?
16. Is the cluster degraded?
17. Is there a failed disk, NIC, PSU, or node?
18. Is there enough capacity for HA or migration?

### VM state

19. Is the VM powered on?
20. Is the guest OS responsive from console?
21. Is the issue inside the guest OS or below the guest OS?
22. Are VMware Tools/VirtIO drivers relevant?
23. Are multiple VMs affected or only one?
24. Are affected VMs on the same host, VLAN, storage container, or image/template?

### Performance

25. Is the bottleneck CPU, memory, disk, or network?
26. What are the latency, IOPS, throughput, CPU ready/contention, and memory pressure indicators?
27. Is the issue visible in Prism charts?
28. Are there noisy neighbors?
29. Did ADS move or not move workloads?
30. Is the workload multi-threaded and correctly sized?

### Networking

31. Is the VM reachable from the same subnet?
32. Is the gateway reachable?
33. Is DNS resolving correctly?
34. Is the correct VLAN assigned?
35. Are vNICs connected?
36. Did upstream trunking or firewall policy change?
37. Is Flow/microsegmentation involved?

### Customer management

38. Who is the customer incident commander?
39. What update cadence do they expect?
40. What is the immediate mitigation path?
41. What is the rollback option?
42. What evidence is required for RCA?
43. Do we need engineering, product, account team, TAM, or executive communication?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you explain AHV to a customer moving from VMware?
2. How technical do you expect a support manager to be in an AHV escalation?
3. How would you manage a P1 where customer VMs are unavailable on AHV?
4. How would you handle a customer claiming Nutanix caused a production outage?
5. How do you balance fast mitigation with proper RCA?
6. How would you coach support engineers on AHV escalations?
7. How would you communicate during a complex AHV incident involving storage and networking?
8. How would you build confidence with an enterprise customer new to AHV?
9. How would you measure support quality for AHV-related cases?
10. How would you collaborate with SREs and engineering during recurring AHV issues?

### Technical / SRE interview

1. What is AHV?
2. How does AHV relate to AOS?
3. What is Prism’s role in AHV management?
4. What is a CVM and why does it matter?
5. How would you troubleshoot a VM performance issue on AHV?
6. How would you troubleshoot a VM that is powered on but unreachable?
7. What is the difference between hypervisor issue, guest issue, storage issue, and network issue?
8. What is VM high availability?
9. What is live migration?
10. What is ADS?
11. What would you check after a failed host event?
12. What is VirtIO and why does it matter for Windows VMs?
13. How would you approach VMware-to-AHV migration issues?
14. What information would you collect before escalating to engineering?
15. How would you validate whether an issue is platform-wide or isolated?

### Panel / advanced escalation interview

1. Walk us through a P1 escalation involving AHV, storage latency, and an angry customer.
2. How do you prevent escalations from bouncing between storage, virtualization, and networking teams?
3. How do you decide when to involve engineering?
4. How do you keep customer trust when RCA is not yet known?
5. What would your 30/60/90-day learning plan be for AHV?
6. How do you avoid overpromising in vendor support?
7. How would you manage a case where the customer’s configuration is unsupported or risky?
8. How would you identify systemic support gaps from repeated AHV cases?
9. How would you improve runbooks for AHV incidents?
10. How would you ensure global handoff quality across time zones?

---

## 8. Model answers in English

### Q1. What is AHV?

**Model answer:**

> AHV, or Acropolis Hypervisor, is Nutanix’s native hypervisor. It is based on Linux KVM and is tightly integrated with Nutanix AOS and Prism. From a customer perspective, AHV provides enterprise virtualization capabilities such as VM lifecycle management, live migration, VM high availability, virtual networking, monitoring, and operational workflows through Prism.
>
> The important point is that AHV is not managed as an isolated hypervisor. It is part of the Nutanix HCI architecture, where compute, storage, virtualization, and operations are integrated into the same platform.

---

### Q2. How does AHV relate to AOS?

**Model answer:**

> AOS is the Nutanix software platform that provides the distributed storage fabric, data services, cluster services, and operational capabilities. AHV is the native virtualization layer that runs VMs on that platform.
>
> In practical support terms, this means that an AHV issue may involve several layers: the VM, the AHV host, the CVM, AOS services, storage latency, networking, and Prism alerts. So I would not troubleshoot AHV in isolation; I would look at the full Nutanix stack.

---

### Q3. How would you troubleshoot a VM performance issue on AHV?

**Model answer:**

> I would start by defining the symptom and scope. Is it one VM, several VMs, one host, one cluster, or one application? Then I would separate the investigation into CPU, memory, storage, and network.
>
> I would check Prism metrics, active alerts, recent changes, host and CVM health, NCC output, and whether the VM was recently migrated or resized. For Windows VMs, I would also check VirtIO drivers. If the customer recently migrated from VMware to AHV, I would compare pre- and post-migration baselines and validate whether the issue is guest-level, hypervisor-level, storage-related, or network-related.
>
> As a manager, I would ensure the team works from evidence, keeps the customer updated, and focuses first on mitigation before deep RCA.

---

### Q4. How would you manage a P1 where VMs are unavailable after an AHV host failure?

**Model answer:**

> First, I would establish incident control: confirm business impact, affected services, severity, customer contacts, and update cadence. Technically, I would ask the team to validate host state, VM state, HA behavior, cluster capacity, CVM health, Prism alerts, and NCC results.
>
> I would split the work into mitigation and investigation. One engineer would focus on restoring service or finding a workaround, while another collects evidence for root cause. If HA did not behave as expected or there is potential platform instability, I would escalate to SRE or engineering early.
>
> My communication to the customer would be clear: what we know, what we do not know yet, what actions are in progress, and when the next update will be provided.

---

### Q5. What is the role of Prism in AHV operations?

**Model answer:**

> Prism is the main operational interface for Nutanix environments. For AHV, it provides VM management, monitoring, alerts, performance visibility, lifecycle operations, and troubleshooting workflows.
>
> In a support case, Prism is usually one of the first places I would expect the team to look because it provides the operational view of the cluster, hosts, VMs, capacity, and alerts. For a support manager, Prism also helps align technical troubleshooting with customer communication because it gives objective data.

---

### Q6. How would you explain AHV to a VMware customer?

**Model answer:**

> I would explain AHV as Nutanix’s native virtualization platform, integrated directly with the Nutanix HCI stack. Many concepts are familiar to VMware administrators: VMs, vCPUs, memory, virtual disks, vNICs, live migration, HA, host maintenance, and virtual networking.
>
> The difference is the operating model. With AHV, the hypervisor, distributed storage, monitoring, lifecycle management, and many troubleshooting workflows are integrated through Nutanix AOS and Prism. That can reduce operational complexity, but it also requires teams to understand Nutanix-specific concepts such as CVMs, Prism, NCC, LCM, and the distributed storage fabric.

---

### Q7. What would you check if a VM is powered on but unreachable?

**Model answer:**

> I would avoid assuming that a powered-on VM means the service is healthy. I would check console access first to see whether the guest OS is responsive. Then I would validate the VM’s vNIC state, VLAN assignment, IP configuration, gateway reachability, DNS, guest firewall, and any recent network changes.
>
> I would also check whether the issue affects only one VM or multiple VMs on the same host, VLAN, or network segment. If microsegmentation or security policies are involved, I would include those in the triage. The goal is to determine whether this is a guest OS issue, virtual network issue, upstream network issue, or broader cluster issue.

---

### Q8. What is your role as a manager if the SREs are deeper technically than you?

**Model answer:**

> I do not need to be the deepest AHV engineer in the room. My role is to understand the architecture well enough to ask the right questions, detect weak assumptions, prioritize correctly, and remove friction.
>
> In a complex escalation, I would make sure there is a clear owner, a working hypothesis, a mitigation path, a communication cadence, and a decision point for involving engineering. I would also ensure that the team captures evidence for RCA and turns repeated issues into runbooks or knowledge improvements.

---

### Q9. How would you handle a customer who says, “Nutanix is down”?

**Model answer:**

> I would acknowledge the impact but immediately move toward precise scoping. “Nutanix is down” could mean Prism is unavailable, VMs are down, storage latency is high, a network path is broken, a host failed, or a specific application is unavailable.
>
> I would ask the team to determine whether the issue is management-plane, data-plane, compute, storage, network, or guest/application related. At the same time, I would communicate with the customer in business terms: affected services, current mitigation actions, next update time, and what information we need from them.

---

### Q10. What do you need to learn more deeply about AHV?

**Model answer:**

> My current strength is leading enterprise support operations, incident management, SLA/MTTR governance, escalations, monitoring, and cross-functional coordination. My learning focus for Nutanix would be deeper hands-on fluency with AHV, AOS, Prism, CVM services, storage-path troubleshooting, and Nutanix networking.
>
> I would approach this with a structured plan: study AHV architecture, practice common Prism workflows, learn standard triage commands and NCC outputs, review real escalation patterns, and shadow senior SREs. I want to be technical enough to lead escalations effectively while respecting the depth of the SRE specialists.

---

## 9. Connection with my experience

Your background maps well to this role if you frame it correctly.

### Your current strengths

You already bring:

* 24/7 enterprise support leadership.
* Incident management.
* SLA and MTTR ownership.
* Escalation handling.
* Monitoring and observability.
* Cloud operations.
* Customer communication.
* Team coordination.
* Coaching.
* Cross-functional collaboration.
* Tools such as Grafana, Kibana, Jira, Confluence, Salesforce.
* Exposure to AWS, Azure, GCP, Kubernetes, and Linux.

These are directly relevant because Nutanix support is not just about knowing a product. It is about **running a high-quality global support motion for complex enterprise infrastructure**.

### How to connect SaaS/cloud experience to AHV

Use this bridge:

> In SaaS/cloud operations, I learned to troubleshoot distributed systems by separating symptoms across application, compute, storage, network, and control-plane layers. I would apply the same discipline to Nutanix AHV escalations: define impact, isolate failure domains, validate telemetry, coordinate specialists, communicate clearly, and drive mitigation before RCA.

### Mapping your experience to AHV support

| Your experience     | AHV / Nutanix equivalent                                            |
| ------------------- | ------------------------------------------------------------------- |
| 24/7 support team   | Worldwide support coverage and global handoffs                      |
| Incident management | P1/P2 AHV escalations                                               |
| SLA / MTTR          | Restoration targets, update cadence, escalation discipline          |
| Grafana / Kibana    | Prism metrics, alerts, logs, NCC evidence                           |
| Cloud operations    | Distributed infrastructure mindset                                  |
| Kubernetes          | Cluster-level thinking, scheduling, node health, workload placement |
| Linux               | AHV/KVM foundation, CLI comfort, log investigation                  |
| Jira / Confluence   | Case tracking, KBs, runbooks, postmortems                           |
| Salesforce          | Enterprise support case lifecycle                                   |
| Coaching            | Raising technical consistency across support engineers              |

The strongest positioning:

> I am not interviewing as a Senior SRE individual contributor. I am interviewing as a technical support leader who can understand AHV deeply enough to lead escalations, coach teams, communicate with enterprise customers, and partner effectively with SRE and engineering.

---

## 10. Minimum I need to memorize

Memorize these points cold.

1. **AHV = Acropolis Hypervisor**, Nutanix’s native hypervisor.
2. AHV is **based on Linux KVM**.
3. AHV is tightly integrated with **AOS** and **Prism**.
4. Nutanix HCI combines **compute, storage, virtualization, and operations**.
5. **AOS** provides the distributed platform services, especially storage through **DSF**.
6. **Prism Element** manages a cluster; **Prism Central** manages multiple clusters / centralized operations.
7. **CVMs** are critical Nutanix controller VMs running core services on each node.
8. AHV supports enterprise virtualization operations: VM management, live migration, HA, networking, monitoring, and dynamic scheduling.
9. **ADS** handles intelligent VM placement and resource contention avoidance.
10. **NCC** is used for cluster health checks and support diagnostics.
11. **LCM** manages software/firmware lifecycle and versioning.
12. **VirtIO** matters for Windows VM performance on AHV.
13. VM troubleshooting must separate **guest OS, hypervisor, storage, network, and management plane**.
14. In escalations, your role is **structured mitigation + communication + evidence-driven RCA**, not pretending to be the deepest AHV engineer.
15. For VMware customers, explain AHV through familiar concepts but emphasize the integrated Nutanix operating model.

A strong 30-second answer:

> AHV is Nutanix’s native KVM-based hypervisor, integrated with AOS and Prism. It provides enterprise virtualization capabilities such as VM operations, live migration, high availability, virtual networking, monitoring, and dynamic scheduling. In support, AHV matters because customer workloads run there, and incidents often cross compute, storage, networking, and management layers. As a support manager, I need enough AHV fluency to lead escalations, ask the right triage questions, coordinate SREs and engineering, and communicate clearly with enterprise customers.

---

## 11. Advanced / optional level

You do **not** need to master these before the interview, but knowing the vocabulary helps.

### Advanced topics to recognize

* AHV host internals.
* KVM/QEMU architecture.
* AHV Turbo and multi-queue I/O.
* Stargate storage service.
* iSCSI redirector.
* CVM service architecture.
* Nutanix networking internals.
* Open vSwitch / bridge-level concepts.
* Flow Network Security / microsegmentation.
* Advanced processor compatibility.
* NUMA and vCPU sizing.
* GPU/vGPU on AHV.
* Disaster recovery and protection domains.
* Cross-hypervisor migration.
* Advanced NCC checks.
* Log locations and CLI-level diagnostics.
* AOS/AHV compatibility matrix.
* Foundation imaging and cluster creation.
* Calm, Karbon/NKE, Files, Volumes, Objects integrations.

### How to handle advanced questions

Use this pattern:

> I understand the concept and how it affects support triage. I would not claim to be the deepest engineer on that specific subsystem yet, but I would know how to structure the investigation, what evidence to collect, and when to involve the right SRE or engineering owner.

That is credible for a manager role.

---

## 12. Final checklist

Before the interviews, you should be able to explain:

* [ ] What AHV is.
* [ ] How AHV differs from VMware ESXi at an operating-model level.
* [ ] How AHV relates to AOS.
* [ ] What Prism does.
* [ ] What a CVM is and why it matters.
* [ ] What DSF does.
* [ ] What VM HA means.
* [ ] What live migration means.
* [ ] What ADS does.
* [ ] Why VirtIO matters.
* [ ] Why NCC matters in support.
* [ ] Why LCM matters after upgrades.
* [ ] How to triage a VM-down case.
* [ ] How to triage a VM-performance case.
* [ ] How to triage a VM-networking case.
* [ ] How to communicate during a P1.
* [ ] How to position yourself as a technical escalation manager, not a Senior SRE IC.
* [ ] How your Harmonic experience maps to Nutanix support.
* [ ] How to say “I need to go deeper” without sounding weak.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                        |
| ------------------------ | ---------------------------------------------------------------------------------------------- |
| ADC                      | Application Delivery Controller; load balancer or traffic management device.                   |
| ADS                      | Acropolis Dynamic Scheduling; Nutanix VM placement and resource balancing capability.          |
| AHV                      | Acropolis Hypervisor; Nutanix native KVM-based hypervisor.                                     |
| AHV Turbo                | Optimized AHV I/O path using multi-queue behavior to improve VM performance.                   |
| AOS                      | Acropolis Operating System; Nutanix core software platform for storage, compute, and services. |
| API                      | Application Programming Interface; programmatic way to interact with systems.                  |
| Bridge                   | Virtual network construct connecting VM interfaces to host/network paths.                      |
| Cluster                  | Group of Nutanix nodes operating as one distributed system.                                    |
| Compute                  | CPU and memory resources used by workloads.                                                    |
| Control plane            | Management and orchestration layer used to operate infrastructure.                             |
| Controller VM            | Nutanix VM running core AOS services on each node.                                             |
| CPU contention           | Condition where VMs compete for insufficient physical CPU resources.                           |
| CVM                      | Controller VM; core Nutanix service VM on each node.                                           |
| Data plane               | Layer that handles actual workload I/O and service traffic.                                    |
| Datastore                | Storage presented to a hypervisor for VM disks.                                                |
| DNS                      | Domain Name System; resolves names to IP addresses.                                            |
| DR                       | Disaster Recovery; recovery of workloads after site or platform failure.                       |
| DSF                      | Distributed Storage Fabric; Nutanix distributed storage layer.                                 |
| ESXi                     | VMware’s enterprise bare-metal hypervisor.                                                     |
| Flow                     | Nutanix networking/security capability, including microsegmentation use cases.                 |
| Gateway                  | Network device or IP used to route traffic outside a subnet.                                   |
| Guest OS                 | Operating system running inside a VM.                                                          |
| HA                       | High Availability; capability to maintain or restore service after failure.                    |
| HCI                      | Hyperconverged Infrastructure; combines compute, storage, and virtualization.                  |
| Host                     | Physical server running the hypervisor and VMs.                                                |
| Hyper-V                  | Microsoft hypervisor platform.                                                                 |
| Hypervisor               | Software layer that runs and manages virtual machines.                                         |
| I/O                      | Input/Output; read/write operations between systems.                                           |
| IOPS                     | Input/Output Operations Per Second; storage performance metric.                                |
| iSCSI                    | IP-based SCSI protocol used for block storage access.                                          |
| KMS                      | Key Management Server; manages encryption keys.                                                |
| KVM                      | Kernel-based Virtual Machine; Linux virtualization technology underlying AHV.                  |
| LCM                      | Lifecycle Manager; Nutanix software/firmware inventory and upgrade framework.                  |
| Live migration           | Moving a running VM between hosts with minimal disruption.                                     |
| LUN                      | Logical Unit Number; traditional SAN block storage unit.                                       |
| MTTR                     | Mean Time To Resolution/Recovery; incident recovery performance metric.                        |
| MTU                      | Maximum Transmission Unit; largest packet size allowed on a network path.                      |
| NCC                      | Nutanix Cluster Check; health-check and diagnostics tool for Nutanix clusters.                 |
| NCI                      | Nutanix Cloud Infrastructure; Nutanix infrastructure platform family.                          |
| Node                     | Physical Nutanix server participating in a cluster.                                            |
| NUMA                     | Non-Uniform Memory Access; CPU/memory locality architecture relevant for VM sizing.            |
| OVS                      | Open vSwitch; virtual switching technology often associated with KVM-based environments.       |
| P1                       | Priority 1; highest-severity support incident.                                                 |
| Prism Central            | Centralized Nutanix management platform across clusters.                                       |
| Prism Element            | Nutanix management interface for a single cluster.                                             |
| QEMU                     | Emulator/virtualizer component used with KVM-based virtualization.                             |
| RCA                      | Root Cause Analysis; investigation of why an incident happened.                                |
| RDP                      | Remote Desktop Protocol; Windows remote access protocol.                                       |
| SCSI                     | Storage protocol/interface used for block devices.                                             |
| SED                      | Self-Encrypting Drive; disk with built-in encryption capability.                               |
| SLA                      | Service Level Agreement; contractual or operational service target.                            |
| Stargate                 | Nutanix storage service involved in VM I/O handling.                                           |
| Storage container        | Logical Nutanix storage construct for VM disks and policies.                                   |
| ToR switch               | Top-of-Rack switch; physical data center network switch.                                       |
| vCPU                     | Virtual CPU assigned to a VM.                                                                  |
| vDisk                    | Virtual disk attached to a VM.                                                                 |
| vGPU                     | Virtual GPU assigned to a VM.                                                                  |
| VLAN                     | Virtual LAN; logical network segmentation mechanism.                                           |
| VM                       | Virtual Machine; software-defined server running on a hypervisor.                              |
| VM HA                    | VM High Availability; restart/recovery behavior after host failure.                            |
| vNIC                     | Virtual Network Interface Card attached to a VM.                                               |
| vSphere                  | VMware virtualization platform suite.                                                          |
| VirtIO                   | Paravirtualized drivers used for optimized VM disk/network performance on KVM/AHV.             |
| Workload                 | Application, VM, or service running on infrastructure.                                         |

[1]: https://www.nutanix.com/content/dam/nutanix/documents/certifications/ahv-admin.pdf "AHV Administration Guide"
[2]: https://www.nutanix.com/content/dam/nutanix/documents/certifications/advanced-admin-aos.pdf "Acropolis Advanced Administration Guide"
