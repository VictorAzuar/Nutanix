# AHV / Virtualization — NUMA basics

## 1. Short definition

**NUMA — Non-Uniform Memory Access — is a server architecture where CPU sockets or CPU groups have memory that is “local” to them, and accessing local memory is faster than accessing memory attached to another CPU socket.**

In AHV, this matters because large VMs can span multiple physical NUMA nodes. If the VM’s vCPUs and memory are poorly aligned with the host’s NUMA topology, the workload can suffer from higher memory latency and unpredictable performance. Nutanix AHV supports **vNUMA**, allowing a VM to expose a virtual NUMA topology to the guest OS for better performance on large workloads. Nutanix documentation states that AHV hosts support vNUMA and that NUMA affects VM memory access time depending on memory location relative to the processor. ([Nutanix][1])

---

## 2. Clear explanation

A modern enterprise server often has more than one CPU socket. Each socket has direct access to a portion of the system memory. That memory is **local** to that CPU socket. The CPU can still access memory attached to another socket, but that access is **remote** and usually slower.

For virtualization, this creates a practical design question:

**Should a VM fit inside one NUMA node, or does it need to span multiple NUMA nodes?**

Small and medium VMs usually perform best when their vCPUs and memory fit inside a single physical NUMA node. Large VMs — for example large databases, analytics platforms, big application servers, or memory-intensive workloads — may require more CPU or memory than one physical NUMA node can provide. In that case, AHV can use **vNUMA** so the guest OS sees a NUMA-aware virtual topology.

The key idea is not “NUMA is good or bad.” The key idea is:

**NUMA must be respected when sizing and troubleshooting large VMs.**

Nutanix’s AHV best practices distinguish between **vNUMA/wide VMs**, **vUMA/narrow VMs**, and regular VMs. Wide VMs require more CPU or memory than is available in a single NUMA node; narrow VMs fit within one NUMA node; regular VMs may be served by one or more NUMA nodes depending on host capacity and placement. ([Nutanix][2])

A simple mental model:

| VM type              | Typical NUMA behavior                    | Interview-level meaning                   |
| -------------------- | ---------------------------------------- | ----------------------------------------- |
| Small VM             | Fits in one physical NUMA node           | Usually no special NUMA concern           |
| Medium VM            | Should ideally stay within one NUMA node | Avoid oversizing without reason           |
| Large VM / wide VM   | May span NUMA nodes                      | Consider vNUMA and workload awareness     |
| Latency-sensitive VM | NUMA alignment matters more              | Bad placement can increase memory latency |

In AHV, you do not need to present yourself as the person manually tuning every CPU scheduler decision. As a support manager, you need to understand when NUMA becomes relevant, what symptoms it can create, what questions to ask, and when to involve SRE / engineering / performance specialists.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, NUMA matters because enterprise customers often escalate performance problems that are not simple “CPU is high” or “memory is full” cases.

A customer may say:

> “After migrating from VMware to AHV, our database VM is slower.”
> “Latency increased after a maintenance event.”
> “The VM performs well on one node but poorly after migration.”
> “A large VM does not start after changing memory configuration.”
> “The cluster looks healthy, but the application team complains about response time.”

In those situations, a support manager must make sure the team does not stay at a superficial level. NUMA is one of the areas where **infrastructure topology, VM sizing, scheduler behavior, workload profile, and customer perception** meet.

For Nutanix specifically, this topic matters because AHV is the hypervisor layer in the HCI stack. Performance issues may involve compute, memory, storage, CVM behavior, guest OS configuration, workload design, or cluster placement. Nutanix best practices recommend keeping regular VMs within one host NUMA node when possible and using vNUMA for large VMs that require more CPU or memory than one physical NUMA node can provide. ([Nutanix][3])

As a manager, the value is not that you memorize every command. The value is that you can drive the right escalation path:

1. Confirm whether the issue is workload-specific or platform-wide.
2. Validate VM sizing and recent changes.
3. Check whether the VM is large enough for NUMA to matter.
4. Ask whether the issue appeared after migration, failover, host maintenance, or resizing.
5. Ensure SREs collect the right evidence before escalating to engineering.
6. Communicate clearly with the customer without overpromising.

---

## 4. Key concepts

### NUMA node

