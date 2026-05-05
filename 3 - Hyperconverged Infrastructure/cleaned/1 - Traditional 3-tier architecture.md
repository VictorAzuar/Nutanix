# 1 - Traditional 3-tier architecture

## Purpose of this chapter

This chapter establishes the baseline infrastructure model that Nutanix HCI is designed to simplify.

Later chapters should not re-explain the full 3-tier model. They should refer back to this chapter when contrasting HCI with legacy infrastructure.

The goal is not to become a storage, network, or virtualization engineer in one chapter. The goal is to understand the architecture well enough to reason about support escalations, customer environments, and the operational differences between traditional infrastructure and HCI.

---

## Core definition

Traditional **3-tier infrastructure architecture** separates enterprise infrastructure into three main layers:

1. **Network**
2. **Compute**
3. **Storage**

This is not the same as **application 3-tier architecture**, which usually means web, application, and database tiers. Here we are talking about **infrastructure architecture**.

A simple definition:

> Traditional 3-tier infrastructure separates network, compute, and storage into distinct infrastructure domains. Compute runs the workloads, network connects the environment and carries different traffic types, and storage provides persistent shared data services through centralized arrays.

A strong interview answer:

> Traditional 3-tier infrastructure separates compute, network, and storage. Compute hosts run workloads through a hypervisor, storage arrays provide shared persistent data, and the network/SAN connects both worlds. The model is mature and reliable, but troubleshooting can be complex because incidents often cross layers, teams, and vendors.

---

## Why 3-tier matters before learning HCI

Nutanix HCI is easier to understand once the traditional model is clear.

In a classic 3-tier environment:

```text
Compute is separate from storage.
Storage is centralized.
The SAN connects compute to storage.
The hypervisor runs VMs on compute hosts.
The cluster control plane coordinates failover and capacity.
```

In Nutanix HCI, this changes:

```text
Compute and storage live in the same physical nodes.
Storage becomes distributed across the cluster.
The software layer abstracts local disks into a resilient storage fabric.
```

So 3-tier is the baseline. HCI is the contrast.

---

## High-level hardware model

At the hardware level, traditional 3-tier infrastructure can be understood as four physical domains:

```text
Network
   ↓
Compute
   ↓
SAN / Storage Fabric
   ↓
Storage Array
```

The **SAN** is not the storage itself. It is the connectivity layer between compute hosts and centralized storage.

The **storage array** is where the persistent data actually lives.

---

## Key hardware layers

### Network

The network layer provides connectivity for the environment.

It may carry several traffic types:

```text
- Management traffic
- VM/application traffic
- Live migration traffic
- Backup traffic
- Replication traffic
- Internet/WAN/corporate connectivity
- Storage traffic, depending on design
```

Typical components include:

```text
- Switches
- Routers
- Firewalls
- VLANs
- Routing
- Management networks
- Edge connectivity
```

Important clarification:

> Edge is not exactly the same as firewall. Edge usually refers to the boundary between the datacenter and external networks. A firewall may be part of the edge, but the edge can also include routing, NAT, VPN, load balancing, SD-WAN, or internet gateways.

Typical network-related symptoms:

```text
- VM connectivity issues
- Packet loss
- MTU mismatch
- VLAN misconfiguration
- DNS/NTP problems
- Storage path instability when storage uses IP networking
- Application timeouts
```

---

### Compute

Compute is where workloads execute.

It includes:

```text
- Physical servers
- CPU
- RAM
- NICs
- HBAs
- Firmware
- Hypervisor / host OS
- Virtual machines
```

Each compute host runs its own hypervisor. There is no single global hypervisor that magically owns all CPU and RAM across the entire cluster.

A better model is:

```text
Host 1:
  CPU/RAM → Hypervisor → VMs

Host 2:
  CPU/RAM → Hypervisor → VMs

Host 3:
  CPU/RAM → Hypervisor → VMs
```

The cluster management layer coordinates those hosts, but CPU and RAM remain physically local to each server.

