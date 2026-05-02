# AHV / Virtualization — Host Maintenance Mode

## 1. Short definition

**Host maintenance mode** in Nutanix AHV is the operational state used to safely prepare an AHV host/node for maintenance by preventing new VM placement on that host and evacuating or powering off workloads according to their mobility and availability characteristics.

In practical support terms: **maintenance mode is how you drain a Nutanix AHV host before hardware work, firmware upgrades, troubleshooting, reboot, replacement, or controlled intervention.**

Nutanix documentation states that when a host is put into maintenance mode, AOS marks the host as **unschedulable**, attempts to evacuate VMs, and leaves the host in an “entering maintenance mode” state if evacuation cannot complete without remediation. ([Nutanix][1])

---

## 2. Clear explanation

In a virtualized cluster, workloads run as VMs on physical hosts. If you need to work on one physical host, you cannot simply reboot or power it off without considering the VMs, storage resiliency, high availability, and cluster health.

In Nutanix AHV, host maintenance mode is the controlled workflow used to prepare a host for that activity.

At a high level, the process is:

1. **The AHV host is marked unschedulable**
   No new VM instances should be placed on that host.

2. **Running VMs are evacuated**
   AHV/AOS attempts to move eligible VMs to other hosts in the cluster.

3. **Special VMs may block or require shutdown**
   Some VMs cannot be live migrated, for example VMs using GPU passthrough, CPU passthrough, PCI passthrough, or host affinity policies. Nutanix documentation explicitly calls out that these VMs are not migrated automatically and may need to be shut down depending on the maintenance command options. ([Nutanix][2])

4. **Agent VMs / CVM behavior must be understood**
   In Prism-based node maintenance workflows, the system handles guest VM movement and certain infrastructure VM behavior as part of the maintenance sequence. Nutanix’s web console documentation describes high-level internal steps such as live migration of HA VMs, powering off pinned or RF1-enabled VMs, and completing maintenance mode before actual host shutdown. ([Nutanix][3])

5. **Maintenance mode does not necessarily mean the host is powered off**
   A host can be in maintenance mode and still be physically powered on. Maintenance mode prepares it for intervention; shutdown/reboot is a separate action. Nutanix documentation explicitly notes that after entering maintenance mode, the AHV host is not yet shut down. ([Nutanix][3])

For an interview, the key is to explain this as a **risk-controlled evacuation workflow**, not just as a button in Prism.

You should be able to say:

> “Putting an AHV host into maintenance mode is essentially draining the host. AOS marks it unschedulable, attempts to evacuate eligible VMs, and prevents new workloads from being scheduled there. If VMs cannot be migrated because of affinity, passthrough, RF1, or other constraints, the workflow may block or require explicit shutdown. From a support perspective, the important part is to validate cluster health, capacity, resiliency, HA, and customer impact before proceeding.”

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, host maintenance mode matters because it sits at the intersection of:

* **Customer uptime**
* **Controlled change management**
* **Hardware replacement**
* **Firmware upgrades**
* **Hypervisor maintenance**
* **Escalation handling**
* **SLA and MTTR**
* **Risk communication**
* **Cross-functional execution**

In Nutanix support, customers may open cases because:

* A node cannot enter maintenance mode.
* VMs are stuck on the host.
* Prism shows the host as entering maintenance mode indefinitely.
* A hardware replacement requires the node to be safely evacuated.
* A field engineer needs approval to proceed.
* The customer is worried about VM downtime.
* A cluster has insufficient capacity to evacuate workloads.
* A VM has passthrough, affinity, or RF1 constraints.
* Maintenance was started incorrectly and created a degraded cluster condition.

As a manager, you are not expected to be the deepest AHV engineer in the room, but you must be able to lead the escalation intelligently. You need to understand:

* What the technical blockers might be.
* What questions to ask the SREs.
* What risks to communicate to the customer.
* Whether the case is a procedural issue, capacity issue, HA issue, storage resiliency issue, or product defect.
* When to involve engineering, field support, account teams, or customer success.
* How to prevent a routine maintenance operation from becoming a production outage.

