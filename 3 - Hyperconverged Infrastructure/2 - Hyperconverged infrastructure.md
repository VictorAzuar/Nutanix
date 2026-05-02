# Hyperconverged Infrastructure

## 1. Short definition

**Hyperconverged Infrastructure, or HCI, is an architecture where compute, storage, virtualization, networking, and management are integrated into a software-defined cluster of standard servers.**

For Nutanix, the key idea is: **each node contributes CPU, memory, and local disks; Nutanix software pools those resources into a resilient platform managed centrally.** Nutanix describes Nutanix Cloud Infrastructure as combining compute, storage, and networking resources from clustered servers into a single logical pool with resiliency, security, performance, and simplified administration. ([Nutanix][1])

Your interview sentence:

> “HCI replaces the traditional separation between servers, SAN storage, and storage networking with a distributed, software-defined platform where each node contributes compute and storage. In Nutanix, AOS provides the distributed storage and data services, AHV or another hypervisor runs the workloads, and Prism provides the management and operational control plane.”

---

## 2. Clear explanation

In a traditional 3-tier architecture, you normally have:

1. **Compute layer**: servers running VMware, Hyper-V, KVM, etc.
2. **Storage layer**: SAN or NAS arrays.
3. **Network fabric**: FC, iSCSI, Ethernet, storage switches, VLANs, multipathing.

HCI collapses much of that into a **clustered software platform**. Instead of VMs depending on an external storage array, each server/node has local disks, and the HCI software aggregates those disks into a distributed storage system.

In Nutanix terms:

* **AOS** is the core software layer. Nutanix describes AOS as the software foundation that pools local storage from cluster nodes into a unified resource pool and provides the core Nutanix functionality. ([Nutanix][2])
* **DSF / Distributed Storage Fabric** is the software-defined storage layer that aggregates local drives and presents resilient storage to the hypervisor. ([Nutanix][2])
* **AHV** is Nutanix’s integrated hypervisor, but Nutanix can also run with other hypervisors such as VMware ESXi in supported configurations. Nutanix documentation distinguishes AOS as the storage/data-services layer and AHV as the hypervisor that runs VMs. ([Nutanix][2])
* **Prism Element** manages a local cluster; **Prism Central** provides centralized multi-cluster management. ([NutanixBible.com][3])
* **CVM / Controller VM** runs on each node and handles Nutanix storage services. In AHV deployments, the CVM runs as a VM and the disks are presented using PCI passthrough, allowing storage I/O to bypass the hypervisor path. ([NutanixBible.com][4])

A simplified Nutanix HCI mental model:

```text
Application / VM
   ↓
Hypervisor: AHV / ESXi
   ↓
Nutanix CVM on each node
   ↓
AOS / Distributed Storage Fabric
   ↓
Local SSD / NVMe / HDD across all nodes
   ↓
Prism for management, monitoring, upgrades, alerts
```

The important support concept is that **storage is not “somewhere else” anymore**. It is distributed across the same cluster that runs the workloads. That gives simplicity and scale-out benefits, but it also means troubleshooting requires thinking across **compute, storage, network, hypervisor, CVM health, replication, capacity, and workload placement**.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Worldwide Support Manager at Nutanix**, HCI matters because it is the core architecture behind the product. You do not need to be the deepest storage engineer in the room, but you must understand enough to lead escalations, ask the right questions, and communicate credibly with customers, SREs, TAMs, engineering, and account teams.

The support manager angle is this:

**You are managing escalations on a distributed system, not a single appliance.**

That means customer impact can come from several layers:

* A failed disk or node.
* A CVM issue.
* A hypervisor issue.
* Storage latency.
* Network packet loss or misconfiguration.
* Capacity pressure.
* A replication or rebuild event.
* A noisy workload.
* A failed upgrade.
* An external dependency such as VMware, backup software, Kubernetes, AD/LDAP, DNS, or cloud connectivity.

Nutanix AOS uses replication factor, checksums, background re-replication, and availability domains to maintain data availability and integrity after failures. ([NutanixBible.com][5]) That is exactly the kind of topic that appears in enterprise support escalations: the customer is not asking for an academic explanation; they are asking, “Are my workloads safe? Why is latency high? Can I continue production? What is Nutanix doing to recover? What is the risk if another node fails?”

Your leadership message should be:

> “My role is to understand the architecture well enough to structure the escalation, separate symptoms from root cause, protect customer impact, coordinate the right experts, and keep communication clear while the technical investigation progresses.”

---

## 4. Key concepts

### HCI vs traditional 3-tier

**Traditional 3-tier** separates compute and storage. A VM usually runs on an ESXi host and accesses storage over SAN/NAS.

