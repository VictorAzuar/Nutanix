# AHV / Virtualization — Affinity / Anti-Affinity

## 1. Short definition

**Affinity and anti-affinity rules control where virtual machines are allowed or preferred to run inside a virtualization cluster.**

In Nutanix AHV:

**Affinity** usually means:

> “This VM should run only on this host or group of hosts.”

**Anti-affinity** usually means:

> “These VMs should run on different hosts so that one host failure does not affect all of them.”

In Nutanix AHV, the key practical distinction is that **VM-host affinity is mandatory / hard-enforced**, while **VM-VM anti-affinity is soft-enforced / best-effort** in many cases. Nutanix documentation states that VM-host affinity limits Acropolis HA and ADS because a VM cannot be powered on or migrated to a host outside the affinity policy, while VM-VM anti-affinity is enforced on a commercially reasonable or best-effort basis. ([Nutanix][1])

---

## 2. Clear explanation

In an AHV cluster, VMs are placed on physical hosts. Normally, the platform scheduler decides where to run them based on available CPU, memory, capacity, HA constraints, and balancing logic.

Affinity and anti-affinity add **placement intent**.

### VM-host affinity

This means a VM is tied to a specific host or set of hosts.

Example:

> “Run this licensing server only on Host A and Host B.”

This can be useful when a workload has licensing, hardware, locality, or operational constraints. However, it reduces scheduler flexibility. If the allowed hosts are unavailable or overloaded, the VM may not be able to start or migrate.

Nutanix explicitly warns that VM-host affinity can limit Acropolis HA and Acropolis Dynamic Scheduling because the VM cannot be powered on or migrated to a host that does not match the policy. ([Nutanix][1])

### VM-VM anti-affinity

This means several VMs should be kept apart on different hosts.

Example:

> “Run the three Active Directory domain controllers on different AHV hosts.”

This reduces the blast radius of a host failure. If Host A fails, you do not want all replicas of the same service to disappear together.

In modern AHV / Prism Central versions, Nutanix supports **category-based VM-VM anti-affinity policies**, where VMs are grouped through Prism Central categories and the platform attempts to place them on different hosts. Nutanix describes this as useful for managing anti-affinity at scale across many VMs. ([Nutanix][2])

### Hard vs soft rules

This is the interview-critical part.

| Policy type             | Practical behavior                                                                                                                      |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| **VM-host affinity**    | Hard constraint. The VM can only run on allowed hosts.                                                                                  |
| **VM-VM anti-affinity** | Soft / best-effort constraint. AHV attempts to separate VMs, but operations may still proceed even if the rule is temporarily violated. |

Nutanix documentation says VM-VM anti-affinity in Prism Central is **soft enforced** and does not block operations such as VM creation, VM updates, upgrades, host maintenance mode, or manual live migration, even if those operations create a policy violation. ([Nutanix][2])

For a support manager, this means you must not oversimplify the feature as “guaranteed separation.” The better wording is:

> “Anti-affinity expresses placement intent and reduces correlated failure risk, but depending on cluster state and operation type, it may be best-effort rather than an absolute guarantee.”

---

## 3. Why it matters for Nutanix / Worldwide Support

Affinity and anti-affinity matter because they sit at the intersection of:

* **High availability**
* **Incident prevention**
* **Capacity management**
* **Customer expectations**
* **Escalation handling**
* **Operational risk**
* **Scheduler behavior**
* **Platform design trade-offs**

For a **Manager, Worldwide Support** at Nutanix, this topic matters because enterprise customers often discover placement policies during high-pressure moments:

> “Why did all my critical VMs run on the same host?”
> “Why did HA not restart this VM?”
> “Why did maintenance mode move the VM to a host I did not expect?”
> “Why is ADS not balancing the workload?”
> “Why is my VM stuck or unable to power on?”

The support manager does not need to debug every AHV scheduler decision as a Senior SRE would, but must be able to:

1. Understand the risk.
2. Ask the right triage questions.
3. Separate platform behavior from customer configuration.
4. Coordinate SRE / engineering escalation.
5. Communicate clearly to the customer.
6. Avoid promising impossible guarantees.
7. Translate technical root cause into operational impact.

Nutanix HA also has to respect affinity and anti-affinity rules, which means these policies directly affect recovery behavior during host failure scenarios. ([Nutanix][3])

---

## 4. Key concepts

### 4.1 VM placement

VM placement is the decision of **which AHV host runs a VM**.

Placement may happen during:

* VM power-on
* VM migration
* Live migration
* Host maintenance
* HA restart after host failure
* Cluster balancing
* Resource contention recovery

### 4.2 Acropolis Dynamic Scheduling — ADS

**ADS** is Nutanix AHV’s dynamic scheduling mechanism. It helps place and move VMs based on cluster conditions and policies.

For anti-affinity, ADS attempts to maintain compliance, but Nutanix documentation describes this as best-effort / commercially reasonable enforcement. ([Nutanix][4])

Interview phrasing:

> “ADS is responsible for helping maintain optimal VM placement, but placement policies must be understood in context. Some rules are hard constraints, while others are best-effort and can be temporarily violated depending on operational needs.”

### 4.3 Hard enforcement

A hard rule blocks actions that violate it.

Example:

> A VM has host affinity to Host A only. If Host A is down, the VM may not restart on Host B.

This is why VM-host affinity can be dangerous if overused.

### 4.4 Soft enforcement

A soft rule expresses intent, but does not necessarily block operations.

Example:

> Three VMs are configured with anti-affinity, but during maintenance or constrained capacity, two may temporarily run on the same host.

This does not automatically mean AHV is broken. It may be expected behavior depending on cluster state, capacity, policy type, and the operation performed.

### 4.5 VM-host affinity

A policy that restricts a VM to run on selected hosts.

Use cases:

* Licensing tied to physical hosts
* Dedicated hardware
* Compliance / isolation
* Performance locality
* Operational control

Risks:

* Reduced HA flexibility
* VM cannot restart if allowed host is unavailable
* Maintenance becomes more complex
* Resource imbalance
* Unexpected power-on or migration failures

### 4.6 VM-VM anti-affinity

A policy that attempts to separate VMs across different hosts.

Use cases:

* Domain controllers
* Database cluster nodes
* Application replicas
* Load balancers
* Kubernetes control-plane nodes
* Monitoring stack replicas
* Critical active/passive pairs

Risks:

* Misunderstanding “best effort” as “guaranteed”
* Insufficient hosts to satisfy the rule
* Manual migration causing violation
* Maintenance temporarily concentrating workloads
* Customer not monitoring compliance

### 4.7 Prism Element vs Prism Central

Historically, some affinity / anti-affinity configurations were done through **Prism Element** or **aCLI**. More recent AHV / Prism Central versions support **category-based policies** in Prism Central, which are better for scale and centralized management. Nutanix documentation states that category-based VM-host affinity and VM-VM anti-affinity policies in Prism Central allow managing policies for large numbers of VMs using categories. ([Nutanix][5])

### 4.8 Categories

In Prism Central, categories are metadata labels used to group resources.

Example:

| Category    | Value            |
| ----------- | ---------------- |
| AppRole     | DomainController |
| Environment | Production       |
| Tier        | Database         |

Instead of manually selecting individual VMs one by one, a customer can apply anti-affinity to VMs matching a category.

Nutanix documentation for creating VM-VM anti-affinity policies in Prism Central mentions prerequisites such as AOS 7.0 or later, PC 2024.3 or later, existing categories, and VMs associated with categories. ([Nutanix][6])

---

## 5. How it appears in a real escalation

### Escalation scenario 1 — “HA did not restart my VM”

**Customer complaint:**

> “A host failed, but one of our critical VMs did not restart on another node. This is a Sev1.”

**Possible underlying issue:**

The VM had **VM-host affinity** configured to only one host or a restricted set of hosts. During the HA event, AHV could not restart it elsewhere without violating the hard affinity rule.