A strong managerial framing:

> “Maintenance mode is not just an administrative action. In enterprise support, it is a controlled-risk operation. My job as a support manager is to ensure that the team validates health, capacity, workload mobility, customer impact, rollback options, and escalation paths before the host is touched.”

---

## 4. Key concepts

### AHV

**AHV**, or Acropolis Hypervisor, is Nutanix’s native hypervisor. It runs VMs on Nutanix nodes and integrates with the Nutanix platform, Prism, AOS, storage services, and cluster management.

For the interview, you do not need to position yourself as a kernel-level AHV specialist. You need to demonstrate that you understand how AHV participates in VM scheduling, live migration, HA, maintenance, and troubleshooting.

---

### AOS

**AOS** is the Nutanix software layer that provides distributed storage, cluster services, and platform intelligence. In maintenance mode, AOS participates in marking hosts unschedulable and coordinating evacuation behavior. Nutanix documentation describes AOS marking the host as unschedulable and attempting VM evacuation when the host enters maintenance mode. ([Nutanix][1])

---

### Prism Element

**Prism Element** is the cluster-level management UI. Nutanix recommends using the **Enter Maintenance Mode** option in the Prism Element web console for node maintenance. Nutanix also notes that entering or exiting node maintenance mode via CLI is not equivalent to doing it through Prism Element, which is an important support distinction. ([Nutanix][4])

Interview-safe explanation:

> “For standard operational maintenance, Prism Element is generally the preferred interface because it orchestrates the node maintenance workflow more completely. CLI remains important for troubleshooting, automation, and advanced support, but I would be careful not to assume CLI and UI workflows are operationally identical.”

---

### Unschedulable host

When a host is marked **unschedulable**, new VMs should not be placed there. This is part of the drain process. However, being unschedulable does not automatically mean all existing workloads have already moved.

This distinction is important during escalation:

* **Unschedulable** means no new placement.
* **Evacuated** means workloads have moved or stopped.
* **Maintenance mode complete** means the host is ready for the next maintenance step.
* **Powered off** is a separate action.

---

### VM evacuation

**Evacuation** means moving running workloads away from the host. Ideally, this happens using live migration for eligible VMs.

However, evacuation may fail or partially complete if:

* There is not enough capacity on other hosts.
* HA constraints cannot be satisfied.
* A VM has host affinity.
* A VM uses passthrough hardware.
* A VM is pinned to the host.
* The cluster is unhealthy.
* Network or storage conditions prevent safe migration.
* The host is already degraded.
* There are RF1 or resiliency concerns.

Nutanix documentation says that if evacuation fails, the host can remain in an “entering maintenance mode” state and wait for user remediation. ([Nutanix][1])

---

### Live migration

**Live migration** moves a running VM from one host to another with minimal disruption. In a maintenance workflow, live migration is what allows the host to be drained without powering off every workload.

From a support perspective, you should not oversell it. Live migration depends on the VM and cluster being eligible for migration.

Good interview language:

> “Live migration is the ideal path, but I would never assume all VMs are migratable. I would check for passthrough devices, affinity rules, pinned VMs, capacity constraints, HA state, and cluster health before treating maintenance as non-disruptive.”

---

### Non-migratable VMs

Some VMs cannot be migrated automatically. Nutanix documentation mentions examples such as VMs with **GPU, CPU passthrough, PCI passthrough, and host affinity policies**. The CLI option `non_migratable_vm_action` can control whether these VMs block the maintenance operation or are shut down with ACPI shutdown. ([Nutanix][2])

For a manager, this is a very important concept because non-migratable workloads often become customer-impacting.

You should be able to say:

> “If a VM cannot be live migrated, the support conversation changes. We need customer approval, a downtime window, or a remediation plan. That is where communication and change control become as important as the technical procedure.”

---

### HA VMs

High Availability VMs are workloads configured to survive host-level events by being migrated or restarted on another host, depending on the scenario and configuration. During maintenance mode through the web console, Nutanix documentation describes HA VMs being live migrated. ([Nutanix][3])