A **NUMA node** is a group of CPU cores and memory that are physically close together, usually associated with a CPU socket or CPU memory domain.

The practical point: CPU cores access local memory faster than remote memory.

### Local memory access

The VM’s vCPU runs on a physical CPU core and accesses memory local to that same NUMA node. This is the preferred scenario.

### Remote memory access

The VM’s vCPU runs on one NUMA node but accesses memory attached to another NUMA node. This can introduce additional latency.

Nutanix documentation explains that VM performance is optimal when CPU and memory resources come from the same physical NUMA node, and that memory latency can occur when CPU execution and memory access are on different NUMA nodes. ([Nutanix][4])

### vNUMA

**vNUMA** exposes a virtual NUMA topology to the guest VM. This allows the guest operating system and NUMA-aware applications to make better scheduling and memory allocation decisions.

Interview phrasing:

> “vNUMA is useful for large VMs because it helps the guest OS understand the underlying memory topology instead of treating all memory as equally local.”

AHV supports vNUMA on VMs and allows it to be enabled when creating or modifying VMs to optimize memory performance. ([Nutanix][1])

### vUMA

**vUMA — virtual Uniform Memory Access —** presents memory as uniform to the VM. It is more relevant for smaller or narrow VMs that fit inside one NUMA node.

For interview purposes, you only need to know that **vNUMA is for wide/large VMs**, while **vUMA or regular placement is generally enough for smaller VMs**.

### Wide VM

A **wide VM** is a VM whose CPU or memory requirement exceeds the capacity of a single physical NUMA node. These are the VMs where vNUMA is most relevant.

Nutanix documentation uses the idea that vNUMA or wide VMs require more CPU or memory than is available in one physical NUMA node. ([Nutanix][2])

### Narrow VM

A **narrow VM** fits inside one physical NUMA node. It usually does not need vNUMA complexity.

### VM sizing

NUMA issues often start with VM sizing. Oversized VMs can become harder to schedule efficiently and may span NUMA nodes unnecessarily.

A support manager should challenge the assumption that “more vCPU and more RAM always improves performance.” Sometimes oversizing creates worse performance due to scheduling, memory locality, or application behavior.

### Host heterogeneity

If a cluster has hosts with different CPU/memory layouts, large vNUMA-enabled VMs may behave differently depending on where they run. This matters during migrations, maintenance, failover, or capacity events.

Nutanix AHV best practice material recommends homogeneous AHV hosts when using vNUMA/vUMA, or at least understanding the NUMA boundaries of all nodes where these VMs may run. ([HubSpot][5])

### CPU scheduler

The AHV CPU scheduler is responsible for placing VM vCPUs on physical CPU resources. For a manager-level interview, you do not need kernel-level details, but you should understand that the scheduler and VM topology affect performance.

Nutanix best practices state that AHV’s CPU scheduler maintains balance and can spread demanding workloads across physical cores when needed. ([Nutanix][6])

### CVM relevance

In Nutanix, the **Controller VM — CVM —** is critical because it provides core Nutanix storage and data services on each node. NUMA can also be relevant to CVM performance in heavy I/O environments. Some Nutanix field installation documentation refers to pinning CVM vCPUs to a NUMA node to maximize I/O performance under heavy load. ([Nutanix][7])

For interviews, do not overfocus on CVM NUMA tuning unless asked. Mention it only to show you understand that Nutanix performance is not just guest VM performance; it also involves the distributed storage/controller layer.

---

## 5. How it appears in a real escalation

### Scenario 1 — Database VM slower after migration to AHV

A customer migrates a large SQL Server, Oracle, PostgreSQL, or SAP-related VM from VMware to AHV. After migration, the application team reports higher query latency.

Possible NUMA angle:

* The VM is oversized.
* The VM spans multiple NUMA nodes.
* vNUMA is not enabled or not aligned with workload needs.
* The guest OS or database is not NUMA-aware or not configured correctly.
* The VM was migrated to a host with different NUMA boundaries.
* The problem is blamed on AHV, but the root cause may be sizing or topology.

Support manager response:

> “I would first separate whether this is a platform-wide issue or limited to a specific large VM. Then I would have the team review VM sizing, host NUMA topology, recent migration/resizing events, guest OS metrics, and application latency. If the VM exceeds a single NUMA node, vNUMA becomes part of the investigation.”

