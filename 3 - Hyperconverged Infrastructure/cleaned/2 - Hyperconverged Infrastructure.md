# 2 - Hyperconverged Infrastructure

## Purpose of this chapter

This chapter explains **Hyperconverged Infrastructure (HCI)** as the architectural answer to the operational complexity of traditional **3-tier infrastructure**.

It assumes the reader already understands the baseline model from chapter 1:

```text
Traditional 3-tier:
Compute hosts + network + SAN/storage fabric + external storage array
```

This chapter does not repeat every 3-tier detail. Instead, it focuses on what changes when compute, storage, virtualization, networking dependencies, and management are collapsed into a distributed software platform running on standard x86 nodes.

Recommended placement in Confluence:

1. Core definition
2. Practical idea
3. Interview-ready version
4. Comparison diagram: 3-tier vs HCI
5. What changes compared with 3-tier
6. Core Nutanix components
7. Support and escalation implications
8. Glossary

---

## Core definition

**Hyperconverged Infrastructure (HCI)** is a software-defined architecture that combines compute, storage, virtualization, networking integration, and management into a distributed cluster of standard servers.

Instead of separating the environment into independent compute hosts, SAN fabric, and storage arrays, HCI makes each node contribute resources to the cluster:

```text
Each HCI node contributes:
- CPU
- memory
- network interfaces
- local storage
- hypervisor services
- storage/control services
```

The cluster then uses software to make those local resources behave like a resilient shared infrastructure platform.

---

## The practical idea

In a traditional 3-tier architecture, compute and storage are physically and operationally separated.

```text
Compute hosts run VMs.
SAN connects compute to storage.
Storage arrays hold persistent data.
Cluster management coordinates compute failover.
```

In HCI, the model changes:

```text
Each node runs workloads and also contributes local storage.
A distributed software layer pools and protects that storage.
The cluster is managed as one integrated platform.
```

For Nutanix, the practical idea is:

> Each node contributes CPU, memory, local disks, network connectivity, AHV or another supported hypervisor, one Controller VM, and Nutanix AOS services. AOS builds the Distributed Storage Fabric, while Prism provides local cluster management through Prism Element and centralized management through Prism Central.

---

## Interview-ready version

> Hyperconverged Infrastructure replaces the traditional dependency on separate compute hosts, SAN fabrics, and external storage arrays with a distributed software-defined cluster. In Nutanix HCI, each x86 node contributes compute, networking, and local storage. AHV or another supported hypervisor runs the workloads, CVMs run Nutanix AOS services, and the Distributed Storage Fabric pools and protects local disks across the cluster. Prism Element manages the local cluster, while Prism Central manages multiple clusters and broader operations.

---

## Diagram placement

At the beginning of this chapter, show the **3-tier architecture** and **HCI architecture** diagrams side by side.

The objective is not to show every implementation detail. The objective is to make the reader immediately see the architectural shift:

```text
3-tier:
Compute, SAN, and storage are physically separate tiers.

HCI:
Compute, storage, and networking interfaces live inside standard x86 nodes,
and distributed software reconstructs the platform at cluster level.
```

Suggested caption:

> In 3-tier architecture, infrastructure responsibilities are physically separated. In HCI, standard nodes integrate compute, local storage, and network interfaces, while software such as AHV, CVM, AOS, DSF, and Prism turns those nodes into a resilient distributed platform.

---

## What changes compared with 3-tier

### Architectural change

In traditional 3-tier architecture, persistent VM data usually sits on external shared storage. Compute hosts access that storage through a SAN or storage network.

In HCI, storage is physically local to the nodes but logically distributed across the cluster.

This is the key distinction:

```text
3-tier:
VM runs on compute host.
VM disks live on shared storage array.
Another compute host can restart the VM because it can access the same shared storage.

HCI:
VM runs on an HCI node.
VM data is stored across local disks in the cluster through distributed storage.
Another node can restart the VM because protected copies already exist on surviving nodes.
```

HCI does **not** remove networking. Networking remains critical for:

- management traffic
- VM/application traffic
- CVM-to-CVM communication
- storage replication
- cluster health
- live migration or workload mobility
- backup and external integrations
- north-south connectivity to the rest of the datacenter or WAN