Important distinction:

* Planned maintenance should aim for controlled migration.
* Unplanned host failure may require restart behavior.
* HA is not a substitute for capacity planning.

---

### RF1 / Replication Factor 1

**RF1** means data has only one copy. In a production enterprise environment, this is risky because a node outage can directly affect availability. Nutanix’s maintenance mode documentation mentions that pinned and RF1-enabled VMs may be powered off during the maintenance process. ([Nutanix][3])

Interview framing:

> “If I see RF1 in a production escalation, I immediately treat it as a risk factor. Maintenance could become disruptive, and I would want explicit customer awareness before proceeding.”

---

### Pinned VMs / host affinity

A **pinned VM** or a VM with **host affinity** is tied to a specific host or constrained to run on certain hosts. This may block evacuation.

Typical reasons include:

* Licensing tied to hardware.
* GPU locality.
* Performance tuning.
* Legacy application constraints.
* Customer misconfiguration.
* Previous workaround that became permanent.

Support risk:

> A pinned VM can make a routine node maintenance case become a customer-impacting planned downtime.

---

### CVM

The **Controller VM** is the Nutanix VM that runs critical storage and cluster services on each node. CVM handling is central to node maintenance because Nutanix is not just a hypervisor layer; the distributed storage layer is part of the platform.

Do not treat the CVM like a normal guest VM. In an interview, say:

> “I would be careful around the CVM because it is part of the Nutanix control and storage plane. Any maintenance procedure must respect Nutanix’s documented sequence rather than treating the node like a generic hypervisor host.”

---

### One node at a time

Nutanix documentation notes that, for certain AOS release levels, only one node at a time can be placed in maintenance mode per cluster. ([Nutanix][4])

Interview-safe version:

> “As a general operational principle, I would avoid concurrent node maintenance unless the documented procedure, cluster resiliency, and support guidance explicitly allow it. One node at a time is the safer support default.”

---

## 5. How it appears in a real escalation

### Scenario 1 — Host stuck in “Entering Maintenance Mode”

A customer wants to replace a failed DIMM. The field engineer asks the customer to put the AHV host into maintenance mode, but Prism shows the node stuck in **Entering Maintenance Mode**.

Possible causes:

* A VM cannot be migrated.
* A VM has host affinity.
* A VM uses GPU or PCI passthrough.
* Insufficient capacity on remaining hosts.
* HA admission constraints.
* Cluster health issue.
* Storage resiliency issue.
* Network issue preventing live migration.
* CVM or host services are unhealthy.

Support manager response:

> “First, I would ensure we stop any further disruptive action. Then I would ask the team to identify which VMs remain on the host, whether they are migratable, and whether the cluster has enough healthy capacity to absorb them. In parallel, I would align with the customer on business impact and maintenance window expectations. If non-migratable VMs are involved, we need customer approval before shutdown.”

---

### Scenario 2 — Customer expects zero downtime, but one VM is pinned

The customer schedules firmware maintenance expecting live migration. During the operation, one critical VM cannot migrate because it has host affinity.

Escalation risk:

* Customer assumed no outage.
* Maintenance window may be too short.
* Application owner may not be available.
* Support team must explain why the VM cannot move.

Manager behavior:

> “I would separate the technical blocker from the communication issue. Technically, we need to confirm the affinity or passthrough constraint and options. Operationally, we need to reset expectations with the customer, document the risk, and either reschedule with application owner approval or find a safe workaround.”

---

### Scenario 3 — Insufficient capacity during evacuation

A cluster is heavily utilized. Entering maintenance mode would require the remaining nodes to absorb the VMs, but they do not have enough CPU, memory, or HA headroom.

Possible actions:

* Delay maintenance.
* Shut down non-critical VMs with approval.
* Add capacity.
* Temporarily move workloads.
* Rebalance cluster.
* Revisit HA reservations.
* Escalate internally if hardware replacement is urgent.

Manager framing:

> “I would not push the team to force maintenance mode just to meet a hardware timeline. If the cluster cannot safely absorb the workloads, the correct support behavior is to communicate the risk, propose options, and protect customer availability.”

