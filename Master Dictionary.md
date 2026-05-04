# Nutanix Master Acronyms and Keywords Dictionary

Consolidated master dictionary for the Nutanix repository sections: `Hyperconverged Infrastructure`, `AHV and Virtualization`, and `AOS and Storage`.

> Deduplication rule: each term appears once globally. When a term appears in multiple section dictionaries, it is kept in the first topic location where it appears, without repeating the explanation.

## Coverage Summary

| Metric | Count |
|---|---:|
| Original rows across the three dictionaries | 638 |
| Unique master terms after global deduplication | 465 |

## Hyperconverged Infrastructure

### Core HCI Architecture

| Term / Acronym / Keyword | Meaning |
|---|---|
| 3-tier architecture | Traditional infrastructure model separating compute, network, and storage into independent layers. |
| AHV | Acropolis Hypervisor; Nutanix native virtualization hypervisor integrated with AOS and Prism. |
| AOS | Acropolis Operating System; Nutanix software layer providing distributed storage, data services, resiliency, upgrades, and cluster intelligence. |
| Availability domain | Failure boundary such as disk, node, block, rack, or site used to reason about resiliency. |
| Block | Physical Nutanix chassis/enclosure that may contain one or more nodes and can be treated as a failure domain. |
| Block awareness | Replica placement strategy designed to reduce impact from a block-level failure. |
| Cluster | Group of Nutanix nodes operating together as one distributed infrastructure platform. |
| Compute | CPU and memory resources used to run workloads. |
| Compute/storage convergence | Architectural principle of combining compute and storage resources into one distributed platform. |
| Converged infrastructure | Pre-integrated compute, network, and storage stack that may still keep compute and storage logically separate. |
| CVM | Controller Virtual Machine; critical Nutanix VM on each node providing storage and cluster services. |
| Data locality | Nutanix behavior that serves VM I/O near the VM location when possible to reduce latency and network traffic. |
| Distributed system | System composed of multiple cooperating components; central mental model for Nutanix clusters. |
| DSF | Distributed Storage Fabric; Nutanix distributed storage layer aggregating local disks into resilient shared storage. |
| Failure domain | Boundary where a component failure can affect service or data availability, such as disk, node, block, rack, or site. |
| HCI | Hyperconverged Infrastructure; architecture combining compute, storage, virtualization, networking integration, and management in a software-defined cluster. |
| Hybrid node | Node with mixed flash and disk storage, typically SSD/NVMe plus HDD. |
| Node | Physical server in a Nutanix cluster contributing compute and usually storage. |
| Prism | Nutanix management and monitoring interface for health, alerts, capacity, performance, tasks, and operations. |
| Prism Central | Centralized Nutanix management plane for multiple clusters and advanced operational capabilities. |
| Prism Element | Cluster-level Nutanix management interface for a single cluster. |
| Scale-out | Growth model where capacity and/or performance increases by adding nodes. |
| Shared-nothing | Architecture where each node owns local resources while distributed software coordinates the platform. |

### Storage, Data Services, and Resiliency

