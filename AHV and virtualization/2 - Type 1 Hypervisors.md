# AHV / Virtualization — Type 1 Hypervisors

## 1. Short definition

A **Type 1 hypervisor**, also called a **bare-metal hypervisor**, is virtualization software that runs directly on physical server hardware and allows multiple virtual machines to share CPU, memory, storage, and networking resources.

In the Nutanix world, **AHV — Acropolis Hypervisor — is Nutanix’s native Type 1 hypervisor**, integrated with **Nutanix Cloud Infrastructure**, **AOS**, distributed storage, networking, and **Prism** management. Nutanix describes AHV as the default virtualization option for Nutanix HCI, managed through Prism and supporting enterprise workflows such as VM operations, live migration, high availability, and virtual networking. ([Nutanix][1])

---

## 2. Clear explanation

A hypervisor is the layer that allows several **virtual machines** to run on the same physical host. Each VM believes it has its own CPU, memory, disk, and network interfaces, but those resources are actually abstracted and scheduled by the hypervisor.

There are two common categories:

**Type 1 hypervisors** run directly on physical hardware. Examples include **Nutanix AHV**, **VMware ESXi**, **Microsoft Hyper-V**, and **KVM-based enterprise platforms**. They are used in production data centers because they provide better performance, isolation, and operational control.

**Type 2 hypervisors** run on top of a general-purpose operating system. Examples include VirtualBox, VMware Workstation, or Parallels. They are useful for labs, development, and desktop virtualization, but they are not normally the foundation of enterprise server platforms.

For Nutanix, the important point is that AHV is not just “a hypervisor installed somewhere.” It is part of the Nutanix HCI architecture. Nutanix positions AHV as built into **Nutanix Cloud Infrastructure**, not licensed as a standalone product, and integrated with storage, networking, and management services through **Prism**. ([Nutanix][2])

Technically, AHV is based on **Linux, KVM, and QEMU**. In AHV deployments, Nutanix uses a **Controller VM**, or **CVM**, on each node to provide the distributed storage and data services layer. Nutanix Bible describes AHV as based on Linux, QEMU, and KVM, with guest VMs using hardware virtualization. It also describes the CVM architecture, where storage devices are passed through to the CVM using PCI passthrough. ([NutanixBible.com][3])

For interview purposes, you do not need to explain AHV like a kernel engineer. You need to explain it as an enterprise support leader:

> AHV is the virtualization layer in Nutanix HCI. It runs directly on the hosts, provides VM lifecycle, compute scheduling, virtual networking, HA, and live migration capabilities, while Nutanix AOS and the CVMs provide the distributed storage and platform services behind it. Prism gives operations teams a single management plane for the cluster.

That is the level expected from a **technical escalation manager**.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Worldwide Support Manager at Nutanix**, AHV matters because many enterprise escalations will sit at the boundary between:

* Virtualization
* Storage
* Networking
* Hardware
* Guest OS behavior
* Customer operations
* Business-critical SLAs

A customer may say, “Nutanix is down,” but the actual issue could be:

* A VM HA restart after host failure
* A storage latency spike affecting VM I/O
* A virtual NIC / VLAN misconfiguration
* An AHV host in maintenance mode
* A CVM issue impacting data services
* A live migration failure
* A resource contention problem
* A guest OS driver issue
* A third-party integration issue with backup, monitoring, VMware migration, or disaster recovery

As a support manager, your role is not to personally debug every AHV command. Your role is to **structure the escalation**, ask the right questions, coordinate SREs and SMEs, manage customer communication, and ensure the incident moves toward containment, root cause, and prevention.

You should be able to say:

> “I understand AHV as the compute virtualization layer of Nutanix HCI. In an escalation, I would first separate whether the symptoms are VM-level, host-level, cluster-level, storage-path-related, or network-related. Then I would align the right SMEs, validate customer impact, protect data availability, and drive the incident through mitigation, RCA, and follow-up actions.”

That answer positions you as a manager with technical control, not as someone pretending to be a Senior SRE.

---

## 4. Key concepts

### Type 1 hypervisor

A Type 1 hypervisor runs directly on physical hardware. It owns the physical resources and exposes virtual resources to VMs. In enterprise environments, Type 1 hypervisors are preferred because they reduce unnecessary layers and provide stronger performance, reliability, and isolation.

Interview phrasing:

> “A Type 1 hypervisor is the bare-metal virtualization layer. It abstracts physical compute, memory, storage, and networking resources and presents them to VMs in a controlled and isolated way.”

---

### AHV

**AHV** is Nutanix’s native hypervisor. It is integrated with Nutanix HCI, AOS, Prism, and Nutanix’s distributed storage architecture. Nutanix’s current documentation describes AHV as the default option for Nutanix HCI, with familiar enterprise workflows such as VM operations, live migration, VM high availability, and virtual network management. ([Nutanix][1])

Key point:

> AHV is not only a virtualization engine; it is part of the Nutanix operating model.

---

### AOS

**AOS**, historically associated with Acropolis Operating System, is the Nutanix platform software layer that provides distributed storage, VM management services, resource management, and core HCI functionality.

From a support perspective, AHV and AOS are tightly related. If VMs are slow, the problem might appear at the AHV level but originate from storage, replication, CVM health, metadata services, network congestion, or physical hardware.

---

### CVM

The **Controller VM** is central to Nutanix architecture. Each Nutanix node runs a CVM that participates in the distributed storage and control plane. The CVM is not “just another workload VM”; it is part of the Nutanix platform.

Nutanix Bible describes AHV deployments where the CVM runs as a VM and disks are presented using PCI passthrough, allowing direct access to the storage controller and attached devices. ([NutanixBible.com][3])

Interview-safe phrasing:

> “In Nutanix, the CVM is a critical platform component. It provides local access to distributed storage services and participates in cluster-wide data services. In an escalation, CVM health is one of the first things I would want the technical team to validate.”

---

### Prism Element and Prism Central

**Prism Element** manages a single Nutanix cluster.

**Prism Central** provides centralized management across multiple clusters and broader operational visibility.

For a support manager, Prism is important because it is often where the customer sees alerts, host health, VM status, capacity, performance, and operational workflows.

---

### VM lifecycle

Typical VM lifecycle operations include:

* Create VM
* Power on / power off
* Clone
* Snapshot
* Migrate
* Resize CPU or memory
* Attach disks
* Configure NICs
* Delete VM
* Recover or restart VM

In an escalation, lifecycle failures often become urgent because they block production changes, migrations, disaster recovery, or customer application recovery.

---

### Live migration

Live migration means moving a running VM from one host to another with little or no perceived downtime. Nutanix documentation explicitly lists live migration as one of the familiar workflows AHV provides. ([Nutanix][1])

Support impact:

* Useful for maintenance
* Useful for load balancing
* Useful during hardware replacement
* Can fail because of host compatibility, network issues, resource constraints, or VM-specific constraints

---

### VM High Availability

VM HA is the ability to restart VMs on surviving hosts after a host or node failure. The objective is to reduce application downtime.

Support impact:

* Customers care about whether VMs restarted
* Which VMs were impacted
* How long recovery took
* Whether HA behaved as designed
* Whether capacity was sufficient to restart workloads elsewhere

---

### Virtual networking

Virtual networking connects VMs to the physical network through virtual NICs, virtual switches, bridges, VLANs, and uplinks.

In AHV escalations, network issues may appear as:

* VM cannot reach gateway
* VLAN mismatch
* Packet loss
* DNS issues
* Storage/CVM traffic affected
* Management plane inaccessible
* Backup or replication failure

A manager does not need to know every low-level command, but should understand the troubleshooting domains: VM NIC, virtual network, host uplinks, VLAN tagging, physical switch, routing, firewall, DNS, and external dependencies.

---

### Storage path

In Nutanix, storage is not just local disk attached to a VM. Nutanix provides a distributed storage fabric through AOS and CVMs.

So when a VM has I/O latency, the issue may involve:

* Guest OS
* Virtual disk
* AHV host
* CVM
* Storage container
* Replication factor
* Disk health
* Network between nodes
* Data locality
* Cluster capacity
* Background operations

This is where AHV differs from traditional three-tier infrastructure. The virtualization layer and storage layer are integrated.

---

### Resource contention

Resource contention happens when too many workloads compete for CPU, memory, storage I/O, or network bandwidth.

Common symptoms:

* High CPU ready / scheduling delay
* Memory pressure
* VM latency
* Slow application response
* Noisy neighbor behavior
* Backup windows degrading performance
* Migration or maintenance operations slowing production workloads

For a manager, the key is to drive the team to separate **capacity issue**, **performance issue**, **configuration issue**, and **platform fault**.

---

## 5. How it appears in a real escalation

