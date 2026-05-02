# Traditional 3-tier architecture

## 1. Short definition

**Traditional 3-tier infrastructure architecture** is the classic enterprise datacenter model where infrastructure is separated into three main layers:

1. **Compute** — physical servers and hypervisors running workloads.
2. **Network** — LAN, management network, storage network, switching, routing, VLANs, and connectivity.
3. **Storage** — centralized storage arrays providing shared disks, LUNs, volumes, snapshots, replication, and data protection.

In this context, **3-tier architecture does not mean web / application / database tiers**. That is application architecture. Here we are discussing **infrastructure architecture**.

A strong interview definition:

> “Traditional 3-tier infrastructure separates compute, network, and storage into independent layers. Compute runs the workloads, network provides connectivity, and storage provides persistent shared data services. This model is mature and widely deployed in enterprises, but it can be complex to operate and troubleshoot because incidents often cross multiple layers and vendor boundaries.”

---

## 2. Clear explanation

In a traditional enterprise datacenter, the compute layer usually consists of physical servers running a hypervisor such as VMware ESXi, Hyper-V, or another virtualization platform. The VMs run on those hosts, but their persistent disks are often stored externally on shared storage.

The network layer connects everything together. It includes:

* production network traffic;
* management traffic;
* vMotion or live migration traffic;
* backup traffic;
* storage traffic;
* VLANs;
* routing;
* switching;
* firewalls;
* SAN or IP storage connectivity.

The storage layer is normally a centralized storage platform, such as a SAN or NAS array. It provides shared datastores, volumes, LUNs, snapshots, replication, deduplication, compression, caching, and data protection.

A simplified VM I/O path in a traditional 3-tier architecture may look like this:

> **VM → hypervisor → compute host NIC/HBA → network/SAN fabric → storage array → disks/flash → back again**

This is why the architecture is powerful but also complex. When an application becomes slow, the root cause may be in the VM, hypervisor, host CPU, memory, physical server, network, SAN fabric, storage path, storage controller, datastore, or storage array.

Nutanix positions HCI as a way to simplify this model by bringing compute, storage, networking, and virtualization into a more integrated platform. ([Nutanix][2])

---

## 3. Why it matters for Nutanix / Worldwide Support

This matters for a **Manager, Worldwide Support** at Nutanix because many enterprise customers are either:

* migrating from traditional VMware + SAN/NAS infrastructure to Nutanix;
* running Nutanix alongside legacy 3-tier infrastructure;
* comparing Nutanix HCI against their existing compute / network / storage model;
* escalating performance or availability issues involving multiple vendors;
* expecting Nutanix Support to understand both HCI and traditional enterprise infrastructure.

Nutanix describes HCI as combining servers and storage into a distributed infrastructure platform designed to replace legacy infrastructure made of separate servers, storage networks, and storage arrays. ([Nutanix][3])

For a support manager, this is important because you need to understand the customer’s old world and the Nutanix model.

You do not need to be the deepest storage engineer in the room, but you must be able to lead the escalation intelligently:

> “Is the problem in compute, network, storage, virtualization, or the application layer?”

That question is central to enterprise support.

---

## 4. Key concepts

### Compute layer

The **compute layer** provides the processing resources for workloads.

It includes:

* physical servers;
* CPUs;
* memory;
* hypervisors;
* VM placement;
* host clusters;
* HA / DRS / live migration;
* hardware firmware and drivers.

Typical compute-related issues:

* CPU contention;
* memory pressure;
* host failure;
* hypervisor instability;
* firmware mismatch;
* VM placement imbalance;
* noisy neighbor workloads.

Interview phrase:

> “The compute layer is where the workload executes. In a virtualized environment, that means physical hosts running a hypervisor and providing CPU and memory to VMs.”

---

### Network layer

The **network layer** provides connectivity between workloads, hosts, management systems, users, storage, and external services.

It includes:

* LAN switching;
* routing;
* VLANs;
* firewalls;
* load balancers;
* management networks;
* storage networks;
* Fibre Channel SAN;
* iSCSI;
* NFS;
* MTU / jumbo frames;
* multipathing;
* link aggregation;
* DNS / NTP dependencies.

This is where my previous explanation needed correction.

The right framing is:

> “Storage network is not a separate main tier. It is a specialized and very important part of the network tier in traditional enterprise infrastructure.”

In many VMware + SAN/NAS environments, storage traffic is so critical that people talk specifically about the **storage network**. But in the architectural model, it still belongs under **Network**.