| Term / Acronym / Keyword | Meaning |
|---|---|
| Array | Centralized storage system providing block or file storage to servers. |
| Backup | Copy of data or workload state kept for recovery purposes. |
| Block storage | Storage model where data is exposed as fixed-size blocks, commonly through SAN technologies. |
| Capacity | Total usable or available resource amount in a system. |
| Capacity headroom | Free resource margin reserved for growth, rebuild, snapshots, recovery, and safe operations. |
| Centralized storage | Storage architecture where persistent data resides in dedicated external arrays rather than inside each compute host. |
| Compression | Storage efficiency technique that reduces the physical size of stored data. |
| Container / Storage Container | Logical Nutanix storage boundary presented to workloads or hypervisors and used for policy application. |
| DAS | Direct-Attached Storage; storage physically attached to a server. |
| Data placement | Process of deciding where data and replicas are stored across the cluster. |
| Data protection | Mechanisms such as snapshots, replication, backups, and redundancy used to prevent data loss. |
| Data resiliency | Ability to preserve availability and data integrity after failures. |
| Deduplication | Storage efficiency technique that avoids storing duplicate data blocks. |
| Degraded cluster | Cluster state where one or more components have failed or resilience is reduced. |
| Disk rebuild | Process of reconstructing data after a disk failure using redundancy mechanisms. |
| DR | Disaster Recovery; process and technology for recovering services after major failure, outage, or site loss. |
| EC-X | Erasure Coding Extension; Nutanix capacity optimization feature that reduces overhead compared with full replication. |
| Erasure coding | Data protection and capacity-efficiency technique using fragments and parity rather than full extra copies. |
| Failover | Moving or restarting workloads/services on a backup component, host, cluster, or site after failure. |
| Fault tolerance | Ability to continue operating after component failure. |
| HDD | Hard Disk Drive; magnetic disk storage often used for capacity tiers in hybrid systems. |
| Hotspot | Localized resource concentration causing performance degradation. |
| I/O | Input/Output; read and write operations between workloads and storage or network systems. |
| IOPS | Input/Output Operations Per Second; metric for storage read/write operation rate. |
| NAS | Network-Attached Storage; file-level storage accessed over a network. |
| NFS | Network File System; file protocol often used for VMware datastores. |
| NVMe | High-performance SSD interface/protocol used for low-latency storage. |
| Object storage | Storage model where data is stored as objects with metadata. |
| Physical disk | Actual storage hardware such as HDD, SSD, or NVMe. |
| QoS | Quality of Service; resource control or prioritization mechanism. |
| RAID | Traditional disk redundancy model; useful contrast with Nutanix distributed resiliency. |
| Raw capacity | Total physical disk capacity before resiliency, metadata, and operational overheads. |
| Rebalance / Rebalancing | Redistribution of data or load across nodes or disks. |
| Rebuild | Reconstruction of data redundancy after disk or node failure. |
| Replication | Maintaining additional copies of data for resilience or disaster recovery. |
| Replication Factor / RF | Number of data copies maintained for protection. |
| RF2 | Replication Factor 2; two copies of data. |
| RF3 | Replication Factor 3; three copies of data for higher resiliency. |
| SAN | Storage Area Network; dedicated storage architecture providing block-level storage. |
| Snapshot | Point-in-time copy of data or VM state. |
| SSD | Solid-State Drive; flash-based storage device. |
| Storage array | Dedicated external storage system used in traditional architectures. |
| Storage controller | Component that manages storage I/O and data services. |
| Storage fabric | Distributed storage layer coordinating storage resources. |
| Storage pool | Logical or physical aggregation of storage resources across a cluster. |
| Thin provisioning | Allocating logical capacity without reserving all physical capacity upfront. |
| Tiering | Placement or movement of data across media types such as NVMe, SSD, and HDD. |
| Usable capacity | Capacity available after metadata, resiliency, rebuild reserve, and other overheads. |
| VDI | Virtual Desktop Infrastructure. |
| vDisk | Virtual disk object consumed by a VM. |
| Volume | Logical block storage unit presented to a host or application. |

### Virtualization and Compute

| Term / Acronym / Keyword | Meaning |
|---|---|
| ADS | Acropolis Dynamic Scheduler; Nutanix service for workload placement and resource usage optimization. |
| APD | All Paths Down; VMware condition where a host temporarily loses all paths to storage. |
| Application tier | Application architecture layer where business logic runs; distinct from infrastructure 3-tier architecture. |
| CPU | Central Processing Unit; physical compute resource. |
| CPU contention | Multiple workloads competing for limited physical CPU resources. |
| CPU Ready | VMware metric showing time a VM is ready to run but waiting for physical CPU resources. |
| ESXi | VMware enterprise bare-metal hypervisor; Nutanix can run with AHV or ESXi in supported deployments. |
| HA | High Availability; design principle keeping services running or restarting workloads after failures. |
| Host | Physical server running a hypervisor and hosting VMs. |
| Host isolation | Condition where a host loses management or cluster connectivity and may trigger HA behavior. |
| Hyper-V | Microsoft hypervisor for running virtual machines. |
| Hypervisor | Software layer that runs and manages virtual machines. |
| KVM | Kernel-based Virtual Machine; Linux virtualization technology underlying AHV concepts. |
| Live migration | Moving a running VM from one host to another with minimal or no downtime. |
| Memory ballooning | Hypervisor technique used to reclaim memory from VMs during memory pressure. |
| Memory pressure | Condition where host or VM RAM is insufficient for demand. |
| NCI | Nutanix Cloud Infrastructure; Nutanix platform including AOS, AHV, DR, and related capabilities. |
| Noisy neighbor | Workload consuming excessive shared resources and affecting others. |
| NUMA | Non-Uniform Memory Access; server memory architecture relevant to large VM sizing and performance. |
| PDL | Permanent Device Loss; VMware condition where a storage device is permanently unavailable. |
| VM | Virtual Machine; software-defined server running on a hypervisor. |
| VMware | Enterprise virtualization and cloud infrastructure platform/vendor. |
| Workload | Application, VM, service, or process running on infrastructure. |

