# AHV / Virtualization — VM Lifecycle

## 1. Short definition

The **VM lifecycle in Nutanix AHV** is the complete operational path of a virtual machine: **provisioning, configuring, powering on, monitoring, resizing, snapshotting, migrating, protecting, troubleshooting, recovering, and eventually deleting or decommissioning** the VM.

In Nutanix terms, this lifecycle is managed mainly through **Prism Element** for cluster-level operations and **Prism Central** for centralized, multi-cluster management, automation, policies, and broader operational visibility. Nutanix documentation describes VM management as including creating, updating, deleting, protecting, and monitoring VMs and their resources through Prism. ([Nutanix][1])

---

## 2. Clear explanation

A VM on AHV is not just “a server running on a hypervisor.” In an enterprise support context, it is a managed workload composed of several operational layers:

1. **Compute**

   * vCPU allocation
   * Memory allocation
   * Host placement
   * CPU compatibility
   * Scheduling and migration behavior

2. **Storage**

   * Virtual disks
   * Storage containers
   * Replication factor
   * Snapshots / recovery points
   * Data protection policies

3. **Networking**

   * vNICs
   * VLANs
   * Virtual networks
   * Trunk or access mode
   * IP, DNS, routing, firewall, and external network dependencies

4. **Guest OS integration**

   * Operating system health
   * Drivers
   * Nutanix Guest Tools, or **NGT**
   * VirtIO drivers, especially for Windows workloads
   * Application-level behavior

5. **Operational state**

   * Powered off
   * Powered on
   * Paused / suspended depending on context
   * Migrating
   * Snapshotting
   * Recovering
   * Failing to boot
   * Degraded performance

A basic AHV VM lifecycle looks like this:

```text
Create / import VM
        ↓
Configure CPU, memory, disks, NICs, images
        ↓
Install or boot guest OS
        ↓
Install drivers / NGT where applicable
        ↓
Power on and validate application reachability
        ↓
Monitor performance and availability
        ↓
Resize, migrate, snapshot, clone, or protect
        ↓
Troubleshoot incidents and escalations
        ↓
Recover, restore, migrate, or decommission
```

Nutanix documentation explicitly lists common VM actions after creation: start, shut down, launch console, update configuration, take snapshot, attach volume group, migrate, clone, and delete. It also notes that available actions depend on VM status, VM type, and user permissions. ([Nutanix][2])

For interview purposes, the key is not to memorize every button in Prism. The key is to explain that a VM lifecycle is a **Day 0, Day 1, and Day 2 operations model**:

| Phase | Meaning                 | Support focus                                                  |
| ----- | ----------------------- | -------------------------------------------------------------- |
| Day 0 | Architecture and design | Sizing, compatibility, HA, network design                      |
| Day 1 | Deployment              | VM creation, OS install, network attachment, validation        |
| Day 2 | Operations              | Monitoring, incidents, patching, resizing, migration, recovery |

As a support manager, you need to understand the lifecycle well enough to lead escalations, ask the right questions, detect whether the issue is compute, storage, network, hypervisor, guest OS, or customer application, and coordinate the right technical experts.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, VM lifecycle knowledge matters because most customer-impacting incidents eventually surface as VM symptoms:

* “The VM is down.”
* “The VM is slow.”
* “The VM cannot migrate.”
* “The VM lost network connectivity.”
* “The VM does not boot after maintenance.”
* “The VM snapshot failed.”
* “The VM cannot be restored.”
* “The VM is consuming too many resources.”
* “The customer says Nutanix caused an outage.”

Your job is not to be the deepest AHV engineer in the room. Your job is to **lead the support motion**:

* Make sure the team qualifies impact correctly.
* Drive structured triage.
* Prevent premature blame.
* Keep the customer informed.
* Escalate to SRE, Engineering, Product, or Account teams when needed.
* Ensure evidence is collected before risky changes.
* Track SLA, MTTR, customer sentiment, and technical resolution.

Nutanix AHV is central here because AHV is Nutanix’s native hypervisor, and Prism is the management plane customers use to operate VMs. Prism can manage VM lifecycle CRUD operations, and Prism Element tracks per-VM information to help locate issues faster. ([Nutanix][3])

For a manager, the important framing is:

> “I do not need to personally debug every hypervisor-level issue, but I need to understand the VM lifecycle well enough to lead the escalation, ask precise questions, challenge assumptions, and make sure the right diagnostic path is followed.”

That is the positioning you want: **technical escalation manager**, not Senior SRE individual contributor.

---

## 4. Key concepts

### 4.1 AHV

**AHV**, or **Acropolis Hypervisor**, is Nutanix’s native virtualization layer. It runs VMs on Nutanix clusters and integrates with AOS, Prism, storage, networking, and availability services.

Interview-ready phrasing:

> “AHV is the Nutanix-native hypervisor. In support terms, I see it as the virtualization control layer responsible for VM execution, host placement, lifecycle actions, migration, and integration with Prism and the Nutanix distributed platform.”

---

### 4.2 Prism Element vs Prism Central

**Prism Element** manages an individual Nutanix cluster.

**Prism Central** provides centralized management across clusters, policies, reporting, automation, and APIs.

For VM lifecycle:

* Prism Element is often used for cluster-level VM operations.
* Prism Central is more relevant for multi-cluster operations, policy-driven management, and enterprise visibility.
* Nutanix APIs are exposed through Prism Central and Prism Element depending on API version and function. Nutanix’s v4 APIs are positioned as part of the cloud-consumption and automation model in Prism Central. ([Nutanix][4])

Manager-level point:

> “In an escalation, I would clarify whether the issue is with the actual VM operation on the cluster, the Prism UI/API workflow, or a broader Prism Central policy or inventory issue.”

---

### 4.3 VM provisioning

Provisioning is the creation or import of a VM.

Typical configuration items:

* VM name
* vCPU count
* Cores per socket
* Memory
* Boot device
* Virtual disks
* CD-ROM / ISO
* Network adapters
* VLAN / subnet
* Guest OS type
* Image source
* Optional NGT installation
* Optional categories, policies, or protection rules

Support risks during provisioning:

* Wrong guest OS or driver assumptions
* Missing VirtIO drivers
* Wrong VLAN
* Incorrect boot order
* Insufficient storage
* CPU or memory overcommit concerns
* Permissions issue
* Image service problem
* Automation/API failure

---

### 4.4 VM power operations

Common actions include:

* Power on
* Graceful shutdown
* Hard power off
* Reboot
* Console access
* Reset

The support distinction is important:

| Action         | Support meaning                                                  |
| -------------- | ---------------------------------------------------------------- |
| Guest shutdown | OS-aware, graceful                                               |
| Power off      | Hypervisor-level, potentially abrupt                             |
| Reset          | Equivalent to forced restart                                     |
| Console access | Helps distinguish OS/application issue from access/network issue |

Interview-ready phrasing:

> “If a customer says a VM is unavailable, I would not immediately assume the VM is powered off. I would separate VM power state, guest OS state, network reachability, application health, and storage or host-level events.”

---

### 4.5 VM configuration updates

A VM can be updated after creation: CPU, memory, disks, network adapters, boot configuration, and other settings. Nutanix documentation notes that available actions depend on VM status, type, and permissions. ([Nutanix][2])

Some changes may require the VM to be powered off. Others may be hot-add capable depending on configuration, guest OS, and platform support.

Support angle:

* Was the VM recently resized?
* Did the issue start after adding CPU, memory, disk, or NIC?
* Was there a hot-add operation?
* Did the guest OS detect the new resource?
* Is the bottleneck inside the guest OS or at hypervisor/storage/network level?

---

### 4.6 VM snapshots and recovery points

A **snapshot** captures the VM state at a point in time. In Prism Central protection workflows, snapshots may be referred to as **recovery points**. Nutanix documentation states that AOS generates recovery points when a VM is protected by a protection policy. ([Nutanix][5])

Support angle:

Snapshots are useful for rollback and recovery, but they are not a substitute for a full backup strategy.

Escalation risks:

* Snapshot creation fails.
* Snapshot consolidation or deletion issues.
* Restore point unavailable.
* Customer assumes a snapshot is an application-consistent backup.
* Snapshot growth affects capacity.
* Snapshot restore brings back old application state but not external dependencies.

Interview-ready phrasing:

> “I would treat snapshots as a recovery mechanism within the platform, but I would clarify consistency requirements, retention, external backup integration, and whether the customer needs crash-consistent or application-consistent recovery.”

---

### 4.7 VM cloning

A **clone** creates a new VM based on an existing VM.

Common use cases:

* Test environment
* Reproduce a customer issue
* Golden image deployment
* Lab validation
* Pre-upgrade testing

Support risks:

* Duplicate hostname
* Duplicate IP
* Duplicate application identity
* Licensing issues
* Domain membership conflicts
* Security certificates copied incorrectly

Manager-level point:

> “In escalations, cloning can be useful for reproduction, but it must be controlled because duplicating production identity can create secondary incidents.”

---

### 4.8 VM migration

VM migration means moving a VM from one host or cluster to another.

Common types:

* Intra-cluster live migration
* Cross-cluster live migration
* Migration from another hypervisor to AHV
* Planned migration during maintenance
* Emergency migration during resource contention or host issues

Nutanix documentation describes on-demand cross-cluster live migration as allowing guest VMs and associated metadata, such as VM categories, to move across AHV clusters registered to the same or different Prism Central availability zones, with live migration aiming for zero VM downtime. ([Nutanix][6])

Important migration dependencies:

* CPU compatibility
* Network availability at destination
* Storage accessibility
* Host resources
* VM configuration
* Policies and categories
* Guest tools / drivers
* Cluster version compatibility

