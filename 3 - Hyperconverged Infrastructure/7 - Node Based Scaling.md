# Node-based scaling

## 1. Short definition

**Node-based scaling** means increasing or decreasing infrastructure capacity by adding or removing complete server nodes to a cluster. In Nutanix HCI, each node normally contributes **compute, memory, storage, network connectivity, and a Controller VM (CVM)** to the distributed platform.

In simple terms: instead of scaling only storage or only compute independently, Nutanix clusters scale by adding more nodes to the cluster so the platform can distribute workloads, data, metadata, and resilience across more physical resources.

---

## 2. Clear explanation

In traditional enterprise infrastructure, scaling often means buying separate components:

* more servers for compute,
* more SAN/NAS capacity for storage,
* more switches or network fabric for throughput,
* more virtualization hosts for hypervisor capacity.

In **hyperconverged infrastructure (HCI)**, these layers are integrated. A Nutanix node is not just “a server”; it is part of a distributed system. Each node normally runs:

* a **hypervisor** such as AHV or ESXi,
* a **Controller VM**, which participates in the Nutanix storage and control services,
* local disks/SSDs/HDDs/NVMe,
* host networking,
* customer workloads such as VMs or services.

When a node is added, the cluster can increase available compute, memory, storage capacity, storage throughput, and resiliency domain options, depending on the node type and cluster design.

Nutanix documentation describes cluster expansion through **Prism Central** or **Prism Element**, and the expansion process can include preparing nodes, imaging the hypervisor if needed, upgrading AOS if needed, and configuring network settings before adding the nodes to the cluster. The cluster discovers unassigned, factory-prepared nodes on the same subnet during the expansion workflow. ([Nutanix Portal][1])

For interviews, the key idea is:

> Nutanix scales horizontally by adding nodes to a distributed cluster. Each node increases the cluster resource pool, and AOS redistributes services and data according to resiliency, capacity, and performance policies.

This is different from **scale-up**, where you make one box bigger. Nutanix is designed around **scale-out**, where the system grows by adding more nodes.

---

## 3. Why it matters for Nutanix / Worldwide Support

For a **Manager, Worldwide Support** role, node-based scaling matters because many enterprise escalations are not isolated software bugs. They are often linked to:

* customer growth,
* poor sizing,
* capacity exhaustion,
* cluster expansion failures,
* performance degradation after adding workloads,
* lifecycle mismatches between old and new nodes,
* incorrect network preparation,
* replication/resiliency constraints,
* customer expectations around “linear scaling.”

A support manager does not need to be the deepest AOS engineer in the room, but must understand the operational impact of adding or removing nodes.

You need to be able to lead conversations like:

> “Is this a product defect, a scaling limit, a misconfiguration, a capacity planning issue, or an environmental issue?”

Nutanix has explicit prerequisites and considerations for expansion. For example, the process can vary depending on AOS version, hypervisor host type, data-at-rest encryption status, and hardware configuration factors. Nutanix also warns that the total number of nodes must not exceed the cluster maximums. ([Nutanix Portal][2])

That is very relevant to enterprise support because customers may escalate with:

* “We added nodes, but performance did not improve.”
* “The new node is not discovered.”
* “Expansion failed halfway.”
* “Capacity increased, but usable space is lower than expected.”
* “Resiliency is still degraded after expansion.”
* “The cluster is imbalanced.”
* “We need urgent expansion because production is running out of capacity.”

As a manager, your value is coordinating the correct response: Support, SRE, Engineering, Field/Professional Services, Account Team, and the customer.

---

## 4. Key concepts

### Scale-out vs scale-up

**Scale-up** means increasing resources inside an existing system, such as adding more RAM, disks, or CPUs to a server.

**Scale-out** means adding more nodes to a distributed system.

Nutanix HCI is primarily a **scale-out architecture**. The operational advantage is incremental growth: customers can add capacity in controlled units instead of doing disruptive forklift upgrades.

Interview wording:

> “In Nutanix, node-based scaling is a scale-out model. You grow the cluster by adding nodes, and the platform distributes storage, metadata, services, and workloads across the expanded resource pool.”

---

### Node

A **node** is a physical server participating in the Nutanix cluster. Depending on configuration, it may contribute compute, storage, or both.

In AHV deployments, each node runs a **Controller VM**, and Nutanix Bible describes AHV as using a CVM with direct disk access via PCI passthrough, allowing the CVM to handle storage I/O efficiently. ([NutanixBible.com][3])

---

### Cluster

A **cluster** is a group of Nutanix nodes managed as one logical system. Cluster-level services coordinate storage, metadata, resiliency, health, and management.

When a node is added, it joins the cluster resource pool. The objective is not merely “more hardware,” but an expanded distributed system.

---

### Controller VM / CVM

The **Controller VM** is central to Nutanix architecture. It runs on each node and participates in Nutanix distributed services.

For an interview, do not overclaim by saying the CVM is “just storage.” Better wording:

> “The CVM is a key Nutanix software component running on each node. It participates in distributed storage and cluster services, so node health and CVM health are tightly connected.”

---

### AOS

**AOS** is the Nutanix operating software layer that provides distributed storage and data services.

In node-based scaling, AOS is responsible for integrating new node resources into the cluster and maintaining resilience, data distribution, and service health.

---

### AHV

**AHV** is Nutanix’s native hypervisor. Nutanix clusters may also exist with other hypervisors, but AHV is strategically important for Nutanix.

For support interviews, connect AHV to:

* VM placement,
* host maintenance,
* live migration,
* networking,
* CVM operation,
* host health,
* cluster expansion.

---

### Prism Element and Prism Central

**Prism Element** manages an individual cluster.

**Prism Central** provides centralized management across clusters and can also be used for workflows such as cluster expansion. Nutanix documentation states that nodes can be added through Prism Central or Prism Element. ([Nutanix Portal][4])

---

### Storage distribution and resiliency

Nutanix uses distributed storage concepts. Data is protected through mechanisms such as replication factor and resiliency policies.

Nutanix documentation states that cluster data resiliency depends on combinations of replication factor, minimum node count, and minimum block count. ([Nutanix Portal][5])

That matters because adding nodes is not just about raw capacity. You must consider:

* usable capacity,
* replication overhead,
* rebuild capacity,
* fault tolerance,
* block/rack awareness,
* metadata resiliency,
* workload placement.

---

### Replication Factor / Resiliency Factor

**Replication Factor** determines how many copies of data are maintained. Nutanix documentation and architecture references commonly discuss RF2 and RF3 as key resiliency models. Nutanix documentation also explains that availability depends on the replication factor of the cluster or container and the failure domain involved. ([Nutanix Portal][6])

Simple interview framing:

> “Adding nodes increases available raw resources, but usable capacity depends on replication and resiliency configuration. A support manager must avoid promising a one-to-one conversion from raw capacity to usable capacity.”

---

### Block awareness

A Nutanix **block** can contain multiple nodes. Nutanix documentation explains that block-aware clusters place redundant data copies on nodes outside the same block when conditions are met, reducing the impact of block-level failures. ([Nutanix Portal][7])

For escalation purposes, this matters when customers add nodes but still do not meet the right fault domain requirements.

---

### Network discovery

Cluster expansion depends heavily on networking. Nutanix documentation notes that the expand-cluster discovery step requires IPv6 multicast packets through the physical switch; if IPv6 is disabled, the process does not work. ([Nutanix Portal][8])

This is a very interview-relevant troubleshooting detail.

A practical support manager should ask:

> “Is this really a Nutanix software issue, or are the new nodes not discoverable because of network, VLAN, multicast, switch, or subnet constraints?”

---

## 5. How it appears in a real escalation

### Scenario 1: New node is not discovered

Customer says:

> “We racked two new Nutanix nodes, but Prism only detects one.”

