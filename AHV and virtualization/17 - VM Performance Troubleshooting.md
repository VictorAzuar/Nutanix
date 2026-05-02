# AHV / Virtualization — VM performance troubleshooting

## 1. Short definition

**VM performance troubleshooting on AHV** is the structured process of identifying why a virtual machine running on **Nutanix AHV** is slow, unstable, or not meeting expected performance by checking the full stack: **guest OS, application, VM configuration, AHV hypervisor, host resources, storage path, network path, Controller VM, cluster health, and external dependencies**.

In an interview, the key message is:

> “I would not assume the VM is slow because of the hypervisor. I would first define the symptom and business impact, then isolate whether the bottleneck is CPU, memory, storage, network, guest OS, application, host contention, or cluster-level health.”

---

## 2. Clear explanation

AHV is Nutanix’s native hypervisor. In a Nutanix environment, VMs run on AHV hosts, while the Nutanix distributed storage and control services are handled through **Controller VMs**, usually called **CVMs**. This is important because VM performance is not only about the VM itself; it depends on the interaction between the VM, the AHV host, the CVM, the storage fabric, and the network.

A VM performance issue usually appears as one of these symptoms:

| Symptom                                  | Possible area                                                              |
| ---------------------------------------- | -------------------------------------------------------------------------- |
| High CPU usage inside the VM             | Application load, insufficient vCPU, runaway process                       |
| Low CPU usage but application still slow | CPU Ready / scheduling contention, I/O wait, locks                         |
| High disk latency                        | Storage contention, CVM issue, disk group issue, replication/snapshot load |
| High memory pressure                     | Under-sized VM, memory leak, swapping, overcommit                          |
| Packet loss or poor throughput           | vNIC, VLAN, physical NIC, bond, MTU, upstream switch                       |
| Intermittent slowness                    | Noisy neighbor, migrations, backup jobs, snapshots, peak workload          |
| One VM affected                          | VM config, guest OS, application, vDisk/vNIC                               |
| Multiple VMs on same host affected       | Host resource contention, CVM, physical NIC, local hardware                |
| Multiple VMs across cluster affected     | Cluster-wide storage/network/control-plane issue                           |

The practical troubleshooting flow is:

1. **Define the symptom**

   * What is slow?
   * Since when?
   * Is it constant or intermittent?
   * Is there a customer/business impact?
   * What changed recently?

2. **Scope the blast radius**

   * One VM?
   * One host?
   * One subnet?
   * One storage container?
   * One cluster?
   * One customer/application?

3. **Check Prism / monitoring**

   * VM CPU usage
   * CPU Ready Time
   * Memory usage
   * Storage latency
   * IOPS / throughput
   * Network throughput / drops
   * Host contention
   * Alerts and recent tasks

4. **Compare guest-level and hypervisor-level metrics**

   * Guest OS may show high CPU, iowait, swapping, or process-level issues.
   * Prism/AHV may show whether the hypervisor or infrastructure is constraining the VM.

5. **Separate configuration problems from contention**

   * Configuration problem: wrong vCPU count, insufficient RAM, missing VirtIO drivers, wrong disk/vNIC configuration.
   * Contention problem: host overloaded, storage latency, CPU Ready, network bottleneck, noisy neighbor.

6. **Act based on evidence**

   * Resize VM carefully.
   * Move/migrate VM if host contention is suspected.
   * Check CVM/cluster health.
   * Validate network path.
   * Engage SRE/storage/network/product teams if needed.
   * Communicate impact, next action, owner, and ETA.

Nutanix documentation explicitly defines **CPU Ready Time** as the percentage of time a VM waits to use physical CPU out of the total CPU time allotted to the VM, which makes it one of the key metrics when a VM appears slow despite not showing high guest CPU usage. ([Nutanix][1])

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, VM performance troubleshooting matters because performance escalations are among the most sensitive enterprise cases. Customers usually do not open a ticket saying “CPU Ready is high”; they say:

> “Our ERP is slow.”
> “Our database latency has increased.”
> “Users are complaining.”
> “After migrating to Nutanix, performance is worse.”
> “The customer believes AHV is the root cause.”

Your role is not to be the deepest AHV engineer in the room. Your role is to make sure the team:

* Frames the problem correctly.
* Avoids assumptions.
* Collects the right evidence.
* Separates customer perception from technical causality.
* Drives escalation across support, SRE, engineering, networking, storage, and account teams.
* Communicates clearly with enterprise customers.
* Protects SLA, MTTR, customer trust, and internal focus.

A strong answer should show that you understand **performance troubleshooting as an escalation discipline**, not just as a set of commands.

For Nutanix specifically, this matters because AHV, Prism, AOS, CVMs, storage services, and networking are tightly integrated. A VM issue may be caused by the guest OS, but it may also be caused by host contention, storage latency, network configuration, or cluster health. Nutanix’s Prism performance monitoring supports multiple entity types, including VMs, hosts, disks, storage pools, storage containers, volume groups, remote sites, protection domains, replication links, virtual disks, and clusters, which reflects the need to troubleshoot across layers rather than only inside the VM. ([Nutanix][2])

---

## 4. Key concepts

### 4.1 AHV

**AHV** is Nutanix’s native hypervisor. It runs virtual machines on Nutanix nodes and integrates with the broader Nutanix platform.

For interview purposes, do not only say:

> “AHV is a hypervisor.”

Say:

> “AHV is the Nutanix-native virtualization layer. When troubleshooting VM performance on AHV, I would look at the VM configuration, host scheduling, storage path through the CVM, network path, and cluster health through Prism.”

---

### 4.2 Prism Central and Prism Element

**Prism Element** is used to manage an individual Nutanix cluster.
**Prism Central** provides centralized management across clusters.

For performance troubleshooting, Prism is where you would typically start for:

* VM metrics
* Host metrics
* Alerts
* Recent tasks
* Cluster health
* Storage latency
* Network visibility
* Capacity and resource trends

Nutanix documentation notes that Prism Central can create VMs in Acropolis-managed clusters and references VM creation/configuration through the Infrastructure application under Compute & Storage > VMs. ([Nutanix][3])

---

### 4.3 vCPU and CPU Ready

A VM can have enough assigned vCPUs but still suffer if the hypervisor cannot schedule those vCPUs efficiently.

**CPU Ready** means the VM is ready to run, but it is waiting for physical CPU time. This is a classic virtualization troubleshooting concept and is highly relevant in AHV. Nutanix defines the AHV CPU Ready metric as the ratio of VM wait time to the total CPU time allotted to the VM. ([Nutanix][1])

Symptoms of CPU scheduling contention:

* Application slow but guest CPU not always maxed.
* High CPU Ready.
* Multiple VMs impacted on same host.
* Oversized VM with too many vCPUs.
* Host CPU contention.
* Noisy neighbor workload.

Important nuance:

> More vCPU is not always better.

Nutanix AHV best-practice guidance warns against configuring a single VM with more vCPU cores than the physical CPU cores available on the AHV host, because that can cause significant performance problems. ([Nutanix][4])

---

### 4.4 Memory pressure

Memory issues can appear as:

* Guest OS swapping.
* High memory utilization.
* Application heap exhaustion.
* VM under-sizing.
* Host memory pressure.
* NUMA-related performance issues.
* Incorrect large VM placement.

AHV supports memory and CPU hot-plug for guest VMs, allowing increases while powered on, but Nutanix documentation notes that memory and CPU cannot be decreased while the VM is powered on, and cores per socket cannot be changed while powered on. ([Nutanix][5])

Interview angle:

> “Hot-add can help restore service quickly, but I would still review the permanent sizing after the incident. Emergency mitigation and long-term right-sizing are different activities.”

---

### 4.5 vNUMA

**vNUMA** matters for large VMs. If a VM is very large, CPU and memory locality can affect performance.

Nutanix documentation on AHV memory performance recommends configuring vNUMA nodes in relevant scenarios so that local memory access is used efficiently for each CPU. ([Nutanix][6])

For your target role, you do not need to be a vNUMA tuning expert, but you should know this:

> “For very large VMs, I would involve a senior SRE or performance specialist to validate vNUMA, CPU topology, NUMA locality, and host placement.”

---

### 4.6 Storage latency

Storage performance problems are among the most common causes of perceived VM slowness.

You need to distinguish:

* Guest disk latency.
* AHV virtual disk latency.
* Nutanix storage container latency.
* Physical disk latency.
* CVM-related issues.
* Replication/snapshot/backup impact.
* Application-generated I/O patterns.

Typical metrics:

* Read latency
* Write latency
* IOPS
* Throughput
* I/O size
* Queue depth
* Storage controller behavior
* Disk saturation
* CVM CPU/memory
* Cluster resiliency operations

A customer may say “the VM is slow,” but the actual issue may be storage latency caused by a backup job, snapshot storm, rebuild, replication, or an overloaded workload.

---

### 4.7 CVM

The **Controller VM** is central in Nutanix architecture. It provides storage and platform services on each node.

For VM performance troubleshooting, the CVM matters because the VM’s storage path depends on Nutanix storage services. If CVMs are unhealthy, constrained, or affected by cluster issues, multiple VMs can experience performance degradation.

You should not overstate this as “every issue is CVM-related.” A better interview answer is:

> “I would check CVM health when the symptoms suggest storage or cluster-level impact, especially if several VMs on the same node or cluster are affected.”

---

### 4.8 VirtIO drivers

For Windows and some guest OS configurations, **Nutanix VirtIO** drivers are important for optimized virtual disk and network performance.

A missing, outdated, or incorrect driver can create poor disk or network performance. Nutanix documentation explicitly references VirtIO installation for Windows VMs when creating VMs through Prism Central. ([Nutanix][3])

Interview-safe phrasing:

> “If the issue is VM-specific, especially after migration or OS changes, I would validate guest tools and drivers, including VirtIO where applicable.”

---

### 4.9 Network path

Network performance troubleshooting must include both virtual and physical layers:

* VM vNIC
* AHV virtual switch
* VLAN
* Bonding mode
* Physical NIC
* MTU
* Upstream switch
* Firewall/load balancer
* Packet drops
* East-west vs north-south traffic
* DNS or external service dependency

Nutanix maintains AHV networking best-practice documentation, with recent updates covering production network changes, active-backup bond mode, and AHV networking recommendations. ([Nutanix][7])

---

### 4.10 Hot-plug limitations

Hot-plug is useful during escalations but has limitations.

Nutanix documentation states that AHV allows memory and CPU increases while a VM is powered on, but you cannot decrease them while the VM is powered on, and you cannot change cores per socket while the VM is powered on. ([Nutanix][5])

Also, Nutanix AHV CPU guidance notes that adding vCPUs to a running VM may not create new multiqueue virtio-scsi queues, which can limit performance; the guidance is to size VM compute resources when creating them or while powered off when possible. ([Nutanix][4])

This is a strong interview point because it shows maturity:

> “I would not treat hot-add as a universal fix. It can be a mitigation, but it may not fully resolve storage queueing or VM topology issues until a proper maintenance window.”

---

## 5. How it appears in a real escalation

### Scenario 1 — “Critical database VM is slow after migration to AHV”

Customer impact:

* ERP users report slow transactions.
* Customer suspects AHV.
* Severity is high because business operations are affected.

How you should reason:

1. Confirm business impact and urgency.
2. Identify whether the issue started after migration, patching, workload change, or backup.
3. Compare before/after metrics.
4. Check VM CPU, CPU Ready, memory, disk latency, IOPS, throughput, and network.
5. Check whether other VMs on the same host are affected.
6. Validate VirtIO drivers and guest OS configuration.
7. Check storage latency at VM, container, and cluster level.
8. Check cluster health, CVM status, alerts, and recent tasks.
9. Engage SRE/performance/storage specialists if deep analysis is needed.
10. Communicate findings and next actions to the customer.

Strong manager framing:

> “I would make sure we do not debate opinions with the customer. We need a fact-based timeline, correlated metrics, and clear ownership per layer.”

---

### Scenario 2 — “VM has low CPU usage but the application is slow”

Possible causes:

* CPU Ready / scheduling contention.
* Storage latency causing I/O wait.
* Database locks.
* Network latency to dependency.
* Application thread pool exhaustion.
* Guest OS issue.
* Oversized VM causing scheduling inefficiency.

What you should say:

> “Low guest CPU does not prove that compute is healthy. I would compare guest metrics with AHV metrics, especially CPU Ready, storage latency, and network behavior.”

