# AHV / Virtualization — High Availability

## 1. Short definition

**High Availability (HA) in Nutanix AHV** is the capability of an AHV cluster to restart or migrate virtual machines onto surviving hosts when a host fails, so that workloads recover automatically with minimal downtime.

In interview language:

> AHV HA protects VM availability at the infrastructure layer. If an AHV host fails, the cluster detects the failure and restarts affected VMs on other available hosts, depending on available capacity, HA reservation mode, affinity rules, and the health of the cluster.

Nutanix documentation describes AHV VM HA as using **best-effort HA by default**, with **guaranteed HA** available through segment-based resource reservation. ([Nutanix][1])

---

## 2. Clear explanation

In a traditional virtualization platform, VMs run on physical hosts. If one host dies, every VM running on that host is affected. HA exists to answer one question:

> “Where can these VMs safely restart when the host they were running on is no longer available?”

In Nutanix AHV, the hypervisor is part of the Nutanix HCI stack. The cluster includes:

* **AHV hosts**, which run the VMs.
* **CVMs**, or Controller VMs, which provide distributed storage services.
* **Prism Element / Prism Central**, which provide management and visibility.
* **AOS**, which provides the distributed storage and cluster intelligence.
* **Acropolis services**, which coordinate VM placement, migration, and HA behavior.

When a host fails, AHV HA is concerned mainly with **compute availability**: CPU and memory capacity for restarting VMs elsewhere. Storage availability is handled by the Nutanix distributed storage layer, using replication and data locality concepts.

There are two practical modes to understand:

### Best-effort HA

This is the default concept in current AHV documentation. The cluster tries to restart VMs after a host failure, but it has **not reserved capacity in advance**. If the remaining hosts have enough available CPU and memory, the VMs can restart. If not, some VMs may remain powered off.

This is important in support because customers may assume “HA enabled” means “guaranteed restart.” In reality, without reservation, HA depends on available resources at the time of failure.

### Guaranteed HA / HA reservation

Guaranteed HA reserves capacity across the cluster so that VMs can restart after a host failure. Nutanix documentation says AHV uses a **segment-based reservation method** for guaranteed VM HA, and that the older host-based reservation method is deprecated or no longer supported in newer documentation. ([Nutanix][1])

In practical terms, guaranteed HA means:

> The cluster proactively keeps enough capacity free so it can tolerate a defined host failure scenario.

Nutanix documentation also states that when HA reservation is enabled, Prism shows the amount of memory reserved and how many AHV host failures the system can tolerate. ([Nutanix][2])

### Restart versus live migration

Do not confuse HA with live migration.

* **Live migration** moves a running VM from one host to another without planned downtime.
* **HA restart** powers on a VM elsewhere after an unplanned host failure.

For interviews, this distinction matters. HA is about **failure recovery**. Live migration is about **maintenance, load balancing, and proactive placement**.

### Relationship with ADS

**Acropolis Dynamic Scheduling (ADS)** monitors the cluster for compute and storage I/O contention or hotspots. If it detects an issue, it can create a migration plan and move VMs to reduce imbalance. ([Nutanix][3])

ADS is not the same as HA, but it supports operational resilience. If a cluster is already overloaded or imbalanced, HA events become riskier because there may not be enough healthy capacity to restart VMs cleanly.

### Relationship with affinity rules

Affinity policies can affect HA. Nutanix documentation warns that VMs with VM-host affinity policies can only migrate or restart on hosts allowed by the policy. If a VM is pinned to only one host, it cannot be migrated or started on another host during an HA event. ([Nutanix][4])

That is a classic escalation trap: the platform may be working as designed, but policy constraints prevent recovery.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** in the enterprise support / SRE organization, HA matters because it sits at the intersection of:

* customer availability expectations,
* infrastructure design,
* capacity planning,
* incident response,
* escalation management,
* communication under pressure.

Customers do not usually open a critical case saying, “Please explain AHV HA.” They open a case saying:

> “A host failed and some production VMs did not come back.”

Or:

> “We expected zero downtime, but our application was unavailable.”

Your job is not to be the deepest AHV kernel engineer. Your job is to lead the escalation by making the right distinctions quickly:

* Was this an AHV host failure, CVM issue, storage issue, network issue, or guest OS/application issue?
* Were the VMs protected by HA reservation or only best-effort HA?
* Did the cluster have enough remaining capacity?
* Were there affinity rules, anti-affinity rules, or licensing constraints?
* Did the VMs restart but the application did not recover?
* Was there a misunderstanding between **infrastructure HA** and **application HA**?
* Was the customer’s RTO/RPO expectation aligned with the actual architecture?

This is exactly where your background in **24/7 enterprise support, SLAs, MTTR, KPIs, incident management, monitoring, and escalations** is valuable. You can position yourself as someone who can coordinate technical experts, protect customer trust, drive root cause analysis, and improve preventive controls.

---

## 4. Key concepts

### AHV

**AHV** is Nutanix’s native hypervisor. It runs virtual machines on Nutanix nodes and integrates tightly with AOS and Prism.

Interview framing:

> AHV is the virtualization layer in Nutanix’s HCI platform. For HA, it is responsible for VM execution, restart placement, and integration with cluster services.

### AOS

**AOS** is the distributed storage and data services layer. It provides the storage foundation that allows VMs to access their disks even if a physical node fails, assuming the cluster is healthy and data protection requirements are met.

Interview framing:

> AHV HA handles VM restart at the compute layer, while AOS provides distributed storage availability underneath.

### Prism Element and Prism Central

**Prism Element** manages an individual Nutanix cluster. **Prism Central** provides centralized management across clusters.

In HA incidents, Prism is where support teams may check:

* cluster health,
* host status,
* VM status,
* alerts,
* events,
* capacity,
* HA configuration,
* task history.

### Best-effort HA

Best-effort HA attempts to recover VMs after failure but does not guarantee that resources have been reserved in advance.

Practical implication:

> If the cluster is highly utilized, some VMs may not restart after a host failure.

### Guaranteed HA

Guaranteed HA reserves capacity so VMs can restart after a host failure scenario. Current Nutanix documentation describes this as **segment-based reservation**. ([Nutanix][1])

Practical implication:

> The customer trades some usable capacity for stronger recovery guarantees.

### Segment-based reservation

Segment-based reservation divides the cluster capacity logically to ensure there is sufficient reserved space for host failure tolerance. You do not need to explain the internal algorithm deeply in a manager interview, but you should understand the operational outcome:

> The cluster reserves failover capacity across hosts instead of relying on one dedicated standby host.

### Host failure

A host failure means an AHV node is no longer available to run VMs. The failure may be caused by hardware, hypervisor crash, power issue, network isolation, or severe platform failure.

### VM restart

After a host failure, affected VMs are restarted on surviving hosts if allowed by policy and resources.

Nutanix’s own developer article summarizes AHV HA as automatically restarting VMs when a host failure is identified. ([Nutanix][5])

### Capacity

Capacity is not only storage. For HA, the critical resources are often:

* memory,
* CPU,
* host count,
* cluster utilization,
* reserved capacity,
* workload size,
* VM priority or criticality,
* policy constraints.

### Affinity and anti-affinity

Affinity rules constrain where VMs may run. Anti-affinity rules try to separate VMs across hosts.

Support relevance:

> Affinity improves compliance or licensing control, but it can reduce HA flexibility.

### ADS

**Acropolis Dynamic Scheduling** helps balance workloads by monitoring contention and hotspots and triggering VM migrations where appropriate. ([Nutanix][3])

Support relevance:

> ADS can reduce operational risk before a failure, but it is not a replacement for HA design.

### Guest/application HA

Infrastructure HA does not guarantee application HA.

A VM may restart successfully, but the application may still fail because of:

* database corruption,
* cluster quorum loss,
* application dependency failure,
* DNS or load balancer issue,
* guest OS boot problem,
* service startup order,
* expired certificates,
* external storage or network dependency.

This is a critical interview distinction.

---

## 5. How it appears in a real escalation

### Scenario

A financial services customer has a four-node Nutanix AHV cluster. One AHV host fails unexpectedly during business hours. Several VMs restart automatically on other hosts, but two production VMs remain powered off. The customer expected all workloads to recover automatically and opens a Severity 1 case.

### What may be happening

Possible causes include:

* HA was operating in best-effort mode, without guaranteed reservation.
* Remaining hosts lacked enough memory to restart all VMs.
* Some VMs had VM-host affinity rules restricting placement.
* The cluster was already running hot before the failure.
* ADS had detected imbalance, but there was insufficient headroom.
* The failed host also caused network path disruption.
* The VM restarted, but the application did not recover.
* Customer misunderstood infrastructure HA versus application-level HA.

### How you should lead the escalation

As a support manager, your role is to structure the incident:

1. **Stabilize**

   * Identify business impact.
   * Confirm which VMs are down.
   * Confirm whether production services are unavailable.
   * Check whether manual restart or temporary resource adjustment is possible.

2. **Classify**

   * Is this host failure, HA behavior, capacity exhaustion, policy constraint, storage issue, network issue, or guest/application issue?

3. **Coordinate**

   * Bring in AHV/SRE, storage, networking, and customer success if needed.
   * Assign one technical owner and one communications owner.
   * Set a cadence for updates.

4. **Communicate clearly**

   * Avoid saying “HA failed” too early.
   * Say: “We are validating whether the VMs were eligible and resourced for automatic restart under the configured HA mode.”

5. **Drive RCA**

   * Timeline of host failure.
   * HA configuration.
   * Capacity at time of failure.
   * Placement decisions.
   * Affinity constraints.
   * Alerts/events.
   * Customer expectation versus configuration.

6. **Prevent recurrence**

   * Recommend guaranteed HA if business-critical.
   * Review N+1 capacity.
   * Review affinity rules.
   * Review VM sizing.
   * Review monitoring thresholds.
   * Review application-level HA design.

### Strong escalation sentence

> “The immediate priority is restoring service, but in parallel we need to determine whether the behavior was a platform fault, a capacity limitation under best-effort HA, or a policy constraint such as affinity. That distinction is essential for both the RCA and the customer communication.”

---

## 6. Triage questions I should ask

### Impact and scope

* Which VMs are affected?
* Are these production, test, or management workloads?
* What customer-facing services are unavailable?
* What is the current business impact?
* Is there an SLA breach or regulatory exposure?
* When did the issue start?
* Was there a planned maintenance window?

### Cluster and host status

* How many nodes are in the AHV cluster?
* Which host failed or became isolated?
* Are the remaining hosts healthy?
* Are CVMs healthy?
* Are there critical Prism alerts?
* Did the host recover or is it still down?
* Was there a power, hardware, network, or hypervisor event?

### HA configuration

* Is AHV HA configured as best-effort or guaranteed?
* Is HA reservation enabled?
* How many host failures is the cluster expected to tolerate?
* Was the cluster in an OK, healing, or critical HA state?
* Was there enough reserved memory/capacity?
* Were any VMs excluded or constrained?

Nutanix documentation describes HA states such as **OK**, **healing**, and **critical** after HA reservation is enabled. In the healing state, VMs are restarted on available hosts and the system attempts to return to a protected state. ([Nutanix][6])

### Capacity

* What was CPU and memory utilization before the failure?
* Were hosts overcommitted?
* Were memory reservations or large VMs involved?
* Did the cluster have N+1 capacity?
* Were any VMs too large to fit on remaining hosts?
* Were there hotspots before the incident?

### Placement and policy

* Are there VM-host affinity rules?
* Are there anti-affinity rules?
* Are there licensing constraints?
* Are there categories or placement policies in Prism Central?
* Are affected VMs allowed to run on surviving hosts?

### VM and application status

* Did the VM fail to restart, or did it restart but the application failed?
* Is the guest OS booting?
* Are VMware Tools / Nutanix Guest Tools relevant here?
* Are application dependencies available?
* Is there a database, quorum, or load balancer dependency?
* Are DNS, firewall, or routing paths working?

### Customer communication

* What was the customer’s expected RTO?
* Was the architecture designed for that RTO?
* Was HA reservation part of the design?
* Was application-level HA implemented?
* Has the customer recently added workloads without resizing the cluster?

---

## 7. Likely interview questions

### Manager / leadership interview

