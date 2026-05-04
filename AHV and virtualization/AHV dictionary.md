# AHV and Virtualization — Acronyms and Keywords Dictionary

Consolidated glossary for the `AHV and virtualization` section.

> Scope: Consolidated from the accessible Nutanix repository section. Terms are deduplicated inside this file, alphabetized inside each semantic family, and intentionally allowed to repeat across the three section files.

## Core AHV and Virtualization Architecture

| Term / Acronym / Keyword | Meaning |
|---|---|
| aCLI / ACLI | Acropolis command-line interface used for AHV, VM, host, and cluster operations. |
| Acropolis | Nutanix software/platform family including AHV and virtualization management capabilities. |
| ADS | Acropolis Dynamic Scheduling; mechanism for VM placement, balancing, and hotspot reduction. |
| AHV | Acropolis Hypervisor; Nutanix native Type 1 hypervisor for running VMs. |
| AHV CLI | Command-line interface used by engineers for AHV/Nutanix operations. |
| AHV Turbo | Optimized AHV I/O path designed to improve VM storage performance. |
| AOS | Acropolis Operating System; Nutanix distributed software layer for HCI, storage, and platform services. |
| Bare metal | Software running directly on physical hardware. |
| Bare-metal hypervisor | Type 1 hypervisor installed directly on physical hardware. |
| Cluster | Group of Nutanix nodes/hosts operating as one system. |
| Controller VM | Nutanix VM running storage and cluster services on each node; also called CVM. |
| CVM | Controller VM; Nutanix infrastructure VM providing storage/control services on each node. |
| Datastore | Logical storage location for VM files, especially in VMware contexts. |
| Distributed Storage Fabric | Nutanix distributed storage layer across cluster nodes. |
| DSF | Distributed Storage Fabric; Nutanix distributed storage architecture used by AOS/AHV. |
| Guest OS | Operating system running inside a VM. |
| HCI | Hyperconverged Infrastructure; compute, storage, networking, and virtualization integrated into one platform. |
| Host | Physical server/node running AHV and VMs. |
| Hyper-V | Microsoft enterprise Type 1 hypervisor and common comparison/migration source. |
| Hypervisor | Virtualization layer that runs, isolates, and manages VMs. |
| KVM | Kernel-based Virtual Machine; Linux kernel virtualization technology underlying AHV concepts. |
| libvirt | API/daemon commonly used to manage KVM/QEMU VMs. |
| Linux | Kernel/OS family underlying KVM concepts and many infrastructure workloads. |
| Management plane | Control and administration layer such as Prism or vCenter. |
| NCI | Nutanix Cloud Infrastructure. |
| nCLI | Nutanix command-line interface for cluster and platform management. |
| Nutanix Move | Nutanix tool for migrating workloads to AHV. |
| Nutanix Prism | Nutanix web management interface for clusters, hosts, VMs, health, and operations. |
| Prism | Nutanix management and monitoring interface. |
| Prism Central | Centralized Nutanix management plane across clusters. |
| Prism Element | Cluster-level Nutanix management interface. |
| QEMU | User-space emulator/virtualizer providing virtual hardware and VM processes with KVM. |
| qemu-kvm | QEMU using KVM acceleration for virtual machines. |
| Type 1 hypervisor | Bare-metal hypervisor used for enterprise virtualization. |
| Type 2 hypervisor | Hosted hypervisor running on top of a general-purpose OS. |
| vCenter | VMware management platform. |
| virsh | Command-line tool often used with libvirt/KVM. |
| VM | Virtual Machine; software-defined server running on a hypervisor. |
| VM hardware | Virtualized devices presented to a guest OS. |
| VM process | Host-side process representing a running VM. |
| VMware | Enterprise virtualization platform/vendor. |
| VMware Tools | Guest integration tools for VMware VMs. |

## CPU, Memory, NUMA, and Resource Scheduling