**HCI** places compute and storage in the same cluster. Each node contributes resources, and software makes them appear as one platform.

Interview wording:

> “In 3-tier, the storage array is a separate control and failure domain. In HCI, storage becomes distributed software running across the compute nodes. That simplifies scaling and operations, but it changes troubleshooting because VM performance depends on the health of local nodes, CVMs, inter-node networking, metadata, replication, and capacity.”

### Node

A **node** is a physical server in the Nutanix cluster. It contributes CPU, memory, networking, and local disks.

### Cluster

A **cluster** is a group of Nutanix nodes operating together as a distributed platform.

### CVM / Controller VM

The **CVM** is central to Nutanix architecture. It runs Nutanix services on every node and handles storage I/O. In AHV deployments, disks are passed through to the CVM, and AHV is based on Linux, QEMU, and KVM. ([NutanixBible.com][4])

Support relevance:

> If the CVM has problems, storage I/O, cluster health, Prism services, or data availability workflows may be affected.

### AOS

**AOS** is the Nutanix software layer that provides storage services, data protection, backup/DR capabilities, upgrades, and workload/resource operations. The Nutanix Bible describes AOS as the core software stack providing the abstraction layer between the hypervisor and workloads. ([NutanixBible.com][6])

### DSF / Distributed Storage Fabric

**DSF** is the distributed storage layer. Nutanix describes it as aggregating local drives from nodes and presenting them as a single resilient storage pool to the hypervisor. ([Nutanix][2])

### AHV

**AHV** is Nutanix’s native hypervisor. It runs VMs and integrates tightly with AOS and Prism. It is important to know that **AHV is not the same thing as AOS**. AOS is the platform/data-services layer; AHV is the virtualization layer.

### Prism Element and Prism Central

**Prism Element** manages one local cluster.
**Prism Central** manages multiple clusters and provides a centralized management interface. Nutanix describes Prism as a distributed resource management platform for managing and monitoring Nutanix environments locally or in the cloud. ([NutanixBible.com][3])

### Storage pool, container, vDisk

Nutanix AOS storage organizes resources into constructs such as **storage pools**, **containers**, and **vDisks**. A storage pool groups physical devices; a container logically segments the storage pool; and a vDisk represents virtual disk data used by VMs. ([NutanixBible.com][5])

### Replication Factor: RF2 / RF3

Replication Factor defines how many copies of data are maintained.

* **RF2**: two copies.
* **RF3**: three copies, typically for higher resiliency requirements.

For writes, Nutanix synchronously replicates data to another one or two CVMs depending on RF before acknowledging the write. ([NutanixBible.com][5])

### Data locality

Data locality means that Nutanix tries to serve VM I/O locally from the node where the VM is running. After a VM moves to another node, reads may initially come from remote data, but AOS can migrate or cache data locally in the background. ([NutanixBible.com][5])

Support relevance:

> After live migration, HA event, node maintenance, or workload movement, temporary remote reads or re-localization can influence performance.

### Availability domains

Availability domains define how data replicas are placed across disks, nodes, blocks, or racks. Nutanix supports awareness at disk, node, block, and rack levels, with block/rack awareness designed to maintain availability during larger physical failure domains. ([NutanixBible.com][5])

### Capacity and resilient capacity

This is critical for escalations. A cluster may have raw free space but insufficient **resilient capacity** to tolerate and recover from a failure. Nutanix documentation warns that exceeding resilient capacity can prevent the cluster from self-healing and recovering from failures at the configured failure domain. ([NutanixBible.com][5])

### LCM / Life Cycle Manager

LCM manages software and firmware updates. Nutanix describes LCM as tracking software and firmware versions and providing end-to-end update management for cluster components. ([Nutanix][7])

For support, LCM matters because upgrade failures, firmware mismatch, compatibility issues, or skipped pre-checks can become serious escalations.

---

## 5. How it appears in a real escalation

### Scenario 1: “Our VMs are slow after a node failure”

Customer says:

> “After one host failed, our production SQL VMs became slow. Prism shows storage latency. We need to know if data is safe and whether we can keep running production.”

As the escalation manager, you should structure the incident like this:

1. **Business impact**

   * Which applications are affected?
   * Is this revenue-impacting?
   * Is there data loss, downtime, degraded performance, or customer-facing SLA impact?

2. **Scope**

   * One VM, multiple VMs, one node, one cluster, one site?
   * AHV or ESXi?
   * Specific storage container or all workloads?

3. **Failure state**

   * Disk failure, node failure, CVM down, network issue, cluster service issue?
   * Is the cluster still RF-compliant?
   * Is there rebuild/re-replication activity?