### Scenario 1 — Customer reports “all VMs are slow”

A large enterprise customer opens a Sev1 escalation:

> “All production VMs on our Nutanix cluster are slow. Application response time has increased dramatically. We are breaching internal SLA.”

The customer may blame AHV because VMs are affected. But the root cause could be anywhere.

A good support manager would structure the escalation like this:

1. **Clarify business impact**

   * Which applications are impacted?
   * How many VMs?
   * Is this production?
   * Is there customer-facing impact?
   * When did it start?
   * Is it getting worse?

2. **Define scope**

   * One VM, several VMs, one host, one cluster, multiple clusters?
   * Only AHV VMs or also external services?
   * All networks or one VLAN?
   * All storage containers or one workload type?

3. **Separate layers**

   * Guest OS metrics
   * VM metrics
   * AHV host metrics
   * CVM health
   * Storage latency
   * Physical disk health
   * Network health
   * Recent changes

4. **Contain**

   * Stop risky maintenance
   * Pause heavy backup jobs if safe
   * Avoid unnecessary reboots
   * Move workloads if appropriate
   * Engage storage/network/AHV SMEs

5. **Communicate**

   * Provide regular updates
   * Confirm what is known, unknown, and being tested
   * Avoid premature RCA
   * Keep customer leadership informed

Good interview line:

> “In that type of escalation, I would avoid assuming the hypervisor is the root cause just because VMs are impacted. I would drive a layered triage: guest, VM, host, CVM, storage, network, recent changes, and customer impact.”

---

### Scenario 2 — Host failure and VM HA behavior

A host fails in a Nutanix AHV cluster. Some VMs restart successfully, but others do not.

Possible causes:

* Insufficient remaining cluster capacity
* HA policy or configuration issue
* VM pinned to specific resources
* Storage or network issue
* Multiple simultaneous failures
* Ongoing maintenance or upgrade
* Misunderstood customer expectation

Managerial response:

* Confirm impacted VMs
* Confirm current host and cluster state
* Confirm whether data integrity is at risk
* Engage AHV and platform SMEs
* Establish workaround or recovery plan
* Communicate realistic recovery status
* Drive RCA after stabilization

Interview line:

> “I would focus first on service restoration and data safety, then validate whether HA behaved according to configuration and available capacity. After mitigation, I would drive an RCA covering failure trigger, HA decision path, capacity assumptions, and preventive actions.”

---

### Scenario 3 — Live migration failure during planned maintenance

Customer tries to place a host into maintenance mode. Some VMs fail to migrate.

Possible causes:

* Insufficient resources on destination hosts
* Network issue
* VM configuration constraint
* Host compatibility issue
* Ongoing cluster operation
* Storage latency
* Management plane problem

Support manager approach:

* Determine if maintenance is urgent
* Identify affected VMs
* Check whether customer can postpone
* Engage AHV SME
* Validate cluster capacity and alerts
* Provide safe operational path
* Avoid risky force actions without understanding impact

Interview line:

> “For maintenance-related migration failures, I would treat safety and reversibility as priorities. We need to know whether the customer can delay maintenance, whether VMs are protected, and whether the cluster has enough healthy capacity before attempting disruptive actions.”

---

### Scenario 4 — Customer migrating from VMware to AHV

A customer moving from ESXi to AHV reports unexpected behavior after migration.

Possible areas:

* VirtIO drivers
* Guest OS compatibility
* Network mapping
* Disk controller differences
* Backup tool integration
* Monitoring assumptions
* Operational process change
* Team skills gap

Manager angle:

> “This type of escalation is not only technical. It is also change management. I would ensure the customer understands differences between VMware and AHV operations, confirm migration prerequisites, and coordinate technical SMEs to distinguish platform defects from migration/configuration gaps.”

---

## 6. Triage questions I should ask

### Business impact

* Which application or service is affected?
* Is this production, staging, or development?
* How many users or customers are impacted?
* Is there revenue, compliance, or contractual impact?
* What SLA or internal target is at risk?
* Is the situation stable, improving, or degrading?

### Scope

* Is the issue affecting one VM, multiple VMs, one host, one cluster, or multiple clusters?
* Are all workloads affected or only specific workloads?
* Are affected VMs on the same AHV host?
* Are affected VMs using the same network, VLAN, storage container, or application stack?
* Did the issue start after a change?

### Timeline