Typical compute-related symptoms:

```text
- CPU contention
- Memory pressure
- Host failure
- Firmware mismatch
- Driver issues
- VM placement imbalance
- Noisy-neighbor workloads
```

---

### SAN / Storage Fabric

The SAN is the specialized connectivity layer between compute and storage.

It can be based on technologies such as:

```text
- Fibre Channel
- iSCSI
- NFS
- Multipathing
- Zoning
- LUN masking
```

The SAN is not a fourth main tier. It is usually treated as part of the broader network/storage connectivity model, but it deserves explicit attention because many performance and availability incidents happen here.

Important clarification:

> A storage network is not the same thing as a storage array. The SAN carries storage traffic. The storage array stores the data.

Typical SAN-related symptoms:

```text
- Path failure
- Multipathing misconfiguration
- HBA/NIC driver issues
- Fabric zoning issue
- LUN masking issue
- Queue depth saturation
- Storage latency visible from hosts
```

---

### Storage Array

The storage array provides centralized persistent data services.

It includes physical storage devices and storage controller logic:

```text
- HDDs
- SSDs
- NVMe drives
- Storage controllers
- Cache
- RAID or storage pools
- Volumes
- LUNs
- Snapshots
- Replication
- Thin provisioning
```

A simplified storage stack looks like this:

```text
Physical disks / SSDs
        ↓
RAID / storage pool
        ↓
Volumes / LUNs
        ↓
Presented to compute hosts through SAN
        ↓
Datastores / VM disks
```

Important clarification:

> Compute hosts usually do not consume raw physical disks from the storage array. They consume logical volumes or LUNs presented by the array.

Typical storage-related symptoms:

```text
- High datastore latency
- Controller failover
- Pool saturation
- Snapshot impact
- Replication lag
- Thin provisioning exhaustion
- Cache pressure
- Rebuild activity
```

---

## Software view on top of the hardware

The hardware diagram becomes more useful when we add the software/control layers.

A traditional 3-tier virtualization environment usually has:

```text
- Hypervisor / host OS on each compute server
- VMs running on the hypervisor
- Host kernel and I/O stack
- Virtual networking
- SAN/storage connectivity stack
- Storage array services
- Cluster management / HA control plane
```

---

## Compute software layer

Each compute host runs a hypervisor or host OS.

Examples include:

```text
- VMware ESXi
- Microsoft Hyper-V
- KVM-based platforms
- Proxmox VE
```

The hypervisor is responsible for virtualizing physical resources for VMs:

```text
Physical CPU → vCPU scheduling
Physical RAM → vRAM allocation
Physical NICs → virtual switches and vNICs
Storage paths → virtual disks/datastores
```

Inside the hypervisor/host OS, it is useful to think about:

```text
- Kernel
- CPU scheduler
- Memory manager
- Device drivers
- Virtual switch
- Storage I/O stack
- Multipathing client
- VM runtime
```

Do not think of this as separate “CPU kernels” and “RAM kernels”. It is one host OS/hypervisor layer with several subsystems.

A simplified compute software stack:

```text
Physical compute hardware
        ↓
Hypervisor / host OS
        ↓
VMs / workloads
```

---

## Storage I/O path

In traditional 3-tier, a VM’s virtual disk often lives on shared storage.

A simplified I/O path:

```text
VM
 ↓
Hypervisor / host I/O stack
 ↓
HBA or NIC
 ↓
SAN / storage fabric
 ↓
Storage array controller
 ↓
RAID / pool / cache
 ↓
Physical disks or SSDs
```

This is why storage issues can appear as VM performance issues, and why VM performance issues often require checking more than just CPU and memory.

---

## Cluster management / HA control plane

The cluster management layer coordinates compute hosts and workload availability.

It does not create one giant CPU or one giant RAM pool. Instead, it tracks hosts, capacity, VM state, storage accessibility, and policies.

Its responsibilities may include:

```text
- Host health monitoring
- VM restart after host failure
- VM placement
- Capacity awareness
- Admission control
- Live migration coordination
- Maintenance mode workflows
- Cluster alarms
```

Examples:

| Platform    | Hypervisor | Cluster management / HA control plane                                                  |
| ----------- | ---------- | -------------------------------------------------------------------------------------- |
| VMware      | ESXi       | vCenter Server + vSphere HA / DRS                                                      |
| Microsoft   | Hyper-V    | Windows Server Failover Clustering + Hyper-V Manager / SCVMM                           |
| Proxmox     | KVM/QEMU   | Proxmox VE Cluster Manager + HA Manager                                                |
| Nutanix HCI | AHV        | Prism Element for local cluster management; Prism Central for multi-cluster management |

VMware vSphere HA provides high availability for VMs in a vSphere cluster by restarting affected VMs, typically on another host, while vCenter is the central management plane for vSphere operations. ([VMware][1]) Microsoft documents Hyper-V failover clusters and VMM management for configuring cluster properties and managing nodes. ([Microsoft Learn][2]) Proxmox VE uses its cluster manager to group physical servers into a cluster, with Corosync used for reliable cluster communication. ([Proxmox VE][3]) Nutanix Prism Central is used for broader Nutanix environment management, while Prism Element is commonly understood as local cluster management. ([Nutanix Portal][4])

---

## Active capacity and spare capacity

In a 3-tier cluster, not all capacity should be consumed during normal operation.

A useful distinction is:

```text
Active capacity:
Resources currently running production workloads.

Spare capacity:
Reserved or available resources used for failover, maintenance, migration, or rebalancing.
```

Spare capacity does not necessarily mean powered-off hardware. It can be active infrastructure with enough free CPU, RAM, network, and storage access to absorb workloads during an incident.

Example:

```text
Normal state:
Host 1 runs VMs.
Host 2 runs VMs.
Host 3 has enough spare capacity.

Failure state:
Host 1 fails.
Cluster HA restarts Host 1's VMs on Host 2 and Host 3.
Shared storage allows those VMs to access their virtual disks.
```

Important clarification:

> In classic virtualization, the HA layer usually restarts or migrates VMs. It does not usually create a brand-new VM the way a Kubernetes controller might create a replacement pod.

---

## Why shared storage matters in 3-tier HA

Shared storage is one of the reasons traditional 3-tier architectures became so common.

If a compute host fails, another compute host can restart the affected VMs because the VM disks are not trapped inside the failed server.

Simplified model:

```text
VM runs on Host 1.
VM disk lives on shared storage.
Host 1 fails.
Host 2 can restart the VM using the same shared storage.
```

This is different from HCI.

In HCI, storage is local to each node but distributed and replicated by software. In 3-tier, storage is centralized and externally shared through the SAN/storage fabric.

---

## Redundancy model

Traditional 3-tier environments usually build redundancy at several layers.

| Layer   | Redundancy examples                                                    |
| ------- | ---------------------------------------------------------------------- |
| Network | Redundant switches, links, NIC teaming, routing failover, firewalls    |
| Compute | Multiple hosts, HA clusters, live migration, spare capacity            |
| SAN     | Dual fabrics, multipath I/O, redundant HBAs, zoning                    |
| Storage | Dual controllers, RAID/pools, cache protection, snapshots, replication |

The important operational point:

> A 3-tier environment can survive many individual component failures, but only if redundancy is correctly designed and validated across all layers.

A single missing path, misconfigured VLAN, incorrect zoning rule, or storage controller issue can break the availability story.

---

## Common failure scenario

Scenario:

> VMs are slow after a maintenance window.

Possible causes:

| Area                  | Examples                                                                              |
| --------------------- | ------------------------------------------------------------------------------------- |
| Compute               | CPU contention, memory pressure, host firmware issue, VM placement imbalance          |
| Network               | VLAN change, packet loss, MTU mismatch, DNS/NTP issue, management network instability |
| SAN                   | Storage path issue, HBA driver issue, multipathing problem, zoning issue              |
| Storage               | Array latency, controller failover, overloaded pool, snapshot or replication impact   |
| Cluster control plane | HA misconfiguration, admission control issue, DRS/placement imbalance                 |

Manager-level response:

> I would first define business impact, affected scope, and timeline. Then I would split the investigation across compute, network, SAN, storage, and cluster management. The goal is to isolate the failing domain using evidence rather than assumptions, avoid vendor finger-pointing, and keep customer communication structured.

---

## Why this matters for Nutanix support

Most Nutanix customers either migrated from 3-tier, still operate 3-tier alongside Nutanix, or compare Nutanix against a known SAN/NAS operating model.

A Worldwide Support Manager does not need to replace a senior storage engineer, but must understand the customer’s old architecture well enough to lead escalations intelligently.

The central management question is:

> Is the symptom coming from compute, network, SAN, storage, virtualization, application behavior, or a recent change?

This framework is reused throughout the rest of the HCI module.

---

## Comparison point: 3-tier vs Nutanix HCI

Traditional 3-tier:

```text
Compute hosts run hypervisors and VMs.
Storage arrays provide centralized shared storage.
SAN connects compute to storage.
HA depends heavily on shared storage accessibility.
```

Nutanix HCI:

```text
Each node contributes compute and local storage.
AOS/CVM provides distributed storage services.
AHV or another hypervisor runs workloads.
Data is replicated across nodes.
HA depends on surviving nodes having both compute capacity and replicated data.
```

The key contrast:

> 3-tier centralizes storage in an external array. HCI distributes storage across the cluster and abstracts it through software.

---

## Explanation pattern for interviews

Use this structure when explaining 3-tier architecture:

```text
1. Define the architecture.
2. Explain the three layers: network, compute, storage.
3. Clarify the role of SAN/shared storage.
4. Explain how HA works.
5. Mention operational complexity.
6. Contrast with HCI.
```

Example answer:

> Traditional 3-tier infrastructure separates compute, network, and storage. Compute hosts run hypervisors and VMs, storage arrays provide centralized persistent data, and the SAN or storage fabric connects them. The model enables shared datastores, live migration, HA, snapshots, replication, and mature operational practices. The trade-off is operational complexity: incidents often cross compute, network, SAN, and storage domains, which may be owned by different teams or vendors. Nutanix HCI simplifies this by collapsing compute and storage into distributed nodes and managing storage through software.

---

## Triage questions

Use these as the base checklist for 3-tier or HCI comparison cases:

1. What application or service is affected?
2. Is this an outage, degradation, or risk-only issue?
3. What changed recently?
4. Which hosts, VMs, networks, datastores, LUNs, arrays, or sites are in scope?
5. Are compute resources saturated?
6. Are there signs of CPU contention, memory pressure, or VM placement imbalance?
7. Are there packet drops, link errors, VLAN/MTU changes, DNS/NTP issues, or routing changes?
8. Are storage paths healthy from the host perspective?
9. Are there HBA/NIC driver, firmware, or multipathing issues?
10. Is storage latency high at datastore, array, controller, pool, or disk level?
11. Are backup, snapshot, replication, rebuild, or maintenance tasks running?
12. Has there been a controller failover, SAN change, zoning change, or LUN masking change?
13. Who owns each workstream?
14. What is the mitigation path?
15. When is the next customer update?

---

## Common traps

### Trap 1: Confusing infrastructure 3-tier with application 3-tier

Application 3-tier usually means:

```text
Web
Application
Database
```

Infrastructure 3-tier means:

```text
Network
Compute
Storage
```

They are different concepts.

---

### Trap 2: Treating SAN as the storage array

The SAN connects compute to storage. The storage array stores and presents data.

```text
SAN = storage connectivity fabric.
Storage array = storage system.
```

---

### Trap 3: Thinking the cluster has one giant CPU