Nutanix documentation notes that VMs with host affinity can only migrate to hosts specified in the affinity policy during HA events; if only one host is specified, the VM cannot be migrated. ([Nutanix][7])

**Support-manager response style:**

> “We need to verify whether HA failed because of a platform issue or whether the VM had a placement constraint that prevented restart. Let’s check the VM-host affinity configuration, the HA event timeline, available resources on eligible hosts, and whether ADS/HA had any compliant target host.”

### Escalation scenario 2 — “All replicas were on the same host”

**Customer complaint:**

> “We had three application replicas, but two or three ended up on the same AHV host. When that host failed, the application went down.”

**Possible underlying issue:**

Anti-affinity was not configured, was misconfigured, or was violated due to cluster constraints.

**Support-manager framing:**

> “Anti-affinity should be used for workloads where host-level failure domain separation matters. We need to check whether the VMs were part of the correct anti-affinity group or category, whether the cluster had enough hosts to satisfy the rule, and whether any manual migration, maintenance, or capacity constraint caused a temporary violation.”

### Escalation scenario 3 — “Maintenance mode caused unexpected placement”

**Customer complaint:**

> “During maintenance, VMs moved to unexpected hosts and violated our design.”

**Possible underlying issue:**

Maintenance operations, manual migration, or insufficient capacity may result in temporary policy violations, especially for soft anti-affinity policies.

**Support-manager framing:**

> “The key question is whether the policy was hard or soft. VM-host affinity is mandatory, but VM-VM anti-affinity is generally best-effort. We should review whether the maintenance workflow, available host capacity, and configured policies allowed compliant placement.”

### Escalation scenario 4 — “Performance degraded after VM movement”

**Customer complaint:**

> “After migration, latency increased.”

**Possible underlying issue:**

A VM was moved away from a preferred host, or anti-affinity forced placement onto a less optimal host.

**Support-manager framing:**

> “We should correlate the performance degradation with migration events, host utilization, storage latency, network path, and any placement policy. Affinity policies can solve one operational requirement while creating constraints elsewhere.”

### Escalation scenario 5 — “Policy exists but does not behave as expected”

**Customer complaint:**

> “We configured anti-affinity, but Nutanix still allowed two VMs to run on the same host.”

**Likely explanation:**

VM-VM anti-affinity is soft-enforced. The system attempts to maintain compliance but may not block all operations that create a violation. ([Nutanix][2])

**Support-manager framing:**

> “The policy is working as placement intent, but we need to confirm whether the current cluster state allows compliance. Anti-affinity is not always an absolute blocker. We should inspect policy definition, category membership, VM placement, cluster capacity, and recent operations.”

---

## 6. Triage questions I should ask

### Customer-impact questions

1. What is the current business impact?
2. Is this production, pre-production, or development?
3. How many VMs are affected?
4. Is the service down, degraded, or only at risk?
5. Is there an active host failure, maintenance operation, or migration event?
6. Is the customer asking for restoration, explanation, or preventive design review?

### Placement-policy questions

1. Is this VM using **VM-host affinity**?
2. Are the affected VMs part of a **VM-VM anti-affinity** policy?
3. Was the policy created in Prism Element, Prism Central, or through aCLI?
4. Is this a legacy policy or a category-based Prism Central policy?
5. Are the right VMs associated with the right categories?
6. Are the categories applied consistently?
7. Is the customer expecting hard enforcement or best-effort behavior?

### Cluster-state questions

1. How many AHV hosts are in the cluster?
2. How many hosts are currently healthy?
3. Is there enough CPU and memory capacity to satisfy the rule?
4. Is HA enabled and properly configured?
5. Is the cluster under resource contention?
6. Are any hosts in maintenance mode?
7. Are there recent migrations, upgrades, or failover events?

### Timeline questions

1. When was the policy created or changed?
2. When did the VM move?
3. Was the movement manual or automatic?
4. Did this happen after a host failure, upgrade, maintenance, or capacity event?
5. Was there a recent clone, restore, DR operation, or migration?

### Evidence to collect