Possible causes:

* node not on the expected subnet,
* IPv6 multicast blocked,
* switch/VLAN issue,
* node not factory-prepared,
* wrong hypervisor image,
* AOS version mismatch,
* cabling issue,
* IPMI/BMC configuration issue,
* DNS/NTP/network services issue,
* node already assigned to another cluster,
* hardware fault.

As support manager, you would coordinate:

1. Check the exact expansion method: Prism Element, Prism Central, Foundation, manual process.
2. Confirm network prerequisites.
3. Validate the new node’s state.
4. Confirm hypervisor and AOS compatibility.
5. Review cluster health before expansion.
6. Assign SRE/support engineer to collect logs and health checks.
7. Communicate customer impact and next update time.

Strong interview wording:

> “I would avoid treating this as a simple UI problem. Node discovery depends on the environment, especially networking. I would first confirm cluster health, node preparation state, subnet/VLAN placement, multicast requirements, and version compatibility before escalating to engineering.”

---

### Scenario 2: Expansion succeeded, but performance is still poor

Customer says:

> “We added nodes, but latency is still high.”

Possible causes:

* bottleneck is not compute/storage capacity,
* workload imbalance,
* hot VM or hot vDisk,
* network saturation,
* storage tiering/caching behavior,
* insufficient time for data balancing,
* replication/rebuild activity consuming resources,
* CPU ready/steal/overcommit,
* database/application issue,
* noisy neighbor,
* customer expected immediate linear scaling.

As a manager:

* keep the customer focused on evidence,
* separate symptoms from assumptions,
* request performance graphs before/after expansion,
* involve SRE/storage/network specialists,
* define what “success” means: latency, IOPS, throughput, CPU, queue depth, VM response time.

Interview wording:

> “Adding nodes increases the resource pool, but it does not automatically solve every performance issue. I would validate where the bottleneck is: CPU, memory, storage latency, network, VM placement, or application behavior.”

---

### Scenario 3: Capacity increased, but usable capacity is lower than expected

Customer says:

> “We added 100 TB raw, but Prism shows much less usable capacity.”

Explanation:

* replication factor consumes capacity,
* metadata and system overhead exist,
* snapshots/clones may consume space,
* rebuild/reserve thresholds matter,
* storage-only or heterogeneous nodes may behave differently,
* compression/dedup/EC-X may affect effective capacity.

Manager angle:

> “The customer may think in raw capacity, while the platform operates in usable and protected capacity. I would make sure the communication distinguishes raw, usable, logical, effective, and reserved capacity.”

---

### Scenario 4: Expansion fails due to version or hardware constraints

Customer says:

> “Cluster expansion fails during preparation.”

Possible causes:

* AOS version mismatch,
* hypervisor mismatch,
* firmware compatibility,
* encryption configuration,
* unsupported hardware combination,
* maximum cluster size reached,
* unsupported compute-only preparation,
* insufficient prerequisites.

Nutanix documentation states that expansion varies depending on AOS version, hypervisor type, encryption status, and hardware factors, and that cluster maximums must be respected. ([Nutanix Portal][2])

Manager action:

* assign technical owner,
* collect exact error,
* validate support matrix,
* check compatibility and cluster limits,
* protect production before retrying,
* communicate rollback/next-step plan.

---

## 6. Triage questions I should ask

### Customer impact

1. What business service is affected?
2. Is this production?
3. Is there current downtime, degradation, or only expansion risk?
4. Are workloads currently healthy?
5. Is the cluster in a degraded resiliency state?
6. Is there an SLA or maintenance window constraint?

### Expansion context

7. Are we adding HCI nodes, compute-only nodes, or storage-only nodes?
8. How many nodes are being added?
9. Are the nodes the same model/generation as the existing cluster?
10. Are they factory-prepared or newly imaged?
11. Was Foundation used?
12. Is the expansion through Prism Element or Prism Central?

### Software and compatibility