---

### Scenario 3 — “Several VMs on one AHV host are slow”

Possible causes:

* Host CPU contention.
* Host memory pressure.
* CVM issue on that node.
* Physical NIC issue.
* Local disk/disk group problem.
* Noisy neighbor VM.
* Recent VM migrations or maintenance activity.

Management action:

* Declare clear incident ownership.
* Assign one person to customer communication.
* Assign technical streams: compute, storage, network, guest/application.
* Check whether live migration or workload balancing is appropriate.
* Preserve evidence before making changes.
* Avoid repeated random mitigations.

---

### Scenario 4 — “Performance issue only during backup window”

Possible causes:

* Snapshot activity.
* Backup proxy load.
* Increased read I/O.
* Replication traffic.
* Storage latency.
* Network saturation.
* Application quiescing impact.

Interview answer:

> “I would correlate the degradation with scheduled tasks, backup jobs, snapshots, and replication. If the issue is time-bound, the timeline is often the fastest path to root cause.”

---

## 6. Triage questions I should ask

### Business and impact

* What application or service is affected?
* How many users/customers are impacted?
* Is this production?
* What is the severity?
* Is there revenue, contractual, or SLA impact?
* Is there a workaround?

### Timeline

* When did the issue start?
* Is it constant or intermittent?
* Did it start after a migration, upgrade, patch, configuration change, backup, or workload increase?
* Does it happen at specific times?

### Scope

* Is only one VM affected?
* Are other VMs on the same host affected?
* Are other VMs in the same cluster affected?
* Are VMs on other clusters affected?
* Is the issue specific to one VLAN, subnet, storage container, tenant, or application?

### VM layer

* What are the VM specs: vCPU, RAM, disks, NICs?
* Was the VM recently resized?
* Is the VM oversized or undersized?
* Is hot-add involved?
* Are VirtIO drivers installed and current?
* Is the guest OS swapping?
* Is there high CPU, high iowait, or process-level contention?

### AHV / host layer

* What is CPU Ready?
* Is the host overloaded?
* Are there noisy neighbors?
* Are there migrations or scheduled tasks?
* Is host memory under pressure?
* Are alerts present in Prism?

### Storage layer

* What is read/write latency?
* What are IOPS and throughput?
* Is latency VM-specific or cluster-wide?
* Are snapshots, backups, replication, or rebuilds running?
* Is there a CVM issue?
* Are disks or storage containers reporting problems?

### Network layer

* Is throughput low or latency high?
* Any packet loss?
* Any drops/errors on vNIC, host NIC, or switch?
* Is the issue east-west or north-south?
* Any VLAN, MTU, routing, firewall, or load balancer changes?
* Is DNS or an external dependency involved?

### Customer communication

* What has already been tried?
* What evidence has been collected?
* What is the customer’s expectation for updates?
* Who is the customer technical owner?
* Do we need a bridge call?
* Do we need engineering escalation?

---

## 7. Likely interview questions

### Leadership / support manager questions

1. How would you handle a critical customer escalation where a VM on AHV is slow?
2. How do you prevent your team from jumping to conclusions during performance incidents?
3. How would you communicate with a customer who insists Nutanix is the root cause?
4. How do you organize troubleshooting during a Sev1 escalation?
5. How do you balance fast mitigation with proper root cause analysis?
6. How do you measure whether your support team handles performance cases well?
7. How do you coach engineers who are strong technically but weak in customer communication?
8. How do you decide when to escalate to engineering?
9. How would you handle repeated performance escalations from the same enterprise account?
10. How do you create a post-incident improvement plan?

### Technical / SRE questions

1. What are the main areas to check when a VM on AHV is slow?
2. What is CPU Ready and why does it matter?
3. Why can adding more vCPUs make performance worse?
4. How would you distinguish CPU bottleneck from storage bottleneck?
5. What Nutanix components are relevant in VM storage performance?
6. What role does the CVM play?
7. What would you check if multiple VMs on the same AHV host are slow?
8. What would you check if only one VM is slow?
9. How do VirtIO drivers affect VM performance?
10. How would you troubleshoot network performance for an AHV VM?
11. What metrics would you check in Prism?
12. What is the difference between mitigation and root cause analysis?
13. How would you approach intermittent performance degradation?
14. How would you troubleshoot performance after a migration from VMware to AHV?
15. When would you involve storage, network, or engineering teams?