1. VM placement history
2. Current host placement
3. Affinity / anti-affinity policy configuration
4. Prism alerts and events
5. HA event logs
6. ADS-related events
7. Host resource utilization
8. VM power-on / migration task logs
9. Recent maintenance operations
10. Cluster version: AOS, AHV, Prism Central

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you explain affinity and anti-affinity to an enterprise customer?
2. How would you handle a Sev1 where HA did not restart a VM because of affinity rules?
3. How do you balance technical correctness with customer communication during an escalation?
4. How would you coach a support engineer investigating an anti-affinity violation?
5. How would you prevent repeated escalations caused by misconfigured placement policies?
6. How would you communicate “working as designed” when the customer is angry?
7. What KPIs would you monitor around this class of incidents?
8. How would you involve SRE, engineering, account teams, and customer success?

### Technical / SRE interview

1. What is the difference between affinity and anti-affinity?
2. What is the difference between VM-host affinity and VM-VM anti-affinity?
3. Why can VM-host affinity reduce HA effectiveness?
4. What does soft enforcement mean?
5. How does anti-affinity reduce blast radius?
6. What happens if there are not enough hosts to satisfy anti-affinity?
7. How would you troubleshoot a VM that cannot power on because of placement constraints?
8. How would you verify whether a VM placement issue is caused by policy, capacity, or platform failure?
9. What is the role of ADS in AHV placement?
10. How do Prism Central categories help with policy management?

### Panel / escalation-case interview

1. A customer says Nutanix failed because all VMs ended up on one host. What do you do?
2. A customer’s VM did not restart after a host failure. How do you lead the escalation?
3. Engineering says the behavior is expected. The customer disagrees. How do you handle it?
4. A large enterprise wants strict separation for critical workloads. What questions do you ask?
5. How would you write a customer-facing RCA for an affinity-related incident?

---

## 8. Model answers in English

### Q1. What is affinity and anti-affinity in AHV?

**Model answer:**

> “In AHV, affinity and anti-affinity are VM placement policies. Affinity defines where a VM is allowed or intended to run, usually by tying it to a host or group of hosts. Anti-affinity defines separation between VMs, typically to avoid placing multiple critical replicas on the same physical host. The operationally important distinction is enforcement. VM-host affinity can be a hard constraint, which means it can limit HA and migration options. VM-VM anti-affinity is generally used to reduce correlated failure risk, but it may be best-effort depending on the implementation and cluster state.”

### Q2. Why can affinity rules create risk?

**Model answer:**

> “Affinity rules create risk because they reduce scheduler flexibility. For example, if a VM is restricted to a single host and that host fails, HA may not have a compliant target where it can restart the VM. So while affinity may solve a business or licensing requirement, it can also reduce availability if the rule is too restrictive. In support, I would always check whether the policy is intentional, whether there are enough eligible hosts, and whether the customer understands the HA trade-off.”

### Q3. How would you troubleshoot an anti-affinity violation?

**Model answer:**

> “I would first confirm the customer impact and whether this is an active outage or a compliance concern. Then I would verify the policy definition, the VM membership or category assignment, current VM placement, cluster host count, host health, available resources, recent maintenance or migration events, and the AHV/AOS/Prism Central versions. I would also clarify whether the policy is expected to be hard or soft. For VM-VM anti-affinity, I would be careful not to promise absolute enforcement because it is often best-effort and may not block all operations.”

### Q4. A customer says: “Nutanix ignored my anti-affinity rule.” How do you respond?

**Model answer:**

> “I would acknowledge the concern and avoid dismissing it. I would say: ‘Let’s validate whether the rule was configured correctly, whether the VMs were in scope, and whether the cluster had enough healthy capacity to satisfy the rule at the time. Anti-affinity is designed to reduce placement risk, but depending on the policy type and operation, it may be soft-enforced. Our goal is to determine whether this was expected behavior under the current constraints, a configuration issue, or a platform issue that requires escalation.’”

### Q5. How does this relate to HA?

**Model answer:**