Nutanix documentation notes that for cross-cluster live migration across clusters with different CPU features, **Advanced Processor Compatibility**, or **APC**, can be required; otherwise, the destination cluster must support all CPU feature sets from the source. ([Nutanix][7])

Interview-ready phrasing:

> “For a failed migration, I would check whether the failure is caused by compute compatibility, resource availability, network mapping, storage accessibility, policy constraints, VM state, or Prism workflow/API issues.”

---

### 4.9 VM high availability

VM high availability determines what happens to VMs when a host fails.

Nutanix documentation describes that when an AHV host becomes unavailable, VMs that were running on the failed host restart on remaining hosts depending on available resources. It also describes a nondefault **Guarantee** mode that reserves resources to guarantee VM restart capacity during host failure scenarios. ([Nutanix][8])

Support angle:

* Did the host fail?
* Did VMs restart?
* Were there enough resources?
* Was HA guarantee enabled?
* Were some workloads not restarted due to resource constraints?
* Was the issue actually storage/network-related rather than compute host failure?

Manager-level point:

> “For HA incidents, I would focus on impact, restart behavior, capacity reservation, failure domain, timeline, and customer communication. The customer cares less about the theoretical HA model and more about which workloads restarted, which did not, why, and what prevents recurrence.”

---

### 4.10 Acropolis Dynamic Scheduler

**Acropolis Dynamic Scheduler**, or **ADS**, helps with initial VM placement and runtime optimization by using real-time statistics to decide where VMs and volume groups should run. Nutanix documentation says ADS is enabled by default and handles initial placement and runtime optimizations to give workloads better access to resources. ([Nutanix][9])

Support relevance:

* VM performance issues may relate to placement.
* Hotspots may require migration.
* Some VMs may not be automatically migrated, depending on configuration.
* Anti-affinity policies may affect placement.

Interview-ready phrasing:

> “I would consider whether the issue is isolated to one VM, one host, one storage path, one network segment, or a broader scheduling/resource-balancing problem.”

---

### 4.11 Nutanix Guest Tools and VirtIO

**Nutanix Guest Tools**, or **NGT**, is a software package bundled with AOS and installed inside guest VMs to enable advanced VM management functions. Nutanix documentation describes NGT as including the NGT installer, Nutanix Guest Agent service, Nutanix VirtIO package, and supporting components. ([Nutanix][10])

Support relevance:

* Missing drivers can affect boot, disk, or network behavior.
* Guest-agent issues can limit platform-level VM operations.
* Windows migrations to AHV often require careful driver handling.
* Guest integration affects operational visibility and manageability.

Interview-ready phrasing:

> “When troubleshooting a VM issue, I would separate hypervisor symptoms from guest OS integration issues. NGT and VirtIO are important because the VM may be running, but still lack the right drivers or agent-level functionality.”

---

### 4.12 VM networking

VM networking in AHV includes virtual NICs, VLANs, bridges, bonds, and external network dependencies.

Nutanix AHV networking best practices are actively maintained; the current document has updates through March 2026, which is relevant because networking behavior and recommended operational practices evolve over time. ([Nutanix][11])

VM network issues often look like VM issues:

* VM cannot reach gateway.
* VM cannot resolve DNS.
* VM has duplicate IP.
* VM attached to wrong VLAN.
* Security group / firewall blocks traffic.
* Physical switch trunk misconfigured.
* LACP or bond issue.
* Overlay or upstream network issue.
* Application port unreachable.

Interview-ready phrasing:

> “In support, I would avoid saying ‘the VM is down’ until I validate whether the VM is powered on, whether the guest OS is reachable from console, whether the vNIC is attached to the right network, and whether upstream VLAN/routing/firewall dependencies are healthy.”

---

## 5. How it appears in a real escalation

### Scenario 1 — VM unavailable after host failure

Customer says:

> “Several production VMs went down after a Nutanix node failure. We expected HA to protect us.”

Support manager response:

1. Confirm business impact:

   * Which applications?
   * How many VMs?
   * Production or non-production?
   * Is the outage ongoing?
   * SLA/customer impact?

2. Establish timeline:

   * When did host failure occur?
   * When were VMs detected down?
   * Which VMs restarted?
   * Which did not?

3. Split the problem:

   * Host failure?
   * VM restart failure?
   * Capacity issue?
   * Storage issue?
   * Network issue after restart?
   * Guest OS/application issue?

4. Ask technical team to validate:

   * Cluster alerts
   * HA configuration
   * Available resources
   * VM events
   * Host state
   * Prism tasks
   * Recent changes

5. Communicate clearly:

   * “We are validating whether the VMs failed to restart due to HA capacity reservation, host-level resource availability, or a secondary dependency such as storage or networking.”

Good escalation behavior:

> You do not overpromise. You control the bridge, assign owners, maintain a timeline, and ensure the customer receives factual updates.

