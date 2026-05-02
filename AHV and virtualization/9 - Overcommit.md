# AHV / Virtualization — Overcommit

## 1. Short definition

**Overcommit** means assigning more virtual resources to VMs than the host physically has available, based on the assumption that not all VMs will use their full allocation at the same time.

In AHV/Nutanix, this mainly appears in two forms:

**CPU overcommit / oversubscription:** assigning more vCPUs than physical CPU cores.

**Memory overcommit:** allowing VM configured memory to exceed available physical RAM, using mechanisms such as **ballooning** and, if needed, **host swap**. Nutanix documentation describes AHV memory overcommit as a feature that can allow more VMs to fit on a host than physical RAM alone would normally permit, but Nutanix positions it primarily for test/dev and not as a default production practice because of possible performance impact. ([Nutanix][1])

---

## 2. Clear explanation

In virtualization, a VM does not directly own a physical CPU core or RAM chip. The hypervisor abstracts physical resources and schedules VM workloads onto them.

For example:

A host may have:

* 32 physical CPU cores
* 512 GB RAM

But you may configure:

* 80 total vCPUs across VMs
* 600 GB of assigned VM memory

That is **overcommit**.

The key idea is that **configured capacity is not the same as consumed capacity**. Many enterprise VMs are oversized. A VM may be assigned 16 vCPUs but use only 2–4 most of the time. Another VM may have 64 GB RAM but actively use only 30 GB.

### CPU overcommit

CPU overcommit is common. The hypervisor schedules vCPUs onto physical CPU cores. This usually works well when workloads are bursty or lightly utilized.

The risk is contention. If many VMs demand CPU simultaneously, some vCPUs wait before being scheduled. In VMware language this is often discussed as **CPU Ready**; in AHV you should still think in terms of CPU scheduling delay, contention, hotspots, and VM/host utilization.

Nutanix best-practice material says AHV decouples VM vCPU cores from physical cores and recommends following industry-standard virtual-to-physical CPU oversubscription ratios, adjusted to the deployment. It also references not oversubscribing more than **4:1** when starting VMs as a general resource oversubscription recommendation. ([Nutanix][2])

### Memory overcommit

Memory overcommit is more sensitive than CPU overcommit because memory pressure can create sharp performance degradation.

AHV memory overcommit uses two key mechanisms:

**Ballooning:** a guest-side driver cooperates with the hypervisor to reclaim unused memory from inside the VM. Nutanix describes this as the preferred method because it is more performant than host swap. ([Nutanix][1])

**Host swap:** if AHV cannot reclaim enough memory through ballooning, memory may be swapped to host disk. Nutanix warns this can have substantial performance impact. ([Nutanix][1])

Nutanix also states that each AHV memory-overcommitted VM is guaranteed at least **25%** of its allocated memory as physical RAM, with up to 75% potentially reclaimed or swapped. This creates a theoretical high overcommit ceiling, but practical limits are lower because VM startup, HA, migration, and workload behavior consume real memory. ([Nutanix][1])

A simple support-manager way to explain it:

> CPU overcommit is usually about scheduling fairness and latency. Memory overcommit is about whether the platform can safely reclaim unused RAM without pushing workloads into swap. CPU contention tends to degrade gradually; memory contention can degrade abruptly.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, overcommit matters because it sits directly at the intersection of:

* customer performance escalations,
* capacity planning,
* VM sizing,
* cluster health,
* SLA impact,
* upgrade readiness,
* HA behavior,
* noisy-neighbor analysis,
* customer expectation management.

In Nutanix support, a customer may say:

> “The cluster is slow after migrating to AHV.”

But the real issue may be:

* excessive vCPU:pCPU ratio,
* oversized VMs,
* memory overcommit enabled on unsuitable workloads,
* missing VirtIO/balloon drivers,
* host swap activity,
* CVM contention,
* insufficient HA headroom,
* an imbalanced host,
* boot storm after maintenance,
* migration/upgrade pressure,
* SQL/VDI/Kubernetes nodes sized incorrectly.

