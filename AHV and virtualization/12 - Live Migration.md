# AHV / Virtualization — Live Migration

## 1. Short definition

**Live Migration** is the ability to move a running virtual machine from one AHV host to another, or in some cases from one AHV cluster to another, with no planned VM shutdown and ideally no visible service interruption.

In Nutanix AHV, live migration is used for **host maintenance, cluster balancing, upgrades, planned migrations, resiliency operations, and operational continuity**.

---

## 2. Clear explanation

In a virtualized environment, a VM is not tied permanently to one physical server. The VM has:

* **Compute state**: CPU execution state and memory.
* **Storage state**: virtual disks.
* **Network state**: virtual NICs, VLANs, IP addressing, security policies.
* **Placement state**: which host is currently running the VM.

During **live migration**, AHV moves the running VM from a source host to a destination host while preserving VM execution. Conceptually, the hypervisor copies memory pages to the destination host while the VM is still running, then briefly pauses the VM to transfer the final dirty memory pages and CPU/device state, and resumes it on the destination.

In Nutanix, this is especially important because AHV is part of an HCI platform. Compute, storage, networking, and management are integrated through **AHV, AOS, CVMs, Prism Element, and Prism Central**. Live migration is not just a hypervisor feature; it is part of the operational model of the Nutanix cluster.

Nutanix documentation lists live migration as relevant for maintenance activity, AHV or firmware upgrades, load balancing, isolating specific VMs on hosts, and changes in host affinity policy. It also explains that Acropolis Dynamic Scheduling can create a migration plan to reduce resource contention by moving VMs between hosts. ([Nutanix][1])

There are two important scopes:

### Intra-cluster live migration

This is the classic case: a VM moves from one AHV host to another inside the same Nutanix cluster.

Typical use cases:

* Put a host into maintenance mode.
* Evacuate VMs before firmware or AHV upgrades.
* Rebalance CPU or memory load.
* Resolve host contention.
* Respect or adjust placement policies.

When a Nutanix AHV node enters maintenance mode, the documented high-level behavior is that HA VMs are live migrated, while pinned and RF1 VMs may be powered off depending on configuration and constraints. ([Nutanix][2])

### Cross-cluster live migration

Nutanix also supports **On-Demand Cross-Cluster Live Migration**, or **OD-CCLM**, for moving powered-on guest VMs across AHV clusters through Prism Central under supported conditions. Nutanix describes this as live migration with zero VM downtime, without requiring synchronous replication schedules or setting up Nutanix Disaster Recovery for the on-demand workflow. ([Nutanix][3])

For cross-cluster live migration, requirements become stricter. For example, the destination cluster must have enough storage capacity, must be reachable from the source cluster, and the VM must be powered on. Network compatibility and CPU compatibility also matter. Nutanix documentation notes that the destination subnet should have the same IP network prefix, and if Advanced Processor Compatibility is not enabled, the destination must support the CPU feature set used by the source. ([Nutanix][4])

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** at Nutanix, live migration matters because it sits at the intersection of:

* **Customer uptime**
* **Enterprise change management**
* **Cluster operations**
* **Hypervisor troubleshooting**
* **Upgrade readiness**
* **Capacity management**
* **Escalation handling**
* **Customer confidence**

A customer may not care whether the root cause is CPU compatibility, host affinity, networking, Prism Central state, ADS behavior, or a non-migratable VM. They care that a VM did not move, a maintenance window is blocked, or a critical production workload is at risk.

As a support manager, you do not need to be the deepest AHV engineer in the room, but you must be able to:

* Understand the operational impact.
* Ask the right triage questions.
* Separate expected behavior from product defect.
* Coordinate SREs, engineering, TAMs, account teams, and the customer.
* Communicate risk clearly.
* Drive restoration or workaround.
* Protect SLA, MTTR, and customer trust.

A strong interview position would be:

> “I see live migration as both a technical capability and an operational control. In enterprise support, it is often the mechanism that allows maintenance, upgrades, load balancing, and resiliency operations without customer downtime. My role as a support manager would be to make sure we quickly identify whether the issue is capacity, compatibility, VM configuration, networking, policy, or platform behavior, and then coordinate the right technical and customer-facing response.”

---

## 4. Key concepts

### AHV

**AHV** is Nutanix’s native hypervisor. It provides VM lifecycle management, compute scheduling, VM placement, networking integration, and operational functions such as migration.

For interviews, position AHV as analogous in function to VMware ESXi, but tightly integrated into the Nutanix Cloud Platform.

### AOS

**AOS** is the distributed software layer that provides core Nutanix HCI capabilities, especially storage, resiliency, and data services. Live migration depends on the broader health of the cluster, not just the destination hypervisor.

### CVM

Each Nutanix node runs a **Controller VM**, or **CVM**, which provides storage and cluster services. In Nutanix troubleshooting, CVM health is essential because storage access, cluster operations, and management workflows depend on it.

### Prism Element and Prism Central

**Prism Element** manages an individual cluster. **Prism Central** manages multiple clusters and is especially relevant for multi-cluster operations such as OD-CCLM.

Nutanix documents cross-cluster live migration as a Prism Central workflow. ([Nutanix][3])

### ADS — Acropolis Dynamic Scheduling

**ADS** monitors resource contention and can plan VM migrations to reduce hotspots. Nutanix documentation says ADS can create a migration plan when it detects resource contention, and it respects policy constraints such as VM-host affinity and anti-affinity rules. ([Nutanix][1])

For support, ADS is important because a customer may report “unexpected VM movement” or “VMs not balancing,” and the answer may involve policy, resource contention, or scheduling constraints.

### Maintenance mode

Putting an AHV host into maintenance mode is one of the most common operational triggers for live migration. AHV attempts to evacuate migratable VMs from the host. Nutanix documentation states that HA VMs are live migrated during maintenance mode, while VMs with constraints such as GPU passthrough, CPU passthrough, PCI passthrough, or host affinity may not be migrated automatically. ([Nutanix][5])

### Non-migratable VMs

Not every VM can live migrate. Examples include VMs with:

* GPU passthrough.
* CPU passthrough.
* PCI passthrough.
* Strict VM-host affinity.
* Certain special configurations.
* Potentially incompatible CPU feature requirements.
* Cross-cluster network mismatch.
* Insufficient destination resources.

Nutanix documentation specifically identifies live migration restrictions such as GPU passthrough, CPU passthrough, WSL2, and VM-host affinity to one host. ([Nutanix][6])

### CPU compatibility

Live migration requires the destination host or cluster to support the CPU features expected by the running VM. For cross-cluster migrations, Nutanix recommends Advanced Processor Compatibility to increase migration compatibility across CPU generations. ([Nutanix][4])

### Network compatibility

For cross-cluster live migration, the destination network must allow the VM to continue operating with the expected addressing and connectivity. Nutanix documentation states that the destination subnet should have the same IP network prefix as the source VM subnet. ([Nutanix][7])

### Capacity and placement

The destination host or cluster must have enough CPU, memory, and storage capacity. Otherwise, migration may fail or be blocked before execution. Nutanix specifically notes that the destination cluster must have enough storage capacity for migrating guest VMs. ([Nutanix][4])

---

## 5. How it appears in a real escalation

### Scenario A — Maintenance window blocked

A large enterprise customer plans firmware maintenance on an AHV host. They place the host into maintenance mode, but several VMs do not migrate. The maintenance window is at risk.

Possible causes:

* VMs have GPU, CPU, or PCI passthrough.
* VM-host affinity pins the VM to that host.
* Destination hosts lack capacity.
* Cluster resiliency is not healthy.
* HA configuration or RF1 constraints affect VM behavior.
* Prism shows incomplete or stale state.
* There is a host, CVM, or network health issue.

Support manager response:

> “First, I would separate migratable and non-migratable VMs. Then I would verify cluster health, host capacity, VM constraints, and maintenance-mode behavior. If the issue is expected due to passthrough or affinity, we need to communicate the limitation and propose a controlled shutdown or policy change. If migratable VMs are failing unexpectedly, I would escalate to AHV/SRE with logs, timestamps, affected VM UUIDs, source/destination hosts, and Prism task IDs.”

### Scenario B — Live migration fails during load balancing

A customer sees high CPU contention on one AHV host. ADS should rebalance workloads, but VMs are not moving.

Possible causes:

* ADS migration would violate affinity or anti-affinity policy.
* Destination hosts do not have enough capacity.
* The VM is non-migratable.
* Cluster has ongoing tasks, upgrades, or degraded health.
* Resource contention is transient or below threshold.
* Prism or scheduling service has stale data.

Nutanix documentation notes that ADS enforces VM-host affinity during initial placement and migration, and can reject migrations that violate anti-affinity policy. ([Nutanix][8])

### Scenario C — Cross-cluster live migration fails

A customer attempts OD-CCLM between two AHV clusters registered in Prism Central. Migration fails validation.

Possible causes:

* VM is powered off.
* Destination cluster unreachable.
* Insufficient destination storage.
* Source and destination subnets are incompatible.
* CPU features are incompatible.
* APC not enabled.
* Unsupported VM configuration.
* Conflicting DR protection policy or upgrade activity.

Nutanix guidance includes requirements around powered-on VMs, source/destination reachability, storage capacity, network prefix compatibility, and CPU feature compatibility. ([Nutanix][4])

### Scenario D — Customer impact after migration

A VM migrates successfully, but the application reports latency or connection resets.

Possible causes:

* Network path difference after migration.
* VLAN or routing issue.
* East-west traffic impact.
* Storage locality changes.
* Application sensitivity to short stun/pause.
* Monitoring false positives.
* Guest OS or application timeout thresholds.

Support manager focus:

* Was there actual downtime or only monitoring noise?
* Did TCP sessions reset?
* Did the VM lose network connectivity?
* Was storage latency affected?
* Did the event correlate exactly with migration?
* Is the application latency-sensitive?

---

## 6. Triage questions I should ask

### Impact and urgency

1. Which VMs are affected?
2. Are these production workloads?
3. Is there active downtime, degraded performance, or only a blocked maintenance task?
4. What is the business impact?
5. Is there an active maintenance window or change freeze?
6. Is rollback required?

### Scope

7. Is the issue affecting one VM, multiple VMs, one host, or the entire cluster?
8. Is this intra-cluster migration or cross-cluster migration?
9. Is the migration manual, ADS-triggered, maintenance-mode-triggered, or part of an upgrade?
10. Did this work previously?

### VM configuration

11. Does the VM use GPU passthrough, CPU passthrough, PCI passthrough, vGPU, or special hardware mapping?
12. Does the VM have VM-host affinity?
13. Is the VM protected by HA?
14. Is the VM powered on?
15. Does the VM have special networking, volume groups, or security policies?

### Cluster health

16. Is the Nutanix cluster healthy?
17. Are all CVMs up?
18. Is data resiliency healthy?
19. Are there ongoing upgrades or background tasks?
20. Are there active alerts in Prism?

### Destination readiness

21. Does the destination host or cluster have enough CPU and memory?
22. Does the destination cluster have enough storage capacity?
23. Is the destination host compatible from a CPU-feature perspective?
24. Are the required VLANs/subnets available?
25. Are Prism Element and Prism Central showing consistent state?

### Cross-cluster specifics

26. Are both clusters registered in Prism Central?
27. Are the source and destination clusters reachable?
28. Does the destination subnet match the required network prefix?
29. Is Advanced Processor Compatibility enabled where required?
30. Is there an active DR protection policy or upgrade that conflicts with migration?

### Evidence collection

31. What is the exact timestamp of the failed migration?
32. What is the Prism task ID?
33. What are the source and destination hosts?
34. What error message appears in Prism?
35. Are there relevant alerts, logs, or recent configuration changes?

---