---

## 8. Model answers in English

### Q1. How would you troubleshoot a slow VM running on AHV?

**Model answer:**

> I would start by defining the symptom, impact, and timeline. “Slow VM” is too broad, so I would clarify whether the problem is CPU, disk latency, network latency, application response time, or user experience. Then I would scope the blast radius: one VM, multiple VMs on the same host, one cluster, one VLAN, or one application.
>
> From there, I would compare guest OS metrics with Prism and AHV metrics. I would check CPU usage, CPU Ready, memory pressure, swapping, disk latency, IOPS, throughput, network errors, host contention, CVM health, cluster alerts, and recent tasks such as backups, snapshots, migrations, or upgrades.
>
> As a support manager, my role would be to make sure the team follows an evidence-based workflow, assigns ownership per layer, communicates clearly with the customer, and separates short-term mitigation from root cause analysis.

---

### Q2. What is CPU Ready?

**Model answer:**

> CPU Ready is the time a VM is ready to run but is waiting for physical CPU resources from the hypervisor. It is important because a VM can show low or moderate guest CPU usage and still perform poorly if it is waiting to be scheduled.
>
> In AHV, I would use CPU Ready as one indicator of host CPU contention or VM sizing problems. I would also check whether the VM is oversized, whether the host is overloaded, and whether other VMs on the same host are affected. I would avoid assuming that adding more vCPUs will fix the issue, because oversized VMs can sometimes make scheduling less efficient.

---

### Q3. Why can adding more vCPUs make performance worse?

**Model answer:**

> Adding vCPUs can help if the VM is genuinely CPU-starved, but it can make things worse if the root cause is scheduling contention or poor sizing. A larger VM may be harder to schedule efficiently, especially on a busy host. Also, with AHV, some performance characteristics, such as storage queue behavior, may depend on how the VM was sized and started, so hot-adding CPU is not always equivalent to properly sizing the VM during a maintenance window.
>
> I would treat vCPU increase as a controlled change, not as a default reaction. First I would validate CPU usage, CPU Ready, application behavior, and host contention.

---

### Q4. How would you distinguish CPU bottleneck from storage bottleneck?

**Model answer:**

> I would compare guest OS and hypervisor metrics. For CPU, I would look at guest CPU usage, CPU Ready, host CPU contention, and whether the workload is CPU-bound. For storage, I would look at read/write latency, IOPS, throughput, queueing, guest iowait, and whether the issue correlates with backups, snapshots, replication, or other high I/O activity.
>
> If the application is slow but CPU is not saturated, high disk latency or iowait could point to storage. If CPU Ready is high, the VM may be waiting for CPU scheduling. I would also check whether other VMs on the same host or storage container show similar symptoms.

---

### Q5. What role does the CVM play in performance troubleshooting?

**Model answer:**

> The CVM is a key part of Nutanix architecture because it provides storage and platform services on each node. If there are storage-related symptoms, such as high disk latency across multiple VMs, I would check CVM health, resource usage, alerts, and cluster-level storage behavior.
>
> I would not assume every VM performance issue is caused by the CVM, but I would definitely include it in the investigation when the blast radius suggests a host-level or cluster-level issue.

---

### Q6. How would you handle a customer who says “AHV is slow”?

**Model answer:**

> I would acknowledge the impact first, then move the conversation from a broad conclusion to measurable evidence. I would say something like: “We understand the application is slow and we will treat it with urgency. To identify whether the bottleneck is AHV, guest OS, storage, network, or application-related, we need to correlate metrics across the VM, host, cluster, and application timeline.”
>
> Then I would organize the investigation, define update cadence, assign owners, and make sure we provide clear findings rather than defensive explanations. The goal is not to prove the customer wrong; the goal is to isolate the bottleneck and restore service.

---

### Q7. What would you check if several VMs on the same host are slow?

**Model answer:**