> “HA is responsible for restarting VMs after a host failure, but it must operate within the constraints of the cluster and the configured placement policies. If a VM has restrictive host affinity, HA may only be allowed to restart it on specific hosts. If those hosts are unavailable or lack resources, recovery can be impacted. Anti-affinity, on the other hand, helps reduce the likelihood that one host failure takes down all replicas of a service, but it depends on correct configuration and sufficient host capacity.”

### Q6. How would you explain this to a non-technical executive?

**Model answer:**

> “I would explain it as workload placement control. Affinity means we intentionally keep a workload on certain physical servers. Anti-affinity means we intentionally spread related workloads across different physical servers to reduce single-point-of-failure risk. The trade-off is that the more constraints we add, the less freedom the platform has to recover or rebalance automatically. So the right design depends on whether the priority is strict placement, high availability, licensing, performance, or operational simplicity.”

### Q7. How would you lead the escalation as a support manager?

**Model answer:**

> “I would separate the escalation into three tracks: customer impact mitigation, technical investigation, and communication. First, restore or reduce impact if possible by validating current placement and available hosts. Second, assign engineers to verify policy configuration, cluster health, HA/ADS events, and recent operations. Third, keep the customer updated with clear language: what we know, what we are checking, what actions are safe, and what decisions require customer approval. If the behavior is expected but poorly understood, I would still treat it seriously and convert the outcome into a preventive action plan.”

### Q8. What would you include in an RCA?

**Model answer:**

> “I would include the business impact, timeline, affected VMs, configured policies, host state at the time of the event, HA or ADS behavior, whether the policy was hard or soft, and the reason the final placement occurred. I would also include corrective actions, such as adjusting affinity scope, adding eligible hosts, reviewing category assignments, improving monitoring for policy compliance, and documenting operational procedures for maintenance or failover scenarios.”

---

## 9. Connection with my experience

Your background maps well to this topic because this is not just a virtualization feature. It is an **incident-management and operational-risk topic**.

### From 24/7 enterprise support

You already understand:

* Severity classification
* Customer-impact assessment
* Escalation ownership
* SLA pressure
* Communication cadence
* War-room coordination
* Post-incident review
* Preventive actions

Affinity issues often become escalations because the customer experiences a mismatch between **expected resilience** and **actual platform behavior**. That is very similar to SaaS/cloud incidents where customers expect automatic failover but hidden constraints prevent it.

### From cloud operations

In cloud terms, anti-affinity is conceptually similar to:

* AWS Availability Zone spreading
* Kubernetes pod anti-affinity
* Kubernetes topology spread constraints
* Azure availability sets / zones
* GCP regional distribution
* Multi-node replica placement

A good bridge answer:

> “I think of anti-affinity as a failure-domain separation mechanism. In cloud and Kubernetes environments, we use similar concepts to avoid placing all replicas in the same zone, node, or rack. In AHV, the same principle applies at the virtualization host level.”

### From monitoring / KPIs

You can connect this to:

* MTTR
* Incident recurrence
* Change failure rate
* Escalation aging
* SLA breach risk
* Preventive health checks
* Configuration compliance
* Capacity risk

Example:

> “As a support manager, I would not only resolve the incident. I would also look at whether this class of issue should be detectable earlier through health checks, policy compliance monitoring, or operational runbooks.”

### From coaching teams

This is a strong management angle:

> “I would coach engineers to avoid jumping directly to ‘bug’ or ‘customer misconfiguration.’ The right approach is to reconstruct the placement decision: policy, capacity, host state, operation type, and timeline.”

That sounds like a manager who can lead technical teams without positioning himself as the deepest AHV scheduler engineer.

---

## 10. Minimum I need to memorize

Memorize these points.