### Networking and Connectivity

| Term / Acronym / Keyword | Meaning |
|---|---|
| Backup traffic | Network traffic generated by backup systems when copying VM, file, database, or application data. |
| CRC errors | Cyclic Redundancy Check errors; link-level errors suggesting corruption or transmission issues. |
| Data plane | Infrastructure path carrying actual workload, storage, or application traffic. |
| DNS | Domain Name System; service resolving names to IP addresses. |
| East-west traffic | Traffic between systems inside a datacenter, cluster, or application environment. |
| Fabric | Network of switches and links, often used for SAN or datacenter switching. |
| FC | Fibre Channel; high-performance storage networking protocol commonly used in SAN environments. |
| Firewall | Security device or rule set controlling network traffic. |
| HBA | Host Bus Adapter; adapter connecting servers to storage networks, commonly Fibre Channel. |
| IP | Internet Protocol; core addressing and routing protocol for network communication. |
| IPMI | Intelligent Platform Management Interface; out-of-band server management interface. |
| iSCSI | Internet Small Computer Systems Interface; protocol carrying SCSI commands over IP networks. |
| Jumbo frames | Ethernet frames larger than standard MTU, often used in storage or high-throughput networks. |
| LACP | Link Aggregation Control Protocol; bundles multiple physical network links into one logical link. |
| LAN | Local Area Network; network connecting systems within a local site or datacenter. |
| Link flap | Network link repeatedly going up and down. |
| Management network | Network used for infrastructure management traffic. |
| Management plane | Control, configuration, monitoring, and administration layer. |
| Migration traffic | Traffic generated during VM live migration or storage migration. |
| MTU | Maximum Transmission Unit; maximum packet size on a network path. |
| Multipathing | Use of multiple storage paths for redundancy and load balancing. |
| NIC | Network Interface Card; adapter connecting a server to a network. |
| North-south traffic | Traffic entering or leaving a datacenter, cluster, or application environment. |
| NTP | Network Time Protocol; clock synchronization protocol. |
| Packet loss | Network condition where packets fail to reach the destination. |
| Production network | Network carrying application or user-facing service traffic. |
| Routing | Forwarding traffic between IP networks. |
| SAN zoning | Fibre Channel access control mechanism defining which initiators can see which targets. |
| Storage network | Specialized network path carrying storage traffic; part of the network tier in 3-tier architecture. |
| VLAN | Virtual Local Area Network; Layer 2 segmentation method. |

### Operations, Support, and Incident Management

