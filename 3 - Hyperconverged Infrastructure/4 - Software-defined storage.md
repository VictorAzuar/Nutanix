# Software-defined storage

## 1. Short definition

**Software-defined storage (SDS)** is a storage architecture where storage intelligence is moved from dedicated hardware arrays into software. Instead of relying on a traditional SAN/NAS appliance to provide capacity, performance, replication, snapshots, and resilience, SDS uses software to pool physical disks across servers and present them as logical storage resources.

In Nutanix terms, the key concept is the **Distributed Storage Fabric (DSF)**, which is part of **Nutanix AOS**. Nutanix describes DSF as the software-defined storage layer that aggregates local drives from every node in the cluster and presents them as a single resilient storage pool to the hypervisor. ([nutanix.com][1])

---

## 2. Clear explanation

Traditional infrastructure usually has three separate layers:

1. **Compute**: servers running workloads.
2. **Storage**: external SAN/NAS arrays.
3. **Network**: storage and data networks connecting everything.

In this model, storage is usually centralized. If a VM needs data, it often reads or writes through the network to a storage array. Performance, capacity, redundancy, and troubleshooting are strongly tied to the external storage system.

With **software-defined storage**, the storage layer is abstracted and controlled by software. In an HCI platform like Nutanix:

* Each node contributes local SSDs/HDDs/NVMe drives.
* A software layer pools those drives across the cluster.
* The cluster presents logical storage to workloads.
* Data is distributed and replicated across nodes.
* Storage services such as snapshots, compression, deduplication, tiering, and rebuilds are handled by software.

In Nutanix, this is tightly integrated with **AOS**, **AHV / ESXi / Hyper-V**, and **Prism**. A Nutanix storage container can be consumed differently depending on the hypervisor: transparently by AHV, as an NFS datastore for ESXi, or as an SMB share for Hyper-V. ([portal.nutanix.com][2])

A simple way to explain it in an interview:

> “Software-defined storage separates storage services from dedicated storage hardware. In Nutanix, AOS uses the Distributed Storage Fabric to pool local disks across cluster nodes and expose resilient shared storage to the hypervisor. The result is scale-out storage where capacity and performance grow with the cluster, and where resilience, data placement, rebuilds, and storage services are controlled by software.”

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role at Nutanix, SDS matters because it is one of the core foundations of the product.

Nutanix is not just “servers plus disks.” Its value proposition depends on the software layer that makes local disks behave like a resilient enterprise storage platform. Nutanix positions HCI as a way to replace legacy infrastructure made of separate servers, storage networks, and storage arrays with a distributed infrastructure platform driven by intelligent software. ([nutanix.com][3])

As a support manager, you do not need to debug every internal storage algorithm like a senior storage engineer, but you do need to understand:

* Why storage issues can affect VM performance, cluster health, customer SLAs, and business continuity.
* How to guide escalations involving capacity, latency, rebuilds, node failures, disk failures, replication, snapshots, or hypervisor integration.
* How to communicate clearly with customers who may come from a traditional SAN/NAS background.
* How to coordinate between support engineers, SREs, engineering, account teams, and customers during high-severity incidents.

Your positioning should be:

> “I understand SDS well enough to lead escalations, ask the right technical questions, separate symptoms from root causes, and communicate impact and recovery paths to enterprise customers, while relying on deep specialists for low-level product internals.”

That is exactly the right posture for a **technical escalation manager**, not a Senior SRE IC.

---

## 4. Key concepts

### Software abstraction

SDS abstracts storage from the physical disks. The customer does not manage each disk as an isolated device. Instead, the software creates logical constructs such as storage pools, containers, datastores, volumes, or object buckets.

In Nutanix, **storage containers** are logical units exposed to the hypervisor. Prism documentation describes storage containers as accessible transparently for AHV, as NFS datastores for ESXi, and as SMB shares for Hyper-V. ([portal.nutanix.com][2])

### Scale-out architecture

In traditional storage, scaling often means upgrading or adding storage arrays. In Nutanix, storage scales by adding nodes or drives, depending on the architecture and requirements.

This is important because:

* Capacity increases as the cluster grows.
* Performance can also increase because more nodes contribute resources.
* There is no single monolithic storage controller in the traditional sense.
* Failures are handled in a distributed way.