In traditional virtualization, CPU and RAM remain local to each host. The cluster management layer schedules, migrates, and restarts workloads across hosts, but it does not merge all CPUs into one physical processor.

---

### Trap 4: Assuming spare capacity is passive

Spare capacity may be active, powered on, and partially used. The important point is that enough capacity remains available to absorb failover, maintenance, or rebalancing.

---

### Trap 5: Looking only at the layer where the symptom appears

A storage latency symptom may be caused by storage, SAN, network, host drivers, queue depth, snapshots, or workload behavior.

A VM connectivity symptom may be caused by the VM, virtual switch, physical NIC, VLAN, routing, firewall, DNS, or upstream network.

---

## Minimum to memorize

> Traditional 3-tier infrastructure separates network, compute, and storage. Compute hosts run hypervisors and VMs, the SAN connects compute to centralized shared storage, and the storage array presents logical volumes or LUNs built on physical disks, RAID/pools, and cache. The model is mature and powerful, but operationally complex because availability and performance issues often cross multiple layers, teams, and vendors. Nutanix HCI changes the model by distributing storage services across cluster nodes and integrating operations through software.

---

# Appendix - Keywords and acronyms

## Architecture

| Term                  | Meaning                                                                     |
| --------------------- | --------------------------------------------------------------------------- |
| 3-tier infrastructure | Infrastructure model separating network, compute, and storage               |
| HCI                   | Hyper-Converged Infrastructure                                              |
| Active capacity       | Capacity currently running production workloads                             |
| Spare capacity        | Reserved or available capacity for failover, migration, or maintenance      |
| HA                    | High Availability                                                           |
| Control plane         | Software layer that makes management, orchestration, or placement decisions |
| Data plane            | Path where actual workload traffic or I/O flows                             |

## Compute and virtualization

| Term           | Meaning                                                                                       |
| -------------- | --------------------------------------------------------------------------------------------- |
| Compute host   | Physical server running workloads                                                             |
| Hypervisor     | Software layer that virtualizes hardware resources for VMs                                    |
| Host OS        | Operating system or hypervisor layer running directly on the server                           |
| VM             | Virtual Machine                                                                               |
| vCPU           | Virtual CPU assigned to a VM                                                                  |
| vRAM           | Virtual memory assigned to a VM                                                               |
| vNIC           | Virtual network interface assigned to a VM                                                    |
| VM runtime     | Hypervisor component responsible for running VMs                                              |
| CPU scheduler  | Hypervisor subsystem that maps vCPUs to physical CPU time                                     |
| Memory manager | Hypervisor subsystem that manages RAM allocation                                              |
| Overcommit     | Allocating more virtual resources than physically available, based on expected usage patterns |
| Live migration | Moving a running VM from one host to another with minimal downtime                            |
| DRS            | Distributed Resource Scheduler, VMware feature for workload placement and balancing           |
| ESXi           | VMware bare-metal hypervisor                                                                  |
| Hyper-V        | Microsoft hypervisor                                                                          |
| KVM            | Kernel-based Virtual Machine, Linux virtualization technology                                 |
| QEMU           | Emulator/virtualization component commonly used with KVM                                      |
| Proxmox VE     | Virtualization platform based on KVM/QEMU and containers                                      |

## Network

| Term                      | Meaning                                                             |
| ------------------------- | ------------------------------------------------------------------- |
| Network fabric            | Physical and logical network connectivity across the environment    |
| Edge                      | Boundary between datacenter/internal networks and external networks |
| Firewall                  | Security device or service controlling traffic between zones        |
| VLAN                      | Virtual LAN, logical segmentation of a Layer 2 network              |
| Routing                   | Layer 3 forwarding between networks                                 |
| Mgmt / Management network | Network used to manage infrastructure components                    |
| VM network                | Network used by virtual machines                                    |
| vSwitch                   | Virtual switch inside a hypervisor                                  |
| MTU                       | Maximum Transmission Unit                                           |
| NTP                       | Network Time Protocol                                               |
| DNS                       | Domain Name System                                                  |
| WAN                       | Wide Area Network                                                   |
| NAT                       | Network Address Translation                                         |
| SD-WAN                    | Software-defined WAN                                                |