| Term / Acronym / Keyword | Meaning |
|---|---|
| Alert | Notification indicating a health, capacity, performance, or resiliency condition. |
| Availability | Ability of a system or service to remain operational and accessible. |
| Blast radius | Scope of impact of an incident across users, services, VMs, hosts, clusters, or sites. |
| BMC | Baseboard Management Controller; out-of-band hardware management interface for health, power, and remote access. |
| Bottleneck | Constrained component limiting overall performance. |
| Business impact | Practical effect of an issue on customer operations, users, revenue, or SLA. |
| CE | Community Edition; free Nutanix edition used for labs and learning. |
| Change window | Approved period for planned upgrades, maintenance, or infrastructure change. |
| CLI | Command-Line Interface; text-based administration and troubleshooting interface. |
| Cloud operations | Operating cloud or cloud-like production infrastructure. |
| Confluence | Documentation and knowledge-management platform. |
| CSAT | Customer Satisfaction; support performance/customer sentiment metric. |
| Degradation | Partial service impairment where a system remains available but performs below expectation. |
| Enterprise support | Support for business-critical customer environments with strong SLAs and escalation expectations. |
| Escalation | Raising an issue to higher support, SRE, engineering, management, TAM, or account team. |
| Field Engineer | Engineer responsible for on-site hardware replacement or repair. |
| Firmware | Low-level software embedded in hardware devices such as NICs, BIOS, disks, or controllers. |
| GCP | Google Cloud Platform; public cloud provider relevant to hybrid cloud discussions. |
| Grafana | Monitoring and dashboarding platform for metrics and alerts. |
| Gray failure | Partial or intermittent failure without a clean up/down signal. |
| IC | Individual Contributor; hands-on technical role without people-management responsibility. |
| Incident management | Process for detecting, triaging, resolving, communicating, and reviewing incidents. |
| Insights | Nutanix operational analytics/telemetry context for alerts and health indicators. |
| ITSM | IT Service Management; processes and tools for incidents, changes, requests, and problems. |
| Jira | Issue and workflow tracking system used for incidents, escalations, bugs, and tasks. |
| K8s / Kubernetes | Container orchestration platform relevant to distributed systems and workload operations. |
| Kibana | Log analysis and visualization tool. |
| KPI | Key Performance Indicator; metric used to track system, service, or team performance. |
| Latency | Time taken for an operation to complete. |
| LCM | Life Cycle Manager; Nutanix software/firmware lifecycle and update management. |
| Linux | Operating system widely used in infrastructure and cloud environments. |
| Monitoring | Collection and analysis of metrics, logs, events, and alerts. |
| MTTR | Mean Time To Resolution/Restore/Repair; time required to recover service or resolve an incident. |
| NCC | Nutanix Cluster Check; health-check utility for Nutanix clusters. |
| Observability | Ability to understand system state through metrics, logs, traces, alerts, and events. |
| P1 | Priority 1 incident; usually critical production impact. |
| RCA | Root Cause Analysis; formal explanation of what happened, why, impact, and prevention. |
| RFO | Reason for Outage; explanation of outage cause. |
| Runbook | Documented operational procedure for troubleshooting or recovery. |
| SaaS | Software as a Service. |
| Sev1 | Severity 1; critical incident with major production or business impact. |
| SLA | Service Level Agreement; contractual or operational service commitment. |
| SRE | Site Reliability Engineering/Engineer; reliability, automation, performance, and operations role/discipline. |
| Support bundle | Diagnostic package collected for troubleshooting. |
| TAM | Technical Account Manager; customer-facing technical relationship owner. |
| Throughput | Amount of data transferred over time, often MB/s or GB/s. |
| Triage | Structured initial assessment of impact, scope, urgency, symptoms, and next action. |
| Worldwide Support | Global support organization operating across regions and time zones. |
| X-Play | Nutanix Prism Central automation capability based on triggers and actions. |

## AHV and Virtualization

### Core AHV and Virtualization Architecture

| Term / Acronym / Keyword | Meaning |
|---|---|
| aCLI / ACLI | Acropolis command-line interface used for AHV, VM, host, and cluster operations. |
| Acropolis | Nutanix software/platform family including AHV and virtualization management capabilities. |
| AHV CLI | Command-line interface used by engineers for AHV/Nutanix operations. |
| AHV Turbo | Optimized AHV I/O path designed to improve VM storage performance. |
| Bare metal | Software running directly on physical hardware. |
| Bare-metal hypervisor | Type 1 hypervisor installed directly on physical hardware. |
| Controller VM | Nutanix VM running storage and cluster services on each node; also called CVM. |
| Datastore | Logical storage location for VM files, especially in VMware contexts. |
| Distributed Storage Fabric | Nutanix distributed storage layer across cluster nodes. |
| Guest OS | Operating system running inside a VM. |
| libvirt | API/daemon commonly used to manage KVM/QEMU VMs. |
| nCLI | Nutanix command-line interface for cluster and platform management. |
| Nutanix Move | Nutanix tool for migrating workloads to AHV. |
| Nutanix Prism | Nutanix web management interface for clusters, hosts, VMs, health, and operations. |
| QEMU | User-space emulator/virtualizer providing virtual hardware and VM processes with KVM. |
| qemu-kvm | QEMU using KVM acceleration for virtual machines. |
| Type 1 hypervisor | Bare-metal hypervisor used for enterprise virtualization. |
| Type 2 hypervisor | Hosted hypervisor running on top of a general-purpose OS. |
| vCenter | VMware management platform. |
| virsh | Command-line tool often used with libvirt/KVM. |
| VM hardware | Virtualized devices presented to a guest OS. |
| VM process | Host-side process representing a running VM. |
| VMware Tools | Guest integration tools for VMware VMs. |