Interview phrase:

> “The key operational difference is that in a scale-out SDS model, storage is not a single external box. It is a distributed service running across the cluster.”

### Data locality

Nutanix emphasizes serving VM storage I/O locally where possible. Official Nutanix performance documentation states that AOS storage provides storage resources to VMs locally on the same host, allowing each local storage controller to handle I/O requests for VMs running on that node. ([portal.nutanix.com][4])

This matters for performance because local I/O can reduce unnecessary network hops.

Interview phrase:

> “Data locality is one of the reasons HCI can deliver strong VM performance. The platform tries to serve I/O close to where the VM is running, while still maintaining distributed resilience.”

### Resilience and replication

SDS must protect data against disk, node, software, and sometimes site failures. Nutanix documentation states that the Distributed Storage Fabric manages storage resources to preserve data and system integrity if there is node, disk, application, or hypervisor software failure in a cluster. ([portal.nutanix.com][5])

You should know the basic idea of:

* Replication factor.
* Fault tolerance.
* Rebuild operations.
* Cluster health.
* Impact of degraded redundancy.
* Failure domains.

You do not need to go extremely deep unless asked, but you must be able to discuss impact and escalation priority.

### Storage efficiency

SDS platforms usually include features such as:

* Compression.
* Deduplication.
* Erasure coding.
* Thin provisioning.
* Snapshots.
* Clones.
* Tiering.

Nutanix documentation mentions distributed storage capabilities such as data tiering, disk balancing, data locality, and advanced data services. ([portal.nutanix.com][6])

Support angle:

> “Storage efficiency features are valuable, but in an escalation I would always check whether they are contributing to latency, capacity pressure, rebuild duration, or unexpected behavior.”

### Management through Prism

For Nutanix, **Prism** is the management plane used to monitor and operate clusters. In the context of SDS, Prism helps with:

* Cluster health.
* Storage capacity.
* Alerts.
* Container configuration.
* Data resiliency status.
* Performance metrics.
* Upgrade readiness.
* Operational visibility.

For an interview, you should not just say “Prism is the UI.” Say:

> “Prism is important because it gives support teams and customers a single operational view of compute, storage, networking, alerts, and capacity. In an escalation, that shared visibility is essential.”

---

## 5. How it appears in a real escalation

A realistic escalation may not be reported as “software-defined storage is broken.” It will usually appear as one of these symptoms:

### Scenario A: VM latency after a node or disk failure

Customer says:

> “Our critical database VMs are slow after a disk failed. Latency increased and the business is impacted.”

What may be happening:

* A disk or node failure triggered rebuild activity.
* The cluster is maintaining data availability but performance is degraded.
* There may be capacity pressure.
* The workload may be sensitive to write latency.
* There may be network congestion affecting distributed storage traffic.
* A second failure risk may exist if redundancy is degraded.

Manager-level response:

> “I would first stabilize the incident: confirm business impact, affected workloads, current redundancy state, alerts, and whether the cluster is in rebuild or degraded mode. Then I would make sure the right storage/SRE expertise is engaged, align communication cadence with the customer, and avoid risky actions until we understand cluster health.”

### Scenario B: Capacity threshold breach

Customer says:

> “Prism is showing storage capacity warnings. Can we ignore them until next week?”

Risk:

* SDS platforms need free capacity for normal operations, rebuilds, snapshots, and data movement.
* Running too close to capacity can create performance and availability risk.
* Snapshot growth or replication may be consuming unexpected space.

Support-manager response:

> “I would treat capacity warnings as operational risk, not just housekeeping. I would ask for current utilization, growth rate, snapshot usage, recent workload changes, and whether there is enough headroom for failure recovery.”

### Scenario C: After upgrade, storage latency increases

Customer says:

> “After an AOS upgrade, VM latency increased.”

Possible areas:

* Upgrade state or post-upgrade tasks.
* CVM/resource pressure.
* Firmware/driver compatibility.
* Hypervisor integration.
* Network changes.
* Storage balancing/rebuild.
* Workload pattern change coinciding with upgrade.

Your role:

* Ensure timeline reconstruction.
* Separate correlation from causation.
* Drive evidence collection.
* Coordinate support, SRE, engineering if needed.
* Communicate knowns, unknowns, risks, and next steps.

### Scenario D: Customer compares Nutanix with SAN

Customer says:

> “In our old SAN we had dedicated controllers. Why is storage now running on the same nodes as compute?”

Good answer:

> “That is a core architectural difference. In HCI, storage services are distributed across the cluster instead of concentrated in a dedicated array. The advantage is simpler scaling, data locality, and integrated management. The operational model is different, so we need to monitor cluster health, node resources, network paths, capacity, and rebuild status as one system.”

---

## 6. Triage questions I should ask

Use these in interviews and real escalations.

### Impact and scope

* Which workloads or VMs are affected?
* Is the issue affecting all VMs, one host, one container, one cluster, or one site?
* Is this performance degradation, data unavailability, failed operation, or alert noise?
* What is the business impact?
* Is there an SLA breach or customer-facing outage?

### Timeline

* When did it start?
* Was there a recent upgrade, node addition, disk replacement, migration, failover, snapshot job, backup job, or network change?
* Is the issue constant or intermittent?
* Did it coincide with peak workload?

### Cluster health

* Are there active alerts in Prism?
* Is the cluster degraded?
* Are there disk, node, CVM, or network alerts?
* Is data resiliency healthy?
* Is there an ongoing rebuild or rebalancing process?

### Capacity

* What is current storage utilization?
* Is there enough free space for normal operation and failure recovery?
* Are snapshots, clones, backups, or replication consuming unexpected capacity?
* Has growth rate changed recently?

### Performance

* Is the issue read latency, write latency, IOPS, throughput, or CPU wait?
* Is latency visible at the VM, hypervisor, CVM, or storage layer?
* Are only specific workloads affected, such as databases or VDI?
* Is there a noisy neighbor workload?

### Network

* Any packet loss, congestion, MTU issue, VLAN change, switch issue, or NIC errors?
* Are storage-related paths impacted?
* Did anything change in the physical or virtual network?

### Operational risk

* Is there risk of a second failure?
* Are backups/snapshots available?
* Is the customer asking to perform an action that could increase risk?
* Do we need engineering or SRE escalation?

---

## 7. Likely interview questions

### Conceptual questions

1. What is software-defined storage?
2. How is SDS different from traditional SAN/NAS?
3. How does SDS relate to hyperconverged infrastructure?
4. What are the benefits and risks of SDS?
5. Why is data locality important?
6. What is scale-out storage?
7. What happens when a disk or node fails in an SDS platform?
8. What metrics would you monitor in an SDS environment?
9. How would you explain SDS to a customer used to 3-tier architecture?
10. What are the operational challenges of SDS?

### Nutanix-oriented questions

1. What role does AOS play in Nutanix storage?
2. What is the Distributed Storage Fabric?
3. What is Prism’s role during storage troubleshooting?
4. How do AHV, ESXi, and Hyper-V consume Nutanix storage?
5. What kind of storage issues could trigger a high-severity escalation?
6. How would you coordinate a storage-related escalation between support, SRE, and engineering?
7. What would you check first if a customer reports VM latency on Nutanix?
8. How would you handle a capacity-risk escalation?
9. How would you communicate degraded redundancy to an enterprise customer?
10. How would you avoid making a storage incident worse?

### Leadership / manager questions

1. How do you lead a technical escalation when you are not the deepest technical expert?
2. How do you keep the customer confident during a complex storage incident?
3. How do you balance speed of recovery with risk control?
4. How do you ensure proper handoff between regions in a worldwide support model?
5. How do you coach engineers after a major escalation?
6. How do you use KPIs like MTTR, backlog, SLA, and escalation rate in support operations?
7. How do you decide when to involve engineering?
8. How do you manage communication during a Sev1?
9. How do you prevent repeated escalations for the same issue type?
10. How do you translate technical findings into executive-level communication?

---

## 8. Model answers in English

### Question: What is software-defined storage?

**Model answer:**