4. **Performance state**

   * CPU, memory, storage latency, IOPS, throughput.
   * Stargate / CVM load.
   * Network errors, packet loss, MTU issues, switch changes.

5. **Capacity state**

   * Is the cluster near or above resilient capacity?
   * Is there enough capacity to rebuild after failure?

6. **Recent changes**

   * Upgrade?
   * Firmware change?
   * New workload?
   * Backup job?
   * Network change?
   * Node expansion?

Your communication to the customer:

> “The cluster is designed to maintain availability through replication, but we need to verify the current replication state, rebuild progress, capacity headroom, and whether there is resource contention during recovery. We will separate data-risk assessment from performance-impact assessment and provide updates on both.”

That is a strong manager-level answer.

---

### Scenario 2: “Latency increased after live migration or maintenance”

Possible root causes:

* Data is temporarily remote after VM migration.
* Cluster is re-localizing data.
* Network between CVMs is congested.
* A CVM or node is overloaded.
* ADS moved workloads because of hotspot mitigation.
* Backup or snapshot activity is creating additional I/O.

Nutanix data locality is designed so that VM data can “follow” the VM after movement, but reads may be forwarded remotely until data is cached or re-localized. ([NutanixBible.com][5])

Manager-level framing:

> “I would not assume the migration itself is the root cause. I would check whether latency is correlated with remote reads, rebuild activity, CVM load, network errors, or workload contention.”

---

### Scenario 3: “Upgrade failed and customer is stuck in a maintenance window”

Relevant areas:

* LCM inventory.
* Pre-checks.
* Firmware/software dependency chain.
* AOS/AHV compatibility.
* Prism availability.
* Rollback or resume path.
* Customer change window and risk appetite.

Support manager response:

> “First I would stabilize the environment and clarify whether workloads are impacted. Then I would confirm the exact upgrade stage, failed component, pre-check results, current cluster health, and supported recovery path. I would avoid improvising manual actions unless guided by documented procedures or engineering.”

---

## 6. Triage questions I should ask

### Business and incident management

* What is the current customer impact: outage, degradation, risk-only, or planned activity blocked?
* Which business services depend on the affected workloads?
* Is there an active SLA or regulatory impact?
* Is the customer asking for technical recovery, RCA, executive communication, or all three?
* What is the required update cadence?

### Environment

* Is the cluster running AHV, ESXi, or another supported hypervisor configuration?
* How many nodes are in the cluster?
* What AOS, AHV/ESXi, Prism, firmware, and LCM versions are involved?
* Is this one cluster, multiple clusters, stretched/metro, DR, edge, or cloud deployment?
* Is Prism Element or Prism Central affected?

### Failure and health

* Are all nodes and CVMs up?
* Any disk, node, NIC, or CVM failures?
* Any critical alerts in Prism or Insights?
* Is the cluster RF-compliant?
* Is there rebuild, re-replication, or data resiliency activity in progress?
* Is the cluster close to resilient capacity?

### Performance

* Is the issue latency, throughput, IOPS, packet loss, CPU ready, memory pressure, or application response time?
* Are all VMs affected or only specific workloads?
* Did the issue start after migration, node failure, backup, snapshot, upgrade, or network change?
* Are there hot nodes, hot disks, or hot CVMs?
* Any noisy-neighbor workload?

### Network

* Any recent switch, VLAN, MTU, LACP, routing, firewall, or DNS changes?
* Are CVM-to-CVM communications healthy?
* Are there packet drops, CRC errors, retransmits, or asymmetric paths?
* Is storage/CVM traffic sharing congested links with VM traffic?

### Customer communication

* What has already been communicated to the customer?
* Is there an executive escalation?
* Who owns the next update?
* Do we need Engineering, SRE, TAM, Account Team, or Product Support involved?

---

## 7. Likely interview questions

### Conceptual

1. What is hyperconverged infrastructure?
2. How is HCI different from traditional 3-tier infrastructure?
3. What problem does Nutanix solve with HCI?
4. What are AOS, AHV, Prism, and CVM?
5. What is Distributed Storage Fabric?
6. Why does data locality matter?
7. What is replication factor?
8. How does Nutanix tolerate disk or node failure?
9. What is the difference between Prism Element and Prism Central?
10. What are common failure domains in HCI?

### Support / escalation

11. A customer reports high VM latency in a Nutanix cluster. How would you triage?
12. A node failed and the customer is worried about data loss. What do you ask?
13. An upgrade failed during a maintenance window. How do you manage the escalation?
14. A customer says “Nutanix storage is slow.” How do you avoid jumping to conclusions?
15. How do you communicate with an enterprise customer during a major incident?
16. When would you involve Engineering?
17. How do you balance technical investigation with customer communication?
18. How would you manage a disagreement between Support, SRE, Engineering, and the customer?
19. What metrics would you track for a worldwide support team handling HCI escalations?
20. What is the difference between being a technical manager and being the deepest technical IC?