## SAN and storage connectivity

| Term           | Meaning                                                                    |
| -------------- | -------------------------------------------------------------------------- |
| SAN            | Storage Area Network                                                       |
| Storage fabric | Dedicated or specialized connectivity between compute and storage          |
| FC             | Fibre Channel                                                              |
| FC Fabric      | Fibre Channel switching fabric                                             |
| iSCSI          | SCSI storage protocol transported over IP networks                         |
| NFS            | Network File System                                                        |
| HBA            | Host Bus Adapter, commonly used for Fibre Channel connectivity             |
| NIC            | Network Interface Card                                                     |
| MPIO           | Multipath I/O                                                              |
| Multipathing   | Using multiple paths between host and storage for resilience/performance   |
| Zoning         | Fibre Channel mechanism controlling which initiators can see which targets |
| LUN masking    | Storage array mechanism controlling which hosts can access which LUNs      |
| Queue depth    | Number of outstanding I/O requests allowed in a path/device                |

## Storage

| Term              | Meaning                                                            |
| ----------------- | ------------------------------------------------------------------ |
| Storage array     | Centralized storage system with controllers and disks/flash        |
| Controller        | Storage array component that manages I/O and data services         |
| HDD               | Hard Disk Drive                                                    |
| SSD               | Solid State Drive                                                  |
| NVMe              | High-performance storage protocol/interface for flash storage      |
| RAID              | Redundant Array of Independent Disks                               |
| Storage pool      | Logical grouping of physical disks used to create volumes          |
| Cache             | Fast memory used to accelerate reads/writes                        |
| Volume            | Logical storage object presented by a storage system               |
| LUN               | Logical Unit Number, block storage object presented to hosts       |
| Datastore         | Hypervisor-visible storage location for VM files/disks             |
| Snapshot          | Point-in-time copy or reference of data                            |
| Replication       | Copying data to another system or site                             |
| Thin provisioning | Presenting more logical capacity than physically allocated upfront |
| Deduplication     | Eliminating duplicate data blocks                                  |
| Compression       | Reducing data size to save capacity                                |
| Rebuild           | Process of restoring redundancy after disk/component failure       |

## Nutanix contrast terms

| Term          | Meaning                                                                                |
| ------------- | -------------------------------------------------------------------------------------- |
| Nutanix HCI   | Nutanix hyper-converged infrastructure platform                                        |
| AHV           | Acropolis Hypervisor                                                                   |
| AOS           | Acropolis Operating System, Nutanix distributed infrastructure/storage software        |
| CVM           | Controller VM running Nutanix services on each node                                    |
| DSF           | Distributed Storage Fabric                                                             |
| Prism Element | Local Nutanix cluster management interface                                             |
| Prism Central | Centralized/multi-cluster Nutanix management interface                                 |
| RF2 / RF3     | Replication Factor 2 or 3, data redundancy policy in Nutanix-style distributed storage |



[1]: https://www.vmware.com/docs/availability-of-vcenter-server?utm_source=chatgpt.com "Availability of vCenter Server - VMware"
[2]: https://learn.microsoft.com/en-us/windows-server/virtualization/hyper-v/failover-cluster-network-recommendations?utm_source=chatgpt.com "Network recommendations for a Hyper-V cluster | Microsoft Learn"
[3]: https://pve.proxmox.com/wiki/Cluster_Manager?utm_source=chatgpt.com "Cluster Manager - Proxmox VE"
[4]: https://portal.nutanix.com/docs/Prism-Central-Guide-vpc_7_3%3Amul-pc-overview-c.html?utm_source=chatgpt.com "Prism pc.7.3 - Prism Central Overview - Nutanix"