## 7. Likely interview questions

1. What is live migration?
2. How does live migration help during maintenance?
3. What can prevent a VM from live migrating?
4. How would you triage a failed AHV live migration?
5. What is the difference between live migration and cold migration?
6. What is the relationship between AHV, AOS, CVM, and Prism in this context?
7. How does live migration compare to VMware vMotion?
8. What customer impact can live migration have?
9. What would you do if a customer’s maintenance window is blocked because VMs will not migrate?
10. How would you communicate a live migration issue to an enterprise customer?
11. What is ADS and how does it relate to migration?
12. What is cross-cluster live migration?
13. Why does CPU compatibility matter?
14. Why does network compatibility matter?
15. How would you manage an escalation where support, SRE, engineering, and the customer disagree on severity?

---

## 8. Model answers in English

### Q1. What is live migration?

**Model answer:**

> Live migration is the process of moving a running virtual machine from one hypervisor host to another without intentionally shutting down the VM. In AHV, it is used for host maintenance, upgrades, load balancing, and operational continuity. From a support perspective, I think of live migration as a key availability mechanism: it allows the platform to move workloads away from risk or contention while minimizing customer impact.

### Q2. How would you triage a failed live migration in AHV?

**Model answer:**

> I would start by clarifying the impact: whether the customer has downtime, degraded service, or a blocked maintenance window. Then I would identify the scope: one VM, one host, or multiple hosts. Technically, I would check cluster health, CVM status, data resiliency, Prism alerts, destination capacity, and whether the VM has constraints such as GPU passthrough, CPU passthrough, PCI passthrough, host affinity, or CPU compatibility issues. For cross-cluster migration, I would also validate Prism Central registration, source-destination reachability, network compatibility, storage capacity, and CPU feature compatibility. As a manager, I would make sure we collect timestamps, Prism task IDs, VM UUIDs, source and destination hosts, and then drive the right escalation path with clear customer communication.

### Q3. What can make a VM non-migratable?

**Model answer:**

> A VM may be non-migratable if it depends on host-specific resources or policies. Examples include GPU passthrough, CPU passthrough, PCI passthrough, strict VM-host affinity, or unsupported guest/VM configurations. Capacity and compatibility can also block migration: the destination host must have enough resources and compatible CPU features. In cross-cluster migration, network and storage requirements also become critical.

### Q4. How would you handle a customer whose maintenance window is blocked?

**Model answer:**

> I would first stabilize the situation operationally. I would confirm how much time remains in the maintenance window, which VMs are blocking evacuation, and whether the customer has a rollback point. Then I would classify the VMs into expected non-migratable cases versus unexpected failures. If the VMs are blocked by known constraints like passthrough or affinity, I would explain the limitation, propose options such as controlled shutdown, policy adjustment, or rescheduling, and document the risk. If the failure is unexpected, I would escalate with complete evidence and keep the customer updated with clear next steps and decision points.

### Q5. How does this relate to VMware vMotion?

**Model answer:**

> Conceptually, AHV live migration is similar to VMware vMotion: both move a running VM between hosts to support maintenance, balancing, and availability. The difference is the operational ecosystem. In Nutanix, AHV live migration is integrated with AOS, Prism, ADS, CVMs, and Nutanix HCI workflows. So in support, I would not only look at the hypervisor layer, but also cluster health, storage resiliency, Prism task state, VM policies, and Nutanix-specific constraints.

### Q6. What is ADS?

**Model answer:**

> ADS stands for Acropolis Dynamic Scheduling. It is the Nutanix mechanism that helps manage VM placement and resource contention by planning or triggering VM migrations when needed. From a support perspective, ADS matters because a customer might ask why a VM moved, why it did not move, or why a host remains imbalanced. The answer may involve capacity, contention, affinity rules, anti-affinity rules, or policy compliance.

### Q7. Why does CPU compatibility matter?

**Model answer:**