### Managerial

21. How would you coach support engineers on HCI troubleshooting?
22. How do you reduce MTTR in complex infrastructure escalations?
23. How do you ensure handoffs across regions in a 24/7 support model?
24. How do you handle repeated escalations caused by the same product defect?
25. How do you build trust with enterprise customers when RCA is not yet known?

---

## 8. Model answers in English

### Q1. What is hyperconverged infrastructure?

> “Hyperconverged infrastructure is a software-defined architecture that combines compute, storage, virtualization, networking, and management into a cluster of standard servers. Instead of having separate servers, SAN storage, and storage fabric, each node contributes local resources, and the platform pools them into a resilient system. In Nutanix, AOS provides the distributed storage and data services, AHV or another hypervisor runs the workloads, and Prism provides operational management.”

---

### Q2. How is HCI different from traditional 3-tier architecture?

> “In a traditional 3-tier model, compute, storage, and networking are separate layers, often managed by different teams and vendors. HCI collapses much of that into a distributed platform where storage is software-defined and runs across the same nodes that host the workloads. The operational benefit is simpler scaling and management, but troubleshooting requires understanding the interaction between VM workload, hypervisor, CVM, storage replication, capacity, and network.”

---

### Q3. What are AOS, AHV, Prism, and CVM?

> “AOS is the Nutanix software platform that provides distributed storage, resiliency, data services, and operational capabilities. AHV is Nutanix’s native hypervisor for running virtual machines. Prism is the management and monitoring interface, with Prism Element for local cluster operations and Prism Central for centralized multi-cluster management. The CVM, or Controller VM, runs on each node and handles Nutanix storage services and communication with other CVMs in the cluster.”

---

### Q4. What is the role of the CVM?

> “The CVM is one of the most important Nutanix concepts. It runs Nutanix services on each node and handles storage I/O for the local hypervisor and workloads. In simple terms, the hypervisor sends storage requests to the local CVM, and the CVM coordinates local I/O, replication, metadata, and communication with other CVMs. From a support perspective, CVM health is critical because a CVM issue can affect storage performance, cluster services, and data availability workflows.”

---

### Q5. How does Nutanix protect data?

> “At a high level, Nutanix protects data through replication factor, checksums, availability domains, and self-healing behavior. With RF2 or RF3, data is stored in multiple independent locations. Writes are acknowledged only after the required replication is completed. If a disk or node fails, the cluster re-replicates data across healthy nodes to restore the desired protection level. In an escalation, I would verify whether the cluster is currently RF-compliant, whether rebuild is in progress, and whether there is enough resilient capacity to recover safely.”

---

### Q6. A customer reports storage latency. How would you triage?

> “I would start by separating business impact from technical symptoms. First, I would confirm which applications and VMs are affected, whether there is downtime or degradation, and the urgency of the situation. Technically, I would check whether the issue is isolated to one VM, one host, one container, or the entire cluster. Then I would look at cluster health, CVM health, disk or node failures, capacity, rebuild activity, recent changes, network errors, and workload patterns. I would avoid assuming that ‘storage latency’ means a storage defect; in HCI it can be caused by compute contention, CVM load, network issues, data re-localization, backup activity, or application behavior.”

---

### Q7. A node failed. What would you tell the customer?

> “I would acknowledge the risk clearly and avoid overpromising. I would say: ‘The platform is designed to tolerate failures through data replication, but we need to validate the current cluster state. We are checking whether the cluster remains RF-compliant, whether rebuild or re-replication is running, whether capacity is sufficient to restore resiliency, and whether workloads are experiencing performance impact. We will provide separate updates on data availability, performance, and recovery progress.’”

---

### Q8. How would you manage an escalation as a support manager, not as the deepest engineer?

> “My responsibility is to drive structure, prioritization, and communication. I need enough technical depth to ask the right questions and challenge assumptions, but I do not need to personally debug every low-level component. I would define the impact, assign technical ownership, ensure the right SMEs are engaged, keep the customer updated, manage handoffs across time zones, and make sure we move toward mitigation, root cause, and prevention. In complex HCI escalations, coordination is as important as technical diagnosis.”

---

### Q9. Why does data locality matter?