---

### Scenario 2 — VM performance degradation

Customer says:

> “A critical VM is extremely slow after migration to AHV.”

Triage path:

* Is the issue CPU, memory, disk, or network?
* Was the VM recently migrated from VMware?
* Are VirtIO drivers installed?
* Is NGT installed and healthy?
* Is the VM oversized or undersized?
* Is there CPU ready / contention equivalent behavior?
* Is storage latency high?
* Is the issue on one VM, one host, or many VMs?
* Are there noisy neighbors?
* Did ADS move workloads recently?
* Is the application itself saturated?

Manager-level response:

> “I would ask the team to compare VM metrics, host metrics, storage latency, network throughput, and guest OS counters. I would also check whether the issue started after a lifecycle event such as migration, resize, snapshot, restore, or driver change.”

---

### Scenario 3 — VM migration fails during maintenance

Customer says:

> “We cannot evacuate a host because some VMs fail to migrate.”

Triage path:

* Is this live migration within cluster or cross-cluster?
* Are CPU features compatible?
* Is APC enabled if required?
* Is the destination network available?
* Are there enough resources on target hosts?
* Are there pinned devices, passthrough devices, vGPU, or special configurations?
* Are there active snapshots or tasks?
* Is the VM in a supported state?
* Are Prism tasks failing or timing out?

Manager-level response:

> “I would separate this into migration pre-checks, VM-specific blockers, cluster capacity, network mapping, and compatibility constraints. For customer communication, I would provide a safe maintenance recommendation rather than forcing risky moves during an active window.”

---

### Scenario 4 — VM cannot boot after restore

Customer says:

> “We restored a VM, but it does not boot.”

Triage path:

* Was the restore from snapshot/recovery point/backup?
* Is the boot disk attached?
* Is boot order correct?
* Did the VM come from another hypervisor?
* Are drivers available?
* Is the guest OS seeing the disk?
* Is this UEFI/BIOS mismatch?
* Is the restored VM attached to the right network?
* Was application consistency required?
* Are external dependencies missing?

Manager-level response:

> “I would manage this as a recovery escalation: first recover service safely, then preserve evidence, then determine whether the issue is boot configuration, driver compatibility, corrupted guest OS, or restore workflow.”

---

## 6. Triage questions I should ask

### Impact and priority

1. How many VMs are affected?
2. Are these production workloads?
3. Is there active customer impact?
4. Is the issue outage, degradation, or operational blocker?
5. What is the business service behind the VM?
6. What SLA or customer commitment is at risk?

### Timeline

7. When did the issue start?
8. What was the last known good state?
9. Was there a recent change?
10. Was there maintenance, migration, resize, upgrade, restore, or snapshot activity?
11. Did the issue start after a host, cluster, storage, or network event?

### VM state

12. Is the VM powered on in Prism?
13. Is the guest OS responsive from console?
14. Is the application running inside the guest?
15. Is the issue visible from inside the OS?
16. Is the issue visible from Prism metrics?

### Compute

17. Which AHV host is running the VM?
18. Is the host healthy?
19. Is there CPU or memory contention?
20. Was the VM recently resized?
21. Are there special CPU, NUMA, passthrough, or vGPU requirements?

### Storage

22. Are virtual disks attached?
23. Is storage latency elevated?
24. Is the storage container healthy?
25. Are there snapshots or recovery points involved?
26. Is capacity under pressure?
27. Was the VM cloned, restored, or migrated?

### Networking

28. Is the VM connected to the correct virtual network?
29. Is the VLAN correct?
30. Can the VM reach its gateway?
31. Is DNS working?
32. Are other VMs on the same network affected?
33. Was there any upstream switch, firewall, or routing change?

### Guest OS and tools

34. Are NGT and VirtIO installed where applicable?
35. Are guest drivers healthy?
36. Is the OS logging disk, network, or driver errors?
37. Is the guest firewall blocking traffic?
38. Is the application healthy inside the VM?

### Migration / lifecycle actions

39. Did a migration, clone, snapshot, or restore happen recently?
40. Is there an active Prism task?
41. Did the task fail, time out, or partially complete?
42. Is the VM locked by another operation?
43. Is there an automation/API workflow involved?

### Customer management

44. What workaround is acceptable?
45. Can we reboot, migrate, restore, or clone?
46. Is there a maintenance window?
47. Who approves risky operations?
48. What update frequency does the customer expect?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you manage an escalation where multiple VMs are down after a host failure?
2. How do you communicate with a customer during a VM outage?
3. How do you avoid finger-pointing between virtualization, network, storage, and application teams?
4. How do you decide when to escalate to Engineering?
5. How do you balance technical depth with team leadership?
6. How would you coach an engineer handling a difficult VM lifecycle escalation?
7. How do you track SLA and MTTR in virtualization-related incidents?
8. How would you run a post-incident review after a failed migration or HA event?