### CPU, Memory, NUMA, and Resource Scheduling

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
| CPU compatibility | Ability for VMs to run or migrate across hosts with different CPU capabilities. |
| CPU hot-add | Adding CPU to a running VM without powering it off, subject to platform limitations. |
| CPU locality | Running CPU work close to the memory it uses. |
| CPU overcommit | Assigning more virtual CPU capacity than available physical CPU capacity. |
| CPU Passthrough | VM directly exposes/uses host CPU features, which can restrict mobility. |
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

### VM Lifecycle, Migration, HA, and Placement

| Term / Acronym / Keyword | Meaning |
|---|---|
| ACPI Shutdown | Graceful OS-level VM shutdown mechanism. |
| Affinity | Policy tying a VM to a preferred or required host/group of hosts. |
| Affinity Policy | Rule controlling where a VM should or must run. |
| Affinity Rule | Policy constraining VM placement. |
| Anti-affinity | Policy that attempts to keep selected VMs apart on different hosts. |
| Availability Zone / AZ | Logical or physical failure domain for workload placement and migration. |
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
| Firmware Upgrade | Hardware-level software update that may require host maintenance. |
| GPU Passthrough | Direct GPU assignment to a VM; often restricts live migration. |
| Hard enforcement | Rule behavior that blocks actions violating the policy. |
| Hardware Replacement | Physical intervention requiring safe host preparation. |
| Host affinity | Policy restricting a VM to selected physical hosts. |
| Host Evacuation | Moving VMs away from a host, usually before maintenance. |
| Intra-cluster Migration | Migration between hosts inside the same cluster. |
| Lifecycle | Set of operational states/actions for a VM: create, configure, power, resize, migrate, protect, recover, delete. |
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

### Networking, Storage Devices, and Guest Integration

| Term / Acronym / Keyword | Meaning |
|---|---|
| ARP | Address Resolution Protocol; maps IP addresses to MAC addresses. |
| Bond | Logical grouping of physical NICs for redundancy/performance. |
| Bridge | Virtual network bridge connecting VM traffic to host networking. |
| Data path | Route taken by application or VM I/O through the infrastructure. |
| DHCP Guard | Hyper-V switch feature preventing rogue DHCP server behavior. |
| File-Level Restore | Recovery of individual files instead of restoring a full VM. |
| Guest Tools | Software inside a VM enabling guest-aware platform operations. |
| Network segmentation | Separation of management, storage, replication, or guest traffic into different networks. |
| NGT | Nutanix Guest Tools; guest integration software for Nutanix VMs. |
| Open vSwitch / OVS | Virtual switch technology/concept relevant to AHV host networking internals. |
| Packet capture | Diagnostic method for inspecting network packets. |
| Physical NIC bonding | Logical grouping of physical NICs, relevant when network symptoms affect VMs. |
| Protection Domain | Nutanix data protection construct used for snapshot/replication policies. |
| Quiescing | Pausing or coordinating writes so snapshots are more consistent. |
| RDMA | Remote Direct Memory Access; can reduce CPU/network overhead in some high-performance scenarios. |
| RPO | Recovery Point Objective; acceptable data loss measured in time. |
| RTO | Recovery Time Objective; target time to restore service. |
| Snapshot/replication impact | Performance or behavior effect from snapshots or replication during time-correlated cases. |
| SSR | Self-Service Restore; Nutanix feature for file-level recovery from snapshots/recovery points. |
| Storage Capacity | Available storage required on a destination or cluster for VM operations. |
| Storage latency | Delay in storage read/write operations. |
| Storage Locality | Degree to which VM data is local to the host running the VM. |
| VirtIO | Paravirtualized driver framework used for efficient VM devices on AHV/KVM. |
| VirtIO queueing | Storage/network queue behavior tied to VirtIO drivers and vCPU topology. |
| VM disk | Virtual disk resource attached to a VM. |
| vNIC | Virtual Network Interface Card attached to a VM. |
| VSS | Volume Shadow Copy Service; Windows framework for consistent snapshots. |
| VSS writer | Windows component coordinating application-specific VSS snapshots. |