| Term / Acronym / Keyword | Meaning |
|---|---|
| AMD-V | AMD CPU virtualization extension used by hypervisors. |
| APC | Advanced Processor Compatibility; feature to improve CPU compatibility for migrations. |
| Application latency | Time an application takes to respond; customer-visible performance symptom. |
| Balloon driver | Guest driver that helps reclaim unused VM memory for the hypervisor. |
| Ballooning | Memory reclamation method where the guest cooperates with the hypervisor. |
| Baseline | Known normal performance level used for comparison. |
| Boot storm | Many VMs booting simultaneously, often causing CPU, memory, or storage pressure. |
| Capacity planning | Ensuring enough resources for current/future workloads, HA, maintenance, and growth. |
| Contention | Multiple workloads competing for insufficient physical resources. |
| Core per socket / Cores per socket | VM CPU topology setting affecting how CPU is presented to the guest OS. |
| CPU | Central Processing Unit; physical compute resource. |
| CPU compatibility | Ability for VMs to run or migrate across hosts with different CPU capabilities. |
| CPU contention | Competition for physical CPU resources. |
| CPU hot-add | Adding CPU to a running VM without powering it off, subject to platform limitations. |
| CPU locality | Running CPU work close to the memory it uses. |
| CPU overcommit | Assigning more virtual CPU capacity than available physical CPU capacity. |
| CPU Passthrough | VM directly exposes/uses host CPU features, which can restrict mobility. |
| CPU Ready | VMware term for vCPU waiting time; useful conceptually for CPU scheduling delay. |
| CPU scheduler | Hypervisor component that places VM vCPUs onto physical CPU resources. |
| Dynamic scheduling | Automated workload placement and balancing based on resource conditions. |
| Guest swap | Swap configured inside the VM operating system. |
| Heterogeneous cluster | Cluster with hosts that have different hardware configurations. |
| Homogeneous cluster | Cluster with similar or identical hardware across nodes. |
| Host hotspot | Host under disproportionate resource pressure compared with others. |
| Host NUMA boundary | CPU and memory limit of a physical NUMA node on a host. |
| Host swap | Hypervisor-level swapping of VM memory to disk. |
| Hugepages | Large memory pages used to improve performance for some workloads. |
| Intel VT-x | Intel CPU virtualization extension used by hypervisors. |
| Kernel | Core operating system layer that manages hardware resources. |
| Local memory | Memory attached to the same NUMA node as the CPU executing the workload. |
| Memory hot-add | Increasing VM memory while powered on, subject to platform constraints. |
| Memory latency | Delay when CPU accesses memory; higher latency can reduce performance. |
| Memory overcommit | Allocating more VM memory than physically available using reclamation/swap mechanisms. |
| N+1 | Capacity model where one host failure can be tolerated. |
| Narrow VM | VM that fits within a single physical NUMA node. |
| NUMA | Non-Uniform Memory Access; architecture where memory access time depends on CPU-memory location. |
| NUMA boundary | CPU and memory boundary of a physical NUMA node. |
| NUMA node | CPU/memory locality group inside a physical server. |
| NUMA topology | Mapping of CPU sockets/cores and memory locality inside a host or VM. |
| Overcommit | Allocating more virtual resources than available physical resources. |
| Oversized VM | VM assigned more CPU or memory than it actually needs. |
| Oversubscription | Overcommitting virtual resources, often expressed as vCPU:pCPU ratios. |
| pCPU | Physical CPU core/thread available on a host. |
| Remote memory access | CPU accessing memory attached to a different NUMA node. |
| Resource contention | CPU, memory, storage, or network resource constraint affecting workloads. |
| Right-sizing | Adjusting VM resources to match actual workload demand. |
| Scheduler | System component that decides VM placement, movement, or CPU execution scheduling. |
| Swap | Disk-backed memory used when RAM is insufficient. |
| vCPU | Virtual CPU assigned to a VM. |
| vCPU:pCPU ratio | Ratio of allocated virtual CPUs to physical CPU resources. |
| vNUMA | Virtual NUMA topology presented to a VM for large workload performance optimization. |
| Wide VM | VM spanning more than one physical NUMA node. |
| Workload profile | Behavior pattern of an application: CPU, memory, I/O, latency sensitivity, and peaks. |

