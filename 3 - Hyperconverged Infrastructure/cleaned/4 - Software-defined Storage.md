# 4 - Software-defined Storage

## Purpose of this chapter

This chapter explains **software-defined storage (SDS)** in Nutanix HCI.

It assumes the reader already understands:

- chapter 1: the traditional 3-tier baseline
- chapter 2: the Nutanix HCI component model
- chapter 3: scale-out cluster behavior

This chapter goes deeper into the storage side of HCI, but it still stays at a high-to-medium level. The goal is not to become an AOS storage engineer in one chapter. The goal is to understand where storage intelligence lives, how it differs from an external storage array, and how to reason about storage symptoms in support situations.

---

## Core definition

**Software-defined storage (SDS)** separates storage services from a dedicated external storage array and implements those services in software across standard servers.

In Nutanix, SDS is delivered through **AOS** and the **Distributed Storage Fabric (DSF)**.

Local disks inside cluster nodes are:

```text
pooled
protected
monitored
replicated
balanced
presented to workloads as resilient storage
```

Interview-ready version:

> Software-defined storage means that storage intelligence is implemented in software rather than being tied to a traditional external array. In Nutanix, AOS and DSF pool local disks across nodes, protect data through replication and failure-domain awareness, and present resilient storage to workloads through the distributed storage fabric.

---

## The practical idea

In traditional infrastructure, a storage array is a specialized system.

It usually includes:

```text
- storage controllers
- disks / SSDs / NVMe
- cache
- RAID or storage pools
- LUNs / volumes
- snapshots
- replication
- deduplication / compression
- firmware / array OS
- management tools
```

Compute hosts consume that storage through a SAN or storage network.

In Nutanix HCI, the storage intelligence moves into the distributed platform:

```text
Local disks in each node
        ↓
CVMs running AOS services
        ↓
Distributed Storage Fabric
        ↓
Cluster-wide storage pool
        ↓
Storage consumed by VMs/workloads
```

This is the architectural shift:

```text
Traditional model:
Servers depend on an external shared storage array.

Nutanix SDS model:
Cluster nodes contribute local disks, and AOS turns them into distributed resilient storage.
```

---

## Relationship to chapter 2

Chapter 2 introduced the core HCI model:

```text
AHV = hypervisor / host OS
CVM = infrastructure VM running Nutanix services
AOS = distributed platform and storage services
DSF = distributed storage fabric
Prism Element = local cluster management
```

This chapter focuses specifically on the storage part:

```text
How do local disks become shared resilient storage?
```

The short answer is:

> Nutanix does not depend on a central external array. Instead, CVMs running AOS services coordinate local storage across nodes and expose it as a distributed storage system.

---

## Why SDS matters in HCI

In traditional 3-tier infrastructure, storage intelligence lives mainly in the storage array.

The array handles:

- RAID
- caching
- LUN creation
- snapshots
- replication
- storage controller failover
- deduplication
- compression
- tiering
- performance optimization

In Nutanix HCI, storage intelligence becomes part of the distributed platform.

This matters because it changes:

- where storage services run
- how failures are handled
- how capacity is interpreted
- how performance is investigated
- how scaling works
- how support cases should be triaged

SDS does **not** remove storage complexity. It changes where that complexity lives.

Better language:

> Nutanix changes where storage complexity lives. It moves many storage services into distributed software, which can simplify operations, but support still requires strong understanding of capacity, resiliency, locality, network health, CVM health, and workload behavior.

---

## The storage data path at a high level

A simplified write path looks like this:

```text
User VM
  ↓
AHV / virtual disk path
  ↓
Local CVM
  ↓
AOS / DSF services
  ↓
Local storage and remote replicas
```

A simplified distributed model:

```text
Node A CVM ┐
Node B CVM ├── Distributed Storage Fabric
Node C CVM ┘
        ↓
Cluster-wide storage pool
```

The exact implementation is deeper than this chapter needs. The important conceptual point is:

> Storage I/O is handled by the Nutanix distributed storage layer, not by an external storage array.

---

## Hardware/software boundary

One of the confusing parts of Nutanix is the relationship between AHV, the CVM, AOS, and the physical disks.

At a high level, for compute resources:

```text
CPU / RAM / NICs
   ↓
AHV host OS / hypervisor
   ↓
VMs and CVM
```

For storage resources:

```text
Storage controller / disks
   ↓
PCIe / direct-path passthrough to CVM
   ↓
CVM Linux kernel and storage drivers
   ↓
AOS services
   ↓
Distributed Storage Fabric
   ↓
Storage consumed by workloads
```

This means:

- AHV runs the VMs.
- The CVM is hosted by AHV.
- The CVM runs AOS services.
- The CVM receives direct access to local storage devices.
- AOS/DSF turns those devices into part of a distributed storage system.

Do not overinterpret this as two operating systems competing for the same disks.

The better model is:

```text
AHV controls the host virtualization layer.
CVM/AOS controls the storage service layer.
The disks are made available to the CVM for storage services.
```

