# AOS / Storage — CVM - Controller VM

## 1. Short definition

A **CVM, or Controller VM**, is the Nutanix virtual machine that runs on every node in a Nutanix cluster and provides the core storage-control plane and data services for that node. It is the local storage controller that allows Nutanix to turn local disks across multiple servers into a distributed storage system consumed by VMs and applications. Nutanix describes the architecture as a storage controller VM running on every node to form a highly distributed, shared-nothing infrastructure. ([portal.nutanix.com][1])

In interview language:

> “The CVM is the Nutanix storage brain running on each node. It handles local I/O, participates in the distributed storage fabric, manages metadata and data placement, and keeps storage services available even if a node, disk, hypervisor, or CVM component has an issue.”

---

## 2. Clear explanation

In traditional infrastructure, storage may be provided by an external SAN or NAS array. In Nutanix HCI, storage is distributed across the same nodes that provide compute. Each node has local disks, a hypervisor, and a **Controller VM**. The CVMs across the cluster cooperate to expose a shared storage layer to the hypervisor and guest VMs.

The CVM is not “just another VM.” It is a privileged infrastructure VM responsible for critical Nutanix services: user I/O handling, data placement, metadata management, data integrity, availability, garbage collection, compression, deduplication, erasure coding, snapshots, replication, and storage performance optimization. ([portal.nutanix.com][2])

A simplified view:

```text
Guest VM
  ↓
Hypervisor: AHV / ESXi / Hyper-V
  ↓
Local CVM
  ↓
Nutanix Distributed Storage Fabric
  ↓
Local + remote disks across the cluster
```

The most important concept is **data locality**. In a healthy Nutanix environment, VM I/O is normally served through the local CVM on the same host, which helps reduce latency and avoid unnecessary network traffic. The cluster still remains distributed: if local access is unavailable, the system can redirect traffic through other CVMs over the storage network.

On AHV, the CVM runs as a VM, and disks are presented to it using PCI passthrough, allowing the disk controller and attached devices to be passed directly to the CVM while bypassing the hypervisor storage stack. ([NutanixBible.com][3]) On ESXi, Nutanix similarly uses VMDirectPath I/O to pass the storage controller to the CVM. ([NutanixBible.com][4])

For you as a support manager, the key point is not to become the deepest CVM engineer in the room. The key is to understand that when a customer reports latency, storage unavailability, degraded redundancy, replication delay, or VM I/O problems, the CVM is often central to the escalation path.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, the CVM matters because it sits directly at the intersection of:

* customer-facing availability;
* storage performance;
* cluster health;
* incident severity;
* escalation management;
* cross-functional troubleshooting between support, SRE, engineering, networking, hypervisor, and customer teams.

If a CVM is down, overloaded, isolated, misconfigured, or unable to communicate properly with other CVMs, the customer may experience VM latency, degraded resilience, alerts in Prism, failed upgrades, replication issues, or service impact.

Nutanix documentation explicitly states that the cluster monitors CVM health. If the **Stargate** process fails to respond repeatedly, another CVM redirects the storage path for that host to another Stargate, and reads/writes occur over the 10 GbE network until the missing Stargate returns. ([portal.nutanix.com][5])

That is a very interview-relevant point. It shows you understand both the resilience mechanism and the operational trade-off:

> “The system can tolerate a CVM or Stargate issue by redirecting I/O, but that changes the storage path and may increase network dependency and latency. From a support-management perspective, I would treat that as a degraded state requiring controlled triage, customer communication, and restoration of the local CVM path.”

This is exactly the type of answer that positions you as a **technical escalation manager**: you understand what is happening technically, but you frame the response around impact, risk, communication, containment, and recovery.

---

## 4. Key concepts

### CVM per node

Each Nutanix node runs a CVM. The CVMs work together to aggregate local storage resources into a cluster-wide storage pool consumed by workloads. ([portal.nutanix.com][6])

Interview phrasing:

> “Nutanix does not rely on a pair of external storage controllers. It distributes storage control across CVMs running on every node.”