> A running VM may be using CPU features exposed by the source host. If the destination host or cluster does not support those features, the VM cannot safely resume there. That is why CPU generation and CPU feature compatibility matter. In Nutanix cross-cluster scenarios, Advanced Processor Compatibility can help by selecting a compatible CPU baseline, but it needs to be planned correctly.

### Q8. How would you communicate this to a non-technical customer executive?

**Model answer:**

> I would avoid deep hypervisor details unless needed. I would say: “The migration is currently blocked because the platform has detected that this workload cannot safely move to another host under the current configuration. We are validating whether the blocker is capacity, policy, hardware dependency, or compatibility. Our priority is to protect the workload first, then provide the safest path to complete the maintenance or restore normal operation.”

### Q9. What is your role as a manager versus an SRE in this type of escalation?

**Model answer:**

> My role is not to replace the SRE as the deepest technical investigator. My role is to make sure the investigation is structured, the right data is collected, ownership is clear, customer communication is controlled, and decisions are made quickly. I need enough technical fluency to challenge assumptions, understand risk, and explain the issue credibly to customers and internal stakeholders.

---

## 9. Connection with my experience

Your current profile maps very well to this topic if you position it correctly.

You already have experience with:

* **24/7 enterprise support**
* **Incident management**
* **SLA and MTTR ownership**
* **Escalation handling**
* **Monitoring and observability**
* **Cloud operations**
* **Customer-facing communication**
* **Team coordination**
* **Jira / Confluence / Salesforce workflows**
* **Grafana / Kibana / operational dashboards**
* **AWS / Azure / GCP concepts**
* **Linux and distributed systems**

The bridge to Nutanix is:

| Your experience       | Nutanix live migration equivalent                                           |
| --------------------- | --------------------------------------------------------------------------- |
| Incident management   | Migration failure as a production-risk incident                             |
| MTTR                  | Time to restore migration capability or complete maintenance                |
| SLA                   | Customer expectation of uptime and support responsiveness                   |
| Cloud operations      | Workload mobility, capacity, placement, infrastructure health               |
| Monitoring            | Prism alerts, task failures, resource contention, latency symptoms          |
| Escalations           | SRE / engineering / TAM / account team coordination                         |
| Change management     | Maintenance mode, upgrades, firmware operations                             |
| SaaS support          | Explaining customer impact in business terms                                |
| Kubernetes operations | Workload placement and rescheduling analogy                                 |
| AWS/Azure/GCP         | Availability zones, host maintenance, capacity pools, migration constraints |

A good positioning statement:

> “My value is not that I already know every AHV command. My value is that I understand how workload mobility, maintenance, capacity, monitoring, and customer impact connect in enterprise operations. I can ramp up quickly on AHV specifics while bringing mature escalation discipline, communication, and 24/7 support leadership.”

---

## 10. Minimum I need to memorize

You should be able to say these confidently:

1. **Live migration moves a running VM between AHV hosts without planned VM shutdown.**
2. **Main use cases:** maintenance, upgrades, load balancing, resource contention, host evacuation, operational continuity.
3. **AHV live migration is managed through the Nutanix platform, mainly Prism Element / Prism Central depending on scope.**
4. **ADS can trigger or plan migrations to resolve resource contention.**
5. **Maintenance mode uses live migration to evacuate HA VMs from a host.**
6. **Not all VMs can live migrate.**
7. **Common blockers:** GPU passthrough, CPU passthrough, PCI passthrough, VM-host affinity, insufficient capacity, CPU incompatibility, network mismatch, cluster health issues.
8. **Cross-cluster live migration has stricter requirements:** powered-on VM, destination reachability, storage capacity, network compatibility, CPU compatibility, and Prism Central involvement.
9. **Support triage must start with impact, scope, recent changes, cluster health, VM constraints, destination readiness, and evidence collection.**
10. **As a manager, your role is to coordinate technical investigation and customer communication, not to behave like a Senior SRE IC.**

---

## 11. Advanced / optional level

You do not need to master these deeply before the interview, but you should recognize them:

* AHV CLI commands such as `acli host.enter_maintenance_mode`.
* Detailed AHV log locations and log interpretation.
* Exact migration protocol internals.
* CPU flag baselining and Advanced Processor Compatibility details.
* Prism Central API workflows for OD-CCLM.
* Deep networking behavior during migration.
* Storage locality behavior after VM movement.
* Interaction between live migration and DR policies.
* Live vDisk migration across storage containers.
* Edge cases involving volume groups, guest clustering, or latency-sensitive applications.
* Differences between AHV live migration, VMware vMotion, Hyper-V Live Migration, Nutanix Move, cold migration, DR failover, and cross-cluster migration.

Your interview-safe phrasing:

> “I would not claim to know every AHV internal implementation detail yet, but I understand the operational model and the troubleshooting dimensions: workload state, host health, cluster health, capacity, compatibility, policy, and customer impact.”

---

## 12. Final checklist

Before an interview, you should be able to explain:

* What live migration is.
* Why enterprises need it.
* Why Nutanix uses it during maintenance and upgrades.
* What AHV, AOS, CVM, Prism Element, and Prism Central do at a high level.
* What ADS does.
* Why a VM might not migrate.
* How to triage a failed migration.
* How to communicate risk to a customer.
* How to distinguish expected limitation from unexpected platform failure.
* How your support management background applies.

Practice final 30-second answer:

> “Live migration in AHV is the ability to move a running VM between hosts, or in supported cases across clusters, without planned downtime. It is essential for host maintenance, upgrades, load balancing, and operational continuity. In a support escalation, I would focus first on impact and scope, then validate cluster health, CVM health, destination capacity, VM constraints such as passthrough or affinity, CPU compatibility, and network readiness. As a Worldwide Support Manager, my role would be to ensure fast triage, clear ownership, strong customer communication, and the right escalation path between support, SRE, engineering, and the account team.”

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                                |
| ------------------------ | ------------------------------------------------------------------------------------------------------ |
| ACLI                     | Acropolis CLI; command-line interface used for AHV and cluster operations.                             |
| ADS                      | Acropolis Dynamic Scheduling; Nutanix mechanism for VM placement and resource balancing.               |
| Affinity Policy          | Rule that controls where a VM should or must run.                                                      |
| AHV                      | Acropolis Hypervisor; Nutanix’s native hypervisor.                                                     |
| AOS                      | Acropolis Operating System; Nutanix distributed software layer for HCI services.                       |
| APC                      | Advanced Processor Compatibility; feature used to improve CPU compatibility for migrations.            |
| Availability Zone / AZ   | Logical or physical failure domain used for workload placement and migration.                          |
| Cold Migration           | Moving a VM while it is powered off or with downtime.                                                  |
| CPU Compatibility        | Requirement that destination hosts support the CPU features expected by the VM.                        |
| CPU Passthrough          | VM directly uses host CPU features, which can restrict migration.                                      |
| CVM                      | Controller VM; Nutanix VM running storage and cluster services on each node.                           |
| Destination Host         | The AHV host where the VM will be moved.                                                               |
| Destination Cluster      | The target cluster for cross-cluster migration.                                                        |
| Dirty Memory Pages       | Memory pages modified while migration is in progress.                                                  |
| DR                       | Disaster Recovery; processes and policies for recovery after site or cluster failure.                  |
| Enterprise Support       | Support model for business-critical customer environments.                                             |
| Escalation               | Process of raising an issue to higher technical or management levels.                                  |
| Firmware Upgrade         | Hardware-level software update that may require host maintenance.                                      |
| GPU Passthrough          | Direct GPU assignment to a VM; often restricts live migration.                                         |
| HA                       | High Availability; capability to keep or restore workload availability after failure.                  |
| HCI                      | Hyperconverged Infrastructure; integrated compute, storage, networking, and virtualization.            |
| Host Evacuation          | Moving VMs away from a host, usually before maintenance.                                               |
| Intra-cluster Migration  | Migration between hosts inside the same cluster.                                                       |
| Jira                     | Issue and workflow tracking tool often used in support and engineering.                                |
| KPI                      | Key Performance Indicator; metric used to measure operational performance.                             |
| Live Migration           | Moving a running VM without planned shutdown.                                                          |
| Maintenance Mode         | Operational state used to prepare a host for maintenance by evacuating workloads.                      |
| MTTR                     | Mean Time To Resolve / Recover; average time to restore service or resolve issues.                     |
| Network Prefix           | IP subnet portion that should match in some cross-cluster migration scenarios.                         |
| Non-migratable VM        | VM that cannot be live migrated due to configuration or constraints.                                   |
| OD-CCLM                  | On-Demand Cross-Cluster Live Migration; Nutanix feature for live migration across clusters.            |
| PCI Passthrough          | Direct PCI device assignment to a VM; can prevent live migration.                                      |
| Prism Central            | Nutanix multi-cluster management plane.                                                                |
| Prism Element            | Nutanix management interface for a single cluster.                                                     |
| Production Workload      | Business-critical application or VM used in live operations.                                           |
| RF1                      | Replication Factor 1; no redundant data copy, relevant to resiliency and maintenance behavior.         |
| SLA                      | Service Level Agreement; contractual or operational service commitment.                                |
| Source Host              | The AHV host currently running the VM before migration.                                                |
| Source Cluster           | The cluster where the VM currently runs before cross-cluster migration.                                |
| SRE                      | Site Reliability Engineer; role focused on reliability, automation, and operational excellence.        |
| Storage Capacity         | Available storage required on the destination for VM migration.                                        |
| Storage Locality         | Degree to which VM data is local to the host running the VM.                                           |
| Support Manager          | Role responsible for escalation control, team coordination, customer communication, and outcomes.      |
| Task ID                  | Identifier for a Prism operation, useful for troubleshooting.                                          |
| VM                       | Virtual Machine; software-defined server running on a hypervisor.                                      |
| VM-Host Affinity         | Policy tying a VM to one or more specific hosts.                                                       |
| VM UUID                  | Unique identifier for a VM, useful in logs and escalation evidence.                                    |
| vMotion                  | VMware live migration feature; useful comparison point for AHV live migration.                         |
| vNIC                     | Virtual Network Interface Card; VM’s virtual network adapter.                                          |
| Workload Mobility        | Ability to move applications or VMs across hosts or clusters.                                          |
| WSL2                     | Windows Subsystem for Linux 2; listed by Nutanix as a live migration restriction in some AHV contexts. |