13. What AOS version is running?
14. What hypervisor is used: AHV, ESXi, Hyper-V?
15. Are all nodes on compatible versions?
16. Is encryption enabled?
17. Are there mixed hardware or mixed hypervisor considerations?
18. Are we near cluster maximum limits?

### Network

19. Are the new nodes on the same subnet required for discovery?
20. Are VLANs correctly configured?
21. Is IPv6 disabled anywhere?
22. Is multicast allowed on the physical switch?
23. Are IPMI/BMC, host, and CVM networks reachable?
24. Are DNS and NTP correct?

### Health and resiliency

25. Was the cluster healthy before expansion?
26. Are there active alerts?
27. Is any node, disk, CVM, or service degraded?
28. Is the cluster rebuilding or rebalancing?
29. What is the current replication/resiliency factor?
30. Is block/rack awareness relevant?

### Performance

31. What metric is degraded: latency, IOPS, throughput, CPU, memory, network?
32. Did the issue start before, during, or after expansion?
33. Is the load balanced across nodes?
34. Are there hot VMs or hot disks?
35. Is the customer expecting immediate linear improvement?

### Communication and ownership

36. Who owns the customer bridge?
37. Who is technical lead?
38. Do we need SRE, Engineering, Field, or Account Team involvement?
39. What is the next update commitment?
40. What is the mitigation while root cause is investigated?

---

## 7. Likely interview questions

### Manager / leadership interview

1. How would you explain node-based scaling to an enterprise customer?
2. How would you manage an escalation where a customer cannot add new nodes to a Nutanix cluster?
3. How would you handle a customer who added nodes but still sees poor performance?
4. How do you distinguish a technical issue from a sizing or expectation issue?
5. How would you coordinate Support, SRE, Engineering, and Account teams during a cluster expansion escalation?
6. How would you communicate risk during production expansion?
7. What KPIs would you monitor during a scaling-related escalation?
8. How would you coach your support engineers to troubleshoot node expansion failures?
9. How would you avoid over-escalating every cluster expansion issue to Engineering?
10. How would you handle a high-severity customer bridge where the customer blames Nutanix after a failed expansion?

### Technical / SRE interview

1. What is scale-out architecture?
2. What happens when you add a node to a Nutanix cluster?
3. What is the role of the CVM in node-based scaling?
4. What is the difference between raw, usable, and effective capacity?
5. What is the impact of replication factor on usable capacity?
6. What are common reasons a new node is not discovered?
7. Why can networking block cluster expansion?
8. What is block awareness?
9. Why might performance not improve after adding nodes?
10. What checks would you perform before expanding a production cluster?
11. What is the difference between Prism Element and Prism Central for expansion?
12. How do AHV and ESXi considerations affect cluster expansion?
13. How would you troubleshoot a node that joined but remains unhealthy?
14. What risks exist when expanding a degraded cluster?
15. What information would you collect before escalating to Engineering?

---

## 8. Model answers in English

### Q1. What is node-based scaling?

**Model answer:**

> Node-based scaling is a scale-out model where capacity is increased by adding complete nodes to a cluster. In a Nutanix environment, a node typically contributes compute, memory, storage, networking, and a Controller VM that participates in the distributed platform. The important point is that scaling is not just adding hardware; the cluster has to integrate the node into its distributed storage, resiliency, metadata, and management services.

---

### Q2. Why is node-based scaling important in Nutanix?

**Model answer:**

> It is central to the HCI value proposition. Nutanix allows customers to grow infrastructure incrementally instead of performing large, disruptive upgrades. From a support perspective, node-based scaling is important because expansion touches multiple layers: hardware, AOS, hypervisor, networking, storage resiliency, capacity planning, and customer expectations. Many escalations happen when one of those layers is not ready or when the customer expects linear improvement without validating the real bottleneck.

---

### Q3. What happens when a node is added to a Nutanix cluster?

**Model answer:**