### Technical / SRE interview

9. Explain the lifecycle of a VM in AHV.
10. What would you check if a VM does not power on?
11. What would you check if a VM is powered on but unreachable?
12. What is the difference between VM power state, guest OS health, and application health?
13. What are common causes of VM migration failure?
14. What role do NGT and VirtIO drivers play?
15. How would you troubleshoot VM performance degradation?
16. What is the difference between snapshot, clone, backup, and recovery point?
17. How does HA matter in VM lifecycle?
18. What is the role of Prism Element and Prism Central?
19. How would you approach a failed restore?
20. What information would you collect before escalating to Engineering?

### Panel / advanced escalation interview

21. A customer says Nutanix caused an outage after planned maintenance. How do you respond?
22. A VM migration failed during a critical maintenance window. What do you do?
23. A customer demands RCA before the issue is fully understood. How do you handle it?
24. Engineering says they need logs, but the customer wants immediate recovery. How do you prioritize?
25. How do you prevent repeated escalations for the same VM lifecycle issue?
26. How would you define operational KPIs for VM lifecycle support?
27. How do you build a support team capable of handling AHV escalations globally?

---

## 8. Model answers in English

### Q1. Explain the lifecycle of a VM in AHV.

**Model answer:**

> “In AHV, I think of the VM lifecycle as the full operational journey of a workload: creation or import, configuration of compute, memory, disks and networking, guest OS installation or boot, installation of required guest tools or drivers, power operations, monitoring, resizing, snapshotting, cloning, migration, protection, recovery, and finally decommissioning.
>
> From a support perspective, the important point is that each lifecycle phase introduces different failure modes. A provisioning issue may be caused by image, network, or permissions. A runtime issue may be compute, storage, network, guest OS, or application-related. A migration issue may involve resource availability, CPU compatibility, VM state, or network mapping.
>
> As a support manager, I would not try to debug every command myself, but I would make sure the team follows a structured triage model and communicates clearly with the customer.”

---

### Q2. A customer says a VM is down. What do you check first?

**Model answer:**

> “I would first clarify what ‘down’ means. Is the VM powered off in Prism, is the guest OS unresponsive, or is the application unreachable? Those are different problems.
>
> I would check the VM power state, console access, recent lifecycle tasks, host health, cluster alerts, storage latency, and network connectivity. If the VM is powered on and accessible from console, then the issue may be guest OS, network, DNS, firewall, or application-related. If the VM is not powered on, I would check recent host events, HA behavior, capacity, and failed Prism tasks.
>
> In parallel, I would manage impact: number of VMs affected, production severity, customer communication, and whether a safe workaround such as restart, migration, or restore is available.”

---

### Q3. How would you handle a VM performance escalation?

**Model answer:**

> “I would structure the investigation around compute, memory, storage, network, guest OS, and application. First, I would determine whether the issue affects one VM, several VMs on the same host, several VMs on the same storage container, or a broader cluster.
>
> Then I would compare Prism metrics with guest OS metrics. High CPU usage inside the guest is different from hypervisor contention. High disk latency may point to storage or workload behavior. Packet loss or retransmissions may point to networking.
>
> I would also ask about recent lifecycle changes: migration, resize, snapshot, restore, upgrade, driver change, or application deployment. As a manager, my role is to ensure the team narrows the domain quickly and avoids random troubleshooting.”

---

### Q4. What are common causes of VM migration failure?

**Model answer:**

> “Common causes include insufficient destination resources, CPU compatibility issues, network mapping problems, VM state restrictions, attached devices that cannot migrate, policy constraints, active operations on the VM, or Prism workflow/API failures.
>
> For cross-cluster migration, I would pay special attention to CPU compatibility, network availability on the destination, and whether the VM’s metadata and policies can move correctly. Nutanix documentation specifically calls out CPU feature compatibility and Advanced Processor Compatibility for certain cross-cluster migration scenarios.
>
> Operationally, I would ask the team for the exact failed task, error message, VM configuration, source and destination host or cluster, and whether the failure is isolated to one VM or systemic.”

---

### Q5. What is the difference between a snapshot, clone, backup, and recovery point?

**Model answer:**

> “A snapshot captures a point-in-time state of a VM and is often used for short-term rollback or recovery. In Prism Central protection workflows, Nutanix may refer to protected VM snapshots as recovery points. A clone creates a new VM from an existing VM or snapshot, usually for testing, reproduction, or deployment. A backup is usually a broader data protection mechanism with retention, external storage, and recovery objectives.
>
> In support, I would be careful not to let customers assume that a snapshot automatically equals a full backup strategy. I would clarify retention, consistency, restore requirements, and whether the application needs application-consistent protection.”

---

### Q6. How do NGT and VirtIO matter in the VM lifecycle?

**Model answer:**