1. **Affinity controls where a VM should or must run.**
2. **Anti-affinity controls which VMs should be separated.**
3. **VM-host affinity in AHV can be hard-enforced.**
4. **VM-VM anti-affinity is generally soft / best-effort.**
5. **Affinity can reduce HA flexibility.**
6. **Anti-affinity reduces blast radius by spreading replicas across hosts.**
7. **Insufficient hosts or capacity can prevent policy compliance.**
8. **Manual migration, maintenance, or upgrades can temporarily violate soft policies.**
9. **Prism Central supports category-based policies for scale in newer versions.**
10. **In an escalation, always check policy, VM membership, host health, capacity, recent operations, and timeline.**
11. **Do not promise that anti-affinity is an absolute guarantee.**
12. **Explain the trade-off: placement control vs scheduler flexibility.**

Your core interview phrase:

> “Affinity and anti-affinity are not just configuration features; they are availability design controls. In support, the key is understanding whether the policy is a hard constraint or best-effort intent, and how that affected HA, migration, or workload placement during the incident.”

---

## 11. Advanced / optional level

You do **not** need to master all of this before the interview, but it is useful to recognize.

### Advanced areas

1. aCLI configuration for legacy VM-VM anti-affinity.
2. Exact differences between Prism Element and Prism Central policy workflows.
3. Category-based automation at scale.
4. AHV scheduler internals.
5. Detailed ADS decision logic.
6. Interaction with resource overcommitment.
7. Interaction with storage locality.
8. Interaction with Nutanix Disaster Recovery.
9. Behavior differences by AOS / AHV / Prism Central version.
10. API-based policy management.
11. Edge cases involving cloned VMs, restored VMs, or migrated clusters.
12. Cross-cluster behavior in DR or NC2 environments.

### What to say if asked something too deep

Use this answer:

> “I would not pretend to know every internal scheduler decision path. At manager level, my role would be to ensure the team validates the exact policy type, version, host state, capacity, and event timeline, then engages the right AHV/SRE expertise if we need deeper scheduler analysis. But I do understand the operational principle: hard affinity can constrain HA, while anti-affinity reduces risk but may be best-effort.”

That is a strong answer because it is honest, technical, and aligned with the role.

---

## 12. Final checklist

Before your interview, you should be able to explain:

* [ ] What affinity means.
* [ ] What anti-affinity means.
* [ ] Why anti-affinity matters for critical replicas.
* [ ] Why VM-host affinity can be dangerous.
* [ ] Difference between hard and soft enforcement.
* [ ] How affinity affects HA.
* [ ] How ADS relates to placement.
* [ ] Why capacity and host count matter.
* [ ] How Prism Central categories help at scale.
* [ ] How to triage an anti-affinity violation.
* [ ] How to communicate “expected behavior” without sounding dismissive.
* [ ] How to connect the topic to incident management, SLA, MTTR, and customer trust.

A polished verbal summary:

> “In AHV, affinity and anti-affinity are workload placement policies. Affinity restricts where a VM can run, often at the VM-to-host level. Anti-affinity separates related VMs across hosts to reduce correlated failure risk. The important support distinction is enforcement: VM-host affinity can be a hard constraint and may limit HA or migration, while VM-VM anti-affinity is typically best-effort. In an escalation, I would verify the policy, VM membership, host health, available capacity, recent operations, and the customer’s expected behavior. Then I would communicate clearly whether we are dealing with misconfiguration, capacity limitation, expected behavior, or a platform issue.”

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------------------- |
| aCLI                     | Acropolis command-line interface used for AHV/Nutanix operations.                               |
| Acropolis                | Nutanix software layer that includes AHV and virtualization management capabilities.            |
| ADS                      | Acropolis Dynamic Scheduling; AHV mechanism for VM placement and balancing.                     |
| Affinity                 | Policy that ties a VM to a preferred or required host/group of hosts.                           |
| AHV                      | Acropolis Hypervisor; Nutanix’s native hypervisor.                                              |
| Anti-affinity            | Policy that attempts to keep selected VMs apart on different hosts.                             |
| AOS                      | Acropolis Operating System; core Nutanix software platform.                                     |
| Availability             | Ability of a service or VM to remain operational despite failures.                              |
| Blast radius             | Scope of impact when a component fails.                                                         |
| Category                 | Prism Central metadata label used to group resources for policy management.                     |
| Cluster                  | Group of Nutanix nodes/hosts operating as one system.                                           |
| Compliance               | Whether current VM placement satisfies the configured policy.                                   |
| Correlated failure       | Failure where multiple dependent components fail together.                                      |
| CVM                      | Controller VM; Nutanix VM running storage/control services on each node.                        |
| Enterprise support       | Support function handling business-critical customer environments.                              |
| Escalation               | Process of raising an issue to higher support, SRE, engineering, or leadership.                 |
| Failure domain           | Infrastructure boundary where one failure can affect multiple workloads.                        |
| HA                       | High Availability; ability to restart or recover workloads after host failure.                  |
| Hard enforcement         | Rule behavior that blocks actions violating the policy.                                         |
| HCI                      | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform. |
| Host                     | Physical server/node running AHV and VMs.                                                       |
| Host affinity            | Policy restricting a VM to selected physical hosts.                                             |
| Incident management      | Process for restoring service and managing communication during incidents.                      |
| Live migration           | Moving a running VM from one host to another without planned downtime.                          |
| Maintenance mode         | Operational state used to evacuate or service a host.                                           |
| MTTR                     | Mean Time To Repair/Recover; average time to restore service.                                   |
| Node                     | Physical Nutanix server participating in a cluster.                                             |
| Placement                | Decision about which host runs a VM.                                                            |
| Placement constraint     | Rule or condition limiting where a VM can run.                                                  |
| Policy violation         | State where actual VM placement does not match the intended rule.                               |
| Prism Central            | Centralized Nutanix management plane for multiple clusters and policies.                        |
| Prism Element            | Cluster-level Nutanix management interface.                                                     |
| RCA                      | Root Cause Analysis; post-incident explanation of cause and actions.                            |
| Replica                  | Redundant instance of a service or application component.                                       |
| Resource contention      | Situation where CPU, memory, storage, or network resources are constrained.                     |
| Scheduler                | System component that decides VM placement or movement.                                         |
| Sev1                     | Highest-severity incident, usually major production impact.                                     |
| SLA                      | Service Level Agreement; contractual or operational service commitment.                         |
| Soft enforcement         | Best-effort rule behavior that may not block all violating operations.                          |
| SRE                      | Site Reliability Engineering; discipline focused on reliability and operations.                 |
| Triage                   | Initial structured investigation to assess impact, cause, and next actions.                     |
| VM                       | Virtual Machine.                                                                                |
| VM-host affinity         | Policy that restricts a VM to one or more specific hosts.                                       |
| VM-VM anti-affinity      | Policy that attempts to keep selected VMs on different hosts.                                   |
| Workload placement       | Assignment of VMs or services to physical infrastructure.                                       |

[1]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_5%3Aahv-affinity-policies-c.html&utm_source=chatgpt.com "AHV 6.5 - Affinity Policies Defined in Prism Element - Nutanix"
[2]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v11_0%3Amul-anti-affinity-policies-pc-c.html?utm_source=chatgpt.com "VM-VM Anti-Affinity Policies Defined in Prism Central"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2156-NC2-on-Azure%3Atn-nutanix-ahv-virtual-machine-high-availability.html&utm_source=chatgpt.com "Nutanix AHV Virtual Machine High Availability"
[4]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_7%3Aahv-dynamic-scheduling-c.html&utm_source=chatgpt.com "AHV 6.7 - Acropolis Dynamic Scheduling in AHV - Nutanix"
[5]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v11_0%3Amul-affinity-policies-pc-c.html?utm_source=chatgpt.com "AHV 11.0 - VM-Host Affinity Policies Defined in Prism Central"
[6]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v11_0%3Amul-anti-affinity-policy-create-pc-t.html&utm_source=chatgpt.com "AHV 11.0 - Creating a VM-VM Anti-Affinity Policy - portal.nutanix.com"
[7]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_7%3Amul-vm-host-affinity-check-pe-t.html&utm_source=chatgpt.com "AHV 6.7 - Checking VM-Host Affinity From Prism Element"