> At a high level, the node is prepared, discovered, validated, and added to the cluster. Depending on the environment, this may involve imaging or validating the hypervisor, matching or upgrading AOS, configuring networking, and integrating the node into cluster services. After expansion, the cluster can use the new resources, but health, resiliency, and balancing must be monitored. I would always verify cluster health before and after expansion, not just whether the workflow completed.

---

### Q4. A customer says: “We added nodes but performance did not improve.” How would you respond?

**Model answer:**

> I would first acknowledge the expectation but avoid assuming the expansion failed. Adding nodes increases the resource pool, but performance improvement depends on the bottleneck. I would compare before-and-after metrics: storage latency, IOPS, throughput, CPU, memory, network, VM placement, and application behavior. I would also check whether the cluster is rebalancing or rebuilding after the expansion. If the bottleneck is a hot VM, network path, application lock, or external dependency, adding nodes may not immediately improve the symptom.

---

### Q5. A new node is not discovered during expansion. What would you check?

**Model answer:**

> I would check whether the node is correctly racked, powered, cabled, and reachable; whether it is on the correct subnet and VLAN; whether multicast and discovery prerequisites are met; whether the node is factory-prepared or correctly imaged; whether AOS and hypervisor versions are compatible; and whether the existing cluster is healthy. I would also validate IPMI/BMC access, DNS, NTP, and switch configuration. From a support manager perspective, I would make sure an engineer owns the technical investigation while I manage impact, timeline, and customer communication.

---

### Q6. How does replication factor affect node-based scaling?

**Model answer:**

> Replication factor determines how data is protected and directly affects usable capacity. If data is stored with multiple copies, raw capacity does not equal usable capacity. Adding nodes increases raw resources, but the amount available to workloads depends on replication, resiliency, metadata, snapshots, and reserved capacity. This is important in customer conversations because many misunderstandings come from comparing raw capacity purchased with usable protected capacity shown by the platform.

---

### Q7. What risks would you consider before expanding a production cluster?

**Model answer:**

> I would check current cluster health, active alerts, resiliency status, capacity headroom, AOS and hypervisor compatibility, hardware compatibility, network readiness, maintenance window, rollback or pause options, and customer impact. I would avoid expanding a cluster that is already unstable unless expansion is part of an agreed mitigation plan. The key is to reduce risk before introducing new nodes into a production distributed system.

---

### Q8. How would you lead a scaling-related escalation as a manager?

**Model answer:**

> I would separate three tracks: technical diagnosis, customer communication, and internal coordination. Technically, I would ensure we validate health, compatibility, network, capacity, and performance data. For communication, I would provide clear updates, explain what is known and unknown, and set the next action. Internally, I would assign ownership across Support, SRE, Engineering, Field, or Account teams as needed. My role is not to replace the SRE, but to keep the escalation structured, evidence-based, and customer-focused.

---

### Q9. What is the difference between scale-up and scale-out?

**Model answer:**

> Scale-up means making an existing system larger, for example adding more CPU or memory to a server. Scale-out means adding more nodes to a distributed system. Nutanix is primarily a scale-out platform: customers add nodes to increase the cluster resource pool. That model provides incremental growth, but it also requires good design around networking, resiliency, workload distribution, and capacity planning.

---

### Q10. How would you explain this to a non-technical executive customer?

**Model answer:**

> I would say that Nutanix grows like a distributed platform rather than like a single storage array. Each new node adds resources to the shared pool, but the benefit depends on how the application consumes compute, storage, and network. Before promising performance improvement, we need to confirm where the bottleneck is. The expansion may increase capacity and resilience, but the specific business outcome must be validated with metrics.

---

## 9. Connection with my experience

Your background maps well to this topic because node-based scaling is not just a hardware concept. It is an **operations, incident, and customer-management topic**.

At Harmonic, you already deal with:

* 24/7 support,
* incident management,
* SLA and MTTR,
* cloud operations,
* monitoring,
* escalations,
* dashboards,
* customer communication,
* cross-functional coordination,
* capacity and performance conversations.