### Distributed Storage Fabric

The **Distributed Storage Fabric**, often shortened to **DSF**, is the Nutanix storage layer that manages data across nodes and disks. It preserves data and system integrity during failures such as node, disk, hypervisor, or application issues. ([portal.nutanix.com][6])

### Stargate

**Stargate** is a critical Nutanix data-path service associated with I/O. If Stargate on a CVM fails to respond, another CVM can redirect storage traffic. ([portal.nutanix.com][5])

For an interview, you do not need to deep-dive into every internal service, but you should know that Stargate is one of the core services to check when discussing CVM data-path health.

### Local I/O path

In normal operation, a VM’s I/O is handled by the local CVM on the same host. This is a major part of Nutanix’s performance model: local I/O where possible, distributed coordination when needed.

### Remote I/O path during failure

If the local CVM or Stargate is unavailable, another CVM can service the I/O path over the network. This preserves availability but may increase latency or put more pressure on the storage network. Nutanix documentation specifically mentions reads and writes occurring over the 10 GbE network during this condition. ([portal.nutanix.com][5])

### CVM networking

CVM-to-CVM traffic is critical. Storage traffic, replication, metadata operations, and cluster communication depend on healthy networking. Nutanix also documents scenarios involving CVM external management IPs and internal backplane IPs, especially when network segmentation is enabled. ([portal.nutanix.com][7])

### CVM resources

CVM CPU and memory are not arbitrary VM settings. Nutanix recommends keeping CPU settings as configured by Foundation unless Nutanix Support recommends changes. For memory, Nutanix notes that most workloads use less than 32 GB RAM per CVM, while mission-critical workloads with large working sets may require more. ([portal.nutanix.com][8])

Interview phrasing:

> “I would avoid ad-hoc CVM resource changes. CVM sizing and changes should follow Nutanix guidance or Support recommendations because the CVM is part of the storage control and data path.”

### Prism visibility

Prism Element and Prism Central are the main operational interfaces where support teams observe cluster health, alerts, capacity, performance, and infrastructure state. CVM problems are typically visible through alerts, service health, node health, storage latency, and cluster status.

---

## 5. How it appears in a real escalation

### Scenario 1 — Customer reports high VM latency

A customer opens a Sev1 or Sev2 case:

> “Several production VMs are slow. Database response time increased from 5 ms to 80 ms. The customer says nothing changed.”

Possible CVM-related angle:

* local CVM overloaded;
* Stargate issue;
* CVM service degraded;
* storage network congestion;
* one CVM down, causing remote I/O;
* disk or node issue affecting data path;
* recently changed CVM memory, networking, or cluster configuration;
* noisy workload causing contention.

Support-manager response:

> “I would first confirm business impact, affected workloads, scope, timeline, and whether this is a performance degradation or availability incident. In parallel, I would have the technical team check Prism alerts, CVM service health, Stargate status, node health, storage latency, network errors, and recent changes. If a CVM or Stargate issue is causing remote I/O, I would communicate that the cluster is protecting availability but running in a degraded path, and I would coordinate restoration while managing customer expectations.”

### Scenario 2 — CVM down after maintenance

A customer performed maintenance and one CVM did not come back properly.

Symptoms:

* Prism alert about CVM down;
* one host using remote storage path;
* cluster still up but degraded;
* increased latency on VMs from that host;
* possible upgrade or maintenance blocked.

Key management actions:

* confirm whether customer production is impacted;
* check whether redundancy is still healthy;
* ensure no other maintenance actions continue until cluster health is restored;
* coordinate with SRE / L2 / engineering if services do not recover;
* provide clear customer updates: current state, risk, next action, ETA for next update.

### Scenario 3 — Network segmentation or IP change issue

Customer changes CVM IPs or modifies networking. Afterward, replication or cluster communication breaks.

Nutanix documentation includes procedures for changing external CVM management IPs and internal CVM backplane IPs, and warns that remote sites for data protection may need updates before replication resumes. ([portal.nutanix.com][7])

Escalation framing:

> “This is not just an IP-change issue. CVM networking is part of cluster storage communication and data protection. I would treat it as a change-related incident, validate the exact change window, identify which CVM interfaces were modified, assess replication and cluster-health impact, and make sure no further changes are made without a recovery plan.”

### Scenario 4 — Failed upgrade because CVM services are unhealthy

AOS upgrades rely on controlled rolling operations. If CVM health is already degraded, the upgrade may fail or should be paused.

Manager-level response:

> “Before continuing the upgrade, I would prioritize restoring a healthy baseline. I would not allow the team to continue disruptive steps while the cluster is already degraded unless Nutanix Support or engineering explicitly validates the path.”

---

## 6. Triage questions I should ask

### Impact and scope

1. Which applications or VMs are affected?
2. Is this a full outage, intermittent issue, or performance degradation?
3. How many nodes, hosts, clusters, or sites are involved?
4. Is the impact limited to VMs on one host?
5. Are all workloads affected or only storage-intensive workloads?
6. When did the issue start?
7. Is there a clear change window before the symptoms appeared?

### CVM and cluster health

8. Are all CVMs powered on and reachable?
9. Are all CVM services up?
10. Is Stargate healthy on all nodes?
11. Is the cluster reporting degraded redundancy?
12. Are there Prism alerts related to CVM, disk, node, storage, or network?
13. Is any node in maintenance mode?
14. Was any CVM recently restarted, migrated, resized, or modified?

### Storage path and performance

15. Is I/O being served locally or remotely?
16. Are affected VMs concentrated on a specific host?
17. Are storage latency, IOPS, or throughput abnormal?
18. Are there hot disks, full disks, or capacity pressure?
19. Is there active data rebalancing, healing, snapshot, replication, or garbage collection activity?

### Network

20. Are CVMs able to communicate with each other?
21. Any packet loss, MTU mismatch, VLAN issue, switch change, or NIC error?
22. Was network segmentation enabled or modified?
23. Were CVM management or backplane IPs changed?
24. Is storage replication traffic sharing congested uplinks?

### Change and escalation control

25. What changed recently: AOS, AHV, ESXi, firmware, networking, firewall, storage policy, snapshot schedule, backup, monitoring, or workload placement?
26. Is there an active upgrade, expansion, failover, migration, or DR test?
27. Has the customer already restarted any CVMs or hosts?
28. Is the customer asking for immediate recovery, root cause, or both?
29. Do we need engineering involvement?
30. What is the next customer update commitment?

---

## 7. Likely interview questions

### Manager / leadership interview

1. **What is a Nutanix CVM and why is it important?**
2. **How would you manage an escalation where a CVM is down but the cluster is still serving I/O?**
3. **How do you balance technical troubleshooting with customer communication during a storage incident?**
4. **What KPIs would you track for support cases involving storage latency or CVM issues?**
5. **How would you coach an engineer who jumps directly into CLI commands without first clarifying impact?**
6. **How would you handle a customer who wants to restart multiple CVMs during an incident?**
7. **How do you decide when to escalate to engineering?**
8. **How would you communicate degraded redundancy to an enterprise customer?**

### Technical / SRE interview

9. **Explain the role of the CVM in Nutanix HCI.**
10. **What happens if a CVM or Stargate service fails?**
11. **How does Nutanix differ from a traditional SAN-based architecture?**
12. **Why is CVM networking important?**
13. **What would you check if VMs on one host show high storage latency?**
14. **Why should CVM CPU and memory not be changed casually?**
15. **How does data locality relate to CVM behavior?**
16. **How would you triage storage performance degradation in a Nutanix cluster?**

### Advanced / panel interview

17. **A customer reports latency after a network change. How would you lead the bridge?**
18. **A CVM is unreachable during an AOS upgrade. What is your escalation approach?**
19. **How would you explain remote I/O to a non-technical customer executive?**
20. **How do you distinguish between a CVM issue, hypervisor issue, storage issue, and network issue?**

---

## 8. Model answers in English

### Q1. What is a CVM in Nutanix?