Typical network-related issues:

* packet loss;
* latency;
* link flapping;
* VLAN misconfiguration;
* MTU mismatch;
* SAN zoning issues;
* iSCSI path instability;
* NFS connectivity problems;
* firewall or routing changes;
* DNS or NTP issues.

Interview phrase:

> “The network layer is not only user-facing connectivity. In enterprise infrastructure, it also carries management, migration, backup, replication, and storage traffic. That is why network issues can appear as application issues, storage issues, or cluster issues.”

---

### Storage layer

The **storage layer** provides persistent data services.

It includes:

* SAN arrays;
* NAS arrays;
* LUNs;
* volumes;
* datastores;
* storage pools;
* RAID groups;
* snapshots;
* replication;
* deduplication;
* compression;
* caching;
* backup integration;
* disaster recovery.

Typical storage-related issues:

* high datastore latency;
* controller failover;
* disk failure or rebuild;
* overloaded storage pools;
* snapshot chain impact;
* replication lag;
* thin provisioning exhaustion;
* queue depth saturation;
* volume or LUN misconfiguration.

Interview phrase:

> “The storage layer is responsible for persistent data. In traditional 3-tier, this is often centralized shared storage, which gives strong enterprise capabilities but can also become a performance bottleneck or a major failure domain.”

---

### How HCI changes the model

Traditional 3-tier:

> **Compute + Network + Centralized Storage**

Nutanix HCI:

> **Distributed compute and storage, managed through an integrated software-defined platform**

Nutanix’s public positioning is that HCI brings together compute, storage, networking, and virtualization into a single platform, reducing legacy infrastructure silos. ([Nutanix][2])

Important nuance:

Nutanix does **not** eliminate networking. Networking remains critical. But HCI reduces the dependency on a separate external SAN/NAS architecture by distributing storage services across the cluster.

Good interview wording:

> “HCI does not remove the need for networking or storage expertise. It changes the architecture. Instead of compute hosts depending on a separate centralized storage array, Nutanix distributes storage services across the cluster and manages them through software.”

---

## 5. How it appears in a real escalation

### Scenario 1: “VMs are slow after a maintenance window”

Possible layers involved:

| Layer   | Possible cause                                                                    |
| ------- | --------------------------------------------------------------------------------- |
| Compute | CPU contention, memory pressure, host firmware issue, hypervisor problem          |
| Network | VLAN change, packet loss, MTU mismatch, storage path issue, routing problem       |
| Storage | high array latency, controller failover, snapshot issue, datastore capacity issue |

As support manager, you should not jump directly to storage or compute.

A good escalation response:

> “First, I would define the scope and impact: which applications, VMs, hosts, clusters, and time window are affected. Then I would split the investigation across compute, network, and storage. I would ask for hypervisor metrics, host health, network errors, storage latency, recent changes, and customer impact. The goal is to isolate the failing layer without creating vendor finger-pointing.”

---

### Scenario 2: “Customer reports intermittent application timeouts”

Possible root causes:

* overloaded application server;
* VM CPU ready;
* memory ballooning or swapping;
* packet loss;
* firewall timeout;
* DNS issue;
* storage latency;
* backup job causing I/O pressure;
* database latency;
* SAN path failover.

Support manager angle:

> “I would avoid assuming that an application timeout is only an application problem. In 3-tier infrastructure, symptoms are often indirect. Network packet loss can look like application instability. Storage latency can look like database slowness. Compute contention can look like poor application performance.”

---

### Scenario 3: “Datastore latency is high”

This is where the corrected model matters.

You should not say:

> “The three tiers are compute, storage network, and storage.”

You should say:

> “The three tiers are compute, network, and storage. For this specific escalation, the storage-related part of the network becomes critical because storage traffic may traverse Fibre Channel, iSCSI, or NFS.”

Triage split:

| Area    | What to check                                                                   |
| ------- | ------------------------------------------------------------------------------- |
| Compute | host CPU, memory, hypervisor logs, VM placement                                 |
| Network | storage paths, SAN switch errors, packet drops, MTU, zoning, iSCSI/NFS sessions |
| Storage | controller load, disk latency, pool capacity, snapshots, replication, rebuilds  |

---

### Scenario 4: “Customer is migrating from VMware + SAN to Nutanix”

Customer concern:

> “Our SAN is reliable. Why should we move to HCI?”

Strong answer:

> “Traditional 3-tier is mature and can be very reliable when designed and operated well. The tradeoff is operational complexity: separate compute, network, and storage teams, separate lifecycle management, separate scaling cycles, and more complex troubleshooting. Nutanix HCI changes that model by integrating compute and storage services into a distributed platform, which can simplify operations and scaling while still requiring disciplined design, monitoring, and support.”

---

## 6. Triage questions I should ask

### Business impact

1. What service or application is affected?
2. Is this production?
3. Is it a full outage or degradation?
4. How many users, customers, VMs, hosts, or clusters are affected?
5. Is there an SLA breach risk?
6. When did the issue start?
7. Was there a recent change, deployment, upgrade, migration, backup, or maintenance?

### Compute questions

8. Which hosts are affected?
9. Are CPU, memory, or hypervisor resources saturated?
10. Are there hardware alerts?
11. Are VMs concentrated on specific hosts?
12. Are there HA, DRS, vMotion, or live migration events?
13. Are hypervisor logs showing warnings or errors?

### Network questions

14. Is the issue isolated to a VLAN, subnet, rack, switch, or site?
15. Are there packet drops, CRC errors, link flaps, or interface errors?
16. Were there firewall, routing, VLAN, DNS, or MTU changes?
17. Is management traffic affected?
18. Is storage traffic affected?
19. Are iSCSI, NFS, or Fibre Channel paths healthy?
20. Are multipathing policies correct?

### Storage questions

21. Which datastore, volume, or LUN is affected?
22. Is storage latency high?
23. Are reads, writes, or both affected?
24. Is the storage array reporting controller, disk, or pool issues?
25. Are snapshots, backups, or replication jobs running?
26. Is the datastore close to full?
27. Is there a rebuild, failover, or maintenance operation in progress?

### Escalation management questions

28. Who owns each technical workstream?
29. What evidence do we have for each suspected layer?
30. What is the next customer update time?
31. Do we need Nutanix Engineering, VMware, network vendor, storage vendor, or cloud provider engagement?
32. What is the workaround path?
33. What is the recovery path?
34. What will be included in the RCA?

---

## 7. Likely interview questions

1. **Can you explain traditional 3-tier infrastructure architecture?**
2. **What are the three layers in traditional 3-tier infrastructure?**
3. **How is traditional 3-tier different from hyperconverged infrastructure?**
4. **Why did enterprises historically use traditional 3-tier architecture?**
5. **What are the operational challenges of traditional 3-tier?**
6. **How would you troubleshoot performance degradation in a 3-tier environment?**
7. **How would you isolate whether an issue is compute, network, or storage?**
8. **What role does the network layer play in storage performance?**
9. **What is the difference between production network and storage network?**
10. **How would you manage a multi-vendor escalation involving VMware, networking, and storage?**
11. **How does Nutanix simplify the traditional 3-tier model?**
12. **Does HCI eliminate the need for networking expertise?**
13. **What would you say to a customer who trusts traditional SAN more than HCI?**
14. **As a support manager, how deep do you need to go technically?**
15. **How do you avoid vendor finger-pointing in a complex infrastructure escalation?**

---

## 8. Model answers in English

### Q1. Can you explain traditional 3-tier infrastructure architecture?

> “Traditional 3-tier infrastructure separates compute, network, and storage. The compute layer provides servers and hypervisors to run workloads. The network layer provides connectivity between users, applications, hosts, management systems, and storage. The storage layer provides persistent shared storage through arrays, LUNs, volumes, snapshots, replication, and data protection. It is a mature enterprise model, but troubleshooting can be complex because incidents often cross multiple layers.”

---

### Q2. What are the three layers?

> “The three layers are compute, network, and storage. In enterprise virtualization, the network layer can include both normal LAN traffic and storage-related traffic such as Fibre Channel, iSCSI, or NFS. So I would not describe ‘storage network’ as a separate tier; I would describe it as a critical part of the network tier.”

---

### Q3. How is traditional 3-tier different from Nutanix HCI?

> “Traditional 3-tier separates compute, network, and centralized storage. Nutanix HCI uses a distributed software-defined model that brings compute and storage services closer together across cluster nodes and manages them through an integrated platform. Networking remains critical, but the dependency on separate external storage arrays and dedicated storage fabrics is reduced.”

Nutanix describes HCI as bringing together compute, storage, networking, and virtualization into one platform, and as replacing legacy infrastructure with separate servers, storage networks, and storage arrays. ([Nutanix][2])