* When did the issue start?
* Was there a recent upgrade, patch, migration, failover, or maintenance activity?
* Were backup, replication, or batch jobs running?
* Did alerts appear before the customer noticed impact?

### AHV / virtualization layer

* Are VMs powered on and responsive from Prism?
* Are there VM HA events?
* Are there failed live migrations?
* Are hosts healthy?
* Is any host in maintenance mode?
* Are VM resource reservations, limits, or affinity rules involved?
* Are there CPU, memory, or scheduling bottlenecks?

### Storage / AOS / CVM

* Are all CVMs healthy?
* Is storage latency elevated?
* Are disks or nodes degraded?
* Is the cluster under capacity pressure?
* Are there rebuilds, rebalancing, snapshots, or replication jobs running?
* Are there alerts related to metadata, storage containers, or data resiliency?

### Network

* Is the issue isolated to a VLAN or subnet?
* Can the VM reach its gateway?
* Are physical switch ports healthy?
* Are there recent VLAN, firewall, routing, or DNS changes?
* Is management traffic affected?
* Is CVM or storage traffic affected?

### Customer operations

* What workaround is acceptable?
* Can maintenance be postponed?
* Can non-critical workloads be paused?
* Who is the customer decision-maker?
* What update cadence do they expect?
* Is an executive bridge required?

---

## 7. Likely interview questions

1. What is a Type 1 hypervisor?
2. What is the difference between Type 1 and Type 2 hypervisors?
3. What is Nutanix AHV?
4. How is AHV different from VMware ESXi?
5. Why would an enterprise choose AHV?
6. What is the relationship between AHV, AOS, CVM, and Prism?
7. What would you check if VMs are slow on an AHV cluster?
8. How would you handle a Sev1 escalation where multiple VMs are impacted?
9. What is VM High Availability?
10. What is live migration and why does it matter?
11. How would you triage a failed live migration?
12. How would you communicate with a customer during an AHV-related outage?
13. What information would you ask from the customer before escalating to engineering?
14. How would you manage a case where the customer blames AHV but evidence is inconclusive?
15. How would you lead a team of SREs during a complex virtualization escalation?
16. What are common causes of VM performance degradation?
17. What are common networking issues in virtualized environments?
18. How would you approach a VMware-to-AHV migration issue?
19. What do you need to know as a manager versus what should be handled by a Senior SRE?
20. How do you ensure RCA quality after a virtualization incident?

---

## 8. Model answers in English

### Question: What is a Type 1 hypervisor?

**Model answer:**

> A Type 1 hypervisor is a bare-metal virtualization layer that runs directly on physical server hardware. It abstracts CPU, memory, storage, and networking resources and presents them to multiple virtual machines. In enterprise environments, Type 1 hypervisors are preferred because they provide better performance, isolation, reliability, and operational control than hosted hypervisors. Nutanix AHV is an example of a Type 1 hypervisor integrated into the Nutanix HCI platform.

---

### Question: What is Nutanix AHV?

**Model answer:**

> AHV, or Acropolis Hypervisor, is Nutanix’s native enterprise hypervisor. It provides the compute virtualization layer for VMs and is integrated with Nutanix Cloud Infrastructure, AOS, distributed storage, virtual networking, and Prism management. From an operations perspective, AHV allows teams to manage VM lifecycle, live migration, high availability, and virtual networking from the Nutanix platform rather than treating the hypervisor as a separate silo.

---

### Question: How would you explain AHV to a customer coming from VMware?

**Model answer:**

> I would explain AHV as Nutanix’s native hypervisor, similar in purpose to ESXi but more tightly integrated into the Nutanix HCI stack. The customer still gets familiar enterprise virtualization capabilities such as VM management, live migration, HA, and virtual networking, but the operating model is centered around Prism and Nutanix’s integrated compute, storage, and management architecture. I would also be careful to identify operational differences, especially around migration planning, networking, drivers, backup integration, and team processes.

---

### Question: What would you check if VMs are slow on an AHV cluster?

**Model answer:**

> I would avoid jumping directly to the hypervisor as the root cause. I would first define the scope: one VM, multiple VMs, one host, one cluster, or one application. Then I would ask the team to check guest OS metrics, VM resource usage, AHV host health, CPU and memory pressure, CVM health, storage latency, disk or node alerts, and network health. I would also ask about recent changes such as upgrades, backups, migrations, or network changes. As a manager, my role would be to structure the triage, assign the right SMEs, protect the customer’s production environment, and keep communication clear while the technical investigation progresses.