> Software-defined storage is an architecture where storage services are implemented and managed by software rather than being tightly coupled to a dedicated storage array. The software abstracts physical disks, pools capacity, provides resilience, and exposes logical storage to workloads.
>
> In a Nutanix context, this is mainly delivered by AOS and the Distributed Storage Fabric, which aggregates local storage from cluster nodes and presents it as resilient shared storage to the hypervisor. From a support perspective, the important point is that storage behavior depends on the health of the whole distributed system: nodes, disks, CVMs, network, capacity, and workload patterns.

### Question: How is SDS different from traditional 3-tier storage?

**Model answer:**

> In a traditional 3-tier architecture, compute, storage, and networking are separate layers. Storage is usually provided by an external SAN or NAS array, and servers access it over the network.
>
> In SDS, especially in HCI, storage is distributed across the same nodes that provide compute. The storage services are controlled by software, so the cluster can pool local disks, replicate data, and scale by adding nodes.
>
> Operationally, this changes troubleshooting. Instead of only checking the SAN, I need to look at the full platform: VM, hypervisor, storage software, node health, network, capacity, and cluster state.

### Question: Why does SDS matter in Nutanix?

**Model answer:**

> SDS is central to Nutanix because Nutanix is not just a virtualization platform. Its value comes from integrating compute, storage, virtualization, management, and resilience into a software-driven platform.
>
> The Distributed Storage Fabric is fundamental because it allows local disks across nodes to behave like enterprise shared storage. For a worldwide support manager, understanding this is critical because many escalations will involve performance, capacity, resiliency, rebuilds, or customer concerns about how the platform protects data.

### Question: How would you troubleshoot a storage latency escalation?

**Model answer:**

> I would start by defining impact and scope: which workloads are affected, when it started, whether it is read or write latency, and whether the impact is cluster-wide or isolated.
>
> Then I would check recent changes: upgrades, node failures, disk replacements, network changes, backup jobs, snapshot growth, or workload spikes.
>
> In parallel, I would have the team review Prism alerts, cluster health, data resiliency, capacity headroom, CVM/resource utilization, and whether rebuild or rebalancing activity is ongoing.
>
> As a manager, my role is to keep the technical investigation structured, make sure the right specialists are involved, control customer communication, and avoid risky recovery steps until we understand the state of the cluster.

### Question: What are the risks of SDS?

**Model answer:**

> SDS reduces dependency on proprietary storage arrays, but it also means storage health depends on the distributed platform. Risks include capacity exhaustion, network bottlenecks, noisy neighbor workloads, rebuild impact after failures, misconfiguration, and operational misunderstanding by teams used to SAN-based architectures.
>
> From a support perspective, the mitigation is good monitoring, clear runbooks, proper capacity planning, strong change management, and disciplined escalation handling.

### Question: How would you explain Nutanix storage to a non-storage customer?

**Model answer:**

> I would explain that Nutanix uses the disks inside each cluster node and combines them through software into a resilient storage layer. Instead of having a separate storage array, the cluster itself provides storage services. The system distributes and protects data across nodes, while Prism gives visibility into health, capacity, and performance.
>
> The operational benefit is simpler scaling and integrated management. The support responsibility is to monitor the full stack because compute, storage, network, and workload behavior are closely connected.

### Question: As a manager, how technical do you need to be?

**Model answer:**

> I need enough technical depth to ask the right questions, understand failure modes, challenge assumptions, and communicate clearly with both engineers and customers. I do not need to replace a senior SRE or storage specialist, but I do need to understand the architecture well enough to lead the escalation.
>
> In practice, that means understanding concepts like distributed storage, data locality, capacity thresholds, rebuilds, latency, replication, and the relationship between AOS, Prism, hypervisors, and customer workloads.

---

## 9. Connection with my experience

Your Harmonic background maps very well to this topic if you frame it correctly.

You already know:

* 24/7 enterprise support.
* Incident management.
* SLA and MTTR ownership.
* Escalation coordination.
* Monitoring and observability.
* Cloud operations.
* Customer communication.
* Team leadership.
* Cross-functional coordination.
* Jira / Confluence / Salesforce workflows.

The gap is not “support leadership.” The gap is **Nutanix-specific infrastructure vocabulary and architecture**.

You can connect your experience like this:

> “In my current role, I manage enterprise support and operations for production SaaS/cloud environments. I am used to correlating symptoms across infrastructure, applications, monitoring, customer impact, and operational timelines. With Nutanix, the technical domain is different, but the escalation discipline is very similar: define impact, stabilize, collect evidence, identify recent changes, coordinate specialists, communicate clearly, and drive MTTR down without increasing risk.”

You can also say:

> “My strength is not claiming to be the deepest storage engineer in the room. My strength is leading complex incidents where multiple layers interact: infrastructure, monitoring, customer SLAs, escalation paths, and team execution. That is directly relevant to SDS because storage issues in HCI are distributed-system incidents, not isolated box failures.”

This is a strong positioning statement.

---

## 10. Minimum I need to memorize

Memorize these points until you can say them naturally.

1. **SDS means storage services are controlled by software, not tied to a dedicated hardware array.**

2. **In Nutanix, the key SDS component is AOS Distributed Storage Fabric.**

3. **DSF pools local disks from Nutanix nodes and presents them as resilient shared storage to the hypervisor.**

4. **HCI combines compute and storage in the same cluster, unlike traditional 3-tier architecture.**

5. **Scale-out means adding nodes can add capacity and potentially performance.**

6. **Data locality means serving I/O close to the VM when possible, reducing unnecessary network hops.**

7. **Storage issues often appear as VM latency, capacity alerts, degraded redundancy, rebuild activity, snapshot growth, or backup impact.**

8. **A support manager should check impact, scope, timeline, recent changes, Prism alerts, cluster health, capacity, latency, and network.**

9. **The manager’s role is to structure the escalation, involve the right experts, manage customer communication, and avoid risky actions.**

10. **You do not need to be a senior storage IC, but you must understand enough to lead the incident intelligently.**

---

## 11. Advanced / optional level

These are useful, but not mandatory for your first interview unless the SRE panel goes deep.

### Advanced Nutanix concepts

* Replication Factor / resiliency policies.
* Erasure coding.
* Stargate, Cassandra, Curator, Zeus, Medusa — Nutanix internal services.
* CVM architecture.
* Storage I/O path in AHV.
* Rebuild mechanics.
* Disk balancing.
* ILM / tiering behavior.
* Snapshot and clone internals.
* Metro availability / disaster recovery.
* Nutanix Volumes, Files, Objects.
* RDMA / advanced network optimization.
* AHV vs ESXi storage presentation differences.
* Performance troubleshooting using specific Nutanix commands.

### How to handle advanced questions

Do not fake depth. Use this structure:

> “I understand the operational concept, but I would rely on a senior SRE or product specialist for the low-level internals. At manager level, what I would focus on is understanding the impact, confirming the platform state, ensuring the right data is collected, and coordinating the escalation safely.”

That answer is credible.

---

## 12. Final checklist

Before the interview, you should be able to answer these verbally:

* Can I define SDS in under 30 seconds?
* Can I explain SDS vs SAN/NAS?
* Can I explain why SDS is central to Nutanix?
* Can I explain what AOS and DSF do at a high level?
* Can I explain why data locality matters?
* Can I describe a storage latency escalation?
* Can I list the first triage questions I would ask?
* Can I explain capacity risk in an HCI/SDS cluster?
* Can I connect SDS to my experience in SaaS/cloud operations?
* Can I position myself as a technical escalation manager rather than a senior storage IC?
* Can I explain this to a manager, an SRE, and an enterprise customer using different levels of detail?

A strong final interview soundbite:

> “For me, software-defined storage is not just a storage concept; it is an operational model. In Nutanix, storage is part of a distributed platform, so support needs to think in terms of cluster health, workload impact, capacity, resiliency, network, and customer communication. My role as a support manager would be to lead that investigation with structure, involve the right technical depth, and keep the customer confident while we drive toward recovery and root cause.”