---

### Q4. Why did enterprises use traditional 3-tier architecture?

> “Because it provided mature, reliable shared infrastructure for critical workloads. Centralized storage enabled capabilities like shared datastores, snapshots, replication, backup integration, high availability, and live migration. It also allowed specialized teams to manage compute, network, and storage independently. The downside is that this separation can create operational silos and make troubleshooting slower during incidents.”

---

### Q5. How would you troubleshoot high latency in a traditional 3-tier environment?

> “I would first confirm business impact and scope: which applications, VMs, hosts, clusters, datastores, and time window are affected. Then I would investigate the three layers in parallel. For compute, I would check CPU, memory, host health, and hypervisor logs. For network, I would check packet loss, link errors, VLANs, MTU, routing, and storage path health. For storage, I would check datastore latency, array health, controller load, snapshots, replication, capacity, and queue depth. As a manager, I would ensure clear ownership, evidence-based troubleshooting, and structured customer communication.”

---

### Q6. What is the role of the network layer in storage performance?

> “In traditional infrastructure, the network layer is not just user connectivity. It may also carry storage traffic through Fibre Channel, iSCSI, or NFS. If there is packet loss, path instability, zoning misconfiguration, MTU mismatch, or SAN congestion, the symptom may appear as storage latency or VM performance degradation. That is why storage performance incidents often require both storage and network investigation.”

---

### Q7. How would you manage a multi-vendor escalation?

> “I would avoid sequential vendor blaming. I would establish a single incident owner, define the impact and timeline, document recent changes, and create parallel workstreams for compute, network, storage, and application. Each team would provide evidence, not assumptions. I would keep a clear customer communication cadence, escalate vendors when evidence points to their domain, and drive toward service restoration first, then RCA.”

---

### Q8. Does HCI remove the need for networking knowledge?

> “No. HCI changes the architecture, but networking remains critical. Nutanix can simplify the traditional dependency on external storage arrays and dedicated storage fabrics, but cluster health, VM connectivity, replication, management, and hybrid cloud integrations still depend on good network design and troubleshooting.”

---

### Q9. What would you say to a customer who says traditional SAN is safer than HCI?

> “I would acknowledge that traditional SAN-based architecture is mature and can be highly reliable. Then I would move the conversation to requirements: availability, performance, operational complexity, scaling, recovery, supportability, and lifecycle management. HCI is not about removing engineering discipline; it is about using distributed software to simplify infrastructure operations and reduce some of the complexity of separate compute, network, and storage silos.”

---

### Q10. As a support manager, how deep do you need to go technically?

> “I need enough technical depth to ask the right questions, challenge weak assumptions, understand evidence, and communicate credibly with customers and SREs. I do not need to replace a senior storage or network engineer, but I must understand the architecture, the failure domains, and the escalation path. My role is to combine technical fluency with incident leadership, prioritization, SLA management, and customer communication.”

---

## 9. Connection with my experience

Your experience maps very well to this topic because the operating principles are the same as in SaaS/cloud operations.

| Your experience                  | Traditional 3-tier / Nutanix equivalent                          |
| -------------------------------- | ---------------------------------------------------------------- |
| 24/7 support leadership          | Enterprise infrastructure support leadership                     |
| Incident management              | Severity management and customer escalations                     |
| SLA / MTTR ownership             | Support case prioritization and restoration targets              |
| Grafana / Kibana monitoring      | Infrastructure, cluster, VM, network, and storage observability  |
| Cloud operations                 | Hybrid cloud and private cloud operations                        |
| Kubernetes / distributed systems | HCI distributed architecture thinking                            |
| Jira / Confluence / Salesforce   | Case tracking, knowledge base, RCA, customer communication       |
| Team coordination                | Cross-functional support between SRE, engineering, TAMs, vendors |
| Escalation management            | Multi-layer infrastructure escalation ownership                  |

Strong personal positioning:

> “My background is in leading 24/7 enterprise support and operations, so I am used to incidents where the symptom is not necessarily the root cause. Whether the environment is SaaS, cloud, Kubernetes, or traditional infrastructure, the management discipline is similar: define impact, isolate the failing domain, coordinate technical owners, communicate clearly, and drive restoration. For Nutanix, I am strengthening my knowledge of HCI, AOS, AHV, Prism, VMware, networking, and storage so I can lead escalations credibly as a technical support manager.”

---

## 10. Minimum I need to memorize

Memorize this definition:

> “Traditional 3-tier infrastructure separates compute, network, and storage. Compute runs the workloads, network connects systems and carries production, management, migration, backup, and storage traffic, and storage provides persistent shared data services. It is mature and reliable, but operationally complex because incidents can span multiple layers and vendors.”

Memorize this correction:

> “Storage network is not one of the three main tiers. It is a critical part of the network tier.”

Memorize this Nutanix contrast:

> “Nutanix HCI changes the traditional model by integrating compute and storage services into a distributed software-defined platform. It reduces some legacy infrastructure silos, but networking, monitoring, capacity planning, and disciplined operations remain critical.”

Memorize the basic triage structure:

> **Impact → Scope → Recent changes → Compute → Network → Storage → Workaround → Customer updates → RCA**

---

## 11. Advanced / optional level

You do not need to master these deeply before the interview, but you should recognize the terms.

### Advanced compute topics

* CPU ready;
* memory ballooning;
* NUMA;
* hypervisor scheduling;
* host firmware;
* driver compatibility;
* HA admission control;
* live migration failure;
* host isolation.

### Advanced network topics

* VLAN trunking;
* MTU mismatch;
* jumbo frames;
* LACP;
* packet drops;
* CRC errors;
* Fibre Channel zoning;
* iSCSI sessions;
* NFS mounts;
* multipathing;
* storage path failover;
* east-west vs north-south traffic;
* management plane vs data plane.

### Advanced storage topics

* LUN masking;
* VMFS;
* NFS datastore behavior;
* queue depth;
* SCSI reservations;
* APD — All Paths Down;
* PDL — Permanent Device Loss;
* storage controller failover;
* RAID rebuild impact;
* snapshots;
* replication;
* deduplication;
* compression;
* thin provisioning;
* write cache.

### Advanced Nutanix topics

* AOS;
* AHV;
* Prism Element;
* Prism Central;
* Controller VM;
* data locality;
* distributed storage fabric;
* replication factor;
* cluster services;
* NCC health checks;
* lifecycle management;
* Flow networking;
* disaster recovery;
* migration from VMware/SAN to Nutanix.

---

## 12. Final checklist

Before the interview, make sure you can explain:

* [ ] Traditional 3-tier infrastructure means **compute, network, storage**.
* [ ] It is different from **3-tier application architecture**: web, app, database.
* [ ] It is also different from **3-tier network architecture**: access, distribution, core.
* [ ] Compute means servers, hypervisors, CPU, memory, and workloads.
* [ ] Network means production, management, migration, backup, and storage connectivity.
* [ ] Storage means persistent data services, usually centralized SAN/NAS arrays.
* [ ] “Storage network” is part of the **Network** tier, not a separate main tier.
* [ ] Traditional 3-tier is mature but operationally complex.
* [ ] Incidents can cross compute, network, storage, virtualization, and application layers.
* [ ] Nutanix HCI simplifies parts of this model by integrating compute and storage through distributed software.
* [ ] HCI does not eliminate the need for strong networking knowledge.
* [ ] As a support manager, your value is technical fluency plus incident leadership.

Best final answer to practice:

> “Traditional 3-tier infrastructure separates compute, network, and storage. Compute provides the servers and hypervisors where workloads run. Network provides production, management, migration, backup, and storage connectivity. Storage provides persistent shared data services through SAN or NAS arrays. This architecture is mature and widely used, but it can be complex to troubleshoot because performance or availability issues may cross several layers and vendors. Nutanix HCI changes the model by integrating compute and storage services into a distributed software-defined platform, reducing some operational silos while still requiring strong networking, monitoring, capacity planning, and escalation discipline.”


# Annex I: Keywords & Acronyms

Alphabetically sorted.