> “Data locality matters because HCI performance benefits from serving VM I/O close to where the VM is running. In Nutanix, the local CVM typically serves I/O for VMs on the same node. If a VM moves after maintenance or HA, some reads may initially be remote, and the platform can cache or migrate data locally over time. During troubleshooting, I would consider whether recent VM movement, HA events, or workload rebalancing contributed to temporary latency.”

---

### Q10. How does your SaaS/cloud operations background help with HCI support?

> “My background is directly relevant because HCI is a distributed platform. In SaaS and cloud operations, I already work with incident management, monitoring, SLAs, MTTR, capacity, change management, escalation paths, and customer communication. The specific technologies differ, but the operational discipline is similar: detect, scope, mitigate, communicate, root-cause, and prevent recurrence. My learning focus is to map that operating model onto Nutanix-specific components such as AOS, AHV, Prism, CVMs, replication factor, and cluster resiliency.”

---

## 9. Connection with my experience

Your current profile maps well to this topic.

You already understand:

* **24/7 support operations**
* **Incident management**
* **SLA and MTTR**
* **Escalation ownership**
* **Monitoring and observability**
* **Customer communication**
* **Cloud operations**
* **Change management**
* **Team coordination**
* **Post-incident improvement**

The gap is not “support management.” The gap is **Nutanix-specific infrastructure vocabulary and mental models**.

Translate your experience like this:

| Your experience           | Nutanix HCI equivalent                           |
| ------------------------- | ------------------------------------------------ |
| Cloud service degradation | Cluster / VM / storage / network degradation     |
| Kubernetes node pressure  | Nutanix node or CVM resource pressure            |
| SaaS incident commander   | Enterprise escalation manager                    |
| Grafana/Kibana dashboards | Prism / Insights / logs / alerts                 |
| SLA / MTTR                | Support severity, response, restoration, RCA     |
| Cloud capacity planning   | Cluster capacity and resilient capacity          |
| Change windows            | AOS/AHV/firmware/LCM upgrade windows             |
| Cross-team escalations    | Support + SRE + Engineering + TAM + Account Team |
| Customer-facing RCA       | Enterprise post-incident review                  |

Strong positioning statement:

> “My advantage is that I already manage enterprise incidents and 24/7 operations. I am now adding Nutanix-specific depth so I can lead escalations with better architectural fluency. I do not position myself as a Senior SRE IC; I position myself as a technical escalation manager who can understand the system, coordinate the experts, and communicate clearly with enterprise customers.”

---

## 10. Minimum I need to memorize

Memorize this first.

### 30-second definition

> “HCI combines compute, storage, virtualization, networking, and management into a software-defined cluster. In Nutanix, each node contributes local resources. AOS provides distributed storage and data services, AHV or ESXi runs the workloads, CVMs handle storage services on each node, and Prism provides management and monitoring.”

### Core Nutanix terms

* **AOS**: Nutanix software/data-services layer.
* **AHV**: Nutanix native hypervisor.
* **Prism Element**: local cluster management.
* **Prism Central**: centralized multi-cluster management.
* **CVM**: Controller VM running Nutanix services on each node.
* **DSF**: distributed storage fabric.
* **RF2/RF3**: two or three data copies.
* **Data locality**: serving I/O close to the VM.
* **Availability domain**: disk/node/block/rack failure domain.
* **LCM**: lifecycle manager for software/firmware updates.
* **Resilient capacity**: capacity required to recover after failures.

### Minimum escalation logic

When a customer reports an HCI issue, ask:

1. What is the business impact?
2. What is the scope?
3. What changed?
4. Are nodes, disks, CVMs, and Prism healthy?
5. Is the cluster RF-compliant?
6. Is rebuild or re-replication running?
7. Is capacity safe?
8. Is the issue compute, storage, network, hypervisor, workload, or change-related?
9. Who owns customer communication?
10. What is the mitigation path?

---

## 11. Advanced / optional level

You do **not** need to master these deeply for a manager interview, but you should recognize the terms.

### Advanced topics to leave for later

* Stargate internals.
* Cassandra / Medusa metadata internals.
* Zookeeper roles.
* OpLog vs Extent Store.
* BlockStore and SPDK.
* RDMA data path.
* Erasure Coding behavior under different workloads.
* Detailed AHV live migration internals.
* ADS placement algorithms.
* Deep vSphere-on-Nutanix architecture.
* Metro Availability / NearSync / Sync replication details.
* Nutanix Files, Objects, Volumes internals.
* Flow microsegmentation.
* NCC checks and log bundle analysis.
* Foundation imaging and bare-metal deployment.

### What to say if pushed

> “I understand the high-level role of those components and how they may influence an escalation, but I would rely on the appropriate SRE or engineering SME for low-level internals. My role would be to ensure the investigation is structured, the right data is collected, and the customer receives accurate communication.”