As a support manager, you do not need to personally debug every scheduler detail like a kernel engineer. You do need to guide the escalation correctly:

1. separate platform defect from capacity/configuration issue;
2. protect customer impact and SLA;
3. ensure the SREs collect the right evidence;
4. communicate risk in business terms;
5. coordinate Support, Engineering, TAM/Account team, and Customer Success if needed;
6. avoid promising “just add more resources” without validating bottlenecks.

The important managerial posture is:

> “Overcommit is a valid optimization technique, but in enterprise support we must treat it as a controlled risk. During escalations, I want data: ratios, utilization, latency, swap, host balance, HA headroom, VM sizing, and recent changes.”

---

## 4. Key concepts

### vCPU vs pCPU

A **vCPU** is a virtual CPU assigned to a VM. A **pCPU** is a physical CPU core/thread available on the host.

A 4:1 vCPU:pCPU ratio means four virtual CPUs are allocated for each physical CPU unit. That does not automatically mean a problem. It means you must understand workload demand.

### Oversubscription ratio

A ratio expressing how much virtual capacity has been allocated compared to physical capacity.

Example:

* 40 physical cores
* 120 allocated vCPUs
* vCPU:pCPU ratio = 3:1

This can be acceptable for light workloads, but risky for CPU-intensive workloads.

### CPU contention

CPU contention happens when many VMs want CPU at the same time and the hypervisor cannot schedule all vCPUs immediately.

Symptoms:

* application latency,
* poor VM responsiveness,
* high host CPU,
* high scheduling delay / ready-like behavior,
* intermittent performance spikes,
* worse performance despite “low average CPU” inside the guest.

### VM sizing

Oversized VMs can hurt consolidation and performance. For example, assigning 16 vCPUs to a VM that normally uses 2 can increase scheduling complexity and reduce cluster efficiency.

Nutanix best-practice guidance for AHV CPU configuration recommends increasing the number of vCPUs first when a VM needs more than one CPU, rather than increasing cores per vCPU; for example, use 4 vCPU with one core per vCPU for a 4-CPU requirement. ([Nutanix][3])

### Memory ballooning

A cooperative mechanism where a guest driver helps the hypervisor reclaim unused memory from the VM.

In AHV, Nutanix describes ballooning as the preferred mechanism for memory overcommit because it is more performant than host swap. ([Nutanix][1])

### Host swap

A fallback mechanism where the hypervisor swaps VM memory to disk.

This is dangerous for performance-sensitive workloads because disk access is much slower than RAM, even on fast storage. Nutanix notes that the performance impact can be substantial. ([Nutanix][1])

### Guest swap

Swap configured inside the guest OS. Nutanix notes that guest-level swap can be more efficient than host-level swap because the guest OS has better context about which memory pages are safe to move. ([Nutanix][1])

### ADS — Acropolis Dynamic Scheduler

**ADS** helps place and move VMs to reduce hotspots and balance resources in AHV clusters. Nutanix explains that ADS interacts with memory overcommit by trying to place VMs where there is enough memory and by reducing performance impact when overcommit exists. ([Nutanix][1])

### HA headroom

High Availability requires reserved capacity to restart or run VMs if a host fails. Overcommit can conflict with HA expectations because resources that look “available” during normal operation may not be safely available after failure.

Nutanix explains that combining HA with memory overcommit reduces the achievable overcommit level because memory must be reserved for failover behavior. ([Nutanix][1])

### CVM impact

In Nutanix, the **Controller VM** is critical to the storage and data services path. If customer VMs consume excessive CPU/memory and starve the CVM, cluster performance can suffer. In escalations, always distinguish between:

* guest workload pressure,
* host hypervisor pressure,
* CVM resource pressure,
* storage latency,
* network problems.

---

## 5. How it appears in a real escalation

### Escalation scenario 1 — “All VMs are slow after adding more workloads”