---

## Core SDS ideas

### 1. Storage is pooled

Each node contributes local storage devices:

```text
- NVMe
- SSD
- HDD, depending on platform/generation
```

AOS aggregates those devices into a logical cluster-wide storage system.

Workloads do not need to know which physical disk contains a specific block. They consume logical storage.

Conceptually:

```text
Physical devices
        ↓
Storage pool
        ↓
Storage containers / logical presentation
        ↓
VM disks / workload storage
```

### 2. Storage services are distributed

In a storage array, controllers are centralized around the array.

In Nutanix, CVMs across the cluster participate in storage services.

This is why CVM health matters so much:

```text
CVM health affects storage I/O.
CVM-to-CVM communication affects distributed storage behavior.
Cluster network health affects replication and remote reads/writes.
```

### 3. Protection is software-driven

Traditional arrays often rely heavily on RAID and controller-level redundancy.

Nutanix uses distributed software mechanisms, including:

- replication factor
- placement rules
- metadata services
- checksums
- self-healing
- failure-domain awareness
- re-replication after failures

RF2 and RF3 are the simplest entry points:

```text
RF2 = two copies of data
RF3 = three copies of data
```

Example:

```text
RF2:
Data block X exists on Node A and Node B.

If Node A fails:
The copy on Node B remains available.
The cluster can create another copy elsewhere when conditions allow.
```

Key support idea:

> RF configuration describes intended protection. Actual protection also depends on current cluster health, available capacity, failure domains, and ongoing background activity.

### 4. Storage pool is logical, not a physical box

A storage pool should not be imagined as a physical container inside one node.

Better:

```text
The storage pool is a cluster-wide logical construct built from physical storage devices across nodes.
```

This is why adding nodes can expand available storage capacity and performance.

### 5. Performance is tied to locality and network health

Because storage is distributed, performance depends on more than disk speed.

Relevant factors include:

- local disk health
- SSD/NVMe tier behavior
- CVM CPU and memory pressure
- inter-node network latency
- packet loss
- MTU or VLAN problems
- data locality
- rebuild or re-replication
- snapshot or backup activity
- workload write patterns
- capacity pressure

This is different from a basic external-array mental model where “storage latency” is immediately blamed on the array.

### 6. Capacity must be interpreted carefully

In SDS, raw capacity is not the same as usable capacity or resilient capacity.

Useful distinctions:

| Concept | Meaning |
|---|---|
| Raw capacity | Total physical capacity before protection/overhead |
| Usable capacity | Capacity available after protection and platform overhead |
| Free capacity | Currently unused logical capacity |
| Resilient capacity | Capacity available while still being able to tolerate failures and rebuild safely |

A cluster can be technically “not full” and still be operationally risky if it cannot re-protect data after a failure.

---

## SDS vs traditional storage array

| Area | Traditional storage array | Nutanix SDS |
|---|---|---|
| Storage location | External array | Local disks across HCI nodes |
| Intelligence | Array controllers / firmware | CVMs running AOS services |
| Protection | RAID, array replication, controller redundancy | RF2/RF3, placement, self-healing, failure-domain awareness |
| Presentation | LUNs / volumes through SAN/NAS | Logical storage through DSF/AOS |
| Scaling | Add shelves/controllers or replace array | Add nodes/capacity to cluster |
| Failure model | Array controller/disk/shelf redundancy | Distributed data copies across nodes |
| Troubleshooting | Array-centric plus SAN path | Workload + CVM + network + capacity + node health |

---

## Relationship between SDS and DSF

**Software-defined storage** is the general architectural concept.

**Distributed Storage Fabric** is Nutanix’s distributed storage layer that implements that concept inside HCI.

Use this distinction:

```text
SDS = architectural category.
DSF = Nutanix distributed storage fabric.
AOS = Nutanix software platform providing DSF and related services.
CVM = runtime location for AOS services on each node.
```

---

## What SDS is not

SDS does not mean:

```text
- there is no hardware
- storage no longer fails
- network no longer matters
- performance is automatic
- capacity planning disappears
- all disks behave like one magic disk
- troubleshooting becomes trivial
```

SDS means:

```text
Storage intelligence is implemented in software,
distributed across nodes,
and integrated with the HCI platform.
```

---

## Support implications

When a customer says:

```text
“Storage is slow.”
```

Do not assume the root cause is only disks.

Possible domains:

| Domain | Examples |
|---|---|
| Workload | High write rate, bursty database, backup job, snapshot activity |
| Compute | CPU contention, memory pressure, noisy neighbor |
| CVM/AOS | CVM resource pressure, service issue, rebuild, re-replication |
| Network | CVM-to-CVM packet loss, MTU issue, congestion, VLAN issue |
| Capacity | Low headroom, insufficient resilient capacity, hot tier |
| Hardware | Disk errors, NIC errors, firmware/driver issue |
| Platform state | Upgrade, expansion, node failure, maintenance, balancing |