> If several VMs on the same host are affected, I would suspect a host-level issue or a localized resource bottleneck. I would check host CPU and memory utilization, CPU Ready across affected VMs, CVM health on that node, physical NIC errors, storage path health, local disk alerts, and recent tasks such as migrations or maintenance.
>
> I would also compare the affected host with healthy hosts in the same cluster. If the evidence points to host contention, a mitigation could be moving workloads, but I would coordinate that carefully and preserve data for root cause analysis.

---

### Q8. What would you check if only one VM is slow?

**Model answer:**

> If only one VM is affected, I would start with VM-specific factors: guest OS metrics, application logs, process usage, memory pressure, swapping, disk latency, vNIC behavior, drivers, VM configuration, recent changes, and whether the VM was migrated or resized.
>
> I would still check the AHV and cluster view, but the initial hypothesis would be narrower: VM configuration, guest OS, application, driver, or workload-specific behavior.

---

### Q9. How do you manage this kind of escalation as a manager rather than as an individual contributor?

**Model answer:**

> As a manager, I would make sure the right troubleshooting structure is in place. I would define severity, business impact, timeline, customer communication cadence, technical owners, and escalation path. I would expect the engineers to perform the deep technical analysis, but I need to understand the technical domains well enough to challenge assumptions, ask the right questions, and remove blockers.
>
> My value is to keep the investigation disciplined: clear hypotheses, evidence per layer, no random changes, documented actions, customer-facing clarity, and a post-incident review that reduces recurrence.

---

### Q10. How would you connect this to your previous experience?

**Model answer:**

> In my current role, I manage a 24/7 enterprise support operation where incident management, SLA, MTTR, escalations, monitoring, and customer communication are central. The technology stack is different, but the operating model is very similar: define impact, triage quickly, correlate metrics, assign ownership, communicate clearly, and drive the incident to mitigation and root cause.
>
> For Nutanix, I would bring that operational discipline and apply it to AHV, Prism, AOS, storage, networking, and enterprise virtualization cases. I am not positioning myself as a senior SRE individual contributor, but as a technical escalation manager who can understand the architecture, guide the process, and ensure the customer receives a structured resolution.

---

## 9. Connection with my experience

Your strongest bridge is this:

> You already understand production operations. You now need to map that experience into Nutanix terminology and architecture.

### From Harmonic / SaaS / cloud operations to Nutanix AHV

| Your experience                | Nutanix equivalent                                                                  |
| ------------------------------ | ----------------------------------------------------------------------------------- |
| Incident management            | Sev1 / Sev2 enterprise escalation handling                                          |
| SLA / MTTR                     | Support KPIs, time-to-mitigation, time-to-resolution                                |
| Monitoring with Grafana/Kibana | Prism metrics, alerts, logs, cluster dashboards                                     |
| Cloud operations               | Private cloud / hybrid cloud operations                                             |
| Kubernetes troubleshooting     | Multi-layer troubleshooting: workload, node, network, storage                       |
| AWS/Azure/GCP                  | Infrastructure abstraction, capacity, noisy neighbor, availability zones / clusters |
| Jira/Confluence                | Case tracking, RCA documentation, knowledge base                                    |
| Salesforce                     | Enterprise support case lifecycle                                                   |
| Team coordination              | Swarming, escalation ownership, communication cadence                               |
| Customer-facing incidents      | Executive-level enterprise communication                                            |

Your positioning should be:

> “I know how to run high-pressure production escalations. I am building deeper Nutanix-specific technical fluency so I can ask the right questions, challenge assumptions, and help the team move faster.”

This is credible and aligned with a manager role.

---

## 10. Minimum I need to memorize

You should be able to explain these confidently:

1. **AHV is Nutanix’s native hypervisor.**
2. **Prism is the first place to check VM, host, cluster, storage, and alert metrics.**
3. **A slow VM is not automatically a hypervisor issue.**
4. Troubleshooting must isolate:

   * CPU
   * Memory
   * Storage
   * Network
   * Guest OS
   * Application
   * Host contention
   * Cluster health
5. **CPU Ready** means the VM is waiting for physical CPU scheduling.
6. More vCPU is not always better.
7. **Storage latency** is a common cause of perceived VM slowness.
8. **CVM health matters**, especially for storage or host/cluster-wide symptoms.
9. **VirtIO drivers matter**, especially for Windows VMs or migrated workloads.
10. Always scope the blast radius:

    * One VM
    * One host
    * One cluster
    * One network
    * One application