---

### Scenario 4 — Hardware replacement on degraded node

A node has hardware issues and must be serviced. The customer asks whether they can proceed immediately.

Triage priorities:

* Is the cluster healthy enough?
* Is data resiliency OK?
* Are there active alerts?
* How many nodes are in the cluster?
* Is this RF2/RF3?
* Are there already down nodes or degraded disks?
* Are critical VMs on the affected host?
* Can workloads be evacuated?
* Is the CVM healthy?

Support manager answer:

> “Before authorizing maintenance, I would want explicit confirmation of cluster health and resiliency. If we are already degraded, taking another node out could turn a recoverable hardware issue into an outage.”

---

### Scenario 5 — CLI used instead of Prism

An engineer uses CLI to enter maintenance mode, but the customer expected the full Prism node maintenance behavior.

Important nuance:

Nutanix documentation states that entering or exiting node maintenance from CLI is not equivalent to doing it from Prism Element. ([Nutanix][4])

Manager response:

> “I would ask which method was used, because Prism and CLI workflows may not be equivalent. That affects what was actually placed into maintenance and what still needs to be validated before shutdown or hardware work.”

---

## 6. Triage questions I should ask

### Customer impact

1. Which host/node is being placed into maintenance?
2. What is the reason: hardware replacement, firmware, upgrade, troubleshooting, planned reboot?
3. Is this a planned change or an active incident?
4. What is the customer’s expected downtime tolerance?
5. Are any business-critical workloads currently running on the host?
6. Is there an approved maintenance window?

### Cluster health

7. Is the cluster currently healthy in Prism?
8. Are there active critical alerts?
9. Are all CVMs up and healthy?
10. Are any other nodes down, degraded, or already in maintenance?
11. Is storage resiliency healthy?
12. Is the cluster RF2 or RF3?
13. Are there any RF1 workloads?

### Capacity and HA

14. Do remaining hosts have enough CPU and memory capacity?
15. Is HA enabled and healthy?
16. Are there admission-control or failover-capacity concerns?
17. Are workloads already imbalanced?
18. Is live migration working normally?

### VM mobility

19. Which VMs are still running on the target host?
20. Are any VMs pinned to the host?
21. Are there host affinity rules?
22. Are any VMs using GPU passthrough, PCI passthrough, CPU passthrough, or special hardware?
23. Are any VMs configured in a way that blocks migration?
24. Can non-critical VMs be shut down if needed?

### Procedure

25. Was maintenance mode initiated from Prism Element or CLI?
26. What exact command or UI workflow was used?
27. Is the host currently unschedulable, entering maintenance mode, or fully in maintenance mode?
28. Has VM evacuation completed?
29. Has the host actually been shut down, or only placed into maintenance?
30. Is there a rollback plan?

### Escalation management

31. Who owns customer communication?
32. Has the risk been clearly explained?
33. Do we need SRE, engineering, field services, or account team involvement?
34. Is there an SLA or severity impact?
35. What is the next customer-facing update and when?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you explain AHV host maintenance mode to a customer?
2. How would you manage an escalation where a host cannot enter maintenance mode?
3. What risks would you check before approving hardware maintenance on a Nutanix node?
4. How would you handle a customer expecting zero downtime, but a VM cannot be migrated?
5. How would you coordinate between support, SREs, field engineers, and the customer?
6. What metrics matter in this type of escalation?
7. How would you prevent a routine maintenance operation from becoming a major incident?
8. How do you communicate technical uncertainty to an enterprise customer?
9. When would you escalate to engineering?
10. How would you coach a support engineer through this case?

### Technical / SRE interview

1. What happens when an AHV host enters maintenance mode?
2. What does “unschedulable” mean?
3. What happens if VM evacuation fails?
4. Which types of VMs may not live migrate?
5. What is the difference between maintenance mode and host shutdown?
6. Why does cluster capacity matter before entering maintenance mode?
7. Why are RF1 workloads risky?
8. Why is CVM handling important?
9. What is the difference between using Prism and CLI for maintenance mode?
10. What commands or checks would you expect engineers to use?