1. **How would you handle a Sev1 escalation where VMs did not restart after an AHV host failure?**
2. **How do you communicate with an enterprise customer when their understanding of HA does not match the actual configuration?**
3. **How do you balance technical investigation with executive communication during a major incident?**
4. **What KPIs would you track for HA-related escalations?**
5. **How would you prevent repeat incidents after an HA failure or perceived HA failure?**
6. **How do you coordinate SREs, support engineers, engineering, and account teams during an escalation?**
7. **How would you explain the difference between platform HA and application HA to a non-technical stakeholder?**
8. **How would you coach your team to triage HA incidents more effectively?**

### Technical / SRE interview

1. **What happens when an AHV host fails?**
2. **What is the difference between best-effort HA and guaranteed HA?**
3. **What resources matter most for VM HA?**
4. **How can affinity rules affect HA?**
5. **How is AHV HA different from live migration?**
6. **What is the relationship between AHV, AOS, CVMs, and Prism during an HA event?**
7. **What would you check first if some VMs restarted and others did not?**
8. **How would you distinguish an AHV HA issue from a guest OS or application issue?**
9. **How does ADS relate to availability?**
10. **What are common causes of HA recovery failure or partial recovery?**

### Advanced panel / escalation case

1. **A customer says “Nutanix HA failed.” How do you validate or challenge that statement?**
2. **A four-node cluster loses one node and critical VMs stay powered off. Walk us through your escalation process.**
3. **How would you produce an RCA for an HA incident?**
4. **How would you handle a customer demanding compensation or blaming the platform?**
5. **How would you ensure global support handover quality during a long HA escalation?**

---

## 8. Model answers in English

### Q1. What is AHV High Availability?

**Model answer:**

> AHV High Availability is the mechanism that allows virtual machines to recover from an AHV host failure by restarting them on other healthy hosts in the cluster. The key point is that HA protects the VM at the infrastructure layer. It does not necessarily mean zero downtime, and it does not replace application-level HA. In Nutanix, I would look at the HA mode, available CPU and memory, cluster health, affinity rules, and whether the cluster had enough reserved capacity to restart the affected workloads.

---

### Q2. What is the difference between best-effort HA and guaranteed HA?

**Model answer:**

> Best-effort HA means the cluster will try to restart VMs after a host failure, but it has not reserved failover capacity in advance. If the remaining hosts have enough resources, the VMs can restart; if not, some may remain down. Guaranteed HA reserves capacity across the cluster so that VMs can restart after a host failure scenario. From a support perspective, this distinction is critical because many escalations come from a mismatch between customer expectations and the actual HA configuration.

---

### Q3. A customer says HA failed because some VMs did not restart. How would you respond?

**Model answer:**

> I would avoid confirming that HA failed before validating the facts. I would say that we need to determine whether the behavior was caused by a platform issue, insufficient failover capacity, affinity constraints, or a guest/application-level problem. Then I would check the host failure timeline, HA configuration, cluster capacity before and after the failure, VM placement policies, Prism alerts and events, and whether the affected VMs were eligible to restart on surviving hosts. In parallel, I would coordinate restoration actions and maintain a clear communication cadence with the customer.

---

### Q4. What would you check first during an AHV HA escalation?

**Model answer:**

> First, I would confirm impact: which VMs are down, which services are affected, and whether this is production. Then I would check cluster health, failed host status, Prism alerts, and whether the CVMs and storage services are healthy. After that, I would validate HA configuration, available CPU and memory, and any affinity or anti-affinity rules. If the VMs restarted but the service is still down, I would shift part of the investigation toward the guest OS and application dependencies.

---

### Q5. How would you explain infrastructure HA versus application HA?

**Model answer:**

> Infrastructure HA can restart a VM on another host after a host failure. Application HA means the application itself is designed to continue service, often using clustering, replication, load balancing, quorum, or active-active architecture. A VM restart may recover the server, but the application can still fail if it depends on a database, quorum member, network path, or external service. In customer communication, I would make that distinction very clear because it affects both RTO expectations and architecture recommendations.

---

### Q6. How can affinity rules affect AHV HA?

**Model answer:**

> Affinity rules can restrict where a VM is allowed to run. That can be necessary for licensing, compliance, or performance reasons, but it reduces the scheduler’s flexibility during an HA event. If a VM is pinned to a limited set of hosts, and those hosts are unavailable or overloaded, the VM may not be able to restart even if there is capacity elsewhere in the cluster. So in any HA escalation, I would check placement policies and affinity constraints early.

---

### Q7. How does ADS relate to HA?