11. During escalation, separate:

    * Symptom
    * Impact
    * Timeline
    * Scope
    * Hypothesis
    * Evidence
    * Mitigation
    * Root cause
12. As a manager, you are expected to drive structure, ownership, communication, and escalation quality.

---

## 11. Advanced / optional level

You do **not** need to master these deeply before the interview, but knowing they exist helps you sound mature:

| Advanced area               | What to know at manager level                                                           |
| --------------------------- | --------------------------------------------------------------------------------------- |
| vNUMA                       | Important for large VMs and CPU/memory locality                                         |
| VirtIO queueing             | Hot-added vCPUs may not fully change storage queue behavior                             |
| RDMA                        | Can reduce CPU/network overhead in specific high-performance scenarios                  |
| AHV CLI / aCLI              | Useful for engineers; you should know it exists                                         |
| NCC health checks           | Nutanix Cluster Check can validate cluster health                                       |
| ADS                         | Acropolis Dynamic Scheduling can help address hotspots through VM migration             |
| Storage internals           | Stargate, Cassandra, Curator are Nutanix internals; know names, not deep implementation |
| Snapshot/replication impact | Important in time-correlated performance cases                                          |
| Physical NIC bonding        | Relevant when network symptoms affect VMs                                               |
| Guest OS tuning             | Linux/Windows tuning may be needed for high-performance workloads                       |
| Database-specific tuning    | Often outside Nutanix root cause but critical in enterprise escalations                 |

Nutanix AHV documentation references **Acropolis Dynamic Scheduling**, which monitors compute and storage I/O contentions or hotspots and can create a migration plan to reduce hotspots by migrating VMs. ([Nutanix][8])

---

## 12. Final checklist

Before an interview, practice saying this out loud:

### Technical checklist

* [ ] I can explain what AHV is.
* [ ] I can explain what Prism is used for.
* [ ] I can explain CPU Ready.
* [ ] I can explain why more vCPU is not always better.
* [ ] I can separate CPU, memory, storage, and network bottlenecks.
* [ ] I can explain the role of the CVM.
* [ ] I can describe how VirtIO drivers affect performance.
* [ ] I can explain why VM performance issues require guest + hypervisor + cluster correlation.
* [ ] I can describe what I would check if one VM is slow.
* [ ] I can describe what I would check if multiple VMs are slow.
* [ ] I can explain how backups, snapshots, and replication may affect performance.
* [ ] I can explain how I would use metrics and timelines.

### Manager checklist