### Operations, Support, and Escalation

| Term / Acronym / Keyword | Meaning |
|---|---|
| Bridge call | Live incident call with customer and technical teams. |
| Customer impact | Business effect of an issue on customer operations or users. |
| Database-specific tuning | Application/database optimization often outside Nutanix root cause but relevant in escalations. |
| Guest OS tuning | Linux/Windows tuning that may be required for high-performance workloads. |
| Maintenance window | Approved period for planned infrastructure work. |
| NCC health checks | Validation checks for cluster health and support diagnostics. |
| Storage internals | Advanced Nutanix components such as Stargate, Cassandra, and Curator; recognize names, route deep analysis to specialists. |
| Support Manager | Role responsible for escalation control, coordination, customer communication, and outcomes. |
| Supportability | Whether a configuration is supported and diagnosable by the vendor. |
| Workaround | Safe temporary action to reduce impact while root cause is investigated. |

### VMware, Hyper-V, and Migration Vocabulary

| Term / Acronym / Keyword | Meaning |
|---|---|
| Azure Local | Microsoft hybrid infrastructure platform using Azure-connected on-premises resources. |
| Azure Site Recovery | Microsoft DR service for replication, failover, and failback. |
| BIOS | Legacy firmware boot mode used by older VM types. |
| Broadcom | Owner/vendor associated with current VMware product documentation and support. |
| Checkpoint | Hyper-V point-in-time VM state, similar conceptually to a snapshot. |
| Cluster Shared Volumes / CSV | Microsoft clustering feature allowing nodes to access shared storage volumes. |
| Disaster Recovery / DR | Recovery strategy for site, platform, or major infrastructure failure. |
| Failback | Returning workloads to the original site after DR failover. |
| Failover Clustering | Microsoft clustering technology used for HA in Hyper-V environments. |
| Generation 1 VM | Hyper-V VM using legacy BIOS-style architecture. |
| Generation 2 VM | Hyper-V VM using UEFI-based architecture. |
| UEFI | Modern firmware boot mode used by many newer VM types. |
| vSphere | VMware virtualization suite including ESXi and vCenter. |

## AOS and Storage

### Core AOS Storage Architecture

| Term / Acronym / Keyword | Meaning |
|---|---|
| AOS Storage | Nutanix software-defined distributed storage fabric used by workloads running on the cluster. |
| Container | Logical storage boundary inside a Nutanix storage pool. |
| Extent | Logical unit/chunk of data used internally by AOS storage. |
| Extent Group | Physical/logical grouping of extents used for placement and replication. |
| Extent Store | Persistent bulk storage layer in the Nutanix storage architecture. |
| Genesis | Nutanix service framework that starts and monitors services. |
| Metadata | Data about data: mappings, placement, ownership, replicas, references, and state. |
| Metadata disk | Disk used by the CVM for metadata; relevant to supportability in some features. |
| Metadata service | Service involved in storing, accessing, coordinating, or scanning metadata. |
| NutanixManagementShare | Built-in Nutanix container used by platform features; should not be casually deleted. |
| Stargate | Nutanix/AOS service responsible for storage I/O handling and data path operations. |
| Storage Container | Logical storage object/policy boundary where VMs, files, or vDisks are placed. |
| Storage Policy | Policy-based mechanism for applying storage attributes at supported logical levels. |
| Unified Cache | Nutanix caching layer used in the storage architecture. |
| vBlock | Logical chunk of vDisk address space. |
| Volume Group | Nutanix block storage construct often used for iSCSI/direct block use cases. |