## VM Lifecycle, Migration, HA, and Placement

| Term / Acronym / Keyword | Meaning |
|---|---|
| ACPI Shutdown | Graceful OS-level VM shutdown mechanism. |
| Affinity | Policy tying a VM to a preferred or required host/group of hosts. |
| Affinity Policy | Rule controlling where a VM should or must run. |
| Affinity Rule | Policy constraining VM placement. |
| Anti-affinity | Policy that attempts to keep selected VMs apart on different hosts. |
| Availability | Ability of a service or VM to remain operational despite failures. |
| Availability Zone / AZ | Logical or physical failure domain for workload placement and migration. |
| Blast radius | Scope of impact when a component fails. |
| Block | Option that prevents maintenance from proceeding if non-migratable VMs exist. |
| Category | Prism Central metadata label used to group resources for policy management. |
| Cold Migration | Moving a VM while powered off or with downtime. |
| Compliance | Whether current VM placement satisfies the configured policy. |
| Cordon | Kubernetes term for marking a node unschedulable; useful maintenance analogy. |
| Correlated failure | Failure where multiple dependent components fail together. |
| Destination Cluster | Target cluster for cross-cluster migration. |
| Destination Host | Host where a VM will be moved. |
| Dirty Memory Pages | Memory pages modified while migration is in progress. |
| Drain | Operational process of moving workloads away from a node. |
| DRS | VMware Distributed Resource Scheduler; balances VMs across ESXi hosts. |
| Entering Maintenance Mode | Transitional state while host evacuation is completing or blocked. |
| Evacuation | Moving workloads away from a host before maintenance. |
| EVC | Enhanced vMotion Compatibility; VMware CPU compatibility feature. |
| Failover | Moving/restarting workloads on another host/site after failure. |
| Firmware Upgrade | Hardware-level software update that may require host maintenance. |
| GPU Passthrough | Direct GPU assignment to a VM; often restricts live migration. |
| HA | High Availability; ability to keep or restore workloads after host failure. |
| Hard enforcement | Rule behavior that blocks actions violating the policy. |
| Hardware Replacement | Physical intervention requiring safe host preparation. |
| Host affinity | Policy restricting a VM to selected physical hosts. |
| Host Evacuation | Moving VMs away from a host, usually before maintenance. |
| Intra-cluster Migration | Migration between hosts inside the same cluster. |
| Lifecycle | Set of operational states/actions for a VM: create, configure, power, resize, migrate, protect, recover, delete. |
| Live migration | Moving a running VM between hosts with minimal or no planned downtime. |
| Maintenance Mode | Operational state used to prepare a host for planned work by evacuating workloads. |
| Network Prefix | IP subnet portion that should match in some cross-cluster migration scenarios. |
| Non-migratable VM | VM that cannot be live migrated due to configuration or constraints. |
| OD-CCLM | On-Demand Cross-Cluster Live Migration; Nutanix feature for live migration across clusters. |
| PCI Passthrough | Direct PCI device assignment to a VM; can prevent migration. |
| Placement | Decision about which host runs a VM. |
| Placement constraint | Rule or condition limiting where a VM can run. |
| Policy violation | State where actual VM placement does not match the intended rule. |
| Production Workload | Business-critical application or VM used in live operations. |
| Recovery point | Point in time available for restore. |
| Replica | Redundant instance of a service or application component. |
| RF1 | Replication Factor 1; no redundant data copy, relevant to resiliency and maintenance risk. |
| Soft enforcement | Best-effort rule behavior that may not block every violation. |
| Soft shutdown | Graceful OS-level shutdown initiated through platform integration. |
| Source Cluster | Cluster where the VM currently runs before cross-cluster migration. |
| Source Host | Host currently running the VM before migration. |
| Task ID | Identifier for a Prism operation, useful for troubleshooting. |
| VM Mobility | Feature area related to VM migration/conversion scenarios. |
| VM UUID | Unique identifier for a VM, useful in logs and escalation evidence. |
| VM-Host Affinity | Policy tying a VM to one or more specific hosts. |
| VM-VM anti-affinity | Policy attempting to keep selected VMs on different hosts. |
| vMotion | VMware live migration feature; comparison point for AHV live migration. |
| Workload Mobility | Ability to move applications or VMs across hosts or clusters. |
| Workload placement | Assignment of VMs or services to physical infrastructure. |
| WSL2 | Windows Subsystem for Linux 2; listed in some AHV contexts as migration-relevant restriction. |