> “A CVM, or Controller VM, is the Nutanix virtual machine that runs on every node and provides the core storage services for the cluster. Instead of relying on external storage controllers, Nutanix distributes the storage function across CVMs. The CVM handles local I/O, participates in metadata and data placement, and contributes to the distributed storage fabric. From a support perspective, CVM health is critical because it directly affects VM storage latency, availability, data protection, and cluster resilience.”

### Q2. Why is the CVM important in an enterprise support escalation?

> “Because the CVM is part of the storage data path. If a customer reports VM latency, degraded redundancy, failed replication, or cluster-health alerts, CVM health is one of the first areas I would want the technical team to validate. As a support manager, I do not need to personally run every deep diagnostic, but I need to understand the architecture well enough to ask the right questions, prioritize risk, communicate clearly, and make sure the escalation follows a safe path.”

### Q3. What happens if a CVM or Stargate fails?

> “Nutanix is designed to tolerate failures. If a Stargate process on a CVM becomes unresponsive, another CVM can redirect the storage path, so reads and writes can continue over the storage network until the local service returns. That protects availability, but it may create a degraded state with increased network dependency and possible latency. In an escalation, I would communicate both sides: the platform is resilient and serving I/O, but we need to restore the normal local path and confirm redundancy before making further changes.”

### Q4. How would you manage a customer escalation involving CVM failure?

> “I would first separate impact management from technical diagnosis. On impact, I would confirm affected applications, severity, business impact, and whether there is data unavailability or performance degradation. On diagnosis, I would ask the team to validate Prism alerts, CVM reachability, service health, Stargate status, node health, storage latency, recent changes, and network health. I would also control the bridge: no unnecessary restarts, no parallel risky changes, clear ownership, and regular customer updates. If the issue suggests a product defect, upgrade failure, or repeated service crash, I would escalate to engineering with logs, timeline, symptoms, and business impact.”

### Q5. How would you explain CVM to a non-technical customer executive?

> “I would say: in Nutanix, each server has a built-in storage controller running as a special VM. These controllers work together to provide shared storage. One of those controllers is currently unhealthy, but the cluster has redirected traffic through other controllers to maintain service. That means the environment is protected, but running in a degraded mode, so our priority is to restore the local controller and reduce the risk of further impact.”

### Q6. What would you check if only VMs on one host have high latency?

> “If the impact is concentrated on one host, I would suspect something local to that host: the local CVM, Stargate, disk path, NIC, hypervisor, or a noisy workload. I would check whether the local CVM is up, whether services are healthy, whether I/O has been redirected remotely, whether there are network errors, and whether recent changes affected that host. I would also compare latency and workload metrics across other nodes to determine whether this is node-local or cluster-wide.”

### Q7. How does your current experience transfer to Nutanix support?

> “In my current role, I lead a 24/7 enterprise support operation where incident management, SLA, MTTR, escalation control, monitoring, and customer communication are central. The technology stack is different, but the operational discipline is very similar. For Nutanix, I would apply the same structure: assess impact, stabilize service, identify the fault domain, coordinate technical owners, communicate clearly, and drive root cause after recovery. My learning focus is to map that operational experience into Nutanix-specific components like CVM, AOS, AHV, Prism, Stargate, and the distributed storage fabric.”

### Q8. Why should you avoid changing CVM resources casually?

> “Because the CVM is not a normal workload VM. It is part of the infrastructure data path. CPU, memory, and configuration settings are provisioned according to Nutanix platform guidance, and changes should follow Nutanix recommendations or Support direction. An unplanned change could affect storage services, stability, or supportability.”

### Q9. How do you decide when to escalate to engineering?

> “I would escalate to engineering when the issue exceeds standard troubleshooting or appears to involve a product defect, repeated service crash, data-risk condition, upgrade failure, unexplained performance regression, or customer-critical impact that requires code-level or architecture-level expertise. I would make sure the escalation contains a clean timeline, impact statement, logs, cluster health state, recent changes, reproduction pattern if available, and actions already taken.”

---

## 9. Connection with my experience