That is an acceptable manager-level answer.

---

## 12. Final checklist

Before your interview, make sure you can explain:

* [ ] What HCI is.
* [ ] How HCI differs from traditional 3-tier.
* [ ] Why Nutanix is built around HCI.
* [ ] What AOS does.
* [ ] What AHV does.
* [ ] What Prism Element and Prism Central do.
* [ ] What a CVM is.
* [ ] What DSF is.
* [ ] What RF2/RF3 means.
* [ ] What data locality means.
* [ ] Why capacity and resilient capacity matter.
* [ ] How a node or disk failure affects the cluster.
* [ ] How to triage latency in an HCI environment.
* [ ] How to communicate during a high-severity enterprise escalation.
* [ ] How to position yourself as a technical escalation manager rather than a Senior SRE IC.

Your core message should be:

> “I understand HCI as a distributed enterprise platform. My strength is leading complex support operations: scoping impact, driving triage, coordinating technical experts, managing escalations, and communicating clearly with customers. I am building Nutanix-specific depth around AOS, AHV, Prism, CVMs, data resiliency, and cluster operations so I can lead these escalations effectively.”

# Annex I — Keywords, Acronyms, and Meanings

| Acronym / Term | Meaning                                                                                                                                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **ADS**        | **Acropolis Dynamic Scheduler**. Nutanix service that helps optimize workload placement and resource usage across the cluster.                                                                          |
| **AHV**        | **Acropolis Hypervisor**. Nutanix’s native virtualization hypervisor based on KVM. It runs virtual machines on Nutanix clusters.                                                                        |
| **AOS**        | **Acropolis Operating System**. The core Nutanix software layer that provides distributed storage, data services, resiliency, upgrades, and cluster intelligence.                                       |
| **API**        | **Application Programming Interface**. A way for systems or applications to communicate programmatically. Relevant for automation, integrations, monitoring, and support tooling.                       |
| **AWS**        | **Amazon Web Services**. Public cloud provider. Relevant in hybrid-cloud, migration, backup, DR, and SaaS operations discussions.                                                                       |
| **Azure**      | **Microsoft Azure**. Public cloud provider. Relevant for hybrid-cloud, identity, workloads, backup, and enterprise integrations.                                                                        |
| **Block**      | A physical Nutanix chassis that may contain one or more nodes. In Nutanix, “block awareness” helps protect against chassis-level failure.                                                               |
| **BMC**        | **Baseboard Management Controller**. Out-of-band hardware management interface used for server health, power control, and remote access. Similar concepts include IPMI, iDRAC, and iLO.                 |
| **Cassandra**  | Distributed metadata store used inside Nutanix AOS. Advanced topic; useful to recognize in deeper troubleshooting.                                                                                      |
| **CE**         | **Community Edition**. Free Nutanix edition used for labs and learning. Not the same as production Nutanix enterprise deployments.                                                                      |
| **CLI**        | **Command-Line Interface**. Text-based interface used for administration and troubleshooting.                                                                                                           |
| **CVM**        | **Controller Virtual Machine**. A critical Nutanix VM running on each node. It provides storage services and participates in the distributed storage fabric.                                            |
| **DAS**        | **Direct-Attached Storage**. Storage physically attached to a server. HCI uses local disks from each node but turns them into a distributed storage platform.                                           |
| **DR**         | **Disaster Recovery**. Processes and technologies used to recover services after major failure, outage, or site loss.                                                                                   |
| **DSF**        | **Distributed Storage Fabric**. Nutanix’s software-defined storage layer that pools local disks across nodes and presents resilient shared storage to workloads.                                        |
| **EC-X**       | **Erasure Coding Extension**. Nutanix capacity optimization feature that reduces storage overhead compared with full replication. Advanced topic.                                                       |
| **ESXi**       | VMware’s enterprise hypervisor. Nutanix can run with AHV or VMware ESXi depending on the supported deployment.                                                                                          |
| **FC**         | **Fibre Channel**. Traditional storage networking technology commonly used in SAN environments. Important when comparing HCI with 3-tier architecture.                                                  |
| **GCP**        | **Google Cloud Platform**. Public cloud provider. Relevant in hybrid-cloud, SaaS, and infrastructure operations contexts.                                                                               |
| **HA**         | **High Availability**. Design principle that keeps services running despite failures. In Nutanix, HA can refer to VM restart behavior and cluster resiliency.                                           |
| **HCI**        | **Hyperconverged Infrastructure**. Architecture that combines compute, storage, virtualization, networking, and management into a software-defined cluster.                                             |
| **HDD**        | **Hard Disk Drive**. Magnetic disk storage. Slower but often used for capacity tiers in hybrid storage systems.                                                                                         |
| **I/O**        | **Input/Output**. Read and write operations between workloads and storage. Critical in performance troubleshooting.                                                                                     |
| **IOPS**       | **Input/Output Operations Per Second**. Storage performance metric measuring the number of read/write operations per second.                                                                            |
| **IPMI**       | **Intelligent Platform Management Interface**. Hardware management interface used for out-of-band server control and monitoring.                                                                        |
| **iSCSI**      | **Internet Small Computer Systems Interface**. Storage protocol that carries SCSI commands over IP networks. Common in traditional SAN environments.                                                    |
| **KPI**        | **Key Performance Indicator**. Metric used to track performance of systems, services, or teams. For support: SLA compliance, MTTR, backlog, escalation rate, CSAT.                                      |
| **KVM**        | **Kernel-based Virtual Machine**. Linux virtualization technology. AHV is based on KVM/QEMU.                                                                                                            |
| **LACP**       | **Link Aggregation Control Protocol**. Network protocol used to bundle multiple physical links into one logical link. Misconfiguration can cause network and performance issues.                        |
| **LCM**        | **Life Cycle Manager**. Nutanix tool for managing software and firmware updates across cluster components.                                                                                              |
| **LDAP**       | **Lightweight Directory Access Protocol**. Protocol used for directory services and authentication integrations.                                                                                        |
| **MTTR**       | **Mean Time To Restore / Mean Time To Repair / Mean Time To Resolve**. Support metric measuring how long it takes to restore or resolve an incident. Clarify the exact definition in each organization. |
| **NAS**        | **Network-Attached Storage**. File-level storage accessed over the network, usually via protocols such as NFS or SMB.                                                                                   |
| **NCC**        | **Nutanix Cluster Check**. Nutanix diagnostic framework used to run health checks and identify known issues or misconfigurations.                                                                       |
| **NCI**        | **Nutanix Cloud Infrastructure**. Nutanix platform for running hybrid multicloud infrastructure, including compute, storage, virtualization, and networking capabilities.                               |
| **NearSync**   | Nutanix disaster recovery capability with low recovery point objectives. Advanced DR topic.                                                                                                             |
| **NFS**        | **Network File System**. File-sharing protocol commonly used in Linux/Unix environments and traditional virtualization storage.                                                                         |
| **NIC**        | **Network Interface Card**. Physical or virtual network adapter. NIC health, speed, errors, and configuration are important in HCI troubleshooting.                                                     |
| **NVMe**       | **Non-Volatile Memory Express**. High-performance storage protocol for SSDs. Relevant to modern Nutanix performance architectures.                                                                      |
| **OpLog**      | **Operation Log**. Nutanix write optimization component. Advanced AOS internals topic.                                                                                                                  |
| **PC**         | **Prism Central**. Nutanix centralized management plane for multiple clusters, analytics, automation, and broader operational visibility.                                                               |
| **PE**         | **Prism Element**. Nutanix local management interface for a single cluster.                                                                                                                             |
| **Prism**      | Nutanix management and operations interface. Includes Prism Element and Prism Central. Used for monitoring, alerts, capacity, upgrades, and administration.                                             |
| **QEMU**       | **Quick Emulator**. Open-source machine emulator and virtualizer used with KVM. Part of the virtualization stack behind AHV.                                                                            |
| **QoS**        | **Quality of Service**. Mechanisms used to prioritize or control resource usage, often network or storage-related.                                                                                      |
| **RACK**       | Physical rack-level failure domain. Rack awareness helps protect against the loss of an entire rack in larger deployments.                                                                              |
| **RCA**        | **Root Cause Analysis**. Post-incident process used to identify what happened, why it happened, customer impact, corrective actions, and preventive measures.                                           |
| **RDM**        | **Raw Device Mapping**. VMware concept allowing a VM to access a storage LUN directly. Relevant mainly when discussing VMware environments.                                                             |
| **RF2**        | **Replication Factor 2**. Nutanix stores two copies of data, allowing tolerance of certain failure scenarios depending on cluster design and failure domain.                                            |
| **RF3**        | **Replication Factor 3**. Nutanix stores three copies of data, increasing resiliency at the cost of more capacity overhead.                                                                             |
| **RPO**        | **Recovery Point Objective**. Maximum acceptable amount of data loss measured in time. Example: “We can lose up to 15 minutes of data.”                                                                 |
| **RTO**        | **Recovery Time Objective**. Maximum acceptable time to restore service after an incident. Example: “Service must be restored within one hour.”                                                         |
| **SAN**        | **Storage Area Network**. Dedicated storage network providing block storage to servers. Common in traditional 3-tier architecture.                                                                      |
| **SATA**       | **Serial ATA**. Storage interface commonly associated with HDDs and some SSDs.                                                                                                                          |
| **SCSI**       | **Small Computer System Interface**. Family of standards for connecting and transferring data between computers and storage devices.                                                                    |
| **SLA**        | **Service Level Agreement**. Contractual or operational agreement defining expected service levels, response times, availability, or support commitments.                                               |
| **SMB**        | **Server Message Block**. File-sharing protocol commonly used in Windows environments.                                                                                                                  |
| **SME**        | **Subject Matter Expert**. Specialist with deep expertise in a specific technical area.                                                                                                                 |
| **SSD**        | **Solid-State Drive**. Flash-based storage device. Faster than HDD and commonly used for performance tiers.                                                                                             |
| **Stargate**   | Nutanix service responsible for handling data I/O. Advanced AOS internal component; useful to recognize in performance escalations.                                                                     |
| **SRE**        | **Site Reliability Engineer / Site Reliability Engineering**. Discipline focused on reliability, automation, observability, incident response, and scalable operations.                                 |
| **VLAN**       | **Virtual Local Area Network**. Logical network segmentation mechanism. Incorrect VLAN configuration can break cluster, CVM, management, or VM traffic.                                                 |
| **VM**         | **Virtual Machine**. Software-defined server running on a hypervisor such as AHV or ESXi.                                                                                                               |
| **vDisk**      | **Virtual Disk**. Logical disk attached to a VM. In Nutanix, vDisk data is stored and managed by AOS/DSF.                                                                                               |
| **vMotion**    | VMware live migration technology for moving running VMs between ESXi hosts. Nutanix AHV has equivalent VM mobility concepts, but “vMotion” is VMware-specific.                                          |
| **vNIC**       | **Virtual Network Interface Card**. Virtual network adapter attached to a VM.                                                                                                                           |
| **vSAN**       | VMware’s software-defined storage technology. Often compared with Nutanix HCI, but it is a VMware-specific architecture.                                                                                |
| **Zookeeper**  | Distributed coordination service used inside Nutanix AOS. Advanced internal component; useful to recognize, not necessary to master for a manager interview.                                            |