* [ ] I can define customer impact clearly.
* [ ] I can structure a Sev1 bridge.
* [ ] I can assign technical workstreams.
* [ ] I can set customer update cadence.
* [ ] I can prevent random troubleshooting.
* [ ] I can escalate to SRE/engineering with evidence.
* [ ] I can separate mitigation from RCA.
* [ ] I can connect this to SLA, MTTR, KPIs, and customer trust.
* [ ] I can position myself as a technical escalation manager, not a Senior SRE IC.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword | Meaning                                                                                         |
| ------------------------ | ----------------------------------------------------------------------------------------------- |
| aCLI                     | Acropolis CLI; command-line tool used to manage AHV/Nutanix objects.                            |
| ADS                      | Acropolis Dynamic Scheduling; Nutanix feature that helps address compute/storage hotspots.      |
| AHV                      | Acropolis Hypervisor; Nutanix’s native hypervisor.                                              |
| AOS                      | Acropolis Operating System; Nutanix software layer providing core platform services.            |
| Blast radius             | Scope of impact across VMs, hosts, clusters, networks, or applications.                         |
| Bond                     | Logical grouping of physical NICs for redundancy/performance.                                   |
| Bottleneck               | The constrained resource causing degraded performance.                                          |
| Bridge call              | Live incident call with customer and technical teams.                                           |
| CPU Ready                | Time a VM waits for physical CPU scheduling.                                                    |
| CVM                      | Controller VM; Nutanix VM providing storage and platform services on each node.                 |
| East-west traffic        | Network traffic between internal systems or VMs.                                                |
| Guest OS                 | Operating system running inside the VM.                                                         |
| HCI                      | Hyperconverged Infrastructure; compute, storage, and virtualization integrated in one platform. |
| Hot-add / Hot-plug       | Adding CPU or memory to a running VM.                                                           |
| I/O                      | Input/output operations, usually storage or network activity.                                   |
| IOPS                     | Input/output operations per second; storage performance metric.                                 |
| iowait                   | Time the OS waits for storage I/O to complete.                                                  |
| Jira                     | Issue and case tracking tool commonly used in operations.                                       |
| KPI                      | Key Performance Indicator; operational performance metric.                                      |
| Latency                  | Time taken for an operation to complete.                                                        |
| MTTR                     | Mean Time To Resolve or Recover; key incident/support metric.                                   |
| MTU                      | Maximum Transmission Unit; maximum network packet size.                                         |
| Noisy neighbor           | Workload consuming shared resources and affecting other workloads.                              |
| North-south traffic      | Network traffic entering or leaving the data center/application environment.                    |
| NUMA                     | Non-Uniform Memory Access; CPU/memory locality architecture.                                    |
| Prism Central            | Nutanix centralized management interface across clusters.                                       |
| Prism Element            | Nutanix management interface for a single cluster.                                              |
| RCA                      | Root Cause Analysis; post-incident explanation of cause and prevention.                         |
| RDMA                     | Remote Direct Memory Access; high-performance network memory access technology.                 |
| Replication              | Copying data to another location for resilience or disaster recovery.                           |
| SLA                      | Service Level Agreement; contractual or operational service target.                             |
| Snapshot                 | Point-in-time copy of VM or data state.                                                         |
| Storage container        | Logical Nutanix storage construct used to present storage to workloads.                         |
| Throughput               | Amount of data transferred per unit of time.                                                    |
| Triage                   | Initial structured assessment of impact, scope, and likely cause.                               |
| vCPU                     | Virtual CPU assigned to a VM.                                                                   |
| vDisk                    | Virtual disk attached to a VM.                                                                  |
| VirtIO                   | Paravirtualized drivers used to improve VM disk/network performance.                            |
| VLAN                     | Virtual LAN; logical network segmentation.                                                      |
| VM                       | Virtual Machine.                                                                                |
| vNIC                     | Virtual network interface card attached to a VM.                                                |
| vNUMA                    | Virtual NUMA topology exposed to large VMs.                                                     |
| VMware                   | Common enterprise virtualization platform; relevant in migration/comparison scenarios.          |
| Workaround               | Temporary method to reduce impact without fully fixing root cause.                              |

[1]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Alert-Reference-vpc_2023_4%3Amul-alerts-user-created-metrics-r.html&utm_source=chatgpt.com "Prism pc.2023.4 - Alert Metrics - Nutanix"
[2]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_1%3Awc-performance-management-wc-c.html&utm_source=chatgpt.com "Prism 6.1 - Performance Monitoring - Nutanix"
[3]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2024_2%3Amul-vm-create-acropolis-pc-t.html&utm_source=chatgpt.com "Prism pc.2024.2 - Creating a VM through Prism Central (AHV)"
[4]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2029-AHV%3Anutanix-ahv-cpu-configuration.html&utm_source=chatgpt.com "Nutanix AHV CPU Configuration"
[5]: https://portal.nutanix.com/docs/AHV-Admin-Guide-v10_3%3Aahv-vm-memory-and-cpu-configuration-c.html?utm_source=chatgpt.com "Virtual Machine Memory and CPU Hot-Plug Configurations"
[6]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2038-AHV%3Aahv-memory-improvements-for-performance.html&utm_source=chatgpt.com "AHV Memory Improvements for Performance - portal.nutanix.com"
[7]: https://portal.nutanix.com/page/documents/solutions/details?targetId=BP-2071-AHV-Networking%3ABP-2071-AHV-Networking&utm_source=chatgpt.com "Nutanix AHV Networking Best Practices"
[8]: https://portal.nutanix.com/page/documents/details?targetId=AHV-Admin-Guide-v5_20%3Avmm-requirements-virtio-r.html&utm_source=chatgpt.com "AHV 5.20 - AHV Administration Guide - portal.nutanix.com"