The important change is that HCI reduces the dependency on a separate external storage array and dedicated SAN as the central persistence layer.

---

## 3-tier vs Nutanix HCI

| Area | Traditional 3-tier | Nutanix HCI |
|---|---|---|
| Compute | Dedicated compute hosts | x86 HCI nodes contribute compute |
| Storage | External storage array | Local disks pooled by AOS/DSF |
| Storage fabric | SAN, Fibre Channel, iSCSI, NFS, or similar | Cluster network carries storage/CVM traffic |
| Hypervisor | ESXi, Hyper-V, KVM, etc. | AHV or another supported hypervisor |
| Storage controller | Storage array controllers | CVMs running AOS services |
| Management | Separate tools for compute, SAN, and storage | Prism Element for local cluster management |
| Scaling | Compute and storage often scale separately | Adding nodes can add compute, capacity, and performance |
| Failure model | Shared storage allows VM restart on another host | Replicated data allows VM restart on another node |
| Troubleshooting | Cross-tier and often cross-vendor | Distributed-system troubleshooting inside the HCI platform |

---

## Core Nutanix components

### AHV

**AHV** is the Nutanix native hypervisor. It is the host virtualization layer that runs on each AHV node and executes VMs.

At a high level, AHV is responsible for:

- running virtual machines
- scheduling vCPUs on physical CPU cores
- managing VM memory
- providing virtual networking through the host network stack
- providing the VM I/O path
- supporting HA and VM operations
- hosting the Controller VM

AHV should not be confused with AOS:

```text
AHV = hypervisor / host OS / virtualization layer.
AOS = distributed infrastructure and storage services layer.
```

A useful mental model is:

```text
CPU / RAM / NICs
   ↓
AHV / host OS / KVM-based hypervisor
   ↓
User VMs + Controller VM
```

### AOS

**AOS** is the Nutanix software platform. Historically, people often expand it as **Acropolis Operating System**, but in modern Nutanix language it is usually referred to simply as **AOS**.

AOS provides the distributed services that make HCI work:

- Distributed Storage Fabric
- storage pooling
- replication
- metadata management
- self-healing
- data services
- cluster intelligence
- lifecycle and operational services
- integration with Prism

The main conceptual warning is:

> Do not imagine AOS as a traditional operating system sitting underneath AHV.

A better model is:

```text
Hardware
  ↓
AHV host / hypervisor
  ├── User VMs
  └── Controller VM
       └── AOS services
            └── Distributed Storage Fabric
```

AOS services run through the CVM layer on each node and work together across the cluster.

### CVM

The **Controller VM (CVM)** is a special Nutanix infrastructure VM that runs on each node.

It is not a normal customer workload VM. It is part of the Nutanix platform.

The CVM is responsible for running important Nutanix services, including AOS storage services. CVMs across the cluster cooperate to deliver the Distributed Storage Fabric.

Simple model:

```text
User VM
  ↓
AHV
  ↓
Local CVM
  ↓
AOS / DSF services
  ↓
Local or remote storage resources across the cluster
```

The CVM is one of the most important support concepts because CVM health can affect:

- storage I/O
- cluster communication
- metadata services
- resiliency workflows
- Prism services
- rebuild and re-replication behavior
- performance during background activity

### Storage controller passthrough and the hardware/software boundary

One of the most useful mental models for Nutanix HCI is the split between compute virtualization and storage services.

For compute resources:

```text
CPU / RAM / NICs
   ↓
AHV host OS / hypervisor
   ↓
VMs and CVM
```

For storage resources:

```text
Storage controller / local disks
   ↓
PCIe / direct-path passthrough to the CVM
   ↓
CVM Linux kernel and storage drivers
   ↓
AOS services
   ↓
Distributed Storage Fabric
   ↓
Storage presented back to workloads through the virtualization layer
```

This does not mean “there is no kernel for storage.” It means the important storage I/O path is handled by the CVM and AOS services rather than by a traditional external array controller.

At a high level:

```text
AHV controls the host and runs VMs.
The CVM runs on AHV.
The CVM receives direct access to local storage devices.
AOS inside the CVM turns local disks into distributed storage.
```

This distinction matters because it explains why Nutanix HCI is not simply “servers with local disks.” The local disks become useful cluster storage only because AOS services running in CVMs coordinate placement, replication, metadata, and resiliency across the cluster.

### DSF

**Distributed Storage Fabric (DSF)** is the Nutanix distributed storage layer.

It aggregates local drives across nodes and presents resilient storage to workloads. In a diagram, DSF should not be drawn as only “the disks.” It is better understood as the combination of:

```text
CVMs + AOS services + local disks + cluster network + storage policies
```

The disks provide capacity. The CVMs and AOS services provide the intelligence.

### Storage pool

A **storage pool** is a logical construct built from physical devices across nodes.

The important idea is:

```text
Physical local disks across nodes
        ↓
AOS / DSF
        ↓
Cluster-wide storage pool
        ↓
Storage consumed by workloads
```

The storage pool is cluster-wide. It should not be understood as something that lives inside only one node or one CVM.

### Replication factor

**Replication Factor (RF)** defines how many copies of data the platform keeps.

Common examples:

```text
RF2 = two copies
RF3 = three copies
```

For example, with RF2:

```text
Data block X
 ├── Copy 1 on Node A
 └── Copy 2 on Node B
```

If Node A fails, the surviving copy on Node B remains available, and the cluster can re-protect the data by creating another copy elsewhere if capacity and health allow it.

Key clarification:

> If a node dies, other nodes do not magically access the failed node’s disks. Availability comes from the fact that protected copies already exist on surviving nodes.

For support, the key question is not only “is RF2 or RF3 configured?” but:

```text
Is the cluster currently healthy enough and does it have enough capacity to maintain the intended protection level?
```

### Data locality

Data locality means the platform tries to keep I/O close to where the VM is running when possible.

At a high level:

```text
Best case:
VM runs on Node A.
The needed data is local to Node A.
I/O is served locally.

If needed:
VM runs on Node A.
Some data is on Node B.
I/O uses the cluster network and CVM-to-CVM communication.
```

Data locality is a performance concept, not the same thing as data protection. Data protection comes from replication and failure-domain awareness.

### Prism Element

**Prism Element** is the local management interface for a Nutanix cluster.

For this chapter, think of it as the local cluster control and operations plane.

Prism Element provides visibility and operations for:

- cluster health
- node health
- CVM health
- storage capacity
- performance
- alerts
- VMs
- networking configuration visibility
- resiliency state
- lifecycle operations
- common administrative workflows

In a diagram, Prism Element can be drawn as a red control plane around the HCI cluster. That does **not** mean Prism is in the storage data path. It means Prism is the operational control and visibility plane for the local cluster.

### Prism Central

**Prism Central** provides centralized management across multiple Nutanix clusters.

Use this distinction:

```text
Prism Element = one cluster, local operations.
Prism Central = multiple clusters, centralized management and broader platform capabilities.
```

For this chapter, Prism Element is the main concept. Prism Central becomes more important when discussing multi-cluster management, fleet operations, automation, governance, reporting, and higher-level platform services.

---

## HCI failure model

In a 3-tier environment, VM failover usually depends on shared external storage.

```text
Host A fails.
Host B restarts the VM.
The VM disks are still available on the shared storage array.
```

In Nutanix HCI, the failure model is different:

```text
Node A fails.
The VM that was running on Node A stops.
Another node restarts the VM if HA policy and spare capacity allow it.
The VM data is available because AOS already kept protected copies on surviving nodes.
```

This is why spare capacity matters.

Spare capacity does not necessarily mean powered-off hardware. It means reserved or available capacity that can absorb failure, maintenance, rebalancing, or workload movement.

---

## Active capacity and spare capacity

The diagrams use **Active Capacity** and **Spare Capacity** instead of “primary” and “backup.”

That is intentional.

In both 3-tier and HCI, spare capacity may be:

- powered on
- already participating in the cluster
- used for some workloads
- reserved by policy
- available for HA events
- used during maintenance
- used during rebalancing