> “NGT and VirtIO matter because the hypervisor may be healthy while the guest OS still lacks the right integration or drivers. Nutanix Guest Tools provide guest-level integration capabilities, and VirtIO drivers are especially important for Windows VMs running on AHV.
>
> In an escalation, missing or unhealthy guest tools can show up as boot issues, disk or network driver issues, limited manageability, or problems after migration from another hypervisor. I would ask whether the VM was newly created, migrated, restored, or upgraded, and whether the correct drivers and tools are installed.”

---

### Q7. How would you manage a customer escalation where VMs did not restart after host failure?

**Model answer:**

> “I would first manage the incident as an availability escalation: confirm customer impact, identify affected VMs, establish a timeline, and assign technical owners for compute, storage, networking, and guest OS validation.
>
> Technically, I would ask the team to validate the failed host, HA configuration, available resources, VM restart attempts, cluster alerts, and Prism events. I would also confirm whether the VMs failed to restart or restarted but remained unreachable because of a secondary dependency.
>
> For communication, I would avoid speculation. I would give factual updates: what we know, what we are validating, what mitigation is available, and when the next update will be provided. After recovery, I would drive RCA, corrective actions, and support readiness improvements.”

---

### Q8. How do you position yourself if you are not a Senior SRE?

**Model answer:**

> “My value is not that I am the deepest individual contributor on every AHV internals topic. My value is that I can lead enterprise support teams through complex incidents, ask technically precise questions, organize the escalation, communicate with customers, and ensure the right experts are engaged.
>
> I understand the VM lifecycle well enough to distinguish compute, storage, network, guest OS, hypervisor, and application domains. That allows me to challenge assumptions, reduce MTTR, and protect customer trust while the technical team performs deeper diagnosis.”

---

## 9. Connection with my experience

Your background maps well to this topic if you frame it correctly.

### Your SaaS/cloud/incident management experience

You already understand:

* 24/7 operations
* Incident severity
* SLA ownership
* MTTR reduction
* Escalation management
* Customer communication
* Monitoring
* Dashboards
* Jira / Confluence workflows
* Support handoffs
* Post-incident reviews
* Cloud service dependencies

Now map that to AHV VM lifecycle:

| Your experience             | Nutanix VM lifecycle equivalent                    |
| --------------------------- | -------------------------------------------------- |
| SaaS service down           | VM/application unavailable                         |
| Cloud instance unhealthy    | AHV VM powered on but guest/application degraded   |
| Kubernetes pod rescheduling | VM placement/migration/HA restart                  |
| Cloud monitoring alert      | Prism alert / VM metric / cluster alert            |
| SLA breach risk             | Enterprise support severity management             |
| Incident bridge             | Customer escalation bridge                         |
| Runbooks                    | VM lifecycle triage playbooks                      |
| RCA                         | Post-incident review after VM/host/migration issue |
| Capacity planning           | Host/cluster resource planning                     |
| Release/change issue        | VM resize, migration, upgrade, or policy change    |

Strong positioning:

> “Although my current environment is SaaS/cloud rather than Nutanix AHV specifically, the operational patterns are familiar: workload lifecycle, health checks, resource contention, dependency mapping, incident command, customer communication, and post-incident improvement. My current focus is closing the product-specific knowledge gap around AHV, Prism, AOS, storage, and networking.”

---

## 10. Minimum I need to memorize

Memorize this as your verbal core:

1. **AHV is Nutanix’s native hypervisor.**
2. **Prism Element manages cluster-level VM operations; Prism Central provides centralized management, policy, visibility, and automation.**
3. **VM lifecycle = create, configure, power, monitor, resize, snapshot, clone, migrate, protect, recover, delete.**
4. **A VM issue must be separated into power state, guest OS state, network reachability, application health, and platform health.**
5. **Common escalation domains: compute, memory, storage, network, guest OS, application, Prism task/API, recent change.**
6. **Snapshots/recovery points are point-in-time recovery mechanisms, not automatically a complete backup strategy.**
7. **Migration failures often involve resources, CPU compatibility, network mapping, VM state, attached devices, or policy constraints.**
8. **HA behavior depends on host failure, available resources, and HA configuration/reservation.**
9. **NGT and VirtIO are important for guest integration and driver behavior.**
10. **As a manager, your role is structured triage, escalation coordination, customer communication, and MTTR/SLA ownership.**

Your 30-second answer:

> “The AHV VM lifecycle covers how a VM is created, configured, powered, monitored, modified, migrated, protected, recovered, and eventually deleted. In support, the critical skill is to separate VM power state, guest OS health, application health, network reachability, storage behavior, host health, and recent lifecycle actions. For a Worldwide Support Manager, this matters because many enterprise escalations present as ‘the VM is down’ or ‘the VM is slow,’ but the root cause may sit in compute, storage, network, guest OS, Prism workflow, or customer application. My role is to drive structured triage, coordinate the right experts, communicate clearly with the customer, and reduce MTTR.”