### Escalation / panel interview

1. A customer’s node is stuck in entering maintenance mode. Walk us through your response.
2. A field engineer wants to replace hardware, but the cluster has active alerts. What do you do?
3. A critical VM cannot migrate because of GPU passthrough. How do you handle the customer?
4. The customer wants to proceed despite insufficient capacity. How do you respond?
5. A junior engineer incorrectly told the customer there would be no downtime. What do you do?
6. The customer is angry because maintenance is delayed. How do you manage the escalation?
7. How do you balance SLA pressure with operational safety?
8. How do you ensure follow-up and prevention after the incident?

---

## 8. Model answers in English

### Q1 — What happens when an AHV host enters maintenance mode?

**Model answer:**

> “When an AHV host enters maintenance mode, the platform prepares that host for maintenance by marking it unschedulable, so no new VMs should be placed there. Then the system attempts to evacuate eligible running VMs to other hosts in the cluster. If evacuation cannot complete, for example because of non-migratable VMs, insufficient capacity, affinity rules, or passthrough devices, the host may remain in an entering maintenance mode state until the issue is remediated. I would also distinguish maintenance mode from shutdown: entering maintenance mode prepares the host, but it does not necessarily power it off.”

---

### Q2 — How would you manage an escalation where a host is stuck entering maintenance mode?

**Model answer:**

> “I would first stabilize the situation and prevent further disruptive actions. Then I would ask the technical team to identify what is still running on the host and why evacuation has not completed. The main areas I would check are cluster health, remaining capacity, HA status, storage resiliency, non-migratable VMs, affinity rules, passthrough devices, and whether the workflow was initiated from Prism or CLI. In parallel, I would manage the customer communication: explain that the host is not ready for maintenance yet, clarify the risk, provide a next update time, and align on whether any VM shutdown requires customer approval.”

---

### Q3 — What are examples of VMs that may block maintenance mode?

**Model answer:**

> “VMs may block or complicate evacuation if they cannot be live migrated. Common examples include VMs with GPU passthrough, PCI passthrough, CPU passthrough, host affinity, or pinned placement policies. RF1 or specially constrained workloads can also create risk. In those cases, I would not assume non-disruptive maintenance. I would ask the engineer to confirm the blocker, then align with the customer on whether a shutdown, policy change, or reschedule is acceptable.”

---

### Q4 — Why is capacity important before host maintenance?

**Model answer:**

> “When one host is drained, the remaining hosts must absorb its workloads. If the cluster does not have enough CPU, memory, storage performance, or HA headroom, evacuation can fail or create performance degradation. From a support manager perspective, capacity validation is part of risk control. I would rather delay the maintenance and explain the risk than force the operation and potentially create a larger outage.”

---

### Q5 — What is the difference between maintenance mode and shutting down the host?

**Model answer:**

> “Maintenance mode is the preparation state. It marks the host unschedulable and attempts to evacuate workloads. Shutdown is a separate disruptive action that powers off or reboots the host. In a controlled Nutanix operation, I would want confirmation that maintenance mode completed successfully and that the relevant VMs and cluster services are safe before the host is actually shut down.”

---

### Q6 — How would you explain this to a non-technical customer stakeholder?

**Model answer:**

> “I would explain that before we physically work on the server, we need to safely drain it. That means moving workloads away from it and confirming the cluster can run safely without that node. If some workloads are tied to that server or cannot move automatically, we may need a short application outage or a revised maintenance window. The goal is to avoid turning planned maintenance into an unplanned incident.”

---

### Q7 — How would you coordinate SREs, field engineers, and the customer?

**Model answer:**

> “I would define clear ownership. The SRE or support engineer owns technical validation: cluster health, VM evacuation, alerts, and maintenance state. The field engineer owns the physical intervention once the node is safe. I would own escalation control: customer communication, risk framing, SLA alignment, next updates, and decision points. If we find a product issue or unexpected behavior, I would involve engineering with a concise problem statement, logs, timeline, and business impact.”