---

### Question: How would you manage a Sev1 escalation involving AHV?

**Model answer:**

> I would first establish business impact, affected services, severity, customer stakeholders, and communication cadence. Then I would organize the technical triage by layers: VM, host, AHV, CVM, storage, network, and recent changes. I would make sure we focus first on mitigation and service restoration rather than premature root cause. I would also ensure that actions are safe, documented, and reversible where possible. After stabilization, I would drive a proper RCA with timeline, contributing factors, corrective actions, and preventive measures.

---

### Question: What is the relationship between AHV and the CVM?

**Model answer:**

> AHV is the hypervisor layer running the VMs, while the CVM is a critical Nutanix platform component running on each node. The CVM participates in Nutanix distributed storage and cluster services. In a support escalation, CVM health is very important because a VM performance issue may appear to be a virtualization problem but actually be related to storage path, cluster services, or CVM health.

---

### Question: What is live migration?

**Model answer:**

> Live migration is the process of moving a running VM from one physical host to another with minimal or no perceived downtime. It is important for planned maintenance, load balancing, and operational flexibility. If live migration fails, I would check destination capacity, host health, network connectivity, VM-specific constraints, cluster alerts, and whether there are ongoing operations affecting the cluster.

---

### Question: What is VM High Availability?

**Model answer:**

> VM High Availability is the ability to restart VMs on surviving hosts when a physical host fails. It reduces downtime, but it depends on cluster health, available capacity, configuration, and the nature of the failure. In an escalation, I would validate which VMs were affected, whether HA triggered as expected, whether there was enough remaining capacity, and whether any configuration or infrastructure condition prevented recovery.

---

### Question: How would you handle a customer who insists AHV is the root cause?

**Model answer:**

> I would acknowledge the customer’s impact and concern without accepting an unvalidated root cause. I would say that because the symptoms are visible at the VM layer, AHV is definitely part of the investigation scope, but we also need to validate storage, CVM health, networking, host resources, guest OS behavior, and recent changes. This keeps the conversation fact-based and avoids tunnel vision. My goal would be to drive evidence-based triage while maintaining customer trust.

---

### Question: As a manager, how technical do you need to be with AHV?

**Model answer:**

> I need enough technical depth to understand the architecture, ask the right triage questions, challenge assumptions, prioritize risks, and communicate clearly with both engineers and customers. I do not need to replace a Senior SRE in low-level command execution, but I do need to understand AHV, AOS, CVMs, Prism, HA, live migration, storage path, and virtual networking well enough to lead escalations effectively.

---

## 9. Connection with my experience

Your background maps well to this topic if you frame it correctly.

You are not positioning yourself as:

> “I am already a Nutanix AHV deep specialist.”

You are positioning yourself as:

> “I have led 24/7 enterprise support operations, incident response, SLA management, escalations, monitoring, and cross-functional troubleshooting. I understand how virtualization incidents affect customer-facing services, and I can quickly ramp into AHV-specific architecture while leading the operational side of complex escalations.”

Your strongest bridges:

### Incident management

AHV escalations are still incidents: impact, scope, timeline, mitigation, owner assignment, communications, RCA, and prevention.

Your experience with **MTTR, SLA, KPIs, and escalation management** is directly relevant.

### Cloud operations

Cloud operations teach you to think in layers:

* Compute
* Network
* Storage
* Control plane
* Customer workload
* Monitoring
* Automation
* Change management

That maps well to Nutanix HCI triage.

### Monitoring and observability

Your Grafana/Kibana experience helps you speak about symptoms, baselines, alerts, trends, and correlation.

You can say:

> “In cloud operations, I learned not to troubleshoot from a single metric. I correlate symptoms across layers and time: workload, infrastructure, network, storage, logs, alerts, and recent changes. I would apply the same discipline to AHV/Nutanix escalations.”

### Jira / Confluence / Salesforce

This connects to case hygiene, escalation tracking, customer communication, knowledge base creation, post-incident reviews, and support process maturity.

### Team leadership

For this role, your leadership is central:

* Coordinating 24/7 teams
* Managing pressure
* Keeping customer trust
* Escalating internally
* Coaching engineers
* Translating technical details for executives
* Driving follow-up actions

Interview positioning:

> “My value is that I combine technical understanding with escalation leadership. I can go deep enough to structure the investigation, but I also keep the team aligned on customer impact, safe mitigation, communication, and RCA quality.”