---

## 11. Advanced / optional level

You do **not** need to master these deeply before the interview, but you should recognize them:

### Advanced AHV internals

* AHV host internals
* KVM/QEMU-level behavior
* Libvirt concepts
* VM process-level troubleshooting
* AHV CLI commands
* Log locations and interpretation
* Scheduler internals
* CPU pinning / NUMA optimization
* Advanced memory overcommit behavior

### Advanced networking

* Open vSwitch internals
* Bridge and bond design
* LACP troubleshooting
* Trunk vs access port edge cases
* Overlay networking
* Network Function Virtualization
* Flow / microsegmentation integration

### Advanced storage

* Stargate internals
* OpLog / extent store concepts
* Data locality
* Replication factor behavior
* Erasure coding
* Snapshot chain internals
* Volume groups
* Storage performance counters

### Advanced data protection

* Protection domains
* Async replication
* NearSync
* Metro Availability
* Leap
* Recovery plans
* RPO/RTO design
* Application-consistent recovery

### Advanced automation

* Prism Central APIs
* v4 API lifecycle operations
* Calm automation
* Terraform / PowerShell / SDK workflows
* API-driven VM updates
* RBAC and audit workflows

How to position this:

> “I am familiar with the operational relevance of these areas and know when they matter in escalations. I would rely on senior SREs or Engineering for deep internals, while ensuring the right evidence, logs, customer timeline, and impact data are collected.”

---

## 12. Final checklist

Before an interview, be able to answer these confidently:

* Can I explain AHV in one sentence?
* Can I explain Prism Element vs Prism Central?
* Can I list the VM lifecycle stages?
* Can I troubleshoot “VM down” without jumping to conclusions?
* Can I distinguish power state, guest OS state, network reachability, and application health?
* Can I explain why snapshots are not the same as full backups?
* Can I name common migration failure causes?
* Can I explain why NGT and VirtIO matter?
* Can I describe HA behavior after host failure?
* Can I connect this to SLA, MTTR, incident command, and customer communication?
* Can I position myself as a technical escalation manager rather than a Senior SRE?
* Can I ask good triage questions under pressure?

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword        | Meaning                                                                                                            |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| ADS                             | Acropolis Dynamic Scheduler; Nutanix service for VM placement and runtime resource optimization.                   |
| AHV                             | Acropolis Hypervisor; Nutanix’s native hypervisor.                                                                 |
| AOS                             | Acropolis Operating System; Nutanix software layer providing distributed storage and platform services.            |
| APC                             | Advanced Processor Compatibility; helps migration across clusters with different CPU feature sets.                 |
| API                             | Application Programming Interface; used to automate VM lifecycle operations.                                       |
| Application-consistent snapshot | Snapshot coordinated with the application to improve recovery consistency.                                         |
| Availability Zone / AZ          | Logical availability boundary used in multi-site or multi-cluster architectures.                                   |
| Backup                          | Data protection copy usually designed for longer retention and external recovery.                                  |
| BIOS                            | Legacy firmware boot mode used by some VMs.                                                                        |
| Bond                            | Logical grouping of physical NICs for redundancy or throughput.                                                    |
| Boot order                      | Sequence of devices a VM uses to start its operating system.                                                       |
| Bridge                          | Virtual switching component used to connect VM networking to physical networking.                                  |
| Capacity reservation            | Reserved cluster resources used to improve restart guarantees during failures.                                     |
| Clone                           | New VM created from an existing VM or snapshot.                                                                    |
| Cluster                         | Group of Nutanix nodes working together as one platform.                                                           |
| Compute                         | CPU and memory resources used by a VM.                                                                             |
| Console                         | Direct VM screen access through Prism, useful when network access fails.                                           |
| CPU compatibility               | Requirement that destination hosts support the CPU features needed by a VM.                                        |
| CRUD                            | Create, Read, Update, Delete; basic lifecycle operations.                                                          |
| CVM                             | Controller VM; Nutanix VM running core storage/platform services on each node.                                     |
| Data protection                 | Policies and mechanisms used to protect and recover workloads.                                                     |
| Day 0                           | Design and architecture phase before deployment.                                                                   |
| Day 1                           | Deployment and initial configuration phase.                                                                        |
| Day 2                           | Ongoing operations, monitoring, troubleshooting, and lifecycle management.                                         |
| DNS                             | Domain Name System; translates names to IP addresses.                                                              |
| Escalation                      | Process of raising an issue to higher technical or management levels.                                              |
| Guest OS                        | Operating system running inside the VM.                                                                            |
| HA                              | High Availability; capability to restart or maintain workloads after failure.                                      |
| HA Guarantee mode               | Nutanix HA mode that reserves resources to guarantee VM restart capacity.                                          |
| HCI                             | Hyperconverged Infrastructure; combines compute, storage, networking, and virtualization in one platform.          |
| Host                            | Physical Nutanix node running AHV and VMs.                                                                         |
| Hot-add                         | Adding resources such as CPU, memory, or disks while a VM is running, if supported.                                |
| Incident bridge                 | Live coordination call or channel during a major incident.                                                         |
| ISO                             | Disk image commonly used to install operating systems or tools.                                                    |
| KVM                             | Kernel-based Virtual Machine; Linux virtualization technology underlying many hypervisors, including AHV concepts. |
| LACP                            | Link Aggregation Control Protocol; used for network link aggregation.                                              |
| Live migration                  | Moving a running VM with little or no downtime.                                                                    |
| MTTR                            | Mean Time To Resolve or Restore; key support performance metric.                                                   |
| NGT                             | Nutanix Guest Tools; guest VM tools for advanced management and integration.                                       |
| NIC                             | Network Interface Card; physical or virtual network interface.                                                     |
| NUMA                            | Non-Uniform Memory Access; CPU/memory locality architecture relevant for performance tuning.                       |
| OVS                             | Open vSwitch; virtual switching technology commonly associated with AHV networking.                                |
| Prism Central                   | Centralized Nutanix management plane for multiple clusters, policies, automation, and visibility.                  |
| Prism Element                   | Nutanix management interface for an individual cluster.                                                            |
| Prism task                      | Operation tracked in Prism, such as migration, snapshot, clone, or power action.                                   |
| Recovery point                  | Point-in-time VM recovery state, often used in Prism Central protection workflows.                                 |
| RF                              | Replication Factor; number of data copies stored for resilience.                                                   |
| RCA                             | Root Cause Analysis; formal investigation after an incident.                                                       |
| RPO                             | Recovery Point Objective; maximum acceptable data loss.                                                            |
| RTO                             | Recovery Time Objective; maximum acceptable recovery time.                                                         |
| SLA                             | Service Level Agreement; contractual or operational service target.                                                |
| Snapshot                        | Point-in-time capture of VM state or disk state.                                                                   |
| Storage container               | Logical storage construct where VM disks are placed.                                                               |
| Trunk mode                      | Network mode where a vNIC can carry multiple VLANs.                                                                |
| UEFI                            | Modern firmware boot mode used by many VMs.                                                                        |
| vCPU                            | Virtual CPU assigned to a VM.                                                                                      |
| vDisk                           | Virtual disk attached to a VM.                                                                                     |
| vGPU                            | Virtual GPU assigned to a VM for graphics or compute acceleration.                                                 |
| VLAN                            | Virtual Local Area Network; network segmentation mechanism.                                                        |
| VM                              | Virtual Machine; software-defined server running on a hypervisor.                                                  |
| VM lifecycle                    | Full operational path of a VM from creation to decommissioning.                                                    |
| vNIC                            | Virtual Network Interface Card attached to a VM.                                                                   |
| VirtIO                          | Paravirtualized driver framework important for AHV guest performance and compatibility.                            |
| Workload                        | Application or service running inside a VM or set of VMs.                                                          |