Manager-level framing:

> I would treat storage latency as a symptom, not a root cause. In Nutanix, storage performance can be influenced by workload behavior, CVM health, network consistency, capacity, background resiliency activity, and hardware state.

---

## Escalation example

Scenario:

> A database VM shows high write latency after a backup window.

Structured response:

1. Confirm business impact and scope.
2. Determine whether the issue is one VM, one node, one container/storage object, or cluster-wide.
3. Check whether the timing overlaps with backup, snapshot, replication, migration, rebuild, expansion, upgrade, or failure activity.
4. Validate CVM health, node health, disk health, and network errors.
5. Check capacity and resilient capacity.
6. Identify whether latency is local, remote, workload-induced, or background-task related.
7. Assign technical owners for workload, storage/AOS, network, and customer communication.

Customer-facing language:

> We are validating whether this is a storage-system issue or a workload and cluster-state interaction. We will check latency, CVM health, network health, background activity, and capacity before identifying the safest mitigation path.

---

## Triage questions

1. What workload is affected?
2. Is the symptom latency, IOPS, throughput, timeout, or alerting?
3. Is the issue isolated to one VM, one node, one storage container, one workload type, or the whole cluster?
4. Did it start after backup, snapshot, migration, expansion, upgrade, failover, or node/disk replacement?
5. Are all CVMs healthy and reachable?
6. Are there disk, node, NIC, firmware, or driver alerts?
7. Is rebuild, re-replication, balancing, or cleanup running?
8. Is capacity or resilient capacity under pressure?
9. Are there packet drops, CRC errors, MTU issues, VLAN issues, or CVM-to-CVM network problems?
10. Is the workload generating unusual write pressure?
11. Are snapshots, clones, backups, or replication jobs active?
12. What is the safest workaround: move workload, throttle backup, add capacity, pause risky changes, or engage engineering?

---

## What to avoid

Avoid saying:

> Nutanix removes storage complexity.

Better:

> Nutanix changes where storage complexity lives. Many storage services move from a dedicated array into distributed software running across the HCI cluster.

Avoid saying:

> RF2 means the data is always safe.

Better:

> RF2 means the platform intends to maintain two copies, but we still need to verify current cluster health, capacity, and RF compliance.

Avoid saying:

> Storage latency means the disks are slow.

Better:

> Storage latency can involve workload behavior, CVM health, network health, data locality, rebuild activity, capacity pressure, or hardware issues.

---

## Relationship to other chapters

- Chapter 1 explains the traditional external storage model.
- Chapter 2 introduces AHV, AOS, CVM, DSF, RF, and Prism.
- Chapter 3 explains why SDS benefits from scale-out nodes.
- Chapter 8 explains data locality.
- Chapter 9 explains resiliency and failure behavior.
- Chapter 10 discusses architectural trade-offs.

This chapter should not re-explain the whole HCI model from scratch. It should deepen the storage side of the model introduced in chapter 2.

---

## Minimum to memorize

> Software-defined storage means storage services are delivered by software rather than by a dedicated external storage array. In Nutanix, AOS and DSF pool local disks across nodes, protect data through replication and failure-domain awareness, and present resilient storage to workloads. In support, storage symptoms must be investigated across workload, CVM, network, capacity, hardware, and background activity.

---

## Glossary: keywords and acronyms

| Term | Meaning |
|---|---|
| **SDS** | Software-defined storage. Storage services implemented in software rather than tied to a dedicated external array. |
| **AOS** | Nutanix software platform providing distributed storage and platform services. |
| **DSF** | Distributed Storage Fabric. Nutanix distributed storage layer. |
| **CVM** | Controller VM. Infrastructure VM running Nutanix AOS services on each node. |
| **AHV** | Nutanix native hypervisor. Runs VMs and hosts the CVM. |
| **Storage pool** | Logical pool built from physical devices across cluster nodes. |
| **Storage container** | Logical storage construct consumed by workloads/hypervisors. |
| **RF** | Replication Factor. Number of copies maintained for protected data. |
| **RF2** | Two copies of data. |
| **RF3** | Three copies of data. |
| **Data locality** | Serving I/O close to where the VM is running when possible. |
| **Re-replication** | Recreating protected data copies after a failure or topology change. |
| **Self-healing** | Platform behavior that restores protection after failures when resources allow. |
| **Failure domain** | Boundary used to avoid placing protected copies in the same failure area. |
| **Resilient capacity** | Capacity available while still preserving the ability to tolerate failures and rebuild safely. |
| **PCIe passthrough / direct-path I/O** | Mechanism allowing a VM such as the CVM to access a physical PCIe device directly. |
| **CVM-to-CVM traffic** | Network traffic between Controller VMs for distributed storage and cluster services. |
| **IOPS** | Input/output operations per second. |
| **Latency** | Time taken to complete an I/O operation. |
| **Throughput** | Volume of data transferred over time. |