A customer added several VMs to an AHV cluster. No hardware was added. The cluster now shows intermittent latency.

Likely investigation path:

* Check host CPU utilization.
* Check vCPU:pCPU ratio.
* Identify oversized VMs.
* Review whether CPU contention correlates with business hours.
* Check whether ADS is moving VMs or whether hosts are imbalanced.
* Check whether CVMs have enough resources.
* Check Prism alerts and performance charts.
* Ask what changed: new VMs, backup jobs, migration, antivirus scan, batch jobs.

Manager-level response:

> “We need to confirm whether this is a platform issue or a resource contention issue introduced by workload growth. Let’s gather before/after utilization, VM sizing, cluster capacity, host balance, and any alerts. If we confirm overcommit pressure, we should provide immediate mitigation and a longer-term capacity/sizing recommendation.”

### Escalation scenario 2 — “Performance dropped after enabling memory overcommit”

A customer enabled memory overcommit to fit more VMs into the cluster. Afterward, some applications became slow.

Likely causes:

* memory overcommit used in production,
* missing or outdated VirtIO balloon drivers,
* active host swap,
* memory-sensitive workloads,
* insufficient guest swap,
* no HA headroom,
* too many VMs booting/migrating at once.

Nutanix explicitly positions AHV memory overcommit as useful for test/development but not recommended for production because of possible performance reduction. ([Nutanix][1])

Manager-level response:

> “Memory overcommit may be technically supported, but we need to validate whether this workload profile is appropriate for it. If host swap is active or ballooning is reclaiming memory from active workloads, the support recommendation may be to disable memory overcommit for production-critical VMs, right-size memory, or add capacity.”

### Escalation scenario 3 — “Upgrade or maintenance mode takes too long”

During an AHV upgrade, hosts enter maintenance and VMs need to live migrate. Some migrations are slow or fail.

Relevant overcommit angle:

* VMs may need physical memory available on the target host.
* Memory-overcommitted VMs can take longer to migrate because swapped memory may need to be brought back into physical memory for migration.
* ADS may treat overcommit-enabled VMs as more expensive to migrate under load. Nutanix notes that live migration of overcommit-enabled VMs can take more time and add load during heavy system usage. ([Nutanix][1])

Manager-level response:

> “Before proceeding further, we should verify cluster headroom, active memory pressure, host swap, and whether overcommit-enabled VMs are blocking safe migration. The immediate objective is to protect workload availability; the follow-up is to review capacity policy before the next maintenance window.”

### Escalation scenario 4 — “Customer compares AHV to VMware”

The customer says:

> “In VMware we used high overcommit ratios. Why is AHV behaving differently?”

Good response:

> “The concept is similar, but implementation details, metrics, defaults, and best practices differ by hypervisor. We should not copy ratios blindly. We need to validate actual workload behavior, AHV scheduler behavior, memory reclamation, CVM requirements, HA reservations, and Nutanix best-practice guidance.”

---

## 6. Triage questions I should ask

### Business impact

1. Which applications or services are affected?
2. Is this a production workload?
3. Is the impact latency, outage, degraded throughput, or failed VM operations?
4. When did it start?
5. Is the impact constant or only during peaks?
6. Are SLAs or customer-facing services affected?

### Change context

7. Were new VMs added recently?
8. Was memory overcommit enabled recently?
9. Was there a migration from VMware/Hyper-V to AHV?
10. Did the issue start after an upgrade, expansion, backup change, or maintenance event?
11. Were VM sizes changed recently?

### CPU overcommit

12. What is the cluster-level and host-level vCPU:pCPU ratio?
13. Are a few large VMs dominating CPU demand?
14. Is host CPU high or only guest CPU high?
15. Are VMs oversized?
16. Are there CPU hotspots on specific hosts?
17. Are CVMs showing CPU pressure?

### Memory overcommit

18. Is AHV memory overcommit enabled?
19. Is it enabled on production VMs or only test/dev?
20. Are VirtIO/balloon drivers installed and current?
21. Is host swap active?
22. Is guest swap active?
23. Are memory-sensitive workloads affected, such as databases?
24. Is there enough free memory for HA/failover?