[1]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2038-AHV%3Aahv-virtual-machine-management.html&utm_source=chatgpt.com "AHV Virtual Machine Management - portal.nutanix.com"
[2]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_8%3Awc-vm-manage-acropolis-wc-t.html&utm_source=chatgpt.com "AHV 6.8 - Managing a VM (AHV) - portal.nutanix.com"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2042-Prism%3Avirtual-machine-management.html&utm_source=chatgpt.com "Virtual Machine Management - portal.nutanix.com"
[4]: https://www.nutanix.dev/api-reference-v4/?utm_source=chatgpt.com "API Reference v4 Introduction - nutanix.dev"
[5]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v10_3%3Awc-cluster-vm-snapshot-ahv-c.html?utm_source=chatgpt.com "AHV 10.3 - Virtual Machine Snapshots - portal.nutanix.com"
[6]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_8%3Amul-cluster-cclm-introduction-pc-c.html&utm_source=chatgpt.com "AHV 6.8 - On-Demand Cross-Cluster Live Migration - portal.nutanix.com"
[7]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v10_3%3Amul-cluster-cclm-requirements-r.html&utm_source=chatgpt.com "On-Demand Cross-Cluster Live Migration (OD-CCLM) Requirements"
[8]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2156-NC2-on-Azure%3Atn-nutanix-ahv-virtual-machine-high-availability.html&utm_source=chatgpt.com "Nutanix AHV Virtual Machine High Availability"
[9]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anutanix-acropolis-dynamic-scheduler.html&utm_source=chatgpt.com "Nutanix Acropolis Dynamic Scheduler"
[10]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v7_3%3Aman-nutanix-guest-tool-c.html&utm_source=chatgpt.com "Prism 7.3 - Nutanix Guest Tools"
[11]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2071-AHV-Networking%3ABP-2071-AHV-Networking&utm_source=chatgpt.com "Nutanix AHV Networking Best Practices"