## Networking, Storage Devices, and Guest Integration

| Term / Acronym / Keyword | Meaning |
|---|---|
| ARP | Address Resolution Protocol; maps IP addresses to MAC addresses. |
| Bond | Logical grouping of physical NICs for redundancy/performance. |
| Bridge | Virtual network bridge connecting VM traffic to host networking. |
| Data path | Route taken by application or VM I/O through the infrastructure. |
| DHCP Guard | Hyper-V switch feature preventing rogue DHCP server behavior. |
| DNS | Domain Name System; resolves names to IP addresses. |
| East-west traffic | Network traffic between internal systems or VMs. |
| File-Level Restore | Recovery of individual files instead of restoring a full VM. |
| Firewall | Security device/rules controlling network traffic. |
| Guest Tools | Software inside a VM enabling guest-aware platform operations. |
| I/O | Input/output operations, usually disk or network activity. |
| IOPS | Input/output operations per second; storage performance metric. |
| IP | Internet Protocol address; network identifier for a system. |
| iSCSI | Block storage protocol over IP networks. |
| LACP | Link Aggregation Control Protocol; bundles network links. |
| MTU | Maximum Transmission Unit; maximum packet size on a network path. |
| Network segmentation | Separation of management, storage, replication, or guest traffic into different networks. |
| NFS | Network File System; file protocol commonly used for VMware datastores on Nutanix. |
| NGT | Nutanix Guest Tools; guest integration software for Nutanix VMs. |
| NIC | Network Interface Card. |
| Open vSwitch / OVS | Virtual switch technology/concept relevant to AHV host networking internals. |
| Packet capture | Diagnostic method for inspecting network packets. |
| Physical NIC bonding | Logical grouping of physical NICs, relevant when network symptoms affect VMs. |
| Protection Domain | Nutanix data protection construct used for snapshot/replication policies. |
| Quiescing | Pausing or coordinating writes so snapshots are more consistent. |
| RDMA | Remote Direct Memory Access; can reduce CPU/network overhead in some high-performance scenarios. |
| RPO | Recovery Point Objective; acceptable data loss measured in time. |
| RTO | Recovery Time Objective; target time to restore service. |
| Snapshot | Point-in-time capture of VM disk state or metadata. |
| Snapshot/replication impact | Performance or behavior effect from snapshots or replication during time-correlated cases. |
| SSR | Self-Service Restore; Nutanix feature for file-level recovery from snapshots/recovery points. |
| Storage Capacity | Available storage required on a destination or cluster for VM operations. |
| Storage latency | Delay in storage read/write operations. |
| Storage Locality | Degree to which VM data is local to the host running the VM. |
| vDisk | Virtual disk attached to/backing a VM. |
| VirtIO | Paravirtualized driver framework used for efficient VM devices on AHV/KVM. |
| VirtIO queueing | Storage/network queue behavior tied to VirtIO drivers and vCPU topology. |
| VLAN | Virtual Local Area Network used to segment Layer 2 networks. |
| VM disk | Virtual disk resource attached to a VM. |
| vNIC | Virtual Network Interface Card attached to a VM. |
| VSS | Volume Shadow Copy Service; Windows framework for consistent snapshots. |
| VSS writer | Windows component coordinating application-specific VSS snapshots. |

## Operations, Support, and Escalation