### Nutanix-specific

25. What does Prism show for cluster health and alerts?
26. Is ADS moving VMs or reporting constraints?
27. Are there HA reservations or admission-control limitations?
28. Are migrations failing or slow?
29. Are there storage latency or network symptoms that could be mistaken for CPU/memory contention?
30. Is the CVM properly sized and healthy?

### Customer communication

31. What is the immediate mitigation target: restore performance, start VMs, complete upgrade, or prevent recurrence?
32. Can noncritical VMs be powered off, migrated, or resized?
33. Is adding capacity an option?
34. Does the customer need a formal RCA?
35. Who owns the decision: customer admin, TAM, Support, Engineering, Account team?

---

## 7. Likely interview questions

1. What is overcommit in virtualization?
2. What is the difference between CPU overcommit and memory overcommit?
3. Why is CPU overcommit generally less risky than memory overcommit?
4. How would you explain vCPU:pCPU ratio to a customer?
5. What symptoms would suggest CPU contention?
6. What symptoms would suggest memory pressure?
7. How does AHV memory overcommit work?
8. What is ballooning?
9. What is host swap, and why is it risky?
10. Would you recommend memory overcommit in production?
11. How does overcommit affect HA?
12. How can overcommit affect live migration or upgrades?
13. What would you check in Prism during an overcommit-related escalation?
14. How would you handle a customer who insists that “the cluster has free CPU” but VMs are slow?
15. How would you distinguish a capacity issue from a Nutanix product defect?
16. How would you manage communication during a major performance escalation?
17. How does this relate to your experience managing 24/7 support?
18. As a manager, how deep technically do you need to go?
19. How would you coach an engineer during this type of escalation?
20. How would you write the RCA?

---

## 8. Model answers in English

### Q1. What is overcommit in virtualization?

**Model answer:**

> Overcommit means allocating more virtual resources to VMs than the physical host has available, based on the assumption that not every VM will consume its full allocation at the same time. For CPU, this usually means assigning more vCPUs than physical cores. For memory, it means assigning more VM memory than physical RAM, relying on mechanisms such as ballooning or swap. It is a common capacity optimization technique, but it must be controlled because excessive overcommit can create performance degradation and complicate HA, migration, and incident response.

### Q2. What is the difference between CPU and memory overcommit?

**Model answer:**

> CPU overcommit is mostly a scheduling problem. The hypervisor schedules vCPUs onto physical CPU cores, and if too many workloads demand CPU simultaneously, some vCPUs wait. Memory overcommit is more sensitive because if physical RAM is insufficient, the platform may need to reclaim memory through ballooning or fall back to host swap. CPU contention often causes gradual latency or throughput issues, while memory pressure can create severe performance drops, especially if swapping occurs.

### Q3. How does AHV memory overcommit work?

**Model answer:**

> In AHV, memory overcommit allows the total configured VM memory on a host to exceed the physical memory available, but it relies on mechanisms like ballooning and host swap. Ballooning is preferred because a guest driver can help reclaim unused memory from inside the VM. If there is not enough reclaimable memory, AHV may use host swap, which has a higher performance cost. For that reason, I would treat AHV memory overcommit as a controlled optimization, not something to enable blindly on production-critical workloads.

### Q4. Would you recommend memory overcommit in production?

**Model answer:**

> I would be cautious. Nutanix documentation positions memory overcommit as useful for test and development and warns about potential performance impact in production. My recommendation would depend on workload criticality, observed memory usage, guest driver readiness, HA requirements, and performance tolerance. For production workloads, especially databases or latency-sensitive applications, I would generally avoid memory overcommit unless there is a clear, validated reason and proper monitoring.

### Q5. What would you check during a performance escalation possibly caused by overcommit?

**Model answer:**