### Scenario 2 — Performance changes after host maintenance

A VM performs normally before maintenance. After live migration, host restart, or failover, performance changes.

Possible NUMA angle:

* The VM moved to a host with different CPU/memory topology.
* The target host has less available local memory.
* The cluster is heterogeneous.
* VM placement changed.
* Other large VMs are competing for NUMA-local resources.

Support manager response:

> “I would correlate the performance degradation with maintenance events, VM movement, host placement, and resource contention. If only large VMs are affected, NUMA locality and vNUMA configuration would be part of the escalation checklist.”

### Scenario 3 — Large VM fails to start or behaves inconsistently

A customer creates a very large VM and reports it cannot start on some AHV hosts.

Possible NUMA angle:

* Not enough contiguous resources.
* vNUMA/vUMA configuration is incompatible with host topology.
* Cluster nodes are not homogeneous.
* The VM can start on one host but not another.

Nutanix AHV memory best practices warn that mixing vNUMA, vUMA, and regular VMs on the same AHV host can create start-up issues for vNUMA/vUMA VMs when other VMs migrate. ([Nutanix][3])

Support manager response:

> “I would treat this as a capacity, placement, and configuration issue rather than only a VM power-on issue. The team should validate host NUMA boundaries, VM configuration, current placement, and whether recent migrations consumed resources needed by the large VM.”

### Scenario 4 — Customer asks whether to enable vNUMA everywhere

This is a common enterprise misunderstanding.

Correct answer:

> “No. vNUMA is not a universal performance switch. It is mainly useful for large VMs that exceed one physical NUMA node. For smaller VMs, unnecessary vNUMA complexity can make operations and placement harder without delivering benefit.”

---

## 6. Triage questions I should ask

Use these in interviews to show structured thinking.

### Customer impact

1. Which application or service is affected?
2. Is this a single VM, multiple VMs, or the whole cluster?
3. What is the business impact: latency, outage, degraded throughput, failed batch job, SLA breach?
4. When did the issue start?
5. Was there a recent migration, resizing, upgrade, failover, maintenance window, or workload change?

### VM profile

6. How many vCPUs and how much RAM does the VM have?
7. Is this a large or wide VM?
8. Does the VM exceed the CPU or memory capacity of a single NUMA node?
9. Is vNUMA enabled?
10. Is the guest OS NUMA-aware?
11. Is the application NUMA-aware, for example database, JVM, analytics engine, or in-memory platform?

### Host / cluster profile

12. What AHV host is the VM running on?
13. Are the hosts homogeneous or heterogeneous?
14. What is the physical NUMA topology of the host?
15. Did the VM recently move to another host?
16. Are other large VMs competing for resources on the same host?
17. Is there CPU ready/wait, memory pressure, swapping, or ballooning-like behavior depending on the environment?

### Performance evidence

18. What metric changed: CPU, memory latency, application response time, disk latency, network latency, query time?
19. Is the issue visible from Prism, guest OS metrics, application logs, or monitoring tools?
20. Can we compare performance before and after the event?
21. Is there a baseline from before migration or before resizing?

### Escalation quality

22. What logs, timelines, metrics, and configuration snapshots have we collected?
23. Can we reproduce the issue?
24. Have we isolated AHV/platform factors from guest/application factors?
25. Do we need SRE, performance engineering, virtualization SME, storage SME, or account/customer success involvement?

---

## 7. Likely interview questions

1. What is NUMA and why does it matter in virtualization?
2. How would you explain vNUMA to a customer?
3. When would you enable vNUMA on AHV?
4. Should vNUMA be enabled for every VM?
5. What symptoms could indicate a NUMA-related performance issue?
6. How would you triage a large VM performance issue on AHV?
7. A customer migrated a database VM to AHV and performance degraded. What would you check?
8. How does NUMA relate to VM sizing?
9. What is the risk of oversizing VMs?
10. What is the difference between a wide VM and a narrow VM?
11. How would host heterogeneity affect vNUMA?
12. How would you manage a customer escalation where the customer blames AHV for poor performance?
13. How would you coordinate between support, SRE, engineering, and the customer?
14. What would you expect your SREs to collect before escalating to engineering?
15. How do you communicate a technical performance issue to executives?

---

## 8. Model answers in English

### Question: What is NUMA and why does it matter in virtualization?