[1]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v11_0%3Amul-vm-live-migration-cases-c.html&utm_source=chatgpt.com "AHV 11.0 - Live Migration Cases - portal.nutanix.com"
[2]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_10%3Awc-node-maintenance-mode-enter-wc-t.html&utm_source=chatgpt.com "Putting a Node into Maintenance Mode using Web Console"
[3]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v11_0%3Amul-cluster-cclm-introduction-pc-c.html?utm_source=chatgpt.com "AHV 11.0 - On-Demand Cross-Cluster Live Migration - Nutanix"
[4]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v10_3%3Amul-cluster-cclm-requirements-r.html&utm_source=chatgpt.com "On-Demand Cross-Cluster Live Migration (OD-CCLM) Requirements"
[5]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_8%3Aahv-node-maintenance-mode-put-ahv-t.html&utm_source=chatgpt.com "AHV 6.8 - Putting a Node into Maintenance Mode using CLI - Nutanix"
[6]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_8%3Amul-vm-live-migration-restrictions-c.html&utm_source=chatgpt.com "AHV 6.8 - Live Migration Restrictions - Nutanix"
[7]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_10%3Amul-cluster-cclm-requirements-r.html&utm_source=chatgpt.com "On-Demand Cross-Cluster Live Migration (OD-CCLM) Requirements"
[8]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v10_0%3Aahv-dynamic-scheduling-c.html?utm_source=chatgpt.com "AHV 10.0 - Acropolis Dynamic Scheduling in AHV - portal.nutanix.com"