**Model answer:**

> ADS, or Acropolis Dynamic Scheduling, is not the same as HA. HA reacts to failures by restarting VMs on surviving hosts. ADS proactively monitors for compute and storage I/O contention or hotspots and can migrate VMs to improve balance. Operationally, ADS helps reduce risk because a balanced cluster with enough headroom is better positioned to survive a host failure. But ADS does not replace proper HA reservation, capacity planning, or application-level resilience.

---

### Q8. How would you lead a Sev1 escalation involving AHV HA?

**Model answer:**

> I would split the work into restoration, investigation, and communication. For restoration, I would identify the affected workloads and coordinate immediate recovery actions with the technical team. For investigation, I would assign owners to validate host health, HA configuration, capacity, Prism events, and placement policies. For communication, I would establish a clear update cadence, avoid premature conclusions, and explain what we know, what we are checking, and the next decision point. After service restoration, I would drive an RCA focused on timeline, cause, contributing factors, and preventive actions.

---

### Q9. What are common reasons why VMs may not restart after a host failure?

**Model answer:**

> The common reasons are insufficient remaining CPU or memory, HA running in best-effort mode without reserved capacity, affinity rules restricting placement, oversized VMs that cannot fit on surviving hosts, cluster health issues, storage or network problems, or the VM actually restarting while the guest OS or application fails. I would treat this as a structured elimination process rather than assuming immediately that HA itself failed.

---

### Q10. Why are you credible for this role if you are not yet a Nutanix AHV expert?

**Model answer:**

> My strength is managing enterprise support operations and critical incidents. I already lead a 24/7 support team and work with SLAs, MTTR, monitoring, escalations, customer communication, and cross-functional coordination. For Nutanix, I am building the specific AHV and HCI knowledge needed to speak fluently with SREs and customers. I would not position myself as a senior AHV individual contributor on day one; I would position myself as a technical escalation manager who can understand the architecture, ask the right questions, coordinate experts, and drive customer outcomes.

---

## 9. Connection with my experience

Your current profile maps well to this topic.

### Incident management

You already understand that during a Sev1, the first objective is not academic root cause. It is service restoration, containment, ownership, and communication.

For AHV HA, that means you can say:

> “I would separate immediate restoration from technical RCA, while keeping both workstreams moving.”

### SLA, MTTR, and KPIs

HA incidents are directly tied to:

* MTTA,
* MTTR,
* SLA breach risk,
* recurrence rate,
* escalation aging,
* customer sentiment,
* post-incident action completion,
* time to technical owner assignment,
* time between customer updates.

You can connect your Harmonic experience by saying:

> “In my current role, I manage support performance through KPIs and operational discipline. In an AHV HA escalation, I would apply the same framework: fast impact assessment, clear ownership, structured triage, and measurable follow-up actions.”

### Cloud operations

Your AWS/Azure/GCP experience is relevant because cloud platforms also distinguish between:

* infrastructure availability,
* zone/host failure,
* workload placement,
* autoscaling,
* capacity reservation,
* application resilience,
* customer RTO/RPO expectations.

A strong bridge:

> “The principle is similar to cloud operations: the platform can provide infrastructure-level resilience, but the application must still be architected correctly for the desired availability target.”

### Monitoring

Your Grafana/Kibana experience helps with:

* event timelines,
* before/after resource utilization,
* alert correlation,
* capacity trends,
* incident evidence,
* executive summaries.

In Nutanix, the equivalent operational mindset applies through Prism alerts, events, capacity metrics, and logs.

### Jira / Confluence / Salesforce

Very relevant for worldwide support:

* Jira: engineering escalation, defect tracking, problem management.
* Confluence: runbooks, known issues, RCA templates, handover documentation.
* Salesforce: customer case management, escalation history, entitlement, communication record.

Interview framing:

> “I would make sure that technical investigation, customer case communication, and internal engineering escalation stay synchronized. In global support, losing context between tools or regions is a major operational risk.”

---

## 10. Minimum I need to memorize

Memorize these points:

1. **AHV HA restarts VMs on surviving hosts after host failure.**
2. **HA is not the same as zero downtime.**
3. **HA is not the same as application-level HA.**
4. **Best-effort HA tries to restart VMs but does not reserve capacity.**
5. **Guaranteed HA reserves failover capacity.**
6. **Current Nutanix AHV documentation describes guaranteed HA using segment-based reservation.**
7. **CPU and memory capacity are critical for HA restart.**
8. **Affinity rules can prevent VMs from restarting on otherwise available hosts.**
9. **ADS helps balance workloads but is not the same as HA.**
10. **In escalations, validate facts before saying “HA failed.”**
11. **Triage must include cluster health, host status, HA mode, capacity, Prism alerts, and placement rules.**
12. **Customer communication must distinguish platform behavior, configuration, and application design.**

Best short verbal answer:

> AHV HA protects VMs from host failure by restarting them on other hosts in the cluster. The main operational distinction is best-effort versus guaranteed HA: best-effort depends on available capacity at the time of failure, while guaranteed HA reserves capacity in advance. In an escalation, I would check cluster health, failed host status, HA configuration, available CPU and memory, affinity policies, and whether the issue is really infrastructure HA or application recovery.

---

## 11. Advanced / optional level

You do not need to master these deeply before the interview, but you should recognize them.

### Advanced Nutanix-specific areas

* Exact mechanics of segment-based reservation.
* HA behavior across different AHV/AOS versions.
* Detailed Prism Central policy behavior.
* Failure detection internals.
* CVM failure versus AHV host failure scenarios.
* Stretched cluster behavior.
* Metro availability or disaster recovery design.
* Nutanix Files / Objects / database workload HA considerations.
* Red Hat Pacemaker / Corosync guest clustering on AHV.
* Fencing agents and guest-level cluster integration.

### Advanced troubleshooting

* Reading low-level AHV logs.
* Interpreting Genesis, Stargate, Curator, or Zookeeper-related failures.
* Deep networking packet analysis.
* Storage replication factor edge cases.
* Complex multi-site failover.
* Performance contention during HA recovery.
* API-based VM state validation.

### How to handle advanced questions

Use this pattern:

> “I would not pretend to debug the lowest-level AHV internals from memory. My approach would be to structure the incident, collect the right evidence from Prism and logs, involve the right SRE or engineering owner, and keep the customer communication accurate. At the technical level, I would first validate HA mode, capacity, placement constraints, and cluster health before escalating into deeper platform logs.”

That is a strong manager answer.

---

## 12. Final checklist

Before an interview, you should be able to explain:

* [ ] What AHV is.
* [ ] What HA does in AHV.
* [ ] What happens when a host fails.
* [ ] Difference between HA restart and live migration.
* [ ] Difference between infrastructure HA and application HA.
* [ ] Best-effort HA versus guaranteed HA.
* [ ] Why capacity reservation matters.
* [ ] Why CPU and memory headroom matter.
* [ ] How affinity rules can block recovery.
* [ ] How ADS relates to cluster balance.
* [ ] What to check in Prism during an escalation.
* [ ] How to lead a Sev1 customer call.
* [ ] How to avoid premature conclusions.
* [ ] How to communicate with executives.
* [ ] How to produce an RCA.
* [ ] How to connect the topic to your own enterprise support experience.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword  | Meaning                                                                                             |
| ------------------------- | --------------------------------------------------------------------------------------------------- |
| ADS                       | Acropolis Dynamic Scheduling; Nutanix service that helps balance workloads and reduce hotspots.     |
| Affinity Rule             | Policy that restricts where a VM can run.                                                           |
| AHV                       | Acropolis Hypervisor; Nutanix’s native hypervisor.                                                  |
| Anti-affinity             | Policy used to keep selected VMs separated across hosts.                                            |
| AOS                       | Acropolis Operating System; Nutanix distributed storage and data services layer.                    |
| Application HA            | Resilience built into the application layer, such as clustering or active-active design.            |
| Best-effort HA            | HA mode where VMs are restarted if enough resources are available, without reserved capacity.       |
| Capacity Headroom         | Spare CPU, memory, or storage capacity available for failover or growth.                            |
| Cluster                   | Group of Nutanix nodes working together as one platform.                                            |
| Compute Contention        | CPU or memory pressure affecting VM performance or placement.                                       |
| Confluence                | Documentation platform often used for runbooks, RCA, and knowledge base articles.                   |
| Critical State            | HA or cluster condition indicating protection or resources may be insufficient.                     |
| CVM                       | Controller VM; Nutanix VM running storage and cluster services on each node.                        |
| Enterprise Support        | Support model for business-critical customers with SLA, escalation, and communication requirements. |
| Escalation                | Process of raising an issue to higher technical or management attention.                            |
| Failover                  | Movement or restart of services after a failure.                                                    |
| Guaranteed HA             | HA mode that reserves capacity for VM restart after host failure.                                   |
| Guest OS                  | Operating system running inside a virtual machine.                                                  |
| HA                        | High Availability; capability to maintain or restore service after failure.                         |
| HA Reservation            | Reserved cluster capacity used to guarantee VM restart after host failure.                          |
| HCI                       | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform.     |
| Healing State             | Transitional HA state where the cluster restarts VMs and tries to return to protected status.       |
| Host                      | Physical Nutanix node running AHV and VMs.                                                          |
| Host Failure              | Loss or isolation of a physical AHV host.                                                           |
| Hotspot                   | Resource imbalance or contention on a host or storage path.                                         |
| Infrastructure HA         | Platform-level VM recovery after host or infrastructure failure.                                    |
| Jira                      | Tool commonly used for issue tracking, engineering escalation, and problem management.              |
| KPI                       | Key Performance Indicator; metric used to manage operational performance.                           |
| Live Migration            | Moving a running VM from one host to another without planned downtime.                              |
| Memory Reservation        | Memory capacity kept available for failover or guaranteed workload operation.                       |
| MTTA                      | Mean Time to Acknowledge; time until support acknowledges an incident.                              |
| MTTR                      | Mean Time to Restore or Resolve; time to recover service or close the issue.                        |
| N+1                       | Capacity model where the cluster can tolerate one node failure.                                     |
| Node                      | Physical server participating in the Nutanix cluster.                                               |
| Prism Central             | Centralized Nutanix management plane for multiple clusters.                                         |
| Prism Element             | Nutanix management interface for a single cluster.                                                  |
| RCA                       | Root Cause Analysis; post-incident explanation of cause and prevention.                             |
| RPO                       | Recovery Point Objective; acceptable data loss window.                                              |
| RTO                       | Recovery Time Objective; acceptable recovery time.                                                  |
| Salesforce                | CRM/case management platform often used for customer support cases.                                 |
| Segment-based Reservation | AHV guaranteed HA reservation method that reserves capacity across cluster segments.                |
| Sev1                      | Severity 1; critical incident with major business impact.                                           |
| SLA                       | Service Level Agreement; contractual or operational service commitment.                             |
| SRE                       | Site Reliability Engineering; discipline focused on reliability, automation, and operations.        |
| Storage I/O Contention    | Storage performance pressure affecting workload behavior.                                           |
| VM                        | Virtual Machine; software-defined server running on a hypervisor.                                   |
| VM Restart                | Powering a VM back on, usually on another host, after failure.                                      |
| VM-Host Affinity          | Rule that limits a VM to a specific host or host group.                                             |
| Workload                  | Application, service, or VM consuming infrastructure resources.                                     |

[1]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v10_3%3Awc-high-availability-acropolis-c.html&utm_source=chatgpt.com "AHV 10.3 - VM High Availability in Acropolis - portal.nutanix.com"
[2]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2156-NC2-on-Azure%3Atn-nutanix-ahv-virtual-machine-high-availability.html&utm_source=chatgpt.com "Nutanix AHV Virtual Machine High Availability"
[3]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_10%3Aahv-dynamic-scheduling-c.html&utm_source=chatgpt.com "AHV 6.10 - Acropolis Dynamic Scheduling in AHV - portal.nutanix.com"
[4]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v10_0%3Aahv-affinity-policies.html&utm_source=chatgpt.com "AHV 10.0 - VM-Host Affinity Policies - portal.nutanix.com"
[5]: https://www.nutanix.dev/2025/07/21/ahv-internals-red-hat-ha-clusters-with-nutanix-ahv/?utm_source=chatgpt.com "AHV Internals: Red Hat HA clusters with Nutanix AHV"
[6]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v10_3%3Aahv-acropolis-cluster-setting-ha-t.html&utm_source=chatgpt.com "Enabling High Availability Reservations for the Cluster"