| Acronym / Keyword           | Meaning                                                                                                                                                                                         |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **3-tier architecture**     | Traditional infrastructure model separating **Compute**, **Network**, and **Storage** into independent layers.                                                                                  |
| **AOS**                     | **Acropolis Operating System**. Nutanix’s core software platform that provides distributed storage, virtualization integration, cluster services, data protection, and management capabilities. |
| **APD**                     | **All Paths Down**. VMware condition where a host temporarily loses all paths to a storage device, but the device is expected to return.                                                        |
| **Application tier**        | In application architecture, the middle layer where business logic runs. Not the same as infrastructure 3-tier architecture.                                                                    |
| **Array**                   | A centralized storage system providing block or file storage to servers.                                                                                                                        |
| **Availability**            | The ability of a system or service to remain operational and accessible.                                                                                                                        |
| **Backup traffic**          | Network traffic generated by backup systems when copying VM, file, database, or application data.                                                                                               |
| **Blast radius**            | The scope of impact of an incident: how many users, services, VMs, hosts, clusters, or sites are affected.                                                                                      |
| **Block storage**           | Storage presented as raw blocks, commonly through SAN technologies such as Fibre Channel or iSCSI.                                                                                              |
| **Bottleneck**              | A constrained component limiting overall system performance.                                                                                                                                    |
| **Cache**                   | High-speed memory or flash used to accelerate read/write operations.                                                                                                                            |
| **Capacity planning**       | Forecasting compute, network, and storage resource needs to avoid saturation or service degradation.                                                                                            |
| **Centralized storage**     | Storage architecture where persistent data resides in dedicated external arrays rather than inside each compute host.                                                                           |
| **Cluster**                 | A group of servers working together to provide shared compute, storage, virtualization, or availability services.                                                                               |
| **Compute**                 | The infrastructure layer responsible for CPU, memory, physical servers, hypervisors, and workload execution.                                                                                    |
| **Controller**              | Component responsible for managing storage operations, I/O processing, caching, or data services.                                                                                               |
| **CPU**                     | **Central Processing Unit**. The processor resources used by hosts and VMs.                                                                                                                     |
| **CPU contention**          | A condition where multiple workloads compete for limited CPU resources.                                                                                                                         |
| **CPU Ready**               | VMware metric showing the time a VM is ready to run but waiting for physical CPU resources.                                                                                                     |
| **CRC errors**              | **Cyclic Redundancy Check errors**. Network or storage-link errors indicating data corruption or transmission problems.                                                                         |
| **CVM**                     | **Controller Virtual Machine**. In Nutanix, a VM running on each node that provides core storage and platform services.                                                                         |
| **Data plane**              | The part of the infrastructure that carries actual workload, storage, or application traffic.                                                                                                   |
| **Database tier**           | In application 3-tier architecture, the backend layer where databases store and retrieve application data.                                                                                      |
| **Datastore**               | A logical storage container used by hypervisors to store VM files and virtual disks.                                                                                                            |
| **Deduplication**           | Data reduction technique that removes duplicate data blocks to save storage capacity.                                                                                                           |
| **Degradation**             | A partial service impairment where the system is still running but performing below expected levels.                                                                                            |
| **Disk rebuild**            | Process of reconstructing data after a disk failure using redundancy mechanisms.                                                                                                                |
| **DNS**                     | **Domain Name System**. Service that translates names into IP addresses. DNS issues can cause application, management, or cluster connectivity problems.                                        |
| **DR**                      | **Disaster Recovery**. Processes and technologies used to restore services after a major failure.                                                                                               |
| **DRS**                     | **Distributed Resource Scheduler**. VMware feature that balances workloads across hosts in a cluster.                                                                                           |
| **East-west traffic**       | Traffic between systems inside the datacenter or cluster, such as VM-to-VM or node-to-node traffic.                                                                                             |
| **ESXi**                    | VMware’s bare-metal hypervisor used to run virtual machines on physical servers.                                                                                                                |
| **Fabric**                  | A network of switches and links, often used when referring to SAN or datacenter switching infrastructure.                                                                                       |
| **Failover**                | Automatic or manual transition from a failed component to a redundant component.                                                                                                                |
| **FC**                      | **Fibre Channel**. High-performance storage networking protocol commonly used in SAN environments.                                                                                              |
| **Firmware**                | Low-level software embedded in hardware components such as servers, NICs, HBAs, disks, or controllers.                                                                                          |
| **Firewall**                | Security device or software controlling traffic between networks or systems.                                                                                                                    |
| **Flow Networking**         | Nutanix networking and security services, including microsegmentation and virtual networking capabilities.                                                                                      |
| **Gray failure**            | A partial or intermittent failure that does not produce a clean “up/down” signal but causes instability or degradation.                                                                         |
| **HA**                      | **High Availability**. Capability that keeps services running or restarts workloads after failures.                                                                                             |
| **HBA**                     | **Host Bus Adapter**. Hardware adapter used by servers to connect to storage networks, commonly Fibre Channel.                                                                                  |
| **HCI**                     | **Hyperconverged Infrastructure**. Architecture that combines compute and storage into a distributed software-defined platform.                                                                 |
| **Host**                    | A physical server running a hypervisor and hosting VMs.                                                                                                                                         |
| **Host isolation**          | Condition where a host loses management or cluster connectivity and may trigger HA behavior.                                                                                                    |
| **Hyper-V**                 | Microsoft’s hypervisor for running virtual machines.                                                                                                                                            |
| **Hypervisor**              | Software layer that allows multiple VMs to run on one physical server. Examples: VMware ESXi, Microsoft Hyper-V, Nutanix AHV.                                                                   |
| **I/O**                     | **Input/Output**. Read and write operations between workloads and storage or network systems.                                                                                                   |
| **IOPS**                    | **Input/Output Operations Per Second**. A storage performance metric measuring the number of read/write operations per second.                                                                  |
| **IP**                      | **Internet Protocol**. Core addressing and routing protocol for network communication.                                                                                                          |
| **iSCSI**                   | **Internet Small Computer Systems Interface**. Storage protocol that carries SCSI commands over IP networks.                                                                                    |
| **ITSM**                    | **IT Service Management**. Processes and tools for managing IT services, incidents, changes, requests, and problems.                                                                            |
| **Jumbo frames**            | Ethernet frames with larger-than-standard MTU, often used in storage or high-throughput networks.                                                                                               |
| **LACP**                    | **Link Aggregation Control Protocol**. Protocol used to combine multiple network links into one logical connection.                                                                             |
| **LAN**                     | **Local Area Network**. Network connecting systems within a local site or datacenter.                                                                                                           |
| **Latency**                 | Time taken for an operation to complete. In infrastructure, often refers to network latency or storage read/write latency.                                                                      |
| **Lifecycle management**    | Process of managing upgrades, patches, firmware, drivers, compatibility, and maintenance over time.                                                                                             |
| **Link flap**               | A network link repeatedly going up and down, causing instability.                                                                                                                               |
| **Live migration**          | Moving a running VM from one host to another with minimal or no downtime. VMware calls this vMotion.                                                                                            |
| **LUN**                     | **Logical Unit Number**. A logical block storage device presented from a storage array to hosts.                                                                                                |
| **Management network**      | Network used for infrastructure management traffic, such as hypervisor, cluster, storage, and monitoring access.                                                                                |
| **Management plane**        | The part of the infrastructure responsible for control, configuration, monitoring, and administration.                                                                                          |
| **Memory ballooning**       | Hypervisor technique used to reclaim memory from VMs during memory pressure.                                                                                                                    |
| **Memory pressure**         | Condition where hosts or VMs do not have enough available RAM for workload demand.                                                                                                              |
| **Migration traffic**       | Network traffic generated when moving workloads, such as VM live migration or storage migration.                                                                                                |
| **MTTR**                    | **Mean Time To Resolution** or **Mean Time To Restore**. Metric measuring how long it takes to resolve or restore service after an incident.                                                    |
| **MTU**                     | **Maximum Transmission Unit**. Maximum packet size allowed on a network path. MTU mismatch can cause connectivity or performance issues.                                                        |
| **Multi-vendor escalation** | Incident involving multiple vendors, such as Nutanix, VMware, network vendors, storage vendors, or backup vendors.                                                                              |
| **Multipathing**            | Use of multiple storage paths between hosts and storage systems for redundancy and load balancing.                                                                                              |
| **NAS**                     | **Network Attached Storage**. File-based storage accessed over a network, commonly using NFS or SMB.                                                                                            |
| **NCC**                     | **Nutanix Cluster Check**. Nutanix health-check tool used to validate cluster health and detect known issues.                                                                                   |
| **NFS**                     | **Network File System**. File protocol commonly used to present shared datastores to hypervisors.                                                                                               |
| **NIC**                     | **Network Interface Card**. Hardware adapter that connects a server to a network.                                                                                                               |
| **North-south traffic**     | Traffic entering or leaving the datacenter, cluster, or application environment.                                                                                                                |
| **NTP**                     | **Network Time Protocol**. Protocol used to synchronize system clocks. Time drift can affect authentication, logs, clusters, and distributed systems.                                           |
| **NUMA**                    | **Non-Uniform Memory Access**. Server memory architecture where CPU sockets have local and remote memory access characteristics.                                                                |
| **Operational silo**        | Separation of teams, tools, or responsibilities that can slow troubleshooting and ownership during incidents.                                                                                   |
| **Packet loss**             | Network condition where packets fail to reach their destination, causing retransmissions, latency, or application errors.                                                                       |
| **PDL**                     | **Permanent Device Loss**. VMware condition where a storage device is permanently unavailable to a host.                                                                                        |
| **Prism Central**           | Nutanix centralized management interface for managing multiple clusters and advanced services.                                                                                                  |
| **Prism Element**           | Nutanix management interface for a single Nutanix cluster.                                                                                                                                      |
| **Production network**      | Network carrying application or user-facing service traffic.                                                                                                                                    |
| **Queue depth**             | Number of outstanding I/O operations a host, adapter, device, or storage system can handle.                                                                                                     |
| **RCA**                     | **Root Cause Analysis**. Post-incident analysis identifying what happened, why it happened, impact, resolution, and preventive actions.                                                         |
| **Replication**             | Copying data from one system or site to another for availability, backup, or disaster recovery.                                                                                                 |
| **Routing**                 | Process of forwarding traffic between different networks.                                                                                                                                       |
| **SAN**                     | **Storage Area Network**. Dedicated network that provides block-level storage access between servers and storage arrays.                                                                        |
| **SCSI**                    | **Small Computer System Interface**. Protocol family used for block storage communication.                                                                                                      |
| **Shared storage**          | Storage accessible by multiple hosts, commonly used for VM mobility and high availability.                                                                                                      |
| **SLA**                     | **Service Level Agreement**. Formal commitment defining service expectations such as response time, uptime, or resolution targets.                                                              |
| **SMB**                     | **Server Message Block**. File-sharing protocol commonly used in Windows environments.                                                                                                          |
| **Snapshot**                | Point-in-time copy of data, VM state, or storage volume used for backup, rollback, or protection.                                                                                               |
| **Storage**                 | The infrastructure layer responsible for persistent data services, such as SAN, NAS, volumes, LUNs, snapshots, and replication.                                                                 |
| **Storage array**           | Dedicated storage system providing centralized storage to multiple hosts.                                                                                                                       |
| **Storage controller**      | Component in a storage system that handles I/O processing, caching, failover, and data services.                                                                                                |
| **Storage network**         | Specialized part of the network layer used for storage traffic, such as Fibre Channel, iSCSI, or NFS. Not a separate main tier in the canonical 3-tier model.                                   |
| **Storage path**            | Connectivity route between a compute host and a storage device.                                                                                                                                 |
| **Switching**               | Network function that forwards traffic within the same network or VLAN.                                                                                                                         |
| **Thin provisioning**       | Allocating logical storage capacity without immediately reserving all physical capacity.                                                                                                        |
| **Throughput**              | Amount of data transferred over time, usually measured in MB/s, GB/s, or Gbps.                                                                                                                  |
| **Triage**                  | Structured process of assessing impact, scope, urgency, symptoms, and possible root cause during an incident.                                                                                   |
| **VLAN**                    | **Virtual Local Area Network**. Logical network segmentation within a physical network.                                                                                                         |
| **VM**                      | **Virtual Machine**. Software-defined machine running an operating system and applications on a hypervisor.                                                                                     |
| **VMFS**                    | **Virtual Machine File System**. VMware clustered filesystem used for storing VM files on block storage.                                                                                        |
| **vMotion**                 | VMware live migration technology for moving running VMs between hosts.                                                                                                                          |
| **Volume**                  | Logical storage unit presented by a storage system.                                                                                                                                             |
| **Workaround**              | Temporary action to restore or reduce customer impact before permanent root cause resolution.                                                                                                   |
| **Workload**                | Application, VM, service, database, or process consuming infrastructure resources.                                                                                                              |
| **Zoning**                  | Fibre Channel SAN configuration controlling which hosts can communicate with which storage targets.                                                                                             |

## Core sentence to memorize

> **Traditional 3-tier infrastructure separates Compute, Network, and Storage. The storage network is not a separate tier; it is a critical part of the Network tier, especially in SAN, iSCSI, NFS, or Fibre Channel environments.**


[1]: https://www.vmware.com/docs/vmw-vSAN-Availability-Technologies?utm_source=chatgpt.com "vSAN Availability Technologies - VMware"
[2]: https://www.nutanix.com/together/infrastructure?utm_source=chatgpt.com "Nutanix | Your infrastructure. All together now."
[3]: https://www.nutanix.com/hyperconverged-infrastructure?utm_source=chatgpt.com "What is Hyperconverged Infrastructure (HCI) - FAQs | Nutanix"