---

### Q8 — What would you do if the customer insists on proceeding despite warnings?

**Model answer:**

> “I would be firm but collaborative. I would explain the specific risk, not just say ‘it is not recommended.’ For example: the cluster does not currently have enough capacity to evacuate the host, or a critical VM cannot migrate without shutdown. I would document the risk, propose safer alternatives, and involve the right customer stakeholders. If they still want to proceed, I would ensure explicit approval and internal alignment before any disruptive action.”

---

### Q9 — How does this relate to incident management?

**Model answer:**

> “Host maintenance mode has a strong incident-management dimension because the technical action affects customer availability. The support team needs to manage risk, communication, timeline, decision points, and rollback. The same principles I use in SaaS operations apply here: validate health before change, monitor during execution, communicate clearly, escalate early when risk increases, and perform post-incident or post-change review if anything deviates from the expected path.”

---

### Q10 — As a manager, how deep do you need to go technically?

**Model answer:**

> “I do not need to be the deepest individual contributor on AHV internals, but I need enough technical fluency to ask the right questions, detect risk, and guide the escalation. For host maintenance mode, I need to understand unschedulable state, VM evacuation, live migration, non-migratable workloads, HA, CVM importance, cluster capacity, and customer-impact scenarios. That allows me to manage the case effectively while relying on SREs for command-level execution.”

---

## 9. Connection with my experience

Your Harmonic background maps well to this topic because host maintenance mode is operationally similar to concepts you already know from SaaS/cloud and 24/7 enterprise support.

### Similarities with your current experience

| Your experience                | Nutanix host maintenance mode equivalent                        |
| ------------------------------ | --------------------------------------------------------------- |
| Incident management            | Managing failed or blocked maintenance as a live escalation     |
| SLA / MTTR ownership           | Reducing customer impact during node maintenance                |
| 24/7 support leadership        | Coordinating engineers across regions and shifts                |
| Cloud operations               | Draining infrastructure before planned intervention             |
| Kubernetes operations          | Similar to cordoning and draining a node                        |
| Monitoring with Grafana/Kibana | Watching health, alerts, workload movement, and recovery        |
| Jira / Confluence              | Case tracking, runbooks, postmortems, procedural documentation  |
| Salesforce                     | Customer case ownership and communication history               |
| Escalation management          | Coordinating SRE, field, engineering, and customer stakeholders |
| Coaching                       | Helping engineers separate technical action from customer risk  |

The strongest analogy for you is probably Kubernetes:

* **Kubernetes cordon** → mark node unschedulable.
* **Kubernetes drain** → evict workloads.
* **AHV maintenance mode** → mark AHV host unschedulable and evacuate eligible VMs.
* **Non-evictable pods / PDB constraints** → non-migratable VMs, affinity, passthrough, RF1, or capacity constraints.

A good interview bridge:

> “My background is not originally Nutanix-specific, but the operational pattern is very familiar. In cloud and Kubernetes environments, we also drain nodes, protect capacity, validate health, and manage customer impact before planned maintenance. The Nutanix-specific layer I am studying is how AHV, AOS, Prism, CVMs, HA, and VM mobility rules affect that process.”

That answer positions you correctly: not pretending to be a senior AHV SRE, but showing fast translation from your existing operational expertise.

---

## 10. Minimum I need to memorize

Memorize these points cold:

1. **Maintenance mode drains a host before maintenance.**
2. **AOS marks the AHV host unschedulable.**
3. **Eligible VMs are evacuated or live migrated to other hosts.**
4. **If evacuation fails, the host can remain in “entering maintenance mode.”**
5. **Non-migratable VMs may include GPU/PCI/CPU passthrough or host-affinity VMs.**
6. **Pinned or RF1 workloads may require shutdown / can be risky.**
7. **Maintenance mode is not the same as shutting down the host.**
8. **Prism Element is generally the preferred operational method.**
9. **CLI and Prism workflows may not be equivalent.**
10. **Before maintenance: check cluster health, CVMs, capacity, HA, alerts, resiliency, and customer impact.**
11. **Do not take multiple nodes down casually.**
12. **As a manager, your role is risk control, escalation coordination, and communication.**