## Interview-critical acronyms to memorize first

| Acronym       | Meaning                                            |
| ------------- | -------------------------------------------------- |
| **HCI**       | Hyperconverged Infrastructure                      |
| **AOS**       | Acropolis Operating System                         |
| **AHV**       | Acropolis Hypervisor                               |
| **CVM**       | Controller Virtual Machine                         |
| **DSF**       | Distributed Storage Fabric                         |
| **Prism**     | Nutanix management platform                        |
| **PE**        | Prism Element                                      |
| **PC**        | Prism Central                                      |
| **RF2 / RF3** | Replication Factor 2 / 3                           |
| **HA**        | High Availability                                  |
| **DR**        | Disaster Recovery                                  |
| **RPO / RTO** | Recovery Point Objective / Recovery Time Objective |
| **LCM**       | Life Cycle Manager                                 |
| **NCC**       | Nutanix Cluster Check                              |
| **MTTR**      | Mean Time To Restore / Repair / Resolve            |
| **SLA**       | Service Level Agreement                            |
| **RCA**       | Root Cause Analysis                                |
| **SME**       | Subject Matter Expert                              |

## One-sentence verbal summary

> “For this topic, the key acronyms are HCI, AOS, AHV, CVM, DSF, Prism, RF, HA, DR, LCM, NCC, SLA, MTTR, and RCA, because together they describe the architecture, the operational model, and the escalation language used in Nutanix enterprise support.”



[1]: https://www.nutanix.com/library/datasheets/nci "Nutanix Cloud Infrastructure (NCI)"
[2]: https://www.nutanix.com/products/nutanix-cloud-infrastructure/distributed-storage "A Modern Storage Fabric | Nutanix"
[3]: https://www.nutanixbible.com/3a-book-of-prism-architecture.html "The Nutanix Cloud Bible"
[4]: https://www.nutanixbible.com/5a-book-of-ahv-architecture.html "The Nutanix Cloud Bible"
[5]: https://www.nutanixbible.com/4c-book-of-aos-storage.html "The Nutanix Cloud Bible"
[6]: https://www.nutanixbible.com/4-book-of-aos.html "The Nutanix Cloud Bible"
[7]: https://portal.nutanix.com/docs/Life-Cycle-Manager-Guide-v3_3%3Atop-lcm-overview-c.html?utm_source=chatgpt.com "LCM 3.3 - Life Cycle Manager Overview - Nutanix"