**Model answer:**

> NUMA stands for Non-Uniform Memory Access. In a multi-socket server, each CPU socket has memory that is local to it. Accessing local memory is faster than accessing memory attached to another socket. In virtualization, this matters because a large VM may span multiple physical NUMA nodes. If the VM’s vCPUs and memory are not aligned efficiently, the workload can experience additional memory latency. For AHV support, I would mainly associate NUMA with large VM sizing, vNUMA configuration, host topology, and performance troubleshooting.

### Question: How would you explain vNUMA to a customer?

**Model answer:**

> vNUMA exposes a virtual NUMA topology to the guest operating system. It helps large VMs understand that not all CPU and memory access is equal, so the guest OS and NUMA-aware applications can make better scheduling and memory placement decisions. I would explain that vNUMA is not something we enable blindly for every VM. It is most useful for large or wide VMs that need more CPU or memory than a single physical NUMA node can provide.

### Question: When would you consider vNUMA on AHV?

**Model answer:**

> I would consider vNUMA when a VM is large enough that it exceeds the resources of a single physical NUMA node, especially for performance-sensitive workloads like databases, analytics, large application servers, or in-memory platforms. I would also check Nutanix best practices and application-specific recommendations. For smaller VMs that fit inside one NUMA node, I would avoid unnecessary complexity unless there is a specific reason.

### Question: A customer says their database VM became slower after migrating to AHV. What would you do?

**Model answer:**

> I would first establish impact and timeline: when the degradation started, what changed, and whether it affects only that VM or multiple workloads. Then I would review VM sizing, vCPU count, memory, host placement, AHV host topology, and whether the VM exceeds a single NUMA node. I would ask the team to compare pre- and post-migration metrics, including application latency, CPU utilization, memory pressure, and storage latency. If the VM is large, vNUMA and NUMA locality become part of the investigation. I would also avoid assuming AHV is the root cause until we have correlated infrastructure metrics with guest OS and application evidence.

### Question: Should vNUMA be enabled everywhere?

**Model answer:**

> No. vNUMA is not a universal optimization. It is mainly useful for wide VMs that need more resources than one physical NUMA node can provide. For smaller VMs, the best practice is usually to keep them within a single NUMA node when possible. Enabling advanced topology features without a workload reason can increase operational complexity and may create placement or start-up challenges in some environments.

### Question: How would you manage this as a support manager rather than an individual contributor?

**Model answer:**

> My role would be to structure the escalation, make sure the team collects the right evidence, and keep communication clear with the customer. I would ensure we have a timeline, business impact, VM configuration, host placement, cluster topology, relevant Prism metrics, guest OS metrics, and application evidence. I would assign the right technical owner, involve SRE or engineering when needed, and communicate in terms of impact, mitigation, next steps, and ownership. I would not personally try to replace the SRE, but I need enough technical fluency to challenge assumptions and prevent the escalation from going in circles.

### Question: What is the risk of oversizing VMs?

**Model answer:**

> Oversizing can make a VM harder to schedule efficiently and may force it to span NUMA nodes unnecessarily. More vCPUs and more memory do not always mean better performance. For latency-sensitive workloads, poor CPU and memory locality can create unpredictable performance. In support, I would look at whether the VM is sized according to actual workload demand or whether it was oversized as a generic response to performance complaints.

### Question: How does host heterogeneity affect NUMA troubleshooting?

**Model answer:**

> If the AHV cluster has hosts with different CPU models, socket counts, core counts, or memory layout, a large VM may behave differently depending on where it runs. This is especially relevant after maintenance, failover, or live migration. In a heterogeneous cluster, I would be more careful with large vNUMA-enabled VMs and would validate whether the target hosts have compatible NUMA boundaries and enough resources.

---

## 9. Connection with my experience

Your current experience maps well to this topic if you position it correctly.

You do **not** need to pretend you are already a Nutanix AHV performance engineer. Your angle is stronger:

> “I manage enterprise support operations, so I know how to turn a vague performance complaint into a structured escalation with evidence, ownership, and customer communication.”

Your existing background connects to NUMA in these ways:

| Your experience                  | How to connect it to NUMA / AHV                                                                                    |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| 24/7 enterprise support          | NUMA issues often appear as high-severity performance escalations, not clean textbook problems                     |
| Incident management              | You can structure timeline, impact, mitigation, root cause, and follow-up                                          |
| MTTR / SLA                       | NUMA misconfiguration can increase time to resolution if support teams only look at generic CPU/memory utilization |
| Cloud operations                 | Similar to instance sizing, placement groups, noisy neighbors, and performance-sensitive workloads                 |
| Monitoring with Grafana / Kibana | You know how to correlate infrastructure metrics, application logs, and timeline events                            |
| Jira / Confluence                | You can drive escalation hygiene: evidence, handoffs, known issues, KB creation                                    |
| Salesforce / customer support    | You understand customer communication, escalation ownership, and expectation management                            |
| Kubernetes / SaaS                | You understand that scheduling and locality matter, even if the abstraction layer hides hardware details           |

A strong interview positioning statement:

> “My strength is not that I have memorized every AHV command today. My strength is that I understand how hardware topology, virtualization, workload sizing, and support process interact. For NUMA-related issues, I would make sure the team validates VM size, host topology, recent movement, workload profile, and evidence from both the platform and the guest before escalating or recommending changes.”

---

## 10. Minimum I need to memorize

Memorize these points:

1. **NUMA means memory access is not uniform.**
2. **Local memory access is faster than remote memory access.**
3. **NUMA matters mainly for large / wide VMs.**
4. **vNUMA exposes NUMA topology to the guest OS.**
5. **Do not enable vNUMA everywhere.**
6. **Keep normal VMs within one NUMA node when possible.**
7. **Large VMs may need vNUMA if they exceed one physical NUMA node.**
8. **Oversizing VMs can hurt performance.**
9. **Host heterogeneity makes NUMA troubleshooting more complex.**
10. **In escalations, ask about VM size, host topology, recent migration/maintenance, workload type, and metrics.**

A concise verbal answer:

> “NUMA is important in AHV because large VMs may span CPU and memory domains. If vCPUs and memory are not placed efficiently, memory latency can increase. vNUMA helps large VMs expose that topology to the guest OS, but it should be used selectively, mainly for wide VMs. In support, I would look at VM sizing, host topology, recent movement, application profile, and performance metrics before deciding whether NUMA is part of the root cause.”

---

## 11. Advanced / optional level

You can leave these as advanced unless asked by SREs:

1. Exact AHV CLI commands for enabling/disabling vNUMA.
2. Deep KVM/libvirt implementation details.
3. Linux kernel NUMA balancing internals.
4. CPU pinning details outside specific support cases.
5. Database-specific NUMA tuning for Oracle, SQL Server, SAP HANA, etc.
6. Detailed interpretation of `numactl`, `/sys/devices/system/node`, or hypervisor-level NUMA maps.
7. Benchmark methodology for memory locality.
8. Advanced CVM NUMA pinning behavior.
9. Specific differences between AHV, ESXi, and Hyper-V NUMA schedulers.
10. Application-level memory allocators and NUMA-aware thread placement.

However, for a technical panel, it is useful to know these phrases:

* “NUMA locality”
* “Remote memory access”
* “Wide VM”
* “vNUMA topology”
* “Host NUMA boundary”
* “Heterogeneous cluster”
* “Workload-aware sizing”
* “Do not oversize blindly”
* “Correlate guest metrics with hypervisor metrics”

---

## 12. Final checklist

Before an interview, you should be able to answer:

| Question                                                     | Ready? |
| ------------------------------------------------------------ | ------ |
| Can I define NUMA in one sentence?                           | ☐      |
| Can I explain why local memory is faster than remote memory? | ☐      |
| Can I explain what vNUMA does in AHV?                        | ☐      |
| Can I explain when vNUMA is useful?                          | ☐      |
| Can I explain why vNUMA should not be enabled everywhere?    | ☐      |
| Can I describe a NUMA-related escalation?                    | ☐      |
| Can I connect NUMA to VM sizing?                             | ☐      |
| Can I connect NUMA to customer performance complaints?       | ☐      |
| Can I explain what evidence I would ask SREs to collect?     | ☐      |
| Can I answer as a support manager, not as a kernel engineer? | ☐      |

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                                          |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| AOS                      | Acropolis Operating System; Nutanix distributed software platform providing storage and virtualization services. |
| AHV                      | Acropolis Hypervisor; Nutanix’s native hypervisor based on KVM.                                                  |
| Application latency      | Time an application takes to respond; often the customer-visible symptom.                                        |
| Baseline                 | Known normal performance level used for comparison.                                                              |
| CPU locality             | Running CPU work close to the memory it uses.                                                                    |
| CPU scheduler            | Hypervisor component that places VM vCPUs onto physical CPU resources.                                           |
| CVM                      | Controller VM; Nutanix VM on each node providing storage/data services.                                          |
| Escalation               | A support case requiring higher technical, management, or engineering involvement.                               |
| Guest OS                 | Operating system running inside the VM.                                                                          |
| HCI                      | Hyperconverged Infrastructure; combines compute, storage, and virtualization in one platform.                    |
| Heterogeneous cluster    | Cluster with hosts that have different hardware configurations.                                                  |
| Homogeneous cluster      | Cluster with similar or identical hardware across nodes.                                                         |
| Host                     | Physical server running AHV and Nutanix services.                                                                |
| Local memory             | Memory attached to the same NUMA node as the CPU executing the workload.                                         |
| Memory latency           | Delay when CPU accesses memory; higher latency can reduce performance.                                           |
| MTTR                     | Mean Time To Resolve / Repair; average time needed to restore service.                                           |
| Narrow VM                | VM that fits within a single physical NUMA node.                                                                 |
| NUMA                     | Non-Uniform Memory Access; architecture where memory access time depends on CPU-memory location.                 |
| NUMA boundary            | CPU and memory limit of a physical NUMA node.                                                                    |
| NUMA node                | CPU/memory locality domain, often associated with a CPU socket.                                                  |
| NUMA topology            | Layout of CPU sockets, cores, and local memory domains.                                                          |
| Oversizing               | Assigning more vCPU or memory than the workload needs, which can hurt scheduling/performance.                    |
| Performance degradation  | Measurable or perceived worsening of system/application behavior.                                                |
| Prism                    | Nutanix management interface used for cluster and VM operations.                                                 |
| Remote memory            | Memory attached to a different NUMA node from the executing CPU.                                                 |
| SLA                      | Service Level Agreement; committed service or support target.                                                    |
| SRE                      | Site Reliability Engineer; role focused on reliability, operations, automation, and incident response.           |
| Triage                   | Initial structured investigation to classify impact, scope, evidence, and next action.                           |
| vCPU                     | Virtual CPU assigned to a VM.                                                                                    |
| VM                       | Virtual Machine.                                                                                                 |
| VM placement             | The host and physical resources where a VM runs.                                                                 |
| vNUMA                    | Virtual NUMA; exposes NUMA topology to a VM for better large-workload performance.                               |
| vRAM                     | Virtual RAM assigned to a VM.                                                                                    |
| vUMA                     | Virtual Uniform Memory Access; presents memory as uniform to the VM.                                             |
| Wide VM                  | Large VM requiring more CPU or memory than one NUMA node can provide.                                            |

[1]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v10_3%3Aahv-vm-memory-management-vnuma-uma-c.html?utm_source=chatgpt.com "AHV 10.3 - Virtual Machine Memory Management (vNUMA)"
[2]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anonuniform-memory-access-and-guest-vms.html&utm_source=chatgpt.com "Nonuniform Memory Access and Guest VMs - portal.nutanix.com"
[3]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anutanix-ahv-memory-configuration.html&utm_source=chatgpt.com "Nutanix AHV Memory Configuration"
[4]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v6_6%3Aahv-vm-memory-management-vnuma-uma-c.html&utm_source=chatgpt.com "AHV 6.6 - Virtual Machine Memory Management (vNUMA)"
[5]: https://f.hubspotusercontent40.net/hubfs/6083598/TechHub/ebooks/Nutanix-AHV-Best-Practices.pdf?utm_source=chatgpt.com "AHV Best Practices - f.hubspotusercontent40.net"
[6]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anutanix-ahv-cpu-configuration.html&utm_source=chatgpt.com "Nutanix AHV CPU Configuration"
[7]: https://portal.nutanix.com/page/documents/details?targetId=Field-Installation-Guide-v5_2%3Afie-cvm-cpu-allocation.html&utm_source=chatgpt.com "Foundation 5.2.x - CVM vCPU and vRAM Allocation - Nutanix"