A compact verbal version:

> “AHV host maintenance mode is the controlled way to drain a Nutanix host. The host is marked unschedulable, eligible VMs are evacuated to other nodes, and any blockers such as passthrough, host affinity, pinned VMs, RF1, capacity, or HA constraints must be handled before the host can safely be serviced. From a support manager perspective, I care about cluster health, customer impact, change control, escalation ownership, and clear communication before any disruptive step.”

---

## 11. Advanced / optional level

You can leave these as advanced for now:

1. Exact `acli` and `ncli` syntax beyond recognizing common commands.
2. Deep AHV scheduler internals.
3. Detailed Stargate/Cassandra/Curator internals during maintenance.
4. Advanced RF2/RF3 rebuild mechanics.
5. Full lifecycle manager upgrade orchestration.
6. Edge cases across AHV, ESXi, and Hyper-V clusters.
7. Deep packet-level troubleshooting of live migration failures.
8. Advanced HA admission control behavior.
9. Automation workflows for maintenance mode.
10. Version-specific behavioral differences across AOS/AHV releases.

However, you should recognize these commands and terms:

* `acli host.list`
* `acli host.get`
* `acli host.list_vms`
* `acli host.enter_maintenance_mode`
* `acli host.exit_maintenance_mode`
* `host.enter_maintenance_mode_check`
* `non_migratable_vm_action`
* `wait=true`
* `acpi_shutdown`
* `block`

Nutanix command reference documentation lists operations such as entering maintenance mode, checking whether a host can enter maintenance mode, exiting maintenance mode, listing hosts, and listing VMs on a host. ([Nutanix][5])

---

## 12. Final checklist

Before saying a Nutanix AHV host is safe for maintenance, validate:

| Area               | Check                                                        |
| ------------------ | ------------------------------------------------------------ |
| Customer impact    | Maintenance window, business-critical VMs, approval          |
| Cluster health     | No unexpected critical alerts                                |
| CVMs               | Healthy and reachable                                        |
| Host state         | Correct target host identified                               |
| Scheduling         | Host marked unschedulable                                    |
| VM evacuation      | No running migratable VMs left on host                       |
| Non-migratable VMs | Passthrough, pinned, affinity, RF1 reviewed                  |
| Capacity           | Remaining nodes can absorb workloads                         |
| HA                 | Failover capacity and HA state understood                    |
| Storage resiliency | No unsafe degradation                                        |
| Procedure          | Prism vs CLI method understood                               |
| Shutdown           | Host shutdown/reboot only after maintenance mode is complete |
| Communication      | Customer updated with risks, ETA, and next step              |
| Escalation         | SRE / engineering / field ownership clear                    |
| Rollback           | Plan exists if evacuation or maintenance fails               |

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword  | Meaning                                                                         |
| ------------------------- | ------------------------------------------------------------------------------- |
| ACLI                      | Acropolis CLI; command-line tool used for AHV/AOS operations.                   |
| ACPI Shutdown             | Graceful OS-level VM shutdown mechanism.                                        |
| Affinity Rule             | Policy constraining where a VM should or must run.                              |
| AHV                       | Acropolis Hypervisor; Nutanix native hypervisor.                                |
| AOS                       | Acropolis Operating System; Nutanix distributed software platform.              |
| Block                     | Option that prevents maintenance from proceeding if non-migratable VMs exist.   |
| Capacity Headroom         | Spare resources available to absorb workloads during failure or maintenance.    |
| Cluster                   | Group of Nutanix nodes operating as one platform.                               |
| Cluster Health            | Overall status of nodes, CVMs, storage, services, and alerts.                   |
| Controller VM             | Nutanix VM running storage and cluster services on each node.                   |
| Cordon                    | Kubernetes term for marking a node unschedulable; useful analogy.               |
| CPU Passthrough           | Direct CPU feature exposure that may limit VM mobility.                         |
| CVM                       | Controller VM; critical Nutanix infrastructure VM on each node.                 |
| Drain                     | Operational process of moving workloads away from a node.                       |
| Entering Maintenance Mode | Transitional state while host evacuation is still completing or blocked.        |
| Escalation                | Support process for high-impact or complex customer issues.                     |
| Evacuation                | Moving workloads away from a host before maintenance.                           |
| Field Engineer            | Engineer performing physical or onsite hardware work.                           |
| Firmware Upgrade          | Low-level hardware/software update often requiring node maintenance.            |
| GPU Passthrough           | Direct GPU assignment to a VM, often preventing live migration.                 |
| HA                        | High Availability; capability to keep or restore workloads after host issues.   |
| Hardware Replacement      | Physical intervention requiring safe host preparation.                          |
| HCI                       | Hyperconverged Infrastructure; compute, storage, and virtualization integrated. |
| Host                      | Physical server/node running the hypervisor and VMs.                            |
| Host Affinity             | Rule tying a VM to a specific host or host group.                               |
| Host Maintenance Mode     | State used to safely prepare a host for maintenance.                            |
| Host Shutdown             | Powering off or rebooting a host; separate from maintenance mode.               |
| Hypervisor                | Software layer that runs virtual machines on physical hardware.                 |
| Jira                      | Case/project tracking tool; relevant for escalation workflows.                  |
| Kibana                    | Log search/visualization tool; analogous monitoring experience.                 |
| Live Migration            | Moving a running VM to another host with minimal disruption.                    |
| Maintenance Window        | Approved time period for planned operational changes.                           |
| MTTR                      | Mean Time To Repair/Recover; key support performance metric.                    |
| Node                      | Nutanix physical server participating in the cluster.                           |
| Non-Migratable VM         | VM that cannot be live migrated because of constraints.                         |
| Non-Migratable VM Action  | Setting controlling whether such VMs block or shut down.                        |
| NCLI                      | Nutanix CLI; command-line tool for Nutanix operations.                          |
| PCI Passthrough           | Direct PCI device assignment to a VM, often limiting migration.                 |
| Pinned VM                 | VM constrained to a specific host.                                              |
| Prism Element             | Nutanix cluster-level management interface.                                     |
| Prism Central             | Nutanix centralized management interface for multiple clusters.                 |
| RF1                       | Replication Factor 1; single data copy, higher availability risk.               |
| RF2                       | Replication Factor 2; two copies of data.                                       |
| RF3                       | Replication Factor 3; three copies of data.                                     |
| SLA                       | Service Level Agreement; contractual service commitment.                        |
| SRE                       | Site Reliability Engineer; role focused on reliability and operations.          |
| Storage Resiliency        | Ability of the cluster to tolerate disk/node failures.                          |
| Support Case              | Customer-reported issue managed through support process.                        |
| Unschedulable             | Host state preventing new VM placement.                                         |
| VM                        | Virtual Machine.                                                                |
| VM Evacuation             | Relocation or shutdown of VMs from a host.                                      |
| Workload Mobility         | Ability to move workloads between hosts safely.                                 |

[1]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v10_0%3Aahv-node-maintenance-mode-put-ahv-t.html&utm_source=chatgpt.com "AHV 10.0 - Putting a Node into Maintenance Mode using CLI"
[2]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_8%3Aahv-node-maintenance-mode-put-ahv-t.html&utm_source=chatgpt.com "AHV 6.8 - Putting a Node into Maintenance Mode using CLI - Nutanix"
[3]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v10_3%3Awc-node-maintenance-mode-enter-wc-t.html&utm_source=chatgpt.com "Putting a Node into Maintenance Mode using Web Console"
[4]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_10%3Awc-node-maintenance-ahv-wc.html&utm_source=chatgpt.com "AHV 6.10 - Node Maintenance Mode - portal.nutanix.com"
[5]: https://portal.nutanix.com/page/documents/details?targetId=Command-Ref-AOS-v5_10%3Aacl-acli-host-auto-r.html&utm_source=chatgpt.com "AOS 5.10 - host - Nutanix"