> I would first clarify impact and timeline: which applications are affected, when it started, and what changed. Technically, I would check Prism alerts, host and cluster utilization, vCPU:pCPU ratio, VM sizing, memory pressure, host swap, guest swap, CVM health, ADS behavior, HA headroom, and recent operations like migrations, backups, or upgrades. I would also compare guest-level metrics with hypervisor-level metrics because the guest may not see the full picture of scheduling or host memory pressure.

### Q6. How would you explain this to an enterprise customer?

**Model answer:**

> I would avoid blaming the customer for overprovisioning. I would explain that virtualization allows flexible allocation, but performance depends on actual concurrent demand. If many VMs request CPU or memory at the same time, the cluster may experience contention. Our goal is to validate whether the issue is caused by resource contention, configuration, workload growth, or a platform defect. Then we can propose immediate mitigations and a longer-term sizing or capacity plan.

### Q7. How does overcommit affect HA?

**Model answer:**

> HA requires enough spare capacity to tolerate a host failure. If a cluster is heavily overcommitted, there may not be enough real CPU or memory headroom to restart or run VMs safely after a failure. This is especially important for memory because VMs may need physical RAM during boot or failover. So, when reviewing overcommit, I would always check HA reservations and failure-domain assumptions, not just normal operating utilization.

### Q8. How would you manage this escalation as a support manager?

**Model answer:**

> I would split the work into impact management, technical diagnosis, and communication. First, confirm business impact and stabilize the customer: reduce load, move or power off noncritical VMs, pause risky operations, or increase capacity if available. Second, assign engineers to collect data: Prism health, host metrics, VM sizing, swap, ADS, CVM health, and recent changes. Third, communicate clearly with the customer: what we know, what we are validating, immediate mitigations, and next update. If evidence points to product behavior or a defect, I would involve engineering with a clean escalation package.

### Q9. How do you distinguish overcommit from a product defect?

**Model answer:**

> I would look for correlation and reproducibility. If performance degradation aligns with high utilization, host imbalance, swap, oversized VMs, or recent capacity changes, it is more likely a sizing or configuration issue. If the cluster has adequate headroom and the behavior is abnormal, inconsistent with expected scheduler behavior, or triggered after a specific version change, then a product defect becomes more plausible. The key is to avoid assumptions and build an evidence-based case.

### Q10. How does your background apply here?

**Model answer:**

> In my current role, I manage 24/7 enterprise support, incidents, SLA, MTTR, escalations, monitoring, and customer communication. Overcommit is exactly the type of topic where technical depth and operational judgment meet. I may not be the deepest AHV scheduler expert in the room, but I know how to drive the escalation: define impact, collect the right evidence, coordinate specialists, communicate risk, and ensure the customer gets both immediate mitigation and a sustainable corrective action.

---

## 9. Connection with my experience

Your SaaS/cloud/incident-management background maps very well to this topic.

### Cloud operations

In AWS, Azure, or GCP, customers often think in terms of instance size, CPU credits, noisy neighbors, autoscaling, right-sizing, and reserved capacity. Overcommit is the on-prem/HCI equivalent of the same operational principle:

> allocated capacity does not always equal consumed capacity, but contention appears when demand becomes concurrent.

### Incident management

You already understand that performance incidents require:

* timeline,
* impact,
* change correlation,
* metrics,
* mitigation,
* ownership,
* RCA.

For Nutanix, you apply the same method, but with HCI-specific evidence:

* Prism alerts,
* VM metrics,
* host metrics,
* cluster capacity,
* CVM health,
* ADS behavior,
* AHV configuration,
* storage/network correlation.

### SLA / MTTR

Overcommit issues are dangerous because they can be intermittent. Averages may look fine, but users experience spikes. Your strength is managing ambiguity:

> “We may not yet know if this is CPU, memory, storage, or network, but we know the customer impact and we know what evidence we need next.”

### Team leadership

This is also a coaching topic. You can guide engineers to avoid common traps:

* looking only inside the guest OS,
* ignoring host-level contention,
* ignoring CVM health,
* assuming CPU usage equals CPU availability,
* missing recent changes,
* treating all overcommit as bad,
* treating all overcommit as safe.

### Customer communication

You can position yourself as someone who translates deep technical findings into customer-safe language:

> “The platform is functioning, but the cluster is running with insufficient headroom for the current workload profile. We recommend immediate mitigation and a capacity/right-sizing review.”

That is exactly the posture of a technical escalation manager.

---

## 10. Minimum I need to memorize

Memorize these points cold:

1. **Overcommit = allocating more virtual resources than physical resources.**
2. **CPU overcommit is common; memory overcommit is riskier.**
3. **CPU overcommit creates scheduling contention.**
4. **Memory overcommit uses ballooning first, host swap if needed.**
5. **Ballooning is preferable to host swap.**
6. **Host swap can severely degrade performance.**
7. **Nutanix positions AHV memory overcommit mainly for test/dev, not as a default production recommendation.**
8. **VirtIO/balloon drivers matter for AHV memory overcommit.**
9. **Overcommit affects HA because failover needs real headroom.**
10. **Overcommit affects live migration and upgrades because VMs need resources on target hosts.**
11. **Always check Prism, host metrics, VM sizing, CVM health, ADS, HA, and recent changes.**
12. **As a manager, your role is not to be the deepest scheduler expert; your role is to drive diagnosis, escalation quality, customer communication, and resolution.**

One sentence to memorize:

> “Overcommit is not inherently bad; unmanaged overcommit is bad. In support, I would validate actual contention, workload criticality, HA headroom, and recent changes before recommending mitigation.”

---

## 11. Advanced / optional level

You do **not** need to master these deeply for a manager interview, but you should recognize them:

* AHV scheduler internals.
* Detailed ADS placement logic.
* NUMA topology and CPU locality.
* Hyperthreading effects.
* Exact AHV performance metric names.
* aCLI/nCLI/API commands for metric extraction.
* Deep Linux KVM/QEMU memory management.
* Host swap implementation details.
* Advanced VM migration mechanics.
* Differences between AHV, ESXi, and Hyper-V memory reclamation.
* Database-specific memory behavior.
* Kubernetes node sizing on AHV.
* VDI boot storms and memory pressure.
* CVM resource reservation and storage I/O path.
* Capacity modeling for N+1 or N+2 HA.

A good interview line:

> “I understand the concepts and operational risks. For low-level scheduler behavior or exact internal thresholds, I would involve the appropriate SRE or engineering expert, but I know what evidence to collect and how to manage the escalation.”

---

## 12. Final checklist

Before an interview, you should be able to answer:

* Can I define overcommit in one sentence?
* Can I explain CPU vs memory overcommit?
* Can I explain why memory overcommit is riskier?
* Can I explain ballooning vs host swap?
* Can I say why VirtIO drivers matter?
* Can I explain vCPU:pCPU ratio?
* Can I explain why oversized VMs are harmful?
* Can I describe Prism checks during an escalation?
* Can I connect overcommit to HA and upgrades?
* Can I explain customer impact without sounding theoretical?
* Can I give a structured triage flow?
* Can I position myself as a technical escalation manager, not a Senior SRE IC?
* Can I connect this to my Harmonic experience in SLA, MTTR, incident response, and 24/7 support?
* Can I say “I would validate with data” instead of guessing?

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                                               |
| ------------------------ | --------------------------------------------------------------------------------------------------------------------- |
| aCLI                     | Acropolis CLI; Nutanix command-line interface for AHV/VM operations.                                                  |
| ADS                      | Acropolis Dynamic Scheduler; AHV scheduler that helps place and move VMs to reduce hotspots.                          |
| AHV                      | Acropolis Hypervisor; Nutanix native virtualization hypervisor.                                                       |
| Balloon driver           | Guest driver that helps reclaim unused VM memory for the hypervisor.                                                  |
| Ballooning               | Memory reclamation method where the guest cooperates with the hypervisor.                                             |
| Boot storm               | Many VMs booting simultaneously, often causing CPU, memory, or storage pressure.                                      |
| Capacity planning        | Process of ensuring enough resources for current and future workloads.                                                |
| Contention               | Multiple workloads competing for insufficient physical resources.                                                     |
| CPU Ready                | VMware term for vCPU waiting time; useful conceptually for CPU scheduling delay.                                      |
| CPU scheduling           | Hypervisor process of assigning vCPUs to physical CPU execution time.                                                 |
| CVM                      | Controller VM; Nutanix VM providing storage/data services on each node.                                               |
| Enterprise support       | Support model for business-critical customer environments with SLAs and escalations.                                  |
| Guest OS                 | Operating system running inside a VM.                                                                                 |
| Guest swap               | Swap configured inside the VM operating system.                                                                       |
| HA                       | High Availability; ability to keep or restart workloads after host failure.                                           |
| HCI                      | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform.                       |
| Host                     | Physical server running the hypervisor and VMs.                                                                       |
| Host hotspot             | A host under disproportionate resource pressure compared with others.                                                 |
| Host swap                | Hypervisor-level swapping of VM memory to disk.                                                                       |
| Hypervisor               | Software layer that runs and manages virtual machines.                                                                |
| KVM                      | Kernel-based Virtual Machine; Linux virtualization technology underlying many hypervisors, including AHV foundations. |
| Latency                  | Delay experienced by an application, VM, storage operation, or network request.                                       |
| Live migration           | Moving a running VM from one host to another with minimal/no downtime.                                                |
| Memory overcommit        | Allocating more VM memory than physically available, using reclamation/swap mechanisms.                               |
| MTTR                     | Mean Time To Resolve/Recover; key support and incident metric.                                                        |
| Noisy neighbor           | Workload that consumes excessive shared resources and affects others.                                                 |
| N+1                      | Capacity model where one host failure can be tolerated.                                                               |
| Overcommit               | Allocating more virtual resources than available physical resources.                                                  |
| Oversized VM             | VM assigned more CPU or memory than it actually needs.                                                                |
| Oversubscription         | Same general idea as overcommit; often used for vCPU:pCPU ratios.                                                     |
| pCPU                     | Physical CPU core/thread available on a host.                                                                         |
| Prism                    | Nutanix management and monitoring interface.                                                                          |
| Prism Central            | Centralized Nutanix management plane across clusters.                                                                 |
| Prism Element            | Cluster-level Nutanix management interface.                                                                           |
| RCA                      | Root Cause Analysis; formal explanation of cause, impact, and prevention.                                             |
| Right-sizing             | Adjusting VM resources to match actual workload demand.                                                               |
| SLA                      | Service Level Agreement; contractual or operational service commitment.                                               |
| SRE                      | Site Reliability Engineer; role focused on reliability, automation, operations, and incident response.                |
| Swap                     | Disk-backed memory used when RAM is insufficient.                                                                     |
| Triage                   | Initial structured investigation to classify impact, cause, urgency, and next actions.                                |
| vCPU                     | Virtual CPU assigned to a VM.                                                                                         |
| vCPU:pCPU ratio          | Ratio of allocated virtual CPUs to physical CPU resources.                                                            |
| VirtIO                   | Paravirtualized driver framework used for efficient VM devices on AHV/KVM.                                            |
| VM                       | Virtual Machine.                                                                                                      |
| Workload profile         | Actual behavior of an application: CPU, memory, I/O, latency sensitivity, and peaks.                                  |

[1]: https://www.nutanix.com/tech-center/blog/ahv-internals-memory-overcommit "AHV Internals: Memory Overcommit | Nutanix / tech center"
[2]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Acpu-and-memory-oversubscription.html&utm_source=chatgpt.com "CPU and Memory Oversubscription - portal.nutanix.com"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anutanix-ahv-cpu-configuration.html&utm_source=chatgpt.com "Nutanix AHV CPU Configuration"