The important point is:

> Spare capacity is not necessarily passive. It is capacity available to preserve service when the environment changes or fails.

In HCI, spare capacity is especially important because a spare node can contribute:

- CPU
- RAM
- NICs
- local storage
- CVM/AOS services
- additional locations for replicated data

---

## Why this matters for Worldwide Support

Nutanix support escalations are distributed-system escalations.

A customer may report:

```text
“Storage latency is high.”
```

But the cause may involve:

- workload behavior
- backup or snapshot activity
- CVM load
- cluster network packet loss
- data locality
- rebuild or re-replication
- low resilient capacity
- disk errors
- NIC errors
- firmware or driver issues
- hypervisor behavior
- HA event side effects

The support manager’s job is to structure the case:

1. Separate business impact from technical symptoms.
2. Define the affected scope.
3. Identify recent changes.
4. Validate cluster health and resiliency.
5. Determine whether the issue is workload, compute, network, storage, capacity, or platform-state related.
6. Assign the right technical workstreams.
7. Communicate facts, risk, next actions, and customer impact clearly.

---

## Escalation example

Scenario:

> After a node failure, production VMs are slow and the customer asks whether data is safe.

Strong response:

> The platform is designed to maintain availability through replication, but we need to validate the current cluster state. We will check whether the cluster is RF-compliant, whether rebuild or re-replication is in progress, whether capacity is sufficient, and whether workloads are seeing performance impact. We will communicate data-risk assessment separately from performance-impact assessment.

Why this is good:

- It does not overpromise.
- It separates data safety from performance impact.
- It recognizes that RF configuration alone is not enough.
- It points to health, capacity, rebuild, and workload impact as separate workstreams.

---

## What not to repeat in later chapters

Later chapters should not re-explain AOS, CVM, AHV, Prism, RF2/RF3, and DSF from scratch.

Use this chapter as the reference point:

> See chapter 2 for the Nutanix HCI component model: AHV, CVM, AOS, DSF, RF, Prism Element, Prism Central, and the HCI failure model.

Chapter 3 can focus on scale-out behavior.

Chapter 4 can focus specifically on software-defined storage.

---

## Minimum to memorize

> HCI combines compute, storage, virtualization, networking dependencies, and management into a distributed software-defined cluster. In Nutanix, AHV runs workloads, CVMs run AOS services, AOS/DSF pools and protects local storage across nodes, and Prism Element provides local cluster management. HCI should be understood as a distributed system, not as a single appliance.

---

## Glossary: keywords and acronyms

| Term | Meaning |
|---|---|
| **HCI** | Hyperconverged Infrastructure. Architecture combining compute, storage, virtualization, networking integration, and management in a distributed cluster. |
| **3-tier architecture** | Traditional architecture separating compute, network/SAN, and storage array layers. |
| **AHV** | Acropolis Hypervisor. Nutanix native hypervisor used to run VMs. |
| **AOS** | Nutanix software platform that provides distributed storage and platform services. Historically associated with “Acropolis Operating System,” but usually referred to as AOS. |
| **CVM** | Controller VM. Special Nutanix infrastructure VM running AOS services on each node. |
| **DSF** | Distributed Storage Fabric. Nutanix distributed storage layer. |
| **RF** | Replication Factor. Number of protected data copies maintained by the platform. |
| **RF2** | Two copies of data. |
| **RF3** | Three copies of data. |
| **Prism Element** | Local cluster management and operations interface. |
| **Prism Central** | Centralized management for multiple Nutanix clusters and broader platform capabilities. |
| **Storage pool** | Logical pool built from physical storage devices across the cluster. |
| **Data locality** | Attempt to serve I/O close to where the VM is running. |
| **Spare capacity** | Available or reserved capacity used for HA, maintenance, rebalancing, or workload placement. |
| **Cluster network** | Network fabric used for management, VM traffic, CVM communication, and storage-related traffic. |
| **PCIe passthrough / Direct-path I/O** | Mechanism allowing a VM, such as the CVM, to directly access a physical PCIe device such as a storage controller. |
| **HA** | High Availability. Capability to restart workloads or preserve service after failures. |