---

## 10. Minimum I need to memorize

You should be able to explain these confidently:

1. **Type 1 hypervisor** means bare-metal virtualization.
2. **AHV** is Nutanix’s native Type 1 hypervisor.
3. AHV is integrated with **Nutanix Cloud Infrastructure**, **AOS**, **CVMs**, **Prism**, storage, and networking.
4. AHV provides VM lifecycle operations, live migration, HA, and virtual networking.
5. **AOS/CVMs provide Nutanix distributed storage and platform services**, so not every VM issue is purely a hypervisor issue.
6. **Prism Element** manages a cluster; **Prism Central** centralizes management across clusters.
7. VM performance issues require layered triage: guest, VM, host, CVM, storage, network, physical hardware, recent changes.
8. In escalations, your priority is impact, containment, safe recovery, communication, and RCA.
9. You are not applying as a Senior SRE IC; you are applying as a technical support leader who can manage complex escalations.
10. Never say “I would just reboot the host” or “it is probably the hypervisor.” Say “I would validate evidence layer by layer.”

A very strong 30-second answer:

> “AHV is Nutanix’s native Type 1 hypervisor, integrated into the Nutanix HCI platform. It provides enterprise virtualization capabilities such as VM lifecycle management, live migration, high availability, and virtual networking, while AOS and the CVMs provide the distributed storage and platform services. In support escalations, I would treat AHV as one layer in a larger system: VM, host, CVM, storage, network, hardware, and recent changes. My role as a support manager would be to structure that triage, coordinate the right SMEs, protect the customer’s production environment, and drive clear communication and RCA.”

Memorize that.

---

## 11. Advanced / optional level

You do **not** need to master these before the interview, but knowing the terms helps you sound credible:

* KVM internals
* QEMU device emulation
* VirtIO drivers
* AHV networking internals
* Open vSwitch / bridge-level troubleshooting
* PCI passthrough details
* AHV command-line operations with `acli`
* Deep CVM service architecture
* AHV scheduler behavior
* Advanced HA policies
* Performance counters and low-level latency analysis
* VM disk controller behavior
* VMware-to-AHV migration tooling
* Backup vendor integration specifics
* Disaster recovery orchestration
* Flow / microsegmentation
* Advanced routing, MTU, bond, LACP, VLAN troubleshooting

For a manager interview, the correct stance is:

> “I am comfortable going deep with specialists, but I would not pretend that my role is to replace the AHV SRE. My responsibility is to understand the architecture, ask the right questions, manage the escalation, and ensure the customer gets a safe and timely resolution.”

---

## 12. Final checklist

Before the interview, make sure you can answer “yes” to these:

* Can I explain what a Type 1 hypervisor is?
* Can I explain why AHV is central to Nutanix HCI?
* Can I explain AHV vs VMware ESXi at a high level?
* Can I explain the role of AOS and CVMs without overcomplicating it?
* Can I explain Prism Element vs Prism Central?
* Can I describe live migration?
* Can I describe VM HA?
* Can I triage “VMs are slow” without jumping to conclusions?
* Can I ask good escalation questions?
* Can I separate business impact from technical symptoms?
* Can I communicate during a Sev1 without overpromising?
* Can I explain what I know deeply and what I would delegate to SREs?
* Can I connect this topic to my 24/7 support, cloud operations, monitoring, SLA, MTTR, and incident management experience?

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                        |
| ------------------------ | ---------------------------------------------------------------------------------------------- |
| ACLI                     | Acropolis CLI; Nutanix command-line interface for AHV and Acropolis operations.                |
| Acropolis                | Earlier Nutanix naming associated with AHV, AOS, and virtualization services.                  |
| AHV                      | Acropolis Hypervisor; Nutanix’s native Type 1 hypervisor.                                      |
| AOS                      | Nutanix platform software providing distributed storage and HCI services.                      |
| Bare-metal hypervisor    | Hypervisor running directly on physical server hardware.                                       |
| Bridge                   | Virtual networking component used to connect VMs to networks.                                  |
| Cluster                  | Group of Nutanix nodes working together as one platform.                                       |
| Compute                  | CPU and memory resources used by workloads.                                                    |
| Containment              | Immediate actions to reduce impact during an incident.                                         |
| Control plane            | Management and coordination layer of a platform.                                               |
| CPU contention           | Too many workloads competing for processor resources.                                          |
| CVM                      | Controller VM; Nutanix VM on each node providing distributed storage and platform services.    |
| Data locality            | Keeping VM data close to where the VM runs to improve performance.                             |
| Distributed storage      | Storage architecture where data and services are spread across cluster nodes.                  |
| DNS                      | Domain Name System; resolves names to IP addresses.                                            |
| ESXi                     | VMware’s Type 1 enterprise hypervisor.                                                         |
| Escalation               | Process of raising an issue to higher technical or management attention.                       |
| Guest OS                 | Operating system running inside a VM.                                                          |
| HA                       | High Availability; capability to reduce downtime after failures.                               |
| HCI                      | Hyperconverged Infrastructure; combines compute, storage, networking, and virtualization.      |
| Host                     | Physical server running the hypervisor.                                                        |
| Hyper-V                  | Microsoft’s enterprise hypervisor.                                                             |
| Hypervisor               | Software layer that creates and runs virtual machines.                                         |
| I/O                      | Input/Output; usually storage or network read/write activity.                                  |
| KVM                      | Kernel-based Virtual Machine; Linux virtualization technology used by AHV.                     |
| LACP                     | Link Aggregation Control Protocol; combines network links for redundancy/bandwidth.            |
| Latency                  | Delay in processing, network, or storage response.                                             |
| Live migration           | Moving a running VM from one host to another with minimal disruption.                          |
| Maintenance mode         | Operational state used to evacuate or prepare a host for maintenance.                          |
| MTTR                     | Mean Time To Restore/Resolve; average time to recover from incidents.                          |
| MTU                      | Maximum Transmission Unit; largest packet size allowed on a network path.                      |
| Network uplink           | Physical network connection from host to external switch.                                      |
| Node                     | Physical Nutanix server participating in a cluster.                                            |
| Noisy neighbor           | Workload consuming excessive shared resources and affecting others.                            |
| PCI passthrough          | Directly presenting physical PCI devices to a VM, used in Nutanix CVM architecture.            |
| Prism Central            | Centralized Nutanix management plane across multiple clusters.                                 |
| Prism Element            | Nutanix management interface for a single cluster.                                             |
| QEMU                     | Open-source machine emulator/virtualizer used with KVM-based virtualization.                   |
| RCA                      | Root Cause Analysis; post-incident investigation of cause and prevention.                      |
| Replication factor       | Data protection policy defining how many copies of data are stored.                            |
| Resource contention      | Competition for CPU, memory, storage, or network capacity.                                     |
| SRE                      | Site Reliability Engineer; role focused on reliability, automation, operations, and incidents. |
| SLA                      | Service Level Agreement; contractual or internal service target.                               |
| SME                      | Subject Matter Expert; specialist in a technical area.                                         |
| Storage container        | Logical storage construct used by Nutanix for VM disks and data.                               |
| Storage latency          | Delay in reading or writing data to storage.                                                   |
| Type 1 hypervisor        | Bare-metal hypervisor running directly on server hardware.                                     |
| Type 2 hypervisor        | Hosted hypervisor running on top of a general-purpose OS.                                      |
| vCPU                     | Virtual CPU assigned to a VM.                                                                  |
| VLAN                     | Virtual LAN; logical network segmentation.                                                     |
| VM                       | Virtual Machine; software-defined server running on a hypervisor.                              |
| VM HA                    | VM High Availability; restarting VMs after host failure.                                       |
| VM lifecycle             | Operations such as create, start, stop, migrate, snapshot, resize, and delete.                 |
| vNIC                     | Virtual network interface card assigned to a VM.                                               |
| VirtIO                   | Virtualized device driver framework commonly used for efficient guest I/O.                     |
| Virtual networking       | Network abstraction connecting VMs to logical and physical networks.                           |
| Virtualization           | Abstraction of physical infrastructure into virtual resources.                                 |
| Workload                 | Application, service, or VM consuming infrastructure resources.                                |

[1]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v11_0%3Aahv-ahv-overview-c.html?utm_source=chatgpt.com "AHV 11.0 - AHV Overview - Nutanix"
[2]: https://www.nutanix.com/products/ahv?utm_source=chatgpt.com "AHV: Virtualization Solution for Enterprise | Nutanix"
[3]: https://www.nutanixbible.com/pdf/5a-book-of-ahv-architecture.pdf?utm_source=chatgpt.com "AHV - AHV Architecture - NutanixBible.com"