### Resiliency, Replication, and Failure Domains

| Term / Acronym / Keyword | Meaning |
|---|---|
| Cassandra | Distributed metadata store technology used by Nutanix services. |
| Checksum | Data-integrity mechanism used to detect corruption. |
| Cluster Resiliency | Ability of the cluster to tolerate failures while preserving service and data protection. |
| Data Resiliency Status | Prism/cluster indicator showing whether the cluster can tolerate failures safely. |
| Degraded Resiliency | State where expected protection level is reduced. |
| Degraded State | Condition where the cluster runs with reduced redundancy, resilience, or performance. |
| Disk Failure | Physical or logical failure/degradation of a storage device. |
| Fault Domain | Component or boundary that can fail, such as disk, node, block, rack, or site. |
| FT | Fault Tolerance. |
| Medusa | Metadata access layer/interface in front of Cassandra. |
| Node Failure | Loss or unavailability of a Nutanix node, host, CVM, or critical node component. |
| Paxos | Consensus algorithm used for consistency in distributed systems. |
| Quorum | Minimum number of members required for safe distributed-system decisions. |
| Re-protection / Reprotection | Restoring required redundancy after a failure or under-replication. |
| Rebalancing | Redistributing data or workload across healthy resources. |
| Rebuild Capacity | Capacity needed to restore redundancy after failed disk, node, block, or rack. |
| Rebuild Capacity Reservation | Reserved capacity feature for recovery/self-healing after failures. |
| Recovery Milestone | Measurable step in incident/rebuild recovery progress. |
| Redundancy factor | Level of data replica protection in a Nutanix cluster. |
| RF | Replication Factor; number of copies maintained for availability. |
| Risk Window | Period where the cluster is online but exposed to higher risk due to degraded resiliency. |
| Under-replicated | Data currently has fewer replicas than required. |
| Zeus | Nutanix library/interface for accessing Zookeeper-backed cluster configuration. |
| Zookeeper | Distributed coordination/configuration service used in Nutanix architecture. |

### Capacity, Data Efficiency, and Storage Economics

| Term / Acronym / Keyword | Meaning |
|---|---|
| Advertised Capacity | Logical capacity presented for a storage container. |
| Capacity deduplication | Deduplication aimed at reducing physical storage capacity usage. |
| Capacity Efficiency | Techniques such as compression, dedupe, and erasure coding to reduce physical storage usage. |
| Capacity optimization | Reducing physical storage usage through efficiency techniques. |
| Capacity Optimization Engine / COE | AOS mechanism for improving storage efficiency with data reduction techniques. |
| Capacity Reservation | Space reserved for specific operational or resiliency purposes. |
| Cold Tier | Lower-performance storage tier, often HDD-based in hybrid systems. |
| Compression delay | Time between data write and post-process compression. |
| Data Efficiency | Techniques such as compression, deduplication, cloning, thin provisioning, and erasure coding. |
| Data reduction | General category including compression, deduplication, and erasure coding. |
| Dense node | Nutanix node with high storage capacity; relevant to some dedupe support considerations. |
| Encrypted data | Data transformed for security; often compresses or deduplicates poorly. |
| Encryption | Protection of data at rest or in transit; affects efficiency behavior. |
| Fingerprint-on-write | Mechanism related to identifying duplicate data during write operations. |
| Fingerprinting | Creating identifiers for data blocks to detect duplicates. |
| Foreground I/O | Active application read/write traffic. |
| Hot Tier | High-performance storage tier, usually SSD/NVMe-based. |
| ILM | Intelligent/Information Lifecycle Management; data movement across tiers based on usage/access patterns. |
| Inline | Processing performed during the write path. |
| Inline compression | Compression performed during data ingestion/write path. |
| Logical Capacity | Capacity as seen by workloads before physical overhead or efficiency effects. |
| Logical usage | Capacity as seen before physical efficiency savings. |
| Physical Capacity | Actual disk capacity consumed at the infrastructure layer. |
| Physical usage | Actual storage consumed after efficiency techniques. |
| Post-process compression | Compression performed after data is written, usually by background jobs. |
| Reserved Capacity | Capacity explicitly reserved for a storage container. |
| Resilient Capacity | Capacity level accounting for ability to tolerate/recover from failures. |
| Storage efficiency | Reduction of physical capacity usage through clones, compression, deduplication, erasure coding, or thin provisioning. |
| Workload suitability | Assessment of whether a workload is appropriate for a feature such as dedupe/compression. |