| Term / Acronym / Keyword | Meaning |
|---|---|
| Alert | Prism or system notification indicating health, performance, or operational issue. |
| Bridge call | Live incident call with customer and technical teams. |
| Confluence | Documentation platform used for runbooks, notes, and knowledge base. |
| Customer impact | Business effect of an issue on customer operations or users. |
| Database-specific tuning | Application/database optimization often outside Nutanix root cause but relevant in escalations. |
| Enterprise support | Support model for business-critical customer environments with SLAs and escalations. |
| Escalation | Process of raising a case to higher technical or management ownership. |
| Field Engineer | Engineer performing physical or onsite hardware work. |
| Firmware | Low-level software on hardware devices such as NICs, disks, BIOS, or controllers. |
| Grafana | Monitoring and visualization tool. |
| Guest OS tuning | Linux/Windows tuning that may be required for high-performance workloads. |
| Incident management | Process for restoring service and managing communication during incidents. |
| Jira | Issue and workflow tracking system. |
| Kibana | Log search and visualization tool. |
| KPI | Key Performance Indicator; operational measurement such as MTTR or SLA compliance. |
| Latency | Delay experienced by an application, VM, storage operation, or network request. |
| LCM | Lifecycle Manager; Nutanix tool for coordinated infrastructure updates. |
| Maintenance window | Approved period for planned infrastructure work. |
| MTTR | Mean Time To Resolve/Recover/Repair; support recovery metric. |
| NCC | Nutanix Cluster Check; health-check utility for Nutanix clusters. |
| NCC health checks | Validation checks for cluster health and support diagnostics. |
| P1 | Priority 1 incident; usually critical business impact. |
| RCA | Root Cause Analysis; post-incident explanation of cause, impact, and prevention. |
| Sev1 | Highest-severity incident, usually major production impact. |
| SLA | Service Level Agreement; contractual or operational service target. |
| SRE | Site Reliability Engineer/Engineering; role/discipline focused on reliability and operations. |
| Storage internals | Advanced Nutanix components such as Stargate, Cassandra, and Curator; recognize names, route deep analysis to specialists. |
| Support Manager | Role responsible for escalation control, coordination, customer communication, and outcomes. |
| Supportability | Whether a configuration is supported and diagnosable by the vendor. |
| Triage | Initial structured investigation to classify impact, cause, urgency, and next actions. |
| Workaround | Safe temporary action to reduce impact while root cause is investigated. |

## VMware, Hyper-V, and Migration Vocabulary

| Term / Acronym / Keyword | Meaning |
|---|---|
| Azure Local | Microsoft hybrid infrastructure platform using Azure-connected on-premises resources. |
| Azure Site Recovery | Microsoft DR service for replication, failover, and failback. |
| BIOS | Legacy firmware boot mode used by older VM types. |
| Broadcom | Owner/vendor associated with current VMware product documentation and support. |
| Checkpoint | Hyper-V point-in-time VM state, similar conceptually to a snapshot. |
| Cluster Shared Volumes / CSV | Microsoft clustering feature allowing nodes to access shared storage volumes. |
| Disaster Recovery / DR | Recovery strategy for site, platform, or major infrastructure failure. |
| ESXi | VMware Type 1 bare-metal hypervisor. |
| Failback | Returning workloads to the original site after DR failover. |
| Failover Clustering | Microsoft clustering technology used for HA in Hyper-V environments. |
| Fault Tolerance | VMware feature for higher VM availability using a secondary VM. |
| Generation 1 VM | Hyper-V VM using legacy BIOS-style architecture. |
| Generation 2 VM | Hyper-V VM using UEFI-based architecture. |
| Hyper-V | Microsoft enterprise Type 1 hypervisor. |
| NGT | Nutanix Guest Tools. |
| NTP | Network Time Protocol; time synchronization service. |
| PDL | Permanent Device Loss; VMware condition where storage device is permanently unavailable. |
| UEFI | Modern firmware boot mode used by many newer VM types. |
| vSphere | VMware virtualization suite including ESXi and vCenter. |