Your current background maps strongly to this topic if you position it correctly.

At Harmonic, you already manage:

* 24/7 support operations;
* incident bridges;
* SLA ownership;
* MTTR reduction;
* customer escalations;
* monitoring and observability;
* production-impact triage;
* coaching and coordination;
* cloud and SaaS operations.

The Nutanix-specific part is the infrastructure vocabulary: **CVM, AOS, AHV, Prism, Stargate, DSF, storage latency, data locality, replication, redundancy factor, cluster health**.

A strong positioning statement:

> “My strength is leading enterprise support teams through complex incidents where multiple technical domains overlap. In SaaS and cloud operations, I have managed incidents involving application, infrastructure, monitoring, networking, and customer communication. For Nutanix, I am building the domain depth around AOS and CVM so I can lead escalations with enough technical fluency to challenge assumptions, ask precise triage questions, and support engineers without trying to replace the Senior SRE.”

This is the exact balance you want: technical fluency without pretending to be the deepest individual contributor.

---

## 10. Minimum I need to memorize

Memorize these points cold:

1. **CVM = Controller VM.**
2. A CVM runs on **every Nutanix node**.
3. The CVM provides core **storage services** for that node and participates in the cluster-wide distributed storage layer.
4. Nutanix uses CVMs to create a **shared-nothing distributed storage architecture** rather than relying on external storage controllers.
5. The CVM handles or coordinates **I/O, data placement, metadata, data integrity, snapshots, replication, compression, deduplication, erasure coding, and garbage collection**. ([portal.nutanix.com][2])
6. **Stargate** is a critical data-path service related to storage I/O.
7. If Stargate/CVM fails, another CVM can redirect the storage path, keeping I/O running over the network, but the system is degraded. ([portal.nutanix.com][5])
8. CVM networking is critical because CVMs need to communicate for storage and cluster operations.
9. Do not casually change CVM CPU/memory; follow Nutanix guidance or Support recommendation. ([portal.nutanix.com][8])
10. In an escalation, focus on **impact, scope, recent changes, CVM health, service health, storage latency, network health, and customer communication**.

One verbal answer to memorize:

> “The CVM is the Nutanix storage controller running as a VM on each node. It serves local VM I/O, participates in the distributed storage fabric, and coordinates data placement, metadata, resilience, and storage efficiency. If a CVM or Stargate service fails, Nutanix can redirect I/O through another CVM, which protects availability but may create a degraded state with higher latency or network dependency. As a support manager, I would focus on impact, containment, CVM and service health, network path, recent changes, and clear customer communication.”

---

## 11. Advanced / optional level

You can leave these for deeper study unless the SRE interview goes more technical:

### Internal AOS services

Examples include Stargate and other Nutanix internal services. Know Stargate first. You do not need to memorize every daemon unless you are interviewing for a hands-on Senior SRE role.

### Cassandra / metadata architecture

Nutanix uses distributed metadata concepts internally. For this role, you mainly need to understand that metadata health matters for storage operations, not necessarily the internal schema.

### Curator, Medusa, Genesis, Zookeeper

These Nutanix internal components may appear in deep troubleshooting conversations. Useful to recognize, but not essential for your first pass.

### Detailed CLI commands

You should know that engineers may use SSH to CVMs and commands like `cluster status`, `ncli`, `acli`, service checks, and log collection. But your role should emphasize escalation coordination, safe process, and interpretation rather than memorizing every command.

### Storage optimization internals

Compression, deduplication, erasure coding, garbage collection, and rebalancing are important, but you can study them separately after CVM basics.

### Hypervisor-specific CVM integration

AHV uses PCI passthrough for devices to the CVM; ESXi uses VMDirectPath I/O. This is useful for technical interviews, but you do not need to over-explain it unless asked. ([NutanixBible.com][3])

---

## 12. Final checklist

You are ready for this topic if you can answer these without notes:

* Can I define CVM in one sentence?
* Can I explain why Nutanix has one CVM per node?
* Can I explain why the CVM is central to storage I/O?
* Can I explain what happens if a CVM or Stargate has a problem?
* Can I explain why remote I/O preserves availability but may affect performance?
* Can I connect CVM issues to customer symptoms like latency, alerts, degraded redundancy, or failed upgrades?
* Can I ask good triage questions about impact, scope, recent changes, Prism alerts, CVM health, service health, and networking?
* Can I explain this to both an SRE and a customer executive?
* Can I avoid sounding like I am pretending to be a Senior SRE IC?
* Can I position myself as a support leader who understands enough architecture to manage escalations safely?

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword   | Meaning                                                                                                          |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| aCLI                       | Acropolis CLI; Nutanix command-line tool used mainly for AHV and Acropolis management.                           |
| AHV                        | Acropolis Hypervisor; Nutanix’s native hypervisor based on Linux KVM.                                            |
| AOS                        | Acropolis Operating System; core Nutanix software providing storage, cluster, virtualization, and data services. |
| Backplane IP               | Internal CVM network IP used for cluster/storage communication in segmented networking designs.                  |
| Bridge                     | Incident conference or war room used to coordinate troubleshooting and communication.                            |
| Cassandra                  | Distributed database technology used in many distributed systems; relevant to metadata concepts in Nutanix.      |
| Cluster                    | Group of Nutanix nodes working together as a single infrastructure platform.                                     |
| Cluster health             | Overall operational state of nodes, CVMs, services, disks, storage, and networking.                              |
| Compression                | Storage efficiency technique that reduces data size.                                                             |
| Controller VM              | Full name for CVM; the Nutanix VM that provides storage and cluster services on each node.                       |
| CVM                        | Controller VM; key Nutanix infrastructure VM running on every node.                                              |
| Data locality              | Serving VM I/O from the local node/CVM whenever possible to reduce latency.                                      |
| Data path                  | The path followed by read/write I/O between VM, hypervisor, CVM, and storage.                                    |
| Data placement             | Logic deciding where data and replicas are stored across the cluster.                                            |
| Data protection            | Mechanisms such as snapshots, replication, and redundancy used to protect workloads.                             |
| Deduplication              | Storage efficiency technique that avoids storing duplicate data blocks.                                          |
| Degraded state             | Condition where the system is still running but with reduced redundancy, performance, or resilience.             |
| Distributed Storage Fabric | Nutanix distributed storage layer that aggregates disks across nodes.                                            |
| DSF                        | Distributed Storage Fabric; Nutanix scale-out storage architecture.                                              |
| Erasure coding             | Storage efficiency/resilience method that reduces capacity overhead compared with full replicas.                 |
| ESXi                       | VMware hypervisor commonly supported in Nutanix environments.                                                    |
| External CVM IP            | CVM management-facing IP address.                                                                                |
| Fault domain               | Area where a failure is isolated, such as node, disk, network, rack, or site.                                    |
| Foundation                 | Nutanix deployment/provisioning tool that configures nodes and CVMs during installation.                         |
| Garbage collection         | Process that reclaims unused storage space and supports storage efficiency.                                      |
| HCI                        | Hyperconverged Infrastructure; architecture combining compute, storage, and virtualization in one platform.      |
| Hyper-V                    | Microsoft hypervisor supported in some Nutanix environments.                                                     |
| Hypervisor                 | Software layer that runs virtual machines, such as AHV, ESXi, or Hyper-V.                                        |
| I/O                        | Input/output; read and write operations from workloads to storage.                                               |
| IOPS                       | Input/output operations per second; key storage performance metric.                                              |
| Jira                       | Ticketing/project management tool often used for support, engineering, and incident workflows.                   |
| KPI                        | Key Performance Indicator; metric used to manage support performance.                                            |
| Latency                    | Delay experienced by an I/O operation or application request.                                                    |
| Local I/O                  | I/O served through the CVM on the same node as the VM.                                                           |
| Metadata                   | Information describing where data lives, how it is protected, and how it is managed.                             |
| MTTR                       | Mean Time To Repair/Restore; average time to recover from incidents.                                             |
| Network segmentation       | Separating management, storage, replication, or backplane traffic into different networks.                       |
| nCLI                       | Nutanix CLI used for cluster and platform management tasks.                                                      |
| Node                       | Physical Nutanix server participating in the cluster.                                                            |
| PCI passthrough            | Direct assignment of PCI devices, such as storage controllers, to a VM.                                          |
| Prism                      | Nutanix management interface for monitoring, administration, alerts, and operations.                             |
| Prism Central              | Centralized Nutanix management plane for multiple clusters and advanced services.                                |
| Prism Element              | Cluster-level Nutanix management interface.                                                                      |
| RCA                        | Root Cause Analysis; post-incident explanation of cause and corrective actions.                                  |
| Redundancy factor          | Level of data replica protection in a Nutanix cluster.                                                           |
| Remote I/O                 | I/O served through another CVM over the network when local path is unavailable or unsuitable.                    |
| Replication                | Copying data to another cluster or site for disaster recovery or data protection.                                |
| Resilience                 | Ability of the platform to continue operating during component failures.                                         |
| SLA                        | Service Level Agreement; contractual or operational target for service response or restoration.                  |
| Snapshot                   | Point-in-time copy used for recovery, backup, or data protection workflows.                                      |
| Stargate                   | Nutanix data-path service responsible for storage I/O handling.                                                  |
| Storage controller         | Component responsible for managing storage access and data services. In Nutanix, this role is performed by CVMs. |
| Storage latency            | Time taken to complete storage read/write operations.                                                            |
| Storage pool               | Aggregated group of physical storage devices used by the Nutanix cluster.                                        |
| Throughput                 | Amount of data transferred over time, usually measured in MB/s or GB/s.                                          |
| VMDirectPath I/O           | VMware mechanism allowing direct device passthrough to a VM, used for CVM access to storage devices on ESXi.     |
| vSwitch                    | Virtual switch used by hypervisors for VM and infrastructure networking.                                         |
| Workload                   | Application, VM, database, or service running on the infrastructure.                                             |
| Zookeeper                  | Distributed coordination technology; useful keyword for advanced Nutanix internal-service discussions.           |