You can position your experience like this:

> “Although my current environment is SaaS/cloud rather than Nutanix HCI, the operational pattern is familiar: capacity changes must be planned, monitored, validated, and communicated. Whether we scale a Kubernetes cluster, a cloud service, or a Nutanix HCI cluster, the support leadership principles are similar: understand the bottleneck, protect production, validate health, define ownership, and communicate clearly.”

Specific analogies:

| Your SaaS/cloud experience      | Nutanix equivalent                          |
| ------------------------------- | ------------------------------------------- |
| Scaling Kubernetes worker nodes | Adding Nutanix cluster nodes                |
| Monitoring pod/node health      | Monitoring host/CVM/cluster health          |
| Cloud capacity planning         | HCI sizing and expansion planning           |
| Incident bridge leadership      | Customer escalation bridge                  |
| SLA / MTTR management           | Severity and restoration management         |
| Grafana/Kibana dashboards       | Prism metrics, alerts, logs                 |
| Jira/Confluence process         | Case notes, KBs, RCA documentation          |
| Cross-team escalation           | Support/SRE/Engineering/Field collaboration |

Strong positioning phrase:

> “I am not positioning myself as the deepest AOS developer or a senior storage IC. I am positioning myself as a technical escalation manager who can understand the architecture, ask the right triage questions, coordinate specialists, and communicate clearly with enterprise customers.”

That is exactly the right framing for this role.

---

## 10. Minimum I need to memorize

Memorize these points:

1. **Node-based scaling = scale-out by adding nodes to the cluster.**
2. In Nutanix, a node typically contributes **compute, memory, storage, networking, hypervisor, and CVM**.
3. Expansion can be done through **Prism Element** or **Prism Central**. ([Nutanix Portal][4])
4. Adding nodes is not just hardware insertion; it involves **discovery, compatibility, networking, AOS/hypervisor alignment, and health validation**.
5. Nutanix expansion depends on factors such as **AOS version, hypervisor type, encryption, and hardware configuration**. ([Nutanix Portal][2])
6. Node discovery can fail because of **networking**, including subnet, VLAN, multicast, or IPv6-related issues. Nutanix documentation notes IPv6 multicast requirements for discovery. ([Nutanix Portal][8])
7. Raw capacity is not usable capacity. **Replication factor and resiliency consume capacity.**
8. Adding nodes does not automatically solve performance issues; you must identify the bottleneck.
9. Cluster expansion should be preceded by **health checks**, compatibility validation, and risk assessment.
10. As a manager, your role is to drive **triage, ownership, escalation, communication, and customer confidence**.

Best 30-second answer:

> “Node-based scaling in Nutanix means expanding the cluster by adding nodes that contribute compute, storage, networking, and cluster services through components like the CVM. It is a scale-out model, so the platform grows incrementally. From a support perspective, the critical areas are cluster health, compatibility, networking, resiliency, capacity planning, and customer expectation management. If an expansion fails or does not deliver the expected result, I would lead an evidence-based triage across hardware, AOS, hypervisor, network, and workload metrics while keeping customer communication clear.”

---

## 11. Advanced / optional level

You do not need to master these deeply before the interview, but you should recognize the terms.

### Advanced areas to study later

* Detailed AOS internals.
* Stargate, Curator, Cassandra, Zookeeper roles.
* Metadata ring behavior.
* Rebuild mechanics after node failure.
* Data locality and read/write path internals.
* Erasure coding behavior after expansion.
* RF2 vs RF3 design trade-offs.
* Storage-only node design.
* Compute-only node constraints.
* Mixed-node clusters.
* AHV networking internals.
* ESXi on Nutanix operational differences.
* Foundation imaging workflows.
* LCM and firmware lifecycle dependencies.
* Nutanix maximum configuration limits.
* Prism Central scale-out versus cluster node expansion.

### How to handle advanced questions

Use this pattern:

> “I understand the concept and the operational impact, but I would rely on the specialist or documentation for the exact limit or internal mechanism. As a support manager, my responsibility is to ensure we validate compatibility, collect the right evidence, involve the right domain expert, and communicate the risk clearly.”

That is credible. Do not pretend to be a Nutanix kernel/storage engineer.

---

## 12. Final checklist

Before an interview, make sure you can explain:

* [ ] What node-based scaling means.
* [ ] Why Nutanix uses a scale-out model.
* [ ] What a node contributes to a Nutanix cluster.
* [ ] What role the CVM plays.
* [ ] Difference between Prism Element and Prism Central.
* [ ] Why cluster health matters before expansion.
* [ ] Why networking can break node discovery.
* [ ] Why raw capacity differs from usable capacity.
* [ ] How replication factor affects capacity and resiliency.
* [ ] Why adding nodes may not fix performance.
* [ ] How to lead a customer escalation around failed expansion.
* [ ] How to communicate uncertainty without losing authority.
* [ ] How to connect this to SaaS/cloud scaling experience.
* [ ] How to position yourself as a technical escalation manager, not a Senior SRE IC.

---

## Annex I — Acronyms, terms, and keywords

| Term / Acronym / Keyword   | Meaning                                                                                          |
| -------------------------- | ------------------------------------------------------------------------------------------------ |
| AHV                        | Acropolis Hypervisor; Nutanix’s native hypervisor.                                               |
| AOS                        | Nutanix software layer providing distributed storage and data services.                          |
| Availability Domain        | Failure domain used to design resiliency across components such as nodes, blocks, or racks.      |
| BMC                        | Baseboard Management Controller; out-of-band hardware management interface.                      |
| Block                      | Physical enclosure that can contain multiple Nutanix nodes.                                      |
| Block Awareness            | Data placement behavior that avoids keeping redundant copies in the same block when supported.   |
| Capacity Planning          | Process of forecasting required compute, memory, storage, and network resources.                 |
| Cluster                    | Group of Nutanix nodes operating as one managed distributed system.                              |
| Cluster Expansion          | Process of adding one or more nodes to an existing Nutanix cluster.                              |
| Cluster Health             | Overall operational state of nodes, CVMs, services, disks, and alerts.                           |
| Cluster Maximums           | Supported limits for node count and other configuration values.                                  |
| Compute-only Node          | Node type mainly contributing compute resources rather than storage capacity.                    |
| Controller VM              | Nutanix VM running on each node and participating in distributed platform services.              |
| CVM                        | Controller VM.                                                                                   |
| Data Locality              | Keeping VM data close to the compute resource using it, where possible.                          |
| Data Resiliency            | Ability to tolerate failures while keeping workloads available and data protected.               |
| Discovery                  | Process by which Prism identifies unassigned nodes available for cluster expansion.              |
| Distributed Storage Fabric | Nutanix distributed storage architecture across cluster nodes.                                   |
| DNS                        | Domain Name System; name resolution service often required for stable infrastructure operations. |
| Effective Capacity         | Capacity after considering optimization such as compression or deduplication.                    |
| ESXi                       | VMware hypervisor that can be used in some Nutanix environments.                                 |
| Fault Domain               | Infrastructure boundary whose failure should be tolerated, such as node, block, or rack.         |
| Foundation                 | Nutanix tool commonly used for imaging and preparing nodes.                                      |
| Grafana                    | Monitoring and visualization tool; useful analogy for operational dashboards.                    |
| HCI                        | Hyperconverged Infrastructure; integrated compute, storage, virtualization, and management.      |
| Hot VM                     | VM generating disproportionate resource demand or performance pressure.                          |
| Hypervisor                 | Software layer that runs virtual machines on physical hosts.                                     |
| IPMI                       | Intelligent Platform Management Interface; out-of-band server management.                        |
| IPv6 Multicast             | Network mechanism relevant to some Nutanix node discovery workflows.                             |
| IOPS                       | Input/output operations per second; storage performance metric.                                  |
| Jira                       | Ticketing/workflow tool used for operational tracking.                                           |
| KPI                        | Key Performance Indicator.                                                                       |
| LCM                        | Life Cycle Manager; Nutanix lifecycle and upgrade management capability.                         |
| Linear Scaling             | Expectation that adding resources produces proportional performance or capacity gain.            |
| MTTR                       | Mean Time To Restore/Repair; key incident management metric.                                     |
| Multicast                  | Network communication pattern used for discovery or group communication.                         |
| Node                       | Physical server participating in the Nutanix cluster.                                            |
| Node-based Scaling         | Scaling method where capacity grows by adding complete nodes.                                    |
| NTP                        | Network Time Protocol; time synchronization service.                                             |
| Prism Central              | Centralized Nutanix management plane for multiple clusters and workflows.                        |
| Prism Element              | Management interface for an individual Nutanix cluster.                                          |
| Raw Capacity               | Total physical storage before protection and overhead.                                           |
| Rebalancing                | Redistribution of data or load after changes such as expansion.                                  |
| Rebuild                    | Process of restoring data protection after component failure or change.                          |
| Replication Factor         | Number of data copies maintained for protection.                                                 |
| RF2                        | Replication Factor 2; two copies of protected data.                                              |
| RF3                        | Replication Factor 3; three copies of protected data.                                            |
| Scale-out                  | Growth model based on adding more nodes.                                                         |
| Scale-up                   | Growth model based on increasing resources inside an existing system.                            |
| SLA                        | Service Level Agreement.                                                                         |
| SRE                        | Site Reliability Engineering / Engineer.                                                         |
| Storage-only Node          | Node type mainly contributing storage capacity.                                                  |
| Subnet                     | Logical IP network segment.                                                                      |
| Triage                     | Structured process to assess impact, symptoms, scope, and next actions.                          |
| Usable Capacity            | Capacity available after protection, overhead, and reserves.                                     |
| VLAN                       | Virtual LAN; network segmentation mechanism.                                                     |
| Workload Imbalance         | Uneven distribution of resource usage across nodes or hosts.                                     |