### Data Path, Locality, and Performance

| Term / Acronym / Keyword | Meaning |
|---|---|
| Background job | Non-foreground task such as post-process compression, scan, cleanup, data optimization, or rebuild. |
| Cache | High-speed memory or flash used to accelerate read/write operations. |
| Curator | Nutanix background service/framework for scans, cleanup, data placement, rebalancing, optimization, and re-protection workflows. |
| Data Drive | Disk used to store user/workload data in the cluster. |
| Disk Slot | Physical location of a disk in a node/chassis. |
| Garbage collection | Process that reclaims unused storage and supports storage efficiency. |
| Hardware Alert | Prism or system alert indicating physical component issue. |
| Local I/O | I/O served through the CVM on the same node as the VM. |
| OpLog / Oplog | Nutanix write buffer/log/staging component used in the storage I/O path, especially for random writes. |
| Performance Degradation | System remains available but performs worse than expected. |
| Queue depth | Number of outstanding I/O operations a host, adapter, device, or VM can queue. |
| Remote I/O | I/O served through another CVM over the network when local path is unavailable or unsuitable. |
| SPDK | Storage Performance Development Kit; user-space storage framework. |
| Write Latency | Time required to complete a write operation. |

### Snapshots, Clones, Backup, and Recovery

| Term / Acronym / Keyword | Meaning |
|---|---|
| Application consistency | Snapshot/backup state where application writes are coordinated to improve recoverability. |
| Clone | Copy of a VM or disk created from an existing source. |
| Clone sharing | Capacity-efficient sharing of unchanged blocks between clone and source. |
| Consistency | Degree to which recovered data/application state is usable and coherent. |
| CSI | Container Storage Interface; standard for Kubernetes storage plugins. |
| EBS | Elastic Block Store; AWS block storage service. |
| Full copy | Complete duplication of all data blocks. |
| Golden image | Standard VM image used to create clones, templates, or many similar VMs. |
| Guest customization | Post-clone changes to OS identity, hostname, network, or application settings. |
| Machine ID | Unique OS or application identity that may need regeneration after cloning. |
| Nutanix Volumes | Nutanix block storage service. |
| Protected VM | VM covered by Nutanix data protection or replication configuration. |
| PVC | Persistent Volume Claim; Kubernetes request for persistent storage. |
| Redirect-on-write | Technique where new writes go to new blocks rather than overwriting existing ones. |
| Restore | Recovery of data or workload from backup, snapshot, or clone. |
| SID | Security Identifier; Windows identity value that may duplicate after cloning. |
| Snapshot retention | Policy defining how long snapshots are kept. |
| Template | Reusable VM base configuration for deployment. |
| Test/dev | Non-production environment used for testing and development. |
| Write divergence | Growth of unique data after clones start writing changes. |

### Protocols, Hypervisors, and Integrations

| Term / Acronym / Keyword | Meaning |
|---|---|
| aCLI | AHV command-line interface used for Nutanix virtualization operations. |
| External CVM IP | CVM management-facing IP address. |
| Foundation | Nutanix deployment/provisioning tool that configures nodes and CVMs during installation. |
| SMBv3 | Server Message Block version 3; file storage protocol. |
| UVM | User VM; customer workload virtual machine. |

### Operations, Support, and Escalation

| Term / Acronym / Keyword | Meaning |
|---|---|
| Boot Drive | Disk/device used for boot-related components, depending on platform. |
| Engineering escalation | Escalation to product engineering for suspected defect or deep product issue. |
| ETA | Estimated Time of Arrival/completion; used for rebuild completion estimates and updates. |
| Insights DB | Nutanix database used by Prism to surface operational and rebuild information. |
| P1 / P2 | Priority levels for severe support incidents. |
| P1 / Sev1 | Highest-priority incident, usually major production impact. |