[1]: https://portal.nutanix.com/page/documents/details?targetId=vSphere-Admin6-AOS-v7_0%3AvSphere-Admin6-AOS-v7_0&utm_source=chatgpt.com "AOS 7.0 - vSphere Administration Guide for AOS - portal.nutanix.com"
[2]: https://portal.nutanix.com/page/documents/details?targetId=Advanced-Admin-AOS%3Aapp-nutanix-cloud-infra-cvm-field-specifications-c.html&utm_source=chatgpt.com "AOS 7.5 - Controller VM (CVM) Specifications - Nutanix"
[3]: https://www.nutanixbible.com/pdf/5a-book-of-ahv-architecture.pdf?utm_source=chatgpt.com "AHV - AHV Architecture - NutanixBible.com"
[4]: https://www.nutanixbible.com/pdf/6a-book-of-vsphere-architecture.pdf?utm_source=chatgpt.com "vSphere - vSphere Architecture - NutanixBible.com"
[5]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_3%3Aarc-controller-vm-failure-c.html?utm_source=chatgpt.com "Prism 7.3 - Controller VM Failure - portal.nutanix.com"
[6]: https://portal.nutanix.com/page/documents/details?targetId=vSphere-Admin6-AOS-v6_0%3Avsp-cluster-introduction-vsphere-c.html&utm_source=chatgpt.com "AOS 6.0 - Overview - Nutanix"
[7]: https://portal.nutanix.com/page/documents/details?targetId=Advanced-Admin-AOS%3Aip-cvm-ip-address-reconfigure-t.html&utm_source=chatgpt.com "AOS 7.5 - Changing the Controller VM IP Addresses (CLI Script) - Nutanix"
[8]: https://portal.nutanix.com/docs/vSphere-Admin6-AOS-v7_5%3Avsp-cluster-configuration-vsphere-r.html?utm_source=chatgpt.com "AOS 7.5 - Nutanix Software Configuration"