[1]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_5%3Awc-cluster-expand-wc-t.html?utm_source=chatgpt.com "Prism 7.5 - Expanding a Cluster - Nutanix"
[2]: https://portal.nutanix.com/page/documents/details?targetId=Prism-Central-Guide-vpc_2024_1%3Amul-node-add-pc-t.html&utm_source=chatgpt.com "Prism pc.2024.1 - Expanding a Cluster through Prism Central"
[3]: https://www.nutanixbible.com/pdf/5a-book-of-ahv-architecture.pdf?utm_source=chatgpt.com "AHV - AHV Architecture - NutanixBible.com"
[4]: https://portal.nutanix.com/docs/Prism-Central-Guide-vpc_7_3%3Amul-node-add-pc-t.html?utm_source=chatgpt.com "Prism pc.7.3 - Expanding a Cluster through Prism Central"
[5]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v7_3%3Aarc-data-resiliency-rack-c.html&utm_source=chatgpt.com "Prism 7.3 - Data Resiliency Levels for Block Fault Tolerance"
[6]: https://portal.nutanix.com/page/documents/details?targetId=Web-Console-Guide-Prism-v6_5%3Aarc-failure-modes-c.html&utm_source=chatgpt.com "Prism 6.5 - Availability Domains - Nutanix"
[7]: https://portal.nutanix.com/page/documents/details?targetId=Web_Console_Guide-NOS_v4_0%3Aarc_block_awareness_c.html&utm_source=chatgpt.com "NOS 4.0 - Block Awareness - Nutanix"
[8]: https://portal.nutanix.com/docs/Web-Console-Guide-Prism-v7_0%3Awc-cluster-expand-wc-r.html?utm_source=chatgpt.com "Prism 7.0 - Prerequisites and Requirements - portal.nutanix.com"