| Keyword / Acronym             | Meaning                                                                                               |
| ----------------------------- | ----------------------------------------------------------------------------------------------------- |
| AHV                           | Acropolis Hypervisor; Nutanix’s native hypervisor.                                                    |
| AOS                           | Acropolis Operating System; Nutanix’s core software platform.                                         |
| API                           | Application Programming Interface; interface for automation and integrations.                         |
| Availability                  | Ability of a system to remain operational and accessible.                                             |
| Backup                        | Copy of data kept for recovery purposes.                                                              |
| Block Storage                 | Storage model where data is stored in fixed-size blocks.                                              |
| Business Impact               | Practical effect of an issue on customer operations, users, revenue, or SLA.                          |
| Capacity                      | Total usable storage available in a system.                                                           |
| Capacity Headroom             | Free capacity reserved for growth, rebuilds, snapshots, and recovery.                                 |
| Cluster                       | Group of nodes working together as one system.                                                        |
| Cloud Operations              | Operation of cloud or cloud-like infrastructure.                                                      |
| Compression                   | Technique that reduces physical storage consumption.                                                  |
| Container / Storage Container | Nutanix logical storage construct presented to workloads or hypervisors.                              |
| CVM                           | Controller Virtual Machine; VM on each Nutanix node providing storage and cluster services.           |
| Data Locality                 | Serving data from the same node where the VM runs whenever possible.                                  |
| Data Protection               | Mechanisms that prevent data loss, such as replication, snapshots, and backups.                       |
| Deduplication                 | Removal of duplicate data blocks to save capacity.                                                    |
| Degraded State                | System is still running but has reduced redundancy, resilience, or performance.                       |
| Disaster Recovery / DR        | Ability to recover systems and data after a major failure or site outage.                             |
| Distributed Storage           | Storage spread across multiple nodes instead of centralized in one array.                             |
| DSF                           | Distributed Storage Fabric; Nutanix SDS layer that pools local disks across nodes.                    |
| Erasure Coding                | Data protection method using fragments and parity to reduce capacity overhead.                        |
| Escalation                    | Raising an issue to higher support, SRE, engineering, or management.                                  |
| ESXi                          | VMware enterprise hypervisor.                                                                         |
| Enterprise Support            | Support for business-critical customer environments with SLAs and escalations.                        |
| Failover                      | Moving workloads or services to a backup system after failure.                                        |
| Failure Domain                | Infrastructure area that can fail independently, such as disk, node, rack, or site.                   |
| Fault Tolerance               | Ability to continue operating after component failure.                                                |
| Firmware                      | Low-level software embedded in hardware devices.                                                      |
| Grafana                       | Dashboard and monitoring tool for metrics and alerts.                                                 |
| HA                            | High Availability; ability to continue operating despite failures.                                    |
| HCI                           | Hyperconverged Infrastructure; combines compute, storage, networking, virtualization, and management. |
| Hypervisor                    | Software that runs and manages virtual machines.                                                      |
| IC                            | Individual Contributor; hands-on technical role without people-management responsibility.             |
| Incident Management           | Process for detecting, triaging, resolving, and reviewing incidents.                                  |
| Infrastructure as a Platform  | Infrastructure delivered through integrated software, APIs, and automation.                           |
| IOPS                          | Input/Output Operations Per Second; storage performance metric.                                       |
| Jira                          | Tool for tracking incidents, tasks, bugs, and escalations.                                            |
| K8s / Kubernetes              | Container orchestration platform.                                                                     |
| Kibana                        | Log analysis and visualization tool.                                                                  |
| KPI                           | Key Performance Indicator; operational performance metric.                                            |
| Latency                       | Time taken for an operation to complete.                                                              |
| Linux                         | Operating system widely used in infrastructure and cloud environments.                                |
| Logical Storage               | Abstracted storage presented to applications or hypervisors.                                          |
| Metro Availability            | High-availability model across two sites or clusters.                                                 |
| Monitoring                    | Collection and analysis of metrics, logs, events, and alerts.                                         |
| MTTR                          | Mean Time to Resolution / Recovery.                                                                   |
| NAS                           | Network Attached Storage; file-level storage accessed over a network.                                 |
| NFS                           | Network File System; protocol commonly used for file storage and ESXi datastores.                     |
| Node                          | Physical server participating in a cluster.                                                           |
| Noisy Neighbor                | Workload consuming excessive shared resources and affecting others.                                   |
| Nutanix                       | Enterprise cloud and HCI platform vendor.                                                             |
| Object Storage                | Storage model where data is stored as objects with metadata.                                          |
| Observability                 | Ability to understand system state through metrics, logs, traces, and events.                         |
| Performance Degradation       | System remains available but performs worse than expected.                                            |
| Physical Disk                 | Actual storage hardware such as HDD, SSD, or NVMe.                                                    |
| Prism                         | Nutanix management and monitoring interface.                                                          |
| Prism Central                 | Centralized management plane for multiple Nutanix clusters.                                           |
| Prism Element                 | Management interface for a single Nutanix cluster.                                                    |
| QoS                           | Quality of Service; controls or prioritizes resource usage.                                           |
| RCA                           | Root Cause Analysis; process of identifying the underlying cause of an issue.                         |
| Rebalance / Rebalancing       | Redistribution of data across nodes or disks.                                                         |
| Rebuild                       | Reconstruction of data redundancy after disk or node failure.                                         |
| Replication                   | Maintaining additional copies of data for resilience.                                                 |
| Replication Factor / RF       | Number of data copies kept for protection.                                                            |
| Resilience                    | Ability to absorb failures and recover quickly.                                                       |
| RFO                           | Reason for Outage; explanation of outage cause.                                                       |
| Runbook                       | Documented operational procedure for troubleshooting or recovery.                                     |
| SaaS                          | Software as a Service.                                                                                |
| SAN                           | Storage Area Network; dedicated network providing block-level storage.                                |
| Scale-out                     | Increasing capacity/performance by adding nodes.                                                      |
| SDS                           | Software-defined Storage; storage services delivered by software abstraction.                         |
| Sev1                          | Severity 1; critical incident with major production or business impact.                               |
| SLA                           | Service Level Agreement; expected service commitment.                                                 |
| SMB                           | Server Message Block; file-sharing protocol common in Windows environments.                           |
| Snapshot                      | Point-in-time copy of data.                                                                           |
| SRE                           | Site Reliability Engineering.                                                                         |
| Storage Array                 | Dedicated external storage system used in traditional architectures.                                  |
| Storage Pool                  | Logical aggregation of physical storage resources.                                                    |
| Thin Provisioning             | Allocating logical capacity without reserving all physical capacity upfront.                          |
| Throughput                    | Amount of data transferred per unit of time.                                                          |
| Tiering                       | Placement or movement of data across media types such as NVMe, SSD, and HDD.                          |
| Triage                        | Initial structured assessment of impact, scope, urgency, and next action.                             |
| VDI                           | Virtual Desktop Infrastructure.                                                                       |
| Virtualization                | Abstraction of physical compute into virtual machines or virtual environments.                        |
| VM                            | Virtual Machine.                                                                                      |
| VMware                        | Virtualization and cloud infrastructure vendor.                                                       |
| Volume                        | Logical block storage unit presented to a host or application.                                        |
| Worldwide Support             | Global support organization operating across regions and time zones.                                  |
| Write Latency                 | Time required to complete a write operation.                                                          |
| X-Play                        | Nutanix Prism Central automation functionality based on triggers and actions.                         |



[1]: https://www.nutanix.com/products/nutanix-cloud-infrastructure/distributed-storage?utm_source=chatgpt.com "A Modern Storage Fabric - Nutanix"
[2]: https://portal.nutanix.com/docs/Prism-Central-Guide-vpc_7_5%3Amul-container-create-ahv-pc-t.html?utm_source=chatgpt.com "Prism pc.7.5 - Creating a Storage Container"
[3]: https://www.nutanix.com/hyperconverged-infrastructure?utm_source=chatgpt.com "What is Hyperconverged Infrastructure (HCI) - FAQs | Nutanix"
[4]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2096-Nutanix-Core-Performance%3Aaos-storage.html&utm_source=chatgpt.com "AOS Storage - Nutanix"
[5]: https://portal.nutanix.com/page/documents/details?targetId=vSphere-Admin6-AOS-v6_7%3Avsp-cluster-introduction-vsphere-c.html&utm_source=chatgpt.com "AOS 6.7 - Overview - Nutanix"
[6]: https://portal.nutanix.com/page/documents/solutions/details?targetId=TN-2131-Oil-and-Gas-Workloads%3Anutanix-distributed-storage-capabilities.html&utm_source=chatgpt.com "Nutanix Distributed Storage Capabilities"
